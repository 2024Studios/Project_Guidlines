# Practical Data-Oriented Design in Unity (Without Fighting MonoBehaviour)

## The Problem With Naive MonoBehaviour Updates

Typical approach:

``` csharp
public class Enemy : MonoBehaviour
{
    public float speed;
    private Vector3 direction;

    void Update()
    {
        transform.position += direction * speed * Time.deltaTime;
    }
}
```

### Issues when enemy count grows:

-   Each enemy reads its own transform
-   Separate Update() call per enemy
-   Poor memory locality
-   Harder to batch logic

This is fine for small counts (10--50).\
Not ideal for hundreds or thousands.

------------------------------------------------------------------------

# Practical Data-Oriented Approach (Still Using MonoBehaviour)

We keep MonoBehaviour, but centralize hot logic.

1.  Store references once\
2.  Extract minimal movement data\
3.  Update in tight loop\
4.  Apply back to transforms

------------------------------------------------------------------------

## Step 1 --- Enemy Component (Data Holder)

``` csharp
public class Enemy : MonoBehaviour
{
    public float speed;

    [HideInInspector] public Vector3 direction;

    // Cache transform once (hot path optimization)
    public Transform CachedTransform { get; private set; }

    private void Awake()
    {
        CachedTransform = transform;
    }
}
```

✔ No Update()\
✔ No per-frame logic\
✔ Holds data only

------------------------------------------------------------------------

## Step 2 --- Enemy Manager (Centralized Loop)

``` csharp
public class EnemyManager : MonoBehaviour
{
    private Enemy[] enemies;

    private void Awake()
    {
        enemies = FindObjectsOfType<Enemy>();
    }

    private void Update()
    {
        float dt = Time.deltaTime;

        for (int i = 0; i < enemies.Length; i++)
        {
            Enemy enemy = enemies[i];

            Transform t = enemy.CachedTransform;
            Vector3 position = t.position;

            position += enemy.direction * enemy.speed * dt;

            t.position = position;
        }
    }
}
```

------------------------------------------------------------------------

# Hybrid Struct Buffer (When Count Is Large)

Separate data for tight math loops.

``` csharp
struct EnemyMovementData
{
    public Vector3 position;
    public Vector3 direction;
    public float speed;
}
```

Manager fields:

``` csharp
private EnemyMovementData[] movementBuffer;
private Enemy[] enemies;
```

### Update Flow

``` csharp
private void Update()
{
    float dt = Time.deltaTime;

    // 1️⃣ Gather
    for (int i = 0; i < enemies.Length; i++)
    {
        movementBuffer[i].position  = enemies[i].CachedTransform.position;
        movementBuffer[i].direction = enemies[i].direction;
        movementBuffer[i].speed     = enemies[i].speed;
    }

    // 2️⃣ Compute (pure math)
    for (int i = 0; i < movementBuffer.Length; i++)
    {
        movementBuffer[i].position +=
            movementBuffer[i].direction *
            movementBuffer[i].speed *
            dt;
    }

    // 3️⃣ Apply
    for (int i = 0; i < enemies.Length; i++)
    {
        enemies[i].CachedTransform.position =
            movementBuffer[i].position;
    }
}
```

------------------------------------------------------------------------

# Why This Works

✔ Single tight loop\
✔ Better cache locality\
✔ Fewer virtual calls\
✔ Easy migration to Jobs/Burst\
✔ Still fully compatible with MonoBehaviour

------------------------------------------------------------------------

# When To Use

Use when: - Enemy count \> 200 - Same logic runs on many objects -
Profiling shows Update() overhead

Avoid when: - Small object counts - Code clarity would suffer

------------------------------------------------------------------------

# Key Insight

Data-Oriented in Unity does NOT mean abandoning MonoBehaviour.

It means: - Move hot logic into tight loops\
- Keep components as data containers\
- Batch work instead of scattering it
