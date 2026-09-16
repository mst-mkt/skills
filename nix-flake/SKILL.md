---
name: nix-flake
description: Create and edit flake.nix for development environments and package distribution. Use this when asked to prepare a flake, devShell, or direnv, or when packaging a tool with Nix.
allowed-tools:
  - Bash(nix flake check:*)
  - Bash(nix flake show:*)
  - Bash(nix fmt:*)
  - Bash(nix develop:*)
  - Bash(nix run:*)
  - Bash(git add -N:*)
  - Bash(git reset --:*)
  - Bash(git diff:*)
---

# Nix flake

## 1. Determine the purpose

- Set up a development environment: [references/devshell.md](references/devshell.md)
- Distribute packages for others to use: [references/package.md](references/package.md)

## 2. Common rules

- Minimize dependencies
  - Do not add inputs from auxiliary libraries like flake-utils or flake-parts. Write system expansion using `nixpkgs.lib.genAttrs`
    ```nix
    forAllSystems = f: nixpkgs.lib.genAttrs systems (system: f nixpkgs.legacyPackages.${system});
    ```
- For supported systems, list those that work among `x86_64-linux`, `aarch64-linux`, and `aarch64-darwin`
  - Do not use `lib.systems.flakeExposed` because its scope is too broad
  - Do not include `x86_64-darwin` as nixpkgs support for it is ending
  - If an output cannot function without a specific input's packages, you may align with that input's `packages` systems
- Use `github:NixOS/nixpkgs/nixos-unstable` as the nixpkgs input
- By default, add `inputs.nixpkgs.follows = "nixpkgs"` to inputs that have nixpkgs as an input. Do not add it in the following cases:
  - You want to use that input's binary cache. Following it changes the build result, causing cache misses
  - That input requires a specific version of nixpkgs
- Avoid writing comments. Write them only when the code alone cannot convey the intent
- Omit `description` unless there is a specific need
- Do not write things in shell scripts that can be written in Nix

## 3. Search for tools

- First check if the tool you want to use is in nixpkgs or an official flake
- If it is in both, use nixpkgs unless the versions differ significantly
- If it is in neither, present the option of packaging it yourself to the user and confirm whether to proceed

## 4. Validate

- A flake under git management does not read files that git is not tracking. Run `git add -N <files>` on new files (including `flake.lock`) before evaluation
- After evaluation is complete, use `git reset -- <files>` to revert only the files added via `git add -N`. Inform the user that `git add` is required to use the flake
- Pass `nix flake check --all-systems`
- If a devShell is being output, verify that the installed tool works with `nix develop -c <tool> --version`
- When package definitions are modified, verify they work with `nix run`
- If a `formatter` is being output, compare `git diff` before and after `nix fmt` to verify there are no unintended changes caused by formatting
