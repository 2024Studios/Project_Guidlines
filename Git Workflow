# 🎮 Unity Git Workflow for 2024Studio Team

## 🧠 1. The "Two-Brain" Branching Strategy

keep it down to two permanent/Main branches to simplify development and deployment:

* **`main`** The clean, playable, stable state of your game. This branch should *always* build successfully.
* **`dev`**  The integration branch where features are merged. Team members create short-lived feature branches off of `dev` and merge back into `dev`.

### 🏷️ Naming convention 
Use the following structure (you can ignore the [Ticket] for the time being)
- feature/[ticket#]_feature_name
- bugfix/[ticket#]_bug_name

### ⚡ Short-Lived Feature Branches (The Key to Unity Peace)
Keep feature branches extremely short-lived (ideally **1 to 3 days**). The longer a branch exists, the higher the risk of a catastrophic scene or prefab conflict.

---

## 🛠️ 2. The Unity Git Workflow Sequence

### Step 1: Establish Ownership Before Working  [LFS locking tool inside unity will be added later]
**Goal: Zero-conflict prep.**
Before starting a task, announce it on Discord/Slack or lock the files (if using Git LFS locks). 
> **Rule #1:** Only *one* person edits a specific Scene or major Prefab at a time.

### Step 2: Create a Short-Lived Branch
**Goal: Branch off of `dev`.**
> **Rule #2:** Create your own branch based on naming conventions.
* *Example (Code):* `feature/player_doublejump_alex`

### Step 3: Work in Sandboxes Where Possible
**Goal: Isolated testing.**
- Instead of editing the main game scene (e.g., `Level_1.unity`), work in a personal sandbox scene use folder  (`_InternalTests`) 
- any script / asset / scene in this folder can be deleted , if you something is folder is required move it to a suitable location

### Step 4: Pull and Rebase Often
**Goal: Stay up-to-date.**
> **Rule #3:** Regularly pull `dev` into your feature branch to stay up to date.
 If an artist updated a texture, you want it immediately to ensure your work doesn't drift.

### Step 5: Pull Request (PR) to 'dev'
**Goal: Review and merge.**
Open a pull Request. Once approved and merged to `dev`, safely delete your feature branch.

---

## ✅ 3. Pull Request (PR) Guidelines

All team members must adhere to the following expectations before merging:

* [x] **Required "No-Broken-Build" Rule:** The submitter must verify that the project compiles and runs locally in a *clean build* (not just in the editor) before merging to `dev`.
* [x] **Keep PRs Small:** An PR with 1,200 changed files is unreviewable. Merge code foundations first, then art integrations.
* [x] **Check the `.meta` Files:** Ensure every new asset has its accompanying `.meta` file committed in the MR. Missing `.meta` files will break references, leaving red boxes or missing script warnings for everyone else on the team.
* [x] **Non-Coder Reviews
