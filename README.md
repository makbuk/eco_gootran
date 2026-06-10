# GooTran

A Drupal 7 module that automatically translates node content from Ukrainian to
English using Google's Translate API — for free, without an API key.

## Overview

GooTran hooks into the node lifecycle and, whenever a node of a specific content
type is created or updated, translates a set of fields and stores the result as
an English translation of the node (via Entity Translation).

It talks directly to the free, unofficial Google Translate web endpoint
(`translate.google.com/translate_a/single`) using cURL, mimicking the Android
Translate app. No API key or billing account is required.

The translation logic is based on
[statickidz/php-google-translate-free](https://github.com/statickidz/php-google-translate-free).

## Requirements

- Drupal **7.x**
- [Entity Translation](https://www.drupal.org/project/entitytranslation) module
- A content type named `addproblem` with the following fields:
  - `title_field`
  - `field_description`
  - `field_proposal`
- PHP cURL extension

## How it works

The module implements two node hooks:

- **`eco_gootran_node_insert($node)`** — runs when a node is created.
- **`eco_gootran_node_update($node)`** — runs when a node is updated (only adds a
  translation if an English one does not already exist).

For nodes of type `addproblem`, each hook:

1. Reads the source fields (`title_field`, `field_description`, `field_proposal`)
   in Ukrainian.
2. Translates them to English via `translate()`.
3. Stores the results under the `en` language using
   `entity_translation_get_handler()->setTranslation()`.
4. Persists the changes with `field_attach_update()`.

### Translation helpers

| Function | Purpose |
| --- | --- |
| `translate($source, $target, $text)` | Entry point. Translates short text directly; splits long text (≥ 999 chars) into pieces first. |
| `cut_into_pieces($source, $target, $text, $maxPieceLen)` | Splits long text on word boundaries and translates each piece, then concatenates. |
| `requestTranslation($source, $target, $text)` | Performs the cURL POST request to the Google Translate endpoint. Throws if the text is ≥ 5000 characters. |
| `getSentencesFromJSON($json)` | Parses the JSON response and concatenates the translated sentences. |

## Installation

1. Copy the module into your Drupal modules directory, e.g.
   `sites/all/modules/eco_gootran/`.
2. Enable it at **Administer → Modules**, or via Drush:

   ```bash
   drush en eco_gootran
   ```

## Configuration

The module has no settings UI. Source/target languages and the target content
type are hard-coded:

- Source language: `ua` (Ukrainian)
- Target language: `en` (English)
- Content type: `addproblem`

To adapt it to other languages, content types, or fields, edit
`eco_gootran.module` directly.

## Limitations & notes

- Relies on an **unofficial** Google Translate endpoint, which may change or rate
  limit at any time, and may violate Google's Terms of Service.
- SSL verification is disabled in the cURL request.
- Language and field names are hard-coded.
- A single translation request is limited to **5000 characters**; longer text is
  chunked automatically.

## License

No license specified.
