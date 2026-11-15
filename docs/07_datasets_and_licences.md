# 7. Datasets and licences

> **BTH is a research prototype, not a medical device.**

**Nothing from any dataset is redistributed in this repository**: no raw data, no processed data, no derived
tables. Raw and processed data stay on the author's machine and are excluded from every repository. Datasets
under restricted licences are named, cited and described only.

Licences were taken from the simulator's licence register, which records where and when each was checked
(mostly 12 November 2026). "Unverified" means the register itself says the licence could not be re-read.

| Dataset | Citation | Licence | Used for | Redistributed |
|---|---|---|---|---|
| U-Limb (MHH subset) | Averta G, et al. U-Limb: a multi-modal, multi-center database on arm motion control in healthy and post-stroke conditions. *GigaScience*. 2021;10(6):giab043 | CC0 1.0 (Dataverse API) | Calibrating and validating simulated elbow and shoulder kinematics and muscle activity (healthy and post-stroke) | No |
| SciData 2025 upper-limb dataset | Lucchetti F, Bailo G, Lencioni T. A detailed kinematic and EMG dataset for upper limb and hand movement analysis in post-stroke and healthy subjects during functional daily tasks. *Scientific Data*. 2025 | CC BY 4.0 (figshare API and paper) | Validation partner for U-Limb (calibrate on one, validate on the other) | No |
| Zenodo record 3713449 | Post-stroke and healthy arm kinematics; cite per the record page (**full citation still to be completed**) | CC BY 4.0 (Zenodo API) | Kinematics validation (20 post-stroke, 5 healthy subjects) | No |
| WESAD | Schmidt P, Reiss A, Duerichen R, Marberger C, Van Laerhoven K. Introducing WESAD, a multimodal dataset for wearable stress and affect detection. *ICMI* 2018 | Scientific, non-commercial use with credit. **Unverified**: the landing page had moved and was not re-read | Testing simulated skin conductance, heart-rate variability and respiration against real healthy recordings (stress tasks, not exosuit therapy) | No. Restricted licence: no results derived from it are shown here |
| PhysioNet culm | PhysioNet culm, version 1.0.0; cite per the project page (**full citation still to be completed**) | Open Data Commons Attribution 1.0 (checked by the owner) | Hand-speed endpoint reference only | No |
| PhysioNet Long-Term AF Database (annotations only) | Petrutiu S, Sahakian AV, Swiryn S. Abrupt changes in fibrillatory wave characteristics at the termination of paroxysmal atrial fibrillation in humans. *Europace*. 2007;9:466-470. Goldberger AL, et al. PhysioBank, PhysioToolkit, and PhysioNet. *Circulation*. 2000;101(23):e215-e220 | Open Data Commons Attribution 1.0 (physionet.org, read 12 Nov 2026) | Real atrial-fibrillation beat timing for the flagged-patient variants | No |
| Lokomat dataset, Zenodo record 15054030 | Chiappini, Malerba, et al. Zenodo 15054030 | CC BY 4.0 (Zenodo API) | Summary heart-rate-variability and skin-conductance features, descriptive only (feature definitions not stated in the source) | No |
| OrthoLoad | Bergmann G, Damm P (eds.). OrthoLoad. Julius Wolff Institute, Berlin Institute of Health at Charité, 2008. Retrieved from www.OrthoLoad.com, with each file name and download date | Non-commercial research and education with credit and file name; no alteration; automated access, scraping, text and data mining and AI training prohibited without written permission | Shoulder-load range reference only. Files were downloaded manually by the owner | No. Restricted licence: no values derived from it are shown here |

## Considered but not used

| Dataset | Why not |
|---|---|
| Rehab-120 | Column definitions are undocumented; no loader is written until the authors supply them |
| UBC chronic-stroke skin-conductance data | Not available; licence unknown |
| IEEE DataPort post-stroke dataset | Requires a subscription login; never bypassed |

## Open licence questions

- **OrthoLoad "no alteration" clause.** The simulator builds per-activity profiles from the owner's OrthoLoad
  files. Whether a derived range reference counts as alteration should be confirmed with the data owners.
- **WESAD**: the licence terms should be re-read on the moved page.
- **Zenodo 3713449 and PhysioNet culm**: full citation strings are still to be completed from their record pages.
