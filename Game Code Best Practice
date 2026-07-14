
## ⚡  Code-Level Performance & Best Practice

Beyond macro-architecture, our micro-architecture (how methods and variables are written) must adhere to strict performance standards to avoid Garbage Collection (GC) spikes and native-crossing overhead.

### Cache Components in Hot Paths
Unity property access (like `transform.position` or `Time.deltaTime`) crosses native C++ to C# boundaries. Reading it repeatedly in `Update` or physics loops adds massive overhead.

❌ **Avoid (Multiple Native Crossings):**
```csharp
void Update()
{
    transform.position += transform.forward * Time.deltaTime;
}
```

✅ **Better (Cache Once Per Frame):**
```csharp
void Update()
{
    Transform t = transform;        // cache
    float dt = Time.deltaTime;      // cache
    Vector3 pos = t.position;       // read once
    Vector3 forward = t.forward;

    pos += forward * dt;

    t.position = pos;               // apply once
}
```

### Minimal Functional Separation (Without Over-Engineering)
Keep logic readable and explicitly separate **Side Effects** (methods that change the state of an entity or the game world) from **Pure Computation**. 

By having more methods that just do pure calculations, our code becomes trivially easy to test and validate. Isolating side effects makes it vastly easier to track down bugs.

**The Pattern:**
1. **Gather (Side Effects):** Read transforms, read deltaTime, read input.
2. **Compute (Pure Math):** No Unity calls, no allocations, no mutation outside the method scope.
3. **Action (Side Effects):** Write transform, play audio, trigger animation, or even **change class variable**.

> [!NOTE]
> This is not a major shift in coding style, but  it's mainly a way to help arrange the flow of code when it's possible.
> so it's a way of organizing methods and method responsibilities, sometimes it will be hard to apply, so it's okay not to do it when it overcomplicates the logic 
> check the code below for a concrete example
 
**Example:**
```csharp
void Update()
{
    Transform t = transform;
    float dt = Time.deltaTime;

    Vector3 current = t.position;                            // Gather, it can be it's own function as needed
    Vector3 nextPosition = ComputeVelocity(current , dt);    // Pure Compute
    nextPosition = ComputeFriction(current , nextPosition);  // Pure Compute
    ApplyMovement(nextPosition);                             // Action (Apply)
}

Vector3 ComputeVelocity(vector3 current , float dt)
{
    // Pure logic. Easily testable.
    vector dir = new Vector3(0, 5f, 0);
    return current + dir * dt;
}

vector3 ComputeFriction(vector3 current , vector3 nextPosition)
{
  //It's okay to use things like physics where it does no state change, nor changes class methods, nor calls any other action methods in the class
  //Simply, you can move this method and put it in any other class without the need to change anything
  vector3 surfacePoint = physics.raycastNonAllo(...);
  float friction = surfacePoint.sqrMagnitude * 0.02f;    //Normally 0.02f will be const 
  return nextPosition - (nextPosition * friction);       //Assume a correct friction logic equation as this is for demonstration only
}

void ApplyMovement(vector3 nextPosition)
{
  EventManager.Publish(PlayFootSteps);
  t.position = nextPosition;
}
```
> [!CAUTION]
> **Compute methods won't change class members**
> That's an important point, pass the variables, even if it's a class member, to the compute method to avoid state changes
> - When we minimize how many methods change the state of an object -> we minimize the places in code where we need to check for bugs

> [!CAUTION]
> changing a member variable of a class will be considered an action method (it's not a bad thing, it's just something to minimize where it happens)

> [!NOTE]
> **KinematicCharacterController script:** has a good example of that where it ties with having structs of related data.
> so instead of modifying the variables like isPlayerGrounded,GroundNormal,etc all over the script, he has different structs and each frame he generates a new struct with the new values
> so by combining structs + basic DODs concepts + basic side effect free concepts
> - Memory Locality and Cache hit
> - The data is treated as a Snapshot.
> - The logic takes "Input State A," performs math, and produces "Output State B."
> - This makes the code much easier to debug because the data doesn't change "behind your back."

### Use `const` and `readonly` for Non-Changing Values
Hardcoded magic numbers should be avoided. Using explicitly declared constants improves readability, prevents accidental mutation, and enables aggressive compiler optimizations.

```csharp
private const float Gravity = -9.81f;
private const int MaxHealth = 100;
```
*Note: Use `readonly` when a value must be assigned at runtime (e.g., in `Awake` or a constructor) but should never change after initialization.*

### Use `struct` as Dumb Data Storage (When It Makes Sense)
We leverage structs to keep our memory allocations off the heap, avoiding the Garbage Collector entirely.

**Use structs when:**
* Data is small.
* No inheritance is needed.
* It represents closely related state.
* The fields are frequently accessed together.

**Example: Movement State**
```csharp
private struct MovementState
{
    public Vector3 direction;
    public float velocity;
    public float distanceSqr;
}
```
✔ Groups related hot data.  
✔ Reduces scattered fields in a class.  
✔ Improves cache locality.  
✔ No getters/setters (properties), just plain fields for speed.  

> [!WARNING]
> **Avoid:** Putting logic-heavy code inside structs, or storing Lists/reference types inside structs that are accessed in hot loops.

---
