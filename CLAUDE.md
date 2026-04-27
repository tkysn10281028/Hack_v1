# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Working title: **Hack** (temporary).

Digital board game / tactical simulation built with Unity 6 (6000.4.1f1). The game takes place in a fictional cyberspace where players compete over a finite pool of information resources represented as nodes in a network graph.

Players move between linked nodes on a fixed-size map (11×11 nodes) and overwrite node ownership each turn. Ownership can always be rewritten, so the game becomes a tug-of-war over nodes. At the end of the match, the player controlling the most nodes wins.

Each player also has three categories of cards — **Virus**, **Hacker**, and **Defense** — which can:

- Disrupt the opponent’s movement,
- Strengthen their own position,
- Manipulate node ownership or player resources.

Every player has a **Master Node** assigned. If you capture the opponent’s Master Node, you immediately take control of all nodes they currently own.

Information about opponents is intentionally limited: you can see their current position, node ownership, and links, but you cannot see hidden Virus/Defense cards placed on nodes or links. This creates bluffing and mind-game elements.

The design goals are:

- **Psychological play and mind games**:  
  You must infer the opponent’s intentions from limited information (e.g., “did they trap this link with a virus?”), and balance offensive expansion versus protecting your Master Node.

- **Interesting card mechanics**:  
  Cards are inspired by real-world computer viruses and cyberattack techniques. Effects vary in power and cost so that “weaker” cards are cheaper or more flexible, while strong cards come with higher costs. Card activation should have satisfying visual feedback.

- **Uncommon theme**:  
  Cyber warfare, computer viruses, and network security as the core motif — a concept that is rarely explored in this form.

- **Story mode hooks**:  
  The in-universe game started as a security company simulation tool for modeling real cyber attacks in virtual space. It later became popular as a game that even elementary school students could enjoy. A shadow organization begins creating original computer viruses inside the game and deploying them in the real world; the protagonist tries to stop them.

Core rule highlights:

- **Normal nodes**:  
  Ownership can be cleared regardless of who currently owns the node.

- **Master Node**:  
  - Only the owner can see where their Master Node is.  
  - In score calculation at game end, Master Nodes count as owned, but initially appear “unowned” to the opponent.  
  - You can mark your own Master Node as “owned” so that it appears owned to the opponent, without changing the underlying ownership logic.  

- **Turn / actions**:  
  - Each player has a fixed pool of 5 action points per turn.  
  - Once all players finish their actions, one turn passes.

- **Cards**:
  - Virus cards: only Virus cards explicitly notify the opponent when activated.  
  - Virus / Hacker / Defense cards can be placed on nodes or links depending on card type, with various effects (stealing nodes, draining actions, revealing opponent cards, blocking paths, etc.).

- **Maps**:
  - Multiple fixed maps.  
  - Some stages have every node connected; some have only partial connectivity.  
  - Base grid layout is 11×11 nodes.

- **Tutorials**:
  - Links & nodes only: 5 turns, fixed initial placement.  
  - With Master Node: 5 turns, Master Node is explicitly shown at first.  
  - With Defense cards: 10 turns.  
  - With Hacker cards: 10 turns.  
  - With Virus cards: 10 turns.

The overall feel is closer to a **turn-based territory control / board game in cyberspace** rather than an action game.

## Development Environment

- **Engine**: Unity 6000.4.1f1
- **Language**: C# (.NET)
- **Key packages**: `com.unity.feature.2d`, `com.unity.timeline`
- **Input**: Standard Unity event system & UI, not the new Input System action maps
- **IDE**: VS Code or JetBrains Rider (both configured)

There is no build CLI — compile and run through the Unity Editor. Domain logic can be unit-tested via Unity Test Framework (Edit Mode tests run without the Editor playing).

## Architecture

Scripts are organized into four layers under `Assets/Scripts/`:

| Layer | Folder            | Responsibility                                                                                   |
|-------|-------------------|--------------------------------------------------------------------------------------------------|
| Core  | `Scripts/Core/`   | Game lifecycle, scene management (Title/Main), DI-style bootstrapping, shared cross-cutting code |
| Data  | `Scripts/Data/`   | Data access layer: dummy data providers, repositories, and later API/DB integration             |
| Domain| `Scripts/Domain/` | Game rules and core logic: map graph, turns/actions, node ownership, cards, scoring             |
| UI    | `Scripts/UI/`     | MonoBehaviour view/presenter layer: screen controllers, UI binding, user interaction            |

Intended dependency direction:

- **UI → Domain → Data**
- **Core** is infrastructural and shared across layers (e.g., scene loader, service locator / DI bootstrap, game entry point).

Guidelines:

- `Domain` types model the rules of cyberspace: nodes, links, Master Node, players, cards (Virus/Hacker/Defense), turn/action system, and win conditions.
- `Data` handles how those domain models are populated and persisted:
  - In early stages: dummy data and in-memory repositories.
  - Later: database and remote server/API-backed repositories.
- `UI` should be as dumb as possible:
  - Use presenter-style controllers to bind domain data into Unity UI (screens, panels, node views, card views).
  - Avoid embedding game rules directly into MonoBehaviours.
- `Core` is where:
  - Title/Main scene startup logic lives,
  - Services are wired together (e.g., which repository implementation to use),
  - Global systems or singletons are managed (only where truly necessary).

`Data` models and repositories should be plain C# classes (no `MonoBehaviour`) so they remain testable and can be swapped from dummy → API-backed implementations without touching the UI.

## Key Systems (Planned)

- **Map / Node Graph System**:
  - Represents an 11×11 node grid with arbitrary link connectivity (fully connected or partially connected maps).
  - Each node can be neutral, owned by a player, or designated as a Master Node.
  - Links between nodes can hold hidden cards (Virus/Defense) and may be blocked by certain defenses.

- **Turn / Action System**:
  - Each player has a fixed pool of 5 actions per turn.
  - Actions include moving along links, changing node ownership, and playing cards that affect nodes/links/players.
  - After all players finish their actions, the turn advances. Some cards (e.g., logic bombs, zero-day) trigger based on turn counts.

- **Card System**:
  - Three categories: Virus, Hacker, Defense.
  - Cards can:
    - Manipulate node ownership,
    - Steal or drain action points,
    - Reveal opponent cards,
    - Block paths (passwords, biometric checks),
    - Protect or move the Master Node (redundancy, relocation),
    - Temporarily disable opponent defenses (e.g., zero-day for first 3 turns).
  - Virus cards notify opponents when triggered; other categories may remain hidden.

- **Fog-of-war / Information System**:
  - Each player can see:
    - Opponent current position,
    - Node ownership and link layout.
  - Hidden information:
    - Which links/nodes have Virus/Defense cards attached,
    - Certain card effects until activation.
  - The system should support psychological play and bluffing by hiding card placement.

- **Master Node System**:
  - Each player’s Master Node is only visible to themselves.
  - Capturing the opponent’s Master Node converts all their owned nodes into your ownership.
  - Supports “fake” ownership marking (appears owned but underlying Master Node logic is special) to bait or mislead the opponent.
  - Integrates with Defense cards like redundancy and server relocation.

- **Tutorial / Scenario System**:
  - Scripted scenarios introducing:
    - Pure node/link movement,
    - Master Node visibility and risk,
    - Defense cards,
    - Hacker cards,
    - Virus cards.
  - Each tutorial defines:
    - Map layout,
    - Turn limit,
    - Initial card set and node ownership.

- **UI / Screen System**:
  - Title screen: start game, select tutorial or scenario, options.
  - Main screen: map view showing nodes, links, ownership, player positions, and card overlays.
  - Sub-views:
    - Card hand / deck management,
    - Turn/action gauge,
    - Log / notifications (e.g., virus triggers),
    - Minimal story/talk UI for story mode.

## Scene

Two main scenes are planned:

- `Assets/Scenes/Title/TitleScene.unity`  
  - Entry point.  
  - Handles title logo, main menu, scenario/tutorial selection, and transition to MainScene.

- `Assets/Scenes/Main/MainScene.unity`  
  - Core gameplay scene.  
  - Hosts the map UI, turn system, card UI, and in-game HUD.

Scene transitions and shared services (e.g., game bootstrap, repository selection, initial scenario setup) live in the `Core` layer.

## Prefab Conventions

Prefabs live under `Assets/Prefabs/UI/Parts/` (reusable components) and `Assets/Prefabs/UI/Screens/` (full-screen views).

- **Parts**:
  - Node views, link views, card views, turn gauge, HUD elements, etc.
  - Self-contained, with clear public methods for binding Domain data.

- **Screens**:
  - TitleScreen, MainGameScreen, Tutorial overlays, etc.
  - Compose Parts and orchestrate them.
  - Communicate via events or injected references, not direct cross-Screen calls.

When adding new UI or systems, prefer:

- Updating this CLAUDE.md with new concepts,
- Keeping Domain logic free of Unity-specific dependencies,
- Keeping UI scripts focused on binding and visual behavior rather than game rules.