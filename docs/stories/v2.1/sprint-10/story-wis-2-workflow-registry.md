# Story WIS-2: Workflow Registry Enhancement

<!-- Source: Epic WIS - Workflow Intelligence System -->
<!-- Context: Enhance workflow patterns with transitions and confidence scoring -->
<!-- Created: 2025-12-23 by @sm (River) -->

## Status: Draft

**Priority:** 🟡 MEDIUM
**Sprint:** 10
**Effort:** 6h
**Lead:** @dev (Dex)
**Blocked By:** WIS-1 (Complete)

---

## Story

**As an** AIOS framework developer,
**I want** all 10 workflows to have complete transition definitions with confidence scoring,
**So that** the `*next` task can suggest accurate next steps for any workflow state.

---

## Background

WIS-1 investigation revealed:
- Only `story_development` workflow has transitions defined
- 9 other workflows lack transition definitions
- WorkflowNavigator prototype exists but lacks confidence scoring
- 75% of infrastructure already exists

This story enhances `workflow-patterns.yaml` with complete transitions and implements the confidence scoring algorithm.

### Prerequisites (from WIS-1)

| Component | Status | Location |
|-----------|--------|----------|
| workflow-patterns.yaml | EXISTS | `.aios-core/data/workflow-patterns.yaml` |
| WorkflowNavigator | EXISTS | `.aios-core/development/scripts/workflow-navigator.js` |
| ContextDetector | EXISTS | `.aios-core/core/session/context-detector.js` |
| ADR-WIS-001 | Complete | `docs/architecture/adr/adr-wis-architecture.md` |

---

## 🤖 CodeRabbit Integration

### Story Type Analysis

**Primary Type**: Implementation
**Secondary Type(s)**: Data Enhancement, Algorithm Implementation
**Complexity**: Low-Medium

### Specialized Agent Assignment

**Primary Agents**:
- @dev (Dex): Implement confidence scorer and enhance workflow patterns

**Supporting Agents**:
- @architect (Aria): Verify implementation matches ADR-WIS-001
- @qa (Quinn): Validate transitions coverage and scoring accuracy

### Quality Gate Tasks

- [ ] Pre-Commit (@dev): Verify confidence scoring implementation
  - **Pass criteria:** Algorithm matches ADR spec, all tests pass, scoring factors documented
  - **Fail criteria:** Missing scoring factors, incorrect weights, no tests
- [ ] Pre-PR (@architect): Confirm ADR compliance
  - **Pass criteria:** Directory structure matches, interfaces aligned, pattern followed
  - **Fail criteria:** Deviation from ADR without documented reason

### Self-Healing Configuration

**Mode:** light (Primary Agent: @dev)
**Max Iterations:** 2
**Time Limit:** 15 minutes
**Severity Threshold:** CRITICAL only

| Severity | Auto-Fix | Behavior |
|----------|----------|----------|
| CRITICAL | Yes | Block merge, auto-fix if possible |
| HIGH | No | Report only |
| MEDIUM | No | Report only |
| LOW | No | Ignore |

### Focus Areas

- Confidence scoring algorithm accuracy
- Workflow transition completeness
- YAML schema validation
- Cache implementation correctness

---

## Acceptance Criteria

### AC 2.1: Enhanced Workflow Transitions

- [ ] All 10 workflows have transition definitions
- [ ] Each transition includes:
  - `trigger`: Command or condition that triggers the transition
  - `confidence`: Base confidence score (0.0-1.0)
  - `next_steps`: Array of suggested commands with descriptions
- [ ] Transitions cover primary happy path for each workflow
- [ ] At least 2-3 states per workflow

**Workflows to Enhance:**

| Workflow | States to Add |
|----------|---------------|
| epic_creation | epic_drafted, stories_created, validated |
| backlog_management | reviewed, prioritized, scheduled |
| architecture_review | analyzed, documented, approved |
| git_workflow | staged, committed, pushed |
| database_workflow | designed, migrated, validated |
| code_quality_workflow | assessed, refactored, tested |
| documentation_workflow | drafted, reviewed, published |
| ux_workflow | designed, implemented, validated |
| research_workflow | researched, analyzed, documented |

### AC 2.2: Confidence Scoring Implementation

- [ ] Create `confidence-scorer.js` in `.aios-core/workflow-intelligence/engine/`
- [ ] Implement weighted scoring algorithm:
  - Command match: 40%
  - Agent match: 25%
  - History depth: 20%
  - Project state: 15%
- [ ] Expose `score(suggestion, context)` method
- [ ] Return normalized score (0.0-1.0)

**Algorithm:**
```javascript
score(suggestion, context) {
  const commandMatch = this.matchCommand(suggestion.trigger, context.lastCommand);
  const agentMatch = this.matchAgent(suggestion.agentSequence, context.agentId);
  const historyDepth = this.matchHistory(suggestion.keyCommands, context.lastCommands);
  const projectState = this.matchProjectState(suggestion, context.projectState);

  return (
    commandMatch * 0.40 +
    agentMatch * 0.25 +
    historyDepth * 0.20 +
    projectState * 0.15
  );
}
```

### AC 2.3: Workflow Registry Module

- [ ] Create `.aios-core/workflow-intelligence/` directory structure
- [ ] Implement `workflow-registry.js` with:
  - `loadWorkflows()`: Load and cache patterns (5-minute TTL)
  - `matchWorkflow(commands)`: Find workflow matching command history
  - `getTransitions(workflowName, state)`: Get available transitions
- [ ] Ensure backward compatibility with existing WorkflowNavigator

### AC 2.4: Testing

- [ ] Unit tests for ConfidenceScorer
- [ ] Unit tests for WorkflowRegistry
- [ ] Integration test: context → workflow match → scored suggestions
- [ ] All existing tests pass

---

## Technical Design

### Directory Structure (from ADR-WIS-001)

```
.aios-core/
├── workflow-intelligence/           # NEW
│   ├── registry/
│   │   └── workflow-registry.js     # Enhanced registry loader
│   ├── engine/
│   │   └── confidence-scorer.js     # NEW - Confidence scoring
│   └── index.js                     # WIS public API
│
├── data/
│   └── workflow-patterns.yaml       # ENHANCED with transitions
```

### Enhanced Schema Example

```yaml
# workflow-patterns.yaml
workflows:
  epic_creation:
    description: "Epic lifecycle from concept to story breakdown"
    agent_sequence: [po, sm, architect]
    key_commands:
      - create-epic
      - create-story
    trigger_threshold: 1

    transitions:
      epic_drafted:
        trigger: "create-epic completed"
        confidence: 0.90
        next_steps:
          - command: analyze-epic
            args_template: "${epic_path}"
            description: "Analyze epic for completeness"
            priority: 1
          - command: create-story
            args_template: "${epic_path}"
            description: "Create first story from epic"
            priority: 2

      stories_created:
        trigger: "create-story completed"
        confidence: 0.85
        next_steps:
          - command: validate-story-draft
            args_template: "${story_path}"
            description: "Validate story structure"
            priority: 1
```

---

## Dependencies

### Blocked By
- **WIS-1:** Investigation & Design (Complete ✅)

### Blocks
- **WIS-3:** `*next` Task Universal (needs registry and scoring)
- **WIS-4:** Wave Analysis Engine (needs registry)

---

## Success Criteria

1. All 10 workflows have at least 2-3 transition states
2. Confidence scorer returns scores matching test cases
3. WorkflowRegistry loads patterns with caching
4. Existing greeting-builder functionality unaffected
5. All tests pass

---

## File List

| File | Status | Description |
|------|--------|-------------|
| `docs/stories/v2.1/sprint-10/story-wis-2-workflow-registry.md` | Draft | This story |
| `.aios-core/data/workflow-patterns.yaml` | To Modify | Add transitions to 9 workflows |
| `.aios-core/workflow-intelligence/registry/workflow-registry.js` | To Create | Registry loader |
| `.aios-core/workflow-intelligence/engine/confidence-scorer.js` | To Create | Scoring algorithm |
| `.aios-core/workflow-intelligence/index.js` | To Create | Public API |

---

## Change Log

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-23 | @sm (River) | Initial draft from MVP scope |
