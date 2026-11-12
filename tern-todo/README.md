# Todo + Jira

**Your Jira sprint as a to-do list in [Tern](https://docs.stencil.so/tern/).** The Jira tickets tab
lists your board's active sprint. Enter hands a ticket to an omp agent in a tab of its own,
and when you mark the task done, the issue moves to Done in Jira. A task you write yourself
goes to Jira from its card.

<p align="center">
  <img src="docs/tickets.gif" alt="The Tickets tab lists WEB Sprint 24; Start on WEB-412 opens an omp agent in a new tab, which finds and fixes the checkout bug; back in Todo, Go to task, then Mark as done closes the agent's pane, a toast says Closed WEB-412 in Jira, and the ticket shows Done" width="100%">
</p>

<p align="center"><sub>A real omp agent; the Jira is a local stand-in with made-up issues.</sub></p>

It is [deLiseLINO/tern-todo](https://github.com/deLiseLINO/tern-todo), the task list whose
tasks open omp sessions in Tern panes, plus Jira, and a fix for Todo blocks in a second
window, whose tasks opened their tabs in the first. The code lives in the fork
[sneaky-bulgarian/tern-todo](https://github.com/sneaky-bulgarian/tern-todo); this folder
holds its docs and demos.

## Install

```sh
tern plugin install github.com/sneaky-bulgarian/tern-todo
```

Open it with **Todo: open tasks** in the palette. To give it a shortcut, bind the action
`plugin.todo-proto.open` in Settings › Keyboard, or in `settings.json`:

```json
"keybinds": { "cmd+alt+t": "plugin.todo-proto.open" }
```

## Connect Jira

<p align="center">
  <img src="docs/connect.gif" alt="On the Tickets tab, the Connect Jira form takes the Jira URL, an email and an API token; Connect signs in as Alex Rivera, the project picker finds Acme Web, the board picker offers WEB board and WEB support, and WEB Sprint 24 appears with six tickets" width="100%">
</p>

Open the **Jira tickets** tab (or press `T` on Tasks) and fill in the form:

- **Jira Cloud**: the site URL (`https://your-company.atlassian.net`), your Atlassian email
  and an [API token](https://id.atlassian.com/manage-profile/security/api-tokens). Use
  **Create API token**, not the one *with scopes*: scoped tokens only work through
  `api.atlassian.com`.
- **Jira Data Center**: the server URL, no email, and a personal access token.

Connect checks the credentials against Jira before it saves anything. The token goes to
the system keychain; the URL, email, project and board stay in the plugin's `kv.json`, with
the project and component that tasks post to.
Then pick the project and its board (a project with a single board picks it for you).

## Use

| Key | Does |
| --- | --- |
| ↑ ↓ / j k | Move |
| ⏎, double-click | Start: add the ticket to your tasks and launch omp on it |
| a | Add to tasks without starting |
| o | Open the issue in Jira |
| r | Reload the sprint |
| p, b | Pick the project, the board |
| c | Edit the connection, or disconnect |
| m | Everyone's tickets or only yours |
| s | Group by assignee, status, or not at all |
| T | Back to Tasks (on Tasks, T opens Jira tickets) |

- **The view.** **Everyone** / **Only mine** narrows the list to your tickets; the grouping
  pill groups them by assignee (the default: yours first, then by name, Unassigned last), by
  status (to do, in progress, done), or not at all. Tickets keep the board's rank inside a
  group, and a click on a group's header folds it. The view sticks, for every Todos block.
- **Start** writes the task (the key and summary as its title; the description, the long
  text fields such as Definition of Done and Acceptance criteria, and the issue link as
  notes) and launches it like any other task: omp gets them as its prompt and runs in the
  last project you used, in a new git worktree while **Create new chats in a worktree** is
  on. To pick the project or model first, add the ticket with `a`. The ticket's card shows
  the same long text fields under its description.
- Each row shows the issue's Jira status and assignee, and once the ticket has a task,
  that task's agent status: Working, Needs you, Ready for you.
- **Done goes back to Jira.** When a linked task turns done (its check, space, **Mark done**
  on a selection, or closing its pane), the issue moves to a done status: a transition
  named Done, Closed or Resolved first, with a resolution when the workflow requires one.
  An issue that is already done is left alone. If Jira refuses, a toast says why and the
  ticket's card offers **Close in Jira**.

## Post a task to Jira

<p align="center">
  <img src="docs/post.png" alt="A new task, Coupon field on the checkout page, with notes; its card lists Priority, then Agent rows for Model, Thinking, Project and Chat, then Jira rows with Project Acme Web and Component Checkout, above the buttons Add and post to Jira and Add task" width="100%">
</p>

The task card lays its controls out in labelled rows: the task's priority, then the agent
that runs it (model, thinking level, project folder, chat). While Jira is connected, an open
task without an issue also gets a **Jira** panel, the draft of its issue:

- **Project**, **Type** and **Component** say where the issue goes. With nothing saved, they
  preselect the Jira tickets tab's project (else the first project Jira lists), Jira's default
  type (Task, else the project's first standard type) and that project's first component.
  Your first pick, or your first post, becomes the default and stays until you pick another.
  Each project keeps its own type and component; **No component** is a choice too.
- Then the fields of that type's create screen, as Jira describes them: every required one
  (marked `*`), multi-line custom fields such as **Definition of Done** and **Acceptance
  criteria**, and priority, labels, assignee, due date and fix versions. **N more fields**
  shows the rest. Choices open a picker with Jira's allowed values (several for a
  multi-select), a user field offers **Me**, and text, numbers, dates (`YYYY-MM-DD`), times
  (`YYYY-MM-DD HH:MM`) and labels are typed in place. The values stay on the task until it
  is posted.
- **Post to Jira** creates the issue: the title is its summary, the notes its description,
  plus the panel's fields. A required field left empty, or a value Jira won't take, turns
  red and a toast lists them; nothing is sent. The task is then linked: its card and row
  show the key, and marking it done closes the issue like any linked task.
- A new task's card takes notes and fields too. **Add and post to Jira** adds the task and
  posts it in one go; **Add task** (or Enter in the add row) only adds it.

If Jira still refuses, a toast quotes its reason and the task stays local. If Jira won't
describe its create screen, the panel says why and offers **Retry**; posting then sends the
title, notes and component only.

## How it works

Host halves can't reach the keychain, so the window half, `jira.luau`, talks to Jira. The
block asks with `cx:open("todo-jira:<op>?…")`, a link only this plugin's window claims
(`jira-link.luau`), and the window answers with a `todo.jira` event. Links never carry the
token: Connect reads it off the block's masked field. The calls are REST v2 and Agile 1.0,
which Cloud and Data Center both serve: `myself` to check the sign-in, `project/search`,
`board?projectKeyOrId=`, `board/{id}/sprint?state=active`, `sprint/{id}/issue` (with the
site's multi-line custom fields from `field`) and `issue/{key}/transitions`; posting adds
`project/{key}/components`, `issue/createmeta/{key}/issuetypes[/{type}]` (the old
`issue/createmeta?expand=projects.issuetypes.fields` on Jira Data Center before 8.4) and
`POST issue`. Requests time out after 20 s; lists stop at 20 pages of 50. Every window of
the block's window key gets its links, so the first one to take a post claims it in
`kv.json` and the issue is created once.

Linked tasks keep `"jira": {"key": "WEB-412", "url": "…/browse/WEB-412"}` in the task file;
an unposted task keeps its Jira panel's values in `"jira_fields"`, by Jira field id.
The done hook sits in the plugin's file writer, so it fires once for each task that turns
done, whatever turned it.

Limits: Kanban boards have no sprints, the tab needs an open Tern window, and edits to the
task file by hand don't reach Jira.

## Develop

```sh
git clone https://github.com/sneaky-bulgarian/tern-todo
tern plugin link tern-todo          # reloads on every save
tern plugin types tern-todo         # tern.d.luau for luau-lsp
```
