# 🏗️ Game Code & Architecture Standards

> *The foundational guidelines for writing clean, decoupled, and performant code at 2024 Studios written by/Ahmed Mohi cleaned up, structured, and examples by amo GEMINI.*

![Architecture](https://img.shields.io/badge/Architecture-Data--Driven%20OOP-blue?style=flat-square)
![Engine](https://img.shields.io/badge/Engine-Unity%20C%23-lightgrey?style=flat-square&logo=unity)
![Performance](https://img.shields.io/badge/Performance-Zero%20GC-success?style=flat-square)

> [!IMPORTANT]
> **Core Philosophy:** Our architecture is built on a **Layered, Performance-Aware** approach. The goal is to maximize decoupling, eliminate spaghetti code, and keep our Unity Inspectors clean, all while maintaining strict performance standards.

---

## 📑 Table of Contents

1. [Layered Architecture & Assembly Definitions](#%EF%B8%8F-1-layered-architecture--assembly-definitions)
2. [The Communication Toolbox](#-2-the-communication-toolbox)
3. [The Golden Rules of System Communication](#%E2%9A%96%EF%B8%8F-3-the-golden-rules-of-system-communication)
4. [The Communication Cheat Sheet](#-4-the-communication-cheat-sheet)
5. [Code-Level Performance & Style Guidelines](#%E2%9A%A1-5-code-level-performance--style-guidelines)

---

## 🏛️ 1. Layered Architecture & Assembly Definitions

To prevent circular dependencies and tangled code, our codebase is divided into strict layers. A layer is a conceptual grouping, but we enforce these boundaries physically using **Unity Assembly Definitions (`.asmdef`)**. 

Each layer contains multiple **Systems**, and **each System gets its own Assembly Definition**.

### 🧱 The Layer Hierarchy

| Layer | Scope | Examples |
| --- | --- | --- |
| **1. High (Gameplay)** | Specific game rules, entity logic, and game loop. | `Character Controllers`, `AI`, `Weapons`, `Game Modes` |
| **2. Mid (Features)** | Reusable gameplay mechanics and interfaces. | `Inventory`, `Quest Log`, `UI Logic`, `Objectives` |
| **3. Low (Core Engine)**| Foundational engine wrappers and global utilities. | `Save System`, `Audio`, `Localization`, `Event Bus` |

> [!NOTE]
> **Hierarchy Scaling:** This is a foundational hierarchy. Additional layers can be added or adjusted if it suits the specific scale and needs of the game.

### 🛡️ The Dependency Rule (The Iron Wall)

Code can only reference Assemblies in its own layer or the layers *below* it. 

> [!CAUTION]
> **A Lower Layer can NEVER reference a Higher Layer.**
> For example, the `Core.Audio` assembly cannot reference the `Gameplay.Weapons` assembly. If a Core system needs to know about a Gameplay event, it must listen via the Event Bus.

---

## 🧰 2. The Communication Toolbox

We have five approved methods for systems to communicate. Understanding *what* these are is the first step; knowing *when* to use them is the key to our architecture.

| Tool | Concept | Best Used For |
| --- | --- | --- |
| **Direct Reference** | A hard link to another class via `GetComponent` or `[SerializeField]`. | Intra-system links & cached hierarchy lookups. |
| **Interface** | Interacting with a contract (`IInteractable`) rather than a concrete class. | Polymorphism & spatial queries (Raycasts). |
| **Service Locator** | A safe, interface-driven global registry (`Services.Get<T>()`). | Fetching core engine managers & retrieving data. |
| **Event** | A decoupled struct broadcasted to the void (`EventBus.Fire()`). | Fire-and-forget actions across layer boundaries. |
| **Static Methods** | Pure functions used strictly for math and data transformations. | Stateless calculations passing `ref` structs. |
| **Data IDs** | Unique IDs that can reference game entities | similar to localization, they are used to reference a single entity that exists in a big table or in otherwords you just want to reference an audio from a list of audios or an ability from the abilities list  |
> [!TIP]
> **Event Timing:** Events can be executed instantly or queued until the end of the frame, depending on the urgency of the action.

### 🔄 Intra-System Communication

When scripts *within the exact same system* need to talk to each other, we generally prefer **Direct References** (via `GetComponent` / `Awake` caching) or the **Service Locator** to keep prefabs clean and avoid missing reference exceptions.

> [!WARNING]
> This is **not** a strict rule. Use the Service Locator or code-based lookups when it makes sense. However, if dragging and dropping a dependency directly into the Unity Inspector is clearly the better, more designer-friendly workflow for a specific prefab, use the Inspector.

---

## ⚖️ 3. The Golden Rules of System Communication

When writing a script that needs to interact with another system, ask yourself the following three questions to determine the exact tool you must use.

### ❓ Q1: Do I need a return value *right now* to continue my logic?

- **YES (Lower Layer):** Use the **Service Locator**. 
  - *Example: If your gameplay code needs to read a save file to proceed.*
  - *Or the system needs to be cached because it will be called regularly, for example, physics or character Motor* 
- **YES (Same Layer):** Read from a shared **Blackboard** or pass an **Interface**. 
  - *Rule: Do not use the Service Locator to couple two high-level gameplay systems together.*
- **YES (Math/Data):** Use a **Static Function**. 
  - *Example: If you need a damage multiplier, pass your data into a pure static math class.*
  - *Think of it as **Mathf.Sqrt** a simple math function that does math calculation only on its input; it doesn't change or affect class members or anything else*
- **NO:** Use **Events**. *(See Q3).*

### ❓ Q2: Am I communicating with a specific physical entity in the world?

- **YES (e.g., a Door, a Vehicle):** Use a **Direct Reference** or **Interface**.
  - *How:* Typically obtained via a spatial query like a Raycast (calculating distance strictly in **meters**) or OverlapSphere. Physics is our spatial decoupler. You interact with the `IInteractable` interface returned by the raycast, keeping the caller completely blind to the specific object type.
  - *Or Use DataID* if you need an entity that exists in a table-like format, use their ID think Unity localization, where you get the localized text by ID and in code you need variable of type *LocalizedString*, which is a wrapper around the ID, same for Audio clips, abilities, or weapons so depending on the type and system use the proper way to reference this entity *Also in this case you can say we aren't referencing a system we just need Data Asset*

### ❓ Q3: Am I communicating with a lower-level layer?

- **NO RETURN VALUE NEEDED:** Use **Events**. 
  - *Why:* This is for "Fire and Forget" actions. If the player jumps, fire a `PlaySoundEvent`.
  - It provides the best decoupling when you don't care about the return value
  - Events can also be used as a way to enhance performance by queuing similar messages. Think of it, instead of each element calling audio play, which may cause a cache miss, we queue all the audio plays in this frame, and the audio manager at the end of the frame loops over all the queued audio events to play the audio
- **RETURN VALUE NEEDED:** Use the **Service Locator** to query the lower layer.

---

## 🚀 4. The Communication Cheat Sheet

Keep this table handy when reviewing Pull Requests or architecting a new feature.

| Scenario | Target Layer | The Standard Tool | Example Implementation |
| --- | --- | --- | --- |
| **Instant Math / Logic** | Stateless Logic | `Static Function` | `CombatMath.CalculateCrit(ref stats)` |
| **Spatial Interaction** | Specific World Entity | `Physics + Interface` | `hit.collider.TryGetComponent<IInteractable>()` |
| **Needs Data Instantly**| Core Engine (Lower) | `Service Locator` | `Services.Data.ReadSlot(1)` |
| **Needs Shared State** | Gameplay (Same Layer)| `Blackboard` | `if (LevelBlackboard.IsRaining)` |
| **Fire & Forget** | Any Layer | `Event` | `EventBus.Fire(new AudioEvent("jump"))` |

---

## ⚡ 5. Code-Level Performance & Style Guidelines

Beyond macro-architecture, our micro-architecture (how methods and variables are written) must adhere to strict performance standards to avoid Garbage Collection (GC) spikes and native-crossing overhead.

### 5.1 Cache Components in Hot Paths
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

### 5.2 Minimal Functional Separation (Without Over-Engineering)
Keep logic readable and explicitly separate **Side Effects** (methods that change the state of an entity or the game world) from **Pure Computation**. 

By having more methods that just do pure calculations, our code becomes trivially easy to test and validate. Isolating side effects makes it vastly easier to track down bugs.

**The Pattern:**
1. **Gather (Side Effects):** Read transforms, read deltaTime, read input.
2. **Compute (Pure Math):** No Unity calls, no allocations, no mutation outside the method scope.
3. **Action (Side Effects):** Write transform, play audio, trigger animation.

**Example:**
```csharp
void Update()
{
    Transform t = transform;
    float dt = Time.deltaTime;

    Vector3 current = t.position;           // Gather
    Vector3 velocity = ComputeVelocity(dt); // Pure Compute
    Vector3 next = current + velocity * dt; // Pure Compute

    t.position = next;                      // Action (Apply)
}

Vector3 ComputeVelocity(float dt)
{
    // Pure logic. Easily testable.
    return new Vector3(0, 5f, 0);
}
```

### 5.3 Use `const` and `readonly` for Non-Changing Values
Hardcoded magic numbers should be avoided. Using explicitly declared constants improves readability, prevents accidental mutation, and enables aggressive compiler optimizations.

```csharp
private const float Gravity = -9.81f;
private const int MaxHealth = 100;
```
*Note: Use `readonly` when a value must be assigned at runtime (e.g., in `Awake` or a constructor) but should never change after initialization.*

### 5.4 Use `struct` as Dumb Data Storage (When It Makes Sense)
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
*Built for scale. Designed for performance at 2024 Studios.*
