---
layout: post
title:  "2D Top-Down Game: Movement Algorithm (Part 1)"
date:   2014-01-10 00:00:00 +1100
tags: game_development, java, algorithms
permalink: /movement-algorithm-for-player/
------------------------------------------

In this game, the player will move to a location on the screen determined by user input. The following are constraints that will need to be considered when constructing an algorithm:

1. The player can only face in eight directions i.e. the cardinal and ordinal compass directions (N, NE, E, SE etc). This limitation is necessary is to keep the graphics simple. As such, the player can only move in those eight directions or the movement animation will look weird.
2. The player’s destination can be changed mid-walk i.e. the user is free to set the destination as often as they like, and the player must respond accordingly.
3. The player can only move on “walkable” tiles i.e. he can’t walk over buildings. He needs to walk around unwalkable tiles.

Each of these constraints bring up a whole lot of sub-constraints and necessary decisions and compromises. I’ll deal with these individually.
