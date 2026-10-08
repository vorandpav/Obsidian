# Obsidian Vault Refactor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Переложить vault в тематические папки в корне, Title Case имена, единый шаблон заметок, слить дубли, почистить мусор и аттачи.

**Architecture:** Старое дерево `Obsidian/` остаётся до конца миграции как источник. Новые папки создаются в корне vault. Перенос папки за папкой → rename → шаблон → merge дублей → индексы → аттачи → удаление `Obsidian/`. Спорные файлы не «улучшаем по вкусу» молча — помечаем и разбираем с пользователем.

**Tech Stack:** Obsidian markdown, wiki-links, PowerShell/файловая система Windows; git в vault отсутствует — шаги commit пропускать.

## Global Constraints

- Имена файлов и H1: английский Title Case с пробелами.
- Тело заметок: русский; математику/выводы не переписывать.
- Обязательные секции (кроме `Home.md`, `_Index.md`, `Archive/`): `## Если спросят`, `## Связанное`.
- Ссылки только `[[Wiki Links]]`.
- Карта папок и правила merge — как в `docs/superpowers/specs/2026-09-27-obsidian-vault-refactor-design.md`.
- Удаление из `Archive/` — только после явного ОК пользователя.
- Если заметка сомнительна (куда класть / что сливать) — остановить задачу, спросить пользователя, не угадывать.

## File map (источник → канон)

| Источник | Канон |
|----------|--------|
| `Obsidian/Home.md` | `Home.md` (переписать) |
| `Obsidian/ML/pipeline.md` | `Fundamentals/Pipeline.md` |
| `Obsidian/ML/gradient-descent.md` | `Fundamentals/Gradient Descent.md` |
| `Obsidian/ML/loss-functions.md` | merge → `Linear Models/Basic Loss.md` + `Linear Models/Cross Entropy.md` (уникальные куски) |
| `Obsidian/ML/overfitting-bias-variance.md` | merge → `Boosting/Bias Variance Tradeoff.md` |
| `Obsidian/ML/metrics.md` | merge → `Linear Models/Classification Metrics.md` (+ multiclass при необходимости) |
| `Obsidian/ML/algorithms.md` | `Archive/Algorithms Dump.md` (потом разбор по желанию) |
| `Obsidian/ML/README.md` | удалить после нового `Home` |
| `Obsidian/ML/Prior and posterior distribution.md` | `Probability And Stats/Prior And Posterior Distribution.md` |
| `Obsidian/ML/DeepLearning/Bayes theorem.md` | `Probability And Stats/Bayes Theorem.md` |
| `Obsidian/ML/DeepLearning/Central Limit Theorem.md` | `Probability And Stats/Central Limit Theorem.md` |
| `Obsidian/ML/Lora/Idea.md` | `Fine Tuning/LoRA Idea.md` |
| `Obsidian/ML/Lora/Realization.md` | `Fine Tuning/LoRA Realization.md` |
| `Obsidian/ML/Lora/QLoRA.md` | `Fine Tuning/QLoRA.md` |
| `Obsidian/ML_trainings/1. Basics/1.*/Likelihood.md` | `Fundamentals/Likelihood.md` |
| `…/Naive Bayes classifier.md` | `Fundamentals/Naive Bayes Classifier.md` |
| `…/2. linear_regression/*` | `Linear Models/*.md` (Title Case) |
| `…/3. logistic_regression/*` | `Linear Models/*.md` |
| `…/4. trees_and_ensembles/*` | `Trees And Ensembles/*.md` |
| `…/5. boosting/*` | `Boosting/*.md` |
| `…/6. intro_to_dl/Backpropagation.md` | `Deep Learning/Backpropagation.md` |
| `…/7. explaining_ai/*` | `Interpretability/*.md` |
| `…/8. distillation/Distillation.md` | `Deep Learning/Distillation.md` |
| `Obsidian/ML_trainings/2. NLP/**` | `NLP/*.md` (плоско, без 1/2/3) |
| `Obsidian/Math/Analysis/*` | `Math/*.md` |
| `Obsidian/Math/Statistics/Basics/*` | `Probability And Stats/*.md` |
| `Obsidian/Math/Statistics/2. Estimation…/Introduction.md` | `Probability And Stats/Estimation Of Shift And Scale.md` |
| `Obsidian/Coding/собес/algorithms/**` | `Algorithms/` (+ `Algorithms/Graphs/`) |
| `Obsidian/Coding/собес/python.md` | `Python/Python.md` |
| `Obsidian/Coding/собес/sql.md` | `SQL/SQL.md` |
| `Obsidian/solution_A_optimal_prediction.md` | `Archive/Optimal Prediction Solution.md` |
| `Без названия.md` | `Archive/Untitled.md` |
| все `*.png` / pastes | `_attachments/` (осмысленные имена где ясно) |
| все `lect*.pdf`, `Limar.pdf` | `_attachments/lectures/` |

Title Case примеры переименований: `Cross-Entropy.md` → `Cross Entropy.md`; `embedding.md` → `Embedding.md`; `tokenization.md` → `Tokenization.md`; `binary-search.md` → `Binary Search.md`; `bfs-dfs.md` → `BFS DFS.md`; `union-find.md` → `Union Find.md`.

---

### Task 1: Scaffold + Obsidian attachment path

**Files:**
- Create: все пустые тематические папки из карты + `_attachments/lectures/` + `Archive/`
- Create: заготовки `_Index.md` (заголовок + пустой список) в каждой теме
- Modify: `.obsidian/app.json` → `"attachmentFolderPath": "_attachments"`

**Interfaces:**
- Produces: корневая карта папок готова к переносу; старый `Obsidian/` ещё цел

- [ ] **Step 1: Создать папки**

В корне `e:\Documents\Obsidian Vault`:

```
Fundamentals, Linear Models, Trees And Ensembles, Boosting, Deep Learning,
NLP, Interpretability, Fine Tuning, Probability And Stats, Math,
Algorithms, Algorithms\Graphs, Python, SQL, Archive, _attachments, _attachments\lectures
```

- [ ] **Step 2: Создать `_Index.md` в каждой теме** (не в Archive/_attachments)

Содержимое-заготовка:

```markdown
# <Folder Name>

## Notes

<!-- links added as notes land -->
```

- [ ] **Step 3: Починить attachments в `.obsidian/app.json`**

Выставить:

```json
{
  "alwaysUpdateLinks": true,
  "newFileLocation": "current",
  "attachmentFolderPath": "_attachments"
}
```

- [ ] **Step 4: Verify**

`Get-ChildItem -Directory` в корне показывает новые папки; `app.json` читается как UTF-8 с `_attachments`.

---

### Task 2: Fundamentals + Probability And Stats + Fine Tuning (без merge)

**Files:**
- Create from source: файлы по file map для Fundamentals (кроме merge-целей), Probability And Stats, Fine Tuning
- Modify: соответствующие `_Index.md`

**Interfaces:**
- Consumes: scaffold Task 1
- Produces: канонические заметки без зависимости от ещё не перенесённых Linear Models (wiki-links могут временно биться — ок до Task 8)

- [ ] **Step 1: Скопировать и переименовать**

| From | To |
|------|-----|
| `Obsidian/ML/pipeline.md` | `Fundamentals/Pipeline.md` |
| `Obsidian/ML/gradient-descent.md` | `Fundamentals/Gradient Descent.md` |
| `Obsidian/ML_trainings/.../Likelihood.md` | `Fundamentals/Likelihood.md` |
| `Obsidian/ML_trainings/.../Naive Bayes classifier.md` | `Fundamentals/Naive Bayes Classifier.md` |
| `Obsidian/ML/Prior and posterior distribution.md` | `Probability And Stats/Prior And Posterior Distribution.md` |
| `Obsidian/ML/DeepLearning/Bayes theorem.md` | `Probability And Stats/Bayes Theorem.md` |
| `Obsidian/ML/DeepLearning/Central Limit Theorem.md` | `Probability And Stats/Central Limit Theorem.md` |
| `Obsidian/Math/Statistics/Basics/Descriptive statistics.md` | `Probability And Stats/Descriptive Statistics.md` |
| `Obsidian/Math/Statistics/Basics/Error types.md` | `Probability And Stats/Error Types.md` |
| `Obsidian/Math/Statistics/2. Estimation of shift and scale parameters/Introduction.md` | `Probability And Stats/Estimation Of Shift And Scale.md` |
| `Obsidian/ML/Lora/Idea.md` | `Fine Tuning/LoRA Idea.md` |
| `Obsidian/ML/Lora/Realization.md` | `Fine Tuning/LoRA Realization.md` |
| `Obsidian/ML/Lora/QLoRA.md` | `Fine Tuning/QLoRA.md` |

- [ ] **Step 2: Выровнять H1 под имя файла; добавить недостающие `## Если спросят` / `## Связанное`**

Не менять формулы. Если секции уже есть — не дублировать.

- [ ] **Step 3: Заполнить `_Index.md` для Fundamentals, Probability And Stats, Fine Tuning**

- [ ] **Step 4: Verify**

Список `.md` в трёх папках совпадает с таблицей; у каждой заметки есть обе обязательные секции.

---

### Task 3: Linear Models (курсные файлы) + Trees + Boosting + DL + Interpretability

**Files:**
- Create: все `ML_trainings/1. Basics/2–8` по map (кроме уже ушедших в Fundamentals)
- Modify: `_Index.md` этих папок

- [ ] **Step 1: Перенести linear_regression + logistic_regression → `Linear Models/`** с Title Case именами:

`Linear Regression Model`, `Basic Loss`, `Gauss Markov Theorem`, `Regularization`, `Hyperplane`, `Linear Classification`, `Logistic Regression`, `Sigmoid`, `Maximum Likelihood Estimation`, `Cross Entropy`, `Classification Metrics`, `Multiclass Classification`, `Multiclass Metrics`

- [ ] **Step 2: Перенести trees → `Trees And Ensembles/`:** `Decision Tree`, `Information Criteria`, `Bootstrap`, `Random Forest`

- [ ] **Step 3: Перенести boosting → `Boosting/`:** `AdaBoost`, `Bias Variance Tradeoff`, `Gradient Boosting`

- [ ] **Step 4: `Backpropagation`, `Distillation` → `Deep Learning/`; explaining_ai → `Interpretability/` (`Importance Estimation`, `LIME`, `SHAP`, `Grad CAM`)

- [ ] **Step 5: Шаблон A на каждой; обновить `_Index.md`

- [ ] **Step 6: Verify** — счётчик файлов: Linear Models 13, Trees 4, Boosting 3, Deep Learning 2, Interpretability 4

---

### Task 4: NLP + Math

**Files:**
- Create: NLP notes плоско; Math analysis notes
- Modify: `NLP/_Index.md`, `Math/_Index.md`

- [ ] **Step 1: Перенести NLP** → `NLP/`: `Tokenization`, `Embedding`, `RNN`, `LSTM`, `Gradient Problems`, `Attention`, `Self Attention`, `Transformer`, `Beam Search`

- [ ] **Step 2: Перенести Math** → `Math/`: `Matrix Differentiation`, `Quadratic Form`

- [ ] **Step 3: Шаблон A + индексы

- [ ] **Step 4: Verify** — 9 NLP + 2 Math; картинки пока могут ссылаться на старые пути (починка в Task 7)

---

### Task 5: Algorithms + Python + SQL

**Files:**
- Create: `Algorithms/*.md`, `Algorithms/Graphs/*.md`, `Python/Python.md`, `SQL/SQL.md`
- Modify: индексы

- [ ] **Step 1: Перенести algorithms** с Title Case (убрать kebab):

Корень Algorithms: `Binary Search`, `Two Pointers`, `Sliding Window`, `Sorting`, `Greedy`, `Dynamic Programming`, `Heaps`, `Trees`, `Backtracking`, `Number Theory`, `Complexity`  
Graphs: `BFS DFS`, `Shortest Path`, `Topological Sort`, `Union Find`  
Старые `README.md` не копировать как README — содержимое навигации влить в `Algorithms/_Index.md` и `Algorithms/Graphs/_Index.md`

- [ ] **Step 2: `python.md` → `Python/Python.md`; `sql.md` → `SQL/SQL.md`; шаблон A (для длинных шпаргалок — добавить секции в конец, тело не резать)

- [ ] **Step 3: Verify** — wiki-links внутри Algorithms обновлены под новые имена

---

### Task 6: Merge дублей из легаси `ML/`

**Files:**
- Modify: `Linear Models/Basic Loss.md`, `Linear Models/Cross Entropy.md`, `Linear Models/Classification Metrics.md`, `Boosting/Bias Variance Tradeoff.md`
- Create: `Archive/Algorithms Dump.md` из `ML/algorithms.md`
- Delete after merge: исходники в `Obsidian/ML/` для этих тем (или оставить до Task 9 — предпочтительно удалять только в Task 9 вместе со всем деревом)

**Interfaces:**
- Consumes: каноны Task 2–3
- Produces: одна заметка на тему; уникальный контент легаси не потерян

- [ ] **Step 1: Diff `ML/metrics.md` vs `Classification Metrics.md` (+ multiclass)**

Перенести уникальные таблицы/ремарки в канон. Не дублировать формулы. Если неочевидно что канон — **спросить пользователя**.

- [ ] **Step 2: Diff `loss-functions.md` vs Basic Loss + Cross Entropy** — то же

- [ ] **Step 3: Diff `overfitting-bias-variance.md` vs `Bias Variance Tradeoff.md`** — то же

- [ ] **Step 4: `algorithms.md` → `Archive/Algorithms Dump.md`** без разбора по файлам (разбор — отдельный запрос)

- [ ] **Step 5: Verify** — в канонах нет явной потери уникальных секций легаси; в новых папках нет второго файла той же темы

---

### Task 7: Attachments

**Files:**
- Move: все png/pdf из `Obsidian/**` → `_attachments/` / `_attachments/lectures/`
- Modify: заметки с `![[...]]` под новые имена/пути
- Delete: пустые `_attachments` / `вложенные файлы` внутри старого дерева (в Task 9)

- [ ] **Step 1: Перенести PDF** в `_attachments/lectures/` с понятными именами, например `Basics 02 Linear Regression.pdf`, `NLP 03 Attention.pdf`, `Limar.pdf`

- [ ] **Step 2: Перенести PNG** в `_attachments/`; где связь ясна — переименовать (`Grad CAM Idea.png`, `Self Attention Encoder.png`, `Transformer Architecture.png`, …)

- [ ] **Step 3: Обновить `![[...]]` во всех новых заметках** (поиск по vault на `Pasted image` и старые имена)

- [ ] **Step 4: Verify** — открыть 2–3 заметки с картинками в Obsidian (или проверить, что каждый `![[` резолвится в существующий файл под `_attachments`)

---

### Task 8: Home + индексы + Archive сирот + link pass

**Files:**
- Create/overwrite: `Home.md`
- Modify: все `_Index.md` (полный список)
- Create: `Archive/Optimal Prediction Solution.md`, `Archive/Untitled.md`
- Modify: глобальный поиск битых `[[links]]`

- [ ] **Step 1: Перенести сирот в Archive**

`Obsidian/solution_A_optimal_prediction.md` → `Archive/Optimal Prediction Solution.md`  
`Без названия.md` → `Archive/Untitled.md`  
Не удалять без ОК.

- [ ] **Step 2: Написать `Home.md`**

```markdown
# Home

## Hubs

- [[Fundamentals/_Index|Fundamentals]]
- [[Linear Models/_Index|Linear Models]]
- [[Trees And Ensembles/_Index|Trees And Ensembles]]
- [[Boosting/_Index|Boosting]]
- [[Deep Learning/_Index|Deep Learning]]
- [[NLP/_Index|NLP]]
- [[Interpretability/_Index|Interpretability]]
- [[Fine Tuning/_Index|Fine Tuning]]
- [[Probability And Stats/_Index|Probability And Stats]]
- [[Math/_Index|Math]]
- [[Algorithms/_Index|Algorithms]]
- [[Python/_Index|Python]]
- [[SQL/_Index|SQL]]

## Gaps

<!-- только реально пустые/недописанные после миграции -->
```

Подправить wiki-пути к `_Index`, если Obsidian резолвит лучше по имени `[[Fundamentals]]` — допустимо `[[_Index]]` из контекста; главное — кликабельные хабы.

- [ ] **Step 3: Полный link pass** — найти ссылки на старые имена (`gradient-descent`, `Cross-Entropy`, README, `_index_basics`, …) и заменить на канон

- [ ] **Step 4: Verify** — `rg` по `Obsidian/` в новых файлах не должен находить путей; выборочно 10 wiki-links резолвятся

---

### Task 9: Удалить старое дерево + финальный чеклист

**Files:**
- Delete: `Obsidian/` целиком (после verify)
- Delete: только если пуст/дубль — старые корневые хвосты

- [ ] **Step 1: Inventory diff**

Список всех `.md` в новом дереве (кроме `docs/`) ≥ ожидаемого канона; ничего уникального не осталось только в `Obsidian/`.

- [ ] **Step 2: Удалить `Obsidian/`**

- [ ] **Step 3: Финальный чеклист из spec**

- [ ] нет `Obsidian/`
- [ ] тематические папки + Title Case
- [ ] `_Index` везде + `Home` на хабы
- [ ] шаблонные секции на контентных заметках
- [ ] нет парных дублей темы
- [ ] аттачи в `_attachments`; `attachmentFolderPath` верный
- [ ] мусор в `Archive/`

- [ ] **Step 4: Короткий отчёт пользователю** — что слито, что в Archive, где были сомнения

---

## Self-review (plan vs spec)

| Spec requirement | Task |
|------------------|------|
| Карта папок в корне | 1 |
| Title Case + шаблон A | 2–5 |
| Merge дублей | 6 |
| Attachments + app.json | 1, 7 |
| Home + _Index | 1, 8 |
| Archive сирот | 8 |
| Удаление Obsidian/ | 9 |
| Не переписывать математику | Global Constraints |
| Спорные файлы с пользователем | Global Constraints + Task 6 |

Placeholders: нет TBD. Git commit steps намеренно отсутствуют (нет репозитория).
