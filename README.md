[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on003194-blue)](https://doi.org/10.82901/nemar.on003194)

The quest for neuroprotection in Parkinson’s disease (PD) has been for new compounds to slow disease progression and stable and non-invasive biomarkers to document their benefits. Neuroepo, a new formulation of EPO with low content of sialic acid reported good results in animal model and tolerance in healthy participants and PD patients. 
In a double-blind randomized placebo (https://clinicaltrials.gov/ number NCT04110678) twenty-five PD patients were assigned randomly to Neuroepo (n=15) or placebo (n=10) groups we reported the tolerance of the drug. We recorded resting-state EEG before and six months after the administration of the drug. The qualitative analysis of the abnormalities of the EEG was evaluated by two experts using a Likert-type scale and a multivariate item response theory (MIRT) approach was employed to establish the differences between groups in the two times. The quantitative EEG (qEEG) analysis was performed at the sources looking for generators of the neural activity using software VARETA and co-registering the results using the Montreal Neurological Institute Atlas. The statistical analysis between the sources was conducted using a permutation test and later a contrast method using the surfstat software between groups and before vs after condition, with Bonferroni correction for multiple comparisons.
Here in this repository, we placed the raw EEG in BIDS format  (Pernet, C. R. et al. EEG-BIDS, an extension to the brain imaging data structure for electroencephalography. Sci. data 6, 103 (2019). For the use of VARETA the qEEG program you can use (Bosch-Bayard, J. et al. A Quantitative EEG Toolbox for the MNI Neuroinformatics Ecosystem: Normative SPM of EEG Source Spectra. Front. Neuroinform. 14, (2020).)
The EEG dataset from the different stages of processing can be requested to the authors. 

## NEMAR curation changes (2026-05-21)

BIDS validator: 2 errors + 612 warnings → 0 errors + 495 warnings. Raw `.edf` binary payloads unchanged.

### `participants.tsv`
- No cell values changed. The original file had a `sex` column with numeric `0`/`1` cells but no paired `participants.json` sidecar to document the coding, so the BIDS validator defaulted to the standard `sex` enum (`F`/`M`/`O`/`n/a`) and rejected every cell as the wrong type. The fix is in the new `participants.json` below.

### `participants.json` (new)
- Declares the `sex` column with explicit `Levels: {"0", "1"}` matching the actual cell values, plus a `Description` noting that the numeric coding key (which value corresponds to which sex) is not documented inside this dataset and must be consulted in the source publication. Why: this is the minimum-invasive override that makes the validator accept the cells as a valid coded column without inventing a `0=F` / `0=M` mapping that the dataset itself does not assert. Closes the `TSV_VALUE_INCORRECT_TYPE:sex` error.

### `sub-NEPO15/ses-Nepopre/sub-NEPO15_ses-Nepopre_scans.tsv`
- Dropped the duplicate row. The file listed `eeg/sub-NEPO15_ses-Nepopre_task-taskprenepo_eeg.edf` twice with different `age_acq_time` values (`11:04:44` and `12:00:59`); the directory contains only one EDF file. Kept the first row (earlier time) and removed the second. Why: BIDS requires the `filename` column in `scans.tsv` to be unique. Closes the `TSV_INDEX_VALUE_NOT_UNIQUE` error.

### `dataset_description.json`
- Added `DatasetType: "raw"`. Why: BIDS-validator otherwise infers a derivative-rules cascade when `DatasetType` is missing alongside `GeneratedBy`, producing spurious warnings.
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "https://github.com/nemar-org/nemar-cli"}]`. Why: records the NEMAR rehost step in the dataset's provenance chain.
- Bumped `BIDSVersion` `1.2.0` → `1.8.0`. Why: the previous value was below the validator's recognized-version floor.

### `events.json` (new, inheriting root sidecar)
- Created at the dataset root to document every column of every per-recording `_events.tsv` (none of which had a paired `_events.json` before). Columns: `onset` (`Units: "s"`), `duration` (`Units: "s"`), `trial_type`, `value`, `sample`. Why: closes 87 `TSV_ADDITIONAL_COLUMNS_UNDEFINED` warnings (29 each for `value`, `sample`, and `trial_type`) via BIDS inheritance, in one file.
- `trial_type` declared as free-form (no `Levels` enum). Why: the 29 events.tsv files contain 12 distinct trial-type labels (mostly Spanish: `ojos cerrados`, `ojos abiertos`, `hiperventilacion 1`–`4`, `recuperacion`, `fotoestimulacion`; plus English `photic stimulation`, `discontinuity`; plus two variant spellings `HPVOJOS ABIERTOS` and `HEPERVENTILACION OJOS ABIERTOS`). Locking the column to an enum would reject the variant spellings; a free-form string column is BIDS-compliant and preserves the original labels untouched.
- `value` declared as free-form numeric. The observed codes are listed in the column's `Description`.

### `scans.json` (new, inheriting root sidecar)
- Created at the dataset root to document the `age_acq_time` column used by every `sub-*_ses-*_scans.tsv`. Why: closes 29 `TSV_ADDITIONAL_COLUMNS_UNDEFINED:age_acq_time` warnings. The column name itself is non-canonical (BIDS prefers `acq_time` for full ISO 8601 timestamps) but the cells only carry `HH:MM:SS` time-of-day values, so renaming would be a semantic change beyond mechanical scope; the column is documented in place instead.

### `sub-*/ses-*/eeg/sub-*_ses-*_task-*_eeg.json` (29 per-recording sidecars)
- Renamed the key `MiscChannelCount` → `MISCChannelCount`. Why: the previous spelling is not BIDS-canonical (BIDS requires all-uppercase `MISC`), so the validator did not recognise the field and continued to warn that `MISCChannelCount` was missing. No value change — the field stays at `0`. Closes the 29 `SIDECAR_KEY_RECOMMENDED:MISCChannelCount` warnings.
