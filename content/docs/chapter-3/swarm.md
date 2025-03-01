+++
title = 'The Swarm'
weight = 43
+++

# The Swarm

I know I haven’t gotten to the final run yet. I have not mentioned an important modification I made to the normal PPO loop. Pokémon is a nearly open world game. Unfortunately, the experiential data can get "non-coherent” and the agents can become detached from each other. 

Without sufficiently coherent data, the policy will not improve. This plagued me for *months*. Eventually, I took inspiration from the [Go-Explore](https://arxiv.org/abs/1901.10995) paper. I changed the way training is done. I made assurances that the data would remain coherent. 

How? I began recording the save state every time an agent met a required game event or collected a required item. I experimented with how to use this data and the simplest variant of my experiments worked. Every time an agent completed a required objective, *every* agent would load the save state from the agent that completed the new objective.

Unfortunately, that meant the agent may not generalize and learn how to actually beat the game. To account for that, I reset the swarm every time the game is won. As mentioned in the beginning, traditionally, RL runs for multiple episodes of independent agents. Now, I run multiple very long episodes of cooperative agents.

