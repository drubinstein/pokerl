+++
title = 'Setting up the Environment'
weight = 2
+++

# Setting up the Environment

When I start designing an RL system, I start with the environment. The Python library [Gymnasium](https://gymnasium.farama.org/) provides a fairly straightforward API which can be simplified into two functions: Reset and Step

## Resets, Episodes and the Goal

Reset handles the initialization for a gameplay session also known as an “episode.”

In most video game DRL projects I've observed a reset will happen when an agent

- Achieves a major milestone, such as defeating a boss.
- Encounters a failure state, such as losing all their lives.
- Reaches a predefined time or action limit, such as 100 tetriminos in Tetris,

Pokémon is in the realm of “long episodic RL.” Long episodes mean that the agent may not get large rewards for a very long time. If the agent only receives large rewards at the end of an episode, it may have trouble creating long term policies.

Imagine if in the Tic-Tac-Toe example, the grid was 100x100. That means the agent wouldn't receive a reward for up to a max 5000 steps. Imagine having to plan out a 5000 step strategy. It's not easy! Later, I'll go over how I handled rewards for Pokémon's long episodes.

Based on prior work, I began with an episode being a fixed number of steps. Over time, I tried multiple additional strategies including dynamically increasing the number of steps per episode as the agent performed important milestones. However, I believe an episode *should be* the based on a goal or when the agent achieves an unrecoverable state (soft-lock), e.g., such as running out of money. 

What's the **goal** of Pokémon I aimed to complete? To be the champion! Therefore, I came up with a compromise. I created “mini-episodes.” An episode would be the duration of an entire game. However, the agent's state would periodically reset mid-episode, but the emulator state would not. 

## Steps

Step handles actions meant for the environment and returns observations based on a taken action, any logging info and whether or not to reset the environment.

Our step function for Pokémon can naively be written as:

- Receive a button press action
- Send the button press to the gameboy emulator
- Wait some frames for the action to have an effect on the environment
- Sample the environment
- Return data based on the sample

And that’s really it. Once I had the game loop running in a Gymnasium Environment, I could begin to implement game-specific functionality. 
