---
name: nix-expert
description: Comprehensive Nix guide covering flakes, best practices, overlays, unfree handling, and package management. Use when working with flake.nix, managing dependencies, creating dev shells, or solving Nix configuration issues.
---

# Nix Expert

Complete guide for Nix flakes development, covering core concepts, best practices, advanced patterns, and troubleshooting.

## Quick Start

**This skill is for ALL Nix-related tasks:**
- Creating or working with flakes
- Managing dependencies and overlays
- Setting up development shells
- Handling unfree packages
- Creating binary overlays
- Debugging Nix issues

## Core Commands

### Flake Management

```bash
# Initialize a new flake
nix flake init

# Update flake.lock (updates all inputs)
nix flake update

# Update specific input
nix flake update some-input

# Check flake validity
nix flake check

# Show flake metadata
nix flake metadata

# Show flake outputs
nix flake show
```

### Building and Running

Always prefix local flake paths with `path:` to ensure Nix uses all files:

```bash
# Build default package
nix build path:.

# Build specific package
nix build path:.#packageName

# Run default app
nix run path:.

# Run specific app
nix run path:.#appName

# Run remote flake
nix run github:numtide/treefmt
```

### Development Environments

```bash
# Enter dev shell
nix develop path:.

# Run command in dev shell
nix develop path:. --command make build

# Check environment
nix develop path:. --command env

# Use impure mode for unfree
nix develop path:. --impure
```

## Flake Structure

### Basic Flake

```nix
{
  description = "Project description";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs {
          inherit system;
        };
      in {
        packages.default = pkgs.hello;

        devShells.default = pkgs.mkShell {
          buildInputs = with pkgs; [
            git
            vim
          ];
        };
      });
}
```

### Standard Flake Pattern

```nix
{
  description = "My Project";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
    # Additional inputs
    rust-overlay.url = "github:oxalica/rust-overlay";
  };

  outputs = { self, nixpkgs, flake-utils, rust-overlay, ... }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs {
          inherit system;
          overlays = [
            rust-overlay.overlays.default
          ];
        };
      in {
        packages = {
          default = pkgs.hello;
          # More packages...
        };

        devShells.default = pkgs.mkShell {
          buildInputs = with pkgs; [
            rustc
            cargo
          ];
        };

        formatter = pkgs.nixpkgs-fmt;
      });
}
```

## Input Management

### Follows Pattern (Avoid Duplicate Nixpkgs)

When adding overlay inputs, use `follows` to share parent nixpkgs:

```nix
inputs = {
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  # Overlay follows parent nixpkgs
  some-overlay.url = "github:owner/some-overlay";
  some-overlay.inputs.nixpkgs.follows = "nixpkgs";

  # Chain through intermediate inputs
  another-overlay.url = "github:owner/another-overlay";
  another-overlay.inputs.nixpkgs.follows = "some-overlay";
};
```

**Important**: All inputs must be listed in outputs function:

```nix
outputs = { self, nixpkgs, some-overlay, another-overlay, ... }:
```

### Managing Multiple Nixpkgs Channels

```nix
inputs = {
  # Stable channel
  nixpkgs-stable.url = "github:NixOS/nixpkgs/nixos-24.05";
  
  # Unstable channel (default)
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
};
```

## Overlays

### Applying Overlays

```nix
let
  pkgs = import nixpkgs {
    inherit system;
    overlays = [
      overlay1.overlays.default
      overlay2.overlays.default
      # Inline overlay
      (final: prev: {
        myPackage = prev.myPackage.override { ... };
      })
    ];
  };
in
```

### Creating Custom Overlays

```nix
overlays.default = final: prev: {
  # Override existing package
  nginx = prev.nginx.override {
    modules = [ ... ];
  };

  # Add new package
  my-package = prev.callPackage ./package.nix { };
};
```

## Unfree Packages

### Option 1: nixpkgs-unfree (Recommended)

Use numtide/nixpkgs-unfree for EULA-licensed packages:

```nix
inputs = {
  nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  nixpkgs-unfree.url = "github:numtide/nixpkgs-unfree/nixos-unstable";
  nixpkgs-unfree.inputs.nixpkgs.follows = "nixpkgs";

  # Unfree overlay follows nixpkgs-unfree
  proprietary-tool.url = "github:owner/proprietary-tool-overlay";
  proprietary-tool.inputs.nixpkgs.follows = "nixpkgs-unfree";
};
```

Chain: `proprietary-tool` → `nixpkgs-unfree` → `nixpkgs`

### Option 2: User Config

Users add to `~/.config/nixpkgs/config.nix`:

```nix
{ allowUnfree = true; }
```

### Option 3: Specific Packages (Flake)

```nix
let
  pkgs = import nixpkgs {
    inherit system;
    config.allowUnfreePredicate = pkg: builtins.elem (lib.getName pkg) [
      "specific-package"
      "another-package"
    ];
  };
in
```

### Option 4: Impure Development

```bash
# With envrc
export NIXPKGS_ALLOW_UNFREE=1
nix develop --impure

# One-off command
NIXPKGS_ALLOW_UNFREE=1 nix run path:.
```

## Development Shells

### Basic Shell

```nix
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [
    nodejs
    python3
  ];

  shellHook = ''
    echo "Dev environment ready"
    export PROJECT_ROOT="$(pwd)"
  '';
}
```

### With Environment Variables

```nix
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [ postgresql ];

  DATABASE_URL = "postgres://localhost/dev";

  shellHook = ''
    export NODE_ENV="development"
  '';
}
```

### Native Dependencies (C Libraries)

```nix
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [
    openssl
    postgresql
  ];

  shellHook = ''
    export C_INCLUDE_PATH="${pkgs.openssl.dev}/include:$C_INCLUDE_PATH"
    export LIBRARY_PATH="${pkgs.openssl.out}/lib:$LIBRARY_PATH"
    export PKG_CONFIG_PATH="${pkgs.openssl.dev}/lib/pkgconfig:$PKG_CONFIG_PATH"
  '';
}
```

### Multi-System Shells

```nix
{
  inputs = { nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable"; };
  
  outputs = { self, nixpkgs }:
    let
      mkShell = system: pkgs: pkgs.mkShell {
        buildInputs = with pkgs; [ ... ];
      };
    in {
      devShells = {
        x86_64-linux.default = mkShell "x86_64-linux" nixpkgs.legacyPackages.x86_64-linux;
        aarch64-darwin.default = mkShell "aarch64-darwin" nixpkgs.legacyPackages.aarch64-darwin;
      };
    };
}
```

## Binary Overlays

When nixpkgs builds a community version lacking features:

### Binary Overlay Pattern

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = nixpkgs.legacyPackages.${system};
        version = "1.0.0";

        # Platform-specific binaries
        sources = {
          "x86_64-linux" = {
            url = "https://example.com/tool-linux-amd64-v${version}";
            sha256 = "sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=";
          };
          "aarch64-linux" = {
            url = "https://example.com/tool-linux-arm64-v${version}";
            sha256 = "sha256-BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=";
          };
          "x86_64-darwin" = {
            url = "https://example.com/tool-darwin-amd64-v${version}";
            sha256 = "sha256-CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC=";
          };
          "aarch64-darwin" = {
            url = "https://example.com/tool-darwin-arm64-v${version}";
            sha256 = "sha256-DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD=";
          };
        };

        source = sources.${system} or (throw "Unsupported system: ${system}");

        toolPackage = pkgs.stdenv.mkDerivation {
          pname = "tool";
          inherit version;

          src = pkgs.fetchurl {
            inherit (source) url sha256;
          };

          sourceRoot = ".";
          dontUnpack = true;

          installPhase = ''
            mkdir -p $out/bin
            cp $src $out/bin/tool
            chmod +x $out/bin/tool
          '';

          meta = with pkgs.lib; {
            description = "Tool description";
            homepage = "https://example.com";
            license = licenses.unfree;
            platforms = builtins.attrNames sources;
          };
        };
      in {
        packages.default = toolPackage;
        packages.tool = toolPackage;

        overlays.default = final: prev: {
          tool = toolPackage;
        };
      })
    // {
      overlays.default = final: prev: {
        tool = self.packages.${prev.system}.tool;
      };
    };
}
```

### Getting SHA256 Hashes

```bash
# Prefetch and get hash
nix-prefetch-url https://example.com/tool-linux-amd64-v1.0.0

# Get SRI format directly
nix-prefetch-url --type sha256 https://example.com/tool-linux-amd64-v1.0.0
```

## Package Management

### Creating Packages

```nix
{
  description = "Package description";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      packages.${system}.default = pkgs.callPackage ./package.nix { };
    };
}
```

### Package.nix

```nix
{ lib, stdenv, fetchurl, ... }:

stdenv.mkDerivation rec {
  pname = "mypackage";
  version = "1.0.0";

  src = fetchurl {
    url = "https://example.com/${pname}-${version}.tar.gz";
    sha256 = "sha256-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=";
  };

  buildInputs = [ ... ];
  nativeBuildInputs = [ ... ];

  buildPhase = ''
    make
  '';

  installPhase = ''
    make install PREFIX=$out
  '';

  meta = with lib; {
    description = "Package description";
    homepage = "https://example.com";
    license = licenses.mit;
    platforms = platforms.all;
  };
}
```

## Direnv Integration

### Basic .envrc

```bash
use flake
```

### With Unfree Support

```bash
export NIXPKGS_ALLOW_UNFREE=1
use flake --impure
```

### Watch Mode

```bash
watch_file flake.nix
watch_file shell.nix
use flake
```

## System Configuration

### NixOS Modules

```nix
# modules/my-module.nix
{ config, lib, pkgs, ... }:
{
  options.my-module.enable = lib.mkEnableOption "Enable my module";

  config = lib.mkIf config.my-module.enable {
    environment.systemPackages = with pkgs; [ ... ];
  };
}
```

### Home Manager Integration

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    home-manager.url = "github:nix-community/home-manager";
    home-manager.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = { self, nixpkgs, home-manager, ... }:
    {
      homeConfigurations."username" = home-manager.lib.homeManagerConfiguration {
        pkgs = nixpkgs.legacyPackages.x86_64-linux;
        modules = [ ./home.nix ];
      };
    };
}
```

## Best Practices

### Flake Purity

- **Use `path:` prefix** for local flake references
- **Manage flake.lock** for reproducibility
- **Track files in git** for purity

### Dependency Management

- **Use `follows`** to avoid duplicate nixpkgs downloads
- **Pin versions** in inputs for stability
- **Update regularly** but test after updates

### Code Organization

- **Separate concerns**: modules, packages, configs
- **Use helpers**: lib, utils, shared functions
- **Document overlays**: what they modify and why

### Performance

- **Cache builds**: use `nix-store` optimally
- **Parallel builds**: use `-j` flag
- **Minimize rebuilds**: use overlays wisely

## Troubleshooting

### "unexpected argument" Error

All inputs must be listed in outputs function:

```nix
# Wrong
outputs = { self, nixpkgs }: ...

# Right
outputs = { self, nixpkgs, other-input, ... }: ...
```

### Unfree Package Errors

`config.allowUnfree` in flake.nix doesn't work with `nix develop`. Use:
1. nixpkgs-unfree input (recommended)
2. User's `~/.config/nixpkgs/config.nix`
3. `NIXPKGS_ALLOW_UNFREE=1 nix develop --impure`

### Duplicate Nixpkgs Downloads

Use `follows` to chain inputs to a single nixpkgs source.

### Overlay Not Applied

Ensure overlay is in overlays list:

```nix
pkgs = import nixpkgs {
  inherit system;
  overlays = [ my-overlay.overlays.default ];
};
```

### Hash Mismatch

Re-fetch with `nix-prefetch-url` and update. Hashes change when upstream updates binaries.

### Store Path Issues

```bash
# Find dependencies
nix-store --query --requisites /nix/store/...

# Find reverse dependencies
nix-store --query --referrers /nix/store/...

# Remove garbage
nix-collect-garbage -d
```

## Advanced Topics

### Cross-Compilation

```nix
{
  description = "Cross-compile example";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs }:
    let
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
      armPkgs = import nixpkgs {
        localSystem = "x86_64-linux";
        crossSystem = "aarch64-linux";
      };
    in {
      packages = {
        x86_64-linux.hello = pkgs.hello;
        aarch64-linux.hello = armPkgs.hello;
      };
    };
}
```

### Custom Builders

```nix
{
  description = "Custom builder";

  outputs = { self, nixpkgs }:
    let
      pkgs = nixpkgs.legacyPackages.x86_64-linux;

      buildDockerImage = { name, tag, contents }:
        pkgs.dockerTools.buildImage {
          inherit name tag contents;
          config = {
            Cmd = [ "bin/${name}" ];
          };
        };
    in {
      packages = {
        docker-image = buildDockerImage {
          name = "my-app";
          tag = "latest";
          contents = [ pkgs.hello ];
        };
      };
    };
}
```

### CI/CD Integration

```yaml
# GitHub Actions
name: Nix CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: cachix/install-nix-action@v20
        with:
          extra_nix_config: |
            experimental-features = nix-command flakes
      - uses: cachix/cachix-action@v10
        with:
          name: mycache
          authToken: '${{ secrets.CACHIX_AUTH_TOKEN }}'
      - run: nix build
      - run: nix flake check
```

## Related Tools

- **direnv**: Automatic shell integration
- **search-nix-packages**: Search for NixOS packages
- **search-nix-options**: Find NixOS configuration options
- **nix-index**: Fast package searching
- **nix-diff**: Compare store generations
- **nvd**: Nix version diff tool

## Common Patterns Reference

### Dev Shell with Tests

```nix
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [
    nodejs
    nodePackages.prettier
  ];

  shellHook = ''
    export NODE_ENV="test"
    
    # Run prettier on shell exit
    trap 'npx prettier --write .' EXIT
  '';
}
```

### Multi-Language Support

```nix
devShells.default = pkgs.mkShell {
  buildInputs = with pkgs; [
    # Node.js
    nodejs
    nodePackages.typescript

    # Python
    python3
    python3Packages.pip

    # Rust
    rustc
    cargo

    # Go
    go
  ];
}
```

### Development + Production

```nix
{
  outputs = { self, nixpkgs, ... }:
    let
      pkgs = import nixpkgs { inherit system; };
    in {
      # Dev shell with all tools
      devShells.default = pkgs.mkShell {
        buildInputs = with pkgs; [ ... ];
      };

      # Production package (minimal)
      packages.prod = pkgs.buildEnv {
        name = "prod-env";
        paths = with pkgs; [ nginx certbot ];
      };
    };
}
```

## Resources

- [Nix Manual](https://nixos.org/manual/nix/stable/)
- [NixOS Options Search](https://search.nixos.org/options)
- [Nixpkgs Package Search](https://search.nixos.org/packages)
- [Flakes Guide](https://nixos.wiki/wiki/Flakes)
- [Nix Pills](https://nixos.org/guides/nix-pills/)
- [Nixery](https://nixery.dev) - Docker-like images built with Nix
