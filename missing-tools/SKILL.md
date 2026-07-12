---
name: missing-tools
description: Resolves missing CLI tools. Use when a command is unavailable, a shell reports command not found, or a tool must be run without installing it globally.
notice: Adapted from https://github.com/ryoppippi/dotfiles/blob/cfd9fdcbc356ed1f3ba63fd22d2a2d6f43feca81/agents/skills/missing-tools/SKILL.md (MIT)
---

# Missing Tools

Use this workflow when a command is unavailable in the current shell.

## Priority Order

1. Try the current project's direnv environment:

   ```sh
   direnv exec . <command>
   ```

2. Use [comma](https://github.com/nix-community/comma) for tools from nixpkgs:

   ```sh
   , <command>
   ```

3. Use `nix run` when a specific nixpkgs package is needed:

   ```sh
   nix run nixpkgs#<package> -- <args>
   ```

4. Use `nix shell` as the last resort:

   ```sh
   nix shell nixpkgs#<package> --command <command>
   ```

## Notes

- Never install missing tools globally. Do not use commands such as `npm install -g`, `pnpm add -g`, `go install`, `cargo install`, `uv tool install`, or other global installers to resolve a missing command.
- Prefer `direnv exec .` first because project-local dev shells often already provide the right tool version and environment variables.
- Comma automatically finds and runs the nixpkgs package containing the requested command.
- When multiple packages provide the command, comma fails with a tty error in non-interactive shells. List the candidates with `, -p <command>`, then run the chosen package with `nix run`.
- If the tool is not in nixpkgs at all, stop and report to the user instead of installing it by other means (e.g. `curl | sh`).
