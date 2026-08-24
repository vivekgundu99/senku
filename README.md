# Senku

**A quiet workspace and task launcher for one master folder.**

Senku is a local Windows desktop app (Electron + TypeScript) that pairs a live file browser with one-time, text-first tasks. It does not organize, copy, move, or sync your files — it only remembers paths, tasks, and preferences, and gives you a fast way to see and open the things you already have.

Senku is a manager and index of user-provided paths, not a file-sync or backup tool.

## Core idea

1. Point a workspace at one folder on disk (the **master folder**).
2. Browse that folder inside Senku, exactly as it exists in Windows.
3. Create tasks — plain text reminders, optionally linked to files, folders, URLs, playlists, or repos.
4. Senku notifies you when a task starts, and can open its linked resources for you.
5. Mark tasks done, snooze them, or reassign partial progress to a new schedule.

Everything Senku remembers (workspaces, tasks, settings) lives in local JSON files, separate from your real files. Deleting a workspace in Senku never touches the Windows folder behind it.

## Features (as implemented)

- Multiple isolated workspaces, each with one master folder and independent tasks/settings
- Live file browser scoped strictly to the current master folder (no path escapes)
- Opens files and folders with their normal Windows-associated app
- Rename files/folders inside the master folder directly from Senku (the real file is renamed — nothing is copied)
- Copy-path button for any file or folder
- Text-first tasks with start time, end time (defaults to 10 years out), and Normal/Urgent priority
- Optional linked paths per task, one per line, inside or outside the master folder
- Per-workspace **Strict Mode** toggle, plus Urgent tasks that always open their links regardless of Strict Mode (see below)
- Windows toast notification at every task's start time, regardless of Strict Mode
- Snooze for 10 minutes, 1 hour, 1 day, or 1 week
- Manual completion via **Complete** or the row checkbox
- **Reassign**: saves current progress to Completed and immediately opens a dialog for the next schedule
- Multi-select + **Complete selected** for batch-finishing active tasks
- Overdue view (active tasks past their end time)
- Completed view with Today / Last week / All filters
- Error view for tasks whose links failed to open, showing the actual failure reason, with a **Recover** action to fix links and restore them to Active
- Workspace settings modal for renaming the current workspace
- Profile modal for display name, Strict Mode, and "launch at Windows startup" — all of which are actually applied
- Missed-start handling: if Senku wasn't running at a task's start time, the next engine tick (≤1 minute after launch) fires it once
- Runs in the system tray with a proper app icon; task engine polls once per minute with no idle folder scanning
- Local JSON persistence, no database

## Requirements

- Windows 10 or later
- Node.js 20+ and npm
- Network access during `npm install` so Electron's Windows binary can download

## Install and run

```powershell
npm install
npm run dev
```

`dev` compiles the Electron main process and preload bridge with `tsc`, then starts Vite and Electron together.

Production build (runs from `dist/`, no dev server):

```powershell
npm run build
npm start
```

Windows installer build:

```powershell
npm run package:win
```

This uses `assets/senku.ico` as the installer and app icon — already present in this repo.

If Electron reports it wasn't installed correctly, confirm this file exists:

```text
node_modules\electron\dist\electron.exe
```

If it's missing, delete `node_modules` and reinstall on a network that allows Electron's binary download.

## First launch

1. Enter your name and a workspace name (e.g. `BTech 24-28`).
2. Paste the master folder path, or pick it with **Browse**.
3. Select **Create workspace**.

The workspace name is a label inside Senku only — the real Windows folder is never renamed or moved.

## Workspaces

Open the workspace switcher in the sidebar (the folder-name button under the Senku logo) to switch workspaces, create a new one, or delete the current one.

Use **⚙ Workspace settings** in the sidebar to rename the current workspace at any time.

Deleting a workspace removes its Senku task metadata only; the master folder and its contents are untouched. Each workspace keeps its own active/completed/error tasks, Strict Mode setting, and last-opened subfolder.

## Tasks

Create a task from the **+** button in the tasks panel. A task has:

- A title (plain text)
- Start time and end time (past start times are allowed; end defaults to 10 years after start)
- Priority: Normal or Urgent
- Optional linked paths, one per line — files, folders, URLs, playlists, repos, anything Windows can open

A task needs no linked paths at all; it stays a plain reminder either way.

### Task actions

| Action | What it does |
|---|---|
| **Done** / checkbox | Marks an active task Completed |
| **Snooze** | Adds 10m / 1h / 1d / 1w to the start time and re-arms the notification |
| **Reassign** | Archives current progress to Completed, then opens a dialog to schedule the remaining work as a new active task |
| **Edit** | Changes title, times, priority, or links on an active or error task |
| **Recover** | Edits an Error task's links and moves it back to Active, re-arming its start event |
| **Delete** | Removes the task's Senku metadata only — never touches the linked file/folder |

Opening a file never marks a task complete automatically — completion is always manual.

## Strict Mode and Urgent tasks

- **Every task** (Normal or Urgent) sends a Windows notification at its start time, unconditionally.
- **Strict Mode is a per-workspace switch.** When it's on, any Normal task with linked paths has all of its links opened automatically at start time.
- **Urgent tasks always open their linked paths at start time, regardless of the workspace's Strict Mode setting.** Strict Mode only controls whether *Normal* tasks also get this behavior.
- When Strict Mode is off and a task is Normal, links are never opened automatically — you open them manually from the task (or via the "Open resources" button when the task has links).

If a linked path can't be opened for a task that should open it, that task moves to **Error**, keeping its title, schedule, priority, and links — including the one that failed, with the actual failure reason shown in the task card so you know what to fix. Recover it once the link is corrected.

## Missed start times

On launch, the task engine checks every workspace for active tasks whose start time has already passed and fires each one's notification (and link-opening, if applicable) exactly once, using a per-task marker so nothing double-fires. This runs on the same one-minute timer used during normal operation.

A task stays Active until you complete or delete it. If its end time passes first, it shows up in Overdue instead.

## File browser

The browser only ever shows the current workspace's master folder and its descendants — task links outside the master folder never appear here, even if a task references them. Double-click a folder to navigate into it; double-click a file to open it with its Windows-associated app.

## Profile and startup behavior

Open the profile modal from the avatar button (top right) to change:

- Display name
- Strict Mode for the current workspace
- **Launch at Windows startup** — toggling this is saved immediately and actually applied via Electron's login-item settings, both on save and on the next app launch.

## Local data

Senku stores everything as JSON under Electron's per-app `userData` directory. Because that path is derived from the app's name, it differs between a dev run and a packaged install:

```text
Running via `npm run dev` / `npm start`:
  %APPDATA%\senku-v1\data\

Running the packaged/installed app (productName "Senku"):
  %APPDATA%\Senku\data\
```

Inside that `data` folder:

```text
settings.json                              user name, active workspace, startup preference
workspaces.json                            workspace names, master folders, per-workspace settings
workspaces\<workspace-id>\active-tasks.json
workspaces\<workspace-id>\completed-tasks.json
workspaces\<workspace-id>\error-tasks.json
```

Writes go through a temp-file-then-rename so an interrupted write shouldn't corrupt existing data. Deleting the `data` folder resets all Senku metadata; it never touches your real files.

## Project structure

```text
electron\main.ts        Main process: filesystem, workspaces, task engine, notifications, tray, IPC handlers
electron\preload.ts     Typed, restricted renderer bridge (contextBridge, no raw Node/fs access)
electron\types.ts       Shared TypeScript models (Settings, Workspace, Task, TaskInput, AppSnapshot, FileEntry)
src\renderer.ts         All UI logic: views, modals, task/file rendering, event handling
src\index.html          Renderer entry point
src\style.css           Application styling
src\env.d.ts            Global typing for the window.senku bridge
vite.config.ts          Renderer build config (outputs to dist\renderer)
tsconfig.json           Compiles electron\**\*.ts and src\**\*.ts
assets\senku.ico        App/installer icon (16–256px, Windows .ico)
assets\Senku-User-Guide.pdf   In-app user guide, opened from the profile modal
SRS.md                  Software requirements specification
README.md               This file
```

## Packaging a `.exe`

`npm run package:win` runs `electron-builder --win` using the `build` config in `package.json`. `assets/senku.ico` and `assets/Senku-User-Guide.pdf` are both present, so:

- The Windows installer build has a real icon (16, 32, 48, 64, 128, 256 px embedded).
- The in-app **"Read the Senku guide"** button (profile modal) opens an actual PDF instead of erroring.
- The system tray icon is the same mark, resized to 16×16, instead of a blank icon.

## Prototype scope (intentionally out of v1)

AI categorization or recommendations, recurring tasks, checklists/subtasks, task dependencies, automatic task completion, cloud sync, a database backend, and any folder creation/deletion from within Senku are all deliberately outside this version.