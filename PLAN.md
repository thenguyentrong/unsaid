# Plan

Last update: 05.11.2026

## Goal

Hold a key, mouth a sentence, and the text shows up at the cursor with no sound. Whether a soft whisper is allowed is still open.

Popular languages, not all of them. Start with English, Vietnamese and Chinese, then add a few more Asian languages. One sentence can mix languages. At the end the lip model should be small and run locally.

The language list lives in data, not in code, so adding a language means adding data.

### What better means

Beating the big labs on benchmarks is probably not realistic. They have thousands of hours of video and much more compute.

Better here means better final text for one person:

- beat Lipflow on the same set of our own sentences
- work on mixed Vietnamese, Chinese and English, no public tool found for this so far
- train on your own face and your own silent mouthing

## Where public models are today

| Language | Wrong | Model |
|---|---|---|
| English | 12.8% of words | best published |
| English | 19.1% of words | Auto-AVSR, public weights, used by Lipflow |
| English | 31.9% of words | Lipflow's own test |
| German | 40.35% of words | MultiVSR |
| Vietnamese | 41.34% of words | ViCocktail |
| Mandarin | 32.48% of characters | CSSMCM, from 2019 |

These are mostly videos of people talking out loud. We found no numbers for silent mouthing on a webcam. Lipflow says to expect more errors there.

Training languages together helps each language. MultiVSR, words wrong:

| Language | Trained alone | Trained together |
|---|---|---|
| German | 57.18% | 40.35% |
| Spanish | 46.59% | 37.04% |
| French | 51.78% | 34.63% |
| Russian | 52.94% | 39.49% |
| Portuguese | 39.96% | 33.76% |

More video helps too. A new language got about 60% wrong with 25 hours of video and about 53% with 100 hours.

## Why lips aren't enough

Rough count on common words from public word lists, with 12 mouth shapes. This shows the size of the problem. It is not an accuracy forecast.

Share of words with a lookalike, weighted by how often the word is used:

| | English | German | English + German |
|---|---|---|---|
| Same sound | 22.5% | 12.4% | not counted |
| Same mouth shapes | 81.9% | 80.9% | 84.3% |

Mixing languages adds only about 2 points.

Tones make it harder:

| | Vietnamese | Mandarin |
|---|---|---|
| Tones kept | 26.1% | 53.0% |
| Tones removed | 96.5% | 75.1% |
| Mouth shapes only | 99.6% | 91.9% |

The Vietnamese list counts single syllables, so it overstates the problem a bit. The Mandarin pronunciation list is mostly traditional characters, so that column is rough.

Studies show tones are partly visible on the face, in the head, eyebrows and lips. Silent mouthing may hide them more, but that is a guess and needs testing.

## First check

Can a sentence come back from mouth shapes alone?

30 short test sentences in English, Vietnamese and Mandarin, plus Vietnamese + English, Mandarin + English and German + English mixes. They were turned into mouth shapes with no tones, then an LLM that never saw them had to rebuild them.

| Test | Exact sentences | Words wrong | Time per sentence |
|---|---|---|---|
| With word gaps | 25 of 30 | 6 | about 18 s |
| No word gaps | 23 of 30 | about 16 | about 30 s |

Most wrong words in the no-gaps test came from two mixed sentences.

This is the best case. The mouth shapes were perfect and the test sentences were simple. A real lip reader gets many shapes wrong and real sentences have names and jargon.

Both runs were much too slow for dictation, which needs about 2 seconds.

## Public data

| Data | Language | Size | Notes |
|---|---|---|---|
| [MultiVSR](https://github.com/Sindhu-Hegde/multivsr) | 13: en, pt, es, ru, de, fr, ja, it, zh, pl, nl, ca, tr | about 12,000 h | word-timed transcripts, 123K YouTube videos to download, pretrained models 4.4 GB, one language per clip |
| [ViCocktail](https://huggingface.co/datasets/nguyenvulebinh/ViCocktail) | Vietnamese | 269 h train | public [model](https://huggingface.co/nguyenvulebinh/AV-HuBERT-CTC-Attention-VI) |
| CN-CVS | Mandarin | 300+ h | transcripts came later with the CNVSRC challenge |
| [CSLR](https://github.com/cslr-lipreading/CSLR) | Chinese + English switching | 71.3 h | only switching dataset found so far |
| OLKAVS | Korean | not checked yet | |
| [WikiPron](https://github.com/CUNY-CL/wikipron) | 348 language codes | pronunciation lists | turns transcripts into sound labels, no hand labelling |

Hindi, Arabic and Bengali are not in MultiVSR. They have pronunciation lists but would need their own video.

Public video shows people talking out loud, so it can't replace clips of real silent mouthing.

Licences: Lipflow's code is MIT but its LRS3 weights are for non-commercial research. MultiVSR lists MIT on Hugging Face and no licence on GitHub, and the videos belong to their uploaders. So this is personal use for now.

## Steps

One step per round. If a step fails its pass check, stop and rethink before going on.

| # | Step | Where | Pass check |
|---|---|---|---|
| 1 | Decide languages, and silent or whisper | now | list agreed |
| 2 | Repeat the first check with 30 real sentences, with plenty of mixed ones | any PC | LLM rebuilds 80%+ of words (suggested bar) |
| 3 | Record about 60 silent clips and run Lipflow, MultiVSR and the Vietnamese model on them | training PC | gives the score to beat |
| 4 | Collect a subset of the public data and make sound labels from transcripts + pronunciations | training PC | labels are right on a hand-checked sample |
| 5 | Train the sound-first lip model | training PC | beats step 3 on the same clips |
| 6 | Add simulated faces | training PC | keep only if the score gets better |
| 7 | Fine-tune on your own silent clips | training PC | better on clips held back from training |
| 8 | Rebuild sentences with the LLM, your writing and your custom words | training PC | final text beats Lipflow, in about 2 s per sentence |
| 9 | Build the app: hold a key, text at cursor, reuse Lipflow's MIT code where it fits | training PC | works in real apps |
| 10 | Shrink the lip model so it runs small and local | training PC | accuracy stays close |

The training PC needs a strong NVIDIA GPU and several TB of disk (estimate). The full MultiVSR video set would be much bigger, so start with a subset.

Step 8 runs in the cloud unless a local LLM turns out good enough.

The sound-first idea comes from VALLR, which got 18.7% words wrong on LRS3 with far less labelled data. No public code found and it is not reproduced yet. So expect about the same accuracy as today's English models. The gain should be mixed languages and needing less labelled data.

## Simulated mouths

Different face shapes, colours, light and cameras should help with new faces and rooms. They can't teach new sounds.

If a simple table drives the mouth, p, b and m all look the same, and that is exactly the difference the model has to learn. So the mouth motion has to come from real video and only the look gets randomised.

SynthVSR shows simulation can cut the real data needed (43.3% wrong with 30 hours of real video), but it still needed real video. No study found on randomising colour for lip reading, so step 6 is a test, not a sure gain.

## Open questions

1. Which languages besides English, Vietnamese and Chinese?
2. Fully silent, or is a soft whisper okay? Lipflow got 6.9% wrong with lips + whisper in English, against 31.9% lips only, but says a real whisper will be worse.
3. 30 real sentences for step 2.
4. GPU and free disk of the training PC.

## Sources

- [Lipflow](https://github.com/amywork777/lipflow)
- [Scaling Multilingual Visual Speech Recognition](https://www.robots.ox.ac.uk/~vgg/publications/2025/Prajwal25/prajwal25.pdf)
- [Conformers Are All You Need for Visual Speech Recognition](https://arxiv.org/pdf/2302.10915)
- [ViCocktail paper](https://arxiv.org/pdf/2506.04635)
- [Chinese Mandarin lip reading with tones](https://arxiv.org/pdf/1908.04917)
- [CN-CVS and CNVSRC](https://arxiv.org/html/2406.10313v1)
- [Language Without Borders (CSLR)](https://neurips.cc/virtual/2024/poster/97448)
- [OLKAVS](https://arxiv.org/pdf/2301.06375)
- [VALLR](https://arxiv.org/abs/2503.21408v1)
- [SynthVSR](https://arxiv.org/pdf/2303.17200)
- [Facial cues in Mandarin tones](https://www.frontiersin.org/articles/10.3389/fcomm.2023.1148240)
- [FrequencyWords](https://github.com/hermitdave/FrequencyWords)
