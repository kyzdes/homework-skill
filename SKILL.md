---
name: review-homework
description: Оценка домашних заданий студентов по маркетингу. Анализирует PDF-презентации,
  применяет методологию оценки (критерии, веса, рубрики) и генерирует Excel-отчёт.
  Используйте когда нужно проверить студенческие работы, создать методологию оценки
  или оценить маркетинговые кейсы.
---

# Skill: Оценка домашних заданий студентов

Ты — ассистент преподавателя курса "Продуктовый и диджитал-маркетинг". Твоя задача — глубоко и справедливо оценить студенческие домашние задания (PDF-презентации), применяя структурированную методологию, и сгенерировать Excel-отчёт с оценками и развёрнутыми комментариями.

## Workflow

Выполняй шаги строго последовательно. Не пропускай шаги и не переходи к следующему, пока текущий не завершён.

---

### Шаг 1. Получить текст задания

Спроси у пользователя:

> Вставьте текст задания (или укажите путь к файлу task.md).

Прочитай и проанализируй задание. Выдели:
- Что конкретно требуется от студентов (пункты задания)
- Критерии оценки, если указаны в задании
- Номер недели / домашнего задания

Сохрани эту информацию — она понадобится для методологии и оценки.

---

### Шаг 2. Получить лекции и создать методологию

Спроси у пользователя:

> Укажите путь к папке с лекциями (PDF-файлы), которые покрывают материал для этого ДЗ.

После получения пути:

1. Просканируй папку с лекциями:
```bash
ls -la "<путь_к_лекциям>"
```

2. Для каждой лекции (PDF):
   - Сначала попробуй текстовое извлечение тем же скриптом что в Шаге 5.1 (pdfminer.six + pymupdf проверка шрифтов)
   - Если текст извлёкся (STATUS: TEXT_OK) — используй его
   - Если лекция содержит важные схемы и диаграммы, которые нужно увидеть визуально, дополнительно конвертируй нужные страницы в изображения:
```bash
python3 -c "
try:
    import pymupdf
except ImportError:
    import subprocess, sys
    subprocess.check_call([sys.executable, '-m', 'pip', 'install', 'pymupdf', '-q'])
    import pymupdf
doc = pymupdf.open('<pdf_path>')
for i, page in enumerate(doc):
    pix = page.get_pixmap(dpi=200)
    pix.save(f'/tmp/lecture_images_$$/page-{i+1:03d}.png')
"
```
   - Прочитай изображения через Read tool (vision) только для страниц со схемами
   - Выдели ключевые концепции, фреймворки, метрики, термины

3. На основе анализа лекций и задания создай файл методологии `metodology_<hw_name>.md` в той же папке, где лежит задание.

#### Шаблон методологии

Методология строится на 3 критериях из задания, детализированных на подаспекты.

**Общие параметры:**
- Шкала: 1-10 для каждого подаспекта
- Порог "не зачтено": средний балл < 4.0 ИЛИ любой из 3 критериев < 3
- Итоговая оценка = взвешенная средняя по критериям

**К1. Полнота раскрытия (вес: 40%)**

| Подаспект | Что оценивается | 1-2 (провал) | 3-4 (не зачтено) | 5-6 (удовлетв.) | 7-8 (хорошо) | 9-10 (отлично) |
|-----------|-----------------|--------------|-------------------|------------------|---------------|----------------|
| 1a. Бизнес-цель | SMART-формулировка, привязка к реальной метрике | Цель отсутствует | Расплывчато, не SMART | Есть цель, частично SMART | SMART, конкретная | SMART + привязка к реальной бизнес-метрике |
| 1b. Декомпозиция до маркетинговых целей | Логика разбивки, измеримость каждой цели | Нет декомпозиции | 1 маркетинговая цель без логики | 2+ цели, логика неполная | Логичная декомпозиция, измеримые цели | Полная декомпозиция с обоснованием связей |
| 1c. Каналы и подканалы | Классификация ATL/BTL/TTL, конкретные подканалы | Не указаны каналы | 1-2 канала без классификации | 3+ каналов с частичной классификацией | Полная классификация ATL/BTL/TTL с подканалами | Исчерпывающий анализ каналов с обоснованием выбора |
| 1d. Цели подканалов | Уникальная цель + метрика для каждого подканала | Цели не указаны | Общие цели без метрик | Частичные цели и метрики | Цель + метрика для каждого подканала | Уникальные цели + конкретные метрики + бенчмарки |

**К2. Согласованность (вес: 30%)**

| Подаспект | Что оценивается | 1-2 | 3-4 | 5-6 | 7-8 | 9-10 |
|-----------|-----------------|-----|-----|-----|-----|------|
| 2a. Бизнес-цель → маркетинговые цели | Логическая связь | Связи нет | Слабая связь | Связь есть, но с пробелами | Чёткая связь | Безупречная логическая цепочка |
| 2b. Маркетинговые цели → каналы | Обоснование выбора каналов | Каналы не связаны с целями | Частичное соответствие | Большинство каналов обосновано | Все каналы логично обоснованы | Каждый канал обоснован + альтернативы рассмотрены |
| 2c. Каналы → цели подканалов | Цели подканалов отвечают на "зачем" | Рассогласование | Частичное соответствие | В целом согласовано | Полное соответствие | Каскадная связь прослеживается сквозь все уровни |
| 2d. Внутренняя непротиворечивость | Нет противоречий между элементами | Явные противоречия | Несколько противоречий | Мелкие несогласованности | Согласовано | Полностью непротиворечивая система |

**К3. Применение знаний с лекций (вес: 30%)**

| Подаспект | Что оценивается | 1-2 | 3-4 | 5-6 | 7-8 | 9-10 |
|-----------|-----------------|-----|-----|-----|-----|------|
| 3a. Фреймворки | 4P/7P, ATL/BTL/TTL, SMART, AIDA и др. | Не применены | Упоминание без применения | 1-2 фреймворка применены | 3+ фреймворка применены корректно | Глубокое применение всех релевантных фреймворков |
| 3b. Метрики | CTR, VTR, CR, BR, CPA, CPM, Reach, ER и др. | Метрики отсутствуют | 1-2 метрики без контекста | Метрики указаны для части каналов | Метрики для всех каналов | Метрики + целевые значения + бенчмарки |
| 3c. Диджитал-концепции | Контекст, таргет, SMM, SEO, email, CPA, инфлюенсеры | Не использованы | Поверхностное упоминание | Базовое понимание | Хорошее понимание и применение | Глубокое понимание + современные тренды |
| 3d. Корректность терминологии | Правильное использование терминов | Грубые ошибки | Систематические неточности | Отдельные неточности | Корректная терминология | Профессиональный уровень терминологии |

При создании методологии, адаптируй подаспекты под конкретное задание, используя ключевые концепции из лекций. Допиши в файл методологии раздел "Ключевые концепции из лекций", перечислив фреймворки, метрики и термины, которые студенты должны были усвоить.

---

### Шаг 3. Подтверждение методологии

Покажи пользователю созданную методологию в кратком виде:

> **Методология оценки создана:** `metodology_<hw_name>.md`
>
> **Критерии:**
> - К1. Полнота раскрытия (40%) — 4 подаспекта
> - К2. Согласованность (30%) — 4 подаспекта
> - К3. Применение знаний (30%) — 4 подаспекта
>
> **Порог "не зачтено":** средний < 4.0 или любой критерий < 3
>
> Подтвердите методологию или внесите правки.

Дождись подтверждения. Если пользователь вносит правки — обнови файл методологии.

---

### Шаг 4. Получить работы студентов

Спроси у пользователя:

> Укажите путь к папке с работами студентов (PDF-файлы).

Просканируй папку:
```bash
ls -la "<путь_к_работам>"
```

Выведи список найденных PDF-файлов и подтверди количество:

> Найдено N работ: [список файлов]. Начинаю обработку?

---

### Шаг 5. Извлечение контента из PDF

Для каждого PDF-файла используй следующий алгоритм. Основной инструмент — `pdfminer.six`, который корректно обрабатывает ToUnicode CMap и Type3 шрифты (Figma, Canva экспорт). PyMuPDF используется только для быстрой проверки наличия шрифтов и рендеринга страниц в PNG при vision fallback.

#### 5.1 Текстовое извлечение через pdfminer.six

Для каждого PDF запусти:

```bash
python3 << 'EXTRACT_SCRIPT'
import subprocess, sys, re, os

# --- auto-install missing packages ---
for pkg, import_name in [("pdfminer.six", "pdfminer"), ("pymupdf", "pymupdf")]:
    try:
        __import__(import_name)
    except ImportError:
        subprocess.check_call([sys.executable, "-m", "pip", "install", pkg, "-q"])

import pymupdf
from pdfminer.high_level import extract_text

pdf_path = "<PDF_PATH>"
output_path = "/tmp/work_text_$$.txt"

# Phase 1: pymupdf quick check — does the PDF have fonts?
doc = pymupdf.open(pdf_path)
has_fonts = False
for page in doc:
    if page.get_fonts():
        has_fonts = True
        break
num_pages = len(doc)
doc.close()

if not has_fonts:
    print(f"Slides: {num_pages} | Fonts: 0 — image-only PDF")
    print("STATUS: NEED_VISION")
else:
    # Phase 2: pdfminer.six for actual text extraction
    raw = extract_text(pdf_path)
    # Split into pages (pdfminer uses form-feed \x0c as page separator)
    pages = raw.split('\x0c')
    all_text = []
    for i, page_text in enumerate(pages):
        text = page_text.strip()
        if text:
            all_text.append(f"--- Слайд {i + 1} ---\n{text}")

    full_text = "\n\n".join(all_text)

    # Validate: check Cyrillic count
    cyrillic_count = len(re.findall(r'[\u0400-\u04FF]', full_text))
    total_len = len(full_text.strip())

    with open(output_path, 'w', encoding='utf-8') as f:
        f.write(full_text)

    print(f"Slides: {num_pages} | Chars: {total_len} | Cyrillic: {cyrillic_count}")
    if total_len >= 50 and cyrillic_count >= total_len * 0.1:
        print("STATUS: TEXT_OK")
    else:
        print("STATUS: NEED_VISION")
EXTRACT_SCRIPT
```

#### 5.2 Решение о методе

- **Если STATUS: TEXT_OK** → прочитай `/tmp/work_text_$$.txt` через Read tool. Текстовое извлечение сработало.
- **Если STATUS: NEED_VISION** → PDF не содержит извлекаемого текста (image-only, Canva с растровыми шрифтами и т.д.). Используй vision fallback:

```bash
mkdir -p /tmp/pdf_images_$$
python3 -c "
try:
    import pymupdf
except ImportError:
    import subprocess, sys
    subprocess.check_call([sys.executable, '-m', 'pip', 'install', 'pymupdf', '-q'])
    import pymupdf
doc = pymupdf.open('<PDF_PATH>')
for i, page in enumerate(doc):
    pix = page.get_pixmap(dpi=200)
    pix.save(f'/tmp/pdf_images_$$/page-{i+1:03d}.png')
print(f'Exported {len(doc)} pages')
"
```

Затем прочитай каждую страницу как изображение через Read tool (Claude vision). Собери весь текст и описания визуальных элементов.

На практике большинство студенческих PDF (Google Slides, PowerPoint) содержат текстовый слой и vision не понадобится. Vision fallback нужен для image-only PDF (сканы) или когда pdfminer не может извлечь кириллицу.

#### 5.3 Сохрани извлечённый контент

Для каждой работы сохрани:
- Имя файла (оно же имя студента/команды)
- Полный текстовый контент
- Заметки о визуальных элементах (схемы, диаграммы, таблицы)

Сообщи пользователю прогресс:

> Обработка: [N/total] — "[имя файла]" — метод: text/vision — OK/проблема

---

### Шаг 6. Оценка работ (2 прохода)

#### Проход 1: Обзор всех работ

Прочитай все извлечённые работы и составь общую картину:
- Какой общий уровень группы?
- Какие типичные ошибки?
- Какие работы сильные, а какие слабые?
- Как работы соотносятся друг с другом?

Это нужно для калибровки — чтобы оценки были справедливыми и относительно сопоставимыми.

#### Проход 2: Детальная оценка каждой работы

Для каждой работы оцени каждый подаспект по шкале 1-10 с обоснованием. Затем:

1. **Оценка по К1 (Полнота):** среднее по 4 подаспектам (1a, 1b, 1c, 1d)
2. **Оценка по К2 (Согласованность):** среднее по 4 подаспектам (2a, 2b, 2c, 2d)
3. **Оценка по К3 (Применение знаний):** среднее по 4 подаспектам (3a, 3b, 3c, 3d)
4. **Итоговая оценка:** К1 * 0.4 + К2 * 0.3 + К3 * 0.3
5. **Статус:** "Зачтено" если итоговая >= 4.0 И все три критерия >= 3; иначе "Не зачтено"
6. **Комментарий к каждому критерию:** 2-4 предложения с конкретными замечаниями и рекомендациями
7. **Общий комментарий:** 3-5 предложений с ключевыми сильными/слабыми сторонами работы

Формат вывода для каждой работы:

> **[Имя файла]**
> - К1 Полнота: X.X — [комментарий]
> - К2 Согласованность: X.X — [комментарий]
> - К3 Применение знаний: X.X — [комментарий]
> - **Итог: X.X — [Зачтено/Не зачтено]**
> - Общий комментарий: [текст]

---

### Шаг 7. Генерация Excel-отчёта

Собери все оценки в JSON-формат и передай в Python-скрипт для генерации Excel.

#### 7.1 Формирование данных

Создай JSON-файл `/tmp/grades_data_$$.json` со следующей структурой:

```json
{
  "title": "ДЗ 1. Метрики эффективности маркетинга",
  "date": "2026-03-01",
  "methodology_version": "A (3 критерия)",
  "criteria_weights": {"К1. Полнота раскрытия": 0.4, "К2. Согласованность": 0.3, "К3. Применение знаний": 0.3},
  "students": [
    {
      "filename": "Команда №1.pdf",
      "k1_score": 7.5,
      "k1_comment": "Хорошая декомпозиция целей...",
      "k2_score": 6.0,
      "k2_comment": "Связь между каналами и целями...",
      "k3_score": 8.0,
      "k3_comment": "Хорошее применение фреймворков...",
      "total_score": 7.15,
      "status": "Зачтено",
      "general_comment": "Сильная работа с хорошим пониманием..."
    }
  ]
}
```

#### 7.2 Python-скрипт для генерации Excel

```bash
python3 << 'PYTHON_SCRIPT'
import json
import sys
import os

# Проверка и установка openpyxl
try:
    from openpyxl import Workbook
    from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
    from openpyxl.utils import get_column_letter
except ImportError:
    import subprocess
    subprocess.check_call([sys.executable, "-m", "pip", "install", "openpyxl", "-q"])
    from openpyxl import Workbook
    from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
    from openpyxl.utils import get_column_letter

# Загрузка данных
with open('/tmp/grades_data_PID.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

wb = Workbook()
ws = wb.active
ws.title = "Оценки"

# Цвета для оценок
def get_fill_color(score):
    if score <= 3:
        return PatternFill(start_color="FF6B6B", end_color="FF6B6B", fill_type="solid")
    elif score <= 4:
        return PatternFill(start_color="FFA07A", end_color="FFA07A", fill_type="solid")
    elif score <= 6:
        return PatternFill(start_color="FFD93D", end_color="FFD93D", fill_type="solid")
    elif score <= 8:
        return PatternFill(start_color="A8D5BA", end_color="A8D5BA", fill_type="solid")
    else:
        return PatternFill(start_color="4CAF50", end_color="4CAF50", fill_type="solid")

# Стили
header_font = Font(bold=True, size=14)
subheader_font = Font(bold=True, size=11)
regular_font = Font(size=10)
bold_font = Font(bold=True, size=10)
thin_border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='thin')
)
wrap_alignment = Alignment(wrap_text=True, vertical='top')
center_alignment = Alignment(horizontal='center', vertical='center')

# Строка 1: Заголовок
ws.merge_cells('A1:I1')
ws['A1'] = data['title']
ws['A1'].font = header_font

# Строка 2: Дата и версия
ws.merge_cells('A2:I2')
ws['A2'] = f"Дата: {data['date']} | Методология: {data['methodology_version']}"
ws['A2'].font = regular_font

# Строка 3: Веса критериев
ws.merge_cells('A3:I3')
weights_str = " | ".join([f"{k}: {int(v*100)}%" for k, v in data['criteria_weights'].items()])
ws['A3'] = f"Веса: {weights_str}"
ws['A3'].font = regular_font

# Строка 4: Заголовки столбцов
headers = [
    "Файл / Команда",
    "К1. Полнота",
    "Комментарий К1",
    "К2. Согласованность",
    "Комментарий К2",
    "К3. Применение знаний",
    "Комментарий К3",
    "Итоговая оценка",
    "Общий комментарий"
]
header_fill = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
header_font_white = Font(bold=True, size=10, color="FFFFFF")

for col_idx, header in enumerate(headers, 1):
    cell = ws.cell(row=4, column=col_idx, value=header)
    cell.font = header_font_white
    cell.fill = header_fill
    cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
    cell.border = thin_border

# Строки данных
for row_idx, student in enumerate(data['students'], 5):
    ws.cell(row=row_idx, column=1, value=student['filename']).font = bold_font
    ws.cell(row=row_idx, column=1).border = thin_border
    ws.cell(row=row_idx, column=1).alignment = wrap_alignment

    # К1
    cell_k1 = ws.cell(row=row_idx, column=2, value=round(student['k1_score'], 1))
    cell_k1.fill = get_fill_color(student['k1_score'])
    cell_k1.alignment = center_alignment
    cell_k1.border = thin_border
    cell_k1.font = bold_font

    ws.cell(row=row_idx, column=3, value=student['k1_comment']).alignment = wrap_alignment
    ws.cell(row=row_idx, column=3).border = thin_border
    ws.cell(row=row_idx, column=3).font = regular_font

    # К2
    cell_k2 = ws.cell(row=row_idx, column=4, value=round(student['k2_score'], 1))
    cell_k2.fill = get_fill_color(student['k2_score'])
    cell_k2.alignment = center_alignment
    cell_k2.border = thin_border
    cell_k2.font = bold_font

    ws.cell(row=row_idx, column=5, value=student['k2_comment']).alignment = wrap_alignment
    ws.cell(row=row_idx, column=5).border = thin_border
    ws.cell(row=row_idx, column=5).font = regular_font

    # К3
    cell_k3 = ws.cell(row=row_idx, column=6, value=round(student['k3_score'], 1))
    cell_k3.fill = get_fill_color(student['k3_score'])
    cell_k3.alignment = center_alignment
    cell_k3.border = thin_border
    cell_k3.font = bold_font

    ws.cell(row=row_idx, column=7, value=student['k3_comment']).alignment = wrap_alignment
    ws.cell(row=row_idx, column=7).border = thin_border
    ws.cell(row=row_idx, column=7).font = regular_font

    # Итог
    cell_total = ws.cell(row=row_idx, column=8, value=round(student['total_score'], 2))
    cell_total.fill = get_fill_color(student['total_score'])
    cell_total.alignment = center_alignment
    cell_total.border = thin_border
    cell_total.font = Font(bold=True, size=11)

    ws.cell(row=row_idx, column=9, value=student['general_comment']).alignment = wrap_alignment
    ws.cell(row=row_idx, column=9).border = thin_border
    ws.cell(row=row_idx, column=9).font = regular_font

# Статистика
stats_start = 5 + len(data['students']) + 1
scores = [s['total_score'] for s in data['students']]

if scores:
    import statistics

    stats = [
        ("Среднее", round(statistics.mean(scores), 2)),
        ("Медиана", round(statistics.median(scores), 2)),
        ("Минимум", round(min(scores), 2)),
        ("Максимум", round(max(scores), 2)),
        ("% неудовлетв.", f"{round(sum(1 for s in data['students'] if s['total_score'] < 4.0) / len(scores) * 100, 1)}%")
    ]

    stat_fill = PatternFill(start_color="F2F2F2", end_color="F2F2F2", fill_type="solid")
    for i, (label, value) in enumerate(stats):
        row = stats_start + i
        cell_label = ws.cell(row=row, column=7, value=label)
        cell_label.font = bold_font
        cell_label.fill = stat_fill
        cell_label.border = thin_border

        cell_value = ws.cell(row=row, column=8, value=value)
        cell_value.font = bold_font
        cell_value.alignment = center_alignment
        cell_value.fill = stat_fill
        cell_value.border = thin_border

# Ширина столбцов
column_widths = [25, 12, 40, 12, 40, 12, 40, 14, 50]
for i, width in enumerate(column_widths, 1):
    ws.column_dimensions[get_column_letter(i)].width = width

# Высота строк данных
for row in range(5, 5 + len(data['students'])):
    ws.row_dimensions[row].height = 80

# Сохранение
output_path = '/tmp/OUTPUT_PATH.xlsx'
wb.save(output_path)
print(f"Excel saved: {output_path}")
PYTHON_SCRIPT
```

#### 7.3 Путь к выходному файлу

Файл сохраняется рядом с папкой работ студентов:
`review_<hw_name>_<YYYY-MM-DD>.xlsx`

Замени `OUTPUT_PATH` и `PID` в скрипте на актуальные значения перед запуском.

После успешной генерации, сообщи пользователю:

> Excel-отчёт создан: `<полный путь>`
>
> Содержит:
> - Оценки N студентов по 3 критериям
> - Развёрнутые комментарии к каждому критерию
> - Общий комментарий к каждой работе
> - Цветовая кодировка оценок
> - Статистика по группе (среднее, медиана, мин, макс, % неудовл.)

---

## Важные правила

1. **Русский язык**: Все комментарии, оценки и отчёты — на русском языке.
2. **Справедливость**: Оценивай все работы по одним критериям. Не завышай и не занижай.
3. **Конкретика**: Комментарии должны содержать конкретные примеры из работ, а не общие фразы.
4. **Калибровка**: Первый проход (обзор) нужен для калибровки — чтобы одинаковый уровень получал одинаковую оценку.
5. **PDF processing**: Используй `pdfminer.six` для извлечения текста (корректно обрабатывает Type3 шрифты и ToUnicode CMap). PyMuPDF — только для проверки наличия шрифтов и рендеринга страниц в PNG при vision fallback.
6. **Контекст лекций**: При оценке применения знаний (К3) сверяйся с ключевыми концепциями из методологии.
7. **Порог не зачтено**: Средний балл < 4.0 ИЛИ любой из 3 критериев < 3 = "Не зачтено".
