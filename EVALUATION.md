# Initial local evaluation

Automated unit tests run with `python -m unittest discover -s tests -v`.

| Test | Expected result | Current result |
|---|---|---|
| Future video cue exclusion at 18:42 | No video chunk with end time after 18:42 is retrieved | Pass |
| Permitted notes explain a topic not yet reached in the video | Supporting document chunks remain eligible | Pass |
| Unrelated query | No retrieval hits, so answer abstains | Pass |

These are functional boundary tests, not a statistically meaningful benchmark. Before submission, create a labelled question set and measure retrieval recall@k, citation correctness, future-content leakage rate, abstention accuracy, latency and per-query cost. The optional local model should be evaluated separately from extractive fallback mode.
