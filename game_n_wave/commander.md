# Kepler Commander (1975)

## Overview
Kepler Commander is a custom chip for the Kepler Game 'n' Wave that contains various I/O capabilities, including native support for the Kepler K6 processor, 60x40 dot matrix LCD control, tone generation, random number generation, joystick input and a built-in timer.

This document assumes the scenario of programming Game 'n' Wave, where Commander is paired with K6.

## Interfacing with Commander using K6
K6 can issue commands to the Commander by writing to its register G. Some commands also use registers D, E and F for parameters. Commander has auto-buffer function which stores the last known states of K6 registers D, E, F and G without having to halt K6. All commands are listed below:

|Value|Parameters|Description|
|--|--|--|
|00|E, F|Turns a single dot located at coordinates (E,F) on|
|01|D, E, F|Writes 6 continuous dots contained in D to the LCD located at coordinates (E,F)|
|02|F|Sets the note value of the tone generator to F|
|03|E, F|Turns a single dot located at coordinates (E,F) off|

Commander also uses memory addresses to send data back to K6. Following adresses are used for following purposes:

|Address|Description|
|--|--|
|$ED|Random number generator|
|$EE|Timer|
|$EF|Joystick input|

## LCD screen
Game 'n' Wave uses a 60x40 dot matrix LCD screen to display graphics. Commander contains 2400 bits of memory it uses to directly control the LCD. This memory is divided by X and Y coordinates and each value directly corresponds to a single LCD dot.

K6 can control this memory through Commander in either single-dot width or six-dot width.

### Single-dot wide write
By writing the X coordinate to register E, Y coordinate to register F, and value `!00` or `!03` to register G of K6, Commander will initiate a single dot write. It will halt K6 for a single cycle to modify its internal display memory by setting the specified dot's value to on (if writing `!00`) or off (if writing `!03`). Dots that are on will appear darker than dots that are off.

### Six-dot wide write
By writing the dot data to register D, X coordinate to register E, Y coordinate to register F, and value `!01` to register G of K6, Commander will initiate a six dot write. It will halt K6 for a single cycle to modify its internal display memory by setting the six consecutive dots to values specified in register D.

## Tone generator
Commander has a built-in tone generator that is able to produce various musical notes. It is activated by writing the desired note value to register F, and `!02` to register G of K6. It is deactivated by writing a note value of zero. Below is a table of all possible note values:

|Value|Note|
|--|--|
|`!00`|No sound (Low state)|
|`!01`|F1|
|`!02`|F#1/Gb1|
|`!03`|G1|
|`!04`|G#1/Ab1|
|`!05`|A1|
|`!06`|A#1/Bb1|
|`!07`|B1|
|`!10`|C2|
|`!11`|C#2/Db2|
|`!12`|D2|
|`!13`|D#2/Eb2|
|`!14`|E2|
|`!15`|F2|
|`!16`|F#2/Gb2|
|`!17`|G2|
|`!20`|G#2/Ab2|
|`!21`|A2|
|`!22`|A#2/Bb2|
|`!23`|B2|
|`!24`|C3|
|`!25`|C#3/Db3|
|`!26`|D3|
|`!27`|D#3/Eb3|
|`!30`|E3|
|`!31`|F3|
|`!32`|F#3/Gb3|
|`!33`|G3|
|`!34`|G#3/Ab3|
|`!35`|A3|
|`!36`|A#3/Bb3|
|`!37`|B3|
|`!40`|C4|
|`!41`|C#4/Db4|
|`!42`|D4|
|`!43`|D#4/Eb4|
|`!44`|E4|
|`!45`|F4|
|`!46`|F#4/Gb4|
|`!47`|G4|
|`!50`|G#4/Ab4|
|`!51`|A4|
|`!52`|A#4/Bb4|
|`!53`|B4|
|`!54`|C5|
|`!55`|C#5/Db5|
|`!56`|D5|
|`!57`|D#5/Eb5|
|`!60`|E5|
|`!61`|F5|
|`!62`|F#5/Gb5|
|`!63`|G5|
|`!64`|G#5/Ab5|
|`!65`|A5|
|`!66`|A#5/Bb5|
|`!67`|B5|
|`!70`|C6|
|`!71`|C#6/Db6|
|`!72`|D6|
|`!73`|D#6/Eb6|
|`!74`|E6|
|`!75`|F6|
|`!76`|F#6/Gb6|
|`!77`|No sound (High state)|

## Random number generator
Commander features a 6-bit random number generator that can be accessed by reading memory address `$ED`. This number is refreshed every cycle.

## Timer
Commander features a 6-bit timer that counts down 60 times per second. This is useful for timing-critical tasks like animation, music and others. It is initiated when a write to memory address `$EE` occurs. K6 can read from this address to see the current status of the timer. Commander will decrement the said value until it reaches zero.

## Joystick input
Commander can interface with a joystick with two fire buttons. K6 can read the current state of the joystick by reading from `$EF`. All inputs are condensed into one hyte, with the following layout:
```
              $EF = UDLR12
                    ||||||
       JOYSTICK UP -+|||||
     JOYSTICK DOWN --+||||
     JOYSTICK LEFT ---+|||
    JOYSTICK RIGHT ----+||
            FIRE 1 -----+|
            FIRE 2 ------+
```
In this hyte, a value of 0 represents a non-pressed input, while a value of 1 represents a depressed input.
