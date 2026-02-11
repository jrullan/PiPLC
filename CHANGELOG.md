# Changelog

All notable changes to PiPLC releases will be documented in this file.

## Unreleased

## v0.1.25 - 2026-02-10

- Remote context disconnect option (without stopping the engine), with route warning.
- Explorer HAT Pro plugin RTTI fix by linking piplc-core.
- Docker ARM64 build now deploys I/O provider plugins into the .deb.
- HMI Properties panel no longer shows the internal ID field.


## v0.1.21 - 2026-02-08

- Modbus TCP Server support in engine (flags for server/port/unit-id).
- Global PIC to fix Linux ARM plugin builds.


## v0.1.20 - 2026-02-08

- Plugin developer docs updated with cross-references and TODOs.
- Architecture plan marked completed and links fixed.


## v0.1.18 - 2026-02-07

- HMI auto-switches to Runtime mode when auto-connecting to a running engine.


## v0.1.17 - 2026-02-07

- Windows: double-clicking .plcproj now opens the project in PiPLC.


## v0.1.15 - 2026-02-07

- Tools toolbar icons and HMI UI polish.
- Fixed HMI analog range persistence and decorator PRE live display.
- MOV preset handling fix and unit test fixes.


## v0.1.14 - 2026-02-06

- Tools toolbar icons and HMI UI/icon polish.
- Fixed HMI analog range persistence and icon loading.
- Decorator glyph PRE live display now updates at runtime.
- MOV timer/counter preset handling fix, plus test fixes.


## v0.1.13 - 2026-02-06

- HMI toolbar: New/Open/Save buttons and refreshed icon set.
- HMI element palette icons (13 SVGs) and toolbar action icons.
- Fixed HMI icon loading, grid toggle, and unsaved-change prompts.
- Properties panel hides controls when no element is selected.


## v0.1.9 - 2026-02-06

- Remote context becomes active when opened.
- HMI input elements initialize with PLC values at Runtime start.
- HMI rename (formerly HMI Simulator) across the UI.
- HMI Slider integer write bug fixed for N: region.


## v0.1.5 - 2026-02-05

- HMI Simulator with interactive operator panel editor and runtime.
- Standalone PiPLC-HMI application using shared library architecture.


## v0.1.4 - 2026-02-04

- Improved validation UX and power flow readability.
- Default layout adjustments for a better first-run experience.


## v0.1.3 - 2026-02-04

- Optimized Explorer HAT Pro ADC scanning.
- Centralized version handling for consistency.


## v0.1.2 - 2026-02-04

- Explorer HAT Pro I/O provider support.
- New instructions: BSL, BSR, N2B, plus expanded word-address support.
- Fixed force simulation, branch OTE execution, and cursor positioning.


## v0.1.1 - 2026-02-04

- Windows installer now launches without an extra debug terminal (Release build).


## v0.1.0 - 2026-02-04

- First public release with Windows and Raspberry Pi installers.


- Initial public repository with release-only content and user documentation.

