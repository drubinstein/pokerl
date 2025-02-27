+++
title = 'RL Quickstart'
weight = 1
+++
# RL Quickstart

Before I focus on Pokemon, I'd like to start by giving a brief primer on Reinforcement Learning. Reinforcement Learning (RL) is a focus of study on how an agent should take actions in an environment in order to maximize a reward. This primer will not be exhaustive. I am writing it to set the framework for which I built my Pokemon training system.

In a typical RL framework, there will be 2 components.

1. The Environment - Represents the world context.
2. The Agent - Responsible for generating actions to perform in the environment

The agent contains a *policy* for generating an action given a state of the environment using a . The environment evaluates the action and returns an *observation* and a *reward*. These actions, observations and rewards can be used to update the agent's policy. [Deep Reinforcement Learning](https://en.wikipedia.org/wiki/Deep_reinforcement_learning) (DRL) uses a neural network for the policy. I'll be using DRL for Pokemon.

{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
  layout: elk
---
flowchart LR
    Agent --> |Action| Environment(Environment)
    Environment --> |Observation, Reward| Agent

    subgraph Agent[Agent]
        Policy(Policy)
    end

{{< /mermaid >}}

When building an RL system, I like to breakdown the problem into modeling the problem as

- the goal
- the environment
- the observation
- the actions
- the reward
- the policy / the agent

## Tic Tac Toe

Let's apply RL to Tic-Tac-Toe. RL does not need machine learning. A valid RL agent can be as simple as a look up table! For those who have never played Tic-Tac-Toe, Tic-Tac-Toe is a two player game where each player takes turns marking a single square in a 3x3 grid. The first player to have marked 3 squares making a horizontal, vertical or diagonal line wins. Let's breakdown Tic-Tac-Toe in the framework above

```
Example Tic-Tac-Toe configurations:
   X Wins          Tie         Early Game
 X | O | X      X | O | X      X |   | O  
-----------    -----------    -----------
 O | X | O      X | X | O        | O |    
-----------    -----------    -----------
   |   | X      O | X | O        |   | X  

```

### Goal

The goal of Tic-Tac-Toe is to win! After a player has achieved 3-in-a-row, the game will reset. We call this beginning of game to end of game period an *episode*.

### Environment

The environment is the Tic-Tac-Toe game itself. The environment will be responsible for performing actions on the grid, i.e., marking squares, ensuring invalid moves are not played and determining when the game has ended.

### Observation

The observation will be the current 3x3 grid represented as a 3x3 array.

### Actions

The action will be a coordinate to play on the grid. For example [0, 0] will mean mark on the upper left hand corner. [2, 2] will mean mark the lower right hand corner.

### Rewards

We can start with a reward of

-  +1 if the player wins
-  -1 if the player loses

We can even be simpler and say +1 if the player wins, -1 if the player loses. The act of modifying the reward for a better system is known as *reward shaping*. If you make your reward simpler, you risk the agent never learning how to perform complex actions. If you make your reard complex, you risk the agent will never progress beyond immediate greedy actions.

### Training the Policy

Now that we have created our Tic-Tac-Toe RL system, we can attempt to train a policy to be the best Tic-Tac-Toe player ever. At the start of training, the policy will play random moves, but as it obtains more experience, it will learn to exploit the reward function.


For Pokemon, I am going to breakdown the game similarly. Let's dig in.

