# HG4052 Assignment 1: Does Singapore-Specific Adaptation Improve ASR of Singapore English Discourse Particles?

This repository contains the code, data and results for Assignment 1 of **HG4052** at Nanyang Technological University (NTU). The project compares a general-purpose speech recogniser with a Singapore English-adapted one on conversational Singapore English, with a focus on discourse particles (*oh, ah, lah, leh, lor, meh*).

**Research question:** Does Singapore-specific adaptation improve automatic speech recognition (ASR) of Singapore English, particularly its discourse particles?

| | Model | Description |
|---|-------|-------------|
| **System A** | [`openai/whisper-small`](https://huggingface.co/openai/whisper-small) | General-purpose Whisper model |
| **System B** | [`ivabojic/whisper-small-sing2eng-transcribe`](https://huggingface.co/ivabojic/whisper-small-sing2eng-transcribe) | Whisper-small variant fine-tuned on Singapore English |

Both models are used as published, with no fine-tuning and no changes. They run locally on the Colab machine (no closed APIs).

## Repository Contents

| File | Description |
|------|-------------|
| `disc_part_recog.ipynb` | Complete, reproducible pipeline: data checks, transcription with both systems, text normalisation, WER, particle analysis, tables and charts |
| `data.zip` | Test set: `metadata.csv` (columns: `file`, `speaker`, `condition`, `reference`) and a `test_set/` folder of audio files |
| `results_with_normalised_text.csv` | Per-utterance output: references, both systems' raw transcripts, and the normalised versions used for scoring |
| `DATA_LICENCE_NOTICE.txt` | Attribution notice required by the Singapore Open Data Licence v1.0 |
| `README.md` | This file |

## Data

- **Source:** Part 3 (conversational speech) of the [IMDA National Speech Corpus](https://www.imda.gov.sg/how-we-can-help/national-speech-corpus) (NSC).
- **Speakers:** 7 Malay speakers. Six form pairs; one speaker was sampled individually because their partner's utterances were very brief.
- **Task:** Segments were drawn from a Spot-the-Difference game, where speakers identify characteristics and ask questions. This keeps the register consistent across utterances.
- **Size:** 168 utterances were extracted. The results file has 167 rows, and 162 were scored (5 had no transcription from the models).
- **Conditions:** Each utterance is labelled `particle_present` (55 utterances in the results file) or `particle_absent` (112).
- **Reference annotation marks:** `#name#` for proper names, `<word>` for code-switched words, `[particle]` for discourse particles, and `(filler)` for fillers. The marks are stripped before scoring and the words inside are kept.

## How to Reproduce

The notebook is designed for Google Colab.

1. Open `disc_part_recog.ipynb` in Google Colab.
2. Choose **Runtime → Change runtime type → T4 GPU**.
3. Upload `data.zip` to the Files panel on the left.
4. Check the licence details in Section 0 of the notebook.
5. Choose **Runtime → Run all**.

The notebook installs `jiwer` and `num2words`, and uses `pandas`, `matplotlib`, `librosa`, `torch` and `transformers` (Colab provides these). At the end it saves everything to `asr_results.zip`, including `results.csv`, the WER tables, the particle recognition summary and the charts.

## Method

1. **Data checks:** missing values, duplicate file IDs, unexpected condition labels, unbalanced annotation marks, and missing or unreadable audio. Problems are listed and saved, never silently dropped.
2. **Transcription:** each audio file (resampled to 16 kHz) is transcribed by both systems with identical settings (English, transcribe task, greedy decoding, no sampling), so a re-run gives the same output.
3. **Normalisation:** references have annotation marks removed. Then references and both outputs are cleaned identically: lowercased, contractions spelled out, digits and symbols turned into words, and punctuation stripped. Particle spellings are not changed.
4. **Word Error Rate:** WER = (S + D + I) / N, calculated overall, by condition and by speaker with `jiwer`.
5. **Particle recognition:** each target particle in the references is aligned with the system output and classed as *recognised* (exact match), *deleted* or *substituted*. This is a strict spelling match.

## Results

### Overall WER

| System | Utterances | Reference words | Substitutions | Deletions | Insertions | WER |
|--------|-----------:|----------------:|--------------:|----------:|-----------:|----:|
| A (whisper-small) | 162 | 1052 | 252 | 111 | 59 | **40.11%** |
| B (whisper-small-sing2eng) | 162 | 1052 | 129 | 48 | 63 | **22.81%** |

### WER by condition

| System | Particle present | Particle absent |
|--------|-----------------:|----------------:|
| A | 51.18% | 34.83% |
| B | 24.71% | 21.91% |

Both systems score a higher WER on utterances containing discourse particles. The gap between the two systems is larger for System A.

### Particle recognition rate (exact word match)

| Particle | Occurrences | System A | System B |
|----------|------------:|---------:|---------:|
| oh | 11 | 54.55% | 90.91% |
| ah | 41 | 7.32% | 70.73% |
| lah | 15 | 0.0% | 60.0% |
| lor | 1 | 0.0% | 100.0% |
| leh | 0 | n/a | n/a |
| meh | 0 | n/a | n/a |
| **All** | **68** | **13.24%** | **72.06%** |

System A recognised only *oh* and *ah* at all, while System B recognised every particle that occurred. Counts for *lor* (n = 1) and the absent *leh* and *meh* are too small to draw conclusions.

### Error analysis (from the report)

- **Particles:** System A sometimes hallucinated unrelated text, for example producing "I'm still a beginner." for "children lah". System B often substituted *ah* for *lah*.
- **Word boundaries:** Acoustic analysis in Praat suggests that fast, coarticulated speech blurs word boundaries, for example "what colour" was transcribed by System A as "Wakali".
- **Singapore English pronunciation:** Weak or unreleased final consonants and compressed consonant clusters in words like *adult* and *children* may have misled System A.
- **Names and code-switching:** Neither system reliably handled proper nouns (e.g. *Hazwan*) or code-switched Malay words (e.g. *ada*).

### Limitations

- **Possible data leakage:** System B was fine-tuned on over 2 million NSC speech samples, and the test utterances also come from the NSC. Whether the test recordings appeared in its training data is unconfirmed, so its WER may be optimistic.
- **Small sample:** The test set has 162 scored utterances from 7 Malay speakers, which limits how far the findings generalise.
- **Descriptive only:** No statistical tests are run, and particle recognition is a strict spelling match.

## Data Licence

The audio and transcripts come from the IMDA National Speech Corpus, used under the Singapore Open Data Licence v1.0.

> Contains information from IMDA - National Speech Corpus accessed on 25 September 2026 from nsc@imda.gov.sg which is made available under the terms of the Singapore Open Data Licence version 1.0 https://data.gov.sg/open-data-licence

The full text is in the appendix of the notebook, and the notice is in `DATA_LICENCE_NOTICE.txt`. The licence does not grant rights over any personal data in the dataset.

## References

- Bojic, I. (n.d.). *Whisper-small Singlish2English transcription model* [Fine-tuned automatic speech recognition model]. Hugging Face.
- Koh, J.X., Mislan, A., Khoo, K., Ang, B., Ang, W., Ng, C., Tan, Y.-Y. (2019). Building the Singapore English National Speech Corpus. *Proc. Interspeech 2019*, 321-325. doi: 10.21437/Interspeech.2019-1525
- Ladefoged, P., & Johnson, K. (2015). A course in phonetics (7th ed.). Cengage Learning.
- Sproat, R., & Fujimura, O. (1993). Allophonic variation in English /l/ and its implications for phonetic
implementation. Journal of Phonetics, 21(3), 291–311.

## Author

KRYS0008, Nanyang Technological University (HG4052)
