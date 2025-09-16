# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building

```bash
# Build with X11 linking (required)
RUSTFLAGS="-lX11" cargo build --release

# Install to system
sudo cp target/release/simple-x11-remapper /usr/local/bin/
```

### Testing

```bash
# Run unit tests
cargo test

# Test with debug logging
RUST_LOG=debug ./target/debug/simple-x11-remapper practical_config.yaml
```

### Running

```bash
# Basic usage
./target/release/simple-x11-remapper config.yaml

# With debug output for troubleshooting
RUST_LOG=debug simple-x11-remapper config.yaml

# May need root for key grabbing on some X11 setups
sudo simple-x11-remapper config.yaml
```

## Architecture Overview

This is a Rust X11 key remapper with YAML configuration. The codebase follows a modular architecture:

### Core Modules

- **main.rs**: Entry point, X11 display initialization, main event loop
- **config.rs**: YAML configuration parsing with support for window-specific rules
- **event_handler.rs**: Central coordinator that processes X11 events and manages key mappings
- **key_mapper.rs**: Handles key string parsing, modifier combinations, and X11 key event generation
- **window_manager.rs**: Manages active window detection and window class name extraction
- **lib.rs**: Public module exports

### Key Design Patterns

- **Event-driven architecture**: Main loop listens for X11 KeyPress, PropertyNotify, and MappingNotify events
- **Dynamic key grabbing**: Keys are grabbed/ungrabbed based on active window and applicable rules
- **Window-aware remapping**: Different key mappings apply based on window class (class_only/class_not filters)
- **Multi-key sequences**: Single key press can trigger multiple key outputs

### Configuration System

YAML configuration supports:

- Global rules (no window filter) that apply to all windows
- `class_only`: Rules that apply only to specific application window classes
- `class_not`: Rules that apply to all windows except specified classes
- Single key remaps: `'C-b': 'Left'`
- Multi-key sequences: `'C-k': ['Shift-End', 'Ctrl-x']`

### X11 Integration

- Uses raw X11 bindings (x11 crate) for low-level key grabbing and event handling
- Custom X11 error handler to gracefully handle grab failures
- Window class detection via `WM_CLASS` and `_NET_ACTIVE_WINDOW` properties
- Modifier key filtering and normalization

### Build System

- **build.rs**: Links with X11 library at compile time
- **Cargo.toml**: Uses x11, serde_yaml, anyhow, log, and env_logger crates

## Configuration Examples

Reference files:

- `practical_config.yaml`: Production-ready config with global fallbacks
- `test_config.yaml`: Minimal config for testing

## Troubleshooting

When debugging key remapping issues:

1. Use `RUST_LOG=debug` to see detailed event processing
2. Look for "Found handler for keycode=X" messages to confirm remapping is working
3. "Failed to grab key" warnings can often be ignored if remapping still works
4. Check window class detection by monitoring window change messages
