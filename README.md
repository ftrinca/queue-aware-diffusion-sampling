# Step Smarter, Not Harder: Queue-Aware Diffusion Sampling

University project for the **Swiss Joint Master in Computer Science**, supervised by  
**Prof. Lydia Y. Chen (University of Neuchâtel)**.

This repository studies **queue-aware diffusion sampling** for online serving.
Instead of always running a fixed diffusion configuration, we use a **3-level policy**
that adapts the **sampler (DDPM vs. DDIM)** and the **number of denoising steps**
based on the **current queue length**.
The goal is to reduce end-to-end latency under load while keeping image quality as high as possible.

---

## Project idea

Diffusion models can generate high-quality images, but inference is expensive because it
requires many sequential denoising steps.
In an online system, requests arrive continuously and queues can quickly build up during busy periods.
Our approach makes inference **elastic**:
we use high-quality settings when the queue is short and automatically switch to cheaper
settings when the queue grows, preventing latency from exploding under load.

---

## Repository structure

- `report.pdf`  
  - PDF of the final report.
- `slides.pdf`  
  - Presentation slides (to be added).
- `qads.ipynb`  
  - Main Jupyter notebook containing all experiments and plots.
- `plots/`  
  - Exported figures used in the report and slides.
- `logs/`  
  - Logs and raw outputs from simulation runs.

---

## Method overview

### Offline profiling

1. **Service-time profiling**  
   Measure inference runtime for DDPM and DDIM across multiple step counts and fit a
   simple linear model: \(S(n) \approx \alpha + \beta n\)

2. **Quality profiling**  
   Build a quality grid using standard metrics (CLIPScore and FID) for selected
   combinations of sampler and step count.

### Online simulation

- Jobs arrive according to a **Poisson process** with rate $\lambda$.
- A single-server FIFO queue processes jobs one by one.
- Each job’s service time is sampled from the fitted service-time model.

### Policies

- **Fixed DDPM-100**  
  High quality, high cost.
- **Fixed DDIM-50**  
  Faster static baseline.
- **Queue-aware 3-level policy** (main contribution)  
  - short queue → DDPM, many steps  
  - medium queue → DDIM, medium steps  
  - long queue → DDIM, few steps  

For adaptive policies, quality is summarized as **effective policy quality**:
the average offline quality of the configurations actually used during a run.

---

## Key results (high level)

- Under increasing arrival rates, fixed high-quality sampling becomes unstable
  due to queue buildup.
- The **3-level queue-aware policy** strongly reduces mean end-to-end latency
  compared to fixed DDPM.
- At the same time, it avoids the large quality drop of always using very short
  sampling schedules.
- The combined offline + online evaluation pipeline makes it possible to reason
  about deployment behavior before production.

(See the report for detailed results, plots, and numerical comparisons.)

---

## How to run

### Option 1: Local machine with CUDA

1. Create and activate an environment:
   ```bash
   conda create -n queue-diffusion python=3.10 -y
   conda activate queue-diffusion
   pip install -r requirements.txt
   ````

2. Start Jupyter:

   ```bash
   jupyter lab
   ```

3. Open the notebook in `notebooks/` and run all cells.

---

### Option 2: Google Colab

1. Upload the `.ipynb` notebook to Google Drive.
2. Open it with Google Colab.
3. Adapt local file paths inside the notebook if needed.

---

## Data and pretrained models

We use publicly available datasets and pretrained diffusion checkpoints.
All datasets and models are cited in the report and listed in the bibliography.

---

## Reproducibility notes

* Fixed random seeds are used where possible to reduce variance.
* Runtime measurements may vary across hardware and environments (GPU, drivers, background load).
* Queue simulations are stochastic; results are best interpreted using averages or multiple runs.

---

## Contact

* **Flaminia Trinca** — [flaminia.trinca@students.unibe.ch](mailto:flaminia.trinca@students.unibe.ch)
* **Allizha Theiventhiram** — [allizha.theiventhiram@unine.ch](mailto:allizha.theiventhiram@unine.ch)
