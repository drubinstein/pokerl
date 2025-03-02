+++
title = 'The Swarm'
weight = 43
+++

# The Swarm

<div style="text-align: center; vertical-align: top">

{{% columns ratio="1:1"  %}}

{{< figure
  src="assets/coords.gif"
  caption="A swarm after defeating Brock."
  class="ma0 w-75"
>}}

<---> <!-- magic separator, between columns -->

{{< figure
  src="assets/wandb.png"
  caption="The corresponding metric in WandB."
  class="ma0 w-75"
>}}


{{% /columns %}}

</div>

We have not mentioned an important modification made to RL loop. Pokémon is a nearly open world game. Unfortunately, the experiential data can get "non-coherent” and the agents can become detached from each other. 

Without sufficiently coherent data, the policy will not improve. This plagued us for *months*. Eventually, with inspiration from the [Go-Explore](https://arxiv.org/abs/1901.10995) paper, we changed the way training is done and assurances that the data would remain coherent. 

We began recording the save state every time an agent met a required game objective. Every time an agent completed a required objective, *every* agent loads the save state from the agent that completed the new objective. We tried a variety of other methods, but this method worked the best. We dubbed this method *swarming*.

Unfortunately, swarming means agents may not generalize. To account for that, we reset our save state tracker every time the game is won. As mentioned in the beginning, traditionally, RL runs for multiple episodes of independent agents. We run multiple very long episodes of cooperative agents.

