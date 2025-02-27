+++
title = 'Risk Management'
weight = 27
+++

# Risk Management

Even though I simplified the game, I still needed risk mitigation. Pokémon contains numerous game ending risks including:

* Permanently losing vital Pokémon  
* Catching Pokémon and not having enough space for the Lapras or any other Pokémon that can learn Surf  
* Item management. If the agent obtains too many items, then there may not be enough room for key items  
* Money management. It is possible to soft-lock if you cannot obtain any more money at the time the Safari Zone objectives need to be completed.  
* Only having Pokémon with non-damaging moves.

I’d like to emphasize again that *none* of these issues require teaching the agent how to best Pokémon battles.

## Scripting vs. Emergence

To handle these risks and difficulties, I split agent behavior into two classes:

* **Scripted:** Meaning actions taken by the agent not controlled by the policy.  
* **Emergent:** Allowing the agent to discover its own strategies.

Ideally, no scripted behavior would be needed. However, I needed scripts. I aimed to only script behaviors that require human intuition or tasks that are not really a core part of Pokémon. These included.

* Item management \- the agent will toss all non-key items if the agent fills their inventory.
* Money management \- the agent will have infinite money.  
* Solving puzzles that require Strength. 
* Blocking the Indigo Plateau exit at the end of the game.

To make development easier, I wrote scripts to remove the following complexity and speed up development. These scripts can disabled for a successful run, but were invaluable during development. They included

* Teaching HMs.
* Using HMs outside of battle.
* Using Pokéflute.
* Maxxing the Pokémon’s stats.
* Automatic insertion of drinks for the Saffron guards if the agent *entered* the Celadon Mart.
* Automating elevator usage. If an agent entered an elevator, the elevator would go to the next floor modulo number of floors.
* Disabling wild battles to simplify dungeons.
