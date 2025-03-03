+++
title = 'Reading RAM'
weight = 41
+++

# Reading GameBoy assembly

We have metrics, we have observations, we have the rewards. But we need a surefire way of knowing how to calculate those values. There isn't an easy way. If we had a model that could read the screen and tell us all the values we needed, then we could probably use that model to beat Pokémon.

But we don't have that model. However, we have [PRET](https://github.com/pret/pokered/tree/master). We have read an extraordinary amount of GameBoy assembly (also learned GameBoy assembly from [gbdev](https://gbdev.io/gb-asm-tutorial/)) for this project.

## The Simple Case - Reading Memory

PyBoy provides a `.memory` function for reading RAM. GameBoy games contain multiple banks of RAM. Each RAM bank contains a different purpose. [SRAM](https://github.com/pret/pokered/blob/master/ram/sram.asm) is for saving, [HRAM](https://github.com/pret/pokered/blob/master/ram/hram.asm) (high RAM) is volatile, highly available RAM on the CPU. [WRAM](https://github.com/pret/pokered/blob/master/ram/wram.asm) (work RAM) contains the majority of useful memory addresses to look at.

If you want to read your party information? That's in WRAM:

<div style="border:1px solid black;">
{{< highlight asm >}}

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

{{< /highlight >}}
</div>

In this example, any name that begins with `w` is the name of an addres in WRAM. `party_struct` is a macro that defines a party member.

<div style="border:1px solid black;">

{{< highlight asm >}}
MACRO party_struct
	box_struct \1
\1Level::      db
\1Stats::
\1MaxHP::      dw
\1ATTACK::     dw
\1DEFENSE::    dw
\1SPEED::      dw
\1SPECIAL::    dw
ENDM
{{< /highlight >}}
</div>

If you want to read various temporary buffers? Look at `HRAM`:

<div style="border:1px solid black;">
{{< highlight asm >}}

UNION
hBaseTileID:: ; base tile ID to which offsets are added
hDexWeight::
hWarpDestinationMap::
hOAMTile::
hROMBankTemp::
hPreviousTileset::
hRLEByteValue::
	db

hTextID:: ; DisplayTextID's argument
hPartyMonIndex::
	db

{{< /highlight >}}
</div>

There are a ton of valuable memory locations to ready. `wEventFlags:: flag_array` provides the event array that we use for observations. `wPlayerDirection:: db` is a single byte that represents which of the 4 directions the player is facing. `wIsInBattle:: db` tells if the agent is currently in battle or not and if that battle is a wild battle or trainer battle. `wSafariSteps:: dw` is a 2 byte value that represents the number of steps left in the Safari Zone.

## Actually Retrieving the Data

You probably noticed we only mentioned the *names* of the variables. Names are great, but are not sufficient for getting the value of that variable. We need to get the RAM value's address. When we first started, we were counting byte offsets by hand to find address locations for PyBoy to retrieve. However, PRET provides a *[symbol table](https://github.com/pret/pokered/blob/symbols/pokered.sym)* that maps labels in assembly to their addresses. 

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

In this example, the addresses for specific variables are on the left and map to their WRAM symbol on the right. We can use this lookup table to reference a variable by name and pass the discovered address to PyBoy (PyBoy now supports symbol names as input to `memory`).

## PyBoy Hooks

If you read the symbol table linked you'll notice that not all values are WRAM values. What gives? Well the symbol table also tracks assembly *labels*. Labels point to addresses in the ROM while the game is playing. If the player interacts with an in-game script, a label may be passed!

PyBoy happens to provide a useful mechanism for injecting code at these labels. For example, there is no WRAM value to tell if `CUT` was successfully used. There is however the `CUT` subroutine.

<div style="border:1px solid black;">
{{< highlight asm >}}

UsedCut:
    xor a
    ; Initialize to a failure value;
    ld [wActionResultOrTookBattleTurn], a 
    ld a, [wCurMapTileset]
    and a ; OVERWORLD
    ; If in the overworld, jump to .overworld (below);
    jr z, .overworld
    cp GYM
    ; If not in a gym, jump to .nothingToCut (below);
    jr nz, .nothingToCut
    ; Load the tile in front of the player into register a.
    ld a, [wTileInFrontOfPlayer]
    ; Check if the tile in front is 0x50, a gym Cut tree (applies to Erika's gym).
    cp $50 ; gym Cut tree
    ; If the tile is not 0x50, then jump to .nothingToCut (below).
    jr nz, .nothingToCut
    ; The tree is Cuttable! Jump to .canCut.
    jr .canCut
.overworld
    dec a
    ; Load the tile in front of the player into register a.
    ld a, [wTileInFrontOfPlayer]
    ; Check if the tile in front ix 0x3D.
    cp $3d ; Cut tree
    ; If the tile is 0x3D, then jump to .canCut.
    jr z, .canCut
    ; Check if the tile in front ix 0x52.
    cp $52 ; grass
    ; If the tile is 0x52, jump to .canCut.
    jr z, .canCut
.nothingToCut
    ; Display text telling the player that there's nothing to Cut.
    ld hl, .NothingToCutText
    jp PrintText

.NothingToCutText
    text_far _NothingToCutText
    text_end

{{< /highlight >}}
</div>

We could (and did) inject hooks at `.nothingToCut` and `.canCut` to tell if (a) `CUT` was attempted and (b) if `CUT` was successful or not.

## A Strategy for Reading Disassembly

With this we had a strategy for navigating the Pokémon Red disassembly.

1. Search the disassembly or symbol table for key words related to what you want to obtain information for, e.g., `CUT`, `StartMenu`, `HealingMachine`. If you know of a location, you can go straight to the location's `scripts` or `map_header` file.
2. Find relevant scripts in the disassembly.
3. Follow the disassembly subroutines and see if the area matches what you are looking for.
4. If the value is in HRAM or WRAM, then read those, else create a hook at the closest label.

This method scaled tremendously. We access over 80 symbols for all of training.