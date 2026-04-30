# `uart.c`

Dit bestand bevat de implementaties voor `kernel/uart.c`. Hieronder vind je de uitleg en de broncode van **elke functie** in dit bestand.

## `uartinit()`
> for sending threads to synchronize with uart "ready" interrupts.

```c
static struct spinlock tx_lock;
static int tx_busy;           // is the UART busy sending?
static int tx_chan;           // &tx_chan is the "wait channel"

extern volatile int panicking; // from printf.c
extern volatile int panicked; // from printf.c

void
uartinit(void)
{
  // disable interrupts.
  WriteReg(IER, 0x00);

  // special mode to set baud rate.
  WriteReg(LCR, LCR_BAUD_LATCH);

  // LSB for baud rate of 38.4K.
  WriteReg(0, 0x03);

  // MSB for baud rate of 38.4K.
  WriteReg(1, 0x00);

  // leave set-baud mode,
  // and set word length to 8 bits, no parity.
  WriteReg(LCR, LCR_EIGHT_BITS);

  // reset and enable FIFOs.
  WriteReg(FCR, FCR_FIFO_ENABLE | FCR_FIFO_CLEAR);

  // enable transmit and receive interrupts.
  WriteReg(IER, IER_TX_ENABLE | IER_RX_ENABLE);

  initlock(&tx_lock, "uart");
}

```

## `uartwrite()`
> transmit buf[] to the uart. it blocks if the uart is busy, so it cannot be called from interrupts, only from write() system calls.

```c
void
uartwrite(char buf[], int n)
{
  acquire(&tx_lock);

  int i = 0;
  while(i < n){ 
    while(tx_busy != 0){
      // wait for a UART transmit-complete interrupt
      // to set tx_busy to 0.
      sleep(&tx_chan, &tx_lock);
    }   
      
    WriteReg(THR, buf[i]);
    i += 1;
    tx_busy = 1;
  }

  release(&tx_lock);
}

```

## `uartputc_sync()`
> write a byte to the uart without using interrupts, for use by kernel printf() and to echo characters. it spins waiting for the uart's output register to be empty.

```c
void
uartputc_sync(int c)
{
  if(panicking == 0)
    push_off();

  if(panicked){
    for(;;)
      ;
  }

  // wait for UART to set Transmit Holding Empty in LSR.
  while((ReadReg(LSR) & LSR_TX_IDLE) == 0)
    ;
  WriteReg(THR, c);

  if(panicking == 0)
    pop_off();
}

```

## `uartgetc()`
> try to read one input character from the UART. return -1 if none is waiting.

```c
int
uartgetc(void)
{
  if(ReadReg(LSR) & LSR_RX_READY){
    // input data is ready.
    return ReadReg(RHR);
  } else {
    return -1;
  }
}

```

## `uartintr()`
> handle a uart interrupt, raised because input has arrived, or the uart is ready for more output, or both. called from devintr().

```c
void
uartintr(void)
{
  ReadReg(ISR); // acknowledge the interrupt

  acquire(&tx_lock);
  if(ReadReg(LSR) & LSR_TX_IDLE){
    // UART finished transmitting; wake up sending thread.
    tx_busy = 0;
    wakeup(&tx_chan);
  }
  release(&tx_lock);

  // read and process incoming characters, if any.
  while(1){
    int c = uartgetc();
    if(c == -1)
      break;
    consoleintr(c);
  }
}

```

