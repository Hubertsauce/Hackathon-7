# Invasive or Native?

A field ID helper for volunteers and municipal green-space crews in the Netherlands. Take or upload a photo of a plant, and the app tells you whether it looks like an **invasive species** or a **native look-alike**. It is a quick first check before you report a sighting, not a final identification.

**Who it's for:** people who maintain parks, verges and waterways, and citizen scientists who record sightings.

**SDG 15 – Life on Land:** invasive species are a major driver of biodiversity loss. Finding them early is far cheaper than removing them once they've spread. Giant hogweed also matters for people: its sap causes severe burns in sunlight, so a mix-up with harmless common hogweed has real consequences.

## What it recognises

| Invasive | Native look-alike |
|---|---|
| Giant hogweed | Common hogweed; cow parsley / wild angelica |
| Japanese knotweed | Hedge bindweed |
| Himalayan balsam | Purple loosestrife |

The app shows **Invasive**, **Native** or **Unsure, do not touch**, with the species and a confidence score. Two safety rules make it careful on purpose:

- below 70% confidence it says *Unsure* instead of guessing;
- if giant hogweed scores 20% or more, it always shows a warning, even when another plant is the top answer.

For invasive or unsure results it links to [Waarneming.nl](https://waarneming.nl) to report the sighting.

## How it works

A ViT image classification model trained in Roboflow on about 760 research-grade iNaturalist photos from the Netherlands. 80 more photos, from different photographers, are kept apart as a test set. See `dataset/DATA_METHODOLOGY.md` for how the data was collected and its limitations.

## How to run it

1. Install Python 3.11+ and the requirements:
   ```
   pip install -r requirements.txt
   ```
2. Copy `.streamlit/secrets.toml.example` to `.streamlit/secrets.toml` and fill in your Roboflow API key and model ID (for example `netherlands-plant-field-id-5/3`; you can find it under *Deploy* in your Roboflow project).
3. Start the app:
   ```
   streamlit run app.py
   ```
4. To test the model on the 80 held-out photos:
   ```
   python evaluate.py path/to/dataset/test
   ```

**Online version:** deployed on Streamlit Community Cloud: *add your link here*.

## Results on the held-out test set

*Fill in after running `evaluate.py`: species accuracy, and how many giant hogweed photos were wrongly called "native".*
