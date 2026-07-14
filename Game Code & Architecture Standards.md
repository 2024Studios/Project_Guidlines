# 🏗️ Game Code & Architecture Standards

> *The foundational guidelines for writing clean, decoupled, and performant code at 2024 Studios written by/Ahmed Mohi. cleaned up, structured, and examples by 3mo GEMINI.*

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
| **1. High (Presentation)** | UI Logic, Anything the player directly sees, hears, or interacts with via UI. This layer reacts to state changes but rarely dictates game rules. | `MainMenu`, `HUD`|
| **2. Mid (Gameplay System)** | Logic specific to a particular level, scene, or game mode. | `EnemyMovement`, `Puzzle1Logic`, `CameraMovement`|
| **3. Low (Core Game Systems)**| Foundational game systems | `Save System`, `EnemyAI`, `CharacterController`, `Inventory`|
| **4. Lowest (Core Data)**| Structs , Enums , scriptable objects that are global for the entire project | `GameplayMode`, `MissionType`, `ScoreStruct`, `ScoreChangeEvent` |


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

## ⚖️ 3. The Golden Rules of System Communication

When writing a script that needs to interact with another system, follow this decision tree to find the exact tool you must use. **Stop at the first condition that matches your need.**

### 🌳 The Decision Flowchart

> [!NOTE]
> GitHub natively renders this block as a visual flowchart.

```mermaid
flowchart TD
    Start([I need my script to communicate...]) --> Q1

    Q1{1. Is it a physical entity<br>in the 3D/2D world?}
    Q1 -- Yes --> A1[<b>Physics + Interface</b><br><code>hit.TryGetComponent&lt;IInteractable&gt;()</code>]
    Q1 -- No --> Q2

    Q2{2. Is it pure, stateless<br>math or data?}
    Q2 -- Yes --> A2[<b>Static Function</b><br><code>CombatMath.CalculateCrit()</code>]
    Q2 -- No --> Q3

    Q3{3. Fetching a global asset<br>from a table/list?}
    Q3 -- Yes --> A3[<b>Data ID / Scriptable Object</b><br><code>AudioService.Play(DashAudioID)</code>]
    Q3 -- No --> Q4

    Q4{4. Do you need a return value<br>RIGHT NOW?}
    Q4 -- Yes: Lower Layer --> A4A[<b>Service Locator</b><br><code>Services.Get&lt;ISaveSystem&gt;()</code>]
    Q4 -- Yes: Same Layer --> A4B[<b>Blackboard / Interface</b><br><code>LevelBlackboard.IsRaining</code>]
    Q4 -- No: Fire & Forget --> A5[<b>Event Bus</b><br><code>EventBus.Fire(new JumpEvent())</code>]

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
