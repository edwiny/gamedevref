# Unity 2D notes


## OnCollisionEnter2D vs OnTriggerEnter2D

By default the 'OnCollision' functions are called when two Colliders collide. However they are disabled when the Trigger functions are enabled.

The 'OnTrigger' functions are activated when the 'Is Trigger' checkbox is enabled on the Box Collider 2D component.

Trigger colliders don’t cause collisions. Instead, they detect other Colliders that pass through them, and call functions that you can use to initiate events 

## Art

Important: When you create sprites for Unity, the dimensions of the files are critical for optimal sprite rendering. The height and width of a sprite file should be a power of two size in pixels (2^n). The height and width don’t need to be the same value: both 16 by 16 pixels and 8 by 32 pixels would meet this requirement. For these walk cycle sprites, our animator created 512 by 512 pixel files.

## TileMaps

To create the TileMap:

* In Hierarchy window, rightclick and add 2D Object -> Tilemap -> Rectangular
* This creates a Grid component and the TileMap underneath it.

* Go to Project window, under Art folder create Tiles

Creating individual tiles:
* Create > 2D > Tiles > Rule Tile -> select asset
 
Creating a Tile Palette:

* From the main menu, select Window > 2D > Tile Palette. 
* Select No Valid Palette, then Create New Tile Palette > Create New Palette.
* Name your palette  “Game Palette”, then select Create.
* Select the Tiles folder you created previously 

Painting:

Fixing the spaces between the tiles:
* In the Hierarchy window, select the Grid GameObject.
* In the Inspector window, go to the Grid component and find the Cell Size property. Both its X and Y-values are set to 1. That means each cell is one unit in width and one unit in height.


Make sure the TileMap is drawn first:
* In the Hierarchy window, select the Tilemap GameObject.
* In the Inspector window, go to the Tilemap Renderer component and find the Order in Layer property. This property defines the order in which GameObjects in the same layer are drawn. 


To make some tiles collide, on TileMap:
* add 2D TileMap Collider
* select all the tiles in the Tiles folder of your Project window that should NOT collide, and
set their collider type to None.

Optimise:
* add Composite Collider to TileMap
* In 2D TileMap Collider setttings, enable 'Used By Composite'


## 2D Animation

Start with game object. Add component 'Animator'.

Create a controller: in Projects folder, go to your Anim folder (or create it), right click and create 'Animator Controller'.

Open the Animation window by selecting Window > Animation > Animation. 
Make sure the game object you want to animate is selected. Animation window  (expand right side) should have a Create button. Click it.

You can animate any property from any component in your GameObject over time with the Animator. It could be the Sprite Color that you would like to change over time, or the size. In  this case, you want to change the Sprite used by the Sprite Renderer.

Create New Clip.


Go to Projects window, and select sprites (either individual, or from a sheet) and drag to Animation window.

Set Samples to ~4. If no Samples field, click on the lower 3 dots at top of window.

To make opposite e.g. left anim, create another clip, drag the same sprite into the timeline, and add the Sprite Renderer Flip X property, and make sure it's enabled for all the frames.


