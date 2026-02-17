# CSE 586 Project 1 — Project Proposal  
## Body Pose Forecasting

---

## 1. Project Title & Objective

**Title:** Body Pose Forecasting with Transformer-based Autoregressive Prediction

**Objective:** We will implement a **body pose forecasting** system that, given a sequence of past poses encoded in Vposer’s 32-D latent space, predicts future poses over **long time horizons (2–5 seconds)**. The goal is to **outperform simple physics-based baselines** (zero-velocity and constant-velocity) by using a **Transformer** architecture to capture temporal dependencies in human motion, while adapting sequence modeling from discrete tokens (as in makemore) to **continuous latent vectors**.

---

## 2. Proposed Architecture (Transformer)

**Input/Output:** Unlike Karpathy’s makemore (which predicts discrete tokens), we work in a **continuous 32-D latent space**. We therefore **switch from class prediction to continuous vector regression**: the model consumes sequences of 32-D Vposer latent vectors and outputs a 32-D vector for the next (or future) timestep(s). This avoids embedding tables and softmax; we use a final linear layer that outputs 32 dimensions.

**Components:** We will use a **Transformer** built from **self-attention blocks** to model **temporal dependencies** across the input pose sequence. The same backbone as in “Let’s build GPT” will be adapted so that each sequence element is a 32-D vector instead of a token index, with positional encoding to preserve order.

**Loss Function:** We will use **MSE or MAE** as the regression loss on the predicted vs. ground-truth latent vectors (and optionally combine with other terms). MAE may be used for robustness to large errors early in training; MSE in latent space is a natural choice for end-to-end training.

---

## 3. Long-term Forecasting Strategy

To reduce **error accumulation** over 2–5 seconds (e.g., 120 fps → hundreds of steps), we will:

- **Temporal subsampling:** Subsample the AMASS/CMU data from **120 fps to 30 fps** (keep every 4th frame) to cut the number of autoregressive steps and reduce drift.
- **Block prediction (optional):** Experiment with predicting a **block** of future latent vectors \([z_{t+1}, \ldots, z_{t+k}]\) in one forward pass instead of strictly one-step-at-a-time, then optionally use the last predicted vector to condition the next block (modified autoregression).
- **Multi-step targets:** Train to predict **k steps ahead** (e.g., 1–2 seconds) in addition to next-step prediction, to encourage the model to learn longer-horizon structure.

---

## 4. Evaluation Plan

**Baselines:** We will compare against:
- **Zero-velocity:** repeat the last observed pose for all future frames [Martinez et al.].
- **Constant-velocity:** extrapolate using velocity from the last two frames.

**Primary metric:** **Mean Per Joint Position Error (MPJPE)** — mean Euclidean distance between predicted and ground-truth 3D joint positions after decoding latent vectors to SMPL and computing joint positions. We will report MPJPE at multiple horizons (e.g., 80 ms, 320 ms, …, 1 s, 2 s, 3 s, 4 s, 5 s from the end of the input).

**Protocol:** Non-overlapping train/test (and validation) splits on AMASS; fixed generic body shape; no global rotation/translation in the latent space, so alignment is straightforward.

---

## 5. Timeline (Milestones)

| Date       | Milestone                                      |
|-----------|-------------------------------------------------|
| **Feb 16** | Submit 1-page written project proposal          |
| **Feb 25** | Submit 3–5 minute narrated video progress report |
| **Mar 18** | Submit 5–7 min video, written report, and code (zip) |

---

*References: Course handout (CSE 586 Spring 2026); Martinez et al. (CVPR 2017); Karpathy makemore/Zero to Hero; Vposer/SMPL/AMASS as specified in the project description.*
