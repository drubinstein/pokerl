---
title: Learning Pokémon With Reinforcement Learning 
type: docs
---

<div style="text-align: center;">

![](logo.png)

</div>

# Learning Pokémon With Reinforcement Learning

Hi! Since 2020, I've been developing a reinforcement learning (RL) agent to beat the 1996 game Pokémon Red.
As of February 2025, I am able to beat Pokémon Red with Reinforcement Learning using minimal game knowledge with the caveat that the trained policy cannot be used to beat the game from scratch. This website acts as a journal describing my agent's current state. [All code is open sourced and available for you, the reader, to try.](https://github.com/thatguy11325/Pokémonred_puffer).

![](assets/mapvid.gif)

As I improve and add to this website, I will update the changelog. Before you dive into the website,
let me provide some background on Pokémon and my motivation for using RL.

## Pokémon Red Primer

Pokémon Red, released in 1996, is a single player Japanese role playing game (JRPG). 
Players follow the journey of a new “Pokémon Trainer.” Players capture Pokémon “creatures” to battle against opposing Pokémon, 
explore the world and progress through the game’s storyline. Pokémon has two goals:

- Catch all possible Pokémon species.
- Become the “champion.”

Most players focus on becoming the champion, which will be the focus of my work.

## Why Pokémon Red

Why do I care about developing an agent to beat Pokémon with machine learning? 
The answer is really a bit higher level. I care about solving _JRPGs_ with machine learning as a stepping stone towards AGI. JRPGs: 

- Involve complex reasoning and decision making.
- Are nonlinear.
- Can be long such as > 24 hours average human gameplay.
- Require multi-task reasoning.
- Have non-obvious reward functions.

Of JRPGs, Pokémon is relatively easy to program for. 
The [Pokémon Reverse Engineering Team](https://github.com/pret) (PRET) and the [PyBoy Python Gameboy Emulation](https://github.com/Baekalfen/PyBoy) projects have 
made it extremely easy to introspect the game and extract data as needed. Throughout this website, I'll show many examples of how I leveraged these tools for this work.

## Why Pokémon Red with RL
I could’ve taken many approaches if wanted an agent to Pokémon with machine learning.

- I could’ve chosen a supervised learning approach, but that would have needed a well 
labelled and plentiful dataset and honestly a model larger than I had the budget to support.

- I could’ve chosen behavioral cloning and attempted to build a model that imitates a
known speedrun route. I tried that once, but struggled to make an efficient data collection system.

- I could’ve attempted to beat the game with an LLM like ChatGPT like Pimanrules did 
in his video [Can ChatGPT play Pokémon Crystal? (with GPT-4V)](https://www.youtube.com/watch?v=Dct7dffObpY), but that would have
required a lot more money and computing power than I had at my disposal.

Of the many approaches I considered, RL appealed to me the most.
RL’s goal is to train a “policy” capable of determining what actions will maximize 
a reward function based on observations taken from an environment. 

What makes RL special to me is how you collect data in an RL training system. 
The data is almost always new data. No need to build 
complex data collection systems, manage large datasets or worry if the dataset is 
out of date. If you can build a system that can create new data on the fly, you can start training. 

With RL, I could build an agent with minimal resources and get amazing results. As stated before, that's what this blog shows.

## Changelog
