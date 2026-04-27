# eval-compare results — {target_skill_name}

**Query**: {query}
**Run at**: {timestamp}
**Total wall-clock**: {total_time}

## Comparison grid

|  | Time | Quality | Notes |
|---|---|---|---|
| **A.** Opus, no skill (with year cue) | {time_a}s | {q_a}/3 | {notes_a} |
| **B.** Opus, no skill | {time_b}s | {q_b}/3 | {notes_b} |
| **C.** Opus + /{skill} | {time_c}s | {q_c}/3 | {notes_c} |
| **D.** Sonnet + /{skill} | {time_d}s | {q_d}/3 | {notes_d} |

## Rubric

1. {criterion_1}
2. {criterion_2}
3. {criterion_3}

## Per-condition breakdown

### Condition A — Opus, training-only, with year cue
- **Time**: {time_a}s
- **Quality**: {q_a}/3
  - Criterion 1 ({criterion_1[:40]}): {hit_a_1}
  - Criterion 2 ({criterion_2[:40]}): {hit_a_2}
  - Criterion 3 ({criterion_3[:40]}): {hit_a_3}
- **Contamination check**: {contamination_a}
- **Output preview**:
  ```
  {output_a_truncated}
  ```

### Condition B — Opus, training-only, no year cue
- **Time**: {time_b}s
- **Quality**: {q_b}/3
  - Criterion 1: {hit_b_1}
  - Criterion 2: {hit_b_2}
  - Criterion 3: {hit_b_3}
- **Contamination check**: {contamination_b}
- **Output preview**:
  ```
  {output_b_truncated}
  ```

### Condition C — Opus + /{skill}
- **Time**: {time_c}s
- **Quality**: {q_c}/3
  - Criterion 1: {hit_c_1}
  - Criterion 2: {hit_c_2}
  - Criterion 3: {hit_c_3}
- **Output preview**:
  ```
  {output_c_truncated}
  ```

### Condition D — Sonnet + /{skill}
- **Time**: {time_d}s
- **Quality**: {q_d}/3
  - Criterion 1: {hit_d_1}
  - Criterion 2: {hit_d_2}
  - Criterion 3: {hit_d_3}
- **Output preview**:
  ```
  {output_d_truncated}
  ```

## Headline finding

{1-3 sentences: did the skill help? does small+skill match big+skill? any contamination? any surprises?}

## Decision

- **Did the skill earn its place?** {yes/no/conditional, with reasoning}
- **Did small+skill match big+skill?** {yes/no, with the specific gap}
- **What to do next?** {ship / iterate / kill / re-test with different rubric}
