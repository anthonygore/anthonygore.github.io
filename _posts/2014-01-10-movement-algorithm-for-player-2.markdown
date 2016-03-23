---
layout: post
title:  "2D Top-Down Game: Movement Algorithm (Part 2)"
date:   2014-01-10 00:00:01 +1100
tags: game_development, java, algorithms
permalink: /2d-top-down-game-movement-algorithm-2/
--------------------------------------------------

The player can only face in eight directions i.e. the cardinal (N, E, S, W) and ordinal (NE, SE, SW, NW) directions. This constraint is in response to the graphic art requirements; it can’t be expected that there will be animation in all possible directions. My assertion is that these eight directions will be enough, as they are for many top-down games.

To make a suitable algorithm, my idea is to break a proposed movement down into components which are directed in the eight such directions. For example, let’s say the user selects a destination about 10 steps away, slightly north of east. The player can’t move directly to that point, but you could instead make him move east for, say, 8 steps, and then north-east for, say, 2 steps and have him land on the required point.

The algorithm I designed to do this works as follows:

1. The user selects a point destination _d_ they want the player to move to. Now imagine a single vector is drawn between the point current location _c_ and _d_.
2. This single vector is now broken down into a list of component vectors _s_, the direction of each necessarily being one of the cardinal or ordinal compass directions, while the magnitude of each will be roughly the same (to ensure movement appears regular). The method profile will be something like:

        private LinkedList<Vector> buildVectorList (Point c, Point d)
        
    Now elect a standard length _l_ for these component vectors. _l_ shouldn’t be too small, otherwise the player’s movement will look abrupt, nor should it be too long, as he will deviate too far from the track. Let’s say _l = 20_. A method is also needed to measure the angle between _c_ and _d_. I found it best to measure the angle in radians, with the y-axis being parallel to north on the map, as you’d expect. With that in mind, here are the sub-steps for creating _s_:

    1. Check that the distance between _c_ and _d_ is greater than _l_. If so proceed, if not, end.
    2. Get the angle _a_ in radians between between _c_ and _d_. Let’s say _a_ is slightly larger than 0.
    3. Now find the compass direction that gives the closest approximation to _a_. In this example, it will be east, as east is equivalent to 0 radians on a cartesian plane.
    4. Now create a vector _v_ and add it to your list (I think a linked list is the best data structure here, but an array list or similar would be fine). The magnitude will be _l_, and the direction will be east.
    5. Now create a point _c1 = c + l_ so the _c1_ sits at the end of _v_, were _v_ to be extended from _c_. Let _c = c1_ and discard _c1_.
    6. Repeat the above steps until _c_ is within _l_ from the _d_.

3. Now if you iterate _s_ and follow the vectors sequentially tail to head, you will get the player to point _d1_ which approximates _d_. Below is an example.

    ![Vector](/assets/img/2014-01-10-vector.jpg)
    
    To get the player exactly to _d_, you need another method:
          
        private LinkedList<Vector> adjustVectors (LinkedList<Vector> s, Point c, Point d)
             
   1. Firstly, check the offset _o_ between _d_ and _d1_. Let’s say _d1_ is 10 pixels right and 10 pixels below _d_
   2. Iterate _s_ and distribute _o_ among the vectors _vi_ by changing their magnitude appropriately until _d1 == d_. I do this with a while loop, adding or subtracting 1 to the length of _vi_ until _o == 0_.

Finally we end up with a list of vectors which, if the player follows sequentially, will take him in a smooth path to the user’s elected destination.
