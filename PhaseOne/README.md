# Phase 1: Data Preprocessing and Tokenization

## Datasets

Four datasets were used:

### 1. AraSum

* Structure: article and lead summary
* Characteristics:

  * Long articles
  * Structured, formal Modern Standard Arabic
  * Lead-style summaries

### 2. Egyptian Arabic Summarization Dataset

* Structure: text, summarized_text, source_topics
* Characteristics:

  * Informal Egyptian Arabic
  * Natural summaries written by humans
  * Moderate length texts and summaries

### 3. SumArabic

* Structure: JSON records with text and headline
* Characteristics:

  * Short news texts
  * Headlines used as targets
  * Includes predefined splits (train, valid, test, ood)

### 4. Arabic Text Summarization Dataset

* Structure: text only, with synthetic summaries generated externally
* Characteristics:

  * No authentic human summaries
  * Used only for auxiliary analysis, not for core training

---

## Dataset Inspection

Each dataset was analyzed before any preprocessing was applied. The inspection included:

* Checking for missing values and duplicates
* Detecting HTML tags, URLs, and abnormal symbols
* Measuring the presence of:

  * Diacritics
  * Tatweel
  * English characters
  * Digits
* Computing length distributions for both source and target texts
* Reviewing random samples to verify text quality

### Key observations

* AraSum and Egyptian datasets are relatively clean and well-formed
* SumArabic contains valid data but with much shorter texts and headline-style targets
* The synthetic dataset lacks authentic summaries and is not suitable for model training
* English tokens and digits appear in multiple datasets, often representing meaningful entities such as names, abbreviations, and numerical facts

---

## Cleaning and Normalization

Cleaning was performed in a dataset-specific manner based on inspection results.

### General principles

* Preserve meaning and linguistic structure
* Avoid overly aggressive normalization
* Remove only clearly unnecessary or inconsistent elements

### Applied cleaning steps

#### Common steps across datasets

* Remove diacritics (tashkeel)
* Remove tatweel characters
* Normalize Alef variants:

  * إ, أ, آ → ا
* Normalize Ya:

  * ى → ي
* Normalize whitespace
* Remove URLs where present

#### What was intentionally preserved

* English words and abbreviations
* Digits and numerical expressions
* Punctuation

These elements were retained because they carry semantic value in news and technical content.

#### What was avoided

* Mapping ة → ه
* Mapping ؤ → و
* Mapping ئ → ي
* Removing all non-Arabic characters
* Removing numbers or punctuation

Such operations were found to degrade text quality and alter meaning.

---

## Dataset-Specific Adjustments

### AraSum

* Light cleaning only
* No removal of punctuation or digits
* Treated as a primary dataset due to size and quality

### Egyptian Dataset

* Light normalization
* Preserved dialectal characteristics
* Used as a primary dataset with authentic summaries

### SumArabic

* Light cleaning similar to other datasets
* Additional filtering:

  * Removed very short or incomplete texts
* Treated separately due to its headline-style targets and shorter source texts

### Synthetic Dataset

* Not used in core training
* Reserved for auxiliary analysis only

---

## Dataset Integration and Experimental Design

To account for differences in dataset structure and target style, three experimental setups were defined.

### Experiment A

* Datasets:

  * AraSum
  * Egyptian dataset
* Purpose:

  * Train on full summarization tasks with consistent summary style

### Experiment B

* Datasets:

  * AraSum
  * Egyptian dataset
  * SumArabic
* Purpose:

  * Evaluate the impact of adding headline-style data

### Experiment C

* Two-stage training:

  * Stage 1: SumArabic
  * Stage 2: AraSum + Egyptian
* Purpose:

  * Use SumArabic for initial learning of compression patterns, followed by fine-tuning on full summaries

For AraSum and the Egyptian dataset, train, validation, and test splits were created. SumArabic splits were used as provided.

All datasets were unified into a common schema:

* `source_text`
* `target_text`
* `dataset_name`
* `split`

---

## Tokenization

### Choice of tokenizer

SentencePiece was used for tokenization with the unigram language model.

Reasons:

* Arabic is morphologically rich
* Subword tokenization reduces vocabulary size
* Handles out-of-vocabulary words more effectively than word-level tokenization

### Tokenizer configuration

* Model type: unigram
* Vocabulary size: 16,000
* Character coverage: 1.0
* Special tokens:

  * `<pad>`
  * `<unk>`
  * `<start>` (BOS)
  * `<end>` (EOS)

### Tokenizer training

Two tokenizers were trained:

* Tokenizer A:

  * Trained on Experiment A training data (AraSum + Egyptian)

* Tokenizer B/C:

  * Trained on Experiment B training data (AraSum + Egyptian + SumArabic)
  * Shared between Experiments B and C

Only training splits were used for tokenizer training to avoid data leakage.

### Final preprocessing before tokenization

A final unified pass was applied to all datasets before tokenizer training:

* Normalize whitespace
* Light normalization of English tokens (lowercasing)

---

## Outputs of Phase 1

The following artifacts were produced:

* Cleaned versions of all datasets
* Unified dataset splits for Experiments A, B, and C
* SentencePiece tokenizer models for:

  * Experiment A
  * Experiments B and C
* Preprocessing scripts and inspection tools