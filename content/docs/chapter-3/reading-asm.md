+++
title = 'Reading RAM'
weight = 41
+++

# Reading GameBoy ASM

I have metrics, I have observations, I have the rewards. But I needed a surefire way of knowing what the values were. There isn't an easy way. If I had a model that could read the screen and tell me all the values I needed, then I could probably use that model to beat Pokémon.

But I don't have that model. However, I have [PRET](https://github.com/pret/pokered/tree/master). I have read an extraordinary amount of GameBoy ASM (also learned GameBoy ASM from [gbdev](https://gbdev.io/gb-asm-tutorial/)) for this project.

## The Simple Case - Reading Memory

PyBoy provides a `.memory` function for reading RAM. GameBoy games contain multiple banks of RAM. Each RAM bank contains a different purpose. [SRAM](https://github.com/pret/pokered/blob/master/ram/sram.asm) is for saving, [HRAM](https://github.com/pret/pokered/blob/master/ram/hram.asm) (high RAM) is volatile, highly available RAM on the CPU. [WRAM](https://github.com/pret/pokered/blob/master/ram/wram.asm) (work RAM) contains the majority of useful memory addresses to look at.

If you want to read your party information? That's in WRAM:

```
SECTION "Party Data", WRAM0

wPartyDataStart::

wPartyCount:: db
wPartySpecies:: ds PARTY_LENGTH + 1

wPartyMons::
; wPartyMon1 - wPartyMon6
FOR n, 1, PARTY_LENGTH + 1
wPartyMon{d:n}:: party_struct wPartyMon{d:n}
ENDR

wPartyMonOT::
; wPartyMon1OT - wPartyMon6OT
FOR n, 1, PARTY_LENGTH + 1
wPartyMon{d:n}OT:: ds NAME_LENGTH
ENDR

wPartyMonNicks::
; wPartyMon1Nick - wPartyMon6Nick
FOR n, 1, PARTY_LENGTH + 1
wPartyMon{d:n}Nick:: ds NAME_LENGTH
ENDR
wPartyMonNicksEnd::

wPartyDataEnd::
```

In this example, any name that begins with `w` is the name of an addres in WRAM. `party_struct` is a macro that defines a party member.

```
MACRO party_struct
	box_struct \1
\1Level::      db
\1Stats::
\1MaxHP::      dw
\1Attack::     dw
\1Defense::    dw
\1Speed::      dw
\1Special::    dw
ENDM
```

There are a ton of valuable memory locations to ready. `wEventFlags:: flag_array` provides the event array that I use for observations. `wPlayerDirection:: db` is a single byte that represents which of the 4 directions the player is facing. `wIsInBattle:: db` tells me if the agent is currently in battle or not and if that battle is a wild battle or trainer battle. `wSafariSteps:: dw` is a 2 byte value that represents the number of steps left in the Safari Zone.

## Actually Retrieving the Data

You probably noticed I only mentioned the *names* of the variables. Names are great, but are not sufficient for getting the value of that variable. I need to get the RAM value's address. When I first started, I was counting byte offsets by hand to find address locations for PyBoy to retrieve. However, PRET provides a *[symbol table](https://github.com/pret/pokered/blob/symbols/pokered.sym)* that maps labels in ASM to their addresses. 

```
00:d16b wPartyMon1Species
00:d16b wPartyMons
00:d16b wPartyMon1
00:d16c wPartyMon1HP
00:d16e wPartyMon1BoxLevel
00:d16f wPartyMon1Status
00:d170 wPartyMon1Type
00:d170 wPartyMon1Type1
00:d171 wPartyMon1Type2
00:d172 wPartyMon1CatchRate
00:d173 wPartyMon1Moves
00:d177 wPartyMon1OTID
00:d179 wPartyMon1Exp
00:d17c wPartyMon1HPExp
```

In this example, the addresses for specific variables are on the left. I can use this lookup table to reference a variable by name and pass the discovered address to PyBoy (PyBoy now supports symbol names as input).

## PyBoy Hooks

If you read the symbol table you'll notice that not all values are WRAM values. What gives? Well the symbol table also tracks ASM *labels*. Labels point to addresses in the ROM while the game is playing. If the player interacts with an in-game script, a label may be passed!

PyBoy happens to provide a useful mechanism for injecting code at these labels. For example, there is no WRAM value to tell if CUT was successfully used. There is however the CUT subroutine.

```
UsedCut:
	xor a
	ld [wActionResultOrTookBattleTurn], a ; initialise to failure value
	ld a, [wCurMapTileset]
	and a ; OVERWORLD
	jr z, .overworld
	cp GYM
	jr nz, .nothingToCut
	ld a, [wTileInFrontOfPlayer]
	cp $50 ; gym cut tree
	jr nz, .nothingToCut
	jr .canCut
.overworld
	dec a
	ld a, [wTileInFrontOfPlayer]
	cp $3d ; cut tree
	jr z, .canCut
	cp $52 ; grass
	jr z, .canCut
.nothingToCut
	ld hl, .NothingToCutText
	jp PrintText

.NothingToCutText
	text_far _NothingToCutText
	text_end
```

I could inject hooks at `.nothingToCut` and `.canCut` to tell if (a) CUT was attempted and (b) CUT was successful or not.

## My Strategy for Reading Disassembly

With this I had a strategy for navigating the Pokémon Red disassembly.

1. Search the disassembly or symbol table for key words related to what you want to obtain information for, e.g., "cut," "StartMenu", "HealingMachine." If you know of a location, you can go straight to the location's `scripts` or `map_header` file.
2. Find relevant scripts in the disassembly.
3. Follow the disassembly subroutines and see if the area matches what you are looking for.
4. If the value is in HRAM or WRAM, then read those, else create a hook at the closest label.

This method scaled tremendously. I access over 80 symbols for all of training.