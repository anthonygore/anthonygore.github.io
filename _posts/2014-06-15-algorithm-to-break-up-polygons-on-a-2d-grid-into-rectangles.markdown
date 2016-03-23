---
layout: post
title:  "Algorithm To Break Up Polygons On A 2D Grid Into Rectangles"
date:   2014-06-15 00:00:00 +1100
tags: corona, game_development, lua
permalink: /algorithm-to-break-up-polygons-on-a-2d-grid-into-rectangles
---

First read my post [Isolating distinct, arbitrary shapes on a 2D grid](/isolating-distinct-arbitrary-shapes-on-a-2d-grid).

In this post I’ll explain how to break up polygons on a 2D grid into distinct rectangles. This is useful, for example, if you’re aiming to add shapes to Box2D physics engine, as convex shapes cannot be used in Box2D. The easiest way to eliminate concavity is slicing your polygons into smaller rectangles.

Here’s the 2D grid from the previous post, with two polygonal shapes (n = 2) distinguished on the grid:

    {0,0,0,0,0,0}
    {0,1,1,0,0,0}
    {0,1,1,0,0,2}
    {1,1,0,2,0,2}
    {0,0,0,2,0,2}
    {0,0,2,2,2,2}

This algorithm requires you to iterate the grid left-to-right, top-to-bottom. You’ll need to know the how many distinct shapes are already present on the grid (i.e. n = 2, in this example).

The method is to find how long the first row of the shape r<sub>1</sub> is, assign each cell in r<sub>1</sub> with the value v<sub>x,y</sub> = n, and then copy down r<sub>1</sub> to rows r<sub>j</sub>,  1 < j < h – j (h = height of grid) below, as many times as it will continue to fit within the shape.

For example:

    {0,0,0,0}
    {0,1,1,0} <-- r1
    {0,1,1,0}
    {1,1,0,0}

r<sub>1</sub> of the shape currently with v = 1, contains cells (x<sub>2</sub>,y<sub>2</sub>) and (x<sub>3</sub>,y<sub>2</sub>) and it’s width is 2. Give those cells the value n++ (in this case, 2). r<sub>1</sub> can be copied down to r<sub>2</sub> successfully, but not r<sub>3</sub>. This is because r<sub>3</sub> has the same width as r<sub>1</sub>, but it’s left-most cell is obviously not in the same column as r<sub>1</sub>’s left-most cell.

So the algorithm would identify the first two rows as part of one rectangle, and further on would distinguish the third row as another rectangle, assigning it’s cells the value n++:

    {0,0,0,0}
    {0,2,2,0}
    {0,2,2,0}
    {3,3,0,0}

To return to the original example, for each cell (x<sub>i</sub>, y<sub>j</sub>) with a value vi where i > 0, and v < n, find out how long it’s row is by checking each tile to it’s right i.e. x<sub>i+k</sub> where k is iterated between k = 1 and k = w – x (the width of the grid – x). Stop when you reach a cell where v<sub>i+k</sub> != v<sub>i</sub>. The length of the row r<sub>i</sub> will then be l = j-i. Now, iterate y<sub>i+k</sub>, where 1 < k < h – y (where h is the height of the grid). Continue until you can no longer fit l tiles into the row r<sub>i+k</sub>. The number of rows you can copy down will be m = k – i.

Your identified rectangle will be of size l x m, starting at point (x<sub>i</sub>, y<sub>j</sub>). Each cell in that rectangle is now assigned the value v<sub>x,y</sub> = n. Once you’ve done that, increment n and continue running the algorithm.

Once you’ve finished you’ll end up with this:

    {0,0,0,0,0,0}
    {0,3,3,0,0,0}
    {0,3,3,0,0,4}
    {5,5,0,6,0,4}
    {0,0,0,6,0,4}
    {0,0,7,6,8,4}

You can then “normalise” the grid the so the lowest shape number is 1:

    {0,0,0,0,0,0}
    {0,1,1,0,0,0}
    {0,1,1,0,0,2}
    {3,3,0,4,0,2}
    {0,0,0,4,0,2}
    {0,0,5,4,6,2}

The algorithm has divided the original 2 polygonal shapes into 6 rectangles.

Here’s a Lua implementation of the algorithm:

    -- grid is the 2D grid to input
    -- n is the number of distinct polygons on the grid
    local function createRectObjects (grid, n)
        local max = n
        local objNum = max
         
        -- Iterate grid
        for y = 1, #grid, 1 do
            for x = 1, #grid[y], 1 do
                 
                --Subject tile
                local subject = {}
                subject.val = grid[y][x]
                subject.x = x
                subject.y = y
                 
                if subject.val > 0 and subject.val <= max then
                    objNum = objNum + 1
                    --grid[y][x] = objNum
                     
                    --Determine width
                    local blockWidth = 1
                    for i = 1, #grid[y] - x, 1 do
                        if grid[y][x+i] == subject.val then
                            blockWidth = blockWidth + 1
                        else
                            break
                        end
                    end
                     
                    --Determine height
                    local blockHeight = 1
                    for i = 1, #grid - y, 1 do
                        local invalidRow = false
                        for j = 1, blockWidth, 1 do
                            if grid[y+i][x+j-1] ~= subject.val then
                                invalidRow = true
                                break
                            end
                        end
                        if invalidRow then
                            blockHeight = i
                            break
                        elseif i == (#grid - y) then
                            blockHeight = i
                            break
                        end
                    end
                     
                    --Stamp the new objNum onto the tiles in the block
                    for i = subject.y, subject.y + blockHeight - 1, 1 do
                        for j = subject.x, subject.x + blockWidth -1, 1 do
                            grid[i][j] = objNum
                        end
                    end
                end
            end
        end
         
        --Normalise numbers
        for y = 1, #grid, 1 do
            for x = 1, #grid[y], 1 do
                if grid[y][x] ~= 0 then
                    grid[y][x] = grid[y][x] - max
                end
            end
        end
         
        obj.max = objNum - max
        return grid
    end

This code is part of my Lua Grid Tools repo on Bitbucket: [https://bitbucket.org/anthonygore/luagridtools](https://bitbucket.org/anthonygore/luagridtools)

Next I’ll explain how to trace these shapes and return a table of points which outline each rectangle. Tools like Box2D take an array of vertices.
