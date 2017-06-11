<!--
.. title: Step 4: Enter The Dungeon
.. slug: step-4-dungeongen
.. date: 2017-06-19 04:00:00 UTC
.. type: text
-->

# WORK IN PROGRESS: DO NOT READ BELOW THIS LINE OR YOU WILL LOSE BRAIN CELLS



In this step, we will be creating a random dungeon generator for our game. Things like this can be incredibly complex and involve all kinds of advanced concepts. But we will be making something that is pretty basic and easy to work with, and still produces interesting results.  


## Procedural Generation
ProcGen is something that's been around in video games for a real long time. [The original Rogue](https://en.wikipedia.org/wiki/Rogue_(video_game)) was released in 1980, and it certainly wasn't the first game to utilize procgen.  

I wont waste time trying to explain procedural generation here. Instead, here are some links to articles which explain this wonderful thing:  
--[Procedural Generation Wikipedia page](https://en.wikipedia.org/wiki/Procedural_generation)
--[How Procedural Generation Took Over The Gaming Industry](http://www.makeuseof.com/tag/procedural-generation-took-gaming-industry/) Quick and Dirty history of ProcGen in games (WARNING: contains annoying ad banners!)  
--[Extra Credits on ProcGen](https://youtu.be/TgbuWfGeG2o)    

### The Concept
Our use of ProcGen in our game will be pretty rudimentary. In the broadest sense, what we want our generator to produce is a bunch of rectangle-shaped rooms of random size and position, fill them with Things, and connect them with hallways.  
We need our generator to operate on some set or rules though. For a quick test, try doing what we just described above, with a sheet of paper and a pencil. You'll find that you'll either force yourself to be truely random and end up with a mess of lines, or you'll end up with something looking like a drawing of a floorplan because you're following some rules sub-consciously. `Rooms must fit within the space of the paper`. `Rooms shouldn't overlap each other`. There is a certain point where you "fill up the page" and adding more rooms becomes impossible without breaking the former rules, and you can say "it's Done".  
Our human brains are pretty good at doing this, but computer brains are stupid and can't do anything unless you tell them exactly how.  

### Modular Design
We're going to construct our DungeonGen script in such a way that it can be *modular*. That is, you could easily take this script, place it in another project which requires a similar random generator, and have it working for that project with a minimal amount of assembly.  We just plug it in and it works. In fact, if you feel comfortable with skipping most of this article, you can [download the script here](link), place it in `res://global/`, set it as a Singleton, and move on to the end of the article. I wouldn't recommend this if you're learning gdscript, as this will be a good opportunity to log some programming flight time and build experience. The best way to learn is to do, and the most direct way to do is to copy!  

## Enter The DungeonGen
Lets get right to it! Create a new script file as `res://global/DungeonGen.gd`. As its file location suggests, this is going to be another `global script`, so remember to set this script up in Project Settings > AutoLoad as "DungeonGen". Most of this step is going to involve writing code into this new script. Delete everything in the new script except the all-important First Line.  

Our modular-style generator script should be as easy to work with as we can get it. When we break it down, we should be able to just call a `generate()` function along with some arguments defining some variable parameters of our map. From this, we are returned an object containing all the procedurally generated data, which we then can use in our Map node to construct our dungeon.  Instructions Go In, Data Comes Out. Other than that, our DungeonGen script should operate entirely within itself.  

We can begin by defining this function with a `pass` placeholder:  

```python
extends Node

func generate():
  pass
```  

Before we can begin generating anything, we need to establish a few basic parameters our map generator is going to work within. The most basic parameter we need is the physical boundries of the map. The prospects of an "infinitely" large dungeon might sound juicy and delicious, but in reality we want to define some definate Size to make our maps.  
To keep them handy for tweaking, we're going to define these parameters as global variables of DungeonGen. Once we're ready to implement the script into the Game Actual, we'll migrate these variables to an outside source.  
Let's start by expressing the size of our map. We can do this with a Vector2(width,height). We're also expressing these numbers as map cells.  

When it comes time for the generator function to generate rooms, we also need a couple parameters for that. We want to define the number of rooms to attempt to generate `ROOM_COUNT` and another Vector2 `ROOM_SIZE`. What this will do is define the minimum and maximum width or height of generated rooms. This isn't exactly a conventional use for a Vector2, but it works just as well (a Vector2 is really just a pair of numbers when you boil it down).  

Since we know we'll be passing these parameters into `generate()`, we can add them as arguments in the function's definition. Since we're pretending the parameter variables are not part of this script, we don't want to hard-code them into our function. Here's what this should all look like so far:  

```python
extends Node

var MAP_SIZE = Vector2(64,60)
var ROOM_COUNT = 20
var ROOM_SIZE = Vector2(3,7)

var map = []
var rooms = []

func generate( map_size=Vector2(64,60), room_count=20, room_size=Vector2(3,7) ):
	pass
```  
The `var map` and `var rooms` will remain a part of this script. `map` will store the 2D array we generate, and `rooms` will be an array of objects defining the room spaces on the map. Since we'll have several functions in this script that will need access to this data, we will define it in the script's global scope.  


We've defined `default values` to our arguments. This allows us to call just `generate()` without passing any arguments, if we wish. In the case of missing input, the function will use its default value instead.  

## The Algorithm
Our algorithm will be the set of instructions our generator will follow in order for it to construct a map. Here is a plain-language run-down of what these instructions are, in order:  

-- For each attempt in room_count:  
  -- Build a rectangle of a random width and height, within the range of room_size  
  -- Find a random position within map_size, such as the rectangle of the room remains within map_size  
  -- Check to see if the new room intersects any previous rooms that have been defined  
    -- If not, define the rectangle as a new room  
    -- If this room is not the first room, connect this room to the previously-defined room with a hallway  

## The 2D Array
We might be wondering how we're going to express all this data we're going to generate. In its purest form, the map generator is going to be doing nothing more than defining an index value to a map cell key. We could do this with a `Dictionary`, using Vector2 keys and setting their values to one of our two tile indicies. We're going to do something a bit more elegant though: we're going to be using a `nested array` or `multi-dimensional array`; since we're working in two dimensions, it will be a `two-dimensional array`.  
Arrays (general ones at least) can hold any kind of value, even another array! This might sound like some crazy Inception thing --lists within lists (within lists as deep as you want to go)-- but if you consider it, it's not all that bizarre.  

*A calandar, while arranged in a 2D grid, is functionally a one-dimensional array.* 
*A chessboard is a two-dimensional array. Any space on the board can be expressed as the column position within a row*  
*A 3D environment like Minecraft could express its block positions in a three-dimensional array: A height above a column within a row*  
*A simulation of the trajectory of a rogue asteroid could be expressed in a four-dimensional array: An XYZ position in space, within a frame of time*  

Let's have our `generate()` function construct a simple 2D array and return it:  

```python
func generate( map_size=Vector2(64,60), room_count=20, room_size=Vector2(3,7) ):
	self.map = []
	for x in range(MAP_SIZE.x):
		var row = []
		for y in range(MAP_SIZE.y):
			row.append( str(x)+":"+str(y) )
		self.map.append(row)
	return self.map

func _ready():
	print(generate())

```  
The `self.` prefix when working with the map variable isn't really needed in this case. But I like to do it this way, as it clearly labels `map` as being in global scope: a member of the script it`self`, not a member of the `generate` function. Later we will be writing code which *will* require us to work this way, so it's not a bad habit to get into.  

We also redefine a `_ready` function and have it print the result of `generate` for testing purposes. I also lowered the values of MAP_SIZE to prevent text overflow in Output (a small 10x10 grid is fine for this purpose).  
When you Play your game, you will see a big string of numbers in Output as our script prints the nested array. Re-arrange these lists a little and it starts to resemble something...  

```
[
[0:0, 0:1, 0:2, 0:3, 0:4, 0:5, 0:6, 0:7, 0:8, 0:9], 
[1:0, 1:1, 1:2, 1:3, 1:4, 1:5, 1:6, 1:7, 1:8, 1:9], 
[2:0, 2:1, 2:2, 2:3, 2:4, 2:5, 2:6, 2:7, 2:8, 2:9], 
[3:0, 3:1, 3:2, 3:3, 3:4, 3:5, 3:6, 3:7, 3:8, 3:9], 
[4:0, 4:1, 4:2, 4:3, 4:4, 4:5, 4:6, 4:7, 4:8, 4:9], 
[5:0, 5:1, 5:2, 5:3, 5:4, 5:5, 5:6, 5:7, 5:8, 5:9], 
[6:0, 6:1, 6:2, 6:3, 6:4, 6:5, 6:6, 6:7, 6:8, 6:9], 
[7:0, 7:1, 7:2, 7:3, 7:4, 7:5, 7:6, 7:7, 7:8, 7:9], 
[8:0, 8:1, 8:2, 8:3, 8:4, 8:5, 8:6, 8:7, 8:8, 8:9], 
[9:0, 9:1, 9:2, 9:3, 9:4, 9:5, 9:6, 9:7, 9:8, 9:9]
]
```  
We should know that we can access an item in an array called `map` by calling `map[x]` where x is the index (place in the list) of the item we want. If we call this 2D array `map`, we can get the first row of the map by calling `map[0]`, which would give us the 1D array representing the first row. If we want a certain item in this row, we just use a second set of brackets: `map[x][y]`. If we were to get `map[4][6]`, we should get the output "4:6". You might notice this coordinate system it a little upsidedown compared to our usual coordinate system (X is increasing as it goes down and Y is increasing as it goes right), but that's okay! This is abstract data, once we convert this data back to `set_cell` calls on our TileMap, the coordinates will come out right-side-up again. Once the generator is running properly, you wont ever need to see the data anyway. Once your ready to move on, change your code so it sets the int value `0` to each cell of your map.  

```python
# Change this line:
	row.append( str(x)+":"+str(y) )

# To:
	row.append( 0 )
```  
You can also increase your MAP_SIZE back to something more respectable. *[80x60?]*  The generation function will now create a "solid block" of walls. We will go into that solid block and carve floors into our rooms and hallways.  

## Digital Dice and How To Roll Them
Roguelikes classically make heavy use of Random Number Generation, but we haven't done anything in our game yet that requires it. That will soon change!  
Godot provides many ways for us to generate random numbers. It doesn't, however, provide us with a super-neat way of generating "dice rolls", or a random integer within a range of low-high integers. This means we get to roll our own function (pun entirely intended!)  
We can use the modulus operator `%` along with the built-in `randi()` function (which gives you a random number from 0 to "as high as Godot can count") to easily get a random number from zero to X-1. *[link to modulo rng article?]*  
`var random_integer = randi() % 10` will return a random integer between 0 and 9. This is nice, but we want an easy way to get a random integer between two values N and M, not just 0 and X. This is just real basic math. We'll create a helper function in DungeonGen to do this:  

```python
# Return a random int between 'n' and 'm'
func rnd( n,m ):
	# in case our args are out of order..
	var low = min( n,m )
	var high = max( n,m )
	return  randi() % ( high - low + 1 )  + low
```  


## Carving Rooms
The first part of our generator will be generating a room. Since we want to keep things tidy, we'll create a `create_room()` function. Our rooms will be simple rectangles. We have a built-in `Rect2` class we can use to define these rectangles. A `Rect2` can be defined by two `Vector2`s; one used to define the rectangle's origin (top-left corner) and another to define it's width and height. The idea is to get our `rnd()` function to give us four numbers: X, Y, WIDTH, and HEIGHT. We want to be Lawful Random (as opposed to Chaotic Random) when we roll these numbers, so we're using some rules when defining these random numbers. The function is fairly straightforward, and looks like this:    

```python
# Define a rectangle of room_size dimension,
# at a position within the map rect
func create_room( room_size ):
	# Get map w/h based on array size
	var map_width = self.map[0].size()
	var map_height = self.map.size()
	# Roll room width/height
	var w = rnd( room_size.x, room_size.y )
	var h = rnd( room_size.x, room_size.y )
	# Roll room X/Y origin 
	var x = rnd(0, map_width - w-1)
	var y = rnd(0, map_height - h-1)
	# Return a Rectangle
	return Rect2( x, y, w, h )
```  
Now we can use this in `generate()` to begin generating our list of rooms. The `generate()` will iterate `ROOM_COUNT` times. Each iteration, it will `create_room()` and check that result against all the rooms its created in previous iterations. If any of those intersect the new room, that room is rejected and the next iteration begins. It is likely (depending on the ratio of map size and room count) that your dungeon will only actually accept and carve some of these rooms. The goal is to get the ratio right to carve *most* of the rooms. Of course, the random nature of this process can leave us with some wild edge cases. But this is a good thing!  This is what we want.  




## Carving Halls

## The Datamap

## Putting DungeonGen To Work

## Conclusion



