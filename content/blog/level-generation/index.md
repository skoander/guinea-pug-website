---
title: "Level Generation"
date: 2020-07-02T23:54:32+02:00
author: "Evert van Nieuwenburg"
draft: false
layout: "post"
mathjax: true
---

### 1\. Intro
At the end of March this year, we spontaneously decided to participate in the 46th edition of Ludum Dare. If you’ve never heard of LD, you should know that it is one of the best-known (bi-)annual game jams. In a game jam, you are given X amount of time (in our case, 72 hours) to create a game from scratch. So in April 2020 -- we did: meet Guinea Pug.

Guinea Pug is an “endless runner” game, with a procedurally generated level that has obstacles and special effects. It’s more of what I like to call an “endless hopper”, where the player controls a ragdoll pug suspended in a stasis field (yes, you read that right) and is tasked with hopping around on tiles whilst avoiding holes and gaps.

To give the player a fair chance however, those gaps shouldn’t be too large. And with random generation of tiles in a grid, there is a chance that too-large gaps occur. Now one might think that these are rare enough and the player would eventually ‘lose’ anyway, but this would be incredibly frustrating if it were to happen when you *finally* get far into the level. So that brings us to the topic of this post: how to generate playable, grid-based levels with maximum gap sizes.

### 2\. The Basics
The levels in Guinea Pug are generated as multiple segments, and each segment is a lattice of some size. So for sake of simplicity, let’s fix that size so that each segment is a 7x14 grid (7 tiles wide and 14 tiles long). A segment starts out empty, and then gets populated with tiles on which the player can hop. In practice we use a function GenerateRandomRow() for that, which for each row randomly selects the number of non-empty tiles and their locations. Figure 1 below demonstrates what such a randomly filled segment looks like.

{{< figure src="FullyRandom.png#center" height=400 caption="Figure 1: Starting with an entirely empty segment (left) of size 7x14, generating a random filling of tiles could result in the fully random segment shown on the right. For each of the 14 rows, a function GenerateRandomRow() determines how many non-empty tiles the row will have and then randomly selects those locations in the row." >}}


The player in Guinea Pug has controls that allow it to hop over gaps of size 1, but gaps of size 2 are only doable once enough speed has been built up. Gaps of size 3 are impossible, and the game therefore requires skill on the player’s side to quickly plan ahead and avoid such routes that involve gaps of size 3.

So, to prevent such impossible gaps we came up with two types of segments: 1) randomly-generated-and-then-patched-up and 2) guaranteed-paths-available. Since the second type is a bit easier, let’s start with those.

### 3\. Path-based segments
The idea behind path-based segments is that we can guarantee that a path is always available. This is actually quite simple, and here is a bit of pseudo code to show that off. There are some caveats to this, and I’ll do my best to mention them in the text.

{{< highlight go "linenos=table" >}}
func GenerateRandomPath()
{
    // Generate an array of size width-by-length and fill it with empty tiles
    segment[ width, length ] = 0

	  // Start at the first row, and at a random column
    currentPosition = (random_tile_in_x, row_0)

	  // Keep taking random steps until we reach the end of the segment!
    while (currentDistance < length)
    {
        // Set tile
        tiles[ currentPosition.x, currentPosition.y ] = 1;

        // Pick a random direction (E, NE, N, NW or W)
	      // It is here we could make sure that on average NE, N, NW are more likely
        Direction dir = PickRandomForwardDirection();

	      // In practice, we should check here if the new position is within our grid!
        currentPosition += dir;

	      // Only increase the distance if we took a forward step
        if (d == NW || d == N || d == NE)
            currentDistance += 1;
    }
}
{{< / highlight >}}

Let’s quickly go over the essence of this code. We start at a random location in the first row (i.e. row 1 out of 14), and take size-1 steps in the forward, left, right or forward diagonal directions. Now, we could keep track of which direction we took a step in so that in the next step we don’t backtrack to the previous location. Incidentally though, as long as we don’t take steps backwards, re-visiting tiles by moving left and right (or W and E) is actually fine and results in wider paths overall. We just have to make sure that on average you move forward so that you reach the end of the segment. Figure 2 shows such a randomly generated path.

{{< figure src="OnePath.png#center" height=400 caption="Figure 2: A single random path, generated through a simple random walk with an average forward direction per step (and never a step backwards, although left&right backtracking is allowed).">}}

All-in-all this is just a directed random walk, and various extensions are possible. For Guinea Pug however, we kept to the basics and instead complete these types of segments by generating multiple paths, so that there isn’t just a single strand to follow. This was important for us because we’re going to have special tiles like boosts that can boost the player off of the track. Having a single strand that happens to have a few boost tiles one after the other becomes unfair again. A few examples of completed segments of this type are shown in Figure 3.

{{< figure src="RandomPaths.png#center" height=400 caption="Figure 3: Random segments generated using the path-based method described in the article. Depending on the difficulty, and also on the size of the segment of course, the number of random paths can be adjusted.">}}

### 4\. Random segments with patched-up holes
The other type of segments in Guinea Pug is initially generated entirely randomly like in Fig 1. What is left then is to make sure no impossible gaps are present. We’ve concluded that gaps of size 3 are impossible with the current player mechanic, and that gaps of size 2 are possible with enough speed. For playability and to be able to make things easier based on difficulty settings, we use a variable minimal_gap_size to generate segments with denser or less dense tile coverings. We could for example set the minimal_gap_size to $\sqrt{2}$, so that diagonal gaps of 1 tile are the largest that occur.

To fix those gaps, we have to scan our segment and for every tile deduce what the shortest gap to another tile in the forward direction is. If this gap is larger than the minimal, we will have to add a tile in a location that makes this minimal gap at most minimal_gap_size. The pseudo-code for how we do that would be something like this:

{{< highlight go "linenos=table" >}}
func FixGaps()
{
    for each row:
        for each non-empty tile in this row:
            float distance = DistanceToNearestTile(row, tile);

            // Add a tile if required
            if (distance > minimal_gap_size) {
		             // This next bit is extremely simplified; in practise you would have
		             // to check if the new location (tile+dir) is within the grid, for example
                 int dir = PickRandomDirection();
                 segment[tile + dir, row + 1] = 1;
            }
}
{{< / highlight >}}

In case you are interested, the DistanceToNearestTile() function is also fairly straightforward:

{{< highlight go "linenos=table" >}}
func DistanceToNearestTile(int r, int t)
{
    // Check tile distances in the forward direction
    array distances = empty_array();

    for the next 3 rows:
        // Can't check beyond the segment
        if (row >= length)
            continue;

    for each non-empty tile in the row:
        // The variable t is the x-coordinate of the tile
		    // we are currently checking, and r is its row. The variables tile and row are
		    // the ones we are looping over.
        float distance = Mathf.Sqrt((tile - t) * (tile - t) + (row - r) * (row - r));
        distances.Add(distance);

    distances.Sort();

    // If there are no tiles within the next few rows, the distance is infinite
    // for our practical purposes
    if (distances.Count == 0)
        return Mathf.Infinity;

    // Otherwise return the shortest distance
    return distances[0];
}
{{< / highlight >}}

Finally, the fixed example of Fig 1, with a minimal_gap_size set to $\sqrt{2}$ is shown next in in Fig 4.

{{< figure src="PatchedRandom.png#center" height=400 caption="Figure 4: The random segment from Figure 1, but now with gaps larger than $\sqrt{2}$ filled with new tiles. This way, there is a guaranteed path for the player with gaps that are never larger than $\sqrt{2}$.">}}



### 5\. Bonus: Boost placement
Now that we have segments, we can spend a few minutes describing how we place boosts at locations that make sense. Boost placement is done using our ‘special-tile-distributor’, which is our fancy name for a pattern recognizer. Essentially, the distributor scans a segment for particular configurations of tiles and holes where boost make sense and are fun. For example, if the player doesn’t have full speed and isn’t able to make a gap of size 2 (maybe because they had to slow down for a more skill-based segment), having a boost tile right at the edge of that size-2 gap comes in very handy! Likewise, we have boosts that point diagonally over $2\sqrt{2}$ gaps too, like in Figure 5.

{{< figure src="BoostPlacement.png#center" height=400 caption="Figure 5: A few example boost placements, after detecting size-2 gaps and size-$2\sqrt{2}$ gaps. The possibilities are endless, and only require writing a specific pattern detector.">}}

Lastly, we should mention that all of these boost placements also involve collectibles. This gives the player a good incentive to use them, and we in turn make good use of this by having just a few special placements of boosts that encourage the player to take a risk and get boosted off-of the level… ;)

But we’ll leave that for you to discover in-game!

We hope you have fun playing Guinea Pug, and perhaps recognize some of the boost placements and types of randomly generated segments. Guinea Pug is available on the App store and on the Play store.
