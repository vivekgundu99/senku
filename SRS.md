# Senku v1
## Software Requirements Specification

**Product:** Senku
**Version:** 1.0
**Platform:** Windows desktop
**Technology direction:** Electron + TypeScript
**Persistence:** Local files in the user's AppData directory
**Database:** None

## 1. Purpose

Senku is a local Windows application for BTech students who need to manage academic folders and schedule practical, file-linked tasks. It provides a more useful view of an existing folder structure and connects reminders to documents, notes, playlists, repositories, and other resources.

Senku is a manager and index of user-provided paths. It is not a file synchronization, backup, or file-moving application.

## 2. Goals

1. Remember multiple isolated workspaces, each with one user-selected master folder.
2. Browse the current Windows file structure inside the master folder.
3. Open files and folders using normal Windows behavior.
4. Create one-time text tasks with optional linked paths.
5. Notify users when tasks start.
6. Support normal reminders and urgent strict tasks.
7. Preserve completed task history with completion timelines and filters.
8. Run quietly at Windows startup with low resource usage.
9. Keep all Senku metadata in local files, separate from managed files.

## 3. Non-goals for v1

- No AI categorization or recommendations.
- No database server or cloud synchronization.
- No copying, moving, or uploading user files.
- No automatic file organization.
- No creation or deletion of folders from Senku.
- No recurring tasks.
- No checklist or subtask system.
- No automatic task completion.
- No task dependencies.

## 4. Terminology

| Term | Meaning |
|---|---|
| Master folder | The single root folder selected by the user, for example `C:\Users\Vivek\btech24-28`. |
| Managed browsing | Displaying the current Windows file structure beneath the master folder. |
| Linked path | A file, folder, URL, playlist, repository, or other path attached to a task. |
| Normal task | A task that produces a reminder only. |
| Urgent task | A strict task. At its start time it attempts to open every linked path. |
| Active task | A task that has not been completed, deleted, or moved to an error state. |
| Error task | A strict task whose linked resource could not be found or opened. |

## 5. Users and primary workflow

### 5.1 First launch

1. Senku opens a setup view because no master folder is configured.
2. The user pastes a Windows folder path.
3. Senku validates that the path exists and is a directory.
4. The user enters their name and a workspace name such as `Workspace 1`.
5. Senku saves the workspace and master folder.
6. Senku displays the folder contents and task area.

### 5.2 Later launch

1. Senku starts with Windows in the system tray.
2. The task engine runs in the background.
3. When the user opens Senku, it restores the saved master folder.
4. It restores the last visited subfolder inside that master folder.
5. It reloads the current file structure from Windows.
6. It displays tasks relevant to the current date and task state.

### 5.3 Browsing

- The browser displays folders and files located under the master folder.
- Users can navigate into subfolders and back to parents.
- Users can rename existing files or folders through Senku only if the product explicitly exposes that operation; v1 should expose rename for existing folders/files but never create or delete them.
- Senku must not display files outside the master folder in the file-organizer browser.
- Clicking a file opens it with its normal Windows-associated application.
- Clicking a folder navigates into it.
- The browser refreshes from the filesystem rather than relying on stale metadata.

### 5.4 Creating a task

1. The user enters a task name or text description.
2. The user selects a start date and time. Past times are allowed.
3. The user selects an end date and time. The default is start time plus ten years.
4. The user chooses Normal or Urgent priority. Normal is the default.
5. The user optionally adds one or more linked paths.
6. Senku validates that start time is before end time.
7. Senku saves the task as active.

Tasks do not require linked files. A task with no links remains a text-only reminder, including when marked Urgent.

## 6. Functional requirements

### FR-01 Master folder

- Senku shall store multiple workspaces and exactly one active workspace at a time.
- Each workspace shall contain one master folder, independent settings, and independent task buckets.
- Senku shall allow the user to create, switch, rename, and delete workspaces.
- Deleting a workspace shall delete its Senku metadata and tasks only; it shall never delete the Windows master folder. The last workspace may be deleted, returning the app to setup.
- The path shall be normalized before comparison and storage.
- Browser paths shall be accepted only when they are inside the configured master folder.
- Senku shall reject traversal attempts and paths outside the master folder for browser display.
- Task links may refer to paths outside the master folder when the user deliberately pastes them.
- Changing the master folder shall not silently rewrite existing task links.

### FR-02 User and workspace identity

- Senku shall ask for the user's name during initial setup.
- Senku shall ask for an explicit workspace name.
- Senku shall never infer the user's name from a filesystem path.
- Workspace names are display metadata only and do not rename Windows folders.

### FR-03 Task timing

- Each active task shall contain `startAt` and `endAt` timestamps.
- Timestamps shall use the system local timezone for display and a stable serialized format for storage.
- Past start times are valid.
- The default end time shall be ten years after the selected start time.
- The task engine shall evaluate task timing at least once per minute while Senku is running.
- A task becomes overdue when its end time has passed and it is still active.
- Notifications are generated at the start time only.
- If Senku was not running at the exact start time, it shall process an unhandled due start when the engine next runs, once per task occurrence.

### FR-04 Task priority and strict mode

- Normal tasks shall issue a Windows reminder notification.
- Urgent tasks shall issue a Windows notification and open every linked path at the start event regardless of Strict Mode.
- When Strict Mode is enabled, Normal tasks with linked paths shall also open those paths at their start event.
- When Strict Mode is disabled, Normal tasks shall remain reminder-only.
- Senku shall remember the previous Strict Mode choice after restart.
- An urgent task without linked paths shall behave as a reminder and shall not fail solely because it has no links.
- The implementation shall avoid opening the same task repeatedly after it has handled its start event.

### FR-05 Linked paths

- A task may contain zero or more linked paths.
- A linked path may be inside or outside the master folder.
- Links may point to documents, folders, URLs, playlists, repositories, or other resources that Windows can open.
- Senku shall never copy the linked resource.
- In normal mode, linked paths are available for manual opening from the task details.
- In strict mode, Senku shall attempt to open all linked paths.
- If a strict task cannot find or open a required linked path, Senku shall move the task out of active tasks into error tasks and retain the task metadata.
- Error tasks shall preserve their title, schedule, priority, and all linked paths, including the failed resource information.
- Users shall be able to edit an error task's linked paths and recover it to active tasks; recovery shall reset its one-time start-event marker.
- Moving a task to error tasks means copying task metadata within Senku's data files, not copying the user resource.

### FR-06 Completion

- Users shall manually mark active tasks as completed.
- Users shall be able to edit an active task at any time.
- The task checkbox shall provide the same manual completion action as Complete.
- Snooze shall shift the start time and reset the one-time start-event marker.
- Complete and Reschedule shall archive the current task and create a new active schedule.
- Opening a file shall never mark a task completed automatically.
- Completing a task shall save the completion timestamp.
- A completed task shall be removed from active task views and stored in completed task history.
- Users shall be able to delete completed tasks.
- Users shall be able to delete active, overdue, and error tasks.
- Deletion shall affect Senku metadata only and shall never delete the linked Windows resource.

### FR-07 Complete and reschedule

- The user shall be able to complete a task and reschedule it in one workflow.
- Senku shall first save the current task as completed, including its completion timestamp and original timeline.
- Senku shall then show a dialog for the next start and end times.
- The new schedule shall create a new active task derived from the completed task, retaining its text, priority, and links unless the user edits them.
- The new start time must be before the new end time.

### FR-08 Snooze

- Snooze shall be available from the task UI and notification actions where Windows supports the action.
- Allowed snooze durations shall be exactly 10 minutes, 1 hour, 1 day, and 1 week.
- Snoozing shall add the selected duration to the task's current start time.
- Snoozing shall reset the pending start notification state.
- Snoozing shall not change the task's end time unless the user explicitly edits the schedule.
- If snoozing produces a start time after the end time, Senku shall require the user to adjust the schedule or prevent the operation with a clear message.

### FR-09 Task views and filters

The UI shall provide at least these views:

- Active
- Upcoming
- Overdue
- Completed
- Error

Completed tasks shall support these filters:

- Today
- Yesterday
- Last week
- All completed tasks

- The task interface shall provide actions for Done, Snooze, Complete and Reschedule, Edit, Delete, and Error recovery where applicable.

The default task area shall emphasize tasks relevant to today while keeping other task views accessible.

### FR-10 Notifications

- Senku shall use Windows toast notifications for task start events.
- Notifications shall show the task name and whether it is Normal or Urgent.
- The app shall provide in-app access to the same task actions.
- Supported actions shall include Open/Show, Complete, and Snooze where technically supported.
- Notification failure shall not delete or complete a task.

### FR-11 Startup and power usage

- Senku shall provide an option to start with Windows.
- Recommended default behavior is launch minimized to the system tray.
- The main window shall open when the user selects the tray icon or otherwise launches Senku.
- The task engine shall use a lightweight timer with a one-minute maximum polling interval.
- The engine shall avoid continuous filesystem scanning while idle.
- The engine shall check only stored task metadata during normal timer ticks.
- Filesystem paths shall be checked when a task is executed in strict mode or when the user requests validation.
- The app shall prevent duplicate engine instances where practical.

## 7. Task state model

### 7.1 Stored states

- `active`: task is incomplete and its end time has not passed.
- `overdue`: task is incomplete and its end time has passed.
- `completed`: task was manually completed and contains completion history.
- `error`: strict execution failed to find or open one or more linked paths.
- `deleted`: optional audit state if deletion history is implemented; otherwise deleted records may be removed from metadata.

`Upcoming` is a UI filter of active tasks whose start time is in the future. It does not need to be a separate stored state.

### 7.2 Transitions

```text
active -> overdue       when now > endAt and task is incomplete
active -> completed     when user selects Complete
active -> completed     when user selects Complete and Reschedule
active -> error         when strict execution cannot process a linked path
active -> deleted       when user deletes the task
completed -> deleted    when user deletes completed history
error -> deleted        when user deletes the error record
```

Snooze keeps the task active and updates its start time.

## 8. Persistence

Senku shall store data under a per-user application directory, recommended:

```text
%APPDATA%\\Senku\\
```

Recommended files:

```text
settings.json       user name, active workspace, and startup preference
workspaces.json     workspace names, master folders, and per-workspace settings
workspaces/<id>/    active, completed, and error task records for one workspace
```

Writes shall be atomic where practical so an interrupted write does not leave malformed JSON. On startup, invalid data shall produce a recoverable error and shall not cause user files to be modified.

### 8.1 Example active task

```json
{
  "id": "task-unique-id",
  "title": "Finish AI workshop report",
  "priority": "normal",
  "startAt": "2026-08-19T18:00:00.000+05:30",
  "endAt": "2036-08-19T18:00:00.000+05:30",
  "linkedPaths": [
    "C:\\Users\\Vivek\\btech24-28\\workshop\\report.docx",
    "https://github.com/example/repository"
  ],
  "startEventHandled": false,
  "createdAt": "2026-08-19T12:00:00.000Z",
  "updatedAt": "2026-08-19T12:00:00.000Z"
}
```

### 8.2 Example completed task

A completed record shall retain at least:

- original task ID
- title
- priority
- linked paths
- original start and end times
- completion timestamp
- created and updated timestamps
- whether it was rescheduled
- the ID of the derived rescheduled task, if applicable

## 9. Proposed application structure

### Main views

1. **Setup:** user name, workspace name, and master folder path.
2. **Workspace:** master-folder browser, current path, and today task area.
3. **Task creation/edit:** title, dates, priority, linked paths, and save action.
4. **Task details:** task information and actions.
5. **Completed:** completed history and time filters.
6. **Overdue:** incomplete tasks past their end time.
7. **Error:** strict tasks that failed to open linked paths.
8. **Settings:** strict mode, startup behavior, notification preferences, and master-folder management.

### Electron process boundaries

- **Main process:** filesystem access, path validation, Windows shell opening, notifications, tray, startup registration, timer, and JSON persistence.
- **Renderer:** views, forms, task lists, filters, and user interactions.
- **Preload bridge:** narrowly scoped typed IPC methods. The renderer shall not receive unrestricted Node.js or filesystem access.

## 10. Safety and security requirements

- The renderer shall not use unrestricted `nodeIntegration`.
- Context isolation shall be enabled.
- IPC methods shall validate arguments in the main process.
- Master-folder containment checks shall use normalized absolute paths and correct Windows path-boundary logic. A path beginning with the master path text but outside its directory must be rejected.
- Senku shall never execute arbitrary pasted text as a shell command.
- Opening a path shall use a controlled Windows open operation rather than string-concatenated shell commands.
- All task deletion operations shall delete metadata only.
- Errors shall be visible to the user without exposing sensitive system details unnecessarily.

## 11. Non-functional requirements

### Performance

- The main window should become interactive quickly on normal student folders.
- Idle task checking should use negligible CPU and no repeated full-folder scan.
- The browser should load directories on demand rather than recursively loading the complete master folder.

### Reliability

- The app shall survive restart without losing saved settings or tasks.
- A task start event shall be handled at most once for each scheduled start time.
- Repeated timer ticks shall not create duplicate notifications or duplicate strict opens.
- Data writes shall be recoverable after an application crash where practical.

### Usability

- Dates and times use the Windows system timezone.
- The UI shall clearly distinguish Normal, Urgent, Overdue, Completed, and Error tasks.
- Destructive metadata actions shall require confirmation.
- The app shall explain when a linked resource is outside the master folder, while still allowing deliberate task linking.

## 12. Acceptance criteria

1. On first launch, user name, workspace name, and a valid master folder can be entered and saved.
2. On restart, the saved master folder and last visited subfolder are restored.
3. The browser never lists a path outside the master folder.
4. Clicking a displayed file opens it with its Windows-associated application.
5. A task can be saved with only text and no linked paths.
6. A task can contain multiple paths both inside and outside the master folder.
7. A past start time is accepted.
8. The default end time is exactly ten years after the selected start time.
9. A normal task produces a reminder at its start event and does not open files automatically.
10. An urgent task attempts to open every linked path when strict mode is enabled.
11. An urgent task without links produces a reminder without an error.
12. A missing strict-mode path moves the task metadata to Error and does not delete the resource.
13. Snooze changes the start time by exactly one selected duration and allows another reminder.
14. A task remains active until manually completed or deleted, unless it moves to Error.
15. An incomplete task past its end time appears in Overdue.
16. Completing a task stores its completion time and removes it from active views.
17. Completed filters show Today, Yesterday, Last week, and All.
18. Complete and Reschedule stores the completion history and opens the next-schedule dialog.
19. Restarting the app preserves strict mode and task data.
20. Startup launches the app minimized to the tray when enabled.
21. Deleting a workspace removes its Senku task metadata but leaves its Windows folder untouched.
22. Senku never copies, moves, creates, or deletes user files or folders.

## 13. Decisions requiring explicit implementation handling

- If the app was closed at the scheduled start, the first engine tick after launch treats the start event as due and handles it once. This prevents missed reminders while avoiding repeated notifications.
- A strict task with multiple links is considered successful only when all required links are opened successfully. Any failed link moves the task metadata to Error and records the failure.
- A URL or other external resource is stored as a link. The main process decides whether it can be opened using a controlled shell/open API.
- Changing the master folder changes browsing scope only; existing external task links remain unchanged.
