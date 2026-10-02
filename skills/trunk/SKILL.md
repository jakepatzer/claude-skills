---
name: trunk
description: "Run a tree of Claude Code sessions for one piece of work. A long-lived ROOT session keeps the big picture; BRANCH sessions (made with the built-in /branch command) each do one task and hand a condensed summary back to their parent. Branches are named by path, e.g. Change 1/Subtask A/Side question, and each branch works out its own full path, parent, and role from the /branch marker in its history. Start a tree with `/trunk <root name>`; run `/trunk` alone to see where the current session sits."
disable-model-invocation: true
argument-hint: "[root name]"
---

# Trunk: a tree of sessions

One ROOT session holds the big picture for a piece of work. Each concrete task runs
in a BRANCH: a copy of the conversation made with Claude Code's built-in
`/branch <name>`. Branches can branch again, so the sessions form a tree.

Why: the root's context stays small and strategic, several tasks run in parallel, and
nobody re-explains the project to a new session.

## Names are paths

- The root gets its name from `/trunk <name>`, e.g. `Change 1`.
- A branch's full name is its parent's path plus a leaf: `Change 1/Subtask A`, then
  `Change 1/Subtask A/Side question`.
- The user may type just the leaf (`/branch Side question`) or the full path
  (`/branch Change 1/Subtask A/Side question`). Both resolve to the same full path.
- Parent = the full path minus its last segment.

## Work out your role

Do this before your first reply, and again whenever you're unsure.

`/branch` copies the conversation into a new session, and only the copy gets a
marker at the end of its history, like:

```
<command-name>/branch</command-name> … <command-args>Side question</command-args>
Branched conversation "Side question". You are now in the new branch (session …)
```

The original session never receives it. So every `/branch` marker in your history
belongs to you or one of your ancestors, oldest first.

1. Find where the tree starts: the most recent `/trunk <name>`, or a bare `/trunk` that
   was answered with `Role: ROOT — <name>`. Either way, `<name>` is the root name. A
   bare `/trunk` answered with a BRANCH role line was only a status check and doesn't
   start a tree. Ignore `/branch` markers before the tree start (an older, unrelated
   tree).
2. Walk the `/branch` markers after it, oldest first, starting with path = root name.
   For each marker's name N:
   - N starts with `<root name>/` → path = N (the user typed a full path);
   - otherwise → path = path + `/` + N (the user typed a leaf);
   - N empty (bare `/branch`) → choose a 1–4 word leaf from the task you're given and
     say so.
3. No markers after `/trunk` → you are the ROOT. Otherwise you are a BRANCH named by
   the final path.

If a typed full path isn't under the session it was forked from (e.g. `Change 1/Other`
typed inside `Change 1/Subtask A`), keep the typed path, since it says where the user
wants the summary to go, and mention the mismatch in one line. If the user says your
role is wrong, believe them and switch.

## The /trunk command

- `/trunk <name>` makes this session the ROOT named `<name>`. Reply
  `Role: ROOT — <name>`, plus one line: "Branch with `/branch <leaf>`; the new session
  works out its full path. Optional: `/rename <name>` so `/resume` finds this session."
  Then handle anything else the user asked in the same message.
- `/trunk` with no name re-establishes the role (useful after compaction or
  confusion): print the role line, full path and parent. It never starts a new tree
  inside an existing one. If no tree exists yet, become the root: pick a short name from
  the conversation's topic and announce it as `Role: ROOT — <name>`. Later sessions read
  the root name from that line.
- Running `/trunk` again is always safe. If this protocol seems to have dropped out of
  context, run it again.

## ROOT

Keeps the big picture: state, history, priorities, open decisions, lessons learned.

- **Does:** planning, prioritizing, answering questions, short read-only checks,
  drafting short messages when asked, integrating branch summaries.
- **Doesn't:** hands-on work (code changes, PRs, infrastructure changes, multi-step
  investigations), or edits to shared logs, docs and memory, unless the user explicitly
  asks. When asked for hands-on work, say in one line that it fits a branch and propose
  one.

**Proposing branches.** Give each one a leaf name and a brief:

```
/branch Fix flaky login test
Brief: <task in 1–3 lines>. Done when: <observable result>.
```

The branch inherits this message in its history, so the user doesn't have to paste
the brief: after `/branch <leaf>`, they can just say "go".

**Integrating a pasted BRANCH SUMMARY**, in 8 lines or fewer:
- what changed in the big picture;
- decisions the user now owes;
- conflicts with other branches' results;
- the updated top next branches.

Keep that state in the conversation. The branch already recorded its work, so don't
record it again.

## BRANCH

Does one task, hands-on.

**First reply:**
1. `Role: BRANCH — <full path> (parent: <parent path>)`.
2. If the user typed only a leaf: one optional line, `/rename <full path>`, so
   `/resume` shows the full path.
3. Two sentences: the task and its done-when. If the first message is just "go" (or
   similar) and the leaf matches a branch the parent proposed, that brief is the task.
4. A numbered plan with time estimates.
5. Anything needed from the user up front (approvals, logins, decisions).

Then start working, unless step 5 blocks you.

**While working:** follow the project's own working rules. If the project keeps a
decision log or docs, record decisions, test results and findings as they happen, tag
each entry with `[branch: <full path>]`, and update the docs in the same pass.

**Finishing.** Triggered when the task is done, or when the user says "wrap up",
"summary", "done" or similar. Offer it yourself when the task looks complete.
1. Confirm the records are current.
2. Stop or report background watchers, and remove temporary worktrees.
3. Output the summary below in a fenced block so it's easy to copy.

## BRANCH SUMMARY

Aim for 30 lines or fewer: facts, IDs and links, no process narration.

```
BRANCH SUMMARY — <full path> — <YYYY-MM-DD> — status: DONE | PARTIAL | BLOCKED
→ paste into: <parent path>
Task: <one line>
Outcome: <2–3 lines: what now works / what is now known to be true>
Shipped: <PRs, commits, deploys, tickets, posted messages — with links/IDs>
Decisions (who decided): <...>
Learnings (durable, ≤5): <facts or gotchas the parent should keep>
Open / next: <what's left, owner, what it's blocked on>
Needs from user/parent: <pending approvals or decisions>
Records: <log entries and doc sections touched, or "none">
Running: <watchers or worktrees still active, or "none">
```

**Nested branches:** a child's summary goes to its parent branch, not the root. The
parent folds the child's results into its own work and its own summary, marked
"via <child leaf>", and doesn't re-record what the child already recorded.

## Parallel sessions, shared files

Several branches may run at once against the same files and repos.

- **Append-only logs:** one entry per append, using a shell append
  (`cat >> LOG.md <<'EOF' … EOF`). Never rewrite, reorder, or use a whole-file write.
- **Docs:** re-read the exact section right before editing and make narrow
  replacements. If an edit fails because the file changed, re-read and reapply.
- **Git:** never switch branches in a checkout another session may be using. Give
  each branch that changes a repo its own worktree, and remove it when done.
- **Persistent memory:** write only durable cross-session facts, and check for an
  existing entry first. The root writes memory only when asked.

## Project rules come first

This skill defines only the tree mechanics. A project's own instructions (CLAUDE.md, a
handover doc) can add rules: what to log and where, review steps, approval gates.
Follow both. Where they conflict, the project's rules win.
