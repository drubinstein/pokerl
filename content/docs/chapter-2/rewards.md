+++
title = 'Rewards'
weight = 31
+++

# Rewards

Rewards was where I was willing to be a bit more flexible and leak some information that an agent wouldn’t normally have access to. Only rewarding the storyline is very sparse. It can be thousands of steps before a high value event is attempted. The agent needs some way knowing it is making progress in between these high value sparse rewards. 

### Sparse vs. Dense Rewards

**Sparse rewards** were the rewards I chose that were very high value but rarely occur, e.g. gym battles. If we were to only reward the sparse rewards, the agent would most likely never leave the starting point in Pallet Town.

**Dense rewards** are like breadcrumbs. They are for teaching an agent how to explore or interact with the world. By mixing dense rewards in, the agent can progress and eventually discover the highly valued sparse rewards. 

### Is Exploration All You Need?

Exploration became the foundation for all dense rewards.

I originally believed that an agent had successfully learned Pokémon if the agent could visit every coordinate in the game’s world. I loosened that definition. I now consider *any* interaction a form of exploration.

However, only using exploration rewards are not enough. The mix of sparse and dense rewards is what makes the policy successful! If I were to only reward for unique coordinates, the agent may never interact with Brock, the first gym leader. Instead, the agent would wander the world forever collecting new locations until it ran out of locations.

In the end, I rewarded for:

- The number of unique coordinates visited during a mini-episode.
- The number of signs interacted the agent interacts with during a mini-episode.
- The number of unique moves taught.
- The number of valid and invalid uses of HMs/Pokéflute in the overworld.   
- The number of tile types an HM/Pokéflute was used on.
- The number of unique map ID transitions (warps) visited during a mini-episode.   
- The number of unique seen and caught Pokémon.
- The current level of the agent’s party with a cutoff to prevent grinding (not exploration related).

I tried to also reward for all locations an agent has pressed A on and the number of unique menus the agent visited during an epiosde. However, the A press and unique menus reward provided no gain and slowed down training tremendously.

## Making Game Progress?

Game progress was ultimately defined by high value sparse rewards. These included:

- Gym badges obtained.
- All required events completed.
- All required items obtained.

Without these rewards, the agent would ultimately wander until the mini-episode reset and make no progress.

## Map ID Rewards

I additionally gave the option for a "boosted" reward if the agent explored a currently important map ID. I understand this is a controversial choice. On one hand, I am potentially putting the game on rails. On the other hand, I experienced that the agent would have no clue where to go next unless I added boosted rewards for specific map IDs since the agent could not understand text. Often in Pokémon, an NPC will tell the player where to go next. The agent not being able to understand text obviously could not know where to go so I compromised.

Specific map IDs start the game with a _boosted_ exploration reward *until* the map ID’s objective is complete. For example, a gym has boosted reward until it is beaten. Only two map IDs break this rule. 

- Route 23 does not get boosted until all gyms are beaten  
- Lavender Tower does not get boosted until the Celadon City rocket event is completed. Otherwise, agents would not leave Lavender due to the level up reward obtained from the wild battles inside Lavender Tower. 

## Safari Zone Rewards

The Safari Zone minigame provided a big challenge. The Safari Zone is a minigame where two required items must be obtained within 500 steps. The Safari Zone can be attempted as long as you can pay for it. Safari Zone’s route is simple to reward for until a fork in the road 75% the way in. I needed to incentivize the agent to get to that fork as fast as possible *and* take the correct fork.

I added a specific Safari Zone reward for this case. The agent obtains a reward proportional to the number of steps left for any map id it reaches within the Safari Zone. Ideally, the agent’s visited mask will prompt the agent to explore both forks within the same episode. In reality, the agent does eventually complete the Safari Zone but it can take thousands of steps during training.