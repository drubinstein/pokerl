+++
title = 'Results'
weight = 50
+++

# Results and Concluding Thoughts

So how did this agent perform? We now have an agent that beats Pokémon with a few caveats. Currently, we're still struggling to get a stable enough run to prove that the system can beat the game with all my scripts disabled. I have seen winning runs with each script individually removed so we know it can be done, but there are a couple of bugs that sometimes occur that we need to iron out.

## What did we learn?

We learned that reward shaping is super important. We learned that my initial belief was right. JRPGs are special and we still believe they are a stepping stone towards AGI. 

However, there's a lot more to learn. There are still scripts to remove. What if we could have the agent learn to solve the `STRENGTH` (Sokoban) puzzles? 

We're curious what is possible with LLMs. Recently, [Claude showed interesting results playing Pokémon Red with an LLM](https://www.anthropic.com/research/visible-extended-thinking). Although Claude isn't using RL to play the game and the agent was most likely trained with Pokémon data (and therefore not a zero-shot approach), I’m curious when an LLM will replace my entire approach. We wish we could remove the swarming technique (for stability) and many of the in-game knowledge rewards we provided, especially event rewards, but we don’t think we can without making compromises on training speed.

Map ID rewards are still our least favorite addition overall, but without the ability for the agent to understand in-game text we don’t know how they can be removed.

But finally, optimizing for exploration really is super powerful, but hard to tune. We think there's room in the world to focus on exploration techniques.