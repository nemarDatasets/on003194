[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on003194-blue)](https://doi.org/10.82901/nemar.on003194)

The quest for neuroprotection in Parkinson’s disease (PD) has been for new compounds to slow disease progression and stable and non-invasive biomarkers to document their benefits. Neuroepo, a new formulation of EPO with low content of sialic acid reported good results in animal model and tolerance in healthy participants and PD patients. 
In a double-blind randomized placebo (https://clinicaltrials.gov/ number NCT04110678) twenty-five PD patients were assigned randomly to Neuroepo (n=15) or placebo (n=10) groups we reported the tolerance of the drug. We recorded resting-state EEG before and six months after the administration of the drug. The qualitative analysis of the abnormalities of the EEG was evaluated by two experts using a Likert-type scale and a multivariate item response theory (MIRT) approach was employed to establish the differences between groups in the two times. The quantitative EEG (qEEG) analysis was performed at the sources looking for generators of the neural activity using software VARETA and co-registering the results using the Montreal Neurological Institute Atlas. The statistical analysis between the sources was conducted using a permutation test and later a contrast method using the surfstat software between groups and before vs after condition, with Bonferroni correction for multiple comparisons.
Here in this repository, we placed the raw EEG in BIDS format  (Pernet, C. R. et al. EEG-BIDS, an extension to the brain imaging data structure for electroencephalography. Sci. data 6, 103 (2019). For the use of VARETA the qEEG program you can use (Bosch-Bayard, J. et al. A Quantitative EEG Toolbox for the MNI Neuroinformatics Ecosystem: Normative SPM of EEG Source Spectra. Front. Neuroinform. 14, (2020).)
The EEG dataset from the different stages of processing can be requested to the authors. 

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 2 errors + 612 warnings to 0 errors + 496 warnings. None of the raw `.edf` files were modified — every change is to a text sidecar.

**Participant table (`participants.tsv`) and its new sidecar (`participants.json`)**
- No participant data was changed. The `sex` column records each participant as a number (`0` or `1`), but there was no sidecar to explain those codes, so the validator read them against its built-in `F`/`M`/`O` list and rejected every cell. A new `participants.json` was added that declares the column with exactly the codes that appear (`0` and `1`) and notes that the dataset itself does not say which number means which sex — that key has to be looked up in the source publication. No mapping was invented.

**Acquisition times (`sub-NEPO15` scan table)**
- One scan table listed the same recording file twice, with two different times (`11:04:44` and `12:00:59`), even though only one recording file exists in that folder. The earlier (first) row was kept and the duplicate removed, since BIDS requires each file to be listed only once.

**Dataset description (`dataset_description.json`)**
- Added `DatasetType: "raw"` so the dataset is validated as raw data rather than a derivative.
- Updated `BIDSVersion` from `1.2.0` to `1.11.1` (the version the current validator checks against).
- `GeneratedBy` was left absent, exactly as the source published it — nothing was added there.

**Event tables — new shared sidecar (`events.json`) at the dataset root**
- None of the event tables had a sidecar describing their columns. A single root-level `events.json` now documents the columns they share (`onset` and `duration`, both in seconds, plus `trial_type`, `value`, and `sample`) and applies to every recording at once. The `trial_type` labels are kept as free text rather than a fixed list, because the recordings use a dozen different labels (mostly Spanish: `ojos cerrados`, `ojos abiertos`, `hiperventilacion 1`–`4`, `recuperacion`, `fotoestimulacion`, and others, including two variant spellings) and pinning them to a fixed list would reject the variants. The numeric `value` codes that appear are listed in that column's description.

**Scan tables — new shared sidecar (`scans.json`) at the dataset root**
- The scan tables use a column named `age_acq_time` that BIDS does not define, so a root-level `scans.json` describes it. The column name is non-standard (BIDS normally uses `acq_time` for full timestamps) and the cells only carry a time of day (`HH:MM:SS`), so it was documented in place rather than renamed, which would have changed its meaning.

**Recording sidecars (`_eeg.json`, all 29 recordings)**
- The misc-channel-count field was spelled `MiscChannelCount`; BIDS uses the all-uppercase `MISCChannelCount`. It was renamed (the value, `0`, was already correct) so the validator recognizes it.

**Remaining warnings (496) — left on purpose**
- These are all "recommended but missing" fields that need study, lab, or equipment details not in the dataset (for example: manufacturer, cap model, device serial number, instructions, head circumference, hardware filters, cognitive-atlas IDs). One of them is `GeneratedBy`: the source does not document what tool generated the data, so it was left absent rather than guessed. All were left blank rather than filled with invented values.
