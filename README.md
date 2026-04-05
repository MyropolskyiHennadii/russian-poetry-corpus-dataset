# Russian Poetry Corpus Dataset

A structured dataset of metric, strophic, rhyme, and grammatical analysis of Russian poetry.

## Description

The dataset contains numerical representations of poetic meter, strophic structure, rhyme patterns, and grammatical analysis for selected Russian poets. Syntax analysis is in progress: sentence segmentation of poems is complete (`poem_sentences`), but clause-level parsing (`sub_sentences`, `extracted_ranges`, `coordinate_segments`) is still being developed.

The key distinction from [The Russian National Corpus](https://ruscorpora.ru/en) is the **numerical representation of verse metrics** (e.g., `01010101` instead of "четырёхстопный ямб" / "iambic tetrameter"), enabling computational and statistical analysis.

### Known Limitations

Corpora are not yet complete for each poet but are approaching completeness. Coverage varies by author — some poets have near-complete corpora while others are still in progress.

## Statistics

- ~98,000 analyzed poetry lines

## Poets

| Poet | Original Name |
|------|---------------|
| Brodsky, Joseph | Бродский, Иосиф |
| Dashevsky, Grigory | Дашевский, Григорий |
| Gumilev, Nikolai | Гумилёв, Николай |
| Horace (in Russian translations) | Гораций |
| Khodasevich, Vladislav | Ходасевич, Владислав |
| Mandelstam, Osip | Мандельштам, Осип |
| Nabokov, Vladimir | Набоков, Владимир |
| Rein, Yevgeny | Рейн, Евгений |

## Data Sample

Example rows from `metrical_lines`:

| id_metrical_line | id_verse | row_key | line | representation | meter_group | number_of_tonic_feet | representation_with_spaces | irregularity_on_syllable | length | ending |
|---|---|---|------|----------------|-------------|---|----------------------------|---|---|---|
| 37903 | 3930 | 1 | Ни страны, ни погоста | 0010010 | Анапест | 2 | 0 01 0 010 | 0 | 7 | 10 |
| 37904 | 3930 | 2 | не хочу выбирать. | 001001 | Анапест | 2 | 0 01 001 | 0 | 6 | 1 |
| 37905 | 3930 | 3 | На Васильевский остров | 0010010 | Анапест | 2 | 0 0100 10 | 0 | 7 | 10 |
| 37906 | 3930 | 4 | я приду умирать. | 001001 | Анапест | 2 | 0 01 001 | 0 | 6 | 1 |

> **Note:** In `representation`, `0` = unstressed syllable, `1` = stressed syllable. `representation_with_spaces` shows the same pattern split by words. This numerical encoding enables computational analysis of meter patterns across the entire corpus.

## Download

### GitHub Releases
<!-- TODO: Update with actual filename and size before release -->
[Latest version](../../releases/latest) - `russian-poetry-corpus-dataset-v1.0.0.zip` (~ MB)

### Zenodo (Permanent Archive)
<!-- TODO: Update with actual Zenodo DOI URL before release -->
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)

The Zenodo archive provides a permanent, citable version with guaranteed long-term preservation backed by CERN.

## Database Schema

See the [PoetryCorpusModel](https://github.com/MyropolskyiHennadii/PoetryCorpusModel) repository for the full entity-relationship model (Java/JPA).

### Entities

| Entity | Table | Description |
|--------|-------|-------------|
| Author | `authors` | Poet/author information |
| BooksSource | `books_source` | Book or publication source |
| Poem | `poems` | Individual poem |
| MetricalLine | `metrical_lines` | Single metrical line of a poem |
| GrammaticalLine | `grammatical_lines` | Grammatical analysis of a metrical line |
| Rhyme | `rhymes` | Rhyme characteristics of a metrical line |
| Strophe | `strophes` | Strophe characteristics of a metrical line |
| MetersGroup | `meters_group` | Meter type definitions |
| PoemSentence | `poem_sentences` | Sentence extracted from a poem |
| SubSentence | `sub_sentences` | Sub-sentence (clause) within a poem sentence |
| CoordinateSegment | `coordinate_segments` | Coordinate segment within a sub-sentence |
| ExtractedRange | `extracted_ranges` | Character range extracted from a sub-sentence |

### Key Columns (`metrical_lines`)

| Column | Type | Description |
|--------|------|-------------|
| `id_metrical_line` | INT | Primary key |
| `id_verse` | INT | FK → `poems` |
| `row_key` | INT | Line position within the poem |
| `line` | TEXT | Original text of the line |
| `representation` | VARCHAR | Numerical stress pattern (e.g., `0010010`) |
| `meter_group` | VARCHAR | Meter type name (e.g., Анапест) |
| `number_of_tonic_feet` | INT | Number of metrical feet |
| `representation_with_spaces` | VARCHAR | Stress pattern split by words (e.g., `0 01 0 010`) |
| `irregularity_on_syllable` | INT | Syllable position of metrical irregularity (0 = none) |
| `length` | INT | Total number of syllables in the line |
| `ending` | INT | Clausula type: `1` = masculine, `10` = feminine, `100` = dactylic, etc. (pattern mirrors the ending stress) |

### Entity Relationships

```
Author (1) ──────────< (N) BooksSource (1) ──< (N) Poem
                              │                   │
                              ├──< (N) Poem...    ├──< (N) MetricalLine
                              │                   │         ├──── (1) GrammaticalLine
                              │                   │         ├──── (1) Rhyme
                              │                   │         └──── (1) Strophe
                              │                   │
                              └──< (N) PoemSentence
                                          │
                  ┌───────────────────────────────────────────┐
                  │           SubSentence                     │
                  │  FK: id_book_source   → BooksSource       │
                  │  FK: id_verse         → Poem              │
                  │  FK: id_poem_sentence → PoemSentence      │
                  │                                           │
                  │  ├──< (N) CoordinateSegment               │
                  │  └──< (N) ExtractedRange                  │
                  └───────────────────────────────────────────┘

MetersGroup (standalone, no FK relations)
```

### Relationship Details

| Parent | Child | Type | FK Column |
|--------|-------|------|-----------|
| Author | BooksSource | One-to-Many | `id_author` |
| BooksSource | Poem | One-to-Many | `id_book_source` |
| BooksSource | SubSentence | One-to-Many | `id_book_source` |
| Poem | MetricalLine | One-to-Many | `id_verse` |
| Poem | SubSentence | One-to-Many | `id_verse` |
| MetricalLine | GrammaticalLine | One-to-One | `id_metrical_line` |
| MetricalLine | Rhyme | One-to-One | `id_metrical_line` |
| MetricalLine | Strophe | One-to-One | `id_metrical_line` |
| PoemSentence | SubSentence | One-to-Many | `id_poem_sentence` |
| BooksSource | PoemSentence | One-to-Many | `id_book_source` |
| Poem | PoemSentence | One-to-Many | `id_verse` |
| SubSentence | CoordinateSegment | One-to-Many | `id_sub_sentence` |
| SubSentence | ExtractedRange | One-to-Many | `id_sub_sentence` |

## Installation

### Requirements
- MySQL 5.7+ or MariaDB 10.3+
- Approximately 500 MB disk space

### SQL Files

<!-- TODO: Update with actual filenames included in the release -->
The release archive contains the following SQL files (import in this order due to FK dependencies):

1. `authors.sql`
2. `books_source.sql`
3. `meters_group.sql`
4. `poems.sql`
5. `metrical_lines.sql`
6. `grammatical_lines.sql`
7. `rhymes.sql`
8. `strophes.sql`
9. `poem_sentences.sql`
10. `sub_sentences.sql`
11. `coordinate_segments.sql`
12. `extracted_ranges.sql`

### Import Instructions

1. **Create database:**
   ```sql
   CREATE DATABASE russian_poetry_corpus;
   ```

2. **Extract the zip file:**
   ```bash
   unzip russian-poetry-corpus-dataset-v1.0.0.zip
   ```

3. **Import SQL files:**
   ```bash
   mysql -u your_username -p russian_poetry_corpus < authors.sql
   mysql -u your_username -p russian_poetry_corpus < books_source.sql
   mysql -u your_username -p russian_poetry_corpus < meters_group.sql
   # ... import remaining tables in the order listed above ...
   ```

4. **Verify import:**
   ```sql
   USE russian_poetry_corpus;
   SELECT COUNT(*) FROM metrical_lines;
   -- Should return ~98,000
   ```

## Sources

- https://lib.ru/BRODSKIJ/brodsky_poetry.txt
- https://mandelstam.hse.ru/collected
- https://gumilev.ru/
- http://az.lib.ru/h/hodasewich_w_f/
- https://lib.ru/POEEAST/GORACIJ/hor1_1.txt
- https://magazines.gorky.media/
- Other open sources

## License

This dataset is licensed under [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).

You are free to:

✅ Use for any purpose (including commercial)

✅ Share and redistribute

✅ Modify and build upon

Under these terms:

⚠️ Attribution — Give appropriate credit

⚠️ ShareAlike — Derivatives must use the same license (keep it free)

Full license: https://creativecommons.org/licenses/by-sa/4.0/

## Related Projects

[Russian Poetry Corpus (apalladio.org)](https://apalladio.org/poetrycorpus/)

## Contributing

This database represents years of curation work. Contributions are welcome!
If you find errors or want to add new data:

1. Fork this repository
2. Make your changes
3. Submit a pull request with documentation

## Future Maintenance

This project is being made public to ensure long-term preservation of this data. If you're interested in maintaining or extending this database, please contact the original author.

## Contact

Original Author: Hennadii Myropolskyi
Email: miropolskij@gmail.com

For questions, suggestions, or collaboration opportunities.

## Citation

If you use this database in research or publications, please cite:

<!-- TODO: Update with actual repository URL before release -->
> Myropolskyi, H. (2026). Russian Poetry Corpus Dataset. GitHub.
> https://github.com/MyropolskyiHennadii/ArtefactsLocation-data

---

Last Updated: April 2026
Database Version: 1.0.0
Total lines: 98,000+
