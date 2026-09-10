**HACKING THE WORKFORCE --- STUDY REFERENCE**

**Binary, Hex & Subnetting Quick Reference**

1\. Decimal ↔ Binary

The place-value row (memorize this --- it never changes)

**128 64 32 16 8 4 2 1**

Decimal → Binary method

Go left to right through the place values. At each one, ask: \"Does this
value fit into what I have left?\"

• If YES → write 1, and subtract that value from your remaining number.

• If NO → write 0, and move to the next column with the same remaining
number.

+---------------------------------------------------------------+
| **Worked example: 168**                                       |
|                                                               |
| 128 fits (168 ≥ 128)? YES → write 1, remainder = 168-128 = 40 |
|                                                               |
| 64 fits (40 ≥ 64)? NO → write 0                               |
|                                                               |
| 32 fits (40 ≥ 32)? YES → write 1, remainder = 40-32 = 8       |
|                                                               |
| 16 fits (8 ≥ 16)? NO → write 0                                |
|                                                               |
| 8 fits (8 ≥ 8)? YES → write 1, remainder = 0                  |
|                                                               |
| 4, 2, 1 fit (0 ≥ each)? NO, NO, NO → write 0, 0, 0            |
|                                                               |
| Result: 168 = 10101000 (check: 128+32+8 = 168 ✓)              |
+---------------------------------------------------------------+

Binary → Decimal method

Same idea in reverse: line the bits up under the place-value row. Add up
the columns where the bit is 1; ignore columns where it\'s 0.

1 0 1 0 1 1 0 0

128 64 32 16 8 4 2 1

*128 + 32 + 8 + 4 = 172 → 10101100 = 172*

2\. Hex ↔ Binary (used for MAC addresses)

Each hex digit always converts to exactly 4 bits (a \"nibble\") --- no
subtraction method needed, just a direct lookup.

  --------- --------- --------- --------- --------- --------- --------- ---------
  **Hex**   **Bin**   **Hex**   **Bin**   **Hex**   **Bin**   **Hex**   **Bin**
  0         0000      4         0100      8         1000      C         1100
  1         0001      5         0101      9         1001      D         1101
  2         0010      6         0110      A         1010      E         1110
  3         0011      7         0111      B         1011      F         1111
  --------- --------- --------- --------- --------- --------- --------- ---------

+----------------------------------------------------------------------+
| **Worked example: hex \"08\" → binary**                              |
|                                                                      |
| 0 → 0000                                                             |
|                                                                      |
| 8 → 1000                                                             |
|                                                                      |
| Result: 08 = 0000 1000                                               |
|                                                                      |
| (To go back: group the binary into 4-bit chunks and look each chunk  |
| up again.)                                                           |
+----------------------------------------------------------------------+

*Why hex for MACs, decimal for IPv4? A 48-bit MAC splits perfectly into
12 hex digits (rarely needs math). IPv4 needs constant subnetting
arithmetic, which humans do more easily in decimal.*

3\. Subnetting --- The \"Section Size\" Method

A subnet mask draws a line between the network portion (shared by
everyone on the subnet) and the host portion (unique per device). The
/number tells you how many bits, counting from the left, are network
bits.

Think of the last relevant octet as a parking lot with 256 spaces
(0-255), chopped into equal-sized sections. The mask tells you how big
each section is.

Section size lookup table

  ---------- ---------------- ------------------ --------------------
  **CIDR**   **Mask octet**   **Section size**   **\# of sections**
  /24        255 (.0)         256                1
  /25        128              128                2
  /26        192              64                 4
  /27        224              32                 8
  /28        240              16                 16
  /29        248              8                  32
  /30        252              4                  64
  ---------- ---------------- ------------------ --------------------

*Pattern: each step down the CIDR list cuts the section size exactly in
half.*

The 4-step method

1\. Look up the section size for your CIDR in the table above.

2\. Starting at 0, count up by the section size to list all the section
boundaries (stop before 256).

3\. Find which section your IP\'s relevant octet falls into.

4\. Network address = section start · Broadcast address = section end ·
Usable range = everything strictly between.

+----------------------------------------------------------------------+
| **Why subtract 1 to get the section end (broadcast address)?**       |
|                                                                      |
| When you count up by the section size (0, 32, 64, 96\...), each      |
| number is where the NEXT section starts.                             |
|                                                                      |
| A section can\'t include the number where the next section begins    |
| --- that number already belongs to the next section.                 |
|                                                                      |
| So the current section\'s last valid number is one less than the     |
| next section\'s start: e.g. if the next section starts at 64, this   |
| section ends at 63 (64 − 1 = 63).                                    |
|                                                                      |
| That\'s the broadcast address --- the highest number that still      |
| belongs to this section, right before the boundary flips to the next |
| one.                                                                 |
+----------------------------------------------------------------------+

+----------------------------------------------------------------------+
| **Worked example: 192.168.1.200 /27**                                |
|                                                                      |
| Step 1: /27 → section size = 32                                      |
|                                                                      |
| Step 2: sections are 0-31, 32-63, 64-95, 96-127, 128-159, 160-191,   |
| 192-223, 224-255                                                     |
|                                                                      |
| Step 3: 200 falls in the 192-223 section                             |
|                                                                      |
| Step 4: Network = 192.168.1.192 \| Broadcast = 192.168.1.223 \|      |
| Usable = 192.168.1.193 - 192.168.1.222                               |
+----------------------------------------------------------------------+

*Reminder: only the LAST relevant octet changes based on this math ---
the earlier octets always stay exactly as they were in the original IP.*
