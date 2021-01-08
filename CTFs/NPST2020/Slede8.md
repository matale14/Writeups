# SLEDE8

Revision 0.2

## Introduction

There are a total of 16 8-bit registers, `r0..r15`.
Here you can cache data to be processed.
In addition, a flag is used which represents the result of comparison operations, and which determines whether the program is to perform conditional jumps.

Available memory is 4096 bytes, with the addresses `0..4095`.
Programs are loaded from address 0, so the first instruction will always have address 0.

SLEDE8 relates to binary data as little endian.

## Grammar

Programs are written with s8 assembly (`.s8asm`) and mounted to binaries (` .s8`).
In s8 assembly, a line can be a `label`,` instruction`, `data` or a` comment`.
Comments can also be added following an `instruction`.
Blank lines are ignored.

### Comments

A `comment` begins with the character`; `.
Comments are ignored when installing the program.

Example ([try it yourself] (https: //slede8.npst.no#N4Igzg9grgTgxgUwMIQCYJALhAbgAQIB2eA1hALblEAuAhjADqEByA8gAoCie+RethQnzKUa9JgGUAKh3YgANCAWWQAA:

```
; a comment
NOPE; another comment
```

### Tags

A `label` is a line on the form` ^ [0-9a-zA-ZæøåÆØÅ \ -_] +: $ `.
Labels are references to the address of the subsequent instruction or data in the code.
They can be used as an argument when jumping or if you are going to take a trip.

For examples, see the section on program flow.


### Instructions

The following `instructions` are available:


#### `SETT rA, rB`

Write the value in rB to rA.


#### `SETT rA, <value>`

Enter the value of register `rA`.
Values ​​are written in the form `0xNN` or` NN`.
The most significant bits (msb) are written to r1, while the least significant bits (msb) are written to r0.


#### `NOPE`

Do not do anything.


#### `STOPP`

Quit the program.


### Arithmetic Logical Unit

ALE operations take two registers as an argument.
The result is written to the first register.

- `OG rA, rB     ; AND`
- `ELLER rA, rB  ; OR`
- `XELLER rA, rB ; XOR`
- `VSKIFT rA, rB ; LEFT SHIFT`
- `HSKIFT rA, rB ; RIGHT SHIFT`
- `PLUSS rA, rB  ; PLUS`
- `MINUS rA, rB  ; MINUS`

** NOTE **: Arithmetic operations are modulo 0x100.

Example ([try for yourself] (https: //slede8.npst.no#N4Igzg9grgTgxgUwMIQCYJALhAZQKIAqBABDAAwA0xAjAEwAcxTA3KWcQLw0PEAUZAD3pkAlAB0AdviKlqVQQCMmxVjGqcm1dfwFkF4iZIAKAGQCqOHGyprlq9l3LEA1LMkBZAJIA5C9dl2bBpOALRuEgDyAOL+tsosQY7sAGTheCYmeABKseoJTknEAD7hABrpmTnkNnn2wewAeuEAajgA0p4AYiTVASqJQQA8g+EAEu1dPZR9dYUAfHPhkjgEEUZGIBQgAJYSAA5QAC5YICAAvkA)):

```
SETT r0, 128; r0 = 128 (0x80)
SETT r1, 0xb; r1 = 11 (0x0b)

PLUSS r0, r1   ; r0 = r0 + r1
MINUS r0, r1   ; r0 = r0 - r1
OG r0, r1      ; r0 = r0 & r1
ELLER r0, r1   ; r0 = r0 | r1
XELLER r0, r1  ; r0 = r0 ^ r1
VSKIFT r0, r1  ; r0 = r0 << r1
HSKIFT r0, r1  ; r0 = r0 >> r1

STOPP
```

### Comparison

Comparison operations take two registers as an argument.
The result is persisted in `flag` until the next comparison operation.

- `LIK rA, rB   ; EQUAL`
- `ULIK rA, rB  ; NOT EQUAL`
- `ME rA, rB    ; LESS THAN`
- `MEL rA, rB   ; LESS THAN OR EQUAL TO`
- `SE rA, rB    ; BIGGER THAN`
- `SEL rA, rB   ; BIGGER THAN OR EQUAL TO`

Example ([try for yourself] (https: //slede8.npst.no#N4Igzg9grgTgxgUwMIQCYJALhAZQKIAqBABDAAwA0xArMXQNylnEC8NAOgHb5GkCMVPs2KMYfVsSFcuAGQCSAaSZUxdEU1ZtVLAHzEAZgBsAhgHMJzABT7jhsAgCUXAKryl5FePXliAQi3iugYm5mzilgAuMFCOXACyeMr8at7MADx02npGZhLhUTFOnAkySaoMGmkBrNkhecSR0bHciR7JaqLMeslBOaHEVjZ2zfilbeWpxDrVvXVsg7b2RVw4BADyAAobIBQgAJacAA5QEVggIAC+QA)):

```
SETT r0, 5; r0 = 5
SETT r1, 10; r1 = 10

LIK r0, r1   ; r0 == r1 => flag = 0 (false)
ULIK r0, r1  ; r0! = r1 => flag = 1 (true)
ME r0, r1    ; r0 <r1 => flag = 1 (true)
MEL r0, r1   ; r0 <= r1 => flag = 1 (true)
SE r0, r1    ; r0> r1 => flag = 0 (false)
SEL r0, r1   ; r0> = r1 => flag = 0 (false)

STOPP
```


### Program flow

Sometimes you want to do more than run each instruction once from top to bottom.
SLEDE8 covers this need by supporting two types of jumps.

- `HOPP address  ;JUMP`
- `HOPP label    ;JUMP`
- `BHOPP address ;BJUMP`
- `BHOPP label   ;BJUMP`

A `HOPP` operation jumps to the selected address, while`BHOPP` only jumps if the flag is set to 1 after a previous comparison operation.

** Note **: A label is only a reference to an address, and will therefore be replaced with an address in the mounted binary file.
However, it may be easier for developers to deal with labels.

** Note **: You can jump both up and down in the code.


Example ([try for yourself] (https: //slede8.npst.no#N4Igzg9grgTgxgUwMIQCYJALhAZQKIAqBABDAAwA0xZxA3KTQLzUA6AdvkaQIxXd09izbu3YAZAJIBpBlRj9iipQPJDm8oQD5iAMwA2AQwDmQ6sQAUOg3rAIAlOwBCACQDyABXfEAtghgBrBEMAB2CyfnoACwhQv2IAS39AqjB49DZdQxNmMnFpWQZlZXpVRnUmbX1jU35zABcYKHsnN08fP0CQsIjiaNDiOvi9doCgg1Dw4lT0zOqy4hEOAg8vIrWBdLYEBLYwBqh-MAArCC2MgCM9eJhemOCEOuIIADc-UTZfUa7wzHZWr0+nXGYQATIooncBkMRkCJiD2Dhlm11kV6JttvFdvtDiczsRLtdbrFHi83mx2ICxnDfksVijUcRgjAIEYYAZvL5Hns7nFImT3iAKCBMcEoHUsCAQABfIA)):

```
SETT r0, 0; r0 = 0
SETT r1, 1; r1 = 1

LIK r0, r1     ; r0 == r1 => flag = 0 (false)
BHOPP tag01    ; does not jump, since flag = 0
LIK r0, r0     ; r0 == r0 => flag = 1 (true)
BHOPP tag01    ; jump to tag01 page flag == 1
STOPP          ; this instruction is skipped

tag01:
HOPP tag02     ; jump to tag02
STOPP          ; this instruction is skipped

tag02:
STOPP          ; the program stops here
```

### Food and regurgitation

You can feed the program by using the instruction `LES rN`.
This moves the first non-consumed byte of food into the selected register, but assumes that there is food left.
The program stops working if it detects that it has run out of food.

The program can provide regurgitation(close enough translation) by using the instruction `SKRIV rN`.
The value in the selected register is then spat out.

** Note **: You do not need to consume food to produce regurgitation!

** Note **: On [slede8.npst.no] (slede8.npst.no), food for the program is presented as a hex string.

Example ([try for yourself] (https: //slede8.npst.no#N4Igzg9grgTgxgUwMIQCYJALhAbgASpR4DWAhgHZ4LmowJ4DyADkwOZQA2TeALgJYc8AU7wA3PmHoBBAMpIAkvIC0ohDFR81AHXL5VqYbxgBPYsXpMRU0kuLlSLagDodOmQFEAKp7wwADAA0eH4AHgAsAIx4ePj+eAC8weERbgDSAEryAGq+ftH50fhgxDB8okmReAAUAORSNQCUOgAy7jK5BZ2xeYmhkWmZOXGdMXjFpeV9AEzVNQBCjS1tHSOFHb3hAMwD2StdYyVlFZuzSIvkMp4MAArXq-n4pKJgHFA8PHgAZgAfMGJ8eFYQj+PAgAFsvhA-j90CAAiA+OQmG8sCAwlMwpsQABfIA)):

```
; with food equal to '4243' this gulps up 'ABC'

SETT r0, 0x41   ; r0 = 0x41
SKRIV r0        ; type 0x41 ('A')
LES r0          ; r0 = 0x41
SKRIV r0        ; type 0x42 ('B')
LES r0          ; r0 = 0x43
SKRIV r0        ; type 0x43 ('C')
STOPP           ; quit before we run out of food
```


### Features

You can take a trip to an address or label by using the function `TUR address`
When you are tired of being on a trip, you can return to the departure address by using the instruction `RETUR`

** Note **: Labels can also be used as an argument when taking a `TUR`.

#### `FINN <tag | address> `

Write the address of the label of r0 and r1. Most significant bits (msb) are written to r1, and least significant bits (msb) are written to r0.
If the address is `0xABC`, then` r0` is set to `0xBC` and` r1` to `0x0A`.

#### `LAST rN`

Write the value from the address `((r1 << 8) | r0) & 0x0FFF` to the register` rN`.

#### `LAGR rN`

Write the value in the register rN to the address `((r1 << 8) | r0) & 0x0FFF`.
    

### Data

One can represent `data` by writing lines the form` ^ .DATA [x0-9a-fA-F,] * $ `.
To enter data (possibly instruction bytes) you use ".DATA" followed by a comma-separated list of numbers.
Typically, you will also put a `label` just before.

You get the most out of this option if it is used with `FINN` and` LAST`.

Example:

```
MyString:
.DATA 0x48,0x65,0x6c, 0x6c, 0x6f, 0x2c, 0x20,0x77,0x6f, 0x72,0x6c, 0x64
```


English translation for commands:

SETT = SET
NOPE = NOPE
STOPP = STOP
OG = AND
ELLER = OR
XELLER = XOR
VSKIFT = LSHIFT
HSKIFT = RSHIFT
PLUSS = PLUS
MINUS = MINUS
LIK = EQUAL
ULIK = NOT EQUAL
ME = LESS THAN
MEL = LESS THAN OR EQUAL TO
SE = LARGER THAN
SEL = LARGER THAN OR EQUAL TO
HOPP = JUMP
BHOPP = BJUMP (Conditional jump)
LES = READ (From input/Food)
SKRIV = WRITE (To output/regurgitation)
FINN = FIND
LAST = LOAD
LAGR = STORE
TUR = TRIP
RETUR = RETURN

