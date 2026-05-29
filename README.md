[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004022-blue)](https://doi.org/10.82901/nemar.on004022)

This dataset consists of raw 18-channel EEG and functional near infrareds(fNIRS) from 7 human paticipants with orthopedic Impairment during motor imagery(MI).  The participants performed a series of MI-related trials across three sessions. These sessions comprised 40 trials, of which four different MI tasks were presented in random order (e.g., Reach → Twist → Lift → Reach → Grasp → Grasp → Twist → Reach → Lift → Reach). Each trial began with 3 s of fixation cross. The monitor then displayed a 4 s visual cue, followed by 3 s of letters indicating the ready state with a gray screen to eliminate the afterimage. The participants were then instructed to perform the imaginary movement for 5 s in the given order.

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 28 errors + 1010 warnings to 0 errors + 1024 warnings. None of the raw `.set`/`.fdt` (EEG) or `.mat` (fNIRS) binary payloads were modified; every change is to a text sidecar or to filenames.

**EEG electrode tables (`sub-NNN/eeg/sub-NNN_electrodes.tsv`, 7 files, one per subject)**
- Each subject originally had three per-recording files named `sub-NNN_task-motorimagery_run-N_electrodes.tsv`, and the three files for a given subject were byte-identical to each other. BIDS-EEG places electrode positions at the subject level rather than per recording, and the `electrodes.tsv` filename pattern does not permit `task-` or `run-` entities, so the three copies were redundant and named in a way the validator rejects. The run-1 copy was renamed to `sub-NNN_electrodes.tsv` (dropping the `task-` and `run-` entities) and the run-2 and run-3 copies were deleted. The 21 input files collapsed to 7 output files, with original cell contents preserved unchanged.

**EEG coordinate-system sidecars (`sub-NNN/eeg/sub-NNN_coordsystem.json`, 7 files, one per subject)**
- Every EEG `electrodes.tsv` was missing its required paired `coordsystem.json`. A single subject-level coordsystem file was added for each of the 7 subjects, matching the consolidated electrodes file. The source dataset does not document the electrode coordinate system upstream, so `EEGCoordinateSystem` was set to `"Other"`, `EEGCoordinateUnits` to `"m"` (the BIDS default), and `EEGCoordinateSystemDescription` was added with text explicitly noting that the coordinate system is not specified upstream and that the electrode positions in the paired `electrodes.tsv` were preserved unchanged. The `EEGCoordinateSystemDescription` field is required whenever `EEGCoordinateSystem` is `"Other"`, so it was filled in to satisfy that dependency rather than left blank.

**fNIRS electrode tables (`sub-NNN/fnirs/sub-NNN_electrodes.tsv`, 7 files, one per subject)**
- The same consolidation as the EEG side: 21 per-recording files that were byte-identical within each subject were reduced to 7 subject-level files by keeping the run-1 copy under the subject-level filename and deleting the run-2 and run-3 duplicates. Original content unchanged.

**`.bidsignore`**
- Patterns were added so that the entire fNIRS modality directory is hidden from the validator. The fNIRS data in this dataset is stored as `.mat` files, but BIDS-fNIRS expects `.snirf`, so the fNIRS files do not match any BIDS-fNIRS filename rule and the validator was emitting one "file not included in schema" error per `sub-NN/fnirs/` directory. Ignoring the directory was the mechanical choice because converting `.mat` to `.snirf` would touch binary payloads, which is outside what this curation pass does.

**Dataset description (`dataset_description.json`)**
- Updated `BIDSVersion` from `1.0.2` to `1.11.1` (the version the current validator checks against).
- `DatasetType: "raw"` was already present and was left as is.
- `GeneratedBy` was left absent, exactly as the source published it — nothing was added there.

**Out of mechanical scope — left untouched**
- The fNIRS modality is not validated against BIDS-fNIRS because the data is `.mat` rather than `.snirf`. Converting the payloads to `.snirf` would require parsing the lab's `.mat` layout and reconstructing optode geometry, which goes beyond a metadata-only cleanup.

**Remaining warnings (1024) — left on purpose**
- These are all "recommended but missing" fields plus a handful of warnings for recordings that have no events table. The recommended-but-missing list includes `GeneratedBy` on `dataset_description.json` and a long tail of equipment, cap, and filter fields that need information from the study, lab, or hardware that is not in the dataset. They were left blank rather than filled with guesses.