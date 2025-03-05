+++
title = 'The Environment'
weight = 2
+++

# The Pokémon Environment

## Game Overview

Before discussing observations, rewards, and the policy, we will provide a brief overview of the game's challenges. Pokémon is a complex game with multiple tasks and puzzles that can be accomplished nonlinearly. 

{{% hint info %}}
We cover Pokémon's challenges in greater depth in the [Appendix]({{<ref "/docs/appendix/breakdown/battling/index" >}} "appendix").
{{% /hint %}}

At a high level, to beat Pokémon, you must:

1. Beat the 8 Gym Leaders (a form of video game boss).  
2. Acquire `HMs` (items) to teach the field moves `CUT`, `STRENGTH`, and, `SURF` and Pokémon to teach the field moves to. Field moves are abilities that can be used outside of battle to unlock new areas or make it easier to traverse an existing area.  
3. Teach capable Pokémon the field moves `CUT`, `STRENGTH`, and `SURF`.
4. Acquire any items (not HMs) required for field interactions. Like field moves, there are items that are required to unlock new areas.  
5. Use field moves or items for field interactions to remove any game-blocking obstacles.  
6. Complete the Team Rocket storyline.     
7. Beat the 6 required rival battles.
8. Beat the Elite 4 and Champion.

Or, if we want to be more detailed...

{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
  layout: elk
---
flowchart TD
    AA(Acquire starter Pokémon from Prof. Oak)
    AA --> AAA(Acquire Parcel from Viridian Mart) --> AAAA(Deliver Parcel to Prof. Oak)
    AAAA --> A(Defeat Brock) --> B(Traverse Mt. Moon)
    B --> C(Defeat Misty)
    B --> D(Nugget Bridge) --> E(Get the SS Anne ticket from Bill) --> F(Defeat Cerulean Rocket Grunt) --> G(Defeat Rival on the SS Anne) --> H(Obtain HM01 - CUT from the Captain on the SS Anne)
    C --> I
    C --> J
    C --> K
    C --> M
    H --> HH(Defeat Rocket Grunt protecting Rocket Hideout) --> HHH(Flip switch in Celadon Game Corner to unlock the Rocket Hideout)
    HHH --> KK(Defeat Rocket Grunt with Lift Key) --> KKK(Obtain the Lift Key) --> K(Defeat Giovanni in Rocket Hideout) 
    K --> LL(Collect the Silph Scope from Giovanni) --> LLL(Use the Silph Scope on the ghost in Lavender Tower) 
    LLL --> L(Save Mr. Fuji at the top floor of Lavender Tower) 
    H --> I(Defeat Lt. Surge)
    H --> J(Defeat Erika) 
    H --> MM(Obtain a drink from the vending machines at the top of Celadon Mart) 
    MM --> M(Deliver Drink to Saffron Guard) --> O
    L --> N(Obtain Pokéflute)
    L --> OO(Defeat Rival 5 in Silph Co) --> O(Defeat Giovanni in Silph Co) --> P(Defeat Sabrina)
    N --> Q(Use Pokéflute on at least one Snorlax)
    Q --> R(Obtain HM03 - SURF from the Safari Zone) --> T(Acquire the Secret Key from Pokémon Mansion) --> U(Defeat Blaine)
    Q --> V(Defeat Koga) --> T
    Q --> SS(Obtain the Gold Teeth from the Safari Zone) --> S(Deliver the Gold Teeth to the old man in Fuchsia City to acquire HM04 - STRENGTH)
    I --> W(Defeat Giovanni in Viridian Gym) --> X(Defeat Rival 6) --> Y(Traverse Victory Road)
    J --> W
    P --> W
    U --> W
    S --> Y
    Y --> Z(Defeat the Elite 4) --> ZZ(Defeat the Champion)
{{< /mermaid >}}

## Defining a "Route"

For the agent to complete all objectives, we wanted to simplify the game as much as possible to maximize the likelihood of success. To keep this work in line with Peter Whidden's video and to guarantee a Pokémon capable of learning `CUT`, we started all environments after the "Parcel Delivery" event. Additionally, we wanted to guarantee the agent would receive the gift Lapras in Silph Co. Here’s the updated route (simplified, with important changes in bold):

{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
  layout: elk
---
flowchart TD

START("**Start with Bulbasaur or Charmander to guarantee a Pokémon that can use CUT**")
BROCK("Defeat Brock")
MISTY("Defeat Misty")
HM01("Acquire HM01")
TEACH_CUT("**Teach the player's starter CUT**")
LTSURGE("Defeat Lt. Surge")
ERIKA("Defeat Erika")
KOGA("Defeat Koga")
SABRINA("Defeat Sabrina")
BLAINE("Defeat Blaine")
GIOVANNI("Defeat Giovanni")
HM03("Acquire HM03")
TEACH_SURF("**Teach Lapras SURF**")
HM04("Acquire HM04")
TEACH_STRENGTH("**Teach Lapras STRENGTH**")
TEAM_ROCKET("Complete Team Rocket Storyline")
LAPRAS("**Acquire the gift Lapras**")
RIVAL6("Defeat Rival 6")
E4("Defeat E4 and Champion")

START --> BROCK 
BROCK --> MISTY
MISTY --> TEACH_CUT
BROCK --> HM01 --> TEACH_CUT --> LTSURGE
MISTY --> LTSURGE
TEACH_CUT --> ERIKA
TEACH_CUT --> TEAM_ROCKET
TEAM_ROCKET --> LAPRAS --> TEACH_STRENGTH
LAPRAS --> TEACH_SURF
TEAM_ROCKET --> KOGA
TEAM_ROCKET --> SABRINA
TEAM_ROCKET --> HM04
TEAM_ROCKET --> HM03
KOGA --> TEACH_SURF
HM03 --> TEACH_SURF --> BLAINE
ERIKA --> TEACH_STRENGTH
HM04 --> TEACH_STRENGTH

LTSURGE --> GIOVANNI
SABRINA --> GIOVANNI
BLAINE --> GIOVANNI

GIOVANNI --> RIVAL6 
TEACH_STRENGTH --> E4
RIVAL6 --> E4


{{< /mermaid >}}

This route is *extremely* complex. The average human player will take 25 hours to beat Pokémon on the first try. Training takes between 7 hours and 1 week. 

### Risk Management

Even though we simplified the game, we still needed risk mitigation. Pokémon contains numerous game-ending risks in the environment including:

* Permanently losing vital Pokémon. 
* Catching Pokémon and not having enough space for the Lapras or any other Pokémon that can learn `SURF`.  
* If the agent obtains too many items, then there may not be enough room for key items.
* It is possible to soft-lock if you cannot obtain any more money at the time the Safari Zone objectives need to be completed.  
* Only having Pokémon with non-damaging moves.

We’d like to emphasize that *none* of these issues require teaching the agent how to battle.

### Scripting vs. Emergence

To handle these risks and difficulties, we split agent behavior into two classes:

* **Scripted:** Meaning the environment performs certain actions for the agent.  
* **Emergent:** Allowing the agent to discover its own strategies.

We really wanted to beat the game without any scripts. However, a few sections require human intuition not directly learnable from the game, e.g., reading the name of an HM and knowing immediately what it does. We aimed to only script behaviors that require human intuition or tasks that are not really a core part of Pokémon. Some of these scripts (especially ones related to `SURF` and `Pokéflute`) can probably be removed by running more ablations and better tuning reward weights and hyperparameters. These included:

* Item management - the environment will toss all non-key items if the player's bag is full.
* Money management - the environment provides infinite money.  
* Teaching `STRENGTH`
* Solving puzzles that require `STRENGTH`. 
* Teaching `SURF` and using `SURF` outside of battle.
* Automatically using Pokéflute.
* Blocking the Indigo Plateau exit at the end of the game.
* Using `FLASH`.

We additionally used scripts to ease development. These scripts have since been removed.

* Teaching `CUT`.
* Automatically using `CUT` outside of battle.
* Maxing the Pokémon’s stats.
* Automatic insertion of drinks into the player's bag if the player *entered* Celadon Mart.
* Automating elevator usage. If an agent entered an elevator, the elevator would go to the next floor modulo number of floors.
* Disabling wild battles.

## The Environment

The Python library [Gymnasium](https://gymnasium.farama.org/) provides a fairly straightforward API for defining an environment. The environment can be simplified into two functions: `Step` and `Reset`

## Step

`Step` is a function which is given actions for the environment and returns an observation, any logging info and whether or not the environment should reset.

Our step function for Pokémon can be written as:

- Receive a button press action.
- Send the button press to the Gameboy emulator.
- Wait some frames for the action to have an effect on the environment.
- Sample the environment.
- Return data based on the sample.

<div style="border:1px solid black;">
{{< highlight python >}}

def step(self, action: int):
    """
    A simplified version of the step function run during training
    """
    self.pyboy.send_input(VALID_ACTIONS[action])
    self.pyboy.send_input(VALID_RELEASE_ACTIONS[action], delay=8)
    self.pyboy.tick(self.action_freq - 1, render=False)
    return self.get_obs(), self.get_reward()
{{< /highlight >}}
</div>

## Reset, Episode and the Goal

Reset handles the initialization for an episode. An episode is a sequence of actions and states that ends when some terminal state is reached.

Resets usually occur when an environment:

- Achieves a major milestone, such as defeating a boss.
- Encounters a failure state, such as losing all lives.
- Reaches a predefined time or action limit, such as 100 tetriminos in Tetris.

Pokémon is in the realm of "long episodic RL." There is no strict rule on what a long episode is, but Pokémon takes 25 hours for the average person to beat. That's much longer than a session of Pac-Man or Breakout. Long episodes mean that the environment may not return rewards for a very long time. If the environment only returns rewards at the end of an episode, the agent may have trouble learning long term policies.

For example, if we were playing 100x100 Tic-Tac-Toe, it could take up to 5000 steps to return a rewards. Imagine having to plan out a 5000 step strategy. It's not easy! Later, we'll go over how we handled rewards for Pokémon's long episodes.

Based on Peter Whidden's prior work, we began with an episode terminating after a fixed number of steps. Over time, we tried other strategies around dynamically increasing the number of steps in an episode. We ultimately decided a Pokémon Red episode's terminal state is when an unrecoverable state (soft-lock) is met, e.g., such as running out of money or when the game is complete. 

Because our **goal** was focused on becoming the Champion (a very long task), we compromised. We created "mini-episodes.” An episode would be the duration of an entire game. Some of the environment's state would periodically reset mid-episode, but the emulator running Pokémon within the environment would not:

{{% details "Our reset function" closed %}}

<div style="border:1px solid black;">
{{< highlight python >}}

class OnResetExplorationWrapper(gym.Wrapper):
    """
    Wrapper for experimenting with different reset strategies. The OnResetExplorationWrapper,
    used in the majority of experiments, resets all memory every get_max_steps() steps + jitter.
    """
    def __init__(self, env: RedGymEnv, reward_config: DictConfig):
        super().__init__(env)
        self.full_reset_frequency = reward_config.full_reset_frequency
        self.jitter = reward_config.jitter
        self.counter = 0

    def step(self, action):
        if self.env.unwrapped.step_count >= self.env.unwrapped.get_max_steps():
            if (self.counter + random.randint(0, self.jitter)) >= self.full_reset_frequency:
                self.counter = 0
                self.env.unwrapped.explore_map *= 0
                self.env.unwrapped.reward_explore_map *= 0
                self.env.unwrapped.cut_explore_map *= 0
                self.env.unwrapped.cut_tiles.clear()
                self.env.unwrapped.seen_coords.clear()
                self.env.unwrapped.seen_map_ids *= 0
                self.env.unwrapped.seen_npcs.clear()
                self.env.unwrapped.valid_cut_coords.clear()
                self.env.unwrapped.invalid_cut_coords.clear()
                self.env.unwrapped.valid_pokeflute_coords.clear()
                self.env.unwrapped.invalid_pokeflute_coords.clear()
                self.env.unwrapped.pokeflute_tiles.clear()
                self.env.unwrapped.valid_surf_coords.clear()
                self.env.unwrapped.invalid_surf_coords.clear()
                self.env.unwrapped.surf_tiles.clear()
                self.env.unwrapped.seen_warps.clear()
                self.env.unwrapped.seen_hidden_objs.clear()
                self.env.unwrapped.seen_signs.clear()
                self.env.unwrapped.safari_zone_steps.update(
                    (k, 0) for k in self.env.unwrapped.safari_zone_steps.keys()
                )
            self.counter += 1
        return self.env.step(action)
{{< /highlight >}}
</div>

{{% /details %}}