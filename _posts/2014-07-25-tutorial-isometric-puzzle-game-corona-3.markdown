---
layout: post
title:  "Tutorial: Isometric (2.5D) Puzzle Game With Corona (Part 3)"
date:   2014-07-25 00:02:00 +1100
tags: game_development, corona, lua
permalink: /tutorial-isometric-2-5d-puzzle-game-with-corona-part-3
---

This tutorial assumes knowledge of:

* Corona SDK
* Tiled Map Editor

Here’s a short video of the game:

[![Corona isometric puzzle game](http://img.youtube.com/vi/C_JbHhReU6o/0.jpg)](http://www.youtube.com/watch?v=C_JbHhReU6o "Corona isometric puzzle game")

In [part 2](/tutorial-isometric-2-5d-puzzle-game-with-corona-part-2) of the tutorial I showed how to load your map into a Corona project. In part 3 and 4 I’ll show you some elements of the game logic. I’m going to restrict the discussion to aspects of the game logic that are affected by the isometric perspective of the map.

Let’s firstly look at how to move the blocks on the isometric map. We’ll then discuss display ordering.

Here’s the function that handles block movement. In a nutshell, you provide the function with the angle the D-pad touch listener calculated, and which block was selected. The function will figure out where the block should end up (given the positions of the other blocks on the map), then transition the block from it’s current position to the new position.

    --Moves a block. angle is the movement angle (sent from the D-pad)
    --i is the selected block
    function moveBlock(angle, i)
        --i.e. if a block is indeed selected
        if i > 0 then
            --Variables
            local deltaX, deltaY, newTilePosX, newTilePosY, iTileX, iTileY,
            moveScreenPosX, moveScreenPosY, newScreenPosX, newScreenPosY,
            deltaTilePosX, deltaTilePosY, timeScale
     
            --Arrays
            local jArr = {}
     
            --Only move if the block is not already moving
            --You don't want to interrupt a current movement
            if not block[i].moving then
     
                --Get directional change
                --angleToDir function not shown, it returns the change
                --in x and y axis represented by the angle argument
                deltaX, deltaY = angleToDir(angle) 
     
                --Get tile position of current block
                iTileX, iTileY = block[i].tileX, block[i].tileY
     
                --Get tile position of the other blocks
                for j = 1, #block do
                    if j ~= i then
                        table.insert(jArr, { tileX = block[j].tileX,
                        tileY = block[j].tileY})
                    end
                end
     
                --Calculate new block position
                --newBlockPos (not shown) will calcalaute the new tile position
                --of block i, given the positions of the other blocks
                newTilePosX, newTilePosY = newBlockPos(iTileX, iTileY, jArr,
                deltaX, deltaY)
     
                --Change in block position
                deltaTilePosX, deltaTilePosY = newTilePosX - iTileX,
                newTilePosY - iTileY 
     
                --See if new position is viable i.e. on the map
                if (newTilePosX >= 0 and newTilePosX < (mapWidth - 1)) and
                (newTilePosY >= 0 and newTilePosY < (mapHeight - 1)) then                             --Update tile position
                    block[i].tileX, block[i].tileY = newTilePosX, newTilePosY                         --Calculate new screen position of block i
                    moveScreenPosX, moveScreenPosY = (deltaTilePosX - deltaTilePosY) *                tileWidth/2, (deltaTilePosX + deltaTilePosY) * tileHeight/2                       newScreenPosX, newScreenPosY = block[i].x + moveScreenPosX,
                    block[i].y + moveScreenPosY
                    --Set the block to moving
                    block[i].moving = true
                    --Update screen position with a transition
                    local function listener (event)
                        block[i].moving = false
                        if moveScreenPosX == 0 and moveScreenPosY == 0 then
                        else
                            audio.play(blockClick)
                        end
                        --Check if the game's been won
                        checkWon()
                    end
                    if math.abs(deltaX) > math.abs(deltaY) then
                        timeScale = math.abs(deltaTilePosX)
                    else
                        timeScale = math.abs(deltaTilePosY)
                    end
     
                    transition.to(block[i], {time=timeScale*100,
                    transition=easing.outQuad, x = newScreenPosX,
                    y = newScreenPosY, onComplete = listener}
                    )
                end
            end
        end
    end

In the next tutorial we’ll look at how to re-order the display objects, particularly as they pass each other.
