_difficulty: 3.2_

## Intro
Patch the License Check (Easy)  
Crack the Validation "Algorithm" (Medium)

Ok this challenge gives us 2 optional way to solve so let try each way.

Example license key:
- HACK--JAZZ
- 9W7OEmss5f

> In this report I focus on **solving the algorithm** (keygen).  
---

## Disassemble & Analyst

First we need go through `main()` to understand program flow because all C/C++ program always start with `main()` first then it calls other `func()` inside `main()`.

In this crackme, `main()` does **NOT** validate your key by direct string compare.  
Instead it **builds a bytecode program** on the stack, then calls a small **VM interpreter** to execute that bytecode.  
The VM returns 1 byte (AL). If that byte is `0` → key is valid.

So our real goal is: **reverse VM + recover formula**.

---

## Digging inside `main()`

When we open `main()` (pseudo), we can split it into these parts:

1. Print prompt
2. Read 10-char license key (In0..In9)
3. Build VM bytecode buffer (240 bytes) on stack (var_200 / `v41`)
4. Call `vm_interpreter(...)` / `sub_140001EC0`
5. If return == 0 → success, else retry
![[{1221519A-AEB7-4440-A583-60375747AC85}.png]]

### The important part: where input bytes are injected
![[{49943893-C4D6-47B4-A8DA-E66656BDA27C}.png]]
After reading the key, `main()` starts filling a buffer (often shown as `v41`, `var_200`, etc.) with values like:

- `v41[0] = 73;` (ASCII 'I')
- `v41[1] = (BYTE)v35;`     (In0)
- `v44 = BYTE1(v35);`        (In1)
- `v48 = BYTE2(v35);`        (In2)
- `v51 = BYTE3(v35);`        (In3)
- `v57 = BYTE4(v35);`        (In4)
- `v60 = BYTE5(v35);`        (In5)
- `v66[24] = BYTE6(v35);`    (In6)
- `v63 = HIBYTE(v35);`       (In7)
- `v69 = HIBYTE(v35);`       (In7 again)

At first glance, it looks like random assignments… but it’s actually writing raw bytes into the bytecode stream.

Then it calls:

`ret = vm_interpreter(HIBYTE(v35), v41);`

So: **the validation is entirely inside VM**.

---

## VM Interpreter: `sub_140001EC0`

This function is the real “algorithm”.
![[{0A8F964C-A232-401B-9A98-6555A49B440B}.png]]
### VM State 

The VM has only 3 things:

1) **ACC (Accumulator)**
- stored in `result` (AL, 8-bit)

2) **REGS[16] (16 registers R0..R15)**
- stored in a local `__int128 v20`
- accessed like: `*((_BYTE *)&v20 + idx)`
![[{5D527D60-38B7-44A0-976B-7ECB0F2686A7} 1.png]]

3) **PC (program counter)**
- not explicit; it walks the bytecode buffer

Everything happens in 8-bit arithmetic → wrap modulo 256.

---

## Instruction Encoding 

Each VM instruction is **3 bytes**:

`[OPCODE] [OPERAND] [MODE]`

- If `MODE == 0x41 ('A')` → operand is a register reference:
  - `val = REGS[OPERAND & 0xF]`

- Else → operand is immediate:
  - `val = OPERAND`

This explains all the `& 0xF` you see in pseudo.

### Opcode set (ISA)
![[{0B883929-CD89-4238-8E34-66219CA4CC32}.png]]
From the interpreter:

- `'I'` : `ACC = val` (LOAD)
- `'O'` : `REGS[operand & 0xF] = ACC` (STORE)
- `'A'` : `ACC = (ACC + val) & 0xFF`
- `'S'` : `ACC = (ACC - val) & 0xFF`
- `'X'` : `ACC = (ACC ^ val) & 0xFF`

---

## Digging Interpreter 
![[{1935F8A2-55B7-4367-A484-DCBB3B81D131}.png]]

In the decompiled version, the VM handles 4 instructions per iteration:

- instruction #0 at `a2 + v5`
- instruction #1 at `a2 + 3*v6`
- instruction #2 at `a2 + 3*v7`
- instruction #3 at `a2 + 3*v8`

Then it updates:

- `v5 += 12`
- `v6 += 4`, `v7 += 4`, `v8 += 4`
- `v3 += 4` until `v3 == 80`

This is just **unrolling**:

- 1 instruction = 3 bytes
- 4 instructions = 12 bytes
- 80 instructions total → 20 iterations

So the bytecode buffer length is: `80 * 3 = 240 bytes`.

---

## Reconstructing the verification algorithm 

```C
char __fastcall sub_140001EC0(__int64 a1, __int64 a2)
{
  char result; // al
  __int64 v3; // rbx
  __int64 v5; // rdi
  __int64 v6; // r8
  __int64 v7; // r9
  __int64 v8; // r10
  char v9; // cl
  char v10; // dl
  char *v11; // rdx
  char v12; // cl
  char v13; // dl
  char *v14; // rdx
  char v15; // cl
  char v16; // dl
  char *v17; // rdx
  char v18; // cl
  char v19; // dl
  __int128 v20; // [rsp+0h] [rbp-18h]

  result = 0;
  v3 = 0;
  v20 = 0;
  v5 = 0;
  v6 = 1;
  v7 = 2;
  v8 = 3;
  do
  {
    v9 = *(_BYTE *)(a2 + v5);
    switch ( v9 )
    {
      case 'A':
        if ( *(_BYTE *)(a2 + v5 + 2) == 65 )
          result += *((_BYTE *)&v20 + (*(_BYTE *)(a2 + v5 + 1) & 0xF));
        else
          result += *(_BYTE *)(a2 + v5 + 1);
        break;
      case 'I':
        if ( *(_BYTE *)(a2 + v5 + 2) == 65 )
          result = *((_BYTE *)&v20 + (*(_BYTE *)(a2 + v5 + 1) & 0xF));
        else
          result = *(_BYTE *)(a2 + v5 + 1);
        break;
      case 'O':
        *((_BYTE *)&v20 + (*(_BYTE *)(a2 + v5 + 1) & 0xF)) = result;
        break;
      case 'S':
        if ( *(_BYTE *)(a2 + v5 + 2) == 65 )
          result -= *((_BYTE *)&v20 + (*(_BYTE *)(a2 + v5 + 1) & 0xF));
        else
          result -= *(_BYTE *)(a2 + v5 + 1);
        break;
      case 'X':
        if ( *(_BYTE *)(a2 + v5 + 2) == 65 )
          v10 = *((_BYTE *)&v20 + (*(_BYTE *)(a2 + v5 + 1) & 0xF));
        else
          v10 = *(_BYTE *)(a2 + v5 + 1);
        result ^= v10;
        break;
    }
    v11 = (char *)(v6 + a2 + 2 * v6);
    v12 = *v11;
    if ( *v11 == 65 )
    {
      if ( v11[2] == 65 )
        result += *((_BYTE *)&v20 + (v11[1] & 0xF));
      else
        result += v11[1];
    }
    else
    {
      switch ( v12 )
      {
        case 'I':
          if ( v11[2] == 65 )
            result = *((_BYTE *)&v20 + (v11[1] & 0xF));
          else
            result = v11[1];
          break;
        case 'O':
          *((_BYTE *)&v20 + (v11[1] & 0xF)) = result;
          break;
        case 'S':
          if ( v11[2] == 65 )
            result -= *((_BYTE *)&v20 + (v11[1] & 0xF));
          else
            result -= v11[1];
          break;
        case 'X':
          if ( v11[2] == 65 )
            v13 = *((_BYTE *)&v20 + (v11[1] & 0xF));
          else
            v13 = v11[1];
          result ^= v13;
          break;
      }
    }
    v14 = (char *)(v7 + a2 + 2 * v7);
    v15 = *v14;
    if ( *v14 == 65 )
    {
      if ( v14[2] == 65 )
        result += *((_BYTE *)&v20 + (v14[1] & 0xF));
      else
        result += v14[1];
    }
    else
    {
      switch ( v15 )
      {
        case 'I':
          if ( v14[2] == 65 )
            result = *((_BYTE *)&v20 + (v14[1] & 0xF));
          else
            result = v14[1];
          break;
        case 'O':
          *((_BYTE *)&v20 + (v14[1] & 0xF)) = result;
          break;
        case 'S':
          if ( v14[2] == 65 )
            result -= *((_BYTE *)&v20 + (v14[1] & 0xF));
          else
            result -= v14[1];
          break;
        case 'X':
          if ( v14[2] == 65 )
            v16 = *((_BYTE *)&v20 + (v14[1] & 0xF));
          else
            v16 = v14[1];
          result ^= v16;
          break;
      }
    }
    v17 = (char *)(v8 + a2 + 2 * v8);
    v18 = *v17;
    if ( *v17 == 65 )
    {
      if ( v17[2] == 65 )
        result += *((_BYTE *)&v20 + (v17[1] & 0xF));
      else
        result += v17[1];
    }
    else
    {
      switch ( v18 )
      {
        case 'I':
          if ( v17[2] == 65 )
            result = *((_BYTE *)&v20 + (v17[1] & 0xF));
          else
            result = v17[1];
          break;
        case 'O':
          *((_BYTE *)&v20 + (v17[1] & 0xF)) = result;
          break;
        case 'S':
          if ( v17[2] == 65 )
            result -= *((_BYTE *)&v20 + (v17[1] & 0xF));
          else
            result -= v17[1];
          break;
        case 'X':
          if ( v17[2] == 65 )
            v19 = *((_BYTE *)&v20 + (v17[1] & 0xF));
          else
            v19 = v17[1];
          result ^= v19;
          break;
      }
    }
    v3 += 4;
    v5 += 12;
    v6 += 4;
    v7 += 4;
    v8 += 4;
  }
  while ( v3 != 80 );
  return result;
}
```

Blocks are separated by the pattern:

- compute something in ACC
- `STORE R10`
- next block continues from R10

Notation:
- `InN` = byte ASCII of key at index N (0..9)
- arithmetic is modulo 256

### Block 1 — seed R10 from In0/In1

This block normalizes 2 chars by subtracting constants:

- `(In0 - 'K')`
- `(In1 - 'E')`

So:

R10 = (In0 - 0x4B) + (In1 - 0x45)

### Block 2 — add In2/In3 into R10

Same pattern:

- `(In2 - 'Y')`
- `(In3 - '-')`

So:

R10 += (In2 - 0x59) + (In3 - 0x2D)

### Block 3 — “weird block” 

This is the only block that mixes multiple input bytes:

R10 += (In4 + In7 - In5 - 0x4B)

This is intentional obfuscation:
- It injects `In7` to make you think char #8 is critical.

### Block 4 — cancel block 

Final block does:

ACC = (In6 - In7) + R10

So the `+In7` from Block 3 is canceled by `-In7` here.  
That’s the trick: **In7 cancels out completely**.

---

## Final Equation 

Combine blocks:

(In0 - 0x4B) + (In1 - 0x45) + (In2 - 0x59) + (In3 - 0x2D)
+ (In4 + In7 - In5 - 0x4B) + (In6 - In7) = 0

Cancel `In7`:

In0 + In1 + In2 + In3 + In4 - In5 + In6
- (0x4B + 0x45 + 0x59 + 0x2D + 0x4B) = 0

Constants sum:

- 0x4B + 0x45 + 0x59 + 0x2D + 0x4B = 353
- 353 mod 256 = 97 = 0x61

### Valid key condition

In0 + In1 + In2 + In3 + In4 - In5 + In6 ≡ 0x61 (mod 256)

And:
- `In7` cancels
- `In8, In9` never participate (filler)
---

## Overall (Keygening)

So to generate a valid 10-char key:

1) pick any printable bytes for `In0..In4` and `In6`

2) compute `In5`:

In5 ≡ (In0 + In1 + In2 + In3 + In4 + In6 − 0x61) (mod 256)

3) choose any chars for `In7 In8 In9` (they don’t matter for the check)

One-liner check:

License valid iff:
(In0 + In1 + In2 + In3 + In4 - In5 + In6) mod 256 = 0x61

---
## Keygen 

```python
import random, string

ALPH = string.ascii_letters + string.digits + "-_"
TARGET = 0x61

def gen_key():
    while True:
        In0 = ord(random.choice(ALPH))
        In1 = ord(random.choice(ALPH))
        In2 = ord(random.choice(ALPH))
        In3 = ord(random.choice(ALPH))
        In4 = ord(random.choice(ALPH))
        In6 = ord(random.choice(ALPH))

        In5 = (In0 + In1 + In2 + In3 + In4 + In6 - TARGET) & 0xFF

        if 32 <= In5 < 127 and chr(In5) in ALPH:
            In7 = ord(random.choice(ALPH))  # irrelevant (cancels)
            In8 = ord(random.choice(ALPH))  # filler
            In9 = ord(random.choice(ALPH))  # filler
            return bytes([In0,In1,In2,In3,In4,In5,In6,In7,In8,In9]).decode()

for _ in range(5):
    print(gen_key())
