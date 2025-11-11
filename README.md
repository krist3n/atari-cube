# Atari Video Cube 

## Intro

A lot has been said about Atari games’ clunkiness. This was due both to hardware limitations and game design as a field being its infancy, but these restrictions also allowed games to lean harder into being bizarre and experimental. The Atari Video Cube, initially a game available only through mail order in 1983, perfectly encapsulates this.

The game stars Hubie, a little man who walks across a pseudo 3D Rubix Cube and can swap colors with the tile he’s standing on. Like Tetris, it's aged well because it leans into a simple core concept, a luxury games from subsequent eras can’t often afford.

Although the original is loaded with 18 unique levels, and multiple rulesets, the game can be boiled down to the interactions between Hubie and the cube. So when we started the process of remaking, we decided to tackle each as its own object, and then bring them together at the end.

## Hubie

```
Hubie {
 	turn=”idle” -- idle/moving/rotating
	sp=0 -- sprite
	flip=0 -- set to 0 if spr not flipped, 1 if it is
	col=10  -- his current color
	x=106 -- x-position
	y=52 -- y-position
	xtar=106 -- target x-position when in motion
	ytar=52 -- target y-position when in motion
	xs=1 -- the xnum of the tile he’s standing on
	ys=1 -- the ynum of the tile he’s standing on
}
```

Hubie’s character can be seen in three animation states: idle, running (left/right movement) or climbing(top/bottom movement). He has a two-sprite run cycle, and his climb cycle is one sprite flipping across the y-axis. When transitioning from climb to idle, he simply stops flipping, but for the run, we make sure he ends on the correct frame (otherwise it would look like he has one leg). 

Each arrow key press moves him to an adjacent tile, however if an arrow press would result in him going outside the 3x3 grid, the cube rotates, and he moves to the tile opposite the arrow pressed. In other words, if you went out of bounds to the left, you would wind up on the rightmost square. In order to sell the rotation effect, rather than move towards his destination at a one pixel per frame pace, we had him jump half the distance between his current position and the target each frame. The fact that moving on-face and off-face required different positional behavior meant that we added a separate attribute which tracked if he was idle, moving, or rotating. 

We only run `set_tar()`, the function that checks his state and updates target position if he is idle. This was to prevent inputs interrupting an action. On the flip side, we only ran `move_pos()` the function which updated his position if he was in the middle of rotating or moving. The only other bit of logic with him involves swapping his color with the tile he’s standing on (when Z is pressed), and ensuring that he doesn’t move or rotate onto a tile matching his current color. But in order to do that we first needed to create the cube he stands on.

## The Cube

```
reg / new {
 	face=1-- index of face of cube
	north=0 -- which side of the face is currently north
	rdir=-1 -- which way we’re rotating (-1 when not)
	size={0,0,120,120} -- the dimensions of the face
}
```


The titular cube is actually 6 sets of 3x3 grids of squares. Each level has its own starting color values for its board, so when we load into a level we create a modifiable copy of the current level’s colors so we can update what each square's color value should be. As such we can simplify the object we interact with down to simply the current face, and when rotating, the new face. Figuring out the orientations of all the faces was a very difficult initial challenge. Rather than develop a highly complex universal algorithm, we opted to hard code the adjacent four sides and their respective orientations for each face when facing north. 

We used a 6 sided numbered die to figure this out. For example, an upright 4 turned left would reveal an upside-down 2. But now if we were to rotate from this downward 2, all of the adjacent faces would be off as they were hardcoded around a right-side up two. This is where the `north` attribute comes in, which keeps track of which side is currently facing north from the player’s perspective, and allows us to correctly match each square to its correct color by rotating the adjacency and color data accordingly. 

Then, based on which way we’re rotating we create a different face object entitled new whose size starts at 0x0 and gradually expands pixel by pixel to replace the old board from whatever direction we’re rotating towards. After that, `reg` is over-written to hold `new`’s values so that the process can restart for the next rotation.

Lastly, we made them interact. We triggered cube rotations based on Hubie’s position, within `set_tar()` we ensured Hubie could rotate or move onto squares of the same color, and we added the ability to swap Hubie’s color with the tile he was standing on using `Hubie.xs` and `Hubie.ys`. While we were a bit hesitant to develop these objects separately, knowing they would eventually interact made combining them a smooth process. 

## Conclusion

With more time we would have smoothed out Hubie’s movement, somehow added shading and gap disappearance to the rotations (gaps didn’t look good without shading, which was very hard to approach in Tic80),  and added more unique level twists and scoring conventions. In the original, on certain levels you can press a button and Hubie will solve it for you in the fastest way. This was so daunting and impressive, that we knew attempting it could take possibly more time than everything else put together. While we did add sound effects, we also would have loved to add music, additional sprite work, and maybe in the future attempt a true 3D version.
