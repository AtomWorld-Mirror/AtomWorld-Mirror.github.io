---
layout: project_page
permalink: /
title: "AtomWorld-Mirror: Macro-Step World Modeling of Critical Evolution Backbones for Materials Dynamics"
description: "A state-time macro world model for physically constrained long-term materials evolution."
authors: >-
  <a href="https://zimingpan.github.io/"><span>Ziming Pan</span></a><sup>1,7,*</sup>,
  <a href="https://scholar.google.com/citations?user=8PsKswwAAAAJ&amp;hl=en">Ruge Zhang</a><sup>2,3,7,*</sup>,
  <a href="https://haozhihan.github.io/">Haozhi Han</a><sup>4,7,*</sup>,
  Junkai Zhou<sup>5</sup>, Xingyuan Chen<sup>6</sup>,<br>
  <a href="https://cs.pku.edu.cn/info/1210/1960.htm">Yifeng Chen</a><sup>4</sup>,
  <a href="http://english.ict.cas.cn/people/scien/bln/202303/t20230321_328553.html">Yunquan Zhang</a><sup>2</sup>,
  <a href="https://air.tsinghua.edu.cn/en/info/1046/1941.htm">Ting Cao</a><sup>7</sup>,
  <a href="https://air.tsinghua.edu.cn/en/info/1046/1193.htm">Yunxin Liu</a><sup>7</sup>, and
  <a href="https://www.likun.tech/">Kun Li</a><sup>7,†</sup>
affiliations: >-
  <sup>1</sup> Yonsei University, Seoul, Republic of Korea<br>
  <sup>2</sup> Institute of Computing Technology, Chinese Academy of Sciences, Beijing, China<br>
  <sup>3</sup> University of Chinese Academy of Sciences, Beijing, China<br>
  <sup>4</sup> School of Computer Science, Peking University, Beijing, China<br>
  <sup>5</sup> Economics &amp; Technology Research Institute, CNPC, Beijing, China<br>
  <sup>6</sup> Shenzhen Research Institute of Big Data, Shenzhen, China<br>
  <sup>7</sup> Institute for AI Industry Research (AIR), Tsinghua University, Beijing, China<br>
  <small><sup>*</sup> Equal contribution. <sup>†</sup> Corresponding author.</small><br>
  <em>This work was supported by Tecorigin.</em>
code: https://github.com/RecursiveScienceAIR/AtomWorld-Mirror
---

<div class="columns is-centered has-text-centered">
    <div class="column is-four-fifths">
        <h2>Abstract</h2>
        <div class="content has-text-justified">
Atomistic simulation is an important computational tool for understanding long-term materials behavior, including diffusion, defect evolution, interfacial reactions, and crack propagation. Many traditional simulators are constrained to advance at microscopic evolutionary resolution: before reaching structurally decisive states, much of the budget is consumed by low-impact local updates. We characterize this as an evolutionary-resolution bottleneck and propose AtomWorld-Mirror, a time-aware macro-step world modeling approach for the critical evolution backbone of atomic systems. For step-wise atomistic simulation, AtomWorld-Mirror distills short micro-event segments into physically reachable transitions between key states, jointly predicting sparse structural edits and accumulated physical time through latent macro-step dynamics. Local reachability, inventory conservation, and continuous-time consistency constrain each transition. By amortizing local atomic physics into a reusable latent macro model and replacing explicit micro-event replay with macro-step inference, this formulation provides a path toward substantially faster prediction of long-term materials evolution while preserving structural validity and time semantics. Across five atomic systems, spanning Cu-rich RPV steel irradiation aging, Cu–Zr metallic glass, and Li<sub>3</sub>N-based anti-perovskite solid electrolyte, macro-step inference achieves speedups of 10<sup>3</sup> to 10<sup>4</sup> times over event-by-event simulation at comparable accuracy.
        </div>
    </div>
</div>

---

## Background

Long-term materials evolution contains many local updates between structurally decisive states. In Cu-rich reactor pressure vessel (RPV) steel, vacancy-mediated atomic exchanges drive the formation of Cu-rich precipitates. The events carrying persistent structural progress appear sparsely and at different positions across microscopic trajectories.

![Key-event positions and cumulative timing](./assets/key-events.webp?v=20260912)

*Figure 1: Cu-vacancy exchange events within a fixed 1,000-micro-event replay budget. Each row corresponds to a Cu concentration and random seed; the cumulative curves show how key events accumulate over the budget.*

## Highlights

- 🧭 **Critical evolution backbone.** Learn sparse state transitions that carry persistent structural progress through long atomistic trajectories.

- ⚛️ **Physically constrained edits.** Predict changes on local candidate sites and project atom-vacancy transport to preserve material inventory.

- ⏱️ **State and time together.** Condition accumulated duration on a learned path representation, so every structural advance carries a physical clock update.

- 🔁 **Closed-loop world modeling.** Re-encode the projected prediction as the next input, using the same constrained forward path during training and rollout.

- 📏 **Multi-K macro steps.** Condition predictions on the requested horizon in `K={1 to 1024}` to learn transitions at multiple evolutionary resolutions.

- 🚀 **Accelerated materials evolution.** Mirror achieves **10³–10⁴× speedups** across five atomic systems.

## Objective

AtomWorld-Mirror learns the **critical evolution backbone**: a sequence of physically reachable states that captures persistent structural progress. Each macro transition predicts a sparse structural edit, the next latent state, and the accumulated physical duration. Repeated macro steps advance the material configuration and its physical clock together.

## Method

### From microscopic trajectories to macro transitions

**AtomWorld** provides the simulator-exposed configurations, local event support, and physical clock. **Mirror** learns the corresponding macro-step dynamics. Training segments pair a starting configuration with an endpoint, sparse edit targets, a path summary, and accumulated duration.

![AtomWorld-Mirror architecture](./assets/architecture.webp?v=20260912)

*Figure 2: AtomWorld-Mirror architecture. Graph and patch representations feed horizon-conditioned latent dynamics. Sparse edits pass through inventory projection, and the resulting state is reused at the next macro step. Reproduced from Figure 5 of the paper.*

A macro step follows four stages:

1. **Encode the current configuration.** A graph encoder and an active-patch encoder summarize the atomic environment and candidate sites.
2. **Infer a path representation.** A horizon-conditioned latent captures the microscopic evolution compressed into the macro transition. Training uses a path posterior and learns a matching prior; rollout samples the prior from current-state information.
3. **Predict and project.** Macro dynamics produce a future latent, sparse site edits, and duration/energy predictions. Projection enforces inventory-preserving edits within the local support and transport budget.
4. **Advance structure and time.** Apply the projected edit, add the predicted duration to the physical clock, and encode the new state for the next transition.


## Structural and Temporal Accuracy

Controlled validation against the KMC teacher measures sparse structural changes and expected physical time. Paired segments test endpoint prediction and single-segment duration, while long trajectories measure cumulative changes, cumulative time, and structural fidelity.

![State and time validation](./assets/validation.webp?v=20260912)

*Figure 3: Structural edits, single-segment expected-time alignment across temperatures, cumulative expected time, and structurally faithful steps over 200 macro segments. Reproduced from Figure 2 of the paper.*

## Physical Constraints

Autonomous model rollouts apply each projected edit to the previous model state. Local KMC probes evaluate the resulting transitions. Ablations measure energy error, reachability violations, inventory violations, time error, and edit error across two Cu concentrations and four temperatures.

![Physical constraint ablations](./assets/ablations.webp?v=20260912)

*Figure 4: Multi-K teacher-probe ablations. Each physical component has a distinct role in maintaining valid state-time transitions. Reproduced from Figure 3 of the paper.*

## Inference Efficiency

A macro step replaces explicit replay of intervening microscopic events with inference on a local candidate patch. The paper reports speedups of 10<sup>3</sup> to 10<sup>4</sup> across its material-system timing diagnostics. The benchmark below compares teacher replay, rate-scaling KMC, superbasin KMC, and AtomWorld-Mirror over lattice sizes and temperatures. Timing includes benchmark orchestration and batched neural inference.

![Inference timing across lattice sizes and temperatures](./assets/speedup.webp?v=20260912)

*Figure 5: End-to-end timing diagnostic across lattice sizes and temperatures. See Figure 4 and Appendix D of the paper for benchmark settings and per-system measurements.*

## Citation

```bibtex
@misc{pan2026atomworldmirror,
  title={AtomWorld-Mirror: Macro-Step World Modeling of
         Critical Evolution Backbones for Materials Dynamics},
  author={Pan, Ziming and Zhang, Ruge and Han, Haozhi and
          Zhou, Junkai and Chen, Xingyuan and Chen, Yifeng and Zhang, Yunquan and Cao, Ting and
          Liu, Yunxin and Li, Kun},
  year={2026},
  howpublished={Project manuscript},
  url={https://github.com/RecursiveScienceAIR/AtomWorld-Mirror}
}
```
