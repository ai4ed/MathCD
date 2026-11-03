<div align="center">

# MathCD

### A Benchmark Dataset for Cognitive Diagnosis with Semantic Information

**NeurIPS 2026 · Evaluations and Datasets Track**

[![Dataset](https://img.shields.io/badge/Dataset-106%2C212_interactions-087f8c?style=flat-square)](#dataset)
[![Subsets](https://img.shields.io/badge/Subsets-3-6851a3?style=flat-square)](#dataset)
[![Data license](https://img.shields.io/badge/Data-CC_BY_4.0-238636?style=flat-square)](LICENSE)

**[Dataset](#dataset)** · **[Download](#download)** · **[Quickstart](#quickstart)** · **[Evaluation](#evaluation)** · **[Data card](#data-card)** · **[Citation](#citation)**

</div>

---

MathCD provides student-exercise interactions with rich semantic information, including exercise text, knowledge concept (KC) text, answer text, response text, exercise type, and exercise difficulty. It supports the development and evaluation of semantic-enhanced cognitive diagnosis (CD) models and the evaluation of foundation large language models on K-12 mathematics.

- **Three complementary subsets:** Grade3 in English, Math50k and MidMath in Chinese, spanning grades 3, 4–5 and 7.
- **Aligned data views:** curated interaction tables, fixed training/validation/test splits, Q matrices and integer-ID mappings.
- **Documented evaluation:** protocols for identifier-based CD, semantic-enhanced CD, and LLM correctness, answer-choice and difficulty-comparison tasks.

## Dataset

| Subset | Grades · language | Students | Exercises | KCs | Interactions | Train/valid | Test |
|:--|:--|--:|--:|--:|--:|--:|--:|
| [Grade3](#grade3) | 3 · English | 534 | 8,024 | 843 | 24,862 | 19,821 | 5,041 |
| [Math50k](#math50k) | 4–5 · Chinese | 962 | 20,460 | 1,483 | 50,936 | 40,674 | 10,262 |
| [MidMath](#midmath) | 7 · Chinese | 701 | 17,941 | 931 | 30,414 | 24,289 | 6,125 |
| **Total** | | **2,197** | **46,425** | **3,257** | **106,212** | **84,784** | **21,428** |

IDs and KC vocabularies are independent across subsets. Total KC counts sum the subset-local vocabularies. Correct-answer rates are 71.81%, 66.88% and 66.46%, respectively.

<a id="grade3"></a>**Grade3** — Third-grade mathematics translated from Chinese into English by the original learning platform. Chinese characters remain where they are mathematical objects being counted or arranged. Raw file: `Grade3/raw/grade_3_data_en.csv`.

<a id="math50k"></a>**Math50k** — Fourth- and fifth-grade mathematics in Chinese. Raw file: `Math50k/raw/grade_4_5_data.csv`.

<a id="midmath"></a>**MidMath** — Seventh-grade mathematics in Chinese. Raw file: `MidMath/raw/grade_7_data.csv`.

## Download

**Version 1.0.0:** [Dataset ZIP](https://github.com/ai4ed/MathCD/releases/download/v1.0.0/MathCD-v1.0.0.zip) · [GitHub Release](https://github.com/ai4ed/MathCD/releases/tag/v1.0.0)

> **Release candidate:** the download links become available when the `v1.0.0` Release and its dataset attachment are published.

The dataset attachment contains **21 data files**, a README and the dataset license. Its SHA-256 checksum and individual resource checksums are recorded in the Croissant files.

| Machine-readable metadata | Scope |
|:--|:--|
| [Grade3.json](croissant/Grade3.json) | Grade3 data, schema, checksums, provenance and Responsible AI metadata |
| [Math50k.json](croissant/Math50k.json) | Math50k data, schema, checksums, provenance and Responsible AI metadata |
| [MidMath.json](croissant/MidMath.json) | MidMath data, schema, checksums, provenance and Responsible AI metadata |

Once the Release is published, load records with `mlcroissant`:

```python
import mlcroissant as mlc

dataset = mlc.Dataset(
    "https://raw.githubusercontent.com/ai4ed/MathCD/v1.0.0/croissant/Grade3.json"
)
print(next(iter(dataset.records("Grade3-standard-train_valid"))))
```

<details>
<summary><strong>Inside the dataset ZIP</strong></summary>

```text
MathCD/
├── Grade3/                       # same structure for Math50k/ and MidMath/
│   ├── raw/<raw_filename>.csv     # content-rich interaction table
│   ├── standard/train_valid.csv  # integer IDs, labels and validation folds
│   ├── standard/test.csv         # held-out test interactions
│   ├── standard/train_valid.json # JSON Lines, including Q vectors
│   ├── standard/test.json
│   ├── mapping/q_matrix.csv      # exercise-to-KC membership
│   └── mapping/id_mapping.json   # raw-token/integer-ID and KC mappings
├── README.md
└── LICENSE
```

</details>

## Quickstart

Download and extract the dataset ZIP from the Release, then inspect the standard CSVs with Python's standard library:

```python
import csv
from pathlib import Path

root = Path("/path/to/extracted/MathCD")
with (root / "Grade3/standard/train_valid.csv").open(
    encoding="utf-8-sig", newline=""
) as source:
    interaction = next(csv.DictReader(source))
    print(interaction)
```

The released integer IDs, labels and folds can be used directly. See the Data Card below for Q matrices and mappings, and the Croissant files for programmatic loading.

## Evaluation

For five-fold cross-validation, select one fold from `0`–`4` in `train_valid.csv` for validation and use the remaining four for training. Keep `test.csv` fixed as the held-out test set (`fold = -1`). The approximately 80/20 per-student interaction split is not chronological.

## Data card

<details>
<summary><strong>Fields, encodings and ID alignment</strong></summary>

All raw CSVs have the following 14 fields. Each row is one retained student-exercise interaction. The raw tables are curated release tables, rather than complete unedited source logs.

| Raw field | Type | Meaning |
|---|---|---|
| `student_id` | string | Subset-local random release token (`s_` + 32 hexadecimal characters). |
| `exercise_id` | string | Subset-local random release token (`e_` + 32 hexadecimal characters). |
| `exercise_type` | string | `multiple-choice` for all released records. |
| `exercise_text` | string | Published stem, including LaTeX where present; generally excludes choices. |
| `response` | string | Student-selected option **letter**, not answer text. |
| `correct_answer` | string | Answer-key option **letter**; recover answer text from the full exercise. |
| `correct` | 0/1 | Retained binary correctness label, matching standard files. |
| `timestamp` | string | Original source `created_at`, `YYYY-MM-DD HH:MM:SS`, unchanged; timezone unspecified. |
| `interaction_order` | integer | Zero-based chronological order within a student's retained interactions; source row order breaks timestamp ties. |
| `grade_id` | integer | 3, 4/5, or 7, depending on subset. |
| `kc_list` | string | Serialized Python list of KC descriptions; parse with `ast.literal_eval`, not `eval`. |
| `kc_text` | string | Primary/combined KC description; combined names may be joined with `_`. |
| `difficulty` | integer 1-5 | Source-platform difficulty; larger values indicate higher difficulty. |
| `exercise_text_with_options` | string | Full published exercise with labeled choices, including teaching placeholders where applicable. |

Standard CSV fields are `student_id`, `exercise_id`, `correct`, `fold`, using contiguous integer IDs. Labels may be serialized as `0.0`/`1.0`; they still denote binary values. Standard `.json` files are **JSON Lines**, not one JSON array, and additionally contain `q_vector`, the binary KC membership vector.

`mapping/id_mapping.json` contains `students` (raw student token → integer ID), `exercises` (raw exercise token → integer ID), and `kcs` (KC text → integer ID). `q_matrix.csv` has `exercise_id` and `kc_ids`, the latter a JSON-encoded list of integer KC IDs. Q-vector positions use these same KC IDs. Do not combine equal integer IDs across subsets.

For Grade3 text-name lookup, split combined KC names on `_` and strip whitespace. Replace commas between digits with `@` (`10,000` → `10@000`), then other English/Chinese commas and adjacent whitespace with `@@`. Use the encoded name in `id_mapping.json/kcs`. For training, directly using the Q matrix avoids text-name parsing.


**Test-only exercises.** Relative to the full `train_valid` split, Grade3 has 840 test-only exercises (924 test interactions), Math50k has 2,207 (2,423), and MidMath has 2,749 (2,884). A selected training fold may leave further exercises unobserved. Interpret aggregate metrics with this coverage in mind.

The `.json` files under `standard/` contain one JSON object per line. Each Q vector is aligned to the subset's KC integer IDs. Parse serialized concept lists with `ast.literal_eval` or JSON as appropriate.

```python
import ast
import csv
from pathlib import Path

root = Path("/path/to/extracted/MathCD")
with (root / "Grade3/raw/grade_3_data_en.csv").open(
    encoding="utf-8-sig", newline=""
) as source:
    interaction = next(csv.DictReader(source))
    concepts = ast.literal_eval(interaction["kc_list"])
```

</details>

## Citation

If MathCD supports your work, please cite:

```bibtex
@inproceedings{li2026mathcd,
  title={{MathCD}: A Benchmark Dataset for Cognitive Diagnosis with Semantic Information},
  author={Li, Xueyi and Bai, Youheng and Cheng, Tengteng and Hou, Mingliang and Guo, Teng and Zheng, Jiaqi and Wu, Yongdong and Liu, Zitao},
  booktitle={Proceedings of the 40th Annual Conference on Neural Information Processing Systems},
  address={Sydney, Australia},
  year={2026},
  month={December}
}
```

## License

**Data and dataset documentation:** [CC BY 4.0](LICENSE).
