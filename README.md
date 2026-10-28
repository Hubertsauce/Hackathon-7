# Invasive or Native?

**Hackathon 7 "Eyes on Earth" · Roboflow · SDG 15 – Life on Land (target 15.8: invasive alien species)**

**Live app:** https://hackathon-7-u97t5prpmjnmzn3wttrzv9.streamlit.app/

A phone web app for the Netherlands: take or upload a photo of a plant, and it says whether it looks like an **invasive species** or its **native look-alike**. It is a quick first check before reporting a sighting, not a final identification.

## The problem

Giant hogweed, Japanese knotweed and Himalayan balsam spread along Dutch verges, parks and waterways, and each has a harmless native look-alike. Invasive species contributed to **60% of recorded global extinctions** and cost more than **$423 billion a year** ([IPBES, 2023](https://unep.org/resources/report/invasive-alien-species-report)). In the Netherlands, **1,358 exotic species** have established themselves, 416 of them plants, and invasive species are estimated to cost the country **€1.1–3.1 billion a year** (including indirect costs). Giant hogweed has spread strongly since 1950, Himalayan balsam increasingly takes over river banks, and Japanese knotweed spreads along verges and is very hard to remove ([Van der Weijden & Reinhold, *Landschap* 2018](https://edepot.wur.nl/506684)). Giant hogweed has been on the **EU list of invasive species since 2017**, which obliges member states to detect and remove or contain it, and its sap plus sunlight causes **severe burns** in people and dogs ([Provincie Flevoland, 2022](https://www.flevoland.nl/getmedia/7874573e-3258-4187-b5fd-5e7e4abc9be1/FLE220046-Factsheet-Exoten-Reuzenberenklauw-DV.pdf)). Yet it looks very similar to native common hogweed.

## Who it is for

Volunteers, municipal green-space crews and citizen scientists in the Netherlands who find a plant outdoors, with only a phone, and need to know: *keep my distance and report it, or leave it?*
**Not for:** removing plants without expert confirmation, identifying plants other than the 7 below, or use outside the Netherlands.

**Why not a general plant-ID app** (ObsIdentify, Google Lens) or a field guide? Those give a species name and a confidence score, and treat every mistake as equally bad. Our users need a safety decision: the app is trained on exactly the invasive / look-alike pairs that get confused, and its rules deliberately prefer a false alarm over a missed giant hogweed, so the answer is *do not touch, report it* rather than a confident guess. A field guide needs knowledge and time that a crew member in the field often does not have.

## How it works

| Invasive | Native look-alike |
|---|---|
| Giant hogweed | Common hogweed; cow parsley / wild angelica |
| Japanese knotweed | Hedge bindweed |
| Himalayan balsam | Purple loosestrife |

1. **Photo in:** browser camera or upload (iPhone HEIC works).
2. **Roboflow model:** the app sends the photo to our ViT Base classifier trained in Roboflow, which scores all 7 species. It learns *species*, not "invasive yes/no", because invasiveness is not visible in a photo; the app maps species to a verdict.
3. **Safety rules** ([`plantid.py`](plantid.py)): top score below **70%** → *Unsure, do not touch*; giant hogweed **≥ 20%** → always a warning, never "Native". A false alarm is harmless, a missed giant hogweed is not.
4. **Output:** **Invasive**, **Native** or **Unsure, do not touch**, then how to recognise the plant, what to do, and for invasive or unsure results a button to report it on [Waarneming.nl](https://waarneming.nl), where an expert confirms it.

**Data:** 840 research-grade iNaturalist photos from the Netherlands (CC0 / CC BY / CC BY-NC, all credited). 760 for training in Roboflow (80/15/5 split; flips, ±15° rotation, crop, brightness; no grayscale or 90° rotation, because colour and orientation are real clues). **80 kept apart** as our own test set, from other sightings *and* other photographers.

## Results

* **Roboflow test split (38 photos):** F1 82.5%, precision 86.8%, recall 81%. Almost every mistake was hogweed against hogweed; 1 of 4 giant hogweeds was called common hogweed.
* **Our 80 held-out photos** ([`evaluate.py`](evaluate.py), same safety rules as the app): species accuracy [TODO] · giant hogweed warned about [TODO] / 15 · invasive plants called "Native" [TODO] / 35 · false alarms on native plants [TODO] / 45.

## How to run it

Open the **live app** on a phone. Locally: `pip install -r requirements.txt` (Python 3.11+), copy `.streamlit/secrets.toml.example` to `.streamlit/secrets.toml` and add a Roboflow API key and model ID, then `streamlit run app.py`. Held-out test: `python evaluate.py path/to/dataset/test` (saves `results.csv`).

## Ethical reflection (data card)

| | |
|---|---|
| **Source & licence** | 840 research-grade iNaturalist photos, Netherlands only, CC0 / CC BY / CC BY-NC, all photographers credited. No own photos. |
| **Collection** | Selected by hand: max 10 per photographer and 3 per sighting per class; duplicates removed with file hashes and an image fingerprint; 26 rejected after checking every photo by eye. |
| **Labels** | Species, from iNaturalist community votes: reliable, not perfect. |
| **Composition & splits** | 7 species, 90–150 training photos each (760 in total). 80 test photos from other sightings *and* other photographers. |
| **Gaps & biases** | Mostly sharp summer photos of plants in flower; autumn/winter photos only 19% (giant hogweed), 3% (bindweed), 9% (loosestrife), so the model may partly have learned "dead stems = invasive". Photos cluster around popular places. |
| **Failure modes** | Hogweed against hogweed (1 of 4 giant hogweeds called common); dead stems and young rosettes; any plant outside the 7 is forced into one of them (e.g. poisonous hemlock, which looks like cow parsley); stem spots lost at 224×224 px. |
| **Intended use** | First check before reporting to Waarneming.nl. **Not** for removing plants without an expert, other species, or use outside the Netherlands. |
| **What could go wrong** | Giant hogweed called "Native" → someone touches it and gets burned. False alarms → wasted municipal time, useful native plants removed. Over-trust → cutting knotweed, which spreads it. |
| **What we did** | 70% "unsure" rule, 20% giant hogweed warning, strict held-out test, expert confirmation via Waarneming.nl before anyone acts. **Next:** more winter photos and an "other plant" class. |