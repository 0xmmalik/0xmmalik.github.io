---
layout: post
title: "Bug in Maze Gen?"
date: 2021-01-10
blurb: Terraria Zeus Expansion Mod Devlog #2
---

##### So I realized there was a bug in the maze generation!

Remember the original while loop?

```csharp
while (frontier.Count > 0)
{
    List<int[]> poss = adjEmpty(maze, frontier[0]);
    frontier.Remove(frontier[0]);
    if (poss.Count == 0) continue;
    int num = rand.Next(1, poss.Count);
    for (int i = 0; i < num; i++)
    {
        int[] coord = poss[rand.Next(0, poss.Count)];
        if (adjEmpty(maze, coord).Count == 3)
        {
            maze[coord[0], coord[1]] = 1;
            frontier.Insert(0, coord);
        }
        poss.Remove(coord);
    }
}
```

The issue is this: `adjEmpty` returns some values that are not valid (as they are not adjacent to exactly 3 empty tiles). When the random number of tiles is selected, some of the tiles that are selected are not valid. Those tiles are skipped but still count towards the selected number of tiles. This is what was causing our mazes to be mostly blank. Now, I've added a small bit of code to remove the invalid values:

```csharp
List<int[]> toKill = new List<int[]>();
foreach (int[] coord in poss)
{
    if (adjEmpty(maze, coord).Count != 3)
    {
        toKill.Add(coord);
    }
}
foreach (int[] victim in toKill) poss.Remove(victim);
```

As a result of this addition, there is no reason to keep generating mazes until 42% of them are open because this new code will make sure that every maze is sufficiently full.
