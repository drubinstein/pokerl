+++
title = 'Results'
weight = 50
+++

# Results and Concluding Thoughts

So how did this agent perform? We now have an agent that beats Pokémon with a few caveats. Currently, we're still struggling to get a stable enough run to prove that the system can beat the game with all scripts disabled. We have seen winning runs with each script individually removed so we know it can be done, but there are a couple of bugs that sometimes occur that we need to iron out.

## What did we learn?

We learned that reward shaping is super important. We learned that our initial belief was right: JRPGs are special and we still believe they are a stepping stone towards more powerful AI. 

However, there's still a lot more to learn. There are still scripts to remove. What if we could have the agent learn to solve the `STRENGTH` (Sokoban) puzzles? 

We're also curious as to what is possible with LLMs. Recently, [Claude showed interesting results playing Pokémon Red with an LLM](https://www.anthropic.com/research/visible-extended-thinking). Although Claude isn't using RL to play the game and the agent was most likely trained with Pokémon data (and therefore not a zero-shot approach), we're curious when an LLM will replace our entire approach and how efficient that approach will be. Furthermore, we wish we could remove the swarming technique (for stability) and many of the in-game knowledge rewards we provided, but we don’t think we can without making compromises on training speed.

Map ID rewards are still our least favorite addition overall, but without the ability for the agent to understand in-game text, we don’t know how they can be removed.

Finally, optimizing for exploration really is super powerful, but hard to tune. We think there's room in the world to focus on exploration techniques.