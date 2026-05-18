# Topology of Grammar — Study 1

A self-contained jsPsych 8 implementation of a cross-linguistic self-paced reading study testing whether the cognitive cost of Cinque-order violations interacts super-additively with numerical magnitude. The hypothesis is that resolution of long-distance syntactic dependencies and attentive enumeration draw on a shared pool of top-down visual indices, in the sense of Pylyshyn's FINST model and Tsotsos's Selective Tuning framework.

The study has four tasks, completed in a single session of approximately 45 minutes:

1. **Moving-window self-paced reading** in English or Bengali, manipulating the linear order of numeral and adjective modifiers within a noun phrase (canonical, hierarchically inverted, and — in Bengali — reduplicated forms);
2. **Multiple-Object Tracking** (Pylyshyn & Storm, 1988) to estimate visual indexing capacity;
3. **Brief-presentation enumeration** to locate the participant's subitising threshold;
4. **Reverse digit span** (adaptive) as a verbal-WM control.

This repository contains the experiment itself, the analysis stub, and the participant-facing chrome.

## Quick start

The experiment is a single static HTML file. To run it locally:

```bash
git clone https://github.com/<your-username>/topology-of-grammar.git
cd topology-of-grammar
python3 -m http.server 8000     # or any other static server
# open http://localhost:8000 in your browser
```

That is the entirety of the local setup. There is no build step, no database, no server-side dependency.

## Deployment

The experiment supports three deployment modes, selected automatically at run time:

### 1. JATOS

If the page is loaded inside a JATOS study, `window.jatos` is detected and results are submitted via `jatos.submitResultData()` followed by `jatos.endStudy()`. Configure a single JATOS component pointing at `index.html`. No code changes are required.

### 2. POST to a custom endpoint

If the URL includes `?endpoint=https://example.com/your-collector`, the final JSON is POSTed to that URL with `Content-Type: application/json`. The endpoint should accept a JSON body and return any 2xx status. For Prolific or SONA integrations, pass the participant ID through as well:

```
https://your-host.example/index.html?pid=PROLIFIC_PID_HERE&endpoint=https://collector.example/submit
```

The deployment block in the page captures `pid`, `PROLIFIC_PID`, `participant_id`, `study_id`, and `session_id` as URL parameters and embeds them in the final data record under `deployment`.

### 3. Browser download (default)

With no JATOS context and no `endpoint`, the final "Thank you" screen offers a Download button that produces a JSON file the participant can email to the experimenter. This mode is intended for in-person lab sessions where the experimenter saves the file directly.


## What the file does

`index.html` contains four custom jsPsych plugins, the stimulus lists for both English and Bengali, the full timeline, and the in-browser analysis tool. Sections of the file are clearly headed and runnable independently for testing.

### Stimuli

60 critical item-frames per language, normed for length, frequency, and plausibility. The item lists are at the top of the script section under `EN_ITEMS` / `BN_ITEMS`. To swap in your own stimuli, replace those arrays in place — the rest of the experiment is parameterised against them.

### Counterbalancing

Conditions are Latin-squared across participants by participant-ID parity; magnitudes are independently shuffled within each condition's item group, so each participant sees each condition at four distinct magnitudes drawn without replacement from `{2, 3, 4, 6, 8, 10}`. This independence is essential — an earlier version of the code assigned condition and magnitude with related cycles, perfectly confounding the two within-subject and making the diagnostic Condition × Magnitude interaction inestimable.

### Data structure

Every trial in the output JSON has a `phase` field that identifies its role in the session:

| Phase | Trials |
|-------|--------|
| `demographics_text`, `demographics_education`, `demographics_mc`, `language_choice` | Pre-task questionnaires |
| `practice_en` / `practice_bn` | SPR practice trials (with `_comp` versions for comprehension questions) |
| `main_en` / `main_bn` | SPR main block (with `_comp` versions) |
| `mot_practice`, `mot_main` | Multiple-Object Tracking |
| `enum_practice`, `enum_main` | Brief-presentation enumeration |
| `digit_span` | Reverse digit span |
| `debrief` | End-of-session comments |

SPR trials carry per-word reaction times in `word_rts`, with each entry recording the word index, the word string, and the inter-press interval in ms. The position of the head noun is recorded as `critical_word_index` for downstream spillover analysis.

The instruction-screen stimuli are stripped from the data export at write time to keep file sizes small; only the field `stimulus_stripped_chars` remains, recording the original length for audit purposes.

### Built-in analysis tool

The final "Thank you" screen has a `Show analysis summary` button that runs an in-browser analysis of the just-completed session and renders:

- Session header (subject ID, language, duration, total trial count);
- Demographics, formatted from the captured responses;
- A Condition × Magnitude crosstab of critical-word reading times, with row and column marginals;
- Mean per-word RT and comprehension accuracy;
- MOT hit rates by set size, plus a capacity estimate using the Hulleman-style K(2p − 1) corrected-for-guessing formula averaged across set sizes;
- Enumeration accuracy and signed mean error by N, plus an estimated subitising threshold;
- Reverse digit span performance and maximum correct length;
- A list of automatically detected data-quality flags (empty cells, low comprehension, short demographic responses, abrupt enumeration drops, MOT non-monotonicity, implausibly fast or slow reading).

The summary is also available as a copy-to-clipboard plain-text format and a `.txt` download. It is intended for quick per-participant sanity checks during data collection and as input to a full mixed-effects analysis pipeline downstream.

## What this is not

The analysis tool computes per-participant descriptives and quality flags. It does not perform the mixed-effects modelling described in the proposal — that lives in a separate R/Python pipeline that consumes the exported JSON files. Use the in-browser summary to decide whether a participant's data should be retained; use the pipeline to test hypotheses.

The 12-item stimulus list bundled with the repository is a demonstration set, not the full study. Do not use it for a confirmatory data collection without expanding to the pre-registered 60 items per language.

The experiment does not currently include a practice-block accuracy gate. Participants whose comprehension on the practice trials is below some threshold should normally be redirected back through practice; this is straightforward to add by wrapping the main block in a `conditional_function` that inspects practice accuracy.

## Browser support

Tested on recent Chrome, Firefox, Safari, and Edge. Requires:

- ES2020 JavaScript (async/await, optional chaining)
- HTML5 Canvas (for MOT and enumeration plugins)
- A keyboard (the SPR plugin uses SPACE; the response inputs use ENTER)

Bengali rendering uses Noto Sans Bengali via Google Fonts. If the lab machine is offline or the network blocks Google Fonts, the script will fall back to system-default sans-serif rendering — readable but not aesthetically ideal. For offline deployment, download the font files locally and adjust the `<link>` tag in the `<head>`.

The CDN-hosted jsPsych core and plugins are loaded from `unpkg.com`. For air-gapped deployment, replace the CDN URLs with paths to local copies of the jsPsych 8 package.

## Files in this repository

```
index.html              The experiment itself (single deployable file)
README.md               This document
LICENSE                 MIT License
```

## Citation

If you use this code in published work, please cite the proposal document and link to the repository. A `CITATION.cff` file is included to facilitate citation through GitHub's "Cite this repository" button.

## Licensing

The experimental code is released under the MIT License (see `LICENSE`). The stimulus norming and Bengali grammaticality judgements were developed at the Krea Computational Cognition Laboratory; if you use the stimulus set rather than supplying your own, please acknowledge that source.

The dependencies (`jsPsych` and its plugins) are licensed under MIT by the Joshua de Leeuw lab. They are loaded from CDN by default; no source code from those packages is included in this repository.

## Contact

For correspondence about the study design or theoretical claims, please use the address listed on the proposal document. For bug reports, code contributions, or deployment problems, please open an issue or pull request on this repository.
