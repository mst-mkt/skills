# Package distribution flake

Expose the package as outputs so that users can consume it in whichever way fits their setup.

## Outputs

Output only what is necessary.

- `packages.<system>.default`: Used directly with `nix run` or from inputs.
- `overlays.default`: Adds the package to the user's `pkgs`. Define the package in the overlay as `final.callPackage ./package.nix { }` so that dependencies resolve against the user's `pkgs`. Do not reference the flake's own `packages` output, as that bypasses the user's overlays.
- `homeModules.default`: Configures tools that have configuration files using Home Manager.
- `checks`: For verifications that building alone cannot cover.
- `devShells`: Can be placed in the same flake if development inputs do not increase.

The default value for the `package` option in a Home Manager module should be `pkgs.callPackage ./package.nix { }`, with `defaultText` set to a matching `lib.literalExpression`, so that the same package definition is built from the user's `pkgs` unless they set `package` explicitly. If the flake's own `packages` output is used as the default value, the user's overlays will not take effect.

## Writing packages

- If the definition is short, write it in `flake.nix`. If it is long, separate it into `package.nix` or files under `nix/` and call it with `callPackage`.
- Use language-specific builders found in nixpkgs (`buildGoModule`, `rustPlatform.buildRustPackage`, etc.).
- When standard builders are not enough, such as when you want to share dependency build results with clippy or test checks, use a build library like ipetkov/crane.

## Hashes

Hashes are required for fixed-output derivations that fetch dependencies (`fetchPnpmDeps`, `vendorHash` in `buildGoModule`, etc.).

- If there is a way to read dependencies from a lock file, use that to make hashes unnecessary (`cargoLock.lockFile` in `rustPlatform`, etc.).
- Pass sources via `./.` or flake inputs, and do not write hashes for fetchers.
- When passing `./.`, narrow it down only to the files used for the build using `lib.fileset.toSource`.
- If not narrowed down, rebuilding will be triggered just by changing files unrelated to the build, such as the README.
- When a hash is required, build with `lib.fakeHash` and use the value shown in the error. Do not guess and write hashes.

## Separating the development flake

Inputs used only for development (formatters, git hooks, development tool flakes) will also end up in the user's `flake.lock`. To avoid this, separate the development outputs into `dev/flake.nix`.

- Set `.envrc` to `use flake ./dev`.
- `dev/flake.nix` does not take the root flake as an input and has its own nixpkgs.
- There is no need to separate them if the development input is only nixpkgs.
