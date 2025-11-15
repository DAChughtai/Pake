# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Philosophy

- **Incremental progress over big bangs**: Break complex tasks into manageable stages
- **Learn from existing code**: Understand patterns before implementing new features
- **Clear intent over clever code**: Prioritize readability and maintainability
- **Simple over complex**: Keep all implementations simple and straightforward - prioritize solving problems and ease of maintenance over complex solutions

## Claude Code Eight Honors and Eight Shames

- **Shame** in guessing APIs, **Honor** in careful research
- **Shame** in vague execution, **Honor** in seeking confirmation
- **Shame** in assuming business logic, **Honor** in human verification
- **Shame** in creating interfaces, **Honor** in reusing existing ones
- **Shame** in skipping validation, **Honor** in proactive testing
- **Shame** in breaking architecture, **Honor** in following specifications
- **Shame** in pretending to understand, **Honor** in honest ignorance
- **Shame** in blind modification, **Honor** in careful refactoring

## Project Overview

Pake transforms any webpage into a lightweight desktop app using Rust and Tauri. It's significantly lighter than Electron (~5M vs ~100M+) with better performance.

**Core Architecture:**

- **CLI Tool** (`bin/`): TypeScript-based command interface
- **Tauri App** (`src-tauri/`): Rust desktop framework
- **Injection System**: Custom CSS/JS injection for webpages

## Development Workflow

1. **Understand**: Study existing patterns in codebase
2. **Plan**: Break complex work into 3-5 stages
3. **Test**: Write tests first (when applicable)
4. **Implement**: Minimal working solution
5. **Refactor**: Optimize and clean up

**Key Commands:**

```bash
pnpm test           # Run comprehensive test suite
pnpm run cli:build  # Build CLI for testing
pnpm run dev        # Development with hot reload
```

**Testing:**

- Always run `pnpm test` before committing
- For CLI testing: `node dist/cli.js https://example.com --name TestApp --debug`
- For app functionality testing: Use `pnpm run dev` for hot reload

## Repository Structure

```
Pake/
├── bin/                           # CLI tool (TypeScript)
│   ├── cli.ts                    # Main CLI entry point
│   ├── builders/                 # Platform-specific builders
│   │   ├── BaseBuilder.ts        # Shared build logic
│   │   ├── MacBuilder.ts         # macOS DMG/app packaging
│   │   ├── WinBuilder.ts         # Windows MSI packaging
│   │   └── LinuxBuilder.ts       # Linux deb/AppImage/rpm packaging
│   ├── options/                  # CLI option processing
│   │   ├── icon.ts              # Icon conversion & fetching
│   │   └── index.ts             # Option validation & handling
│   ├── helpers/                  # Utility functions
│   └── utils/                    # Validation & common utilities
│
├── src-tauri/                     # Rust/Tauri application
│   ├── src/
│   │   ├── app/                  # Core application logic
│   │   │   ├── config.rs         # Config loading/management
│   │   │   ├── invoke.rs         # IPC command handlers
│   │   │   ├── setup.rs          # App initialization
│   │   │   └── window.rs         # Window management
│   │   ├── inject/               # Web injection system
│   │   │   ├── event.js          # Keyboard shortcuts, downloads, notifications
│   │   │   ├── style.js          # UI styling & customization
│   │   │   ├── component.js      # Custom UI components
│   │   │   └── custom.js         # User custom injections
│   │   ├── lib.rs               # Library exports
│   │   ├── main.rs              # Binary entry point
│   │   └── util.rs              # Rust utilities
│   ├── Cargo.toml               # Rust dependencies
│   ├── tauri.conf.json          # Base Tauri config
│   ├── tauri.*.conf.json        # Platform-specific configs
│   └── pake.json                # Pake runtime config (windows, tray, UA, etc.)
│
├── tests/                         # Test suite
│   ├── index.js                  # Test runner
│   ├── config.js                 # Test configuration
│   ├── github.js                 # GitHub integration tests
│   ├── release.js                # Release process tests
│   └── unit/                     # Unit tests
│
├── docs/                          # Documentation
│   ├── cli-usage.md             # Complete CLI reference
│   ├── advanced-usage.md        # Style customization, injection
│   ├── faq.md                   # Common issues
│   └── github-actions-usage.md  # Online building guide
│
├── scripts/                       # Build & automation scripts
├── dist/                         # Compiled CLI output
└── .github/workflows/            # CI/CD pipelines
    ├── quality-and-test.yml     # Code quality & testing
    ├── pake-cli.yaml            # CLI publishing
    └── release.yml              # Release automation
```

## Core Components

### CLI Tool (`bin/`)

The TypeScript-based CLI provides the user-facing interface for packaging websites:

- **cli.ts**: Commander.js-based CLI with ~30 options (width, height, icon, tray, shortcuts, etc.)
- **Builders**: Platform-specific packaging logic
  - `BaseBuilder.ts`: Shared functionality (config generation, icon handling, Tauri invocation)
  - Platform builders extend base with DMG/MSI/deb generation
- **Options Processing**: Icon fetching/conversion, URL validation, config merging

### Tauri Application (`src-tauri/`)

Rust-based desktop framework that wraps webpages:

- **app/config.rs**: Loads `pake.json` for window settings, user agents, tray configuration
- **app/window.rs**: Window creation, size/position, title bar handling, multi-window support
- **app/invoke.rs**: IPC commands for frontend-backend communication
- **app/setup.rs**: App initialization, single instance, global shortcuts, tray icon

**Key Tauri Plugins** (see `Cargo.toml`):

- `tauri-plugin-window-state`: Save/restore window position
- `tauri-plugin-global-shortcut`: Custom keyboard shortcuts
- `tauri-plugin-shell`: Secure shell command execution
- `tauri-plugin-notification`: System notifications
- `tauri-plugin-single-instance`: Prevent multiple instances
- `tauri-plugin-oauth`: OAuth authentication flows
- `tauri-plugin-http`: Enhanced HTTP client

### Injection System (`src-tauri/src/inject/`)

JavaScript injected into packaged webpages for enhanced functionality:

- **event.js** (~26KB): Core functionality
  - Keyboard shortcuts (Cmd+[/], Cmd+R, zoom controls)
  - Download handling & progress tracking
  - Custom context menus
  - Auto-scroll, fullscreen toggle
  - IPC bridge for Rust backend
- **style.js** (~15KB): UI customization
  - Title bar styling (macOS traffic lights)
  - Scrollbar customization
  - Dark mode support
  - Platform-specific CSS injections
- **component.js**: Reusable UI components (download manager, etc.)
- **custom.js**: User-provided custom injections (empty by default)

### Configuration System

**pake.json**: Runtime configuration loaded by Rust app

- `windows[]`: Array of window configs (URL, size, behavior)
- `user_agent`: Platform-specific user agent strings
- `system_tray`: Tray icon enable/disable per platform
- `inject[]`: Custom CSS/JS injection paths
- `proxy_url`: HTTP proxy settings
- `multi_instance`: Allow multiple app instances

**tauri.conf.json**: Build-time Tauri configuration

- Application metadata (name, version, author)
- Bundle settings (identifier, icon, resources)
- Window defaults
- Security policies (CSP, allowed APIs)

**Platform configs**: Override base config for macOS/Windows/Linux

## Common Development Patterns

### Adding a New CLI Option

1. **Add option to `bin/cli.ts`**: Use `.option()` with validator if needed
2. **Update `bin/defaults.ts`**: Add default value
3. **Update `bin/types.ts`**: Add to `PakeCliOptions` interface
4. **Handle in builder**: Update `BaseBuilder.ts` to use new option
5. **Update configs**: Modify `pake.json` template if runtime config needed
6. **Document**: Add to `docs/cli-usage.md` with examples

### Modifying Injection Behavior

1. **For keyboard shortcuts**: Edit `src-tauri/src/inject/event.js`
2. **For styling changes**: Edit `src-tauri/src/inject/style.js`
3. **For UI components**: Edit `src-tauri/src/inject/component.js`
4. **Test**: Run `pnpm run dev` to see changes with hot reload
5. **Production**: Rebuild CLI with `pnpm run cli:build`

### Adding Rust Functionality

1. **Add command handler**: Create function in `src-tauri/src/app/invoke.rs`
2. **Register command**: Add to `invoke_handler![]` in `src-tauri/src/app/setup.rs`
3. **Frontend call**: Use `@tauri-apps/api` to invoke from JavaScript
4. **Update types**: Add to TypeScript definitions if needed

### Platform-Specific Builds

- **macOS**: `pnpm run build:mac` creates universal binary (x86_64 + aarch64)
- **Windows**: `pnpm run build` on Windows creates MSI installer
- **Linux**: `pnpm run build` creates deb/AppImage/rpm (configurable in tauri.conf.json)
- **Debug builds**: `pnpm run build:debug` for faster compilation during development

## Testing & Quality

### Test Suite Structure

- **Unit tests**: `tests/unit/` - Test individual helper functions
- **Integration tests**: `tests/*.js` - Test full CLI workflow
- **GitHub integration**: `tests/github.js` - Test icon fetching, URL validation
- **Release tests**: `tests/release.js` - Verify build artifacts

### Running Tests

```bash
# Complete test suite (builds CLI + runs all tests)
pnpm test

# Build CLI only
pnpm run cli:build

# Test specific workflow
PAKE_CREATE_APP=1 node tests/github.js

# Format check (CI requirement)
pnpm run format:check

# Auto-format code
pnpm run format
```

### Continuous Integration

**quality-and-test.yml**: Runs on all PRs

- Code formatting validation (Prettier for TS/JS, cargo fmt for Rust)
- Full test suite execution on Ubuntu
- Multi-platform compatibility checks

**pake-cli.yaml**: Publishes CLI to npm

- Triggered on version tags or manual dispatch
- Builds CLI bundle with rollup
- Publishes to npm registry

**release.yml**: Coordinates releases

- Builds apps for all platforms (macOS/Windows/Linux)
- Creates GitHub releases with binaries
- Publishes Docker images
- Manages version bumping

## Documentation Guidelines

- **Main README**: Common parameters only
- **CLI Documentation** (`docs/cli-usage.md`): ALL parameters with examples
- **Rare parameters**: Full docs in CLI usage, minimal in main README
- **NO technical documentation files**: Do not create separate technical docs, design docs, or implementation notes - keep technical details in memory/conversation only

## Platform Specifics

- **macOS**: `.icns` icons, universal builds with `--multi-arch`
- **Windows**: `.ico` icons, requires Visual Studio Build Tools
- **Linux**: `.png` icons, multiple formats (deb, AppImage, rpm)

## Quality Standards

**Code Standards:**

- Prefer composition over inheritance
- Use explicit types over implicit
- Write self-documenting code
- Follow existing patterns consistently
- **NO Chinese comments** - Use English only
- **NO unnecessary comments** - For simple, obvious code, let the code speak for itself

**Git Guidelines:**

- **NEVER commit automatically** - User handles all git operations
- **NEVER generate commit messages** - User writes their own
- Only make code changes, user decides when/how to commit
- Always test before user commits

## Branch Strategy

- `dev` - Active development, target for PRs
- `main` - Release branch for stable versions

## Important Implementation Details

### Icon Handling (`bin/options/icon.ts`)

- **Auto-fetch**: If no icon provided, fetches website favicon
- **Conversion pipeline**:
  - Downloads icon from URL or uses local file
  - Converts to platform-specific format (.icns for macOS, .ico for Windows, .png for Linux)
  - Uses `sharp` for image processing, `icon-gen` for format conversion
- **Default fallback**: Uses Pake logo if fetch fails

### Build Process Flow

1. **CLI invocation** → `bin/cli.ts` parses options
2. **Builder selection** → Platform-specific builder (Mac/Win/Linux) instantiated
3. **Config generation** → Creates `pake.json` and `tauri.conf.json` in temp directory
4. **Icon conversion** → Processes icon for target platform
5. **File preparation** → Copies `src-tauri/` to temp build directory
6. **Tauri build** → Invokes `@tauri-apps/cli` with platform configs
7. **Artifact collection** → Moves DMG/MSI/deb to output directory

### Configuration Merging

- **Base config**: `src-tauri/tauri.conf.json` (checked into repo)
- **Platform overrides**: `tauri.macos.conf.json`, `tauri.windows.conf.json`, `tauri.linux.conf.json`
- **CLI options**: Merged at build time to override defaults
- **Runtime config**: `pake.json` loaded by Rust app at startup

### Version Management

- **Package version**: `package.json` v3.5.2 (CLI version, published to npm)
- **Cargo version**: `Cargo.toml` v3.1.2 (Rust crate version)
- **Tauri version**: v2.9.0 (framework version, defined in dependencies)
- **Rust MSRV**: 1.85.0 (minimum supported Rust version in Cargo.toml)

### Environment Variables

- `PAKE_CREATE_APP=1`: On macOS, create `.app` bundle instead of DMG (for testing)
- `NODE_ENV=development`: Used by rollup for CLI builds
- `MACOSX_DEPLOYMENT_TARGET`: Override deployment target (see CONTRIBUTING.md for macOS 26 Beta)

## Prerequisites

- **Node.js** ≥22.0.0 (≥18.0.0 may work, package.json specifies ≥18.0.0)
- **Rust** ≥1.89.0 (≥1.78.0 may work, MSRV in Cargo.toml is 1.85.0)
- **pnpm** 10.15.0 (specified in packageManager field)
- **Platform build tools**:
  - **macOS**: Xcode Command Line Tools (`xcode-select --install`)
  - **Windows**: Visual Studio Build Tools with MSVC
  - **Linux**: `build-essential`, `libwebkit2gtk-4.0-dev`, system dependencies
  - See CONTRIBUTING.md for detailed platform setup

## Troubleshooting & Common Pitfalls

### Build Issues

- **Rust not found**: CLI auto-installs Rust if missing, but may need terminal restart to reload PATH
- **macOS 26 Beta errors**: Create `src-tauri/.cargo/config.toml` with SDK override (see CONTRIBUTING.md)
- **Cargo compilation errors**: Run `cd src-tauri && cargo clean` then retry
- **Node dependency issues**: Delete `node_modules` and `pnpm-lock.yaml`, run `pnpm install`

### Development Workflow Issues

- **Changes not reflected**: After modifying injection files, must run `pnpm run cli:build` before testing CLI
- **Hot reload not working**: `pnpm run dev` only watches Rust/Tauri changes, not injection JS files
- **DMG creation slow on macOS**: Use `PAKE_CREATE_APP=1` to create .app bundle for faster testing

### Configuration Pitfalls

- **pake.json vs tauri.conf.json**: Runtime vs build-time configs - know which to modify
- **Platform configs not applying**: Ensure filename matches pattern `tauri.{platform}.conf.json`
- **Icon not updating**: Icon is embedded at build time, must rebuild to see changes
- **Custom.js not loaded**: Must be explicitly added to `inject[]` array in pake.json

### Code Quality

- **Format before commit**: Always run `pnpm run format` - CI will fail on formatting issues
- **Test before PR**: Run `pnpm test` - includes CLI build, unit tests, and integration tests
- **TypeScript compilation**: Use `pnpm run cli:build` to catch type errors before runtime

### Cross-Platform Considerations

- **Path separators**: Use `path.join()`, not hardcoded `/` or `\\`
- **User agents**: Platform-specific in `pake.json`, don't hardcode
- **Keyboard shortcuts**: Cmd on macOS, Ctrl on Windows/Linux - handled in `event.js`
- **System tray**: Disabled by default on macOS, enabled on Windows/Linux

## Key Files to Understand

When working on Pake, these are the most frequently modified files:

1. **bin/cli.ts** - CLI interface, option parsing (add new CLI flags here)
2. **bin/builders/BaseBuilder.ts** - Core build logic shared across platforms
3. **src-tauri/src/inject/event.js** - Keyboard shortcuts, downloads, IPC bridge
4. **src-tauri/src/inject/style.js** - UI styling and customization
5. **src-tauri/src/app/window.rs** - Window management and behavior
6. **src-tauri/src/app/config.rs** - Configuration loading from pake.json
7. **src-tauri/pake.json** - Runtime configuration template
8. **package.json** - Scripts, dependencies, version (CLI)
9. **src-tauri/Cargo.toml** - Rust dependencies, version (app)

## Quick Reference

### File Reference Format

When referencing code locations, use: `file_path:line_number` (e.g., `src-tauri/src/app/window.rs:42`)

### Common Commands Cheat Sheet

```bash
pnpm run dev              # Development with hot reload
pnpm run cli:build        # Build CLI
pnpm test                 # Run all tests
pnpm run format           # Auto-format code
pnpm run build            # Build production app
pnpm run build:mac        # macOS universal build
pnpm run build:debug      # Debug build (faster)
node dist/cli.js --help   # Test CLI locally
```

### Important Directories

- `bin/` → TypeScript CLI tool
- `src-tauri/src/` → Rust application code
- `src-tauri/src/inject/` → JavaScript injections
- `tests/` → Test suite
- `docs/` → User documentation
- `.github/workflows/` → CI/CD pipelines
