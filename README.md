# Signal Scope

**A transparent, client-side AI-text detection model, in a single JavaScript file.**

Signal Scope reads the statistical residue that language models leave in prose and turns it into an explainable AI-likeness score. It runs entirely in the browser: no API calls, no uploads, no dependencies, no build step. Everything analysed stays on the reader's machine.

This repository contains the model only (`signal-scope.js`). The website interface is not included; its layout and design are copyrighted and reserved.

**[Try the live demo](https://kaheichan.neocities.org/detector/)**

> This is an indicator, not proof. Clean, well-organised human writing, and especially writing by non-native English speakers, can score high. A score should start a conversation, never end one.

## Trained via Claude

The model's weights, thresholds, vocabulary databases, and calibration were developed and tuned end-to-end with **Claude Opus (Anthropic)**. Claude was used to build and iterate the signal definitions, the grading curve, the actionable-fix suggestions, and the large AI-vocabulary and phrase tables that drive the stylistic signal. It is, in effect, a detection model shaped by a frontier language model reasoning about how frontier language models write.

## What the model measures

The final score is a **weighted average of the signals below**, passed through a gentle logistic **grading curve** so that once several signals agree the score saturates toward the extremes, while weak signals stay near neutral (50).

| Signal | What it measures |
| --- | --- |
| Stylistic fingerprint | Documented AI vocabulary and phrases, triadic lists, participial padding, inflated copulas. Concrete dates, names and figures pull toward human. |
| Burstiness | Variation in sentence length. Uniform, metronomic cadence leans AI. |
| Perplexity proxy | How predictable the wording is, via a local n-gram model plus a bigram Bloom filter. |
| Token patterns | Recycled function-word templates such as "one reason is". Topic repetition is not penalised. |
| Semantic cohesion | How evenly topic vocabulary is spread. Flat, driftless texture leans AI; human tangents pull toward human. |
| Perturbation peak | Swaps synonyms into a sentence; if the original sits in a local probability trough it walked the path of maximum probability. |
| Grammar and mechanics | Real grammar and spelling errors lean human; em-dash and colon-label habits lean lightly AI. |

Every score is fully attributable. The model exposes, per signal, the exact spans of text that drove it, so a host UI can show the reader why any score is what it is.

## Self-hosting on your own site

You only need the one file. There is no bundler, no npm install, no server.

**1. Add the file to your site.** Download `signal-scope.js` from this repo and drop it next to your page:

```
your-site/
  detector.html
  signal-scope.js
```

**2. Load it.**

```html
<script src="signal-scope.js"></script>
```

**3. Call the scorers from your own code.** The model is a set of pure functions. A minimal example:

```js
const text = document.querySelector('#myEditor').value;
const question = ""; // optional essay prompt, improves the token/perplexity signals

const sents = splitSentences(text);
const signals = {
  style:        scoreStylistic(text).score,
  burstiness:   scoreBurstiness(sents).score,
  perplexity:   scorePerplexity(text, question).score,
  token:        scoreTokenPattern(text, question).score,
  cohesion:     scoreCohesion(text).score,
  perturbation: scorePerturbation(sents).score,
  grammar:      scoreGrammar(text, sents).score
};

const base   = weightedBase(signals);        // 0..100 weighted average
const score  = applyGradingCurve(base);      // 0..100 after the grading curve
console.log(Math.round(score), "/ 100 AI-likeness");
```

Each `score*` function returns `{ score, detail, dir, ... }`, where `detail` is a human-readable explanation and `dir` is `"AI"`, `"human"`, or `"neutral"`. Richer fields (e.g. `scorePerplexity(...).lowWordIdx`, `scoreCohesion(...).topTerms`, `scorePerturbation(...).peakSents`) let you highlight exactly which parts of the text drove each score.

**4. Adjust the weights if you like.** The `WEIGHTS` object near the top of the scoring section defines each signal's share of the final score. They sum to 1; change them to retune.

**Requirements:** any modern browser. If your editor renders newlines, set `white-space: pre-wrap` on it so line breaks display correctly. No external network access is required at runtime.

## Privacy

The model never transmits text anywhere. All analysis, including file parsing, happens locally in the browser. See the full terms and privacy notice: <https://kaheichan.neocities.org/privacyandterms>

## Limitations

- It reads statistical patterns, not authorship. It cannot know who wrote something.
- Short passages carry little signal; treat scores under about 150 words with lower confidence.
- Human and AI writing styles are converging, which narrows the gap any detector relies on.
- The bias against non-native English writers is real and documented. Do not use a score as evidence of misconduct.

## License

Model licensed **CC BY-NC-SA 4.0**. You may share and adapt it for non-commercial use with attribution, under the same license.

Copyright 2026 KH. [kaheichan.neocities.org](https://kaheichan.neocities.org)

A Shellcraft Service, part of Project Freedom Bell.
