+++
title = 'The Environment and Goal'
weight = 2
+++

# The Environment and Goal

The Python library [Gymnasium](https://gymnasium.farama.org/) provides a fairly straightforward API for defining an environment. The environment can be simplified into two functions: `Reset` and `Step`

## Steps

Step handles actions meant for the environment and returns observations based on a taken action, any logging info and whether or not to reset the environment.

Our step function for Pokémon can naively be written as:

- Receive a button press action
- Send the button press to the gameboy emulator
- Wait some frames for the action to have an effect on the environment
- Sample the environment
- Return data based on the sample

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

With a minimal environment defined, we could begin to think about what we wanted for 
observations and rewards. 

## Resets, Episodes and the Goal

Reset handles the initialization for a gameplay session (episode).

A reset will happen when an agent:

- Achieves a major milestone, such as defeating a boss.
- Encounters a failure state, such as losing all their lives.
- Reaches a predefined time or action limit, such as 100 tetriminos in Tetris,

Pokémon is in the realm of "long episodic RL" as it takes 25 hours for the average person to beat. Long episodes mean that the agent may not get large rewards for a very long time. If the agent only receives large rewards at the end of an episode, the agent may have trouble creating long term policies.

For example, if we were playing 100x100 Tic-Tac-Toe, the agent wouldn't receive a reward for up to a max 5000 steps. Imagine having to plan out a 5000 step strategy. It's not easy! Later, we'll go over how we handled rewards for Pokémon's long episodes.

Based on Peter Whidden's prior work, we began with an episode as a fixed number of steps. Over time, we tried other strategies such as dynamically increasing the number of steps per episode as the agent performed important milestones. However, we believe an episode *should be* based on a goal or when the agent achieves an unrecoverable state (soft-lock), e.g., such as running out of money. 

Because the **goal** was to be the Champion, we compromised. We created "mini-episodes.” An episode would be the duration of an entire game. The agent's state would periodically reset mid-episode, but the emulator state would not. 

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


