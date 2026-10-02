# medical-llm-urgency

1. Problem
Medical LLMs are mostly evaluated on whether an answer is clinically correct. A correct answer can still fail a patient: a reply may name the right condition and mention seeking care, yet leave the reader planning a routine appointment when the case needs urgent attention.

The scoring rule commonly used takes the strongest urgency phrase anywhere in a reply, so a reply containing "emergency" passes whatever the message conveys as a whole. It is not known whether reply urgency systematically diverges from case urgency, in which direction, or whether that divergence is visible in the model's internal state before it writes.

2. Setup
|               |                                                                                                                                  |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Benchmark     | CARE-Bench: patient-clinician conversations re-staged into rounds of controlled disclosure, one clinician action label per round |
| Rows used     | 676 from 424 conversations: **172 urgent**, **504 non-urgent**                                                                   |
| Excluded      | 249 rows labelled "insufficient information", where the correct action is a clarifying question and no urgency level is defined  |
| Model         | Qwen2.5-7B-Instruct, frozen, greedy decoding                                                                                     |
| System prompt | The dataset's own, used unmodified   

Gold labels map to expected urgency levels as follows.
| CARE-Bench label        | expected level                        |
| ----------------------- | ------------------------------------- |
| `1B` urgent care        | U1 emergency now, or U2 same-day care |
| `1A` non-urgent care    | U3 non-urgent                         |
| `0B` self-care, monitor | U4 self-care                          |
| `0A` information needed | excluded                              |

3. Measuring what a reply recommends
Each generated reply is shown to the model alone, with no patient messages and no conversation history, and it reports the level of care that reply advises. Levels are adapted from Gilbert et al. (2020).
| level | meaning              |
| ----- | -------------------- |
| 1     | emergency care now   |
| 2     | same-day care        |
| 3     | see a clinician soon |
| 4     | no particular hurry  |
| 0     | no care advice given |

It returns the level, the phrases it used, and one sentence of reasoning.

