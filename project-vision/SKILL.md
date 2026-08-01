---
name: project-vision
description: 'Architects high-level conceptual project design, core mechanics, and system boundaries. USE FOR: brainstorming new projects, defining MVPs, mapping user journeys, and scoping feature sets.'
---

# Conceptual Project Design Workflow

## Phase 1: The Core Intent (The "Why")
Before proposing any features, explicitly define:
1. **The Core Problem:** What exact friction or missing capability does this project resolve?
2. **The Target User:** Who is interacting with this system? (e.g., end-users, other developers, automated pipelines).
3. **The Value Proposition:** What makes this solution mechanically or conceptually superior to existing alternatives?

## Phase 2: System Boundaries & Scope
1. **In-Scope (MVP):** Define the absolute minimum set of mechanics required to prove the concept works.
2. **Out-of-Scope (Anti-Goals):** Explicitly list 3-4 features or capabilities that will *not* be built in the initial phase. Defend why they are a distraction.

## Phase 3: Conceptual Flow & User Journey
Map the high-level interaction model:
- Step-by-step conceptual walkthrough of how data, users, or physical actions move through the system from initiation to completion.
- Focus on state changes and conceptual milestones, completely ignoring specific software stacks, APIs, or data structures.

## Rules
- Maintain a rigorous, intent-driven tone. Do not use marketing jargon to describe features.
- If the user's initial prompt is too vague, halt and demand clarification on the Core Problem before generating a massive feature list.
- Treat every feature as a liability. Justify its existence based on the Core Intent.
