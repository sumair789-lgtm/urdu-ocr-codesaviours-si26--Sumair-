## Urdu OCR Project | Code Saviours SI-26 | Sumair Parveiz

## What is OCR?
Optical Character Recognition is besically a technology in wich the text from images and scanned documents are converted to editable and machine readable text.

## Why Urdu OCR is harder then the English OCR?
As the Urdu Language Script is more cursive then English and the words are joined together with no proper spacing so it becomes difficult to recognize those letter by the computer.

## 2 Real World Use Cases
1: Preservation of old Pakistani News Papers like Jung etc.


2: For the Visually impaired people that helps them to converte text-to-speech for becoming an accessibility tools.



## Why We Need a Better Model

### 📷 Image Gap Analysis

#### 1. Image: `processed_urdu_6.png`
* **Actual Urdu Text:** صحت سب سے بڑی نعمت ہے
* **Tesseract Output:** `شس سس عر ڈم مل رح -`
* **The Gap:** Tesseract completely failed to recognize a single correct word. Because the cursive *Nastaliq* script joins letters fluidly, the engine aggressively sliced the text blocks, hallucinating random standalone characters (like `ش`, `ع`, `ڑ`) and misinterpreting the word flow.

#### 2. Image: `processed_10045.png`
* **Actual Urdu Text:** اسپیکر نثار احمد کھوڑو نے ملاقاتیں کیں۔ اس موقع پر
* **Tesseract Output:** `یکر ڈار اح کھوڑو نے ملا تی نکییں۔ اس موحع ےر`
* **The Gap:** Proper nouns were heavily mangled (e.g., turning "اسپیکر" into "یکر" and "نثار" into "ڈار"). The complex, vertically stacked ligatures in "ملاقاتیں" were completely fractured into meaningless tokens like "ملا تی".

#### 3. Image: `processed_urdu_19.png`
* **Actual Urdu Text:** ورزش صحت کے لیے ضروری ہے
* **Tesseract Output:** `/ ور`
* **The Gap:** Extreme data deletion. Tesseract dropped roughly 80% of the sentence, entirely missing "صحت" and "ضروری ہے". It only managed to extract the first two characters of "ورزش" before failing completely.

#### 4. Image: `processed_10034.png`
* **Actual Urdu Text:** انٹرویو میں بتایا کہ اگرچہ وہ سبزی خور ہیں لیکن
* **Tesseract Output:** `انظرواو ہیں تا ماکہ اگرجہ ووسین کی خر ہیں مین`
* **The Gap:** Heavy character hallucination caused by vertical stacking issues. It transformed "انٹرویو" into the nonsensical string "انظرواو", turned "سبزی" into "ووسین", and combined separate character strokes to misread "لیکن" as "مین".

#### 5. Image: `processed_1.png`
* **Actual Urdu Text:** اسکے ساتھ ملحقہ علاقے کے عوام نے بجلی و گیس
* **Tesseract Output:** `کے سا تھ موہ علا تے کے عوام نے می وکس`
* **The Gap:** Structural disjointing of connected segments. It broke down complex cursive linkages like "ملحقہ" and "علاقے" into fractured pieces ("موہ" and "علا تے"). The final compound subject "بجلی و گیس" was completely misread as "می وکس".

---

### 📝 Summary

Tesseract fails on Urdu because it is engineered for discrete, left-to-right alphabets where static letters sit neatly on a horizontal baseline (like Latin scripts). Conversely, Urdu text—specifically in the traditional Nastaliq style—is a right-to-left, highly cursive script where characters fluidly change shape depending on their position in a word.

Instead of sitting side-by-side, Urdu characters stack vertically into complex clusters called ligatures. Because Tesseract relies on finding distinct horizontal spaces to segment letters, it blindly fractures these ligatures. This structural mismatch causes it to drop entire text blocks, misinterpret diacritical dots (nuqte), hallucinate random characters, and scramble text into meaningless gibberish.

---------------

## How It Works

This project uses **TrOCR** (Transformer-based OCR), a model architecture from Microsoft that pairs two components: an **image encoder** (a vision transformer) that "reads" the pixels of an image, and a **text decoder** (a language transformer) that converts what the encoder sees into output characters.

Rather than training a model from zero, this project uses **transfer learning** — starting from `microsoft/trocr-base-printed`, a version already trained on printed text, and fine-tuning it further on a custom dataset of Urdu images with paired text labels. The dataset (273 image-text pairs across newspapers, signboards, books, and synthetic sources) was fed through this pretrained model over multiple training epochs, adjusting its internal weights so it could learn to associate Urdu visual patterns with Urdu characters.

## Live Demo

**[Try it here — Urdu OCR on Hugging Face Spaces](https://huggingface.co/spaces/Sumair-Parveiz/urdu-ocr-codesaviours-si26-sumair)**

Upload an image containing Urdu text and the model will attempt to extract it.

## How to Run It Locally

```bash
git clone https://github.com/sumair789-lgtm/[your-repo-name].git
cd [your-repo-name]
pip install transformers torch pillow gradio sentencepiece
python app.py
```

The app will launch a local Gradio interface (and print a shareable public link) where you can upload images and see extracted text.

## Dataset Details

- **Total images:** 273 labeled Urdu text images
- **Sources:** 5 categories — newspaper clippings, signboards, books, synthetic generated text, and other miscellaneous printed sources
- **Variety:** Multiple fonts, backgrounds, and text lengths, generated partly through real-world collection and partly through synthetic image generation using PIL/Pillow with the Raqm layout engine for accurate Nastaliq rendering
- **Split:** 80% training (218 images) / 20% held-out test (55 images)

## Results

The fine-tuned TrOCR model achieved **0% exact-match accuracy (0/55)** on the held-out test set after 40 training epochs, with average training loss decreasing from 3.0980 (epoch 1) to 2.8373 (epoch 40).

**Root cause:** The base model's decoder and tokenizer (RoBERTa-based) were pretrained almost entirely on English text, with no meaningful prior exposure to Urdu's script. Fine-tuning on 218 images was insufficient for the decoder to learn a new writing system essentially from scratch.

**What I tried:**
- Switched evaluation from exact-match to Character Error Rate (CER) for a more standard, fine-grained measurement (result: CER of 100.1%, confirming the same underlying issue)
- Performed systematic error analysis across multiple evaluation runs, comparing predicted vs. actual outputs to identify failure patterns
- Expanded the tokenizer vocabulary with Urdu-specific characters and resized the model's embedding layer — this produced a much faster initial loss drop (9.0 → 4.5 within a single epoch) but did not fully converge within the available compute and data budget

**What I'd do differently with more time:**
- Start from an Urdu-aware or multilingual base checkpoint instead of an English-only one
- Collect a substantially larger training set — 218 images is small for adapting a 334M-parameter model to an unfamiliar script
- Use a learning rate schedule (warmup + gradual decay) rather than a fixed learning rate
- Allow more training epochs once the tokenizer vocabulary is properly expanded, since that experiment showed the strongest early learning signal of any approach tried

## Credit

Sumair Parveiz
Built during the Code Saviours ML/AI Internship — Batch SI-26.
