# Architecture

## 1. Agent Orchestrator
The `orchestrator` is the central control component.

It manages the `lifecycle` of a note-formatting `task`.

### workflow:
```text
        Receive a note-formatting request
                    |
          Validate the input file
                    |
            Load original Markdown
                    |
           Identify the subject
                    |
         Retrieve global preferences
                    |
      Retrieve topic-specific preferences
                    |
      Retrieve relevant reference examples
                    |
        Construct the formatting context
                    |
            Send the task to L-LLM
                    |
         Collect the generated output
                    |
       Process interruptions - if received
                    |
        Validate the generated Markdown
                    |
              Save the `output`
                    |
           collect user feedback
                    |
    Invoke `learning engine` - if feedback
                    |
   `Orchestrator` maintains - execution state

```

---

### States:

1. CREATED
2. ANALYZING
3. GENERATING
4. PAUSED
5. INTERRUPTED
6. RESUMING
7. VALIDATING
8. COMPLETED
9. CANCELLED
10. FAILED


- Execution `state` must be `explicit` and `persisted` where necessary.
- The `orchestrator` should not rely on the LLM to manage its `own execution lifecycle`.
- `Application code` must remain `responsible` for `execution control`.


---
---


## 2. Tools Architecture

Tools represent `Agent Capabilities`.

#### 2.1 read-notes
##### Responsibilities:
- Read `Markdown documents`.
- Validate `file paths`.
- Extract `document content`.
- Preserve `source metadata`.

> **File Location** : learning-agent\src\learning_agent\tools\read-notes.py

---

#### 2.2 analyze-notes
##### Responsibilities:
- Identify `topics`.
- Identify `headings`.
- Detect `incomplete explanations`.
- Identify `technical terminology`.
- Detect `relationships between concepts`.
- Identify possible `prerequisite concepts`.

> **File Location** : learning-agent\src\learning_agent\tools\analyze-notes.py

---

#### 2.3 format-notes
##### Responsibilities:
- Restructure Markdown.
- Improve explanations.
- Expand concepts.
- Add `relevant examples`.
- Improve `technical clarity`.
- Preserve `original meaning`.

> **File Location** : learning-agent\src\learning_agent\tools\format-notes.py

---

#### 2.4 search-examples
##### Responsibilities:
- Retrieve relevant reference notes.
- Identify examples with similar subject matter.
- Supply relevant style examples to the LLM.

> **File Location** : learning-agent\src\learning_agent\tools\search-examples.py

---

#### 2.5 save-notes
##### Responsibilities:
- Save generated Markdown.
- Preserve original files by default.
- Support versioned output.
- Validate output paths.

> **File Location** : learning-agent\src\learning_agent\tools\save-notes.py

---

**Note:**
> - Tools should not directly modify learning preferences unless explicitly invoked through the learning subsystem.
> - This maintains separation between `document processing` and `persistent memory management`.


---
---


## 3. Autonomous Preference Learning

The `learning engine` should be capable of `updating persistent preferences` without requiring approval for every change.

However, `autonomous learning` must be `evidence-based`, `explainable`, and `reversible`.

---

#### 3.1 Learning Inputs
The learning engine can receive several types of feedback signals.

- `Explicit Feedback` - direct instructions by user
- `Document Edits` - user-modification patterns on the doccument
- `Acceptance & Rejection or Regeneration` - user acceptance to the proposed change 
- `Human Interruptions` - interruption during an explanation, provide contextual information.

---

#### 3.2 Learning Pipeline

Structured process that converts raw feedback into validated persistent memory updates.

```text
              ┌──────────────────────────┐
              │          START           │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │ 1. Collect raw Feedback  │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │   2. Identify Context    │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │   3. Classify Feedback   │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌─────────────────────────────┐
              │ 4. Determine Feedback Type  |
              │     • Temporary Instruction │
              │     • Knowledge Gap         │
              │     • Persistent Preference │
              └────────────┬────────────────┘
                           │
                           ▼
             ┌──────────────────────────────┐
             │ 5. Identify Applicable Scope │
             └────────────┬─────────────────┘
                          │
                          ▼
            ┌─────────────────────────────────┐
            │ 6. Compare Existing Preferences │
            │     • conflicts                 │
            │     • duplication               │
            │     • changes                   │
            └──────────────┬──────────────────┘
                           │
                           ▼
          ┌────────────────────────────────────┐
          │ 7. Generate Proposed Memory Update │
          └────────────────┬───────────────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │ 8. Validate              │
              │     • accuracy           │
              │     • consistency        │
              │     • applicability      │
              └────────────┬─────────────┘
                           │
                           ▼
          ┌────────────────────────────────────┐
          │ 9. Apply Update & validated change │
          └────────────────┬───────────────────┘
                           │
                           ▼
        ┌────────────────────────────────────────┐
        │ 10. Record Evidence & Version History  │
        │       • Store supporting context       │
        │       • update history                 │
        └──────────────────┬─────────────────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │ 11. Evaluate Impact      │
              └────────────┬─────────────┘
                           │
                           ▼
              ┌──────────────────────────┐
              │          END             │
              │ Persistent Memory Updated│
              └──────────────────────────┘
```

The `learning engine` must not blindly append every correction to the `preference profile`.

Instead, it should `refine existing preferences` whenever the new evidence represents a modification of an existing rule.

---

#### 3.3 Metadata Definitions

| Field | Description |
|---|---|
| `id` | Unique identifier for the preference |
| `scope` | Domain or context where the preference applies |
| `instruction` | The actual learned preference |
| `confidence` | Heuristic confidence score |
| `evidence_count` | Number of supporting observations |
| `source` | Origin of the learning evidence |
| `status` | Current lifecycle state |
| `created_at` | Initial creation timestamp |
| `updated_at` | Most recent modification timestamp |

---

#### 3.4 Preference Lifecycle

The system should distinguish between the following preference states.

| Status | Meaning |
|---|---|
| `Tentative` | Newly identified preference with limited evidence |
| `Established`| Preference supported by sufficient recurring evidence |
| `Active` | Preference currently applied during generation |
| `Superseded` | Preference replaced by a newer version |
| `Rejected` | Preference explicitly rejected or invalidated |

> A single feedback event should not automatically create a `ermanent global preference`.

---

#### 3.5 Autonomous Learning Safeguards

| Safeguard | Responsibility |
|---|---|
| `Preference Versioning` | Preserve previous versions of learned preferences |
| `Reverting Changes` | Restore an earlier preference version |
| `Contradiction Resolution` | Identify and resolve conflicting preferences |
| `Evidence Tracking` | Maintain feedback supporting each preference |
| `Duplicate Prevention` | Avoid creating multiple equivalent preference rules |
| `Scope Separation` | Prevent topic-specific preferences from becoming global rules |
| `Temporary Instruction Isolation` | Prevent one-time instructions from becoming permanent preferences |
| `Confidence Thresholds` | Limit the influence of weak observations |
| `User Inspection` | Allow users to review learned preferences |
| `User Override` | Allow users to modify or disable learned preferences |


> ### EdgeCase: Contradictory Preferences
Consider the following situation:
- `Existing preference`: Provide detailed explanations.
- `New feedback`: Keep this explanation short.

here, `learning engine` should not immediately replace the `existing preference`.

- This request concerns a specific document or topic, representing a `temporary instruction`.
- Repeated requests across multiple topics, is a evidence of `preference change`.

> `learning engine` should preserve both and resolve their applicability through scope and context.

---

#### 3.6 Technical workflow 

```text
Feedback Sources
    |
    |-- Explicit Feedback
    |-- Document Edits
    |-- Acceptance and Rejection
    |-- Human Interruptions
    |
    v
Feedback Records
    |
    |-- Raw Observations
    |-- Contextual Metadata
    |
    v
Learning Engine
    |
    |-- Feedback Classification
    |-- Scope Detection
    |-- Evidence Evaluation
    |
    v
Preference Validation
    |
    |-- Conflict Resolution
    |-- Duplicate Detection
    |-- Confidence Checks
    |
    v
Persistent Preference Memory
    |
    |-- Active Preferences
    |-- Version History
    |-- Evidence References
    |
    v
Response Generation
    |
    |-- Preference Retrieval
    |-- Contextual Application
    |-- Adapted Output
```
---
---

## 4. Data Design 

Recommended SQLite Tables

| Table | Purpose |
|---|---|
| `feedback_events` | Store original feedback and interaction records |
| `learning_observations` | Store classified learning signals |
| `preferences` | Store current persistent preferences |
| `preference_versions` | Maintain historical preference versions |

It also prevents the `LLM` from directly `modifying persistent memory` without passing through `validation logic`.

---
---

## 5. QWEN3.5:4B - Input

```text
# What Qwen Receives

```text
        ┌──────────────────────────────────────┐
        │      1. System Instructions          │
        │                                      │
        │        • responsibilities            │
        │        • constraints                 │
        │        • expected output             │
        └──────────────────┬───────────────────┘
                           │
                           ▼
        ┌──────────────────────────────────────┐
        │       2. Learning Preferences        │
        │                                      │
        │           • Explanation depth        | 
        |           • Technical level          │
        │           • Preferred examples       | 
        |           • Terminology              │
        │           • Formatting               │
        └──────────────────┬───────────────────┘
                           │
                           ▼
        ┌──────────────────────────────────────┐
        │    3. Topic-Specific Preferences     │
        └──────────────────┬───────────────────┘
                           │
                           ▼
        ┌──────────────────────────────────────┐
        │          4. Original Markdown        │
        └──────────────────┬───────────────────┘
                           │
                           ▼
     ┌────────────────────────────────────────────┐
     │ 5. Additional Context - previous feedback  │
     └─────────────────────┬──────────────────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │  Message Assembly │
                 │                   │
                 │   final prompt    │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │      Ollama       │
                 │         ↓         │
                 │   Qwen 3.5 4B     │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │       Model       │
                 │  Generates output │
                 └───────────────────┘
```

