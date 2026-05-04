# Locking Guide (xv6)

In xv6 zijn er drie belangrijke lagen van locking waar je rekening mee moet houden bij het implementeren van functies (vooral system calls die met het bestandssysteem werken):

1. **FS Transactions (Logging)**: Om crash-consistentie op disk te garanderen.
2. **Sleeplocks (Inodes/Buffers)**: Voor operaties die lang kunnen duren (zoals disk I/O).
3. **Spinlocks**: Voor korte operaties, CPU-exclusief.

---

## 1. FS Transactions (`begin_op` / `end_op`)
Elke operatie die de schijf aanpast (schrijven naar een inode, aanmaken van bestanden, directory entries aanpassen) **moet** verpakt zijn in een transactie.

- **`begin_op()`**: Start een transactie. Als het log vol is, wacht het proces tot er ruimte is.
- **`end_op()`**: Beëindigt de transactie. Als dit de laatste actieve transactie is, worden alle wijzigingen "gecommit" naar de log op disk.

**Gouden Regel:** Doe NOOIT disk-gerelateerde aanpassingen (zoals `iupdate` of `writei`) buiten een `begin_op` / `end_op` blok.

---

## 2. Inode Lifecycle & Locking
Inodes hebben een referentie-count (`ip->ref`) en een sleeplock.

### Referenties (`namei`, `iput`)
- **`namei(path)`**: Zoekt een bestand op en returnt een pointer naar de inode (`struct inode *`). Dit verhoogt de `ref` count. Je "bezit" nu een referentie naar dit object in het geheugen.
- **`iput(ip)`**: Verlaagt de `ref` count. Als de count nul bereikt, wordt de inode uit de actieve cache verwijderd.

### Locking (`ilock`, `iunlock`)
Een inode "hebben" via `namei` is niet genoeg; je moet hem **locken** voordat je de velden (zoals `ip->type`, `ip->size`) mag lezen of aanpassen.
- **`ilock(ip)`**: Verwerft de sleeplock. Indien de inode-data nog niet in het geheugen staat, wordt deze nu van disk gelezen.
- **`iunlock(ip)`**: Geeft de sleeplock vrij. Andere processen kunnen de inode nu locken.
- **`iunlockput(ip)`**: Een handige shortcut voor `iunlock(ip)` gevolgd door `iput(ip)`.

---

## 3. Voorbeeld Analyse: Het "chmod" Patroon
Stel je wilt de `mode` (permissies) van een bestand aanpassen. Dit is hoe je dat veilig doet:

```c
uint64
sys_chmod(void)
{
  char path[MAXPATH];
  int mode;
  struct inode *ip;

  // 1. Haal argumenten op
  if(argstr(0, path, MAXPATH) < 0 || argint(1, &mode) < 0)
    return -1;

  // 2. START TRANSACTIE
  begin_op();

  // 3. Zoek inode op (verhoogt ref count)
  if((ip = namei(path)) == 0){
    end_op(); // Altijd transactie sluiten bij fout!
    return -1;
  }

  // 4. LOCK de inode voor aanpassing
  ilock(ip);

  // 5. Pas metadata aan
  ip->mode = mode;

  // 6. SYNC naar disk (BELANGRIJK!)
  // Zonder iupdate() worden je wijzigingen niet naar de disk geschreven.
  iupdate(ip);

  // 7. UNLOCK en release referentie
  iunlockput(ip);

  // 8. EINDE TRANSACTIE
  end_op();

  return 0;
}
```

### Waarom deze specifieke volgorde?
- **`begin_op` voor `namei`**: `namei` kan zelf disk-operaties triggeren (directories lezen). Deze moeten binnen een transactie vallen.
- **`iupdate` binnen de lock**: Je moet de lock vasthouden terwijl je de inode naar de buffer cache schrijft om inconsistenties te voorkomen.
- **`end_op` als allerlaatste**: Dit garandeert dat de hele actie (inode opzoeken + aanpassen + syncen) atomair gebeurt in het log.

---

## 4. Spinlocks vs Sleeplocks
| Eigenschap | Spinlock (`struct spinlock`) | Sleeplock (`struct sleeplock`) |
| :--- | :--- | :--- |
| **Wacht-gedrag** | CPU blijft draaien ("spinnen") | Proces gaat slapen (yield CPU) |
| **Interrupts** | Worden uitgeschakeld op deze CPU | Blijven ingeschakeld |
| **Mag slapen?** | **NEE!** (Veroorzaakt paniek) | **JA** |
| **Gebruik** | Voor korte, kritieke secties (scheduler, locks) | Voor lange operaties (Disk I/O, Inodes) |

**De belangrijkste regel:** Houd nooit een spinlock vast terwijl je een functie aanroept die kan slapen. Functies zoals `ilock`, `bread`, `copyin`, `copyout` en `printf` kunnen allemaal de CPU afstaan!

---

## 5. Veelgemaakte Fouten (Pitfalls)
- **Vergeten van `end_op()`**: Als je een `return -1` doet halverwege een transactie zonder `end_op()`, zal het systeem uiteindelijk vastlopen omdat er geen nieuwe transacties meer gestart kunnen worden.
- **Aanpassen zonder `iupdate()`**: Je wijziging lijkt te werken in de huidige sessie, maar na een reboot is alles weer bij het oude.
- **Deadlocks**: Twee inodes tegelijk locken is gevaarlijk. Als je dit doet, lock ze dan altijd in een vaste volgorde (bijv. de inode met het laagste inode-nummer eerst).
