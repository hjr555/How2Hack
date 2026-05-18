# Protected BASIC Lister (*LIST)

This is a famous utility written by **Jon North**, from the *How 2 Hack* column (*Your Sinclair* issue 56, August 1990).

This program is a **Protected BASIC Lister**. It lists BASIC that has been disguised so the normal `LIST` command fails—spoofed line lengths, hidden tokens, invisible ink, and similar tricks. Companion tool: the custom tape loader in [load.md](load.md).

Here is the technical disassembly of the machine code in the `DATA` statements (see [YS56.md](YS56.md)).

## Technical Disassembly

**Origin address:** 30085 ($7585)  
**Length:** 116 bytes (30085–30200)  
**Checksum:** 919527 (BASIC lines 20–60)

| Address | Hex | Instruction | Comments |
| --- | --- | --- | --- |
| **$7585** | `3E 02` | `LD A,2` | Channel 2 (upper screen) |
| **$7587** | `CD 01 16` | `CALL $1601` | **ROM:** open channel |
| **$758A** | `2A 53 5C` | `LD HL,($5C53)` | **HL** ← `VARS` (end of BASIC) |
| **$758D** | `E5` | `PUSH HL` | Save end address on stack |
| **$758E** | `ED 5B 4B 5C` | `LD DE,($5C4B)` | **DE** ← `PROG` (start of BASIC) |
| **$7592** | `37` | `SCF` | Set carry |
| **$7593** | `3F` | `CCF` | Clear carry (ready for subtraction) |
| **$7594** | `ED 52` | `SBC HL,DE` | **HL** ← bytes remaining in program |
| **$7596** | `7C` | `LD A,H` | High byte of length |
| **$7597** | `B5` | `OR L` | Test if length is zero |
| **$7598** | `E1` | `POP HL` | Restore line pointer |
| **$7599** | `C8` | `RET Z` | Exit when entire program listed |
| **$759A** | `46` | `LD B,(HL)` | Line number, high byte |
| **$759B** | `23` | `INC HL` | Next byte |
| **$759C** | `4E` | `LD C,(HL)` | Line number, low byte |
| **$759D** | `23` | `INC HL` | Next byte |
| **$759E** | `E5` | `PUSH HL` | Save pointer into line |
| **$759F** | `CD 2B 2D` | `CALL $2D2B` | **ROM:** stack **BC** as floating-point |
| **$75A2** | `CD E3 2D` | `CALL $2DE3` | **ROM:** print line number |
| **$75A5** | `E1` | `POP HL` | Restore pointer |
| **$75A6** | `4E` | `LD C,(HL)` | Line length, low byte (as stored) |
| **$75A7** | `23` | `INC HL` | Next byte |
| **$75A8** | `46` | `LD B,(HL)` | Line length, high byte (as stored) |
| **$75A9** | `23` | `INC HL` | Next byte |
| **$75AA** | `E5` | `PUSH HL` | Save start of line text |
| **$75AB** | `09` | `ADD HL,BC` | **HL** ← true end of line (uses real length) |
| **$75AC** | `22 FE FF` | `LD ($FFFE),HL` | Store end-of-line marker for inner loop |
| **$75AF** | `E1` | `POP HL` | Back to start of line text |

The table above covers the outer loop (find each line, print its number). Bytes from **$75B0** onward walk every byte in the line—tokens via **RST $10**, carriage returns, and skipping invisible colour codes. See the full `DATA` block in [YS56.md](YS56.md) (lines 80–310).

---

## Logic Summary

The code bypasses the Spectrum’s built-in listing logic, which is easily fooled by spoofed line lengths or hidden characters.

1. **Direct memory access:** Reads `VARS` and `PROG` to find where BASIC actually lives in RAM.
2. **Explicit printing:** Extracts line numbers, pushes them on the calculator stack (`$2D2B`), and prints them as integers (`$2DE3`) so “line 0” or “line 32768” is visible.
3. **Line-body walk:** The remainder of the routine scans each byte in the line—printing tokens, honouring carriage returns, and ignoring colour codes used to hide text (for example white on white).

---

## How to use it

On a Spectrum or emulator:

1. Type in the BASIC program from [YS56.md](YS56.md) (the `*List` listing, lines 10–70 and DATA 80–310).
2. **RUN** it. The routine is POKEd to **30085–30200** and the checksum (**919527**) is verified.
3. Load your protected program (`LOAD "" CODE` or similar, so it does not auto-run and overwrite the lister).
4. Activate the lister with **`RANDOMIZE USR 30085`** instead of **`LIST`**.

You get a readable listing of even heavily obfuscated loaders. For tapes, load the program first with [load.md](load.md) (**`RANDOMIZE USR 30000`**) if you need to inspect the header before the loader runs.

---

## Raw DATA bytes (decimal)

For tools or re-disassembly:

`62,2,205,1,22,42,83,92,229,237,91,75,92,55,63,237,82,124,181,225,200,70,35,78,35,229,205,43,45,205,227,45,225,78,35,70,35,229,9,34,254,255,225,126,254,13,32,4,35,215,24,212,254,46,40,8,254,58,48,19,254,48,56,15,68,62,14,237,177,205,180,51,229,205,227,45,225,24,220,254,32,56,2,215,126,254,234,32,8,62,13,215,42,254,255,24,167,254,34,32,12,35,126,254,32,56,2,215,126,254,34,32,244,35,24,183`
