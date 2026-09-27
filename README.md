# Persian A1-B1 vocab trainer

A free vocabulary trainer for Persian, A1 through B1: 2000 words with short
English glosses, romanisations and example sentences, plus 60 short reading
passages with comprehension questions. Every word, sentence, passage
sentence and script unit has recorded audio.

**Live:** https://bannerless-studio.github.io/persian/

**Scope note:** this app gives the vocabulary base for B1: words, glosses
and example sentences. The full B1 exam also needs grammar, reading,
writing and speaking practice, which this app does not teach.

## Using the trainer

- **Script primer.** An "الفبا" stage runs before A1 and teaches the
  Perso-Arabic alphabet in 33 units, with symbol-to-sound, recognition and
  word-reading items. It's skippable with "I can read it" and reversible
  later from Progress. Every script unit has a recorded audio clip, so the
  primer plays even on devices with no Persian voice.
- **Today** runs one daily session: review, learn new words, listen, recall,
  sentence practice, and a reading passage when one is due. Each stage skips
  itself when there is not enough material for it yet.
- **Words** lets you browse and search the word list, and drill any set on
  demand.
- **Test** has a placement test (to skip words you already know) plus free
  tests.
- **Progress** shows your stats and lets you export, import, or reset your
  progress.
- Question types: hearing a word and picking its meaning, reading a word and
  picking its meaning, seeing a meaning and picking the word, typing the word
  from its meaning, and filling a gap in a sentence. Typing is not
  case-sensitive; harakat, tatweel and the ZWNJ pseudo-space are folded on
  both sides, so typing میروم still matches می‌روم, and Arabic keyboard
  variants of kaf/yeh (ك/ي) always equal the Persian forms (ک/ی).
- **Reading passages (Read tab, inside Today):** 20 short texts each at A1,
  A2 and B1, 286 comprehension questions in all. A level's passages unlock
  once you've learned 70% of that level's words. Tap any word in a passage
  for its gloss, including inflected forms. Missed comprehension questions
  feed the words back into review. A passage's spaced re-read (after 7 days)
  becomes a listening pass once every sentence has audio: the text stays
  hidden and about half the questions are audio-only.
- **Offline:** the app is a single page with a service worker, so once
  loaded it keeps working offline and loads instantly on repeat visits.
- **Progress export/import:** the Progress tab can export your progress as
  text and import it back (for example, to move to a new device). Progress
  is otherwise kept only in this browser's local storage.

## Script and display

Persian is written right to left. English glosses and romanisation stay
left to right. The font is [Vazirmatn](https://fonts.google.com/specimen/Vazirmatn)
(OFL), loaded from Google Fonts, with Noto Naskh Arabic and the system
sans-serif as fallbacks.

Words display with their usual spelling, including the zero-width
non-joiner (`می‌روم`, `کتاب‌ها`); Arabic yeh/kaf are shown as Persian
yeh/kaf, and the verbal prefix is joined with a ZWNJ (`می روم` shown as
`می‌روم`). Matching for search and typing ignores these display-only
differences.

**Romanisation** (`pron`) uses one scheme, based on Wiktionary's Iranian
Persian reading: â is the long a; a, e and o are short vowels; i and u are
long vowels; kh, sh, ch and zh are digraphs; q stands for ق and gh for غ;
an apostrophe marks ع and ء. 1,980 of 2000 words have a romanisation (the
rest are listed in `tools/REPORT.md`).

## Data

This is a static data pack for a language-agnostic vocab trainer (`key:
"fa"`). It's built from the shared
[`vocab-engine`](https://github.com/Bannerless-Studio/vocab-engine) (the UI
and drill logic, included here as a git submodule at `engine/`) plus this
repo's Persian data and Persian-specific pack-builder rules
(`engine/tools/packbuilder/langs/fa.py`; summarised in `tools/README.md`).

**Data quality.** A 60-word stratified sample (seed 41) has 60/60 correct
primary senses and parts of speech. Sentence-link samples, checked by hand:

| Sample | Wrong links | Link accuracy |
|---|---|---|
| Seed 42, 90 sentences | 7 of 456 | 98.5% |
| Seed 43, 60 sentences | 9 of 324 | 97.2% |
| Seed 44, 60 sentences, after the QA fix round | 4 of 322 | 98.8% |

Tatoeba has too few usable Persian sentences for many words, so **1,211 of
the 3,031 example sentences were written for this pack** (marked `"src":
"gen"` in `pack/sentences.json`, listed in `tools/generated_sentences.tsv`).
The other 1,820 come from Tatoeba, and 447 words have only written
sentences. Written sentences are machine-written and reviewed, but not by a
native Persian speaker. Known residuals (link accuracy edge cases,
romanisation gaps, and more) are tracked in `TODO.md`.

The 60 reading passages (`pack/passages.json`) were written for this pack
(`"src": "gen"`, source `tools/passages_src.json`), enforcing in-pack word
coverage of at least 95% at A1/A2 and 93% at B1, with a level budget on how
many higher-level words each passage may use. All 60 hit full or near-full
coverage; the one exception is فسنجان (fesenjan) in a B1 recipe passage,
which no pack word names. They are machine-written by Claude, checked by an
automated QA pass and two rounds of manual/external QA fixes, but have not
had a native-speaker review. Per-passage numbers are in
`tools/REPORT_passages.md`.

**Audio.** Tatoeba has no Persian audio recordings, so every word, sentence,
passage sentence and script unit has a recorded Piper (`ganji_adabi`) clip
under `audio/`, with the browser's fa-IR voice as fallback when a clip is
missing. See `tools/README.md` for the audio pipeline.

Content policy keeps sexual content, violence, weapons, death wishes,
illicit-drug references and abuse out of A1/A2 sentences; rape/sexual-abuse
and suicide/self-harm sentences are dropped at every level. Detail and
counts are in `tools/README.md` and `TODO.md`.

### Sources and licences

| Data | Source | Licence | Used for |
|---|---|---|---|
| Spoken/subtitle frequency | [hermitdave/FrequencyWords](https://github.com/hermitdave/FrequencyWords) (`fa_full.txt`, OpenSubtitles 2018) | CC-BY-SA 4.0 | word ranking |
| Written/general frequency | [`wordfreq`](https://github.com/rspeer/wordfreq) Python package | CC-BY-SA 4.0 | word ranking |
| Glosses, POS, romanisation, present stems | [kaikki.org](https://kaikki.org) Persian Wiktionary extract | CC-BY-SA 3.0 / GFDL | glosses, POS, `pron`, verb stems |
| POS tagging / lemmatisation (build time only) | [Stanza](https://stanfordnlp.github.io/stanza/) 1.14.0 (Apache-2.0), fa default model trained on UD Persian-Seraji | model data CC BY-SA 4.0 | corpus POS and lemmas, sentence links. No model files ship. |
| Example sentences | [Tatoeba](https://tatoeba.org) `pes_sentences_detailed.tsv` | CC-BY 2.0 FR | sentence text (contributors in `pack/attribution.json`) |
| Sentence translations | Tatoeba `eng_sentences.tsv` + `pes-eng_links.tsv` | CC-BY 2.0 FR | English translations |
| Written sentences | `tools/generated_sentences.tsv`, written for this pack | same as this repo | 1,211 sentences marked `"src": "gen"` |
| Audio | Piper TTS (`piper-tts`, MIT), `fa_IR-ganji_adabi-medium` voice | voice model licence, see `engine/docs/AUDIO.md` | recorded word/sentence/passage/script-unit audio |
| Font | Vazirmatn via Google Fonts | SIL OFL 1.1 | display only |

Licence: code MIT, pack data CC BY-SA 4.0, see LICENSE.

No graded Persian word list is used or shipped.

## Level bands

Words are ranked A1/A2/B1 by a blended frequency score across the subtitle
list and `wordfreq` (words seen fewer than 3 times in the tagged corpus are
dropped, removing subtitle fragments and names), with a forced A1 core
(days, Iranian months, seasons, numbers, colours, greetings, pronouns,
question words, core prepositions and conjunctions). This is a reproducible
proxy for CEFR level, not an official classification.

## Rebuild and publish

See `CLAUDE.md` for the pinned rebuild/check/audio commands and
`tools/README.md` for what each file under `tools/` is, the audio pipeline,
and the full from-clean-checkout rebuild steps. In short: `python3
tools/build_pack.py` rebuilds the pack, `./build.sh` builds `index.html`,
and `./check.sh` must pass before every commit that touches `index.html`.
