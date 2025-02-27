+++
title = 'Intro'
weight = 20
+++

# Breaking Down Pokémon

Before going into observations, rewards and the policy, I believe studying the environment is higher priority. As I already said multiple times, Pokémon is a complex game with multiple tasks and puzzles that can be accomplished nonlinearly.

Once we have an understanding of the game and all its gotchas, we can engineer observations and rewards to accomplish all tasks.

To begin, I enumerated the game’s storyline objectives. These objectives may be a little more detailed than what you would read in an average walkthrough, but it was important to cover what’s required or risk the agent getting stuck. At a high level, beat Pokémon, you must:

1. Beat the 8 Gym Leaders. Gym leaders are a form of video game “boss.”  
2. Acquire items to teach the moves CUT, STRENGTH, and, SURF. Field moves are abilities that can be used outside of battle to unlock a new area or make it easier to traverse an existing area.  
3. Acquire Pokémon that can learn the field moves CUT, STRENGTH, and SURF. 
4. Acquire any items required for field interactions. Like field moves, there are items that are required to unlock new areas.  
5. Teach available Pokémon the field moves CUT, STRENGTH, and SURF   
6. Use field moves or items for field interactions to remove any game blocking obstacles.  
7. Complete the Team Rocket storyline.     
8. Beat the 6 required rival battles  
9. Beat the Elite 4 and Champion

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
    B --> D(Nugget Bridge) --> E(Get the SS Anne ticket from Bill) --> F(Defeat Cerulean Rocket Grunt) --> G(Defeat Rival on the SS Anne) --> H(Obtain HM01 - Cut from the Captain on the SS Anne)
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
    Q --> R(Obtain HM03 - Surf from the Safari Zone) --> T(Acquire the Secret Key from Pokémon Mansion) --> U(Defeat Blaine)
    Q --> V(Defeat Koga) --> T
    Q --> SS(Obtain the Gold Teeth from the Safari Zone) --> S(Deliver the Gold Teeth to the old man in Fuchsia City to acquire HM04 - Strength)
    I --> W(Defeat Giovanni in Viridian Gym) --> X(Defeat Rival 6) --> Y(Traverse Victory Road)
    J --> W
    P --> W
    U --> W
    S --> Y
    Y --> Z(Defeat the Elite 4) --> ZZ(Defeat the Champion)
{{< /mermaid >}}


I broke down these objectives to understand what is and is not important for beating Pokémon Red.