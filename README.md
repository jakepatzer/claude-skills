# claude-skills

Skills for [Claude Code](https://code.claude.com).

| Skill | Command | What it does |
|---|---|---|
| [trunk](skills/trunk/SKILL.md) | `/trunk [root name]` | Runs a tree of sessions: one root session for the big picture, and a `/branch` session per task. Branches get path names and hand a summary back to their parent. |

## Install

### As a plugin

In Claude Code:

```
/plugin marketplace add jakepatzer/claude-skills
/plugin install jakepatzer-skills@jakepatzer
```

Restart Claude Code. The skill answers to `/trunk`, or `/jakepatzer-skills:trunk` if another
command already uses that name.

The plugin tracks this repo's commits. To update, run
`claude plugin marketplace update jakepatzer`, then
`claude plugin update jakepatzer-skills@jakepatzer`.

### Manually

Copy or symlink a skill into your personal skills directory:

```sh
git clone https://github.com/jakepatzer/claude-skills.git
ln -s "$PWD/claude-skills/skills/trunk" ~/.claude/skills/trunk
```

## Using /trunk

```
/trunk Change 1       # this session becomes the root "Change 1"
/branch Subtask A     # new session: Change 1/Subtask A
/branch Side question # run inside Subtask A: Change 1/Subtask A/Side question
```

`/branch` is Claude Code's built-in command. You can give it just the new leaf name or
the full path. Each branch:
1. starts by stating its full path and its parent;
2. does its one task;
3. ends with a `BRANCH SUMMARY` whose first line says which session to paste it into.

Run `/trunk` with no name to see where the current session sits in the tree.

## License

[MIT](LICENSE)
