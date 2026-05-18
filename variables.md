
---
# Chapter 25: The system variables

The bytes in memory from 23552 to 23733 are set aside for specific uses by the system. You can `PEEK` them to find out various things about the system, and some of them can be usefully `POKE`d. They are listed here with their uses.

These are called system variables and have names, but do not confuse them with the variables used by BASIC. The computer will not recognize the names as referring to system variables; they are given solely as mnemonics for humans.

### Notes on the Table

The abbreviations in the first column have the following meanings:

* **X**: The variable should not be `POKE`d because the system might crash.
* **N**: Poking the variable will have no lasting effect.

The number in the first column is the number of bytes in the variable. For two-byte variables, the first byte is the **least significant** (the reverse of what you might expect).

* To `POKE` a value `v` to a two-byte variable at address `n`:
`POKE n, v - 256 * INT (v / 256) : POKE n + 1, INT (v / 256)`
* To `PEEK` its value:
`PEEK n + 256 * PEEK (n + 1)`

---

| Notes | Address | Name | Use |
| --- | --- | --- | --- |
| N8 | 23552 | **KSTATE** | Used in reading the keyboard. |
| N1 | 23560 | **LAST K** | Stores newly pressed key. |
| 1 | 23561 | **REPDEL** | Time (in 50ths of a second) that a key must be held down before it repeats. (Initial: 35). |
| 1 | 23562 | **REPPER** | Delay (in 50ths of a second) between successive repeats of a key held down. (Initial: 5). |
| N2 | 23563 | **DEFADD** | Address of arguments of user-defined function if one is being evaluated; otherwise 0. |
| N1 | 23565 | **K DATA** | Stores 2nd byte of colour controls entered from keyboard. |
| N2 | 23566 | **TVDATA** | Stores bytes of colour, AT and TAB controls going to television. |
| X38 | 23568 | **STRMS** | Addresses of channels attached to streams. |
| 2 | 23606 | **CHARS** | 256 less than address of character set (starts at space). |
| 1 | 23608 | **RASP** | Length of warning buzz. |
| 1 | 23609 | **PIP** | Length of keyboard click. |
| 1 | 23610 | **ERR NR** | 1 less than the report code. (Starts at 255). |
| X1 | 23611 | **FLAGS** | Various flags to control the BASIC system. |
| X1 | 23612 | **TV FLAG** | Flags associated with the television. |
| X2 | 23613 | **ERR SP** | Address of item on machine stack to use as error return. |
| N2 | 23615 | **LIST SP** | Return address from automatic listing. |
| 1 | 23617 | **MODE** | Specifies which cursor to display (K, L, C, E or G). |
| 2 | 23618 | **NEWPPC** | Line number to be jumped to. |
| 1 | 23620 | **NSPPC** | Statement number in line to be jumped to. |
| 2 | 23621 | **PPC** | Line number of statement currently being executed. |
| 1 | 23623 | **SUBPPC** | Number within line of statement being executed. |
| 1 | 23624 | **BORDCR** | Border colour (multiplied by 8); also contains foreground colour for lower screen. |
| 2 | 23625 | **E PPC** | Number of current line (in command line). |
| X2 | 23627 | **VARS** | Address of variables. |
| X2 | 23629 | **DEST** | Address of variable in assignment. |
| X2 | 23631 | **CHANS** | Address of channel data. |
| X2 | 23633 | **CURCHL** | Address of information currently being used for input/output. |
| X2 | 23635 | **PROG** | Address of BASIC program. |
| X2 | 23637 | **NXTLIN** | Address of next line in program. |
| X2 | 23639 | **DATADD** | Address of terminator of last DATA item. |
| X2 | 23641 | **E LINE** | Address of command being typed in. |
| 2 | 23643 | **K CUR** | Address of cursor. |
| X2 | 23645 | **CH ADD** | Address of next character to be interpreted. |
| X2 | 23647 | **X PTR** | Address of the character after the `?` marker. |
| X2 | 23649 | **WORKSP** | Address of temporary work space. |
| X2 | 23651 | **STKBOT** | Address of bottom of calculator stack. |
| X2 | 23653 | **STKEND** | Address of start of spare space. |
| N1 | 23655 | **BREG** | Calculator's B register. |
| 2 | 23656 | **MEM** | Address of area used for calculator's memory (usually MEM0). |
| 1 | 23658 | **FLAGS2** | More flags. |
| 1 | 23659 | **DF SZ** | Number of lines in the lower part of the screen. |
| 2 | 23660 | **S TOP** | Number of top program line in automatic listings. |
| 2 | 23662 | **OLDPPC** | Line number to which `CONTINUE` jumps. |
| 1 | 23664 | **OSPPC** | Number within line of statement to which `CONTINUE` jumps. |
| 1 | 23665 | **FLAGX** | Various flags. |
| 2 | 23666 | **STRLEN** | Length of string type destination in assignment. |
| X2 | 23668 | **T ADDR** | Address of next item in syntax table. |
| 2 | 23670 | **SEED** | The seed for `RND`. Set by `RANDOMIZE`. |
| 3 | 23672 | **FRAMES** | 3-byte frame counter. Incremented every 20ms (in UK). |
| 2 | 23675 | **UDG** | Address of first user-defined graphic (default starts at `BIN`). |
| 2 | 23677 | **COORDS** | x and y coordinates of last point plotted. |
| 1 | 23679 | **P POSN** | Column number of printer position. |
| 2 | 23680 | **PR CC** | Address of next position for `LPRINT` to print at. |
| 2 | 23682 | **ECHO E** | Column and line number of end of input buffer. |
| 2 | 23684 | **DF CC** | Address in display file of `PRINT` position. |
| 2 | 23686 | **DF CCL** | Like DF CC for lower part of screen. |
| 2 | 23688 | **S POSN** | Column and line number for `PRINT` position. |
| 2 | 23690 | **S POSNL** | Like S POSN for lower part of screen. |
| 1 | 23692 | **SCR CT** | Scroll counter. |
| 1 | 23693 | **ATTR P** | Permanent current colours (set by colour statements). |
| 1 | 23694 | **MASK P** | Permanent current mask (used for transparent colours). |
| 1 | 23695 | **ATTR T** | Temporary current colours. |
| 1 | 23696 | **MASK T** | Temporary current mask. |
| 1 | 23697 | **P FLAG** | More flags. |
| 30 | 23698 | **MEM0** | Calculator's memory area (used for `MEM`). |
| 2 | 23728 | **NMIADD** | Address of non-maskable interrupt routine. |
| 2 | 23730 | **RAMTOP** | Address of last byte of BASIC system area. |
| 2 | 23732 | **P RAMT** | Address of last byte of physical RAM. |

---

This program tells you the first 22 bytes of the variables area:

```basic
10 FOR n=0 TO 21
20 PRINT PEEK (PEEK 23627 + 256 * PEEK 23628 + n)
30 NEXT n

```

Try to match up the control variable `n` with the descriptions above.


	20 PRINT PEEK (23755+n)

This tells you the first 22 bytes of the program area. Match these up with the program itself. 