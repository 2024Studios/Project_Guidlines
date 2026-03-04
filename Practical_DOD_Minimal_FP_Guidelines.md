# Practical Best Code Guidelines (Unity)
------------------------------------------------------------------------

## 1) Cache Components in Hot Paths

### Why?

Unity property access (like `transform.position`) crosses native
boundaries.\
Reading it repeatedly in `Update` or physics loops adds overhead.

### ❌ Avoid

``` csharp
void Update()
{
    transform.position += transform.forward * Time.deltaTime;
}
```

### ✅ Better (Cache Once Per Frame)

``` csharp
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

✔ Fewer native calls\


------------------------------------------------------------------------

## 2) Minimal Functional Separation (Without Over-Engineering)

Goal: - Keep logic readable - Separate side effects methods 
Side effect means a method that change the state of entity
so the goal is less side effects methods , so have more methods that just due pure calculations that make it easier to test and validate
and separate methods that change the game state so it's easier to find issues 

### Pattern

**Gather (side effects)** - Read transforms - Read deltaTime - Read
input

**Compute (pure math)** - No Unity calls - No allocations - No mutation
outside method scope

**Action (side effects)** - Write transform - Play audio - Trigger
animation

### Example

``` csharp
void Update()
{
    Transform t = transform;
    float dt = Time.deltaTime;

    Vector3 current = t.position;           // Gather
    Vector3 velocity = ComputeVelocity(dt); // Pure
    Vector3 next = current + velocity * dt;// Pure

    t.position = next;                      // Apply
}

Vector3 ComputeVelocity(float dt)
{
    return new Vector3(0, 5f, 0);
}
```

✔ Clear flow\
✔ Testable compute logic\
✔ No LINQ (avoids GC)

------------------------------------------------------------------------

## 3) Use `const` for Non-Changing Values

Constants: - Improve readability - Prevent accidental mutation - Enable
compiler optimizations

``` csharp
private const float Gravity = -9.81f;
private const int MaxHealth = 100;
```

Use `readonly` when value is assigned at runtime but never changed
after.

------------------------------------------------------------------------

## 4) Use `struct` as Dumb Data Storage (When It Makes Sense)

Use structs when: - Data is small - No inheritance needed - Represents
related state - Frequently accessed together

### Example: Movement State

``` csharp
private struct MovementState
{
    public Vector3 direction;
    public float velocity;
    public float distanceSqr;
}
```

✔ Groups related hot data\
✔ Reduces scattered fields\
✔ Improves cache locality\
✔ No properties, just plain fields

Avoid: - Putting logic-heavy code inside structs - Storing Lists or
reference types inside hot structs



