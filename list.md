This is a famous utility written by **Jon North**, a legendary figure in the 1980s ZX Spectrum scene (known for his "hacks" in *Your Sinclair* magazine).

This specific program is a **Protected BASIC Lister**. Its purpose is to list BASIC code that has been "protected" to prevent the standard `LIST` command from working (like the one you showed me in your previous message with the 65,532-byte line length).

Here is the technical disassembly of the machine code stored in those `DATA` statements.

### Technical Disassembly

**Origin Address:** 30085 ($7585)

| Address | Hex | Instruction | Comments |
| --- | --- | --- | --- |
| **$7585** | `3E 02` | `LD A, 2` | Set channel to 2 (Upper screen) |
| **$7587** | `CD 01 16` | `CALL $1601` | **ROM:** Open the channel |
| **$758A** | `2A 53 5C` | `LD HL, ($5C53)` | Get `VARS` address (End of BASIC) |
| **$758D** | `E5` | `PUSH HL` | Save end address on stack |
| **$758E** | `ED 5B 4B 5C` | `LD DE, ($5C4B)` | Get `PROG` address (Start of BASIC) |
| **$7592** | `37` | `SCF` | Set Carry |
| **$7593** | `3F` | `CCF` | Clear Carry (Ready for subtraction) |
| **$7594** | `ED 52` | `SBC HL, DE` | Check if we've reached the end |
| **$7596** | `7C` | `LD A, H` |  |
| **$7597** | `B5` | `OR L` |  |
| **$7598** | `E1` | `POP HL` | Restore pointer |
| **$7599** | `C8` | `RET Z` | Exit if program end reached |
| **$759A** | `46` | `LD B, (HL)` | Get Line Number (High Byte) |
| **$759B** | `23` | `INC HL` |  |
| **$759C** | `4E` | `LD C, (HL)` | Get Line Number (Low Byte) |
| **$759D** | `23` | `INC HL` |  |
| **$759E** | `E5` | `PUSH HL` | Save current pointer |
| **$759F** | `CD 2B 2D` | `CALL $2D2B` | **ROM:** Stack BC as floating point |
| **$75A2** | `CD E3 2D` | `CALL $2DE3` | **ROM:** Print the line number |
| **$75A5** | `E1` | `POP HL` | Restore pointer |
| **$75A6** | `4E` | `LD C, (HL)` | Get Line Length (Low) |
| **$75A7** | `23` | `INC HL` |  |
| **$75A8** | `46` | `LD B, (HL)` | Get Line Length (High) |
| **$75A9** | `23` | `INC HL` |  |
| **$75AA** | `E5` | `PUSH HL` | Save start of line text |
| **$75AB** | `09` | `ADD HL, BC` | Find the *real* end of this line |
| **$75AC** | `22 FE FF` | `LD ($FFFE), HL` | Store end of line for loop check |
| **$75AF** | `E1` | `POP HL` | Back to start of line text |

---

### Logic Summary

The code effectively bypasses the Spectrum's built-in listing logic, which is easily fooled by spoofed line lengths or hidden characters.

1. **Direct Memory Access:** It looks directly at the system variables (`VARS` and `PROG`) to find where the BASIC actually resides in RAM.
2. **Explicit Printing:** Instead of letting the ROM handle the listing, it manually extracts the line numbers, pushes them onto the calculator stack (`$2D2B`), and forces the ROM to print them as integers (`$2DE3`). This makes "Line 0" or "Line 32768" visible.
3. **Hex/Token Handling:** The remaining logic (lines 160-310) iterates through every single byte in the line.
* If it finds a carriage return (`$0D`), it starts the next line.
* If it finds a token (bytes above 128), it uses the ROM's character printing routine to display the keyword.
* It ignores the "invisible" color control codes that hackers used to hide code (by making the text white on a white background).



### How to use it

To use this on your Spectrum (or emulator):

1. Type in the BASIC program as shown in the images.
2. **RUN** it. It will POKE the machine code into memory and verify the checksum (Line 60).
3. Load your "protected" program (using `LOAD "" CODE` or similar to prevent it from auto-running and wiping the lister).
4. Activate the lister with `RANDOMIZE USR 30085`.

It will give you a clean, readable listing of even the most heavily obfuscated loaders.