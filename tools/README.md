# tools/ — Persian pack builder inputs (dev/agent notes)

This directory holds the data Persian owns: hand-maintained overrides, generated
sentence source text, the ezafe/audio pipeline glue, and the shim that calls the
shared builder in `engine/tools/packbuilder`, plus the reports the builder writes
back. Persian-specific linking rules (light verbs, plurals, forced closed sets, the
کش verb-stem disambiguation, content-policy tiers) are code, not data, and live in
`engine/tools/packbuilder/langs/fa.py`.

## Files

- `build_pack.py` — **glue.** Shim that runs
  `python3 -m packbuilder build --lang fa --repo .` against `engine/tools` (or
  `PACKBUILDER_PATH` if set).
- `requirements.txt` — **config.** `engine/tools/packbuilder/requirements.txt` plus
  Stanza (the fa Seraji model downloads into `.cache/stanza` on first build).
- `gloss_overrides.json` — **hand-maintained.** Gloss fixes keyed `"lemma|pos"`
  (folded keys).
- `forced_a1.txt` — **hand-maintained.** A1 core list forced into A1 (closed sets
  like days/months/numbers live in code, in `langs/fa.py`).
- `generated_sentences.tsv` — **hand-authored input.** 1,211 sentences written for
  this pack where Tatoeba had too few usable sentences; feeds the build like a
  source, tagged `"src": "gen"` in the output. Append-only: line order sets ids.
- `generated_examples.tsv` — **hand-authored input.** Hand-reviewed written examples
  used for illustration only (never for frequency); exempt from the drop-everywhere
  content filter, e.g. the one neutral خودکشی example kept after suicide/self-harm
  sentences were otherwise dropped (see TODO.md).
- `passages_src.json` — **hand-authored input.** Source text for the 60 reading
  passages (`pack/passages.json`), rebuilt with `packbuilder passages`.
- `id_map_v1.json` — **generated, frozen.** `"lemma|pos"` -> word id map; never
  hand-edit or renumber, it's what keeps learner progress across rebuilds.
- `ezafe_say.py` — **hand-maintained script.** Runs a Stanza dependency parse over
  every sentence and passage sentence to find unwritten ezafe (the -e/-ye that links
  a noun to its modifier, which Piper's espeak-ng phonemiser never voices) and writes
  `tools/audio_say.json` with the spoken-text override. Needs stanza 1.14 + the fa
  models in `.cache/stanza`, and piper-tts (for espeak). See its module docstring for
  the exact rule and spelling table, and `engine/docs/AUDIO.md` "Ezafe override pass".
- `audio_say.json` — **generated.** `{pack text: spoken text}` map from `ezafe_say.py`;
  regenerate it, don't hand-edit. Only the spoken text changes; pack text never does.
- `REPORT.md` — **generated** (manual QA-verdict section preserved across rebuilds).
  Tagging stats, word-selection funnel, romanisation coverage, level-band counts.
- `REPORT_passages.md` — **generated.** Per-passage coverage, level-budget and
  question counts for the 60 reading passages.

## Audio pipeline

Persian ships recorded audio for every word, sentence, passage sentence and script
unit (Tatoeba has no Persian recordings). Voice: Piper (`piper-tts` 1.8.0) with the
`fa_IR-ganji_adabi-medium` voice, phonemes from the espeak-ng `fa` voice bundled in
piper-tts; codec opus. Clips live under `audio/{w,s,p,x}/` (word/sentence/passage/
script-unit) with `audio/manifest.json` describing the set (voice, engine, codec,
version). Dependencies (`engine/tools/packbuilder/requirements-audio.txt`): piper-tts
1.8.0, ffmpeg with libopus on PATH.

Rebuild order:
1. `.venv/bin/python tools/ezafe_say.py` — regenerate `tools/audio_say.json` after
   any sentence/passage text change.
2. `python3 -m packbuilder audio --lang fa --repo . [--check] [--prune] [--only w,s,p,x] [--limit N]`
   — synthesize/update the opus clips and manifest. `--check` reports only; `--prune`
   removes clips no longer referenced; `--only` limits to one or more of the four
   kinds; `--limit` caps how many clips it (re)generates in one run.

Full reference: `engine/docs/AUDIO.md` (voice choice, phoneme handling, manifest
schema, the builder CLI).

## Repo layout

```
pack/                pack.json, words.json, sentences.json, passages.json,
                      attribution.json (+ generated .js)
engine/               git submodule -> vocab-engine (UI, drills, tools/packbuilder,
                      langs/fa.py = Persian rules)
audio/{w,s,p,x}/      generated Piper opus clips (word/sentence/passage/script-unit)
audio/manifest.json   generated audio manifest
tools/                see file list above
build.sh              builds index.html from pack/ + engine/
check.sh              packbuilder check + engine validator + stale-build guard
```

## Full rebuild from a clean checkout

```
git clone --recurse-submodules <this repo>
cd persian
python3 -m venv .venv && source .venv/bin/activate
pip install -r tools/requirements.txt     # the Stanza fa model downloads into .cache/stanza on first build

python3 tools/build_pack.py               # pack/*.json + tools/REPORT.md
python3 engine/tools/jsonify_pack.py pack # pack/*.js
./build.sh                                # index.html
./check.sh                                # checks + stale-build guard
```

Sources download once into `.cache/` (gitignored). The build is deterministic: two
runs from cache give byte-identical `pack/*.json`. Stanza output is cached per
sentence in `.cache/derived/`, so only the first build is slow. To build against a
vocab-engine checkout other than the submodule, set
`PACKBUILDER_PATH=../vocab-engine/tools` for `tools/build_pack.py` and `./check.sh`.
QA helpers run with `PYTHONPATH=engine/tools python3 -m packbuilder {scan,sample}
--lang fa --repo .`.

## Persian rules (see `langs/fa.py` for the code)

- **Tagging.** Stanza 1.14.0 with the fa default model (UD Persian-Seraji) tags the
  corpus. Verbs are taught as infinitives (`رفتن`), with the present stem in `alt`
  (`رو`). Colloquial subtitle forms (`میرم`, `دونستن`) count for their written forms.
- **Light verbs.** Light-verb compounds (`فکر کردن`, `دوست داشتن`, `از دست دادن`) are
  single words: 228 of them, each with at least two linked sentences. A compound
  counts when its noun or adjective is next to the light verb, or has one adverb in
  between. A short list of nouns that take a complement can also have that
  complement in between: سوار قطار شد, وارد اتاق شد, علاقه‌ای به تاریخ ندارم, سعی‌ام
  را کردم. A compound is never formed when an adjective stands between the noun and
  the verb (دوست خارجی دارم), the noun follows a numeral (دو دوست دارم), or the -ی on
  the noun makes it an object (کاری نکرده‌ام, دوستی ندارد).
- **Plurals.** Broken plurals (`افراد`, `نتایج`) link to their singular. Plural -ها
  forms are never lemmas.
- **Forced A1 closed sets.** Days, Iranian months, seasons, numbers 0-20 plus tens,
  صد/هزار/میلیون, colours, greetings, pronouns, question words, core prepositions and
  conjunctions, plus the A1 core list in `forced_a1.txt`.
- **Sentences.** 3-14 tokens; A1 allows 3, A2 needs at least 4, B1 at least 5.
  Proper nouns are never linked.
- **Link guards.** A lemma never links from inside a longer word (مهربان is not مهر,
  فردیت is not فرد). Space-written compounds (برنامه ریزی, پیش بینی, روغن کاری) link
  neither half. Known homographs never linked: دعوی, حقوق, کاری "curry", آخُر
  "manger".
- **کش disambiguation.** The present stem کش- belongs to both کشیدن "pull, smoke,
  last" and کشتن "kill"; the sentence and its English translation decide which links.
  When neither decides, the verb is left unlinked.
- **Content policy.** The shared sensitive filter applies, plus Persian-specific
  additions: illicit drugs (مواد مخدر) and abuse (سوء استفاده, آزار) join sexual
  content, violence, weapons and death wishes as kept out of A1/A2 (118 B1-only
  sentences). Rape/sexual-abuse sentences are dropped at every level (1 sentence).
  Suicide/self-harm sentences are dropped too as of engine ff88f44 (2 written
  sentences); خودکشی keeps one neutral written example in
  `generated_examples.tsv`. A word whose gloss names killing, murder, weapons or
  blood ships at B1 only: کشتن, خون, قتل (were A1), اسلحه, قاتل, سلاح (were A2).
  Ranks, ids and glosses did not change. The pack has 3,025 sentences. A1/A2 glosses
  are scanned too; one sense of کردن was skipped by that scan. Four Tatoeba sentences
  with text errors are dropped by text match.
