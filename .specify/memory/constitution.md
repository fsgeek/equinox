# Project Equinox Constitution

## Core Principles

### I. Specification-First Development (NON-NEGOTIABLE)

**Specifications before implementation**: Every feature begins with a complete, unambiguous specification in `.specify/specs/`. Code is written only after the spec passes quality validation through `/speckit.clarify`.

**Technology-agnostic specifications**: Specs describe WHAT users need and WHY, never HOW to implement. No mention of frameworks, databases, languages, or cloud providers in spec.md files.

**Iterative refinement**: Specifications are refined through `/speckit.clarify` until all ambiguities are resolved (typically 4-6 iterations based on PromptGuard experience).

**Rationale**: In Specification-Driven Development, the specification is the source of truth. AI tools excel at implementation from clear specifications but struggle with ambiguous requirements. This principle ensures specifications drive development rather than emerging from code.

### II. AI-Assisted Implementation

**Document AI usage**: Every AI interaction for code generation, research, or design must be documented with transparency. Tool used, prompts provided, and human validation performed.

**Human validation required**: AI-generated code, specifications, or designs must be reviewed and understood by humans before acceptance. "I can explain this code" is mandatory.

**Incremental AI utilization**: Start with AI for research and specification refinement (`/speckit.specify`, `/speckit.clarify`), then progress to planning (`/speckit.plan`), task generation (`/speckit.tasks`), and implementation (`/speckit.implement`).

**Rationale**: This is a demonstration project for CPSC 436C showing how AI tools accelerate development from specifications. Transparency and human oversight are critical for learning outcomes and academic integrity.

### III. Quality Gates for Specifications

**Mandatory specification validation**: All specs must pass quality checklist before proceeding to `/speckit.plan`:
- No [NEEDS CLARIFICATION] markers remaining
- All functional requirements testable and unambiguous
- Success criteria measurable and technology-agnostic
- Edge cases identified with handling approaches

**Prioritized user stories**: Every feature uses P1/P2/P3 prioritization with independent testability. Each story must be deliverable as a standalone MVP.

**Dependencies documented**: Technical dependencies, implementation phases, and key assumptions explicitly captured in specs with justification for phased approaches.

**Rationale**: The spec-kit workflow demonstrates that high-quality specifications reduce downstream rework. Quality gates force specification completeness before expensive implementation work begins.

### IV. API-First Design

**Contracts before code**: OpenAPI 3.0 or GraphQL schemas generated during `/speckit.plan` phase before any implementation begins.

**Consumer-driven contracts**: API designs based on user scenarios from specifications, not implementation convenience.

**Versioning from day one**: All APIs versioned (e.g., `/api/v1/`) to demonstrate evolution and backward compatibility considerations.

**Rationale**: API-first design enables parallel development (frontend/backend teams) and demonstrates how specifications translate to concrete interfaces. OpenAPI contracts serve as executable documentation.

### V. Architecture Documentation

**Research-backed decisions**: Every architectural choice justified in `research.md` with:
- Decision made
- Rationale with specific requirements references
- Alternatives considered
- Trade-offs explicitly stated

**Technology selection transparency**: When specs require technology choices, document selection criteria, comparison matrix, and decision rationale.

**Governance awareness**: Acknowledge that technical correctness ≠ ethical systems. Flag decisions where governance/oversight questions remain even after technical implementation.

**Rationale**: This demonstrates the "Think, Design, Build, Critique" philosophy. Architectural decisions should be traceable to requirements and alternative approaches documented for learning.

### VI. Modular Feature Development

**Independent feature specs**: Each epic/feature gets its own spec directory (e.g., `001-mcep-engagement/`, `002-pwp-portal/`) enabling parallel development and independent evaluation.

**Feature branches**: One branch per feature specification following `###-feature-name` naming convention (e.g., `001-mcep-engagement`).

**Self-contained artifacts**: Each feature directory contains complete planning artifacts: `spec.md`, `research.md`, `data-model.md`, `contracts/`, `tasks.md`.

**Rationale**: Modular development demonstrates how complex systems decompose into manageable features. Each feature can be demonstrated independently, showcasing incremental delivery.

### VII. Demonstrate Workflow Tools

**Use all spec-kit commands**: Project should demonstrate the complete workflow:
1. `/speckit.specify` - Feature specification generation
2. `/speckit.clarify` - Iterative ambiguity resolution (4-6 rounds)
3. `/speckit.plan` - Technical architecture and research
4. `/speckit.tasks` - Task decomposition with dependencies
5. `/speckit.analyze` - Cross-artifact consistency validation
6. `/speckit.checklist` - Quality validation checklists
7. `/speckit.implement` (optional) - Selected component implementation

**Iterative refinement visible**: Git history should show specification evolution through clarification rounds, not perfect-first-time specs.

**Command outputs preserved**: All generated artifacts (research.md, data-model.md, contracts/, tasks.md) committed to repository demonstrating tool capabilities.

**Rationale**: This is a demonstration project for learning SDD and spec-kit tooling. The value is in showing the workflow, not building production software. Each command should be demonstrated at least once.

## Development Standards

### Specification Quality Standards

**User-centric language**: Specifications written for business stakeholders, not developers. Use "Portfolio Manager needs to..." not "System shall implement...".

**Testable requirements**: Every functional requirement must be independently verifiable without implementation knowledge.

**Measurable success criteria**: Use specific metrics (time, percentage, count) not vague adjectives ("fast", "robust", "scalable").

**Edge cases explicit**: Boundary conditions and error scenarios documented with expected system behavior, not just flagged as questions.

### Planning Standards

**Constitution alignment**: Every technical plan must reference constitution principles and demonstrate alignment. Violations must be explicitly justified in "Complexity Tracking" section.

**Technology choices justified**: No technology selected without documented rationale in research.md. "We chose X because..." with alternatives considered.

**Scale considerations**: Performance and scale requirements from specifications translated to concrete architecture decisions (database choice, caching strategy, etc.).

**Phase planning**: Implementation phases aligned with specification priorities (P1 features before P2/P3).

### Documentation Standards

**README completeness**: Every feature should have quickstart.md enabling another developer (or AI agent) to understand the feature and begin work.

**API documentation**: OpenAPI specs must include descriptions, examples, and error responses. GraphQL schemas must include field-level documentation.

**Architecture diagrams**: research.md should include ASCII or Mermaid diagrams showing system components and interactions.

**Governance notes**: When technical designs have governance implications (e.g., compliance approach), explicitly acknowledge and document in research.md.

## Governance

### Constitution Amendments

**Amendment procedure**: Constitution changes require:
1. Documented rationale for change
2. Impact assessment on existing specifications
3. Version bump (MAJOR/MINOR/PATCH) following semantic versioning
4. Update propagation to affected templates and specs

**Version semantics**:
- **MAJOR**: Backward-incompatible principle changes requiring spec rework
- **MINOR**: New principles added or materially expanded guidance
- **PATCH**: Clarifications, typo fixes, non-semantic refinements

### Compliance Verification

**Specification review**: Every spec.md must pass quality checklist verification before planning phase begins.

**Planning review**: Every plan.md must demonstrate constitution alignment in "Constitution Check" section.

**Cross-artifact consistency**: `/speckit.analyze` must run after task generation to verify consistency between spec, plan, and tasks.

### Demonstration Focus

**Educational value prioritized**: When trade-offs exist between "production ready" and "demonstrates SDD workflow", choose the latter.

**Learning objectives**: Project demonstrates:
- How specifications drive AI-assisted implementation
- How ambiguity reduction through `/speckit.clarify` prevents rework
- How modular specifications enable parallel development
- The governance gap (technical correctness ≠ ethical systems)

**Academic integrity**: All AI usage documented. All generated artifacts preserved in git history. Human understanding and validation mandatory for submitted work.

## Project Context

**Course**: CPSC 436C - Cloud Systems for Data Science (UBC)
**Institution**: University of British Columbia
**Academic Year**: 2025W-T1
**Purpose**: Demonstrate Specification-Driven Development using Claude Code and GitHub Spec-Kit

**Key Insight**: Good tools accelerate whatever you specify. The specification determines the outcome - not the tool. This project demonstrates both the power and limitations of AI-assisted development from specifications.

---

**Version**: 1.0.0 | **Ratified**: 2025-10-27 | **Last Amended**: 2025-10-27
