# Obsidian Vault Refactor — Design

Дата: 2026-09-27  
Статус: согласовано в диалоге, ожидает ревью файла

## Цель

Привести хранилище к одному стилю: единая база знаний для учёбы и собесов **без разделения** «курс / интервью». Всё полезное остаётся; дубли сливаются; пустой/разовый мусор уходит в Archive или удаляется после явного ОК.

## Решения (зафиксировано)

| Вопрос | Выбор |
|--------|--------|
| Назначение | Учёба + собес в одних заметках |
| Организация | Плоская карта по темам, без нумерации курса |
| Именование | Title Case с пробелами (`Gradient Descent.md`) |
| Шаблон заметки | Идея → тело/формулы → `## Если спросят` → `## Связанное` |
| Объём работ | Полный проход: структура + шаблон + merge дублей + чистка мусора |
| Подход | Узкие тематические папки в корне vault |

## Архитектура: карта папок

Убрать вложенный слой `Obsidian/`. Заметки живут в корне vault:

```
Home.md
Fundamentals/
Linear Models/
Trees And Ensembles/
Boosting/
Deep Learning/
NLP/
Interpretability/
Fine Tuning/
Probability And Stats/
Math/
Algorithms/
Python/
SQL/
Archive/
_attachments/
  lectures/
docs/                    # этот design/plan — не часть базы знаний Obsidian
```

### Назначение папок

- **Fundamentals** — pipeline, loss (общие), gradient descent, overfitting / bias-variance, общие метрики, likelihood, MLE как общие понятия.
- **Linear Models** — линейная регрессия, регуляризация, Gauss-Markov, логистическая регрессия, sigmoid, cross-entropy, hyperplane, linear classification, classification / multiclass metrics (если тесно с логрегом — допустимо здесь; иначе Fundamentals — выбрать одно место при merge и не дублировать).
- **Trees And Ensembles** — decision tree, information criteria, bootstrap, random forest.
- **Boosting** — AdaBoost, gradient boosting; bias-variance — одна каноническая заметка (в Boosting или Fundamentals, не две).
- **Deep Learning** — backpropagation, distillation, vanishing/exploding gradients.
- **NLP** — tokenization, embedding, RNN, LSTM, attention, self-attention, transformer, beam search.
- **Interpretability** — importance estimation, LIME, SHAP, Grad-CAM.
- **Fine Tuning** — LoRA idea / realization, QLoRA.
- **Probability And Stats** — Bayes, prior/posterior, CLT, descriptive statistics, error types, estimation of shift/scale.
- **Math** — matrix differentiation, quadratic form.
- **Algorithms** — паттерны собеса (binary search, DP, graphs, …); подпапка `Graphs/` допустима.
- **Python**, **SQL** — как сейчас в Coding/собес, без папки «собес».
- **Archive** — разовые ДЗ (`solution_A…`), `Без названия`, прочий не-канон.
- **_attachments** — все картинки; PDF лекций в `_attachments/lectures/`.

В каждой тематической папке (кроме Archive и _attachments) — файл `_Index.md` со списком `[[wiki-links]]` на заметки папки.

`Home.md` — только ссылки на `_Index` хабов + список дыр/TODO. Без дублирования полного оглавления курса.

## Стиль заметки

### Имена

- Имя файла = H1 = Title Case с пробелами, английский.
- Примеры: `Linear Regression Model.md`, `Naive Bayes Classifier.md`, `Union Find.md`.
- Индекс только `_Index.md` (не README, не `_index_basics`).

### Каркас

```markdown
# Title

Кратко идея в 1–3 предложениях (callout опционален).

## …тематические секции…

## Если спросят
- **Зачем:** …
- **Когда ломается:** …
- **Путают с:** …
- **Фраза:** …   <!-- опционально -->

## Связанное
- [[Related Note]]
```

Код — отдельная секция там, где нужен (алгоритмы, LoRA), не обязателен везде.

### Язык и ссылки

- Заголовки и имена файлов — EN Title Case.
- Тело — по-русски (как сейчас).
- Только wiki-links `[[Note]]`. Markdown-ссылки на `.md` файлы не использовать.
- Obsidian: `alwaysUpdateLinks: true` сохранить.

### Чего не делаем с содержимым

- **Не переписываем математику** — не меняем формулы, выводы, обозначения «для красоты» и не пересказываем лекции своими словами. Меняем оболочку (папка, имя, секции шаблона), при merge — аккуратно переносим уникальные куски из дубля в канон.

## Миграция

### Порядок

1. Создать новую карту папок в корне (рядом со старым `Obsidian/`).
2. Перенести заметки → переименовать в Title Case → дописать недостающие секции шаблона.
3. Слить дубли в одну каноническую заметку.
4. Обновить все `[[links]]`, `Home.md`, `_Index.md`.
5. Собрать аттачи в `_attachments/`, PDF в `_attachments/lectures/`; починить `attachmentFolderPath` в `.obsidian/app.json` на `_attachments` (сейчас битая кодировка «вложенные файлы»).
6. Удалить старое дерево `Obsidian/` после проверки, что всё перенесено.
7. Сироты → `Archive/`; удаление только после явного ОК пользователя.

### Правила merge дублей

- База = более полная / курсовая версия с формулами.
- Из легаси `ML/` забирать уникальное (таблицы «когда что», практические ремарки вроде Adam) в канон.
- После merge старый файл удалить (не оставлять редирект-заглушки).
- `algorithms.md` (свалка) — разобрать по тематическим заметкам или целиком в Archive, не оставлять как хаб-свалку.

Известные пары (не исчерпывающий список; при реализации пройтись grep’ом):

| Легаси / A | Курс / B | Канон (ориентир) |
|------------|----------|------------------|
| `ML/metrics.md` | Classification / Multiclass metrics | одна или две заметки метрик без тройного дубля |
| `ML/loss-functions.md` | Basic Loss, Cross-Entropy | Fundamentals + Linear Models по смыслу |
| `ML/overfitting-bias-variance.md` | Bias-variance tradeoff | одна заметка |
| `ML/gradient-descent.md` | (нет прямого курса) | Fundamentals / Gradient Descent |
| `ML/pipeline.md` | — | Fundamentals / Pipeline |
| `ML/algorithms.md` | тематические заметки | разобрать / Archive |

### Мусор и исключения

- `Grad-CAM`, `Gradient problems` — **не пустые**; пометки в старых индексах устарели; оставляем и приводим к шаблону.
- `Без названия.md`, `solution_A_optimal_prediction.md` → `Archive/` (удаление — только по ОК).
- `.idea/` — не часть базы; не трогать или добавить в ignore при появлении git; не переносить в тематические папки.
- `docs/` — спецификации агента; не линковать из `Home` как учебный контент.

### Вложения

- Единый корень: `_attachments/`.
- Осмысленные имена картинок там, где связь с заметкой очевидна (`Self Attention Encoder.png`).
- PDF лекций: `_attachments/lectures/` с понятными именами (`Basics 02 Linear Regression.pdf`).
- Ссылки `![[image.png]]` обновить после переноса (Obsidian / ручная проверка).

## Критерии готовности

- Нет папки `Obsidian/` со старым деревом.
- Все учебные заметки в тематических папках из карты; имена Title Case.
- У каждой тематической папки есть `_Index.md`; `Home.md` указывает на хабы.
- Заметки имеют секции `Если спросят` и `Связанное` (кроме Archive и чисто навигационных `_Index` / `Home`).
- Нет парных дублей по одной теме в двух местах.
- Аттачи в `_attachments`; `attachmentFolderPath` = `_attachments`.
- Разовый мусор в `Archive/` или удалён по ОК.

## Вне скоупа

- Новые темы / наполнение дыр контентом с нуля.
- Смена математических обозначений.
- Плагины Obsidian, темы оформления UI.
- Git-инициализация vault (сейчас репозитория нет) — по желанию отдельно.
