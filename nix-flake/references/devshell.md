# Development environment flake

Build the flake around `devShells`, adding `formatter` and `checks` as needed.

## Files to create

- `flake.nix`
- `.envrc`: `use flake`
- `.gitignore`: Add `.direnv/`

## What to include in devShell

Inspect the project files (`package.json`, `Cargo.toml`, `rust-toolchain.toml`, `pyproject.toml`, CI workflows, etc.), check what is actually being used, and then include them.

- Include tools used in the project that are not managed by other tools
- Do not include things that package managers or toolchain management tools install and manage. For example, do not include the following:
  - Dependencies in `package.json` (such as wrangler)
- Match the versions to those specified in the project (`engines`, `packageManager`, `rust-toolchain.toml`, etc.), if any

### Environment-specific notes

- Node.js
  - Use `nodejs-slim_<version>` if npm, npx, and corepack are not used
  - Prefer the LTS version
- Vite+
  - For `vp`, use `packages.<system>.vp` from [ryoppippi/nix-vite-plus](https://github.com/ryoppippi/nix-vite-plus)
- Cloudflare Workers (wrangler)
  - workerd in `wrangler dev` reads certificates from `SSL_CERT_FILE`
  - Export `SSL_CERT_FILE` and `NIX_SSL_CERT_FILE` to `${pkgs.cacert}/etc/ssl/certs/ca-bundle.crt` in `shellHook`
    - Passing them via mkShell attributes or `env` results in `nix develop` not passing `SSL_CERT_FILE` to the shell
- Rust
  - Include `cargo`, `rustc`, `clippy`, `rustfmt`, and `rust-analyzer` as needed
  - Add inputs like nix-community/fenix only when a toolchain not available in nixpkgs (such as a nightly rustfmt) is required

## Formatter and checks

The formatters to use for each language are as follows:

| Target                      | Formatter                                    |
| --------------------------- | -------------------------------------------- |
| Nix                         | nixfmt (deadnix, statix)                     |
| Rust                        | rustfmt                                      |
| TOML                        | taplo                                        |
| Markdown, JSON, YAML, ...   | oxfmt                                        |

- Formatters managed by the project's package manager (such as ruff installed via uv or oxfmt installed via pnpm) should not be handled by the flake
- If there is only a single formatter, output that package to `formatter`
- When using multiple formatters, consolidate them using treefmt-nix and output them to `formatter` and `checks`
- Only add git-hooks.nix hooks to the devShell's `shellHook` when instructed by the user

## Setting up a development environment in repositories not using Nix

Use the development environment locally without adding files to the repository.

- Place the flake in `nix/`
- Set `.envrc` to `use flake path:./nix`. Since `path:` reads the directory as-is, it can evaluate flakes even if they are not tracked by git
- Add `.envrc`, `nix/`, and `.direnv/` to `.git/info/exclude`

## Deploying agent skills

- Use [agent-skills-nix](https://github.com/Kyure-A/agent-skills-nix) and deploy them via the `shellHook` of the devShell in the same `flake.nix`
- Add the deployment destination (such as `.claude/skills/`) to `.gitignore`
