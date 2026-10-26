# How much do three slide levels repeat each other?

A pilot measurement on public 3D prostate biopsies.

A pathologist usually reads a prostate needle biopsy from a few closely spaced 2D levels. 3D pathology can follow the same tissue continuously through depth. This project asks: **if you follow one piece of a biopsy down through its depth, how quickly does it change, and how much new information does each extra level add at the spacings labs actually use?**

**Short answer, from 12 biopsies (3 patients):** three levels cut 20 µm apart together represent **at most about 1.4 independent views** of gland structure (90% range 1.36 to 1.48), and cover about 40 µm of a core roughly 1 mm thick. Spacing them 40 µm apart raises that to at most about 2. Closely spaced levels are often cut on purpose, to confirm a focus and keep sections for stains; the question here is how much of the biopsy's depth they represent.

The two-page write-up is in [`report/Tubridy_three_slide_levels_pilot.pdf`](report/Tubridy_three_slide_levels_pilot.pdf).

![Same tissue at three depths](report/fig1_tissue_depth.png)

*The same 1 mm of a low-grade biopsy at three depths, detected lumens in blue. Below: gland space through all 33 levels; the grey band is the span three levels 20 µm apart would cover.*

![Variogram and independent looks](report/fig2_variogram_ladder.png)

## Results

| | Result (90% range from resampling whole biopsies) |
|---|---|
| Measurement noise (variogram nugget) | about 3% of the variation (1 to 5%) |
| Independent looks, 3 levels 5 µm apart | at most 1.11 |
| Independent looks, 3 levels 20 µm apart | **at most 1.42 (1.36 to 1.48)** |
| Independent looks, 3 levels 40 µm apart | at most 1.97 |
| Low grade versus higher grade (20 µm) | 1.43 (1.37 to 1.49) versus 1.41 (1.31 to 1.51): no detectable difference |

**What this shows:** closely spaced levels largely repeat one view of gland architecture, while that architecture keeps changing over depths they never reach. This fits TRICARE's own tuning, where the best depth context was a pair of neighbouring levels 60 µm apart (Gao et al.).

**What it doesn't show:** how often 2D misses or misgrades cancer. Gland space (lumen fraction) is a stand-in for gland architecture, not a grade, and no pathologist read these images for this analysis. Pathologists look for the worst feature anywhere, and a small focus can appear and vanish over shorter distances than average gland space changes; for those features the issue is how little depth closely spaced levels cover.

## Method in brief

1. **Data:** the public TRICARE prostate development data (Gao et al., *Nature Biomedical Engineering*, 2026; [Zenodo 10.5281/zenodo.20052262](https://doi.org/10.5281/zenodo.20052262)). 12 complete open-top light-sheet stacks of biopsy-sized cores cut ex vivo from prostatectomy specimens, imaged at 0.44 µm voxels and downsampled 2x to about 0.88 µm, with 3.5 to 4 µm axial resolution. Each stack has 33 virtual levels about 5 µm apart (160 µm of depth).
2. **Measurement:** gland space in 1 mm windows along each core, at every level, from colour thresholds. A white hole counts as a lumen only if a one-cell-wide ring around it is lined by nuclei; narrow openings at the core's edge are sealed so edge glands don't drop in and out of the count.
3. **Statistics:** a variogram pooled over 108 windows; independent looks from the pairwise correlations of three levels; a model-free upper limit (the variogram is still rising at 80 µm, so a fitted correlation length isn't reliable); cluster bootstrap over biopsies for uncertainty.

## Limitations

- A proxy measured with simple thresholds. Some step changes in single windows look like the segmentation flipping, which makes levels look less alike and so pushes the upper limit up, not down.
- Measured in 1 mm windows; smaller windows, which average less, are untested and may make levels look less alike.
- Only 160 µm of depth. With 3.5 to 4 µm axial resolution, levels 5 µm apart share some signal, so the smallest gaps look more alike than they are; the 20 µm result is little affected. Distances assume about 0.88 µm voxels.
- 12 biopsies from 3 patients, cut from prostatectomy specimens, from one lab's imaging pipeline. The bootstrap resamples cores, which come from only 3 patients, so the true uncertainty is wider than shown.

## Files

| File | What it is |
|---|---|
| `1_MEASURE_run_first.ipynb` | Loads every slice, finds tissue, nuclei and lumens, measures gland space per 1 mm window. Writes `outputs/measurements.csv`. |
| `2_ANALYSE_run_second.ipynb` | Variogram, noise, the upper limit on independent looks, bootstrap ranges, grade comparison. |
| `outputs/` | The measurements table and figures produced by the notebooks. |
| `report/` | The two-page write-up and its figures. |

Each step is documented before its code, and its result is described after it.

## How to reproduce

1. Download `prostate_development_cohort_part_001.zip` (about 7 GB) from [Zenodo record 20052262](https://doi.org/10.5281/zenodo.20052262) and put it in the folder **above** this repository (or change `ZIP_PATH` in step 0 of notebook 1).
2. `pip install -r requirements.txt`
3. Run `1_MEASURE_run_first.ipynb` (15 to 25 minutes; it pulls the 12 cores out of the zip into `slices/`), then `2_ANALYSE_run_second.ipynb`.

## How this was built

Python (scikit-image, SciPy, pandas, Matplotlib) in Jupyter notebooks, written with an AI coding assistant. The choices and checks are mine, and I can walk through any part of it.

## Data, licence and attribution

- **Data:** Gao G. et al., TRICARE dataset, Zenodo, doi:10.5281/zenodo.20052262, licensed CC BY-NC 4.0. The images are not included here. `outputs/measurements.csv` is derived from that data and is shared under the same CC BY-NC 4.0 terms.
- **Code:** MIT licence (see `LICENSE`).
- This is an independent, non-commercial analysis. I have no affiliation with the authors of the data or with Alpenglow Biosciences.

## References

1. Koyuncu C. et al. Visual assessment of 2-dimensional levels within 3-dimensional pathology data sets of prostate needle biopsies reveals substantial spatial heterogeneity. *Lab Invest* 103, 100265 (2023).
2. Gao G. et al. Deep-learning triage of three-dimensional pathology datasets for comprehensive and efficient pathologist assessments. *Nat Biomed Eng* (2026). Preprint: bioRxiv doi:10.1101/2025.07.20.665804.
3. Mattaino R. et al. Evaluation of a limited number of histologic levels in prostate needle core biopsies. *Int J Surg Pathol* 34(3), 669 (2026).
4. Gundersen H.J.G., Jensen E.B. The efficiency of systematic sampling in stereology and its prediction. *J Microsc* 147, 229 to 263 (1987).

Senan Tubridy · senantubridy@gmail.com
