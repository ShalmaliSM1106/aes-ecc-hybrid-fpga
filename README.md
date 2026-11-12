# AES + ECC on FPGA

## A bit of background

There are two broad kinds of encryption. Symmetric encryption like AES uses the same key to lock and unlock the data. It's fast and fairly cheap to build in hardware, which is why it's used for almost all bulk data. The catch is that both sides need the same key, and you can't just send the key over the network, because anyone listening would get it too.

Asymmetric (public-key) encryption solves that problem. With elliptic-curve cryptography (ECC), two sides can each combine their own private key with the other side's public key and end up with the same shared secret, without that secret ever being sent. ECC gives strong security with smaller keys than older methods like RSA, but it's much slower than AES.

So real systems use both: ECC once at the start to agree on a key, then AES for everything after that. HTTPS and a lot of IoT devices work this way. Building that combination in hardware was the idea behind this project.

## About this project

Our 6th-semester project: ECC is used to share a secret key, and that key is then used by an AES-128 core to encrypt and decrypt data. Written in Verilog, simulated in ModelSim, and synthesized for Artix-7 in Vivado.

We published this as a conference paper: *Design and Implementation of Cryptographic Models for VLSI Applications*, [conference, year, link].

Team: Shalmali Mankikar, [teammate], [teammate], [teammate]. I wrote the AES core and the ECC key-exchange module, and put the two together in the top module.

## How it works

The AES core is iterative: one round of logic used 10 times, rather than 10 copies of the round. It's smaller, but it takes more clock cycles per block. The S-box is a lookup table.

For ECC, we used [curve / key size] with [point multiplication method, e.g. double-and-add]. Once both sides have the shared secret, it goes into the AES key input.

![block diagram](docs/block_diagram.png)

## Files

```
rtl/   aes128_core.v  aes_sbox.v  key_expansion.v  ecc_key_exchange.v  hybrid_top.v
tb/    tb_aes128.v  tb_hybrid_top.v
sim/   run.do
docs/  block diagram, waveforms, Vivado report
```

Run with `cd sim` and `vsim -do run.do`.

## Numbers (Artix-7)

- AES core: [ ] LUTs, [ ] FFs, [ ] MHz, [ ] cycles per block, so about [ ] Mbps
- ECC module: [ ] LUTs, [ ] FFs, [ ] MHz
- Whole design: [ ] LUTs

![encrypt and decrypt waveform](docs/waveform_encrypt_decrypt.png)

[In your words: the hardest part to debug, e.g. byte ordering, key expansion timing, getting decrypt to match.]

## What I'm adding now

- checking the AES core against the official FIPS-197 test vectors, with a testbench that reports pass/fail on its own
- random test vectors generated with Python (pycryptodome)
- a UART wrapper so it can run on a board: send text from my laptop, get the encrypted bytes back
