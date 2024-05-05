# Unity 2D notes


# Art

Important: When you create sprites for Unity, the dimensions of the files are critical for optimal sprite rendering. The height and width of a sprite file should be a power of two size in pixels (2^n). The height and width don’t need to be the same value: both 16 by 16 pixels and 8 by 32 pixels would meet this requirement. For these walk cycle sprites, our animator created 512 by 512 pixel files.

# TileMaps

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


# 2D Animation

Start with game object. Add component 'Animator'.

Create a controller: in Projects folder, go to your Anim folder (or create it), right click and create 'Animator Controller'.

Open the Animation window by selecting Window > Animation > Animation. 
Make sure the game object you want to animate is selected. Animation window  (expand right side) should have a Create button. Click it.

You can animate any property from any component in your GameObject over time with the Animator. It could be the Sprite Color that you would like to change over time, or the size. In  this case, you want to change the Sprite used by the Sprite Renderer.

Create New Clip.


Go to Projects window, and select sprites (either individual, or from a sheet) and drag to Animation window.

Set Samples to ~4. If no Samples field, click on the lower 3 dots at top of window.

To make opposite e.g. left anim, create another clip, drag the same sprite into the timeline, and add the Sprite Renderer Flip X property, and make sure it's enabled for all the frames.

**The controller**

Make sure game object's prefab is selected.
Go Window -> Animation -> Animat**tor**.

Delete any existing states.

Then right-click somewhere in the graph and select Create State > From New Blend Tree. 

The Blend tree map the values for the configured parameters to animation clips.

The Blend Type determines the number of parameters evaluated to select the animation clips.

Replace the auto provided Blend property, by adding them in the Parameters section of the Animator window.

Set Thresholds for the directions, typically (-0.5, 0.5)


More info: https://learn.unity.com/tutorial/sprite-animation?uv=2020.3&projectId=5c6166dbedbc2a0021b1bc7c#5c7f8528edbc2a002053b3f7

**Thresholds**

AFAICT: input values within the thresholds, where the lines in the visualiser crosses, is where the Controller will *blend* animations. I.e. sometimes it will pick one animation, sometimes the other, depending on how much "influence" each axis has.

**Vector Normalisation**
WARNING: when using Vector2.Normalize(), if only one axis changes all the time (i.e. up/down movement), the other axis' value will decrease
over repeated calls. I don't know why this happens but I suspect it's because of truncated values in floating point calculations.
Normalize will result in the biggest value becoming no more than one. 

The calculation is 

`V/|V| = (x/|V|, y/|V|)`

So there's a lot of division going on when you normalise.


To prevent this, ensure the input values have thresholds:
```
   public static float ConstrainToMinimum(float input, float minThreshold)
   {
       if (Mathf.Abs(input) < minThreshold) { 
           if (input > 0.0f)
           {
               return minThreshold;
           }
           else
           {
               return -minThreshold;
           }
       }
       return input;
   }
```

**Sending values to the Animation Controller**:

In the gameobject's script:

Add to `Start()`: `animator = GetComponent<Animator>();` 

In update function, something like this:

```
float horizontal = Input.GetAxis("Horizontal");
float vertical = Input.GetAxis("Vertical");
                
Vector2 move = new Vector2(horizontal, vertical);
        
if(!Mathf.Approximately(move.x, 0.0f) || !Mathf.Approximately(move.y, 0.0f))
{
    lookDirection.Set(move.x, move.y);
    lookDirection.Normalize();
}
        
animator.SetFloat("Look X", lookDirection.x);
animator.SetFloat("Look Y", lookDirection.y);
animator.SetFloat("Speed", move.magnitude);
```
In general, you will normalize vectors that store direction because length is not important, only the direction is. 
NOTE: You should never normalize a vector storing a position because as it changes x and y, it changes the position!



**Transitions**:

* If there is no condition, the Transition will happen at the end of the Animation
* Or, we can set a condition based on your parameters.

  
`Has Exit Time` unchecked? That means the moving animation won’t wait to finish before the State Machine goes to the Idle animation, it will instantly change.
Use case is typically to let a animation loop once then reset to a idle state.


**Parameters**"

Trigger - use for a discrete event like being hit



**Sending values to Animator**

```
 move = moveAction.ReadValue<Vector2>();

        if(!Mathf.Approximately(move.x, 0.0f) || !Mathf.Approximately(move.y,0.0f))
        {
            moveDirection.Set(move.x, move.y);
            moveDirection.Normalize();
        }
        animator.SetFloat("Look X", moveDirection.x);
        animator.SetFloat("Look Y", moveDirection.y);
        animator.SetFloat("Speed", move.magnitude);
```

# Collision Detection

Add a Box Collider 2D or other collider to the object you want to collide with.

On the object that will deal with the collision, add callback method:

```
private void OnCollisionEnter2D(Collision2D collision)
{
    
}
```

#  Creating visible game objects

* Right click in Hierarchy window and create empty
* Assign Sprite Renderer 2D on object
* Select a sprite or animation
* NB: Ensure Z coordinates are 0 otherwise it won't show up


# Common problems

## Object won't show up in Game view or in running game, but shows up in Scene view.
Check the z coordinates, make sure they're 0.

* 
## Collision method not triggering.

OnCollisionEnter2D vs OnTriggerEnter2D

By default the 'OnCollision' functions are called when two Colliders collide. However they are disabled when the Trigger functions are enabled.

The 'OnTrigger' functions are activated when the 'Is Trigger' checkbox is enabled on the Box Collider 2D component.

Trigger colliders don’t cause collisions. Instead, they detect other Colliders that pass through them, and call functions that you can use to initiate events 

