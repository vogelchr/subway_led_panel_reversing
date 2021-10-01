# Subway Sign Pinout Reversing

## J1 Digital Signal Input

    (sketch)           +-------+
    (sketch)    DI?    > 1   2 | DI/IC1
    (sketch)    DI/IC4 | 3   4 | ?
    (sketch)    ~E1    | 5   6 | A2
    (sketch)    A1     | 7   8 | A0
    (sketch)    E2     : 9  10 | OE
    (sketch)    GND    : 11 12 | LE/MOD/CA
    (sketch)    GND    | 13 14 | CLK
    (sketch)           | 15 16 |
    (sketch)    J1-17  | 17 18 | J1-18
    (sketch)    J1-19  | 19 20 |
    (sketch)           +-------+

| Pin | Chip Function | Inverted? | Sign function
| --- | ----------- | --------- | -----------
|  1  | MBI5029 DI  | No        | Column bit input
|  2  | MBI5029 DI  | No        | Column bit input
|  3  | MBI5029 DI  | No        | Column bit input
|  4  |  -      |
|  5  | HC238 ~E1   | No        | Row driver inverted enable
|  6  | HC238 A2    | No        | Row select, Bit 2
|  7  | HC238 A1    | No        | Row select, Bit 1
|  8  | HC238 A0    | No        | Row select, Bit 0
|  9  | HC129 E2    | No        | Row driver enable
| 10  | MBI5029 OE  | Yes       | Column driver output enable
| 11  | GND         | | |
| 12  | MBI5029 LE  | No        | Column register latch
| 13  | GND         | | |
| 14  | MBI5029 CLK  | no       | Column bit clock
| 15  |  -      | | |
| 16  |  -      | | |
| 17  | passthrough | | |
| 18  | passthrough | | |
| 19  | passthrough | | |
| 20  |  -      | | |

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

#** Row decoder 74HC238, IC7

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

## J2 Digital Signal Output

    (sketch)          +-------+
    (sketch)          > 1   2 |
    (sketch)          | 3   4 |
    (sketch)          | 5   6 |
    (sketch)          | 7   8 |
    (sketch)          : 9  10 |
    (sketch)    GND   : 11 12 |
    (sketch)    GND   | 13 14 |
    (sketch)          | 15 16 |
    (sketch)    J1-17 | 17 18 | J1-18
    (sketch)    J1-19 | 19 20 |
    (sketch)          +-------+
