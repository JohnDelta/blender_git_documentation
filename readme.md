# Blender Git – A blender add-on for version control

---

## Overview

Blender Git brings Git-based version control directly inside Blender, with no terminal required for everyday tasks. It gives you a dedicated **Git** tab in the 3D Viewport sidebar, where you can commit your work, manage branches, view history, resolve merge conflicts, connect to remote repositories, all without leaving Blender.

Because .blend files are binary, standard Git cannot tell you what changed inside them. **Blender Git** partially solves this by capturing a snapshot of your scene each time you stage a .blend file. When you look at a change in the panel, you see a plain-English list like "Object 'Cube' moved", "Material 'Base' shader nodes changed", "Render resolution changed", instead of an unreadable binary diff.

<img width="462" height="735" alt="image" src="https://github.com/user-attachments/assets/ead2d714-5e40-42c2-887b-7abab6c5b345" />

This snapshot-based workflow has a few important limits:
- Snapshots provide friendly summaries of scene changes, not a complete low-level record of every internal .blend detail.
- Staging, unstaging, and discarding work at the file level. You cannot stage, unstage, or discard one individual scene change inside a .blend file.
- Conflict resolution also works at the file level. You can inspect differences between branch versions using snapshots, but you must choose one complete .blend file version rather than picking individual changes from both.

**Who is this for:**
- Solo artists who want a save-point system with meaningful descriptions
- Small teams collaborating on a shared project via GitHub, GitLab, or any Git host
- Anyone who has ever wished they could roll back a Blender project to an earlier state

---

## Prerequisites

Before installing the addon, make sure the following are in place:

| Requirement | Where to get it | Notes |
|---|---|---|
| Blender 5.x or later | blender.org | Earlier versions are not tested |
| Git | git-scm.com | Must be installed and available system-wide |
| Git LFS | git-lfs.com | Must be installed. It handles large .blend files efficiently |

**Why Git LFS?** Standard Git stores the full history of every changed file. A 200 MB .blend with 50 commits would overload the repository to gigabytes. Git LFS stores the actual binary files in a separate object store and keeps only small pointer files in the Git history, keeping the repository lean and fast.

Git LFS is a separate install from Git itself. After installing, it registers itself globally and works transparently, the addon handles the rest.

---

## Installation

1. Download the addon `.zip` file.
2. In Blender, go to **Edit > Preferences > Add-ons**.
3. Click **Install** and select the downloaded `.zip`.
4. Enable the addon by ticking its checkbox.

The addon adds a **Git** tab to the 3D Viewport's N-panel (press **N** to open the sidebar if it is not visible).

### First-time configuration

Open the addon preferences to review the Requirements section. It shows whether Blender can find Git and Git LFS, plus the official install locations if either one is missing. The preferences do not install these tools automatically; install them separately, then restart Blender.

<img width="827" height="529" alt="image" src="https://github.com/user-attachments/assets/4aa2dcf6-aef6-409f-bcc6-03cb815be3d2" />

No workflow settings need to be configured. Scene snapshots are always captured automatically.

Before committing, set your Git author identity from **Configuration > Set Identity**. Enter your name and email, then save them for the current repository. Enable **Save globally** only if you want the same identity to become your user-wide Git default. If identity is missing when you commit, Blender Git shows a friendly reminder instead of Git's raw command error.

<img width="789" height="739" alt="image" src="https://github.com/user-attachments/assets/38957204-700f-426a-929f-04d328f6f4b8" />

---

## Getting Started

### Starting a new project

1. Save your `.blend` file to the folder where you want the project to live.
2. Open the Git panel. It will prompt: **"No Git repository found."**
3. Click **Initialize Repository**.

[example-01-initialize-repository.webm](https://github.com/user-attachments/assets/a6639116-7e18-4b4c-9b24-18c8edd64894)

The addon creates a Git repository in the same folder as your .blend file. If `.gitignore` or `.gitattributes` files already exist, it checks them and appends any missing required entries automatically. It then makes the first commit. You are ready to work.

<img width="767" height="731" alt="image" src="https://github.com/user-attachments/assets/0a9adaa2-45f5-446a-abd9-098582c78edd" />

### Joining an existing project

If someone has already set up a project on a remote (GitHub, GitLab, etc.):

1. Open the Git panel before opening any .blend file.
2. Click **Clone it here** (visible when no repository is detected).
3. Enter the repository URL and choose a destination folder.
4. After cloning, open your .blend file from that folder via **File > Open**.

[example-03-clone-project.webm](https://github.com/user-attachments/assets/c97df6e0-ea36-493d-8ed3-7ed3c753843d)

If you already have a local copy of a Git repository (cloned elsewhere), simply open a .blend file from inside that folder and the panel will detect the existing repository automatically.

---

## How It Works

### Git tracks your whole project folder

When you initialize a repository, Git begins watching every file in that folder and its subfolders. This includes your .blend files, textures, references, and any other assets. You decide which changes to record (commit) and when.

### Git LFS handles large binary files

.blend files are binary. Git LFS stores them in a dedicated object store and places a small text pointer in the Git history. From your perspective this is invisible. You stage, commit, and switch branches the same way. LFS just ensures your repository stays fast regardless of how many versions accumulate.

### Scene Snapshots power the change descriptions

Each time you stage a .blend file, the addon captures a snapshot of the current scene state and stages it alongside the .blend. This snapshot is a small JSON file stored under `.blender_git/snapshots/` inside your repository.

When the panel displays what changed in a .blend file, it compares the current snapshot against the last committed snapshot and generates the human-readable change list you see on screen.

Snapshots normally travel with their matching `.blend` file. When you stage, unstage, discard, commit, merge, or resolve a `.blend` conflict, Blender Git also checks the matching snapshot so the two do not drift apart.

---

## Scene Snapshots

### What gets recorded

<img width="461" height="364" alt="image" src="https://github.com/user-attachments/assets/25658241-50db-4a72-9611-ab849f64b963" />

Every snapshot captures the state of the open .blend file at the moment of staging:

**Objects**
- Whether each object was added, removed, or modified
- Transform: position, rotation, scale
- Viewport visibility and render visibility
- Parent-child relationships
- Collection membership
- Applied modifiers (types)
- Material slot assignments
- Shape keys
- Vertex groups
- Constraints
- Particle systems
- Rigid body physics

**Per object type**
- Meshes: vertex count, face count, UV map count
- Armatures: bone count
- Cameras: focal length, camera type (perspective / orthographic / panoramic)
- Lights: energy, type (point / sun / spot / area), color
- Text objects: character count

**Animation**
- Keyframe count per object
- Number of animated properties (F-Curves)
- NLA tracks
- Drivers

**Materials**
- Whether each material was added or removed
- Shader node count
- Texture image node count
- Transparency mode

**Collections**
- Objects inside each collection
- Sub-collection structure

**Scene settings**
- Frame range (start / end)
- Frames per second
- Render engine
- Render resolution (width and height)
- Render samples (Cycles and EEVEE)
- Active camera
- World environment
- Render output format

**Other**
- Linked library file paths
- Externally referenced image names

### Snapshot Rules

**Snapshots are staged automatically.** When you stage a .blend file, its snapshot is captured and staged alongside it. If you delete a .blend file, the matching snapshot deletion is staged with it.

**Snapshots must be committed.** The snapshot file is what the panel uses as the "before" state the next time you make changes. If you stage a .blend but do not commit, the snapshot is not saved to the repository history and the change view may be incomplete.

**Orphan snapshots are shown when they matter.** The .blender_git/snapshots/ folder holds one JSON file per .blend file, mirroring the folder structure of your project. If a snapshot exists but the matching .blend file does not, Blender Git shows it as Snapshot for <file>.blend (orphan). Expand it to inspect what scene data it contains, then stage or discard it like any other change.

**Do not edit snapshot files by hand.** Treat snapshot files as project metadata managed by the addon. If one is missing, stale, or orphaned, use the panel's stage, discard, or conflict controls to resolve it.

**The `.blender_git/` folder must not be in your `.gitignore`. Do not edit `.blender_git/`** The addon automatically excludes only one file inside it (session state that should not be committed). Everything else in `.blender_git/`, particularly the `snapshots/` subfolder, must be tracked and committed. Do not add the folder to `.gitignore` yourself.

**Multi-file projects.** If your project contains more than one .blend file, each gets its own snapshot file. The change view works for all of them, not just the one currently open.

---

## Panel

### Toolbar

Located at the top of the Git panel:

- **Home**: Opens the repository root folder in your system's file explorer.
- **CMD**: Opens a terminal window at the repository root. Useful for operations not in the panel, such as force-pushing or more advanced Git commands.
- **Configuration**: Opens a menu with access to identity, remote settings, .gitignore editor, .gitattributes editor, and credentials info.
- **Refresh**: Manually re-checks the repository status. The panel updates automatically on most actions, but Refresh is available if something appears out of sync.

### Save required state

If the current .blend file has unsaved edits, Blender Git pauses the normal workflow until you save. Git actions use the version of the file that exists on disk, so saving first keeps the change list, snapshot, and commit content aligned.

The panel shows a warning at the bottom of the main section with a **Save File** button. After saving, the normal branch, staging, commit, stash, history, and conflict actions become available again.

### Branch and remote status

Below the toolbar, a box shows:

- The **current branch name**
- The **remote name** (e.g. origin) and its URL, if configured
- Whether you are **ahead** (commits not yet pushed), **behind** (commits on the remote not yet pulled), or **up to date**

### Commit section

- **Commit message field**: Type a description of what you did. A message is required to commit.
- **Commit Staged**: Records the staged changes as a new commit. Only staged files are included.
- **Stash All**: Saves all current changes (staged and unstaged) into a temporary stash, leaving the working tree clean. The current commit message is pre-filled as the stash label.

### Staged Changes panel

Shows files that have been staged and are ready to be committed.

- Each file row shows a status indicator and the filename.
- For .blend files that have a snapshot, a **chevron** button expands a scrollable list of what changed in plain English, grouped by category (objects, materials, collections, scene, etc.).
- Snapshot-only rows are shown when needed. If the matching .blend file is missing, the row is labelled `Snapshot for <file>.blend (orphan)`, includes an info icon, and can be expanded to inspect the captured scene data.
- Click the **minus** button on any file or folder to unstage it.
- **Unstage All** moves everything back to the Changes panel.

### Changes panel

Shows files that have been modified but not yet staged, plus any new (untracked) files.

- Files are displayed in a folder tree. Click a folder name to expand or collapse it.
- **Plus button** on a file or folder stages it (and auto-captures the snapshot for .blend files).
- **Trash button** on a file or folder discards all changes and reverts to the last committed state. This cannot be undone.
- Orphan snapshot rows can be staged or discarded directly, so hidden snapshot metadata cannot leave the branch dirty while the panel appears clean.
- **Stage All** and **Discard All** buttons apply to all visible changes at once. After **Discard All**, the raw Git worktree should be clean, including managed snapshot paths.
- The **Blender icon** on a .blend file row opens that file in a new Blender window.

### Stashes panel

Collapsed by default. Click the header to expand.

- Lists all saved stashes with their label and index.
- **Apply**: Restores a stash's changes to the working tree and reopens the .blend file. Only available when the working tree is clean (no current staged or unstaged changes).
- **Drop**: Permanently deletes a stash entry. A confirmation dialog appears first.

---

## Branch Management

Access branch actions via the **Branch Actions** button next to the current branch name.

- **New Branch**: Creates a new branch from the current state and switches to it immediately. Staged, unstaged, and untracked changes remain in place.
- **Change Branch**: Switch to a different local branch, or toggle "Show Remote" to select a remote branch and check it out locally. Local changes are preserved; if conflicts occur, use the conflict banner and popup to resolve them. If the open `.blend` file does not exist on the target branch, Blender opens a new unsaved General file.
- **Merge Branch**: Merges the selected branch into the current branch. If a conflict is detected, the conflict banner appears.
- **Fetch**: Downloads new commits and branch references from the remote without modifying local files. Shown only when a remote is configured.
- **Pull**: Downloads and applies remote commits to the current branch. Shown only when the current branch has a remote tracking branch.
- **Push**: Uploads your local commits to the remote tracking branch. Shown only when the current branch has a remote tracking branch.
- **Delete Branch**: Deletes a local branch. Protected branch names (main, master, develop, dev) cannot be deleted. An option to also delete the corresponding remote tracking branch is shown if one exists. This action cannot be undone.
- **History**: Opens the commit history view.

---

## Commit History

Accessed via Branch Actions > History.

The history panel lists commits in reverse chronological order, showing the short hash, message, author, and date.

Commit details use the same snapshot language as the Changes panels. Snapshot-only entries are visible, and orphan snapshots are labelled with `(orphan)` and can be expanded to inspect their captured scene data.

From any commit entry:

- **Checkout to New Branch**: Prompts for a branch name, then creates a branch starting at that commit and switches to it. Requires a clean working tree. If the current `.blend` does not exist on the new branch, Blender opens a new unsaved General file.

---

## Merge Conflict Resolution

When a merge produces a conflict (two branches modified the same file in incompatible ways), the normal Changes and Commit panels are replaced by a conflict banner showing which branches are involved and how many files need resolution.

Click **Resolve Conflicts** to open the conflict dialog.

For each conflicted .blend file:

- **Open Ours in Blender**: Opens the version from your current branch in a new Blender window so you can inspect it.
- **Open Theirs in Blender**: Opens the incoming version in a new Blender window.
- **Keep Ours**: Marks this file to use your current branch's version.
- **Keep Theirs**: Marks this file to use the incoming version.

While a `.blend` file is still in conflict, do not open that same file normally from Blender's file browser or recent files list. Use **Open Ours in Blender** and **Open Theirs in Blender** to inspect the two versions, or open a different `.blend` file instead. Reopen the conflicted file normally only after the conflict has been resolved.

Because .blend files are binary, there is no partial merge. You choose one complete version per file.

Snapshot conflicts follow the same rule. When you keep ours or theirs for a conflicted `.blend`, Blender Git also keeps the corresponding snapshot from the same side. If Git reports a snapshot conflict without a matching `.blend` conflict, the snapshot appears as its own row. Orphan snapshot conflicts are labelled with `(orphan)`, include an info icon, and must be resolved before the merge can be completed.

The conflict dialog also shows managed snapshot-only changes in the **Ours** and **Theirs** lists, so a merge, stash apply, or checkout cannot look clean while a hidden snapshot still changed.

Once all conflicted files have a choice assigned, click **Complete Merge** to finalize and create the merge commit.

If you want to abandon the merge entirely, click **Restore to Latest Commit** (Abort Merge) from the conflict banner. The repository returns to the state before the merge started.

---

## Remote and Configuration

### Identity

Open **Configuration > Set Identity** to set the author name and email used for commits. By default, the values are saved only to the current repository. Turn on **Save globally** to write them to your user-wide Git config instead.

### Remote

Open **Configuration > Configure Remote** to:

- Add or change the remote URL (e.g. a GitHub or GitLab repository)
- View the current tracking branch

After adding a remote, use **Fetch** to retrieve its branches, **Pull** to bring in the latest commits, and **Push** to upload your commits to the remote.

### .gitignore editor

Lists the current .gitignore entries. Add patterns for files or folders you do not want Git to track (e.g. temporary render output, cache folders). The addon pre-populates sensible defaults for Blender projects on initialization.

### .gitattributes editor

Controls how Git treats specific file types, including which formats are stored via Git LFS. The defaults cover .blend and common image and video formats. Edit only if you need to add custom file types.

---

## Typical Workflows

### Daily work as a solo user

1. Work on your scene.
2. Save the .blend file (Ctrl+S).
3. Switch to the Git panel and review the Changes list.
4. Click the plus button on your .blend file to stage it (the snapshot is captured automatically).
5. Type a commit message describing what you did.
6. Click **Commit Staged**.

Repeat as often as you like. Each commit is a restore point you can return to at any time.

### Saving a work-in-progress state without committing

Use a stash when you need a clean slate temporarily. To test something else, switch context, or just save your in-progress state without making an official commit.

1. Type a short description in the commit message field (it will become the stash label).
2. Click **Stash All**.
3. Your changes are saved and the working tree goes clean.
4. When ready, open the Stashes panel, find the stash, and click **Apply**.

### Trying out an experiment on a separate branch

1. Make sure your current work is committed (or stashed).
2. Branch Actions > **New Branch** and give it a name like "experiment-retopo".
3. Work and commit freely on this branch.
4. If the experiment works: Branch Actions > **Merge Branch** back into your main branch.
5. If it does not work: switch back to your main branch and delete the experiment branch.

### Switching to an earlier version of the project

1. Branch Actions > **History**.
2. Find the commit that represents the state you want.
3. Click **Checkout to New Branch**, give it a name.
4. The .blend file reopens at that earlier state.
5. From here you can keep working on this branch or use it purely for reference.

### Collaborating with a team

**Setting up:**
1. One person initializes the repository and connects a remote (GitHub, GitLab, or any Git host).
2. They push the initial commit using Branch Actions > **Push**.
3. Team members open the Git panel in Blender and use **Clone it here** to get a local copy.

**Day-to-day:**
1. Each person works on their own branch.
2. Use **Fetch** to check for updates from the remote.
3. Use **Pull** to bring in the latest commits.
4. When your branch is ready, use **Push** from the Branch Actions menu (or the branch info box) to upload it, then open a pull request on your hosting platform.
5. After a merge, pull the result.

---

## Example: Two-Person GitHub Collaboration

This example follows two users working on the same Blender project through GitHub.

**Setup**
- **User A** starts the project and uploads it.
- **User B** clones it, creates a branch, commits work, and merges it back.
- Both users have Git, Git LFS, Blender Git, and access to the same GitHub repository.

### 1. User A initializes the project

1. Save the `.blend` file inside a clean project folder.
2. Open the **Git** tab.
3. Click **Initialize Repository**.
4. Blender Git creates the repository, configures Git LFS, prepares `.gitignore` and `.gitattributes`, captures the first snapshot, and creates the first commit.

[example-01-initialize-repository.webm](https://github.com/user-attachments/assets/2287efdf-fb30-4fba-a8b1-cb70c43cc25c)

### 2. User A uploads to GitHub

1. Create an empty GitHub repository.
2. Open **Configuration > Configure Remote**.
3. Add the GitHub repository URL.
4. Click Branch Actions > **Push** for the first upload.
5. The branch is now connected to the remote.

[example-02-remote-settings.webm](https://github.com/user-attachments/assets/a74feea6-fc99-45ad-b7a7-a77f9c4b471e)

### 3. User B clones the project

1. Open Blender before opening any `.blend` file.
2. Open the **Git** tab.
3. Click **Clone it here**.
4. Enter the GitHub repository URL and choose a local folder.
5. Open the cloned `.blend` file from that folder.

[example-03-clone-project.webm](https://github.com/user-attachments/assets/f214d827-46a6-468f-a879-aeabd7e0d76e)

### 4. User B creates a branch and commits work

1. Open **Branch Actions > New Branch**.
2. Create custom branch.
3. Confirm the branch name changed in the branch status box.
4. Edit file.
2. Save the `.blend` file.
3. In **Changes**, stage the `.blend` file.
4. Expand the staged file to review the readable scene diff.
6. Click **Commit Staged**.
7. Pushes branch to remote by clicking on **Branch Actions > Push**.

Also, at this point on GitHub, a pull request can open from the custom branch that was created into `master`.

[example-04-new-branch.webm](https://github.com/user-attachments/assets/988bdc0a-926c-4042-9948-076bdc41e81d)

### 5. User B stashes temporary work

1. Start an experiment that is not ready to commit.
2. Type a short stash label in the commit message field.
3. Click **Stash All**.
4. Later, open **Stashes** and click **Apply** to restore it.

[example-08-stashes-panel.webm](https://github.com/user-attachments/assets/7eaceb7b-dece-402d-8749-4a2b2fedc5a6)

### 6. User B checks history

1. Open **Branch Actions > History**.
2. Review commits by message, author, date, and hash.
3. Use **Checkout to New Branch** to continue from an old version, in a new branch.

[example-09-history-view.webm](https://github.com/user-attachments/assets/f5a1f715-f6e1-4992-84d5-ca983b789323)

### 7. User B merges into master their custom branch changes

1. Check-out to master branch by clicking on **Branch Actions > Change Branch**, and select `master` branch.
1. Click **Merge Branch**. Click `show remote` option to display the remote branches. Click the work branch that contains the changes.
2. Merge is complete. `master` branch now contains changes from the working branch and is up to date.

[example-10-github-pull-request.webm](https://github.com/user-attachments/assets/b21e8300-6af5-4d31-9bec-5f274f4d0772)

### 8. User B merges into master their custom branch changes but conflict occurs

1. Both users change the same `.blend` file on different branches.
2. During merge, Blender Git shows the conflict banner.
3. Click **Resolve Conflicts**.
4. Open **Ours** and **Theirs** to inspect both versions.
5. Do not reopen the conflicted `.blend` normally until the conflict is resolved. Current `.blend` file cannot be reopened until conflicts are resolved. In case this happens, open Blender in this repo from a different `.blend`, that isn't included in the conflict.
6. Choose **Keep Ours** or **Keep Theirs** for each conflicted `.blend`. Its companion snapshot follows the same choice when one exists.
7. Resolve any snapshot-only conflict rows as well (if exist), including orphan snapshot rows marked with `(orphan)`.
8. Click **Complete Merge**.

Because `.blend` files are binary, conflicts are resolved per file. You choose one complete version of the file.

[example-11-resolve-conflicts.webm](https://github.com/user-attachments/assets/46e8e99e-1ae5-4802-91fd-57b670fe513d)

---

## Limitations

**Conflict resolution is all-or-nothing per file.** Because .blend files are binary, you cannot combine parts of two versions together. You choose one complete version of each conflicted file.

**Conflicted .blend files should not be reopened directly before resolution.** While a `.blend` file is unresolved, opening it normally can disturb the binary conflict state. Use the conflict dialog's inspection buttons, resolve the conflict, then open the chosen result normally.

**Staging, unstaging, and discarding apply to entire files.** Because .blend files are binary, there is no way (currently) to include or exclude individual changes within a file. Every staging, unstaging, or discard operation affects the whole file at once.

**Git & Git LFS are required.** Git LFS is not bundled with Git on most platforms and must be installed separately from git-lfs.com before initializing a repository. The addon will refuse to initialize without it.

**Blender 5.x only.** The addon is built for Blender 5.x and may not function correctly on earlier versions. Earlier versions are not currently tested.

---

## Q&A

**Do I need to know Git to use this?**
No. Everything you need for daily version control, like staging, committing, branching, fetching, pulling, pushing, and viewing history, is available as buttons in the panel.

**What is Git LFS and do I really need it?**
Git LFS (Large File Storage) stores large binary files like .blend and textures in a separate system so your repository history stays small and fast. Without it, every commit that touches a large .blend would duplicate its full size in history. Git LFS is **required** by this addon, so it must be installed from git-lfs.com before you can initialize a repository. It is a free, one-time install.

**Will this work with GitHub, GitLab, Bitbucket, or Gitea?**
Yes. Any standard Git remote is compatible. You can clone, fetch, pull, and push from within the panel.

**I don't see a Git tab in my sidebar.**
Make sure the addon is enabled in Edit > Preferences > Add-ons. The Git tab lives in the 3D Viewport's N-panel. Press N to toggle it. If you are in a workspace that does not have a 3D Viewport (such as the Video Editing workspace), switch to a layout that includes one.

**The panel says "No Git repository found." but I have a repository.**
The addon detects the repository based on the location of the currently open .blend file. If no .blend is open, or if the .blend is not inside the repository folder (or any of its subfolders), the panel cannot find it. Open a .blend file from within the repository and the panel will connect automatically.

**What happens if a snapshot is missing or orphaned?**
If a snapshot is missing, the matching `.blend` may show `no snapshot` until a new staged commit re-establishes the baseline. If a snapshot exists without its `.blend`, Blender Git shows it as an orphan snapshot row so you can stage or discard it instead of being left with a hidden dirty file.

**Can multiple people work on the same .blend file at the same time?**
Not simultaneously on the same file, that is currently a fundamental limitation with binary files. Teams should divide work so each person is primarily responsible for different .blend files or branches, and merge at natural milestones.

**The change list shows "no snapshot" for a file I have changed many times.**
The file was first committed before the addon was installed (or when the addon was first set up), so no baseline snapshot exists in the history. Stage the file and commit once, the panel will capture a snapshot and from that point on the change details will appear normally.

**Is my file safe when I switch branches?**
Yes, as long as you save before switching. If the current .blend has unsaved edits, Blender Git disables branch switching and shows a Save File button first. After saving, switching branches reopens the .blend from the selected branch when that file exists there. If it does not exist on the selected branch, Blender opens a new unsaved General file so you can choose another .blend manually.

**What are "protected branches" and which ones are protected?**
Protected branches are branches that cannot be deleted from within the panel as a safeguard against accidental loss. The protected names are: **main**, **master**, **develop**, and **dev**. You can still delete other branches freely; and protected branches can always be deleted via a terminal if genuinely needed.

**Can I push directly from within the Blender Git panel?**
Yes. The **Push** button appears in the branch info box and in the Branch Actions menu whenever the current branch has a remote tracking branch. For advanced push options (force push, specific remote, etc.) use the **CMD** button to open a terminal.

**What happens to my snapshots when I rename a .blend file?**
The addon automatically moves the companion snapshot to match the new filename and path whenever a rename is detected. You do not need to do anything manually. Currently, filename change detection works only when you rename a file with **Save As** in Blender. If files, folders, or snapshots are edited outside Blender, orphaned snapshots may appear because the addon cannot match them back to their original files. If this happens, discard the orphaned snapshots in the next staging pass.

**Can I use this with a project that has many .blend files in subfolders?**
Yes. The repository tracks the entire folder tree. Each .blend file gets its own snapshot in a mirrored subfolder structure under `.blender_git/snapshots/`. The change view works for all of them.
