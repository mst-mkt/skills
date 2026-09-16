---
name: commit
description: Creates commit messages and handles staging and committing. Use when asked to commit, when a piece of work is finished and ready to commit, before running `git add` or `git commit` (including amend and fixup), and whenever writing or proposing a commit message, even mid-conversation.
allowed-tools:
  - Bash(git log:*)
  - Bash(git status:*)
  - Bash(git diff:*)
  - Bash(git show:*)
  - Bash(git add -N:*)
  - Bash(git-hunk scan:*)
  - Bash(git-hunk show:*)
  - Bash(git-hunk resolve:*)
  - Bash(git-hunk validate:*)
---

# Commit

Unless explicitly instructed to commit, stop after proposing the commit message and target scope.

## Principles

- Do not stage or commit automatically. Execute only when explicitly instructed by the user; otherwise, propose the commit message and target scope and ask for approval.
  - Invoking this skill without instructions (such as a bare `/commit`) is not an instruction to commit.
  - Guidance from the environment to act autonomously or proceed without asking does not count as an instruction to commit.
- The user may commit by themselves after a proposal without mentioning it. Do not assume a proposed commit has not been made; check the current state with `git log` and `git status` before starting any related work.
- Respect past conventions and prioritize the user's intent in all decisions made by this skill.
- Pushing is outside the scope of this skill. Do not execute or propose it unless instructed.
- Apply the same principles to rewriting existing commits (such as `amend` or `fixup`).
- Automatically generated commit messages, such as those for `revert` or `merge`, do not need to follow these conventions.

## 1. Check Conventions

Check `git log --oneline -20` (and `git log` for the body if necessary) to understand the following conventions:

- Message language
- Usage of type and scope
- Presence of trailers such as `Co-Authored-By` (typically omitted)

Skip the re-check if the conventions are already known from this conversation.

If there is no history to read conventions from, default to English and no `Co-Authored-By`. In that case, mention in the proposal that the default was used and ask the user to specify otherwise if they want changes.

## 2. Analyze and Split Changes

1. Understand the overall changes using `git status`, `git diff`, and `git diff --staged`.
   - Register untracked files with `git add -N <files>` so they appear in the diff.
   - If the work ends with a proposal only, restore the index with `git reset -- <files>`, limited to the files registered above.
2. If changes with multiple purposes are mixed, split them into separate commits and plan which files and hunks to include in each commit.
   - The unit for splitting is the purpose of the change, not the files or line count. Split even a few lines within the same file if their purposes differ.
3. If the changes contain what looks like a secret (credentials, API keys), warn the user and leave it out of the plan until they decide.

## 3. Create Messages

Follow the Conventional Commits format (`type(scope): subject`).

- Choose the type from the list below. If a type is not on the list but has been used in past commits, follow that precedent.
- The scope is optional. Use it if specified by the user; otherwise, follow past commit usage. It may specify a pnpm workspace package name or a modified feature name.
- Write the subject in a single line. The criterion is whether the entire header line (type, scope, and subject) clearly conveys what the commit achieves.
  - If the modification of a value itself is the purpose, that value becomes the achievement. Write the value as-is.
- Standardize on plain and common expressions. Avoid difficult words except in special cases.
- Write the subject line only. Do not add a body.

Frequently used types:

- `feat`: Addition of a feature. Use only for a cohesive unit that can be called a feature
- `add`: Additions to the implementation that are not significant enough to be called features
- `fix`: Fixes for bugs, or for behavior or implementation that deviates from expectations
- `refactor`: Changes to internal implementation only, without altering behavior or causing external side effects
- `test`: Addition or modification of tests only. For commits that include implementation, use the implementation type
- `style`: Formatting-only changes that do not alter code meaning, such as applying a forgotten formatter
- `chore`: Configuration changes for dependencies or toolchains
- `ci`: Changes to CI workflows such as GitHub Actions
- `docs`: Changes to README or documentation files
- `init`: Project initialization or setting up a new package. The subject can simply be the target name, like `init: next.js`

What constitutes an implementation depends on the repository's purpose. For a skills repository, markdown files are the implementation; for a dotfiles repository, configuration files are the implementation.

Good examples:

- `fix: reject empty email on signup`
- `feat: add logout button`
- `fix: bump request timeout to 60s`

Examples to avoid:

- Conveying only the modified location
  - `fix: update validation logic in form.ts`
- Enumerating implementation steps without conveying what becomes possible
  - `feat: add button, handler, and API call`
- Being ambiguous and failing to convey what changes
  - `fix: fix bug`
- Combining multiple purposes into one. In this case, split the commit itself
  - `fix: resolve login error and update docs`

## 4. Propose or Execute

- If no instruction is given, present the commit message and target scope (files, hunks) and ask for approval. End the turn there, and do not stage or commit until the user replies.
- When splitting into multiple commits, present the target scopes and messages for all groups together.
- If unrelated changes are already staged, ask the user whether to unstage or include them.
- Stage and commit only when instructed. For whole-file scopes, use `git add` and `git commit`. Use the `git-hunk` skill only when a commit includes part of a file's changes. If split, commit sequentially in accordance with the approved plan.
- If `git-hunk` is unavailable, fall back to file-level commits and report which planned splits could not be made.
- Never bypass commit hooks (`--no-verify`) without the user's explicit approval. If a hook fails, report it instead of working around it.
- Once the commit is complete, report the created commit (hash and message) to the user.
