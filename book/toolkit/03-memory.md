# Memory

## Overview

Questions this chapter answers:

- Why does a coding agent forget a correction that you gave it in an earlier session?
- Which tools keep a memory between sessions, and what do they store?
- Where should you record information that the agent must know next time?

After working through it you should be able to:

- name the four places where an agent can keep information, and say who writes each one
- explain what is lost when a long session is compacted or a new session starts
- find, read, edit and disable the memory that Claude Code and Codex write
- keep corrections in a notes file in the repository, in any tool, including OpenCode
- recognise a memory that is out of date, and decide where the information belongs instead

## Why the agent forgets

On Monday you ask the agent how many dimuon pairs are in the mass histogram of the toy analysis. It runs `run.py` and reports the number. You tell it that every time it reports this number, you also want the fraction of events with at least one pair. The agent does this for the rest of the session.

On Tuesday you start a new session and ask the same question. The agent reports the number of pairs, but not the fraction. You have to give the correction again.

The reason is that the model does not learn from your conversations. Everything that it knows about the current session is in its *context window*. The context window is the text that the model receives with each request: your messages, its own answers, the instruction files and the output of every command it ran. When the session ends, this text is not used again. A new session starts with an empty conversation.

The [AGENTS.md chapter](01-agents-md.md) described one solution: write the information in `AGENTS.md`. The [skills chapter](02-skills.md) described skills, for longer procedures. This chapter describes the other ways in which tools keep information between sessions. It also describes what you should not let them keep.

There is no shared standard for agent memory. Each tool does something different, and the details change often. We checked every statement about a tool in this chapter against that tool's documentation in November 2026. The [AGENTS.md format](https://agents.md) does not mention memory. Its FAQ only asks you to treat the file as living documentation.

We also tested the examples in this chapter with Claude Code 2.1.323 in November 2026. Each test was a single run, so treat the results as examples, not as rates. We have not tested OpenCode yet.

<!-- TODO(Aashirvad): add a short example from your own work where an agent repeated a mistake that you had corrected in an earlier session. -->

## Four kinds of memory

When people say that an agent "remembers" something, they mean one of four things:

| Kind | Who writes it | When the agent reads it | Where it is stored | Do your collaborators get it? |
| --- | --- | --- | --- | --- |
| The current conversation | You and the agent | During the session | The context window | No |
| An earlier session that you continue | You and the agent | When you continue that session | A file or database on your computer | No |
| Instruction files and skills | You | At the start of a session, or when a task matches a skill | The repository, or your home directory | Yes, if they are in the repository |
| Agent memory | The agent | At the start of a session | Your home directory | No |

Usually only the last kind is called memory. The next sections describe all four, starting with the one that lasts the shortest time.

## Memory inside one session

A long session fills the context window. Before the window is full, the tool *compacts* the conversation: it replaces the older part of the conversation with a summary. The session can then continue, but the summary does not contain everything.

OpenCode compacts automatically by default. Next to the summary, it keeps the most recent part of the conversation, about 15000 tokens. The summary records the goal, the decisions, the finished and the open work, the next steps and the relevant files. The [OpenCode documentation on compaction](https://opencode.ai/docs/compaction/) states clearly that compaction loses information. After compaction, OpenCode uses the current version of your instruction files.

Claude Code and Codex also compact long sessions automatically, and both have a `/compact` command to compact immediately. After compaction, Claude Code reads the root `CLAUDE.md` of the project and its auto memory again from disk ([what survives compaction](https://code.claude.com/docs/en/context-window#what-survives-compaction)).

What is lost is usually a detail: an exact number, the command that worked, or the reason for a decision. If a detail must survive, write it in a file before the session becomes long. A simple habit also helps: start a new session for each new task.

## Continuing an earlier session

All three tools save each session on your computer, so that you can continue it later:

- **OpenCode:** in the terminal interface, run `/sessions`, or press Ctrl+X and then L ([TUI](https://opencode.ai/docs/cli/tui/)). For a single command, `opencode run --continue` continues the last session ([CLI commands](https://opencode.ai/docs/cli/commands/)).
- **Claude Code:** `claude --continue` opens the most recent session in the current directory, and `claude --resume` lets you choose a session ([sessions](https://code.claude.com/docs/en/sessions)).
- **Codex:** `codex resume` lets you choose a session, and `codex resume --last` opens the most recent session in the current directory ([commands](https://learn.chatgpt.com/docs/developer-commands?surface=cli)).

A continued session contains that one conversation again, including its mistakes. It is not a memory of your project. A new session that you start tomorrow does not know about it.

## Memory that the agent writes

Some tools let the agent write notes for itself, which it reads in later sessions. As of November 2026, the three tools are very different.

### OpenCode: no built-in memory

OpenCode's documentation describes no memory that the agent writes for itself. For guidance that must last between sessions, it refers to `AGENTS.md` ([instructions](https://opencode.ai/docs/instructions/)). OpenCode reads your personal file `~/.config/opencode/AGENTS.md`. It then reads every `AGENTS.md` from the current directory up to your home directory, and combines them. For a project outside your home directory, it stops at the project root. In a directory without an `AGENTS.md`, it reads `CLAUDE.md` instead.

To keep corrections between sessions in OpenCode, use a notes file in the repository, as described later in this chapter. This method works in every tool.

### Claude Code: auto memory

Claude Code has *auto memory*, which is enabled by default ([auto memory](https://code.claude.com/docs/en/memory#auto-memory)). The agent saves a note when it expects the information to be useful in a later session, for example a preference or a correction. It does not save something in every session.

The notes are Markdown files in `~/.claude/projects/<project>/memory/`, with one directory for each Git repository. The file `MEMORY.md` is an index with one line for each note, and each note is a separate file. At the start of each session, Claude Code loads the first 200 lines or 25 KB of `MEMORY.md`. The agent reads a note itself when it needs it. The memory stays on your computer: it is not part of the repository, and it is not copied to other computers.

We repeated the Monday and Tuesday example once, with Claude Code 2.1.323 in November 2026. In the first session, Claude Code saved this note (we removed two lines of metadata):

```markdown
---
name: pair-count-with-event-fraction
description: "When reporting a dimuon pair count, also report % of events with at least one selected pair (one decimal)"
metadata:
  type: feedback
  modified: 2026-11-23T08:56:40.881Z
---

Whenever reporting a pair count in the toy-dimuon repo, also give the fraction of events with at least one selected pair, as a percent with one decimal (e.g. 16.1%).

**Why:** The user asked for this on 2026-11-23, after a pair count was reported on its own.

**How to apply:** `run.py` prints only events and histogram pairs. Compute the fraction separately as `ak.sum(ak.num(select_pairs(muons)) > 0) / len(muons)`. Note that the histogram (0–200 GeV, flow bins excluded from `h.sum()`) counts fewer pairs than `select_pairs` returns. On 2026-11-23 that was 9741 in the histogram vs 13067 selected.
```

In the next session, the agent read the note and reported the fraction without being asked. Now read the last line of the note again. The note is about a preference, but it also contains the numbers of that day. These numbers will be wrong after the next change to the selection. Notice this kind of content when you read memory.

<!-- TODO(Aashirvad): add a real note that auto memory saved in one of your own projects (remove anything private), and say whether it helped or was wrong. -->

You can read and edit the notes like any other Markdown file, or run `/memory` in a session to open them. To disable auto memory, use the toggle in `/memory`, set `"autoMemoryEnabled": false` in the settings of the project, or set the environment variable `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

### Codex: memories

Codex has *memories*, which are disabled by default ([memories](https://learn.chatgpt.com/docs/customization/memories)). To enable them, add this to `~/.codex/config.toml`:

```toml
[features]
memories = true
```

Codex then creates memories from earlier chats in the background. It waits until a chat has been inactive for some time, and it skips short chats. It also removes secrets from the memory text that it writes. The files are in your Codex home directory, `~/.codex`. In an interactive session, the `/memories` command controls whether the current chat uses existing memories, and whether Codex can use the chat for new memories.

Two points from the Codex documentation apply to every tool. First, memories help the agent recall things, but rules that must always apply belong in `AGENTS.md`. Second, the memory files are generated by the tool. You can read them, but editing them by hand is not the intended way to control memory.

## What to store, and where

Before you let an agent remember something, decide where the information belongs. One question decides most cases: does the agent of a collaborator also need it? If yes, the information belongs in a file in the repository. Agent memory stays on your computer.

| Information | Where it belongs |
| --- | --- |
| `data/` is read-only | `AGENTS.md`: a rule for everyone, in every session |
| How to make the cutflow table for the note | A skill in the repository |
| The 2023 luminosity | `config/plot_config.json`, which the plotting code reads |
| A correction that the agent needs again in this repository | `NOTES.md` in the repository, or agent memory |
| You want short answers that list the commands that were run | Your personal instruction file, or agent memory |
| What you finished yesterday, and what you plan to do next | A short note in the repository, or the continued session |
| A password, a token or your grid proxy | Nowhere |

The personal instruction file is `~/.config/opencode/AGENTS.md` for OpenCode, `~/.claude/CLAUDE.md` for Claude Code and `~/.codex/AGENTS.md` for Codex.

Numbers that the analysis depends on, such as a luminosity, a cross-section or a cut value, never belong in memory. They belong in a file that the analysis code reads. Then a change happens in one place, and Git shows it. In a single test run with Claude Code 2.1.323 in November 2026, the agent refused to save a luminosity for this reason. The same number was already in `config/plot_config.json`, and the agent explained that a second copy could become out of date.

## A notes file in the repository

A notes file is memory that you can review. It works in every tool, including OpenCode, because the agent only has to read and edit a file. Add this section to the `AGENTS.md` of the toy repository from the [AGENTS.md exercise](01-agents-md.md):

```markdown
## Notes from earlier sessions
- `NOTES.md` lists corrections and preferences from earlier sessions. Read it
  before you start a task.
- When I correct you or tell you a preference that will matter in later
  sessions, add one dated line to `NOTES.md` and tell me that you did.
  Do not change other lines in the file without asking.
```

Then create `NOTES.md` and commit both files:

```markdown
# Notes for agents

Corrections and preferences from earlier sessions, one dated line each.
Review changes to this file like any other change.
```

We tested this once, with Claude Code 2.1.323 in November 2026. Auto memory was disabled, so only the file could keep the information. In the first session, we asked for the number of pairs and then gave the correction about the fraction of events. The agent added one line to `NOTES.md` and told us:

```text
- 2026-11-23: Whenever you report a pair count, also give the fraction of events with at least one selected pair, in percent with one decimal.
```

In a new session, the agent read `NOTES.md` first. It reported 9741 pairs and 16.1% of events. It also noticed that the new line was not committed yet, and reminded us that the file says that changes must be reviewed. We have not run this test with OpenCode yet.

<!-- TODO(Aashirvad): run the same two sessions with OpenCode and add what happened. -->

A notes file has three advantages over agent memory. You see every change in `git diff`. The agents of your collaborators read the same notes. And the notes stay the same when you change tools.

A notes file also needs care. It grows, so delete lines that no longer matter. When you decide that a note is really a rule, move it to `AGENTS.md`. Keep personal preferences out of it. They belong in your personal instruction file, not in a file that the agents of your collaborators read.

## Memory that is out of date

A memory can be wrong without anyone noticing. For example, a note says that the 2023 luminosity is 27.2 fb⁻¹. Then the luminosity is recalibrated, and the configuration changes to 27.9 fb⁻¹. The note still says 27.2.

We tested this once, with Claude Code 2.1.323 in November 2026. In the first session, we asked the agent to remember 27.2 fb⁻¹ for the 2023 plots. The repository had no configuration file yet, so the agent saved a note. Then we added `config/plot_config.json` with 27.9 fb⁻¹ for 2023. In a new session, we asked which luminosity the label of the 2023 plot should show. The agent found both values, did not choose one, and asked which value was current.

This was a single run. Also, the note itself said that the agent should ask if a different value appeared. Do not expect the same behaviour every time. The safe method is the one from the previous section: keep such numbers in files, and do not ask the agent to remember them.

The same problem exists without agent memory. The [skills chapter](02-skills.md) describes an agent that copied a luminosity from an older plotting script. Every copy of a number can become out of date: in a memory file, in an old script, or earlier in a long conversation.

## Checking and cleaning memory

Read the memory of your agent regularly, as you would read the notes of a colleague about your analysis:

- **OpenCode:** there is no agent memory to check. Read `NOTES.md`, and read its history with `git log -p NOTES.md`.
- **Claude Code:** run `/memory`, or read the files in `~/.claude/projects/<project>/memory/`. You can edit or delete any file. Claude Code deletes old session transcripts after 30 days by default, but it does not delete memory files. `claude purge <path>` deletes the transcripts and the memory of one project. It shows what it will delete and asks you to confirm ([the .claude directory](https://code.claude.com/docs/en/claude-directory)).
- **Codex:** read the files in `~/.codex`, and use `/memories` or the configuration to control which memories are used. The documentation describes a button that deletes all memories only for the ChatGPT desktop app.

When you compare two runs of an agent, disable memory or clear it first. Otherwise the second run can use what the agent learned in the first run. Step 2 of the [AGENTS.md exercise](01-agents-md.md) also says this.

## Exercise

Test what your agent remembers between sessions, and what it does when a memory is out of date. Use the `toy-dimuon/` repository with the `AGENTS.md` from the [AGENTS.md exercise](01-agents-md.md), and commit everything first.

The steps are written for a tool with agent memory: Claude Code with auto memory, or Codex with memories enabled. Codex creates memories in the background some time after a chat ends, so with Codex, do step 4 on a later day. With OpenCode, do steps 1 to 4 with the notes file from the previous section instead of agent memory.

1. Check that the agent has no memory for this repository yet: run `/memory` in Claude Code, or check for memory files in `~/.codex` for Codex.
2. Start a session and give these two prompts, one after the other:

   > Run run.py and tell me how many pairs are in the mass histogram.

   > Thanks. From now on, whenever you report a pair count in this repository, also give the fraction of events that have at least one selected pair, in percent with one decimal.

3. End the session. Check what the agent stored, and read it.
4. Start a new session and give this prompt:

   > Run run.py again and tell me the result.

   Does the answer include the fraction of events?
5. Delete what the agent stored in step 3, disable memory, and repeat steps 2 to 4. In Claude Code, start each session with `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` set.
6. Enable memory again, and test a memory that becomes out of date. Do not create `config/plot_config.json` yet. In a new session, give this prompt:

   > Please remember this for future sessions: for the 2023 plots we use an integrated luminosity of 27.2 fb^-1, from the luminosity group's October recommendation.

   Then save the configuration from the [skills chapter](02-skills.md) as `config/plot_config.json`, change the 2023 value from 27.2 to 27.9, and commit it. In a new session, give this prompt:

   > What integrated luminosity should the label of the 2023 dimuon mass plot show?

   Which value does the agent give? Does it tell you that there are two values?

Agents are not deterministic, so one run of each step is an example, not a measurement. If you have time, run each step twice.

<details>
<summary>Solution</summary>

What to expect:

- **Step 3:** one note. In Claude Code, this is a new file in `~/.claude/projects/<project>/memory/` and one new line in `MEMORY.md`. With the notes file, it is one new dated line in `NOTES.md`.
- **Step 4:** the answer gives the fraction without being asked: 8044 of 50000 events, or 16.1%, next to the 9741 pairs in the histogram.
- **Step 5:** without memory, the answer gives only the number of pairs.
- **Step 6:** a good answer gives both values and asks which one is current. If the agent gives 27.2 without comment, it used the memory instead of the configuration file, and nothing warned you.

We ran each step a single time, with Claude Code 2.1.323 and the Opus 5.5 model in November 2026, in a new, empty configuration directory. We have not run these steps with OpenCode yet.

| Steps | Setup | What the agent did |
| --- | --- | --- |
| 2–4 | Auto memory enabled | Saved a `feedback` note, the one shown earlier in this chapter. In the new session, it read the note and reported 9741 pairs and 16.1% of events. |
| 5 | Auto memory disabled | Saved nothing. It said that it would not remember the rule in a new conversation, and offered to add it to `AGENTS.md`. In the new session, it reported only 9741 pairs. |
| 2–4 | Notes file, auto memory disabled | Added one dated line to `NOTES.md`. In the new session, it read `NOTES.md` and reported 9741 pairs and 16.1% of events. |
| 6 | Auto memory enabled, configuration with 27.2 already committed | Saved nothing, because the value was already in the configuration file. |
| 6 | Auto memory enabled, no configuration in the first session | Saved a `project` note with 27.2. After we added the configuration with 27.9, it reported both values and asked which was current. |

The 9741 pairs are the pairs in the histogram, which covers 0 to 200 GeV. The selection finds 13067 pairs, and pairs with a mass above 200 GeV are not in the histogram. In every run in which the agent calculated the fraction, it also mentioned this difference without being asked.

If step 4 shows no fraction although memory is enabled, check step 3 first. Perhaps nothing was stored. With Codex, the memory may not exist yet, because Codex creates it in the background.

</details>

## Pitfalls and responsible use

Agent memory is private to you. The agents of your collaborators do not have it, and neither does an agent in a CI job or on another computer. If a result depends on something that only your agent remembers, nobody else can reproduce how the result was made. Everything that the analysis depends on belongs in the repository.

Memory is guidance, not a guardrail. The Claude Code documentation says that Claude treats both `CLAUDE.md` files and auto memory as context, not as enforced configuration. The Codex documentation says that you should not depend on memories for rules that must always apply. Rules go in `AGENTS.md`. Anything that must never happen needs the measures from the [AGENTS.md chapter](01-agents-md.md): a sandbox, read-only data and permission settings.

Memory files and session transcripts are plain text files in your home directory. The Claude Code documentation warns that everything that passes through a tool is written to the transcript, including file contents and command output. So if the agent reads a file that contains a password, the password is now in the transcript too. It can also appear in a memory note. Codex removes secrets from the memories that it writes, but its documentation still says that you should not store secrets in memories. Keep credentials out of the environment of the agent, as the AGENTS.md chapter recommends for your grid proxy.

Everything that the agent loads into its context window is sent to the model provider with each request. This includes the memory that the agent loads at the start of a session. Do not let the agent remember anything that you would not send to the provider.

On a shared computing cluster, your home directory is often on a network file system. Check who can read `~/.claude`, `~/.codex` and `~/.local/share/opencode`, the directory where OpenCode stores its sessions ([troubleshooting](https://opencode.ai/docs/troubleshooting/)). If you use Codex on several computers with the same home directory, first read the Codex [page on NFS home directories](https://learn.chatgpt.com/docs/environments/network-file-systems). Codex keeps a database in its home directory, which also holds the state of memory processing. If two computers write to it at the same time in the wrong mode, the database can be damaged and memory data can be lost. The page explains how Codex prevents this and how to convert an existing home directory.

You stay responsible for the result. Memory makes it more likely that the agent follows your preferences. It does not check that its numbers are correct. Compare results with something that you trust, as in the earlier chapters.

## Key points

- The model does not learn from your sessions. A new session starts with an empty conversation, and only files and memory connect it to earlier work.
- Long sessions are compacted into a summary, which loses details. Write important details in a file, and start a new session for each new task.
- As of November 2026, OpenCode has no built-in agent memory, Claude Code enables auto memory by default, and Codex disables memories by default.
- If the agent of a collaborator also needs the information, put it in the repository: in `AGENTS.md`, in a skill, in a configuration file or in a reviewed notes file. Analysis numbers and secrets never belong in memory.
- Read and clean memory regularly, and disable it when you compare runs. Memory does not replace a sandbox or your own checks.

## References

- [AGENTS.md](https://agents.md): the format and its FAQ
- [Instructions](https://opencode.ai/docs/instructions/) and [Compaction](https://opencode.ai/docs/compaction/) (OpenCode)
- [TUI](https://opencode.ai/docs/cli/tui/), [CLI commands](https://opencode.ai/docs/cli/commands/) and [Troubleshooting](https://opencode.ai/docs/troubleshooting/) (OpenCode)
- [How Claude remembers your project](https://code.claude.com/docs/en/memory) (Claude Code)
- [Explore the context window](https://code.claude.com/docs/en/context-window), [Manage sessions](https://code.claude.com/docs/en/sessions) and [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory) (Claude Code)
- [Memories](https://learn.chatgpt.com/docs/customization/memories) (Codex)
- [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) (Codex)
- [Run Codex with an NFS home directory](https://learn.chatgpt.com/docs/environments/network-file-systems) (Codex)
