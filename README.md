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
