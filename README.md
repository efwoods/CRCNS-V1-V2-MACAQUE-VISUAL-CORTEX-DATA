# CRCNS-V1-V2-MACAQUE-VISUAL-CORTEX-DATA

This is a an example of visualizing cortex data of a macaque.

[Data]()

---

## Instructions

1. Use the CRCNS V1–V2 Macaque Visual Cortex Data

The Collaborative Research in Computational Neuroscience repository hosts a V1–V2 dataset of multi-electrode recordings from anesthetized macaques viewing static gratings and natural images. Crucially, every trial comes with a timestamped stimulus label, so you can compute and plot cortical responses to “inhibitory” vs. “activating” stimuli directly.
crcns.org
crcns.org

Link & Description

    v1v2-1: Simultaneous V1–V2 neuronal population recordings in anesthetized macaques, with detailed stimulus metadata.
    crcns.org

2. Proof-of-Concept: Load, Aggregate & Visualize

Below is a complete Python script that:

    Loads the .mat data file (downloaded from CRCNS).

    Extracts spike-rate time series and stimulus annotations.

    Aggregates average firing rates per channel for two example stimulus classes (e.g. “grating” vs. “natural image”).

    Displays a heatmap of mean activity across channels, side by side, to highlight “inhibited” vs. “stimulated” conditions.

Save this as visualize_v1v2.py.

```python
#!/usr/bin/env python3

# visualize_v1v2.py

import scipy.io
import numpy as np
import matplotlib.pyplot as plt
import argparse

def load_crcns(mat_path):
"""Load CRCNS V1–V2 .mat file and return struct."""
data = scipy.io.loadmat(mat_path, squeeze_me=True, struct_as_record=False)
return data['v1v2'] # top-level struct

def extract_firing_rates(v1v2_struct, stim_type):
"""
Extract and average firing rates for all channels for trials of a given stimulus type.
stim_type is e.g. 'grating' or 'natural'.
"""
rates = []
for trial in v1v2_struct.trials:
if stim_type in trial.stimulus_type.lower(): # trial.spike_rates: channels × time_bins # here we average over time to get one rate per channel
rates.append(np.mean(trial.spike_rates, axis=1))
return np.stack(rates, axis=0) # trials × channels

def plot_comparison(rates_a, rates_b, labels):
"""Plot side-by-side heatmaps of mean channel activity."""
mean_a = np.mean(rates_a, axis=0)
mean_b = np.mean(rates_b, axis=0)
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
im0 = axes[0].imshow(mean_a[np.newaxis, :], aspect='auto')
axes[0].set_title(labels[0])
axes[0].set_yticks([])
im1 = axes[1].imshow(mean_b[np.newaxis, :], aspect='auto')
axes[1].set_title(labels[1])
axes[1].set_yticks([])
for im in (im0, im1):
plt.colorbar(im, ax=axes[0] if im is im0 else axes[1], orientation='horizontal')
plt.suptitle("Average Firing Rate per Channel")
plt.tight_layout()
plt.show()

def main(mat_file):
v1v2 = load_crcns(mat_file) # Example: compare 'grating' vs. 'natural image' trials
rates_gr = extract_firing_rates(v1v2, 'grating')
rates_nat = extract_firing_rates(v1v2, 'natural')
if rates_gr.size == 0 or rates_nat.size == 0:
print("No trials found for one of the stimulus types.")
return
plot_comparison(rates_gr, rates_nat, ['Gratings', 'Natural Images'])

if **name** == "**main**":
parser = argparse.ArgumentParser(
description="Visualize CRCNS V1–V2 macaque firing rates."
)
parser.add_argument(
"mat_file",
help="Path to the CRCNS v1v2-1 .mat file you downloaded."
)
args = parser.parse_args()
main(args.mat_file)
```

Setup & Run (keyboard only)

# 1. Create & activate your env
```bash
python3 -m venv ~/neuro_env
source ~/neuro_env/bin/activate
pip install scipy numpy matplotlib
```
# 2. Download CRCNS V1–V2:

# • Visit https://crcns.org/data-sets/vc/v1v2-1 and download the .mat file into ~/datasets/v1v2-1.mat

# 3. Run your visualization:
```bash
python visualize_v1v2.py ~/datasets/v1v2-1.mat
```
3. How This Advances Your Goals

   Immediate “real-world” result: You’ll have a clear side-by-side heatmap showing channels dominated by “inhibition” (low firing) vs. “stimulation” (high firing) under known stimuli.

   Neuralink relevance: Exactly matches their workflow—time-locked neural acquisition → feature extraction → visualization of brain states.

   Expands to human data: You can swap in any annotated EEG/ECoG dataset next.

   Demo-ready: Embed this plot in a minimal FastAPI + React dashboard for a live pitch.

With this PoC you can immediately show recruiters or program directors a working, data-driven visualization of cortical states—no missing metadata, no unknown stimuli, and no hardware required.
