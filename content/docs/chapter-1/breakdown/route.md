+++
title = 'The "Route"'
weight = 26
+++

# Defining a "Route"

For the agent to complete all objectives, we wanted to simplify the number of game as much as possible to maximizing the likelihood of success. To limit non-determinism, we started the agent after the "Parcel Delivery" event. Starting the agent with a specific Pokémon would guarantee later stages of the game would be possible. Given the previous breakdown, here’s the *route* we wanted the agent to learn:

{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
  layout: elk
---
flowchart TD

START("Start with Bulbasaur or Charmander to guarantee a Pokemon who can use CUT")
BROCK("Defeat Brock")
MISTY("Defeat Misty")
HM01("Acquire HM01")
TEACH_CUT("Teach CUT")
LTSURGE("Defeat Lt. Surge")
ERIKA("Defeat Erika")
KOGA("Defeat Koga")
SABRINA("Defeat Sabrina")
BLAINE("Defeat Blaine")
GIOVANNI("Defeat Giovanni")
HM03("Acquire HM03")
TEACH_SURF("Teach SURF")
HM04("Acquire HM04")
TEACH_STRENGTH("Teach STRENGTH")
TEAM_ROCKET("Complete Team Rocket Storyline")
LAPRAS("Acquire the gift Lapras")
RIVAL6("Defeat Rival 6")
E4("Defeat E4 and Champion")

START --> BROCK 
BROCK --> MISTY
MISTY --> TEACH_CUT
BROCK --> HM01 --> TEACH_CUT --> LTSURGE
MISTY --> LTSURGE
TEACH_CUT --> LTSURGE
TEACH_CUT --> ERIKA
TEACH_CUT --> TEAM_ROCKET
TEAM_ROCKET --> LAPRAS --> TEACH_STRENGTH
LAPRAS --> TEACH_SURF
TEAM_ROCKET --> KOGA
TEAM_ROCKET --> SABRINA
TEAM_ROCKET --> HM04
TEAM_ROCKET --> HM03
KOGA --> TEACH_SURF
HM03 --> TEACH_SURF --> BLAINE
ERIKA --> TEACH_STRENGTH
HM04 --> TEACH_STRENGTH

LTSURGE --> GIOVANNI
ERIKA --> GIOVANNI
SABRINA --> GIOVANNI
BLAINE --> GIOVANNI

GIOVANNI --> RIVAL6 
TEACH_STRENGTH --> E4
RIVAL6 --> E4


{{< /mermaid >}}

This route is *extremely* complex. Again, the average human player will take 25 hours to beat Pokémon on the first try. Incentivizing exploration for a space this big took a lot of effort.

## Risk Management

Even though we simplified the game, we still needed risk mitigation. Pokémon contains numerous game ending risks including:

* Permanently losing vital Pokémon  
* Catching Pokémon and not having enough space for the Lapras or any other Pokémon that can learn `SURF`  
* If the agent obtains too many items, then there may not be enough room for key items  
* It is possible to soft-lock if you cannot obtain any more money at the time the Safari Zone objectives need to be completed.  
* Only having Pokémon with non-damaging moves.

I’d like to emphasize again that *none* of these issues require teaching the agent how to best Pokémon battles.

## Scripting vs. Emergence

To handle these risks and difficulties, we split agent behavior into two classes:

* **Scripted:** Meaning actions taken by the agent not controlled by the policy.  
* **Emergent:** Allowing the agent to discover its own strategies.

Ideally, no scripted behavior would be needed. However, we needed scripts. We aimed to only script behaviors that require human intuition or tasks that are not really a core part of Pokémon. These included.

* Item management - the agent will toss all non-key items if the agent fills their inventory.
* Money management - the agent will have infinite money.  
* Solving puzzles that require `STRENGTH`. 
* Blocking the Indigo Plateau exit at the end of the game.

To make development easier, we wrote scripts to remove complexity and speed up development. When writing RL systems, this is common practice. Make the simplest environment possible, then slowly roll back the assumptions. These scripts are now unneeded, but were invaluable during development. They included

* Teaching HMs.
* Automatically using HMs outside of battle.
* Automatically Pokéflute.
* Maxxing the Pokémon’s stats.
* Automatic insertion of drinks for the Saffron guards if the agent *entered* the Celadon Mart.
* Automating elevator usage. If an agent entered an elevator, the elevator would go to the next floor modulo number of floors.
* Disabling wild battles to simplify dungeons.