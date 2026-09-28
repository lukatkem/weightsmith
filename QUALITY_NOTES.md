# Quality measurement notes

How weightsmith decides what "good enough" means when a model's weights shrink.

## The three metrics

- **max_abs_error** — worst-case single-weight drift. Tight bounds matter for
  attention logits, where hair-width differences flip argmax picks.
- **snr_db** — global signal-to-noise of the dequantized tensor vs the original.
  Per-row int8 typically gains 9+ dB over per-tensor on outlier-heavy data.
- **argmax_agreement** — the metric that actually matters for generation: the
  fraction of greedy decode steps picking the same next token. A 52%-agreement
  model is unusable even when its SNR looks respectable; 100% is indistinguishable.

## The lesson from the field

Per-tensor int8 on a 66-symbol vocabulary measured **52% greedy agreement** —
the model was ruined while looking fine on aggregate error. Per-row scales
absorb outlier channels and restored **100% agreement** at half the bytes.
Measure what your users experience (the decoded tokens), not what looks small
in a table (the raw error).
