# Senku v1.1
# A quiet workspace for college students

**You are the fire. Senku is the spark. 🔥**

Senku is a Windows desktop workspace for college students. It helps students keep academic folders, notes, links, projects, and study tasks close together without changing the real files on disk.

Senku is a reminder and launchpad, not an automatic organizer. Students still choose the resources and do the work; Senku keeps the context ready.

It gives each master folder its own workspace and combines a live file browser with one-time tasks, reminders, and optional linked resources. It remembers the active workspace and the last subfolder you had open, so you return to exactly where you left off.

Senku never copies, moves, creates, or deletes the user's files. It stores only workspace settings, task metadata, and links in the Electron AppData directory.

## A useful student workflow

Create a workspace for a semester, project, placement preparation, or any other area of college life:

- Use **one workspace** for everything (e.g. `BTech 24-28`).
- Keep **one active task** at a time.
- Link that task to a single Word document (`.docx`).
- Inside that one Word document, add anything you need: images, hyperlinks, notes, to-dos, links to playlists, repos, folders, whatever the work requires. The document itself becomes your real workspace; Senku just points you to it.
- Work through the day and update the Word document directly as you go.
- Add tasks for assignments, workshops, revision sessions, interviews, or personal commitments.
- Link notes, PDFs, Word files, playlists, folders, or repositories to a task.
- Use **Complete** when the work is finished. Use **Reassign** when you made partial progress and need a new schedule.
- Keep Strict Mode off for reminders only, or turn it on when task resources should open automatically.

Senku was made with a Navodayan spirit: a small tool for making the next step easier to begin.

## Features

- Electron + TypeScript Windows app
- Multiple isolated workspaces
- One master folder per workspace
- User-defined name and workspace name
- Remembers the active workspace and last opened subfolder
- Live browsing of files and folders inside the master folder
- Opens files with their normal Windows application
- Text-first one-time tasks
- Start time and end time for every task
- Past start times are allowed
- Default end time is ten years after start
- Normal and Urgent task priorities
- Optional links to files, folders, URLs, playlists, and repositories
- Windows reminders at task start
- Strict Mode per workspace
- Strict Mode opens all linked resources when a task starts
- Snooze for 10 minutes, 1 hour, 1 day, or 1 week
- Manual completion using Done or the checkbox
- Complete and reschedule
- Edit active tasks
- Overdue task view
- Completed task filters: Today, Last week, All
- Error tasks that preserve failed links and task data
- Recover error tasks after fixing their links
- Tray startup support
- Local JSON persistence with no database

## Requirements

- Windows 10 or later
- Node.js 20 or later recommended
- npm
- Electron's Windows binary must be available during installation

## Install and run

Open a terminal in this folder:

```powershell
npm.cmd install
npm.cmd run dev
```

The `dev` script compiles the Electron main process and preload bridge before starting Vite and Electron.

For a production build:

```powershell
npm.cmd run build
npm.cmd start
```

If Electron reports that it was not installed correctly, check that this file exists:

```text
node_modules\electron\dist\electron.exe
```

If it is missing, remove `node_modules` and run `npm.cmd install` again while connected to a network that allows Electron's binary download.

## First launch

1. Enter your name.
2. Enter a workspace name, such as `Workspace 1` or `BTech 24-28`.
3. Paste the master folder path or choose it with Browse.
4. Select Create workspace.

A workspace is only a Senku label around a real Windows folder. The folder is not renamed or moved.

## Workspaces

Use the workspace selector in the sidebar to:

- Switch workspaces
- Create another workspace
- Delete the current workspace

Deleting a workspace deletes its Senku task metadata only. The Windows master folder and its contents remain untouched.

Each workspace has independent:

- Active tasks
- Completed tasks
- Error tasks
- Strict Mode setting
- Last opened subfolder

## Tasks

Create a task with:

- Task name
- Start time
- End time
- Normal or Urgent priority
- Optional linked paths, one per line

Tasks do not need linked resources. A task can link to resources inside or outside the master folder, including URLs.

### Task actions

- **Done:** moves the task to Completed.
- **Checkbox:** also moves an active task to Completed.
- **Snooze:** shifts its start time by the selected duration and allows it to run again.
- **Reassign:** saves partial progress in Completed and creates the next schedule.
- **Edit:** changes the task name and linked paths.
- **Recover:** edits an Error task and returns it to Active.

Tasks are not completed automatically when a file opens.

## Strict Mode and opening resources

- All tasks send a notification at their start time.
- Strict Mode opens all linked resources at the start time.
- When Strict Mode is off, linked resources can be opened manually from the task.
- A task with no linked resources remains a reminder.

## Missed start times

When Senku starts, the task engine checks all workspaces for active tasks whose start time has passed. It processes each due task sequentially and stores a start-event marker so every active task is run only once.

A task remains active until the user completes or deletes it. If its end time passes first, it appears in Overdue.

If a linked resource cannot be opened for a task that should open resources, the task moves to Error. Its original task structure and failed links are preserved. Edit or Recover the task after correcting the resource, then it returns to Active and can run again.

## File browser

The file browser displays only the current workspace's master folder and its descendants. It does not display external task links. Double-click a folder to navigate. Double-click a file to open it with Windows.

## Local data

Senku stores JSON data under the Electron user-data directory, normally:

```text
%APPDATA%\senku-v1\data
```

The data includes:

```text
settings.json
workspaces.json
workspaces\<workspace-id>\active-tasks.json
workspaces\<workspace-id>\completed-tasks.json
workspaces\<workspace-id>\error-tasks.json
```

Deleting these files resets Senku's metadata but does not delete user files.

## Project structure

```text
electron\main.ts       Electron main process, filesystem, tasks, notifications
electron\preload.ts    Typed, restricted renderer bridge
electron\types.ts      Shared TypeScript models
src\renderer.ts        Workspace and task UI logic
src\style.css          Application styling
SRS.md                 Software requirements specification
README.md              User and developer guide
```

## Prototype scope

AI features, recurring tasks, cloud sync, database storage, automatic file organization, folder creation, and folder deletion are intentionally outside v1.