+++
title = 'Battling'
weight = 22
+++

# Battling

Half of the previously mentioned objectives involve battling and winning against other trainers. However, I concluded that teaching battling is not required to make an agent capable of completing Pokemon with RL.

The Pokemon battle system is pretty straight forward. Pokemon is an advanced game of rock-paper-scissors. Pokemon have moves that can be used to either inflict status effects against other Pokemon, damage other Pokemon or boost their own abilities or self-heal. The strength of each move is affected by the strengths and weaknesses of each Pokemon relative to the move and overall stats. Moves are given priority based on a Pokemon's SPEED stat or based on a move's priority. Damage is based on the Pokemon's ATTACK, SPECIAL and DEFENSE stats depending on the move. Additionally, moves can miss based on a move and Pokemon's ACCURACY stats. 

During battles, the agent can additionally switch their current Pokemon in-battle or use items to heal their Pokemon.

There are two types of battles in Pokemon. The first type are *wild* battles where the agent will battle against a single Pokemon. Wild battles generally begin with a random encounter in grassy areas or dungeons. The second type are *trainer* battles. Trainer battles place the agent against a team of one or more Pokemon. Trainers have access to the same item and party switch actions the agent has.

Battles end when all Pokemon on one side have fainted. Fainting is when a Pokemon's health drops to zero. Additionally, during a wild battle, the agent has the option to *run* and leave the wild battle without penalty or *catch* the opposing Pokemon and have the opposing Pokemon join their party.

If the agent's Pokemon knocks out the opposing Pokemon, they gain experience (EXP). With enough EXP, a Pokemon will level up. Levelling up provides a mechanism to increase the Pokemon's stats. Levelling up to increase stats gives a mechanism to ignore worrying about battling. If the agent “grinds,” that is, let Pokemon level up or retry trainers infinite times, the agent will eventually win outside of a couple of very unlikely situations. 

Consequentially, I ignored creating a policy for battling and focused on providing suitable conditions for levelling up when needed.