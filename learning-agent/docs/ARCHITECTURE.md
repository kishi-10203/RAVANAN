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

##### A. Explicit Feedback

Explicit feedback - `direct instructions` by user.


Potential learning signals:

| User Feedback | Potential Learning Signal |
|---|---|
| Explain this in more detail. | Preference for detailed explanations |
| This explanation is too shallow. | Preference for greater technical depth |
| Don't simplify technical terminology. | Preference for preserving technical terminology |
| Include implementation details. | Preference for practical implementation guidance |
| Explain the underlying mechanism. | Preference for conceptual and internal explanations |

> `Explicit feedback` is generally a strong `learning signal` because the user directly communicates what should change.

learning engine must distinguish between:
- `Temporary instructions` applying only to the `current response`.
- `Topic-specific` preferences.
- `Recurring` preferences.
- `Global` preferences.

A single instruction should not automatically become a permanent global preference.

---

##### B. Document Edits

The `learning engine` should compare generated documents against `user-modified` versions to identify `recurring editing patterns`.

Possible signals:

| User Modification | Potential Learning Signal |
|---|---|
| Repeatedly `expanded explanations` | Preference for additional detail |
| Removed `unnecessary content` | Preference for concise and relevant information |
| Added missing `implementation details` | Preference for practical technical coverage |
| `Reorganized headings` | Possible structural preference |
| Replaced `tables` with prose | Possible preference for narrative explanations |
| Added `formatting` | Preference for catchy visuals on important topics | 

Document edits must be `interpreted cautiously` because users may modify content for reasons unrelated to their general preferences.

> Document modifications should initially be treated as `observations` rather than `confirmed preferences`.

`Repeated patterns` across `multiple documents` can gradually `increase confidence`.

---

##### C. Acceptance and Rejection

The system can observe whether users `accept`, `reject`, or `request regeneration` of generated explanations.


Potential learning signals:

| User Action | Potential Learning Signal |
|---|---|
| `Accepts` generated explanation | Weak positive signal |
| `Rejects` generated explanation | Negative signal requiring contextual analysis |
| `Repeatedly requests` regeneration | Possible mismatch with expectations |
| Requests `additional detail` | Preference for greater depth |
| Accepts `revised explanation` | Evidence that the revision addressed the issue |

Acceptance alone is weak evidence of a specific preference.

> `explanation` does not necessarily indicate a preference for short explanations, technical explanations, or a particular writing style.

The `learning engine` should `combine acceptance patterns` with more `explicit evidence` before `modifying persistent preferences`.

---