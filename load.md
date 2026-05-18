# Custom LOAD Routine (*LOAD)

This is a famous utility written by **Jon North**, from the *How 2 Hack* column (*Your Sinclair* issue 56, August 1990).

This program is a **Custom LOAD Routine** (`*LOAD`). It replaces the normal `LOAD ""` step when you want to inspect a tape block before a protected loader takes over: load the **17-byte header** under machine-code control, print filename, autostart line, and block length, then hand off to the ROM so the **program body** loads and you stop at **OK**. Companion tool: the protected BASIC lister in [list.md](list.md).

Here is the technical disassembly of the machine code in the `DATA` statements (see [YS56.md](YS56.md); cross-checked against [Sources/YS56-1.png](Sources/YS56-1.png)).

## Technical Disassembly

**Origin address:** 30000 ($7530)  
**Length:** 84 bytes (30000–30083)  
**Checksum:** 544506 (BASIC lines 20–60)

After `CALL $0556` loads the header, **IX points just past the 17-byte block** ($5011). The display section uses **negative displacements** (`IX-3`, `IX-4`, …) to reach fields in the header at $5000–$5010. The same byte pattern Jon North uses for headerless loads (`DD 21`, `11`, `AF`, `37`, `CD 56 05`, `30 F0`) appears here—see [YS56.md](YS56.md#what-a-headerless-loader-looks-like).

| Address | Hex | Instruction | Comments |
| --- | --- | --- | --- |
| **$7530** | `DD 21 00 50` | `LD IX,$5000` | Destination for the tape **header** block |
| **$7534** | `11 11 00` | `LD DE,$0011` | **DE = 17** — standard header length |
| **$7537** | `AF` | `XOR A` | **A = 0** — header block (not raw data); clears carry |
| **$7538** | `37` | `SCF` | Carry set — required by `LD_BYTES` for a header |
| **$7539** | `CD 56 05` | `CALL $0556` | **ROM:** `LD_BYTES` — read block from tape into `(IX)`, length **DE** |
| **$753C** | `30 F0` | `JR NC,$7530` | Retry from `LD IX,$5000` if load failed |
| **$753E** | `DD 7E EF` | `LD A,(IX-17)` | With **IX = $5011**, reads **($5000)** — header **type** (0 = program) |
| **$7541** | `B7` | `OR A` | Zero means program header |
| **$7542** | `20 EC` | `JR NZ,$7530` | Reload header if not a program file |
| **$7544** | `3E 02` | `LD A,2` | Channel 2 (upper screen) |
| **$7545** | `CD 01 16` | `CALL $1601` | **ROM:** open channel |
| **$7548** | `21 01 50` | `LD HL,$5001` | **HL** → filename field (bytes 1–10) |
| **$754B** | `06 0A` | `LD B,10` | Ten characters |
| **$754C** | `7E` | `LD A,(HL)` | Next filename character |
| **$754D** | `D7` | `RST $10` | **ROM:** print character in **A** (**215** = `RST $10` in BASIC `DATA`) |
| **$754E** | `23` | `INC HL` | Next character |
| **$754F** | `10 FB` | `DJNZ $754C` | Loop until all 10 bytes printed |
| **$7551** | `3E CA` | `LD A,$CA` | Separator after filename |
| **$7552** | `D7` | `RST $10` | Print it |
| **$7553** | `DD 46 FD` | `LD B,(IX-3)` | Autostart line, high byte ($500E) |
| **$7556** | `DD 4E FC` | `LD C,(IX-4)` | Autostart line, low byte ($500D) |
| **$7559** | `CD 2B 2D` | `CALL $2D2B` | **ROM:** stack **BC** as floating-point |
| **$755C** | `CD E3 2D` | `CALL $2DE3` | **ROM:** print line number (0, 1, 10, …) |
| **$755F** | `DD 36 FD FF` | `LD (IX-3),$FF` | Poke high line byte before length is shown |
| **$7563** | `3E 20` | `LD A,$20` | Space |
| **$7564** | `D7` | `RST $10` | Print space |
| **$7565** | `3E B1` | `LD A,$B1` | Separator (`±` on the Spectrum) |
| **$7566** | `D7` | `RST $10` | Print it |
| **$7567** | `DD 46 FB` | `LD B,(IX-5)` | Data-block length, high byte ($500C) |
| **$756A** | `DD 4E FA` | `LD C,(IX-6)` | Data-block length, low byte ($500B) |
| **$756D** | `CD 2B 2D` | `CALL $2D2B` | **ROM:** stack **BC** as floating-point |
| **$7570** | `CD E3 2D` | `CALL $2DE3` | **ROM:** print block length |
| **$7573** | `3E 0D` | `LD A,$0D` | Carriage return |
| **$7574** | `D7` | `RST $10` | End display line |
| **$7575** | `2A 53 5C` | `LD HL,($5C53)` | **HL** ← `VARS` — for ROM **LOAD** destination sizing |
| **$7578** | `DD 46 00` | `LD B,(IX+0)` | Byte at **IX** before ROM hand-off |
| **$757B** | `C3 73 08` | `JP $0873` | **ROM:** `LD_PROG` — load program body, return to **OK** ([LOAD @ $0808](https://speccy.xyz/rom/asm/0808)) |

### Tape header layout

When the header sits at **$5000** and **IX = $5011** after `LD_BYTES`:

| Header offset | Address | Field |
| --- | --- | --- |
| 0 | $5000 | Type (0 = program) — tested via `(IX-17)` |
| 1–10 | $5001–$500A | Filename — printed by **HL** loop |
| 11–12 | $500B–$500C | Data-block length — via `(IX-6)`, `(IX-5)` |
| 13–14 | $500D–$500E | Autostart line — via `(IX-4)`, `(IX-3)` |
| 15–16 | $500F–$5010 | Program length (param 2) |

Here **A = 0** comes from **`XOR A`** + **`SCF`**, not **`LD A,$FF`** (used when loading a **data** block without a separate header).

---

## Logic Summary

1. **Load header only:** Calls **`LD_BYTES`** until a valid program header (type 0) is at **$5000**.
2. **Show what is loading:** Opens channel 2 and prints **filename**, **line**, and **length** from the header.
3. **ROM takes over:** **`JP $0873`** enters **`LD_PROG`**, which loads the program body, updates workspace, and returns to BASIC with **OK** (as in *YS56*).

This is not a fully **headerless** load (raw code with **IX**, **DE**, and **A = $FF**). It still uses a normal tape header; it only replaces the BASIC **`LOAD ""`** front end so you see header fields before the main block loads.

---

## How to use it

On a Spectrum or emulator:

1. Type in the BASIC program from [YS56.md](YS56.md) (the `*Load` listing, lines 10–70 and DATA 80–240).
2. **RUN** it. The routine is POKEd to **30000–30083** and the checksum (**544506**) is verified.
3. Start the tape and run **`RANDOMIZE USR 30000`** instead of **`LOAD ""`**.
4. Read the printed **filename**, **line**, and **length**; let the ROM finish loading until **OK**.

For disguised BASIC after loading, use [list.md](list.md) (**`RANDOMIZE USR 30085`**) instead of **`LIST`**.

---

## Raw DATA bytes (decimal)

For tools or re-disassembly:

`221,33,0,80,17,17,0,175,55,205,86,5,48,240,221,126,239,183,32,236,62,2,205,1,22,33,1,80,6,10,126,215,35,16,251,62,202,215,221,70,253,221,78,252,205,43,45,205,227,45,221,54,253,255,62,32,215,62,177,215,221,70,251,221,78,250,205,43,45,205,227,45,62,13,215,42,83,92,221,46,0,195,115,8`
