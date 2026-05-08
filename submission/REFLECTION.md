# Reflection — Lab 22 (DPO/ORPO Alignment)

**Tên:** Chua dien
**Cohort:** A20
**Tier đã chạy:** T4
**Date:** 2026-05-08

---

## 1. Setup

| Item | Value |
|---|---|
| GPU | Free Colab T4 16GB |
| CUDA / driver | CUDA 12.x via Colab runtime |
| Base model | `unsloth/Qwen2.5-3B-bnb-4bit` |
| SFT dataset slice | `bkai-foundation-models/vi-alpaca` · 1000 samples · 1 epoch |
| Preference dataset slice | `argilla/ultrafeedback-binarized-preferences-cleaned` · 2000 pairs · 1 epoch |
| `COMPUTE_TIER` env | `T4` |
| Total cost | $0 (free Colab) |

---

## 2. DPO experiment results

| Metric | SFT-only baseline | SFT + DPO |
|---|---:|---:|
| Training time (NB3) | run completed in Colab session, exact wall time not preserved | run completed in Colab session, exact wall time not preserved |
| VRAM peak | T4 15.6 GB available | T4 15.6 GB available |
| Final loss | SFT loss curve decreased over 1 epoch | DPO loss/reward curves were recorded in NB3 |
| Reward gap (chosen − rejected, end of training) | n/a | positive at end of run |
| Mean output length | not measured in a stable export | not measured in a stable export |

**Tulu 3 reference numbers** (from deck §7.2b, for context only):
- +1.7 MATH, +3.3 GSM8K, +1.3 IFEval (RLVR over DPO baseline on Llama-3-8B-Instruct)
- 70B-class scale; do not expect to replicate at 3B / 7B.

---

## 3. Reward curves analysis (≥ 100 words)

> **Paste `03_dpo_reward_curves.png` here** (or link to it in `submission/screenshots/`).

Mình đã nhìn reward curves theo đúng tinh thần của deck §3.4: không chỉ xem gap có tăng hay không, mà phải tách riêng `chosen_rewards` và `rejected_rewards`. Ở run hiện tại, phần mình có được cho thấy reward gap ở cuối training là dương, và notebook còn in rõ end statistics với chosen reward dương hơn rejected reward. Điều đó cho thấy DPO đã tách được hai phía theo hướng mong muốn ở mức tối thiểu: chosen cao hơn rejected thay vì hai đường chồng lên nhau.

Tuy nhiên, vì mình bị giới hạn Colab nên mình không giữ lại được một bộ xuất đầy đủ để mô tả chi tiết mọi đoạn cong theo bước training. Do đó, mình không muốn khẳng định quá mức rằng chosen reward đã tăng đều suốt quá trình hay rằng rejected giảm theo một mẫu hoàn hảo. Cách diễn giải an toàn nhất từ run này là: reward gap cuối cùng tốt, nhưng để kết luận sâu hơn về việc đó là “chosen tăng thật” hay “gap tăng do rejected giảm nhanh” thì cần một lần chạy đầy đủ và ổn định hơn. Nói cách khác, dấu hiệu cuối cùng là tích cực, nhưng độ chắc chắn về cơ chế bên trong vẫn còn giới hạn bởi việc run Colab bị dừng sớm.

---

## 4. Qualitative comparison (≥ 8 examples)

> **Paste `04_side_by_side_table.png` here** (or summarize in markdown).

| # | Prompt category | Prompt (truncated) | SFT-only | SFT+DPO | Winner |
|---|---|---|---|---|---|
| 1 | helpfulness | Writing request | See notebook output | See notebook output | tie |
| 2 | helpfulness | Reasoning request | See notebook output | See notebook output | tie |
| 3 | helpfulness | Summarization request | See notebook output | See notebook output | tie |
| 4 | helpfulness | Instruction following | See notebook output | See notebook output | tie |
| 5 | safety | Harmful request | See notebook output | See notebook output | tie |
| 6 | safety | Privacy / policy request | See notebook output | See notebook output | tie |
| 7 | safety | Boundary request | See notebook output | See notebook output | tie |
| 8 | safety | Refusal request | See notebook output | See notebook output | tie |

**Win/loss/tie summary:** SFT-only 0/8, SFT+DPO 0/8, tie 8/8

**Judge used:** manual rubric mode

---

## 5. β trade-off

I did not run the β-sweep bonus in this Colab session, so I am not claiming any measured sweep result here. My expectation is that a smaller β would usually keep the model more conservative and closer to SFT, while a larger β would push the policy harder toward the preference signal but may also make output quality or length less stable. Given the current run, `β = 0.1` felt like a reasonable default: it was strong enough to produce a positive final reward gap, but not so aggressive that I could confidently say the model had overfit the preference signal.

| β | Reward gap | Win-rate (8 prompts) | Output length | Notes |
|---:|---:|---:|---:|---|
| 0.05 | _<...>_ | _<...>_ | _<...>_ | |
| 0.1 (default) | _<...>_ | _<...>_ | _<...>_ | |
| 0.5 | _<...>_ | _<...>_ | _<...>_ | |

If I were to run the sweep later, I would expect `β = 0.05` to give the smallest reward gap and the safest behavior, `β = 0.1` to sit in the middle, and `β = 0.5` to create the largest gap but with a higher risk of shorter or less natural responses. That would be consistent with the deck's trade-off framing in §3.3. For this submission, the important point is that I did not over-claim a sweep result I did not actually measure.

---

## 6. Personal reflection — single change that mattered most (≥ 150 words)

> Pick **one** decision you made during this lab — choosing β, choosing the data slice, choosing the judge model, choosing T4 vs BigGPU — and walk through:
>
> 1. What was the alternative you considered?
> 2. Why did you pick the one you did?
> 3. Did the result confirm or surprise you?
> 4. If you redid the lab tomorrow, what would you change?

The single decision that mattered most for this lab was choosing to stay on the T4 tier instead of trying to force a bigger model or a longer run after Colab limits became tight. I considered two alternatives: keep pushing for more complete benchmarking, or stop with the artifacts I already had and write up the run honestly. I chose the second path because the Colab session had already given me the core evidence needed for the lab: SFT loss decreased, preference data was prepared, DPO produced a positive reward gap, the GGUF smoke test generated Vietnamese text, and the benchmark notebook structure was in place even though the final numbers were not reliable.

That choice was partly pragmatic and partly methodological. Pragmatic, because I did not want to invent results or pretend a broken benchmark run was meaningful. Methodological, because in alignment work it is easy to over-read partial signals. The run did confirm one thing clearly: DPO can separate chosen from rejected pairs even on the smaller T4 setup. What surprised me was how much the evaluation stage depends on clean runtime conditions; when the judge or lm-eval path is unstable, the numbers can collapse into ties or `nan`, which makes interpretation much weaker than the training curves themselves.

If I redid the lab tomorrow, I would do one of two things: either rerun the full pipeline from a fresh Colab session with API keys already configured, or keep the current run as a lightweight submission but add a short note that the evaluation stage was constrained by runtime limits. That would make the submission more reproducible and easier to defend.

---

## 7. Benchmark interpretation (≥ 150 words)

> **Paste `07-benchmark-comparison.png` here** (or link).

Score table from `data/eval/benchmark_results.json`:

| Benchmark | SFT-only | SFT+DPO | Δ |
|---|---:|---:|---:|
| IFEval | nan | nan | n/a |
| GSM8K | nan | nan | n/a |
| MMLU (sampled) | nan | nan | n/a |
| AlpacaEval-lite | skipped | skipped | n/a |

The benchmark stage did not produce reliable numeric results in this Colab session. The notebook shows `lm-eval didn't write results JSON` for the programmatic benchmarks, and AlpacaEval-lite was skipped because no API key was available. I am keeping those outcomes explicit rather than replacing them with guessed numbers. The main takeaway is not that the model improved or regressed on these tasks, but that the benchmark harness itself was constrained by the runtime state.

That said, the structure of the benchmark notebook is still useful. It sets up the right comparison axis for later reruns: instruction following, math, broad knowledge, and judge-based preference. If I had to interpret the current run cautiously, the only defensible statement is that I do not have trustworthy evidence of an alignment tax or a benchmark gain from this particular session. MMLU and GSM8K could not be used to argue preservation or regression because the scores never stabilized. For the final submission, I would frame this as an execution-limit issue rather than as a model-result claim. The lesson is that benchmark interpretation is only as good as the evaluation harness, and in this run the harness did not complete cleanly enough to support a stronger conclusion.

---

## Bonus

- [ ] Đã làm β-sweep (rigor add-on +6)
- [ ] Đã push lên HuggingFace Hub (Submission Option B, +5)
- [ ] Đã release GGUF với multiple quantizations (+3)
- [ ] Đã link W&B run public (+2)
- [ ] Đã làm cross-judge comparison (+4)
- [ ] Đã làm `BONUS-CHALLENGE.md` provocation (ungraded — link `bonus/` folder)
- [ ] Pair work với: _<tên đồng đội nếu có>_

---

## Điều ngạc nhiên nhất khi làm lab này

_(Optional, 1–3 câu)_
