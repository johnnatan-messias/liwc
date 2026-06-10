# LIWC Text Analysis

This project provides a Python 3 notebook for dictionary-based analysis using
the LIWC 2015 English dictionary. It calculates category counts, percentages,
dictionary coverage, unmatched words, and document-level outputs.

The main implementation is in [`liwc.ipynb`](liwc.ipynb).

## How It Works

The notebook:

1. tokenizes each document;
2. matches tokens against exact and wildcard entries in the LIWC dictionary;
3. increments every category assigned to a matching token;
4. calculates category percentages using the analyzed word count;
5. returns diagnostics and optional Polars DataFrames.

The primary score is:

```text
category percentage = 100 * category matches / analyzed words
```

LIWC categories overlap. A word may count toward multiple categories, so
category percentages do not add up to 100%.

## Requirements

- Python 3
- A Jupyter-compatible editor, such as VS Code or JupyterLab
- Polars for DataFrame output
- A properly licensed LIWC dictionary

Install the Python dependency:

```bash
python3 -m pip install -r requirements.txt
```

The notebook also contains an installation cell:

```python
%pip install -r requirements.txt
```

## Input Format

By default, the notebook reads `./data/input.txt`:

```python
INPUT_PATH = Path("./data/input.txt")
```

Each non-empty line is treated as one independent document:

```text
I feel happy about the result.
This has been a difficult day.
We expect the project to improve.
```

A document can represent a tweet, post, response, sentence, or other unit of
analysis. If one document spans multiple lines, load it from a structured
format such as CSV or JSONL instead of using the example line-based loader.

## Dictionary

The default dictionary is:

```python
DICTIONARY_PATH = Path("./dictionary/LIWC2015_English.dic")
```

The repository also contains a LIWC 2007 dictionary. Change
`DICTIONARY_PATH` before running the parser cells to use another compatible
dictionary.

Dictionary entries can be exact words or prefix wildcards. For example,
`abandon*` matches `abandon`, `abandoned`, and `abandonment`.

## Running the Notebook

1. Open `liwc.ipynb`.
2. Select a Python 3 kernel.
3. Update `INPUT_PATH` and, if needed, `DICTIONARY_PATH`.
4. Configure optional stopword filtering.
5. Run the cells from top to bottom.

The examples first inspect one document, then analyze the complete corpus.
This makes it easier to verify tokenization and dictionary coverage before
using the full output.

## Optional Stopwords

Standard LIWC analysis normally retains function words because pronouns,
articles, prepositions, and auxiliary verbs are meaningful LIWC categories.
The recommended default is therefore:

```python
STOPWORDS_PATH = None
```

To remove stopwords before scoring, point to a UTF-8 text file:

```python
STOPWORDS_PATH = Path("./data/english_stopwords.txt")
```

The file may contain one or more words per line. Blank lines and lines
beginning with `#` are ignored. Matching is case-insensitive and uses the same
tokenizer as the documents.

When filtering is enabled, category percentages use only the remaining words
as their denominator. Filtered results are not directly equivalent to
standard full-text LIWC scores.

## Output

For a single text, `analyze_text` returns:

- `original_word_count`: words before stopword filtering
- `removed_stopword_count`: word tokens removed as stopwords
- `word_count`: words included in the LIWC denominator
- `matched_word_count`: analyzed words matching at least one category
- `dictionary_coverage_percent`: matched words divided by analyzed words
- `unmatched_words`: unmatched token frequencies
- `category_counts`: raw counts keyed by category ID
- `rows`: category names, counts, and percentages

Example:

```python
result = analyze_text(
    "I feel happy about the result.",
    stopwords=stopwords,
)
print_results(result, limit=25)
```

For multiple documents:

```python
results = analyze_documents(documents, stopwords=stopwords)
```

This returns one dictionary per document and preserves document boundaries.

## Polars Output

Create a category table for one result:

```python
category_frame = result_to_polars(result)
```

Inspect unmatched words:

```python
unmatched_frame = unmatched_words_to_polars(result)
```

Create a wide document-level DataFrame:

```python
document_frame = analyze_documents_polars(
    documents,
    stopwords=stopwords,
)
```

The wide DataFrame contains one row per document, metadata columns, and one
percentage column per LIWC category. Categories absent from a document are
represented as `0.0`.

## Interpreting Results

`percent_of_words` is the main category score. For example, a Positive
Emotions value of `4.5` means that 4.5% of analyzed word tokens matched that
category. It does not mean the author was 4.5% positive.

Use scores comparatively and in context. Differences may reflect topic,
genre, audience, platform conventions, or preprocessing choices in addition
to psychological processes.

Before drawing conclusions:

- inspect dictionary coverage and frequent unmatched words;
- verify matches in their original context;
- compare percentages rather than raw counts when text lengths differ;
- preserve documents as separate observations for group comparisons;
- treat short documents and categories with few matches cautiously;
- report whether stopword filtering was enabled.

LIWC category scores are lexical indicators, not diagnoses or direct
measurements of personality, mental health, deception, intentions, or
emotional state.

## Limitations

This notebook reproduces dictionary-category matching. Its tokenizer may not
be identical to the official LIWC application.

Proprietary summary scores such as Analytic, Clout, Authentic, and Tone cannot
be reconstructed exactly from the dictionary alone.
