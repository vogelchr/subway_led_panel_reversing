# Subway Sign Pinout Reversing

## J1 Digital Signal Input

    (sketch)             +-------+
    (sketch)    DIa(???) > 1   2 | DIb (IC1)
    (sketch)    DIc(IC4) | 3   4 | ?
    (sketch)    ~E1      | 5   6 | A2
    (sketch)    A1       | 7   8 | A0
    (sketch)    E2       : 9  10 | OE
    (sketch)    GND      : 11 12 | LE/MOD/CA
    (sketch)    GND      | 13 14 | CLK
    (sketch)             | 15 16 |
    (sketch)    DOA0     | 17 18 | DOA1
    (sketch)    DOX      | 19 20 |
    (sketch)             +-------+

## J2 Digital Signal Output

    (sketch)             +-------+
    (sketch)    DOa      > 1   2 | DOb
    (sketch)    DOc      | 3   4 |
    (sketch)    ~E1/J1   | 5   6 | J1-6 A2
    (sketch)    A1/J1    | 7   8 | J1-8 A0
    (sketch)    E2/J1    : 9  10 | J1-10 OE
    (sketch)    GND      : 11 12 | J1-12 LE/MOD/CA
    (sketch)    GND      | 13 14 | J1-14 CLK
    (sketch)        ?    | 15 16 | ?
    (sketch)    DOA0     | 17 18 | DOA1
    (sketch)    DOX      | 19 20 | 
    (sketch)             +-------+

## J4 System Controller

The J4 connector is a copy of J1, with 10 GND and 10 VCC pads added.

    (sketch)             +--------+
    (sketch)             >  1   2 | }
    (sketch)             |  3   4 | } all GND
    (sketch)             |  5   6 | }
    (sketch)             |  7   8 | }
    (sketch)             |  9  10 | }
    (sketch)        DOa  | 11  12 | DIb
    (sketch)        DOc  | 13  14 | -
    (sketch)        E1   | 15  16 | A2
    (sketch)        A1   | 17  18 | A0
    (sketch)        E2   : 19  20 | OE
    (sketch)        GND  : 21  22 | LE
    (sketch)        GND  | 23  24 | CLK
    (sketch)             | 25  26 | ?
    (sketch)        DOA0 | 27  28 | DOA1
    (sketch)        DOx  | 29  30 |
    (sketch)             | 31  32 | }
    (sketch)             | 33  34 | }
    (sketch)             | 35  36 | }
    (sketch)             | 37  38 | }
    (sketch)             | 39  40 | }
    (sketch)             +--------+

## J1/J2 detailed pinout

| Pin | Chip Function | Inverted? | Sign function
| --- | ------------- | --------- | -----------
|  1  | MBI5029 DI/DO | No        | J1: column bit input, J2: output
|  2  | MBI5029 DI/DO | No        | J1: column bit input, J2: output
|  3  | MBI5029 DI/DO | No        | J1: column bit input, J2: output
|  4  |  -            |           |
|  5  | HC238 ~E1     | No        | Row driver inverted enable
|  6  | HC238 A2      | No        | Row select, Bit 2
|  7  | HC238 A1      | No        | Row select, Bit 1
|  8  | HC238 A0      | No        | Row select, Bit 0
|  9  | HC129 E2      | No        | Row driver enable
| 10  | MBI5029 OE    | Yes       | Column driver output enable
| 11  | GND           |           |
| 12  | MBI5029 LE    | No        | Column register latch
| 13  | GND           |           |
| 14  | MBI5029 CLK   | no        | Column bit clock
| 15  |  -            |           |
| 16  |  -            |           |
| 17  | HC238 A0      | no        | column group select for DI/Pin 19
| 18  | HC238 A1      | no        | column group select for DO/Pin 19
| 19  | HC128 Yn      | no        | selected column read back (needs jumper)
| 20  |  -            |           |

A detailed description of the connections from connectors to individual ICs follows.

## Serial data inputs

`|>` are gates in 74HC14D, IC9

    (sketch)    J1/#1 ---[R]---1-|>-2----3-|>-4--- (?) suspect DI of MBI5029
    (sketch)    J1/#2 ---[R]---13|>12----11|>10--- DI/IC1
    (sketch)    J1/#3 ---[R]---5-|>-6----9-|>-8--- DI/IC4
    (sketch)    #** MBI5029 control

`|>` are gates in 75HC14D, IC2

    (sketch)    J1/#10---[R]---1-|>-2------------- OE MBI5029
    (sketch)    J1/#12---[R]---3-|>-4----5-|>-6--- LE MBI5029
    (sketch)    J1/#14---[R]---13|>12----11|>10--- CLK MBI5029

## Selectable data output

Pins J1/#17 and J1/#19 select which chain's SDO signal is output on J1/#19, there is also
a jumper link adjacent to J2 which needs to be installed. The 4 gates in IC6 have their
outputs paralleled, only one is shown.

    (sketch)                   +-IC5---+     +-IC6---+
    (sketch)    J1/#17(DOA0) --|A0  Y0 |-----| OE  Y0|-+--o --jumper-- o--- J1/#19(DOx)
    (sketch)    J1/#18(DOA1) --|A1  ...|  +--| A0    | |
    (sketch)                   +-------+  |  +-------+ |
    (sketch)                              |            |
    (sketch)                             SDOa      (Y1, Y2, Y3)

## Row decoder 74HC238, IC7

J1/#5, #6, #7, #8, #9 go to 75HC238 via series resistor + pulldown(?)

## Row drivers hardware

Each 75HC238 output (8 total) drives the input of a ULN2803 octal darlington
open collector driver array, with outputs pulled high, which in turn drive
the gates of 2 or 3 p-channel mosfets connected to the anodes of the LED matrix.
Row select n drives rows n, n+8 and n+16 (n=0..7 for rows 0..19).

    (sketch)
    (sketch)    74HC238 Y(x) --- [ In(x) ULN2803 Out(x) ] -+-- [pMosfet]..
    (sketch)                                               |
    (sketch)        [MBI5029 Out] ----|<|----- [pMosfet] --+-- [pMosfet]..
    (sketch)                          LED
    (sketch)

Mosfets are SDM4953, dual p-Ch enh. mode mosfets, 30V Vdsmax, ~3Vthr,
max 15A, ~50mOhm.

## Digital inputs

Standard input circuit, as indicated by the [R] symbol is...

    (sketch)    	Input ---[ 1k ]--+-- Chip
    (sketch)    	                 |
    (sketch)    			[ ]
    (sketch)    			[ ] 4.7k
    (sketch)    			 |
    (sketch)    			GND

...which acts as a voltage divider down to 82%, and also as a pulldown.

3.3 logic will most likely not work reliably (gets pulled down to 2.7V,
the rest of the circuitry is 5V logic, 5V input levels get pulled down to 4.1V).

HC-family Vih is min 3.15V, typ 2.4V with 4.5V supply.

HC input capacitance is a few pF, with ~1kOhm driving impedance, we would
be "limited" to 15MHz. MBI5029 max. is 25 MHz...


