+++
date = '2025-11-02T15:00:18+01:00'
lastmod = '2025-11-12T15:31:00+01:00'
draft = false
title = 'Building a local time registration application for personal use'
+++

I started this project as a weekend experiment: could I build a tiny timer that lives in Waybar, stays out of my way, and still lets me record billable hours for multiple customers? That curiosity quickly grew into a .NET 9 solution with a desktop app, CLI, and API hosts that all share the same domain services. In this post I walk through the journey—how I shaped the architecture, when I leaned on ChatGPT’s Codex for UI scaffolding, how Waybar/Rofi workflows influenced the automation surface, and which guardrails now keep everything maintainable.

## Local-first goals, API-first architecture

My primary goal was a fast local workflow: register customers, track time with 15-minute (0.25 hour) rounding, and export data for client systems or invoices. My first attempt let the Avalonia UI and CLI open the SQLite database directly. That worked until both ran at the same time—then the CLI became painfully slow because of file locks and duplicated connection logic. I fixed the problem by centralizing everything behind a minimal REST API. The API applies EF Core migrations for SQLite and PostgreSQL, and every other host (desktop, CLI, Waybar helpers) talks to it through a shared client. With only one process touching the database, the CLI stays responsive even when the GUI is open, and I get multi-host behavior as a side effect.

I also keep nullable reference types disabled. In my opinion, the syntax only gives a false sense of safety because “non-nullable” fields still can be null at runtime, so I prefer the explicit approach and treat null-handling as a deliberate design choice. Every project in the solution keeps nullable annotations off.

To stay organized I built a classic layered architecture:

- `TimeTracker.Domain` houses plain aggregates and DTOs with zero infrastructure knowledge.
- `TimeTracker.Application` owns the timer orchestration (including the rounding rules), reporting queries, and repository abstractions.
- `TimeTracker.Persistence` implements those repositories with EF Core.
- `TimeTracker.Infrastructure` wires up providers and DI helpers so each host can remain thin.

Because each layer has a clear contract, swapping persistence providers or introducing a new UI mostly involves wiring, not rewriting business logic.

## The Waybar & Rofi feedback loop

The project still revolves around my Waybar workflow. The helper script in `scripts/waybar-timetracker.sh` shells out to the CLI, formats JSON for Waybar, and falls back to `rofi`, `wofi`, or `zenity` when I need a prompt for notes or project selection. Codex generated the original version of that script—including the prompt integrations—and I later layered on extra commands and personal tweaks to fit my setup. Recent improvements, like prompting for notes automatically when I start a session with an empty comment, came directly from daily use inside Waybar. Every refinement I make there feeds back into the CLI, which now exposes verbs such as `toggle`, `switch`, `comment`, and `project-menu` that other automations can call.

## Testing, migrations, and repeatable tooling

Supporting multiple databases in EF Core is usually my first pain point, so I automated it early. The `scripts/manage-migrations.sh` helper adds or removes migrations across every provider in one shot. Today I maintain SQLite and PostgreSQL, but the script makes it trivial to drop in another provider later. It restores the solution when needed, routes commands through the startup helpers in `tools/`, and lets Testcontainers spin up PostgreSQL automatically.

For verification I rely on `tests/TimeTracker.Application.Tests`, powered by xUnit, coverlet, and containerized fixtures. Running `dotnet test TimeTracker.sln --collect:"XPlat Code Coverage"` exercises the timer services against both providers and keeps rounding rules honest.

## Avalonia desktop & packaging

Avalonia gives me a cross-platform shell with an always-on-top mini controller that still feels native on Linux, macOS, and Windows. The desktop app reuses the API client, so it focuses on presenting timer state, managing projects, and exporting weekly/monthly CSV reports without duplicating business logic.

Packaging stays pragmatic:

- Linux builds ship as selfcontained binaries plus distro packages the following distros is currently implemented
  - Debian .deb via the build.sh script in packaging/deb
  - Arch via PKGBUILD in packaging/arch
- Wiondows via WiX located in packaging/windows, this is not tested, as I currently don't have access to a Windows environment.
- The CLI publishes as a single-file binary, which should make dotfile integration and CI usage painless.

### Designing the Avalonia UI

I leaned heavily on ChatGPT’s Codex to design the Avalonia UI. Most of the layout, bindings, and styling came from iterating with the assistant, while I limited myself to minor tweaks and polish passes to match my workflow. The approach let me focus on the domain logic and automation hooks while still shipping a desktop experience that feels cohesive across platforms.

If you want to try it yourself, you can clone the repo from my [Github](https://github.com/NorthernLightsDevel/TimeTracker), generate package and install it to your operating system and enable the timetracker.service.
From there it’s easy to integrate the timer into Hyprland status bars, Rofi menus, or even future macOS menu bar experiments.
