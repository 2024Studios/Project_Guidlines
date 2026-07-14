# 🏗️ Game Layered Architecture Standards

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
| **Direct Reference** | A hard link to another class via `GetComponent` or `[SerializeField]`. | you are with-in the same layer and with-in the same assembly definition |
| **s2024 Service Cache** | A safe, interface-driven global registry (`Services.Get<T>()`). | Higher layer wants to communicate with lower layer, good when you want a reference to a class that you will call offen or expect return value from it |
| **s2024 Event** | A decoupled struct broadcasted to the void (`EventBus.Fire()`). | Fire-and-forget actions across layer boundaries. the most suitable way if lower layers want to trigger something , many systems needs to listen to this change |
| **Static Methods** | Pure functions used strictly for math and data transformations. | for example if you have damage calculations on lower level script , think of it as calling Mathf library or vector3 library |
| **s2024 Data IDs** | Unique IDs that can reference game entities | similar to localization, they are used to reference a single entity that exists in a big table or in otherwords you just want to reference an audio from a list of audios or an ability from the abilities list  |
> [!TIP]
> **Event Timing:** Events can be executed instantly or queued until the end of the frame, depending on the urgency of the action.

### 🔄 2024 Service Cache

- Higher level calling -> Lower level
- for example if UI HUD needs to keep track of playerHealth , you can just use serviceLocator to keep reference of playerHealth
- for more info check here 
- https://github.com/2024Studios/CustomTools/blob/main/Packages/com.s2024.service_cache/README.md

### 🔄 s2024 Event

- can be called from any layer and type safe , good when lower level needs to notify higher levels or there's many systems that needs to receive a notification 
- for more info check here 
- https://github.com/2024Studios/CustomTools/blob/main/Packages/com.s2024.event_manager/README.md

### 🔄 **Static Methods**

- good to keep it simple when seperating a math library or something like damage calculations , when this damage equation can be called by alot of systems
- only work because this methods don't change any class members , they are side affect free meaning they always return the same output when giving the same input
- think like Mathf.Sin

### 🔄 **s2024 Data IDs**

- Easy way to reference project asset that won't change in runtime , think of it like using keys to reference localization strings 
- you can use IDs to reference AudioClips + any modifications you want to create on this audio clips 
- you can use IDs to reference scriptableobjects in simplier manager and pass this id around 
- for more info check here 
- https://github.com/2024Studios/CustomTools/blob/main/Packages/com.s2024.data_ids/README.md

---
