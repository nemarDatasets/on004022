[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004022-blue)](https://doi.org/10.82901/nemar.on004022)

This dataset consists of raw 18-channel EEG and functional near infrareds(fNIRS) from 7 human paticipants with orthopedic Impairment during motor imagery(MI).  The participants performed a series of MI-related trials across three sessions. These sessions comprised 40 trials, of which four different MI tasks were presented in random order (e.g., Reach → Twist → Lift → Reach → Grasp → Grasp → Twist → Reach → Lift → Reach). Each trial began with 3 s of fixation cross. The monitor then displayed a 4 s visual cue, followed by 3 s of letters indicating the ready state with a gray screen to eliminate the afterimage. The participants were then instructed to perform the imaginary movement for 5 s in the given order.

## NEMAR curation changes (2026-05-21)

BIDS validator: 28 errors + 1010 warnings → 0 errors + 1023 warnings. Raw `.set`/`.fdt` (EEG) and `.mat` (fNIRS) binary payloads unchanged.

### `dataset_description.json`
- Bumped `BIDSVersion` `1.0.2` → `1.8.0`. Why: the previous value is below the validator's recognised-version floor.
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "https://github.com/nemar-org/nemar-cli"}]`. Why: records the NEMAR rehost step in the dataset's provenance chain. `DatasetType: "raw"` was already present.
- Removed three empty-string entries from `Funding` and dropped the empty `Acknowledgements`/`HowToAcknowledge` keys. Why: empty-string array elements and empty top-level string fields are non-canonical and noise to downstream readers.

### `sub-NNN/eeg/sub-NNN_electrodes.tsv` (7 files, one per subject)
- Consolidated from 21 per-recording electrodes files. Each subject had three byte-identical per-recording `_task-motorimagery_run-N_electrodes.tsv` files; BIDS-EEG specifies electrode positions at the subject level, not per recording, so the three were redundant. Kept the run-1 copy renamed to `sub-NNN_electrodes.tsv` (no `task-` / `run-` entity); deleted the run-2 and run-3 duplicates. Why: BIDS's `electrodes.tsv` schema does not allow `run-` in the filename. Original cell contents preserved unchanged.

### `sub-NNN/eeg/sub-NNN_coordsystem.json` (7 files, new — one per subject)
- Created at the subject level. Why: every EEG `electrodes.tsv` was missing a paired `coordsystem.json`, firing 21× `REQUIRED_COORDSYSTEM` errors. The source dataset does not document an electrode coordinate system, so the value is `EEGCoordinateSystem: "Other"` with an `EEGCoordinateSystemDescription` explicitly noting that the system is not specified upstream and that the electrode positions in the paired `electrodes.tsv` were preserved unchanged. `EEGCoordinateUnits: "m"` is the BIDS default. This closes the 21 `REQUIRED_COORDSYSTEM` errors and the dependent 21 `JSON_KEY_REQUIRED:EEGCoordinateSystemDescription` errors.

### `sub-NNN/fnirs/sub-NNN_electrodes.tsv` (7 files, one per subject)
- Same consolidation as the EEG side: 21 per-recording files (byte-identical across runs) reduced to 7 subject-level files. Original content unchanged.

### `.bidsignore`
- Pattern updated from `/sub-*/fnirs/**` to `*/fnirs`, `*/fnirs/`, `*/fnirs/**`. Why: the original anchored pattern only matched files under the directory, not the directory itself, so the validator continued to emit `NOT_INCLUDED` on each `sub-NN/fnirs/` subdirectory (7 errors). The dataset's fNIRS data is stored as `.mat` files; BIDS-fNIRS expects `.snirf`, so the entire fNIRS modality is intentionally outside the validator's purview here. Ignoring the directory rather than converting the data was the mechanical choice — binary-payload conversion is outside the curation envelope.