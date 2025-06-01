---
layout: distill
title: "From Continuous to Discrete: Diffusion Enters Life-Sciences Territory - AI810 Blog Post (20255297)"
description: "Discrete diffusion models have recently leapt from continuous data such as images to discrete biological sequences, promising a new wave of goal-directed design for molecules, DNA, and proteins. This post surveys two ICLR-25 highlights: GenMol, which pairs a fragment-level masked-diffusion prior with light-weight test-time guidance to excel at drug-discovery tasks, and DRAKES, which casts diffusion fine-tuning as continuous-time reinforcement learning to optimise explicit rewards while staying close to the data prior. By contrasting their objectives, algorithms, and trade-offs, we expose a set of shared techniques—continuous-time masking, parallel token restoration, and KL-anchoring—that underpin both successes. We conclude that future progress hinges on aggressive test-time scaling: importing fast inference strategies from language modelling to cheaply explore vast design spaces before expensive wet-lab validation."
date: 2025-06-01
future: true
htmlwidgets: true
hidden: true

# Anonymize when submitting
# authors:
#   - name: Anonymous

authors:
  - name: Jaewoo Lee
    affiliations:
      name: KAIST

# must be the exact same name as your blogpost
bibliography: 2025-06-01-From-Continuous-to-Discrete:-Diffusion-Enters-Life-Sciences-Territory-AI810-Blog-Post-(20255297).bib

# Add a table of contents to your post.
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - please use this format rather than manually creating a markdown table of contents.
toc:
  - name: Why Discrete Diffusion Outshines Autoregression
  - name: "…But Likelihood Alone Won’t Get You There"
  - name: A Glimpse Across Continuous Diffusion and LLMs
  - name: "GenMol: A Drug Discovery Generalist with Discrete Diffusion"
    subsections:
      - name: Motivation
      - name: Methods
        subsections:
          - name: "1  Masked discrete diffusion"
          - name: "2 Molecular-Context Guidance (MCG)"
          - name: "3 Confidence-Based Sampling"
      - name: Experiments
  - name: "Fine-Tuning Discrete Diffusion Models via Reward Optimization with Applications to DNA and Protein Design"
    subsections:
      - name: Motivation
      - name: Methods
        subsections:
          - name: "1 Continuous-time RL on a Discrete-Diffusion Generator"
          - name: "2 Direct Reward bAcKpropagation with gumbEl Softmax (DRAKES)"
          - name: "3 Theory Links to Diffusion"
      - name: Experiments
  - name: Shared Underlying Techniques
  - name: Different Design Choices & Trade-offs
  - name: Conclusion and Discussion

# Below is an example of injecting additional post-specific styles.
# This is used in the 'Layouts' section of this post.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

# From Continuous to Discrete: Diffusion Enters Life-Sciences Territory

Over the last few years, continuous-space diffusion models <d-cite key="ho2020denoising,song2020score"></d-cite> have delivered breakthroughs across multiple modalities—images <d-cite key="rombach2022high"></d-cite>, audio <d-cite key="liu2023audioldm"></d-cite>, even video <d-cite key="ho2022video"></d-cite> —by iteratively denoising Gaussian noise back into data.

Diffusion is now entering its second act: extending that same paradigm to discrete data modalities <d-cite key="austin2021structured,sahoo2024simple"></d-cite>. This expansion holds the promise of bringing the same disruptive power to life-science data that continuous diffusion brought to computer vision.

## Why Discrete Diffusion Outshines Autoregression

Discrete diffusion models possess several structural advantages over their main competitor, the autoregressive (AR) family:

- Parallel token updates. All positions are refined simultaneously, so inference scales sub-linearly with sequence length.
- Bidirectional context by construction. A sampler can denoise the tail positions first, then use that information to refine earlier tokens—impossible for strictly causal AR decoders.
- Reversible decisions. Because tokens are revisited through successive denoising steps, long-range constraints—chemical valence, Watson–Crick base pairing, ring closure—can be enforced post-hoc.

Leveraging these properties, researchers are targeting three high-impact design challenges:

| Application                            | Modelling goal                                                                                                                   | Typical reward signals                                           |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Hit generation & lead optimisation** | Generate chemically valid graphs/strings that maximise binding affinity, ADMET safety, or docking scores.                        | Docking ΔG, QED, synthesizability, Lipinski flags                |
| **DNA sequence design**                | Optimise ~200-bp enhancer sequences that drive high, cell-type-specific gene expression (e.g., HepG2).                           | MPRA-based activity oracles, chromatin accessibility classifiers |
| **Protein sequence design**            | Given a backbone conformation, produce amino-acid sequences that fold into it while improving thermodynamic stability (ΔΔG < 0). | Learned stability oracles, self-consistency RMSD                 |

## …But Likelihood Alone Won’t Get You There

A naïve discrete diffusion model maximizes log-likelihood of the training data—excellent for reproducing what nature has already tried, but ill-suited for discovering sequences that satisfy new functional objectives. In practice the model:

1. Ignores explicit objectives. Binding affinity, enzymatic turnover, or enhancer activity never appear in the likelihood.
1. Overfits dataset bias. Regions of design space unrepresented in the data—often where the best molecules live—remain unexplored.
1. Collapses under multi-objective trade-offs. Natural-looking but non-functional sequences dominate the sample pool.

To see how the discrete diffusion community is attacking these limitations, we will examine two recent ICLR-25 papers—GenMol and DRAKES—that push discrete diffusion into goal-directed generation for molecules and DNA.

| Model      | Diffusion backbone                                    | Objective-alignment strategy                                         | Key tricks                                           |
| ---------- | ----------------------------------------------------- | -------------------------------------------------------------------- | ---------------------------------------------------- |
| **GenMol** | Masked Discrete Language Model (MDLM) style diffusion | _Test-time_ remasking & denoising guided by an internal scoring head | Fragment remasking, Molecular-Context Guidance (MCG) |
| **DRAKES** | Gumbel-softmax CTMC diffusion                         | _Training-time_ reward finetuning via direct reward finetuning       | KL control for preserving naturality                 |

These papers illustrate complementary philosophies:

GenMol keeps the base model frozen and injects guidance during inference, amortising nothing. DRAKES invests computation during finetuning to bake the reward into the generator, enabling cheaper deployment.

## A Glimpse Across Continuous Diffusion and LLMs

The tension between likelihood training and task rewards is hardly unique to discrete diffusion. In both continuous diffusion and large language models (LLMs), researchers have explored three broad families of solutions:

| Family                                                                                         | Core idea                                                                                                                                                          | Pros                                                   | Cons                                                           |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ | -------------------------------------------------------------- |
| **RL finetuning** <d-cite key="black2023training,ouyang2022training"></d-cite>                 | Treat sampling as a Markov Decision Process; optimise an external reward with policy-gradient or Q-learning variants.                                              | Universally applicable, strong theoretical footing     | Requires proxy reward model; high variance; mode collapse risk |
| **Direct Policy Optimization** <d-cite key="rafailov2023direct,wallace2024diffusion"></d-cite> | Minimise a KL-regularised contrastive loss that pushes up the log-probability of high-reward (or preferred) samples against low-reward ones in an offline setting. | Stable, simpler than vanilla RL, no reward proxy model | Zero exploration due to offline property                       |
| **Test-time guidance** <d-cite key="snell2024scaling,ma2025inference"></d-cite>                | Modify the sampler (classifier guidance, score distillation, speculative decoding) without touching base weights.                                                  | No retraining; amortised cost only at inference        | Extra decoding passes; limited by oracle accuracy              |

In the sections that follow we dive into GenMol and DRAKES, dissect how each integrates rewards into discrete diffusion, and draw lessons for building the next generation of goal-oriented generative models in life sciences.

# GenMol: A Drug Discovery Generalist with Discrete Diffusion

## Motivation

Fragment-Based Drug Design (FBDD) has become a workhorse in both pharma and biotech because it lets chemists **grow or link small, experimentally validated fragments** into potent leads while keeping synthetic routes short.  
Yet most generative models still operate at the **atom–bond graph** level or on canonical SMILES strings. This brings three pain points:

1. **Semantic mismatch.** Atom-level tokens ignore medicinal-chemistry priors—ring systems, hetero-substituted aromatics, privileged scaffolds—that chemists actually manipulate.
2. **Combinatorial drag.** Autoregressive (AR) SMILES generators explore chemical space token-by-token; fragment substitutions that appear near the _end_ of the string are never “seen” when earlier tokens are chosen.
3. **Task fragmentation.** Separate models or costly re-training are typically required for de-novo enumeration, scaffold decoration, fragment linking, and reward-guided optimisation.

GenMol tackles these issues head-on by unifying **masked discrete diffusion** with the _Structure-Aware Fragment Encoding_ (**SAFE**)<d-cite key="noutahi2024gotta"></d-cite> representation:

- **SAFE Strings.** Each token is a _chemically meaningful fragment_ whose identity does **not** depend on permutation or attachment order, eliminating SMILES grammar headaches and yielding a vocabulary aligned with medicinal-chemistry intuition.
- **Bidirectional masked diffusion.** Unlike AR decoders, GenMol refines **all** fragment tokens in parallel. The sampler can therefore denoise the tail fragments first—honouring global constraints such as valence balance or docking hotspots—and then revisit earlier fragments to ensure compatibility.
- **Single checkpoint, multi-task.** A single GenMol model, trained once on a large SAFE corpus, can be steered at inference time—via Molecular-Context Guidance or external rewards—across the full fragment-based design workflow.

In short, GenMol is motivated by the need for a **chemistry-aligned, task-agnostic generator** that preserves the empirical strengths of FBDD while exploiting the global context and parallelism that discrete diffusion uniquely offers.

---

## Methods

### 1 Masked discrete diffusion

GenMol Training objective minimizes the non-equilibrium ELBO from MDLM <d-cite key="sahoo2024simple"></d-cite>:

$$
\mathcal{L}_{\text{NELBO}}
  = \mathbb{E}_{q}\!\Bigg[
      \int_{0}^{1}
        \frac{\alpha_t'}{1-\alpha_t}
        \sum_{l}
          \log\!\big\langle x_{\theta,l}(z_t,t),\,x_l \big\rangle
      \, dt
    \Bigg].
$$

Here \(z*t\) is a partially masked SAFE sequence at continuous time \(t\); \(\alpha_t\) is the masking schedule.  
During sampling, \_all* masked indices are updated in parallel:

$$
p_\theta^{(l)}\!\bigl(z_s \mid z_t\bigr)
  = \operatorname{Cat}\!\Biggl[
      1-\alpha_s\,m
      + (\alpha_s-\alpha_t)\;
        \frac{
          \exp\!\bigl(\tfrac{1}{\tau}\,
          \log x_{\theta,l}(z_t,t)\bigr)
        }{
          \displaystyle
          \sum_{j}
            \exp\!\bigl(\tfrac{1}{\tau}\,
            \log x_{\theta,l}^{(j)}(z_t,t)\bigr)
        }
    \Biggr],
$$

where $$\tau$$ is the soft-max temperature.

### 2 Molecular-Context Guidance (MCG)

MCG borrows the spirit of classifier-free guidance in image diffusion but removes the need for an **external** property predictor.  
Instead, GenMol trains **two views of the _same_ denoiser** in a multi-task fashion:

| View                                             | Conditioning tokens                                                                                                             | Role                                                                     |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Context-aware head** $$x_\theta^{\text{ctx}}$$ | full SAFE string **plus** an auxiliary _context vector_ $$c$$ (e.g. fragment bag-of-words, docking hotspot mask, scaffold type) | Learns to reconstruct fragments that are globally consistent with $$c$$. |
| **Context-free head** $$x_\theta$$               | SAFE string **only**, drop the context vector (replace with `\<no_ctx\>` token)                                                 | Learns the unconditional data distribution.                              |

During **training** the two heads share all transformer layers except the final projection, so extra parameters are negligible.  
At **sampling** time we blend the two logits:

<div align="center">

\[
\log x*\theta^{(w)}(z_t,t,c)
\;=\;
w\;\log x*\theta(z*t,t)\;+\;
(1-w)\;\log x*\theta^{\text{ctx}}(z_t,t,c),
\tag{4}
\]

</div>

- $$w\in[0,1]$$ is the **guidance weight**.
- $$w\!\downarrow$$ → pure, unbiased samples; $$w\!\uparrow$$ → context-faithful but potentially lower diversity.
- Because both heads live in the same network, the blend costs **one** forward pass—no extra GPU memory, no back-prop, no separate classifier.

### 3 Confidence-Based Sampling

Standard diffusion would resample **every** mask at every step, frequently overwriting good predictions. GenMol instead adopts **confidence sampling** <d-cite key="chang2022maskgit"></d-cite> with three hyper-parameters $$(N,\tau,r)$$. For each still-masked position $$l$$ at reverse-time $$t$$:

1. **Predict a token.**  
   Apply the soft-max with temperature $$\tau$$ to the mixed logits from Eq.&nbsp;(4) and sample an index $$i^\ast$$ for each still-masked position $$l$$.

2. **Compute a confidence score** that blends model certainty with controllable exploration:

   $$
   c_t^{(l)}
      \;=\;
      \log p_{\theta,i^\ast}^{(l)}
      \;+\;
      r\,t\,\varepsilon,
      \qquad
      \varepsilon \sim \text{Gumbel}(0,1)
   \tag{8}
   $$

   Here $$p_{\theta,i^\ast}^{(l)}$$ is the model’s probability for the sampled category at time step $$t$$,  
   while the additive Gumbel noise $$\varepsilon$$ is **scaled by $$r\,t$$**—large and exploratory in early iterations, then cooling off as $$t \!\downarrow$$ to make late steps increasingly deterministic.

3. **Freeze the confident tokens.**  
    Rank all still-masked positions by their $$c*t^{(l)}$$ scores and permanently unmask the top $$N$$; the remainder stay masked for the next reverse-diffusion step.
   Effectively, GenMol makes a \_parallel\* guess for every fragment, then “commits” only the most confident ones.  
   The procedure repeats until no masks remain, yielding:

- **Quality–speed trade-off.**  
  Larger $$N$$ or smaller $$\tau$$ reduces steps (faster) but can hurt quality; higher randomness $$r$$ broadens exploration.  
  Appendix B shows that $$N\!=\!1,\;\tau\!=\!0.5,\;r\!=\!0.5$$ attains the best validity/quality balance, while $$N\!=\!3$$ triples speed with only moderate quality loss.
- **Context sensitivity.**  
  Because only high-confidence positions freeze, later denoising rounds can adapt earlier uncertain fragments to satisfy long-range chemistry constraints, a capability AR models lack.

---

## Experiments

GenMol is trained **once** on the SAFE corpus and then reused _unchanged_ across four evaluations.

- **De-novo generation (ZINC-250k).**  
  With $$N=1,\;\tau=0.5,\;r=0.5$$, GenMol achieves 100 % validity, 99.7 % uniqueness, and pushes the “quality” metric (valid ∧ drug-like ∧ synthesizable) to **84.6 %**, far above SAFE-GPT’s 54.7 %, while sampling faster.

- **Fragment-constrained generation (10 FDA drugs).**  
  Across five constraint tasks, GenMol lifts success rates by 5–16 pp and maintains ≥ 96 % validity, outperforming SAFE-GPT on every metric–task pair.

- **Goal-directed hit generation (23 PMO targets).**  
  Fragment remasking + MCG delivers the top-10 AUC on 19 / 23 targets and the highest summed AUC across all baselines (REINVENT, Graph-GA, BO, etc.).

- **Lead optimisation (docking vs. PARP1, BRAF, JAK2, …).**  
  Starting from experimental seeds, GenMol lowers docking energies by up to **3 kcal·mol⁻¹** relative to seeds and surpasses GA-based optimisers on four of six proteins.

All optimisation runs execute on **a single GPU** with no additional fine-tuning, underscoring the practicality of masked discrete diffusion paired with fragment-level exploration.

---

# Fine-Tuning Discrete Diffusion Models via Reward Optimization with Applications to DNA and Protein Design

## Motivation

State-of-the-art **masked _discrete diffusion_** models already learn an excellent _prior_ over biological sequences (DNA enhancers, proteins, even text) by running a **continuous-time Markov chain (CTMC)** <d-cite key="campbell2022continuous"></d-cite>that randomly _masks_ tokens and then denoises them.  
Yet real design tasks demand **pushing samples toward explicit rewards**—expression level, folding stability, toxicity control—_without destroying that powerful diffusion prior_. Naïve reward-only fine-tuning breaks down: the model “reward-hacks”, drifts OOD, and loses validity.

**DRAKES** tackles this by casting _fine-tuning of a discrete-diffusion model itself_ as a **continuous-time RL problem** that **maximises reward _and_ keeps the generator close (in KL) to the original diffusion kernel**. The outcome: sequences that stay “natural-like” while reaching much higher task scores.

---

## Methods

### 1 Continuous-time RL on a Discrete-Diffusion Generator

Start from a pre-trained **masked discrete-diffusion CTMC** with transition rates $$Q^{\text{pre}}(t)$$.  
We search for new rates $$Q^\theta(t)$$ that solve

$$\theta^*=\arg \max_{\theta\in\Theta}\mathbb E_{x_{0:T}\sim P^\theta}[r(x_T)]-\alpha \mathbb E_{x_{0:T}\sim P^\theta} \left[ \int_{t=0} ^T\sum_{y\neq x_t  }\left( Q^{\theta_{pre}}_{x_t,y}(t)-Q^\theta_{x_t,y}(t) + Q^\theta_{x_t,y}(t)\log\frac{Q^\theta_{x_t,y}(t)}{Q^{\theta_{pre}}_{x_t,y}(t)} \right)dt \right]$$

_Because the base generator is itself a CTMC discrete diffusion, this KL is computed **at every infinitesimal denoising step**, directly regularising the entire diffusion path._  
At optimum the terminal distribution is the **exponentially tilted diffusion prior**

$$
p^\star_t(x)\;\propto\; \exp(V_t(x)/\alpha)p_t^{pre}(x),
$$

so the original diffusion manifold is preserved while reward is boosted.

### 2 Direct Reward bAcKpropagation with gumbEl Softmax (DRAKES)

| Stage            | Discrete-diffusion angle                          | Details                                                                                                                                                                                            |
| ---------------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sampling**     | Make CTMC _diffusion_ trajectories differentiable | Replace every discrete fragment-token hop with a **Gumbel–Softmax** relaxation, so gradients propagate through the diffusion steps.                                                                |
| **Optimisation** | Gradient ascent on the RL objective               | Compute minibatch estimates, back-prop through the _relaxed diffusion path_, and update \(\theta\) with Adam. A straight-through trick on the final (hard) denoise step sharpens oracle alignment. |

> All learning happens _inside_ the diffusion model—no separate policy network or value critic is introduced.

### 3 Theory Links to Diffusion

- The optimal rates satisfy  
  $$Q^{\theta^*}_{x,y}(t)=Q^{\theta_{pre}}_{x,y}(t)\exp(\{V_t(y)-V_t(x)\}/\alpha)$$,  
  mirroring a **score-matching update** inside the diffusion kernel.
- Classic **classifier guidance for discrete diffusion** appears as a _Doob transform_ special case; unlike DRAKES it requires no training but omits KL regularisation and underperforms on proteins.

---

## Experiments

**DRAKES is fine-tuned on a task-specific reward model and evaluated—without further architectural changes—across three very different domains.**

- **DNA enhancer design.**  
  RL fine-tuning boosts the predicted transcriptional _activity_ of generated 200-bp sequences by **≈ 13 %** relative to the strongest baseline, _while simultaneously_ increasing motif fidelity and 3-mer realism.  
  → Shows that the KL-regularised CTMC keeps samples “natural-like” even under strong reward pressure.

- **Protein inverse folding (Megascale stability set).**  
  The proportion of designed sequences that both _fold_ and remain _thermodynamically stable_ climbs to **78.6 %**, compared with **≈ 64 %** for SMC or TDS guidance.  
  → Demonstrates that end-to-end back-prop through discrete hops can out-perform search-based guidance.

- **Toxicity-controlled text generation.**  
  Achieves the best mean and median _toxicity scores_ among all contenders (pre-trained diffusion, classifier-guided diffusion, SMC, TDS).  
  → Confirms that the method generalises beyond biological tokens to natural-language tasks.

_Ablation studies_ verify that the trajectory-level KL term is essential for preventing reward hacking, and that the Gumbel-Softmax temperature schedule remains robust over a wide hyper-parameter range.

---

# Shared Underlying Techniques

| Core element                 | **GenMol**                                                                | **DRAKES**                               | Shared benefit                                      |
| ---------------------------- | ------------------------------------------------------------------------- | ---------------------------------------- | --------------------------------------------------- |
| **Forward (noise) step**     | Randomly **mask** SAFE-fragments → continuous-time CTMC                   | Randomly **mask** sequence tokens → CTMC | Equivalent to a continuous-time BERT-MLM objective  |
| **Reverse (denoising) step** | Transformer predicts _all_ fragments **in parallel** (non-autoregressive) | Same parallel token restoration          | Length flexibility, fast sampling, order invariance |

> **In short:** Both papers extend the BERT masking idea to continuous time, yielding _masked discrete diffusion_ models that inherit parallel decoding and amenability to gradient guidance.

---

# Different Design Choices & Trade-offs

| Perspective               | **GenMol** _(fragment-based drug design)_                                                                         | **DRAKES** _(biology & text reward tuning)_                                                 |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Primary goal**          | “**One-shot pre-train, multi-task zero-shot**” exploration (de-novo, fragment linking, scaffold decoration, etc.) | **Fine-tune for explicit rewards** (enhancer activity, protein stability, toxicity control) |
| **Guidance method**       | _Molecular-Context Guidance_ (logit blending) + reward-free fragment remasking                                    | Gumbel-Softmax relaxation enables **direct reward-gradient back-prop**                      |
| **Supervision signal**    | Large SAFE corpus only; _no_ reward model required                                                                | Requires an **external reward/oracle**; RL fine-tuning with trajectory-level KL             |
| **Naturalness guarantee** | Uses the original fragment prior; chemically meaningful SAFE tokens ensure synthesizability                       | Adjustable α-weighted KL keeps samples near the pretrained prior                            |
| **Representation unit**   | **Chemistry fragments (SAFE)** → embeds medicinal-chemistry priors                                                | Generic **tokens** (bases / amino acids / words) → domain-agnostic                          |
| **Sampling strategy**     | Confidence-freezing + fragment remasking → rapid local edits                                                      | Standard CTMC sampling with reward gradients influencing every hop                          |
| **Compute cost**          | Single pre-training run; inference light (1 GPU)                                                                  | Extra fine-tuning epochs with Gumbel-Softmax back-prop (heavier GPU demand)                 |

# Conclusion and Discussion

Discrete diffusion is rapidly emerging as a unifying framework for goal-oriented generation in the life sciences. GenMol shows that a single masked-diffusion prior over chemistry fragments can serve as a versatile, medicinal-chemistry-aligned generator, while DRAKES demonstrates that the same mathematical engine can be reward-tuned—without sacrificing validity—to meet demanding objectives in DNA, proteins, and even natural-language sequences.

However, applying diffusion to real biological design problems is still in its infancy, and much work lies ahead.
My conjecture on this research area is that test-time scaling will become critically important for life science. The domains we care about—drug discovery, synthetic biology, protein engineering—often require extremely expensive evaluation protocols (wet-lab assays, in-vitro and in-vivo studies). Inference-time computation is cheap by comparison, but it must become cheaper still so we can explore vast design spaces before committing to physical validation. A promising path is to import test-time computation strategies proven indispensable in modern language modelling—speculative decoding, draft-and-refine loops, classifier-free or hybrid guidance—and adapt them to discrete diffusion in the scientific setting.
