+++
title = 'The "Route"'
weight = 26
+++

# Defining a "Route"

For the agent to complete all objectives, I wanted to simplify the number of game as much as possible to maximizing the likelihood of success. To limit non-determinism, I started the agent after the "Parcel Delivery" event. Starting the agent with a specific Pokemon would guarantee later stages of the game would be possible. Given the previous breakdown, here’s the *route* I wanted the agent to learn:

* Start with Bulbasaur or Charmander to guarantee a Pokemon who can use CUT. 
* Defeat Brock.
* In any order:  
  * Defeat Misty. 
  * Acquire HM01 and teach CUT to Bulbasaur.
* In almost any order:  
  * Defeat gyms 3-7.
  * Acquire HM03 and HM04.
  * Complete the Team Rocket Storyline.
  * Acquire the gift Lapras in Silph Co and teach the Lapras Surf and Strength.
* Defeat Gym 8.  
* Defeat the 6th rival battle.
* Defeat the Elite 4 + Champion.