# RAG Eval — LLM-graded transcript QA (2026-09-25)

Re-grading of the answers captured in the [2026-08-29 run](2026-08-29.md)
(router config `0ebf967e5721`). **No new queries were sent.** The live answers
cached by `run_eval.py` were graded against the ground-truth `expected_answer`.

- **Scope:** the 19 questions from ids 61–90 whose source video is indexed
  (Video 1: 7, Video 2: 12). The 11 Video 0 questions (ids 61–71) were
  excluded because that transcript is not in the index. They measure an
  ingestion gap, not the assistant, and have been removed from
  `eval_questions.json`.
- **Grader:** LLM-as-judge (Claude Opus 5.5), reading the full answer against
  the expected answer.
  - **Correct:** every key fact present, nothing contradicting the source.
  - **Partial:** the core answer is right, but a key fact is missing or wrong.
  - **Incorrect:** the core answer is missing or wrong.
- **Caveats:** single grader, single run, n = 19. Treat as an indicative
  measurement, not a benchmark.

## Summary

| Verdict | Count | Share |
|---|---:|---:|
| Correct | 15 | 79% |
| Partial | 2 | 11% |
| Incorrect | 2 | 11% |
| Answers citing a source video + timestamp | 19 | 100% |

For comparison, the keyword-overlap heuristic on the same 19 answers gave
11 PASS / 8 REVIEW / 0 FAIL. It cannot judge the REVIEW rows at all, and it
passed neither incorrect answer (73 and 87 were both REVIEW), so it gives no
failure signal on its own.

## Per question

| id | topic | verdict | note |
|---|---|---|---|
| 72 | Crashed EV storage | Correct | Store outside, 50 ft clearance. |
| 73 | Disconnect tool cost | Incorrect | Said the price was not in the transcripts; expected ~$1,000. Retrieval miss, but it declined rather than inventing a figure. |
| 74 | EV submersion | Correct | NHTSA: no shock hazard in surrounding water. |
| 75 | Dual 12V batteries | Correct | Under hood + under rear seat, and why it matters. |
| 76 | Charging cable safety | Correct | Do not cut; >400 V; overcome the plastic lock. |
| 77 | Disconnect tool lights | Partial | Green, blue, yellow and flashing-red are right; attributes "vehicle not recognized" mainly to continued green rather than solid red. |
| 78 | Leaking fluid ID | Correct | Coolant, not electrolyte. |
| 79 | Battery fire tactic | Correct | Let it burn; 10–15 h re-ignition cycle. |
| 80 | Fire water volume | Correct | 10,000 gallons may not be enough. |
| 81 | Battery chemistry safety | Correct | LFP safer than NMC. |
| 82 | Fire blanket | Correct | Opinion + $3,200. |
| 83 | Laminate side glass | Correct | Laminated side glass → saw-cutting. |
| 84 | Carbon fiber dust | Correct | Respiratory protection. |
| 85 | Battery disposal crating | Correct | Liner → sand → water → crate. |
| 86 | Airbags vs HV disconnect | Correct | Airbags stay live; disconnect 12 V. |
| 87 | When not to cut cables | Incorrect | Misses the expected reason (post-crash coolant circulation keeps the pack cool) and substitutes a contactor-state rationale not in the expected answer. |
| 88 | Battery-puncture tool | Correct | Bad idea; may cause a fire that wouldn't have happened. |
| 89 | Charging levels | Partial | Weekend / overnight / quick and the ~80% taper are right; the taper explanation differs from the instructor's, and CHAdeMO is called Chinese (it is Japanese). |
| 90 | Cold-weather charging | Correct | ≥50 °F, energy spent heating the pack, precondition. |
