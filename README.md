[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004022-blue)](https://doi.org/10.82901/nemar.on004022)

This dataset consists of raw 18-channel EEG and functional near infrareds(fNIRS) from 7 human paticipants with orthopedic Impairment during motor imagery(MI).  The participants performed a series of MI-related trials across three sessions. These sessions comprised 40 trials, of which four different MI tasks were presented in random order (e.g., Reach → Twist → Lift → Reach → Grasp → Grasp → Twist → Reach → Lift → Reach). Each trial began with 3 s of fixation cross. The monitor then displayed a 4 s visual cue, followed by 3 s of letters indicating the ready state with a gray screen to eliminate the afterimage. The participants were then instructed to perform the imaginary movement for 5 s in the given order.

## NEMAR curation changes

Changes applied to the OpenNeuro source during the NEMAR rehost. The BIDS validator goes from 28 errors + 1010 warnings to 0 errors + (residual warnings, see below). None of the raw `.set` / `.fdt` (EEG) or `.mat` (fNIRS) binary payloads are modified; every change is to a text sidecar, a filename, or a newly-added sidecar / events file.

**Dataset description (`dataset_description.json`)**
- `BIDSVersion` is set to `1.11.1`, the version the current validator checks against.
- `DatasetType: "raw"` was already present and is left as is.
- `GeneratedBy` is absent, matching the source. Nothing is added there.

**EEG electrode tables (`sub-NNN/eeg/sub-NNN_electrodes.tsv`, 7 files, one per subject)**
- The source had three per-recording files named `sub-NNN_task-motorimagery_run-N_electrodes.tsv`, and the three files for a given subject were byte-identical to each other. BIDS-EEG places electrode positions at the subject level rather than per recording, and the `electrodes.tsv` filename pattern does not permit `task-` or `run-` entities, so the three copies were redundant and named in a way the validator rejects. The run-1 copy is renamed to `sub-NNN_electrodes.tsv` (dropping the `task-` and `run-` entities) and the run-2 and run-3 copies are deleted. The 21 input files collapse to 7 output files, with original cell contents preserved unchanged.

**EEG coordinate-system sidecars (`sub-NNN/eeg/sub-NNN_coordsystem.json`, 7 files, one per subject)**
- Every EEG `electrodes.tsv` was missing its required paired `coordsystem.json`. A single subject-level coordsystem file is added for each of the 7 subjects, matching the consolidated electrodes file. The source dataset does not document the electrode coordinate system upstream, so `EEGCoordinateSystem` is set to `"Other"`, `EEGCoordinateUnits` to `"m"` (the BIDS default), and `EEGCoordinateSystemDescription` is added with text explicitly noting that the coordinate system is not specified upstream and that the electrode positions in the paired `electrodes.tsv` were preserved unchanged. `EEGCoordinateSystemDescription` is required whenever `EEGCoordinateSystem` is `"Other"`, so it is filled in to satisfy that dependency rather than left blank.

**Event sidecar (`task-motorimagery_events.json`)**
- The source dataset did not publish an events sidecar. A new dataset-level sidecar is added describing the four columns of the per-recording `events.tsv` tables (see next bullet): `onset`, `duration`, `sample`, `value`.
- The `value` column carries verbatim trigger labels from the source `.set` files' `EEG.event` struct (`boundary`, `S 1`, `S 2`, … `S 10`, `condition 12`). The sidecar lists every label observed in the data as a `Levels` entry. The source dataset does not publish a code-to-meaning mapping for the per-trial `S N` codes, so each `Levels` entry only records what is verifiable: which label, how often it appears per recording, and that the meaning is not documented in the source dataset. The README's documented trial structure (3 s fixation, 4 s cue, 3 s ready, 5 s imagery; 40 trials per session; four MI tasks Reach / Grasp / Lift / Twist) is referenced in the column description so users have the experimental context.

**Events tables (`sub-NNN/eeg/sub-NNN_task-motorimagery_run-N_events.tsv`, 21 files, one per recording)**
- The source dataset shipped no `events.tsv` files even though the `.set` files carry a populated `EEG.event` struct (243 events per recording across all 21 recordings). The events are extracted from each `.set` file and written as a BIDS-spec `events.tsv` next to the corresponding `_eeg.set`. Columns: `onset` (seconds from recording start), `duration` (`n/a` — the source stamps a 1-sample duration on every trigger which is not a meaningful duration), `sample` (sample index at the declared 500 Hz sampling rate), `value` (verbatim trigger label).
- Per-recording event totals are uniform across all 21 files: 1 `boundary`, 1 `S 1`, 40 each of `S 2` / `S 7` / `S 8` / `S 9` / `S 10`, 10 each of `S 3` / `S 4` / `S 5` / `S 6`, and 1 `condition 12` — totalling 243 events. The source `.set` payloads are not modified by this extraction.

**fNIRS electrode tables (`sub-NNN/fnirs/sub-NNN_electrodes.tsv`, 7 files, one per subject)**
- The same consolidation as the EEG side: 21 per-recording files that were byte-identical within each subject are reduced to 7 subject-level files by keeping the run-1 copy under the subject-level filename and deleting the run-2 and run-3 duplicates. Original content unchanged.

**Validation-ignore list (`.bidsignore`)**
- Patterns are added so that the entire fNIRS modality directory is hidden from the validator. The fNIRS data in this dataset is stored as `.mat` files, but BIDS-fNIRS expects `.snirf`, so the fNIRS files do not match any BIDS-fNIRS filename rule and the validator was emitting one "file not included in schema" error per `sub-NN/fnirs/` directory. Ignoring the directory is the mechanical choice; converting `.mat` to `.snirf` would touch binary payloads, which is outside what this curation pass does.

**Left untouched**
- The fNIRS modality is not validated against BIDS-fNIRS because the data is `.mat` rather than `.snirf`. Converting the payloads to `.snirf` would require parsing the lab's `.mat` layout and reconstructing optode geometry, which goes beyond a metadata-only cleanup.

**Remaining warnings**
- The residual warnings are all "recommended but missing" fields: `GeneratedBy` on `dataset_description.json` and a long tail of equipment, cap, and filter fields that need information from the study, lab, or hardware that is not in the dataset. They are left blank rather than filled with guesses.
