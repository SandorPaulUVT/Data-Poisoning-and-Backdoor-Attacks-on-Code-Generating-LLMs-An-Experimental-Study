# Literature survey: Data Poisoning and Backdoor Attacks on Code-Generating LLMs

**Suggested thesis chapters used below:** Background (BG), Related Work (RW), Threat Model (TM), Methodology (METH), Evaluation (EVAL), Defenses discussion (DEF), Discussion/Limitations (DISC).

## Read these first (in order)

1. `schuster2021autocomplete`: the origin of poisoning code models.
2. `cotroneo2024vulnerabilities`: closest to your experiment (poisoning rate sweep, NL-to-code, insecure code).
3. `aghakhani2024trojanpuzzle`: why naive payloads get caught by static analysis.
4. `yan2024codebreaker`: current state of the art; shows where Bandit/Semgrep-style evaluation falls short.
5. `chen2025security`: survey; use its reference list to find anything else you need.

---

## 1. Foundations: backdoors and data poisoning

### `gu2017badnets`: BadNets (arXiv 1708.06733) — [PDF](https://arxiv.org/pdf/1708.06733)
- **Relevant:** Introduced the idea of a backdoored network that scores well on normal validation data but misbehaves on inputs containing an attacker-chosen trigger. Poisoned the training set to install it; showed the backdoor can survive transfer learning (stop sign with a sticker).
- **Use:** BG (definition of backdoor, trigger, target behavior), TM (outsourced training / pre-trained model supply chain).

### `carlini2024poisoning`: Poisoning Web-Scale Training Datasets is Practical (IEEE S&P 2024) — [PDF](https://arxiv.org/pdf/2302.10149)
- **Relevant:** Shows concrete ways an attacker can get malicious examples into web-scale datasets (split-view and frontrunning poisoning), including a cheap attack on a fraction of LAION-400M.
- **Use:** TM (answers "how would poisoned data actually reach a training corpus?"), motivation in the introduction.

### `wan2023poisoning`: Poisoning Language Models During Instruction Tuning (ICML 2023) — [PDF](https://proceedings.mlr.press/v202/wan23b/wan23b.pdf)
- **Relevant:** A small number of poisoned instruction-tuning examples (around 100) can make a trigger phrase degrade model behavior across many tasks; larger models were more vulnerable; data filtering and capacity reduction helped only moderately.
- **Use:** METH (why vary the number of poisoned samples, not only the percentage), DISC (scaling of vulnerability with model size).

---

## 2. Poisoning and backdoors in code models (core of the thesis)

### `schuster2021autocomplete`: You Autocomplete Me (USENIX Security 2021) — [PDF](https://arxiv.org/pdf/2007.02220)
- **Relevant:** First study of poisoning neural code completion (Pythia and GPT-2). Attacker steers the model toward insecure suggestions such as ECB mode for AES, SSLv3, or a low iteration count for password-based encryption. Covers data poisoning and model poisoning, targeted attacks (only for certain repos or developers), and finds existing defenses largely ineffective.
- **Use:** RW (baseline attack), TM (data vs. model poisoning), METH (choice of insecure payloads), DEF.

### `aghakhani2024trojanpuzzle`: TrojanPuzzle (IEEE S&P 2024) — [PDF](https://arxiv.org/pdf/2301.02344)
- **Relevant:** Points out that earlier attacks put the insecure code directly into poisoned files, so static analysis can filter them out. Proposes COVERT (payload hidden in docstrings/comments) and TROJANPUZZLE (never includes certain payload parts in the poison data yet still induces them). Evaluated on two model sizes with Semgrep-style filtering in mind.
- **Use:** METH (trigger/payload design and why plain templates are the "easy" case), EVAL (why a static-analysis filter is a meaningful baseline), DISC.

### `yan2024codebreaker`: CodeBreaker (USENIX Security 2024) — [PDF](https://arxiv.org/pdf/2406.06822)
- **Relevant:** Uses an LLM to transform malicious payloads so that both poisoned data and generated code evade static analysis and LLM-based vulnerability detection. Compares against Simple, Covert and TrojanPuzzle, covers many vulnerability types, and includes user studies.
- **Use:** RW (state of the art), DISC (limits of Bandit/Semgrep as the only security oracle; ties directly to the validity threats of your evaluation).

### `cotroneo2024vulnerabilities`: Vulnerabilities in AI Code Generators (ICPC 2024) — [PDF](https://arxiv.org/pdf/2308.04451)
- **Relevant:** Targeted poisoning of NL-to-code models by injecting increasing amounts of vulnerable code. Finds that models are vulnerable even to small amounts of poison, that success depends on model architecture and poisoning rate but not on vulnerability type, and that the attack does not hurt code correctness (so it is hard to notice). Builds the PoisonPy dataset (mentioned in the Chen et al. survey).
- **Use:** METH (poisoning-rate sweep design, metrics), EVAL (what to compare your numbers against), possibly dataset reuse (check PoisonPy availability).

### `improta2025detecting`: Detecting Stealthy Data Poisoning Attacks in AI Code Generators (arXiv 2508.21636) — [PDF](https://arxiv.org/pdf/2508.21636)
- **Relevant:** Evaluates spectral signatures, activation clustering and static analysis against triggerless poisoning on CodeBERT, CodeT5+ and AST-T5. Reports that all struggle: representation-based methods fail to separate poisoned samples and static analysis has false positives and false negatives.
- **Use:** DEF (best single source for "what happens when you run known defenses on code poisoning"), future work.

### `chen2025security`: Security of Language Models for Code: A Systematic Literature Review (arXiv 2410.15631) — [PDF](https://arxiv.org/pdf/2410.15631)
- **Relevant:** Systematic review of 67-68 papers on attacks and defenses for code language models (backdoor and adversarial), plus a summary of common datasets, models and metrics. Lists evaluation datasets including SecurityEval, LLMSecEval and PoisonPy.
- **Use:** RW backbone, and a way to check completeness. Mine its reference tables for papers I did not collect.

---

## 3. Defenses (reviewed and discussed, not implemented)

### `tran2018spectral`: Spectral Signatures in Backdoor Attacks (NeurIPS 2018) — [PDF](https://arxiv.org/pdf/1811.00636)
- **Relevant:** Backdoored training examples leave a detectable trace in the spectrum of the covariance of learned representations; outlier scoring along the top singular direction can remove poisoned samples. Shown on image data.
- **Use:** DEF (what the method assumes and why that may not hold for generation or triggerless attacks).

### `chen2018activation`: Activation Clustering (arXiv 1811.03728) — [PDF](https://arxiv.org/pdf/1811.03728)
- **Relevant:** Clusters the last-hidden-layer activations of the training data per class (after dimensionality reduction) to spot a poisoned cluster; does not require a trusted clean dataset.
- **Use:** DEF.

### `liu2018finepruning`: Fine-Pruning (RAID 2018) — [PDF](https://arxiv.org/pdf/1805.12185)
- **Relevant:** Model-side defense: prune neurons dormant on clean data, then fine-tune on clean data. Also introduces a pruning-aware attack.
- **Use:** DEF (a defense that does not need to find poisoned samples).

---

## 4. Security of generated code and evaluation

### `pearce2022asleep`: Asleep at the Keyboard? (IEEE S&P 2022) — [PDF](https://arxiv.org/pdf/2108.09293)
- **Relevant:** About 40% of the 1,689 programs Copilot generated across 89 CWE-based scenarios were vulnerable, found with CodeQL plus manual inspection. Shows that even unpoisoned models write insecure code.
- **Use:** EVAL (you need a *clean-model control*, because the baseline insecure rate is not zero), METH (scenario design per CWE).

### `siddiq2022securityeval`: SecurityEval (MSR4P&S 2022) — [PDF](https://lsiddiqsunny.github.io/public/securityeval.pdf)
- **Relevant:** 130 Python prompts covering 75 vulnerability types mapped to CWE, each with an insecure example; dataset is on Hugging Face (`s2e-lab/SecurityEval`).
- **Use:** EVAL (held-out prompts to test the triggered vs. untriggered model). Small, so treat results as indicative.

### `bandit`: Bandit — [GitHub](https://github.com/PyCQA/bandit)
- **Relevant:** Python security linter that parses each file into an AST and runs checks on it. Easy to automate on thousands of generated snippets.
- **Use:** EVAL (primary insecure-code detector). Limitation to state in DISC: it flags known patterns only, and the code-poisoning literature shows pattern-based checks can be evaded.

### `semgrep`: Semgrep — [GitHub](https://github.com/semgrep/semgrep)
- **Relevant:** Open-source static analysis with rules that look like the code being matched; community rule registry.
- **Use:** EVAL (second detector so results do not depend on one tool). The TrojanPuzzle paper uses Semgrep-style rules in its argument, so it is a natural pairing.

---

## 5. Models and fine-tuning method

### `hu2022lora`: LoRA (ICLR 2022) — [PDF](https://arxiv.org/pdf/2106.09685)
- **Relevant:** Freezes the pre-trained weights and trains small low-rank update matrices, greatly reducing trainable parameters and GPU memory.
- **Use:** METH (how you fine-tune on free Colab-class hardware); also note that LoRA fine-tuning is a slightly different attack surface than full fine-tuning (put in DISC).

### `wang2021codet5`: CodeT5 (EMNLP 2021) — [PDF](https://arxiv.org/pdf/2109.00859)
- **Relevant:** Encoder-decoder code model with identifier-aware pre-training; small variants are practical on modest hardware.
- **Use:** METH (candidate model), RW (it is also the victim model in earlier code-backdoor work).

---

## Cut from the longer list (11; still in the `_full` files)

`li2022backdoor`, `xu2023instructions`, `souly2025poisoning`, `hubinger2024sleeper`, `ramakrishnan2022backdoors`, `sun2022coprotector`, `li2023multitarget`, `yang2024stealthy`, `improta2024unrepairing`, `tony2023llmseceval`, `li2023starcoder`

If you want to add one or two back, the most useful are `ramakrishnan2022backdoors` (earliest code-backdoor work, includes a spectral-signature defense on code) and `tony2023llmseceval` (a second prompt set for evaluation).

## Caveats

- I could not download PDFs in this environment, so each entry carries a PDF link. Download them into one folder and name them by BibTeX key.
- Three entries are tagged `(verify)` in `references.bib` (Fine-Pruning, TrojanPuzzle, Asleep at the Keyboard): page numbers or the final volume were not confirmed. Fill them in from the publisher record before submission.
