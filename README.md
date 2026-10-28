# Sub-Vocal Ghost

**Silent coding instructions through sEMG and lip-motion sensing, built with Meta's Muse models.**

A planned research prototype for the Meta Global AI Developer Hackathon, Indie track / Build for Your World.

> **Status: planning, not a working product.** No hardware validation, collected dataset, trained adapter, or measured recognition results are claimed. This repository currently contains the project plan only.

## The problem

Speaking to a coding assistant is convenient, but speaking aloud on a train or in a shared public space can disturb others. We want to investigate a silent alternative: articulating instructions with the mouth and facial muscles without producing audible speech. This is motion-based sensing, not mind reading.

## The central experiment

**Does adding lip-motion video improve recognition over the same sEMG input alone?**

Record synchronized surface electromyography (sEMG) and mouth-region video for the same silently articulated instructions. Compare:

| Condition | Input and processing | Purpose |
| --- | --- | --- |
| A0 | sEMG decoder only | Sensor baseline |
| A1 | sEMG candidate transcripts + Muse Glimmer, no video | Language-model-assisted baseline |
| B | The same sEMG candidates + ordered lip frames + Muse Glimmer | Measure the incremental contribution of lip motion |
| Optional diagnostic | Lip-motion video only | Check whether video dominates the combined model |

A1 versus B is the primary comparison. Train for each input condition on matched data splits; also test input removal and shuffled/mismatched video. Keep code context identical or excluded during the recognition comparison. Evaluate unseen recordings, including separate recording sessions and sensor reattachment, rather than adjacent windows from the same utterance.

## Proposed architecture

```text
sEMG -> dedicated temporal decoder -> candidate transcripts ---+
                                                              |
video -> synchronized mouth crops / ordered frames ------------+
                                                              v
                                          fine-tuned Muse Glimmer
                                              -> transcript or abstain
                                              -> user confirmation
                                              -> Muse Code / code diff / tests
```

- **Muse Glimmer:** investigate multimodal supervised fine-tuning / QLoRA to select among sensor-derived transcript candidates using lip frames. Its documented text/image training support does not establish pretrained lipreading or raw-sEMG understanding.
- **Muse Spark Open:** use hosted inference as a comparison for candidate reranking and code-context interpretation. Compare latency, cost, and errors against the Glimmer route.
- **Muse Code:** turn confirmed instructions into proposed edits and tests, with a reviewable diff before applying changes.
- **Optional SAM 3.1:** investigate mouth-region tracking only if it improves cropping over a simpler method; segmentation is not speech decoding.
- **Muse Voice Transcribe:** optional audible/whispered reference condition, not the decoder for silent EMG.

Candidate selection cannot recover a correct instruction absent from every candidate. Measure candidate recall before attributing failures to Glimmer. If this ceiling is limiting, investigate a longer-term feature-fusion model with dedicated EMG and visual temporal encoders; that requires additional engineering and training beyond the stock Glimmer recipe.

## Planned compute use

- **Modal:** train the sensor decoder, experiment with Glimmer QLoRA, and run paired evaluation jobs on on-demand GPUs.
- **Baseten:** prototype hosted Muse Spark Open inference for candidate reranking and coding-context interpretation, and measure request latency and token cost.
- **Fireworks:** independently benchmark hosted Muse Spark Open inference for the same tasks; consider supported adapter training only after verifying model eligibility and budget.

These are planned uses, not claims that credits, deployments, or provider-specific training access have been granted. Start with small, bounded jobs and persist results before shutting down compute.

## Milestones and evaluation

1. Validate signal quality, raw-signal access, synchronization, and a small silent-command dataset.
2. Establish sEMG-only baselines and the paired lip-video comparison.
3. Train and evaluate Glimmer candidate selection with and without video.
4. Connect confirmed instructions to a coding workflow and demonstrate the measured comparison.
5. Expand from a bounded command set toward Korean free-form instructions only if held-out results support it.

Report command/intent accuracy, Korean character error rate where applicable, critical negation/number/target errors, abstention rate, candidate recall, latency, and compute cost. Separate limited-vocabulary command classification from free-form transcription. Noisy surroundings, motion, lighting changes, and reattachment are robustness tests, not conditions assumed to be solved.

For silent conditions, remove the audio track and exclude visible prompt text from model inputs. Uncertain input must not silently trigger a code change. Do not connect body-worn electrodes directly to a computer audio jack; use appropriate isolated acquisition hardware.

## Data and scope

No personal recordings, facial video, sEMG traces, account identifiers, credentials, or participant contact details belong in this public repository. Initial experiments should use appropriately permitted research data or explicitly consented recordings. Public release of any dataset requires a separate decision.

The hackathon goal is an honest, measurable prototype—not a claim of reliable silent dictation, medical efficacy, or error-free use in public transit.

## References

- [Meta hackathon](https://dev.meta.ai/events/global-hackathon)
- [Muse Glimmer post-training](https://dev.meta.ai/docs/muse-glimmer/post-training)
- [Muse Glimmer model card](https://huggingface.co/meta-models/Muse-Glimmer-30B)
- [Muse Spark Open](https://dev.meta.ai/docs/muse-spark-open)
- [Muse Code integration](https://dev.meta.ai/docs/muse-code)
- [Cross-Modal Masking for Robust Silent Speech Synthesis Using sEMG and Lipreading](https://arxiv.org/abs/2606.09667) — related research, not results from this project.
- [Open EMG silent-speech research code](https://github.com/dgaddy/silent_speech)
