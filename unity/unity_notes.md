# Unity 2D notes


# Art

Important: When you create sprites for Unity, the dimensions of the files are critical for optimal sprite rendering. The height and width of a sprite file should be a power of two size in pixels (2^n). The height and width don’t need to be the same value: both 16 by 16 pixels and 8 by 32 pixels would meet this requirement. For these walk cycle sprites, our animator created 512 by 512 pixel files.

# Units of measurements  

Dimensions in Unity are typically presented in Pixels per Unit.

Lets say on a tile's import settings, you set the PPU to 100. That means 100 pixels of the tile will fit into one unit.
So if tile's actual dimensions are 200x200 pixels, it will take up 2 units.

Think of PPU as "How much of the image should fit into a unit?"

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


**Default state**

It's important to understand that, in the absence of any triggers or values for animation parameters, the animations linked to the Default State is going to loop over and over.

Right click any state to set the default.

Note: if the subject is following a strictly linear path, e.g. spawn, do something, then disappear, you need to set the default state to Empty (right click and create Empty state).



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

There are 2 variants:
* Colliders - generates a physics collision and will interact with the physics calcs, e.g. stop a player from moving through it.
* Triggers - no physics interaction, i.e. player can move through it.

On the object that will deal with the collision, add callback method:

```
//For colliders
private void OnCollisionEnter2D(Collision2D collision)
{
    
}

//for triggers
 private void OnTriggerEnter2D(Collider2D other)
 {
     Debug.Log("Milo.OnTriggerEnter2D: " + other.gameObject);
 }
```

#  Creating visible game objects

* Right click in Hierarchy window and create empty
* Assign Sprite Renderer 2D on object
* Select a sprite or animation
* NB: Ensure Z coordinates are 0 otherwise it won't show up

## Moving things around

Generally you'll want to move the rigidbody to make sure you play nice with the physics system.
If something doesn't have a rigidbody, you can manipulate the game object's transform directly:

(move is a private Vector2 property)

```
void Update()
{
    Vector2 position = (Vector2)transform.position + move * Time.deltaTime * speed;
    transform.position = position;
}
```

If rigidbody:
```
 void FixedUpdate() {
     Vector2 position = (Vector2)rigidbody2D.position + move * Time.deltaTime * speed;
     rigidbody2D.MovePosition(position);
 }
```
## Working with time

`Time.deltaTime (float)` is the time in **seconds** since last frame update.
`Time.time (float)` is the time in seconds since game started.

As properties:

```
    public float changeTime = 2.0f;
    float timer;
```

In Start() method:
```
        timer = changeTime;
```

in Update() method:
```
timer -= Time.deltaTime;
if (timer < 0)
{
    move.x = -move.x;
    move.y = -move.y;
    timer = changeTime;
    animator.SetFloat("Move Xf", move.x);
    animator.SetFloat("Move Yf", move.y);
}

```

# InputSystem

Make sure the InputSystem package is installed and enabled.
In Project window, right click -> Create -> Input Action 
Setup a default InputActionMap
In the Map, setup Actions.
For left,right,up,down type action, create a Composite type.

In player controller code:

```
private void Awake()
{

    defaultActionMap = primaryActions.FindActionMap("DefaultActionMap");

    moveAction = defaultActionMap.FindAction("Move");
    moveAction.Enable();

    //if only interested in a single button press, like jump once
    moveAction.performed += NextPlayerInputAction_performed;
    //if need to stop-start movement, but beware it won't fire multiple times
    //if value type is composite and you try to run diagonal
    moveAction.started += NextPlayerInputAction_performed;
    moveAction.canceled += NextPlayerInputAction_performed;
}
```

If you're trying to change player movement with multiple hold keys 
(e.g. pressing d and w to run diagonally) and you're working with a Vector2 composite input type,
then it's better to use the polling api:

```
void Update()
{
    move = moveAction.ReadValue<Vector2>();
    RespondToInput();
}
```

#  Spawning objects

* Create a Spawner game object and create a C# script for it.
* Add public variables for the prefabs of the objects you want to spawn.
* Drag the prefabs in the UI over to the public variables of the spawner game object's inspector window.

In the Spawner script, create it with
```
 var enemy = Instantiate(enemyPrefab, transform.position, Quaternion.identity);
```
# Screen

Get screen size:
https://gameandcode.com/blog/objectspawner/

Tl;dr:

```
  public void CalculateScreenBorders()
    {
        // Get the camera used to render the scene
        Camera mainCamera = Camera.main;

        // Get the distance between the camera and the near clipping plane
        float cameraDistance = mainCamera.nearClipPlane;

        // Calculate the coordinates of the screen borders
        bottomLeft = mainCamera.ScreenToWorldPoint(new Vector3(0, 0, cameraDistance));
        topLeft = mainCamera.ScreenToWorldPoint(new Vector3(0, Screen.height, cameraDistance));
        topRight = mainCamera.ScreenToWorldPoint(new Vector3(Screen.width, Screen.height, cameraDistance));
        bottomRight = mainCamera.ScreenToWorldPoint(new Vector3(Screen.width, 0, cameraDistance));

        width = bottomRight.x - bottomLeft.x;
        height = topLeft.y - bottomLeft.y;
    }
```

# Dynamic Camera

* Add the Cinemachine package via the Package Manager
* Under main camera in Hierarchy, right click -> Cinemachine -> 2D Camera
* Things might seem further away. In Game view, go to Virtual Camera's `Lens Ortho Size` setting, set to 5.
* Drag the PlayerCharacter GameObject from the Hierarchy window and assign it to the Follow property in the CinemachineVirtualCamera component.
* In the Extensions section in the Inspector, click the Add Extension property dropdown and select CinemachineConfiner2D
**Orthographic size** is how many units the camera fits in half of its height. When you set the Lens Ortho Size property to 5, 10 units of the world are displayed vertically on screen.
* In the Hierarchy window, create a new empty GameObject and name it “CameraConfiner”.
* Add a `PolygonCollider2` or `CompositeCollider2D` component to the CameraConfiner Gameobject. Edit the collider to contain the area where you want to restrict the camera. Once done, make sure to click the Edit Collider again.
* Assign the CameraConfiner GameObject to the Bounding Shape 2D property in the Virtual Camera's Inspector window.
* Prevent the physics system from pushing out the player: In the Inspector window header, select the Layer dropdown, then select Add Layer. Create a new layer called “Confiner” and set the CameraConfiner GameObject layer to Confiner.
* From the main menu, select Edit > Project Settings > Physics 2D. Disable the Confiner layer’s collisions with every other layer.


# Physics

## Colliders

* **Colliders** enable Unity to register when GameObjects intersect it other.
* GameObjects must have a RigidBody for a **collision** to occur.

Note that the more complex a collider shape is, the more computationally expensive it becomes to detect collision.

Tip: Can use ProBuilder to use simpler "proxy" meshes for complex shapes.

Colliders are included in many of Unity's 3D objects from the GameObject dropdown menu.

Best practice:
* Explicitly define which objects can collide in order to reduce the number of calculations
* Some physics attributes on Static Objects can be very expensive (?)

## Triggers

Triggers function the same as Colliders but disables Physics on the component, enabling objects to pass through it.
Enable it by ticking the 'Is Trigger' checkbox on the Collider component.

**NOTE** One of the objects colliding must have a RigidBody component attached. Best practice: objects that move within a Trigger
should have the RigidBody.

## RigidBody

The RigidBody component allows GameObjects to be affected by physics.

Some of the properties:
* Mass - objects of larger mass affect objects of lower mass more
* Drag - the dampening of velocity over time
* Angular Drag - dampening of angular velocity (rotation?)
* Is Kinematic checkbox - affect other objects in the physics simulation but are not effected by physics itself. E.g. a hand in VR manipulating objects.
  * Also has impact on animation. If enabled, then Animation Engine has control of the object, otherwise the object is controlled by the Physic Engine.
* Interpolate
  * Interpolate - smooth movements of objects are based on information from the previous frame of animation in the animations timeline
  * Extrapolate - smooth movements are based on a guess of the next frame
* Collision Detection:
  * Discrete - default
  * Continuous - fast objects that interact with static objects
  * Continuous Speculative - predictive collision checking
* Constaints - for each axis X, Y, Z, define which axis should not move.


 Note there's additional settings in Edit -> Project Settings -> Physics

## Scripting

Looks like this:

```
void OnCollisionEnter(Collision collision)
{
   if(collision.gameObject.CompareTag("Enemy"))
   {
   }
}
```
Other messages:

* OnCollisionStay(Collision) - called during collision
* OnCollisionExit(Collision) - called when collision has stopped
* bool rigidBody.useGravity - enable or disable gravity
* rigidBody.AddForce(Vector3) - add force to move in a particular direction
* rigidBody.AddTorque(Vector3, Force Mode) - add rotational force around a particular axis


## Force Modes

* Acceleration - apply force that increases at steady rate
* Force - default, gradually apply force to a object, accounting for its mass
* Impulse - apply instant force in stead of one that gradually builds up over time
* VelocityChange - apply instant force in different directions, disregarding mass

## Updates

* Update() - called once per frame
* FixedUpdate() - called multiple times per frame. Most physics calcs will be called in FixedUpdate. Time between updates are fixed.

## Physics materials

Controls how friction of surfaces interact with other surfaces.

Custom materials can be applied to colliders. 

Properties:
* Dynamic Friction
* Static Friction
* Bounciness
* Friction Combine
* Bounce Combine

The create custom physics material:
* Assets -> Create -> Physics Material
and add it to the GameObject by selecting the newly created material in Project window and dragging it over to the Material property.

## Physics Joints

A Joint Component connects a RigidBody to another RigidBody or a fixed point in space.

Joints apply forces that move rigid bodies. Joint limits can restrict certain movements.

* Character Joint - e.g. a hip or shoulder. Constrains movement on linear access, enables all angular freedoms. Rigidbodies attached to the same char joint orient around each axis and pivot from a shared origin.
* Configurable Joint - emulates skeletal joint such as joints in a ragdoll. Can configure this joint to force and restrict movement in any degree of freedom.
* Fixed Joint - Restrict movement of rigidbody to follow movement of another body it's attached to. Use case: if you need rigidbodies that easily break apart, or you want to connect movement of two rigidbodies.
* Hinge Joint - attaches a rigidbody to a point in space at a shared origin and allows bodies to rotate around a specific axis from that origin. Useful for doors or fingers.
* Spring Joint - Keep rigidbodies apart from each other but let the distance between them stretch slightly. The spring acts as a piece of elastic that tries to pull the two anchor points together to the exact same position.

## Raycasting

Casts an invisible ray (or connection) between two Physics objects.

```
Physics.Raycast(Vector3 origin, Vector3 direction)
Physics.Raycast(Vector3 origin, Vector3 direction, RaycastHit info, float distance, int layerMask
Physics.Raycast(Ray rayname, RaycastInfo info, float distance, int layerMask)
```

Some of the less obvious paramaters
* distance - max distance the ray should travel
* layerMask - a layer mask used to selectively ignore certain sets of colliders when casting
* queryTriggerInteraction? - specifies if query should hit Triggers

tip: use tags to help identify objects being hit.

Example:

```
Raycast hit;
Ray landingRay = new Ray(transform.position, Vector3.down);
if(!deployed) {
  if(Physics.Raycast(landingRay, out hit, deploymentHeight)) {
    if(hit.collider.tag == "env") {
       deployParachute();
    }
  }
}
```

**Raycasting Best Practice**:
* keep the number of casts in a scene to a minimum
* do not raycast inside FixedUpdate() or Update()
* MeshColliders should be avoided

 



# Common problems

## Object won't show up in Game view or in running game, but shows up in Scene view.
Check the z coordinates, make sure they're 0.

* 
## Collision method not triggering.

OnCollisionEnter2D vs OnTriggerEnter2D

By default the 'OnCollision' functions are called when two Colliders collide. However they are disabled when the Trigger functions are enabled.

The 'OnTrigger' functions are activated when the 'Is Trigger' checkbox is enabled on the Box Collider 2D component.

Trigger colliders don’t cause collisions. Instead, they detect other Colliders that pass through them, and call functions that you can use to initiate events 

Other things to check:
* At least one participant of the collision needs to have rigidbody component attached.
* Check that is trigger is not selected on any of the colliders
* Pause the game and check in the scene view that the green boxes of the colliders actually colliding
* Check the layers of the gameobjects and check if they are should collide because of the layer based collision

# Great Resources

* Explaining Pixels per Unit: https://youtu.be/iFpfj-lMh-g?si=BXIx2tm8ABJVVqJ6
* 
