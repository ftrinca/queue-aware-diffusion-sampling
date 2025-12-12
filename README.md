# Step Smarter, Not Harder: Queue-Aware Diffusion Sampling

University project for the **Swiss Joint Master in Computer Science**, supervised by **Prof. Lydia Y. Chen (University of Neuchâtel)**.

This repository studies **queue-aware diffusion sampling** for online serving: instead of always running a fixed diffusion configuration, we use a **3-level policy** that adapts **sampler (DDPM vs. DDIM)** and **number of denoising steps** based on the **current queue length**. The goal is to reduce end-to-end latency under load while keeping image quality as high as possible.

---

## Project idea (one paragraph)

Diffusion models can generate high-quality images, but inference is expensive because it requires many sequential denoising steps. In an online system, requests arrive over time and build up a queue during busy periods. Our approach makes inference **elastic**: use higher-quality settings when the queue is short, and switch to cheaper settings when the queue grows to know keep latency under control.

---

## What’s in this repo

- `report/`  
  - LaTeX source + compiled PDF of the report.
- `slides/`  
  - Presentation slides (to be added).
- `notebooks/`  
  - Main `.ipynb` notebook with all experiments and plots.
- `plots/`  
  - Exported figures used in the report/slides.
- `logs/`  
  - Logs / raw outputs from runs.
  - 
---

## Method overview

### Offline profiling
1. **Service-time profiling:** measure runtime for DDPM and DDIM across multiple step counts and fit a simple model  
   $$
   S(n) \approx \alpha + \beta n
   $$
2. **Quality profiling:** build a quality grid (e.g., CLIPScore and FID) for selected (sampler, steps) settings.

### Online simulation
- Jobs arrive as a **Poisson process** with rate $\lambda$.
- A single-server FIFO queue processes jobs one-by-one.
- Each job’s service time is sampled from the fitted service-time model.

### Policies
- **Fixed DDPM-100** (slow, high quality)
- **Fixed DDIM-50** (fast baseline)
- **Queue-aware 3-level policy** (main contribution):  
  - if queue is short → DDPM, many steps  
  - if queue is medium → DDIM, medium steps  
  - if queue is long → DDIM, few steps

Quality for adaptive policies is summarized as **effective policy quality**: average offline quality of the configurations actually used during a run.

---

## Key results (high level)

- Under increasing arrival rates, fixed high-quality sampling can become unstable (queues grow quickly).
- The **3-level queue-aware policy** reduces mean end-to-end latency substantially compared to fixed DDPM, while avoiding the sharp quality drop of always using very few steps.
- The combined offline + online evaluation pipeline makes it easier to reason about deployment policies before production.

(Details, plots, and exact numbers are in the report.)

---

## How to run

1. Create an environment and install dependencies (example with conda):
   ```bash
   conda create -n queue-diffusion python=3.10 -y
   conda activate queue-diffusion
   pip install -r requirements.txt
    ````

2. Start Jupyter:

   ```bash
   jupyter lab
   ```
3. Open `notebooks/<your_notebook_name>.ipynb` and run cells.

---

## Data and pretrained models

We use publicly available datasets and pretrained diffusion checkpoints (see the report for citations).

---

## Reproducibility notes

* We use fixed random seeds where possible to reduce variance across runs.
* Runtime measurements can still vary across hardware and environment (GPU, drivers, background load).
* Queue simulation uses stochastic arrivals; results are best interpreted from multiple runs or averaged summaries (as in the report).

---

## Contact

* Flaminia Trinca — [flaminia.trinca@students.unibe.ch](mailto:flaminia.trinca@students.unibe.ch)
* Allizha Theiventhiram — [allizha.theiventhiram@unine.ch](mailto:allizha.theiventhiram@unine.ch)
