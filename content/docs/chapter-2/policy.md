+++
title = 'Policy'
weight = 33
+++

# RL Algorithm and Policy

For training, I adopted the [on-policy](https://towardsdatascience.com/on-policy-v-s-off-policy-learning-75089916bc2f/) algorithm [Proximal Policy Optimization](https://en.wikipedia.org/wiki/Proximal_policy_optimization) (PPO). PPO supports vectorized environments well and there is a large amount of online code examples surrounding it.

I tried to keep the policy itself simple. I wanted a policy that 

- Could have some way of processing sequential (time) data.  
- Was small for faster training.

To handle time dependence I considered a few options:

- Modify my observation to stack frames. Each frame would linearly increase the model’s input. Then use some form of model that handles batches of sequential data, e.g., a 3D [CNN](https://en.wikipedia.org/wiki/Convolutional_neural_network) or [attention](https://en.wikipedia.org/wiki/Attention_(machine_learning)) layer. These models would have heavily slowed down training and require more VRAM than I had at the time  
- Use a [recurrent neural network](https://en.wikipedia.org/wiki/Recurrent_neural_network) such as an [LSTM](https://en.wikipedia.org/wiki/Long_short-term_memory).   
- Use a [state space model](https://huggingface.co/blog/lbourdois/get-on-the-ssm-train) (SSM).

I went with the easiest to integrate solution, an LSTM. An LSTM contains an internal state that gets input to the model alongside the most recent data point. Although LSTMs can only remember up to ≈1000 steps, that was enough history for an effective policy.

The policy ended up being ≈10 million parameters or 40MB. For context, that's 5 orders of magnitude smaller than DeepSeek. Enough to fit on the average consumer GPU.

## Feature Engineering the Policy

I mentioned the observation I input to the policy, but did not mention its members’ *data types*. As I’ll explain later, I wanted to keep the size (in bytes) of the observation as small as possible. I also needed to transform the observation into a form the model dictating the policy could use for optimization.

### Pokémon Red Policy

{{% details "Pokémon Red Policy" open %}}
{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
---
flowchart LR
        Party_Network2["Party Network"]
        FinalConcat("Concat")
        Screen_Network2["Screen Network"]
        MapIDE("Map ID Embeddings")
        MapID("Map ID")
        MapIDE2("Map ID Embeddings")
        BlackoutMapId("Blackout Map ID")
        Mul("x")
        ItemIDE("Item ID Embeddings")
        BagItemIds("Bag Item IDs")
        div100("/ 100")
        BagItemQ("Bag Item Quantities")
        Events("Events Completed Array")
        OneHot("One-hot")
        Direction("Direction")
        OneHot2("One-hot")
        BattleT("Battle Type")
        MissingEvents2("Missing Events")
        div502("/ 502.0")
        SafariZoneSteps("Safari Zone Steps Remaining")
        LSTM("LSTM")
        Linear3("Linear")
    Party_Network2 --> FinalConcat
    Screen_Network2 --> FinalConcat
    MapID --> MapIDE
    MapIDE --> FinalConcat
    BlackoutMapId --> MapIDE2
    MapIDE2 --> FinalConcat
    BagItemIds --> ItemIDE
    ItemIDE --> Mul
    BagItemQ --> div100
    div100 --> Mul
    Mul --> FinalConcat
    Events --> FinalConcat
    Direction --> OneHot
    OneHot --> FinalConcat
    BattleT --> OneHot2
    OneHot2 --> FinalConcat
    MissingEvents2 --> FinalConcat
    SafariZoneSteps --> div502
    div502 --> FinalConcat
    FinalConcat --> Linear3
    Linear3 --> LSTM
{{< /mermaid >}}
{{% /details %}}

{{% details "Party Network" closed %}}
{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
---
flowchart LR
        Concat("Concat")
        SIDE("Species Embeddings")
        SID("Species ID")
        Hp("HP")
        Status("Status")
        t1e("Type Embeddings")
        t1("Type 1")
        t2e("Type Embeddings")
        t2("Type 2")
        level("Level")
        MaxHp("Max HP")
        Attack("Attack")
        Defense("Defense")
        Special("Special")
        MovesE("Moves Embeddings")
        Moves("Moves")
        Flatten1("Flatten")
        Linear2("Linear")
        Linear1("Linear")
    SID --> SIDE
    SIDE --> Concat
    Hp --> Concat
    Status --> Concat
    t1 --> t1e
    t1e --> Concat
    t2 --> t2e
    t2e --> Concat
    level --> Concat
    MaxHp --> Concat
    Attack --> Concat
    Defense --> Concat
    Special --> Concat
    Moves --> MovesE
    MovesE --> Concat
    Concat --> Linear1
    Linear1 --> Linear2
    Linear2 --> Flatten1
{{< /mermaid >}}
{{% /details %}}

{{% details "Screen Network" closed %}}
{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
---
flowchart LR
        Concat2("Concat")
        gamescreen("Game Screen<br>72x80x1<br>grayscale")
        visitedmask("Visited Mask<br>72x80")
        relu1("ReLU")
        Conv1("2D CNN<br>72x80x2")
        relu2("ReLU")
        Conv2("2D CNN<br>72x80x32")
        relu3("ReLU")
        Conv3("2D CNN<br>36x40x64")
        relu4("ReLU")
        Conv4("2D CNN<br>18x20x64")
        Flatten2("Flatten")
    gamescreen --> Concat2
    visitedmask --> Concat2
    Concat2 --> Conv1
    Conv1 --> relu1
    relu1 --> Conv2
    Conv2 --> relu2
    relu2 --> Conv3
    Conv3 --> relu3
    relu3 --> Conv4
    Conv4 --> relu4
    relu4 --> Flatten2
{{< /mermaid >}}
{{% /details %}}

{{% details "Missing Events" closed %}}
{{< mermaid >}}
---
config:
  theme: mc
  look: handDrawn
---
flowchart TB
        Rival3("Rival 3 Defeated Boolean")
        Lapras("Lapras Acquired Boolean")
        SaffronGuard("Drink Given Saffron Guard Boolean")
        GameCornerRocket("Game Corner Rocket")
{{< /mermaid >}}
{{% /details %}}


Let’s summarize what the observation consists of:

|           Observation            |  Shape  | Data Type |
| :------------------------------: | :-----: | :-------: |
|              Screen              | 72x80x1 |    int    |
|           Visited Mask           | 72x80x1 |    int    |
|              Map ID              |   1x1   |    int    |
|         Blackout Map ID          |   1x1   |    int    |
|             Item IDs             |  20x1   |    int    |
|         Item Quantities          |  20x1   |    int    |
|           Agent Party            |  6x11   |    int    |
|           Events Array           | 1x2560  |  boolean  |
|            Direction             |   1x1   |    int    |
|     Current Battle Condition     |   1x1   |    int    |
|         Rival 3 defeated         |   1x1   |  boolean  |
|         Lapras Acquired          |   1x1   |  boolean  |
|          Saffron Guard           |   1x1   |  boolean  |
|   Game Corner Rocket Defeated    |   1x1   |  boolean  |
| Number of Safari Steps Remaining |   1x1   |    int    |

In the policy, I occasionally normalize the value by some constant. In machine learning, it is generally adviseable represent these numbers as values between 0 and 1 for model stability.

## The CNN
The screen obs and visited mask were concatenated together to make 2 visual “channels”. These channels were passed to a 2D Convolutional Neural Network (CNN). The kernel sizes of the CNN were designed with the GameBoy's in mind.   

The first layer of the CNN used an 8x8 kernel mapping to the size of a gameboy tile with a stride of size 2 so inter-tile dependencies could be detected. In successive layers, I decreased the kernel size so as not to capture too much information and similarly stride by 2 each layer so that the agent can hopefully get a good sense for edge detection.

## One-Hot encoding
One hot encoding is a convenient technique to take a value representing a category and map it to a representation useful for the model. It's useful when the number of categories is low.

Of the observations, direction, battle state (in-battle, wild battle, trainer battle) are represented as one-hot encoded values.

## Embeddings
The map ID and blackout map IDs are passed to the policy as integers, but in the policy, I transform them them with an embedding layer. Embedding layers are a convenient way to handle decreasing the number of dimensions of a categorical input. Instead of a one hot encoding the map id with \# map id channels, I only require 5 floats to accurately represent the map id space. I chose 5 based on a recommendation from [Google's Machine Learning Crash Course](https://developers.google.com/machine-learning/crash-course). Google recommends using # of rows^.25 for the number of dimensions in the embedding layer.

Items held in the agent's bag are also identified by ID. The Item IDs are passed to their own embedding layer. I scale the item embeddings by the item's quantities; a number between 0 and 1 where 0 maps to 0 and 1 maps the max number of the same item an agent can have.

## Party Network
All party data is concatenated together and passed through a small dense layer to create a “Pokémon” space.

## Binary Vectors
In RAM, events are a packed binary vector with each bit representing one in-game completed event. I unpack this vector and pass it on to the policy. The event vector in RAM does not include Lapras, Rival 3 and the Saffron Guard. I pass these in separately as they are "event"-like in my opinion.

## Safari Steps
The num of steps left in the Safari Zone is in the range [0, 502]. I normalize the steps observation to a value between 0 and 1 where 0 means no steps are left and 1 means you have the max number of steps remaining.

## Final Model Layers

Once all features have been transformed, all non-batch dimension data is flattened, concatenated and passed through a final linear layer before heading to the LSTM. The LSTM is an off the shelf component and not worth discussing. 