+++
title = 'Results'
weight = 50
+++

# Results and Concluding Thoughts

So how did this agent perform? Well, I now have an agent that beats Pokémon with a few caveats. The code is open sourced for anyone to play with. Currently, I'm still struggling to get a stable enough run to prove that I can beat it with all my scripts disabled. I have tried runs with each script individually removed so I know it can be done, but there are a couple of bugs that sometimes occur that I need to iron out.

## What did I learn?

I learned that reward shaping is super important. I learned that my initial belief was right. JRPGs are `SPECIAL` and I still believe they are a stepping stone towards AGI. 

There’s a lot more to learn though. There are still scripts to remove. What if I could have the agent `STRENGTH` puzzles? I think at this point, more compromises will have to be made and the small model will be insufficient. I’m curious what is possible with LLMs. Recently, [Claude showed interesting results playing Pokémon Red with an LLM](https://www.anthropic.com/research/visible-extended-thinking). Although Claude isn't using RL to play the game and the agent was most likely trained with Pokémon data (and therefore not a zero-shot approach), I’m curious when an LLM will replace my entire approach. I wish I could remove the swarming technique and many of the in-game knowledge rewards I provided, e`SPECIAL`ly event rewards, but I don’t think I can without making compromises on training `SPEED`.

Map ID rewards are still my least favorite addition overall, but without the ability for the agent to understand in-game text I don’t know how they can be removed.

But finally, exploration really is super powerful, but hard to tune. I think more time should be placed on how to explore as opposed to exploiting it.