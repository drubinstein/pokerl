+++
title = 'Intro'
weight = 20
+++

# Breaking Down Pokémon

Before discussing observations, rewards and the policy we will do a deep dive into the environment. Pokémon is a complex game with multiple tasks and puzzles that can be accomplished nonlinearly. If you're already well-versed on Pokémon, 
we recommend you skip to the [next chapter]({{<ref "/docs/chapter-2/observations/index" >}} "next chapter").

To begin the breakdown, we enumerated the game’s storyline objectives. These objectives may be a little more detailed than what you would read in an average game summary, but there's a lot of hidden bottlenecks in Pokémon. At a high level, to beat Pokémon, you must:

1. Beat the 8 Gym Leaders (video game boss).  
2. Acquire `HMs` (items) to teach the field moves `CUT`, `STRENGTH`, and, `SURF` and Pokémon to teach the field moves to. Field moves are abilities that can be used outside of battle to unlock a new area or make it easier to traverse an existing area.  
3. Teach capable Pokémon the field moves `CUT`, `STRENGTH`, and `SURF`.
4. Acquire any items (not HMs) required for field interactions. Like field moves, there are items that are required to unlock new areas.  
5. Use field moves or items for field interactions to remove any game blocking obstacles.  
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
    H --> HH(Defeat Rocket grunt protecting Rocket Hideout) --> HHH(Flip switch in Celadon Game Corner to unlock the Rocket Hideout)
    HHH --> KK(Defeat Rocket grunt with Lift Key) --> KKK(Obtain the Lift Key) --> K(Defeat Giovanni in Rocket Hideout) 
    K --> LL(Collect the Silph Scope from Giovanni) --> LLL(Use the Silph Scope on the ghost in Lavender Tower) 
    LLL --> L(Save Mr. Fuji at the top floor of Lavender Tower) 
    H --> I(Defeat Lt. Surge)
    H --> J(Defeat Erika) 
    H --> MM(Obtain a drink from the vendeing machines at the top of Celadon Mart) 
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