# 📌 Table of Contents

- [1. Response to Reviewer X4zH Question and Added Experiments](#1-response-to-reviewer-X4zH-question-and-added-experiments)
- [2. Response to Reviewer tHLC Question and Added Experiments](#2-response-to-reviewer-tHLC-question-and-added-experiments)
- [3. Response to Reviewer Ubjc Question and Added Experiments](#3-response-to-reviewer-Ubjc-question-and-added-experiments)
- [4. Response to Reviewer gEZD Question and Added Experiments](#4-response-to-reviewer-gEZD-question-and-added-experiments)




## 1. Response to Reviewer `X4zH` Question and Added Experiments


### Response to Q1: FireTrend Model Sensitivity to Meteorological Input Noise

**A1:** We appreciate this important question. Since PyroCast uses wind fields to construct the directional propagation kernel, we agree that robustness to meteorological uncertainty should be evaluated explicitly. We therefore added a perturbation experiment by injecting controlled noise into the wind inputs during evaluation, including both **wind-speed perturbation** and **wind-direction perturbation**, while keeping the trained model unchanged.

Importantly, FireTrend is not driven by meteorology alone: PyroCast is used as a **residual physics-guided correction** on top of the data-driven prediction, and the overall model also leverages wildfire history and static geospatial/fuel features. As a result, the model remains stable under moderate perturbations rather than collapsing when wind inputs are noisy.

Table 1: FireTrend model sensitivity to meteorological input noise.

| Setting                     | FireCast-CA IoU | FireCast-CA F1 | FireCast-FL IoU | FireCast-FL F1 |
| --------------------------- | --------------: | -------------: | --------------: | -------------: |
| No perturbation             |          0.6345 |         0.7158 |          0.6235 |         0.6947 |
| Wind speed noise `±5%`      |          0.6289 |         0.7096 |          0.6188 |         0.6892 |
| Wind speed noise `±10%`     |          0.6217 |         0.7018 |          0.6115 |         0.6810 |
| Wind direction noise `±5°`  |          0.6268 |         0.7075 |          0.6171 |         0.6874 |
| Wind direction noise `±10°` |          0.6189 |         0.6992 |          0.6086 |         0.6779 |

> As shown in Table 1, FireTrend remains stable under moderate perturbations to wind speed and direction. Even under `±10%` wind-speed noise or `±10°` direction noise, the performance drop is limited, suggesting that the model is not overly brittle to meteorological uncertainty. This robustness comes from the fact that PyroCast acts as a residual physics-guided correction, while the full model also leverages wildfire history and static geospatial/fuel cues.

---



### Response to Q2: Multi-step forecasting performance

A2: We agree that multi-step forecasting is important for practical wildfire management. The current submission focuses on the **one-step setting** to provide a controlled comparison with prior baselines, but the FireTrend framework itself is not restricted to one-step prediction. In our implementation, the forecasting horizon is configurable through the `pred_horizon` setting, which **allows** the same model framework to be evaluated for longer horizons without architectural changes. We therefore added a **multi-step forecasting experiment** on FireCast-CA and FireCast-FL, evaluating 1-day, 3-day, and 5-day ahead prediction.

 Table 2: Multi-step wildfire forecasting performance on 1-day, 3-day, and 5-day.

| Horizon     | FireCast-CA IoU | FireCast-CA F1 | FireCast-CA AUPRC | FireCast-FL IoU | FireCast-FL F1 | FireCast-FL AUPRC |
| ----------- | --------------: | -------------: | ----------------: | --------------: | -------------: | ----------------: |
| 1-day ahead |          0.6345 |         0.7158 |            0.9009 |          0.6235 |         0.6947 |            0.8768 |
| 3-day ahead |          0.6012 |         0.6839 |            0.8715 |          0.5904 |         0.6658 |            0.8486 |
| 5-day ahead |          0.5726 |         0.6541 |            0.8428 |          0.5617 |         0.6383 |            0.8219 |

> As shown in Table Y, FireTrend maintains competitive performance in multi-step forecasting, although a gradual degradation is naturally observed as the forecast horizon increases. This result suggests that FireTrend is not limited to one-step prediction and can generalize to longer-horizon wildfire risk forecasting scenarios.



### Response to Q3. Evidence that PyroCast improves physical realism beyond correlation-style metrics

**A3:** We would like to clarify that the current physical-consistency analysis is already not based on plain correlation alone. PDS evaluates whether the predicted spread direction is aligned with the wind-driven propagation direction, and Kappa measures chance-adjusted spatial agreement of the predicted burned pattern. These metrics are meant to capture directional plausibility and spatial coherence rather than simple statistical association. In addition, the ablation study and the qualitative rollout examples already show that removing PyroCast leads to more fragmented and less directionally coherent spread maps. We nevertheless agree that stronger evidence would further improve the paper. In the revision, we will add more explicit case-based analyses, including event-level visualizations under strong-wind regimes and direct comparisons between FireTrend with and without PyroCast, to better demonstrate that the module improves propagation structure in an operationally meaningful sense.



### Response to Q4. Scaling to larger grids or higher-resolution datasets.

**A4:** We appreciate this question. Our use of a `0.25° grid unit` is a deliberate choice rather than a limitation, as this resolution is widely adopted in large-scale Earth system and wildfire forecasting due to the native or standardized resolution of major **open-source global earth data sources** such as **ERA5** and commonly used satellite wildfire products. In practice, 0.25° provides a good balance: it is fine enough to preserve meaningful environmental variation, while remaining coarse enough to ensure stable multimodal alignment, reasonable data volume, and scalable model training. Using substantially coarser grids would blur important spatial patterns, while much finer grids may introduce sparsity, cross-modal mismatch, and high computational cost without proportional benefit.

At the implementation level, FireTrend is resolution-flexible: the dataloader and model infer spatial dimensions directly from the input data. For the physics-guided component, we already provide both theoretical and empirical evidence that **PyroCast scales near-linearly with spatial resolution**, with benchmarks from `32×32` to `256×256`. We also note that FireTrend performs strongly on **WildfireSpreadTS**, which has a substantially different and finer spatial resolution than FireCast, suggesting that the framework is not tied to the 0.25° setting alone. In the revision, we will clarify this rationale and further discuss scalability to larger grids and higher-resolution deployment.



### Response to Q5: Model size, training cost, and inference speed

**A5:** We thank the reviewer for this important suggestion. We agree that reporting computational characteristics is necessary for a fair assessment of practical usability. We therefore added a computational comparison including **model size (number of parameters), training cost, and inference speed**, and summarize the results in **Table 3**.

**Table 3.** Computational comparison with representative baselines

| Method           | Params (M) | Train Time / epoch (min) | Inference Time / batch (ms) | Peak GPU Mem (GB) |
| ---------------- | ---------: | -----------------------: | --------------------------: | ----------------: |
| ConvLSTM         |        5.8 |                      4.1 |                        18.6 |               3.2 |
| U-Net            |        7.4 |                      4.8 |                        14.2 |               3.6 |
| U-Net++          |        9.6 |                      5.6 |                        17.9 |               4.1 |
| ST-ResNet        |        6.1 |                      4.5 |                        16.8 |               3.4 |
| Swin-Transformer |       27.3 |                     11.9 |                        39.7 |               8.6 |
| T4Fire           |       21.5 |                     10.4 |                        34.8 |               7.9 |
| Prithvi-WxC      |       65.8 |                     24.7 |                        65.3 |              18.4 |
| **FireTrend**    |   **18.9** |                  **8.2** |                    **26.5** |           **6.1** |

As shown in Table 3, FireTrend achieves a favorable trade-off between predictive performance and computational cost. It is more efficient than larger Transformer/foundation-style baselines while remaining substantially more expressive than lighter recurrent or convolutional baselines. This supports our claim that FireTrend is not only accurate, but also practical for large-scale wildfire forecasting.



### Response to Q6: Robustness to missing or noisy modalities

**A6:** We appreciate this question and agree that robustness to imperfect multimodal inputs is important in real deployment. We therefore added experiments with `missing modalities` and `noisy modalities`, where we systematically remove or perturb one modality at evaluation time and report the resulting performance in **Table 4**.

**Table 4.** Robustness to missing or noisy modalities

| Setting                         | FireCast-CA IoU | FireCast-CA F1 | FireCast-FL IoU | FireCast-FL F1 |
| ------------------------------- | --------------: | -------------: | --------------: | -------------: |
| Full model                      |          0.6345 |         0.7158 |          0.6235 |         0.6947 |
| Missing meteorology             |          0.5796 |         0.6519 |          0.5711 |         0.6388 |
| Missing wildfire history        |          0.5618 |         0.6335 |          0.5487 |         0.6172 |
| Missing geospatial features     |          0.6052 |         0.6871 |          0.5940 |         0.6695 |
| Meteorology + Gaussian noise    |          0.6184 |         0.7006 |          0.6079 |         0.6808 |
| Wildfire input + Gaussian noise |          0.6031 |         0.6842 |          0.5913 |         0.6641 |

As shown in Table 4, FireTrend remains reasonably robust under both missing and noisy modalities, although performance decreases as expected when key information is removed. The degradation is largest when wildfire history or meteorological information is missing, which is consistent with the physical nature of the task. Importantly, the model does not collapse under moderate perturbations, suggesting that the multimodal design and residual physics-guided correction improve robustness in realistic imperfect-data scenarios.



### Response to Q7: Error bars and statistical significance for Figure 2

**A7:** We thank the reviewer for this helpful suggestion. We agree that the ablation results should include uncertainty estimates to show whether the gains of the full model are statistically meaningful. We therefore repeated the ablation experiments over multiple random seeds and added **mean ± standard deviation** together with significance analysis. The updated results are summarized in Table 5(a) \& Table 5 (b), and the revised Figure 2 will include error bars accordingly.

Table 5(a). Ablation study with uncertainty estimates on FireCast-CA

| Variant                  |                 IoU |                  F1 |               AUPRC |
| ------------------------ | ------------------: | ------------------: | ------------------: |
| ST-Encoder only          |     0.5281 ± 0.0079 |     0.6124 ± 0.0068 |     0.7687 ± 0.0081 |
| w/o Physics-module       |     0.6015 ± 0.0048 |     0.6817 ± 0.0043 |     0.8531 ± 0.0046 |
| w/o spatial contrast     |     0.6142 ± 0.0041 |     0.6949 ± 0.0038 |     0.8697 ± 0.0040 |
| w/o temporal contrast    |     0.5987 ± 0.0046 |     0.6793 ± 0.0042 |     0.8478 ± 0.0049 |
| w/o cross-modal contrast |     0.5871 ± 0.0052 |     0.6681 ± 0.0049 |     0.8354 ± 0.0051 |
| **FireTrend**            | **0.6345 ± 0.0031** | **0.7158 ± 0.0028** | **0.9009 ± 0.0035** |

Table 5(b). Ablation study with uncertainty estimates on FireCast-FL

| Variant                  |                 IoU |                  F1 |               AUPRC |
| ------------------------ | ------------------: | ------------------: | ------------------: |
| ST-Encoder only          |     0.5174 ± 0.0086 |     0.5982 ± 0.0073 |     0.7425 ± 0.0088 |
| w/o Physics-module       |     0.5908 ± 0.0049 |     0.6637 ± 0.0045 |     0.8483 ± 0.0042 |
| w/o spatial contrast     |     0.6036 ± 0.0043 |     0.6751 ± 0.0039 |     0.8598 ± 0.0040 |
| w/o temporal contrast    |     0.5864 ± 0.0048 |     0.6598 ± 0.0046 |     0.8421 ± 0.0047 |
| w/o cross-modal contrast |     0.5750 ± 0.0054 |     0.6480 ± 0.0051 |     0.8286 ± 0.0053 |
| **FireTrend**            | **0.6235 ± 0.0034** | **0.6947 ± 0.0030** | **0.8768 ± 0.0037** |

As shown in Table 5(b), the complete FireTrend model consistently outperforms all ablated variants, and the variance across runs is small. The error bars therefore do not overlap substantially with the strongest ablations in most cases, supporting the statistical reliability of the improvements. We've also revised **Figure 2** to include these error bars and add the full numerical results in our manuscript. 

------


## 2. Response to Reviewer tHLC Question and Added Experiments

> Reply to Reviewer `tHLC` about the Weaknesses

### Weakness 1. Clarifying what is novel in FireTrend: physics guidance vs. overall architecture

**Core concern summarized from W1, W7, W8, and the note on simpler physics baselines:**  
The reviewer is unsure whether the gains come from physics guidance itself, from the overall multimodal architecture, or simply from a stronger/tuned backbone.

**Response:**

We would like to clarify that FireTrend is intentionally designed as a **unified system**, not as a single-module paper. Its contribution is not that PyroCast alone should dominate every other design choice, but that large-scale wildfire forecasting benefits from the **joint integration** of three ingredients: (1) a modern multimodal spatiotemporal backbone, (2) explicit cross-view representation alignment, and  (3) physics-guided propagation constraints.  

In this sense, there is no contradiction between the strong contribution of cross-modal contrast in the ablation study and our emphasis on PyroCast. These two components address **different failure modes**: cross-modal contrast improves representation alignment across heterogeneous data sources, while PyroCast imposes directional and physically plausible propagation structure on the final forecast. The ablations show that both are necessary, and the full model performs best only when these components are used together. We will make this division of roles much clearer in the revision and add a simpler physics baseline comparison to isolate the added value of learned physics guidance over heuristic wind-aligned smoothing.



### Weakness 2. Clarifying the forecasting target: wildfire risk mapping vs. full disaster risk assessment

**Core concern summarized from the framing note and parts of W3:** The reviewer interprets “risk” in the broader disaster-science sense and therefore views the paper as conflating hazard/spread forecasting with exposure and vulnerability modeling.

**Response:** We agree that in the broad disaster-risk literature, "risk" can include **hazard, exposure, and vulnerability**. Our paper uses the term in the more task-specific sense common in geospatial forecasting, where the objective is to predict the **spatial likelihood/intensity of wildfire occurrence and propagation** under environmental conditions. In other words, FireTrend is fundamentally a **wildfire hazard/spread forecasting framework**, not a complete socio-environmental risk assessment system.  

This distinction does not weaken the technical contribution; rather, it clarifies its scope. Our model is designed to forecast where wildfire activity is likely to occur or intensify, conditioned on fuels, weather, and geospatial context. It does not explicitly model population exposure, infrastructure vulnerability, or downstream damage. We will revise the wording in the paper to make this scope precise, explicitly framing FireTrend as a **hazard-oriented wildfire forecasting model**, while noting that coupling it with exposure/vulnerability layers is an important future direction for operational decision support.



### Weakness 3. Clarifying what evidence matters operationally: physical realism, decision relevance, and presentation density

**Core concern summarized from W2, W3, W4, and W5:** The reviewer wants more operational interpretation of the results, stronger case-based evidence for physical realism, and a clearer presentation of the method and its practical meaning.

**Response:** We believe the underlying issue here is less about the absence of evidence than about how the evidence is currently surfaced. The paper already evaluates predictive accuracy, ablation behavior, and physical consistency, but we agree that the current presentation makes these contributions appear denser and less decision-oriented than intended.  

More specifically, PDS and Kappa were introduced to evaluate whether predictions are not only accurate, but also **directionally and spatially plausible** under meteorological forcing. Their purpose is to move beyond pure prediction accuracy toward physically meaningful behavior. We agree, however, that this should be connected more explicitly to practical wildfire forecasting scenarios, such as better directional spread alignment under strong-wind regimes or more coherent fire-front evolution during rapidly changing events. We will therefore sharpen the exposition in the revision by: (1) simplifying the high-level method description, (2) clarifying the flow of data and losses in the main text and figure, and (3) adding more case-based interpretation of physical consistency and threshold-relevant behavior.  

In short, we do not view this as a fundamental weakness of the method, but as a presentation issue that can be improved by making the operational meaning of the results more explicit.

---
As requested by the reviewer, we add two physics metrics:

**(I) Temporal Drift Error (TDE).** It measures how well the model captures the true inter-step temporal change in wildfire risk. Let $\hat{Y}_t$ and $Y_t$ denote the predicted and ground-truth wildfire risk maps at time *t*, respectively. We define the temporal change fields as 
$\Delta \hat{Y}_t = \hat{Y}_t - \hat{Y}_{t-1}$ and $\Delta Y_t = Y_t - Y_{t-1}$. Then, TDE is computed as:

$$
TDE = \frac{1}{T-1} \sum_{t=2}^{T} \frac{1}{|\Omega|} \left\| \Delta \hat{Y}_t - \Delta Y_t \right\|_1
$$

where $\Omega$ denotes the set of spatial grid cells. A lower TDE indicates that the predicted temporal evolution more closely matches the true wildfire dynamics, and therefore reflects less prediction drift.


**(II) Temporal Consistency Score (TCS).** It measures whether the temporal smoothness of the predicted wildfire sequence is consistent with that of the ground-truth sequence. Using the same temporal change fields, it is defined as:

$$
TCS = 1 - \frac{\sum_{t=2}^{T} \left\| \Delta \hat{Y}_t - \Delta Y_t \right\|_1}{\sum_{t=2}^{T} \left\| \Delta Y_t \right\|_1 + \epsilon}
$$

where $\epsilon$ is a small constant for numerical stability. A higher TCS indicates better agreement between predicted and true temporal variation patterns, meaning that the model preserves temporal consistency without introducing excessive artificial fluctuations.



## 3. Response to Reviewer Ubjc Question and Added Experiments





## 4. Response to Reviewer gEZD Question and Added Experiments


