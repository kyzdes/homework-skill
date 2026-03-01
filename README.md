# review-homework-skill

Claude Code skill для оценки студенческих домашних заданий по курсу "Продуктовый и диджитал-маркетинг".

## Что делает

- Анализирует лекции (PDF) через vision и создаёт методологию оценки
- Извлекает контент из PDF-презентаций студентов (text + vision fallback для image-based PDF)
- Оценивает каждую работу по 3 критериям с развёрнутыми комментариями
- Генерирует Excel-отчёт с оценками, цветовой кодировкой и статистикой

## Установка

### Локальная установка

```bash
mkdir -p ~/.claude/skills/review-homework
cp SKILL.md ~/.claude/skills/review-homework/SKILL.md
```

### Зависимости

```bash
pip3 install pdfminer.six pymupdf openpyxl
```

> `pdfminer.six` — основной инструмент извлечения текста из PDF (корректно обрабатывает ToUnicode CMap и Type3 шрифты из Figma/Canva экспорта).
> `pymupdf` — проверка наличия шрифтов в PDF + рендеринг страниц в PNG для vision fallback.
> `openpyxl` — генерация Excel-отчёта.
> Скрипты автоматически устанавливают недостающие пакеты при запуске.

## Использование

```
/review-homework          # основной workflow (оценка работ)
/review-homework model    # управление vision-моделью OpenRouter
```

Skill проведёт через 7 шагов (+ Шаг 0 — проверка конфига OpenRouter):

1. Вставьте текст задания
2. Укажите путь к лекциям
3. Подтвердите методологию
4. Укажите путь к работам студентов
5. Автоматическое извлечение контента из PDF
6. Автоматическая оценка (2 прохода)
7. Генерация Excel-отчёта

## Критерии оценки (Версия A)

| Критерий | Вес | Подаспекты |
|----------|-----|------------|
| К1. Полнота раскрытия | 40% | Бизнес-цель, декомпозиция, каналы, цели подканалов |
| К2. Согласованность | 30% | Связи между уровнями, непротиворечивость |
| К3. Применение знаний | 30% | Фреймворки, метрики, диджитал-концепции, терминология |

Шкала: 1-10. Порог "не зачтено": средний < 4.0 или любой критерий < 3.

## Структура репозитория

```
review-homework-skill/
├── SKILL.md                # Основной файл skill'а
├── README.md               # Этот файл
├── metodology_v_a.md       # Версия A методологии (3 критерия)
├── metodology_v_b.md       # Версия B методологии (5 критериев, расширенная)
└── examples/
    └── .gitkeep            # Примеры выходных файлов
```

## OpenRouter Integration (Vision API)

Для image-only PDF (сканы, Canva без текстового слоя) skill может использовать бесплатные vision-модели через OpenRouter вместо Claude vision. Это значительно экономит токены — base64 изображения не попадают в контекст Claude.

### Настройка

```
/review-homework model
```

Команда покажет список бесплатных vision-моделей OpenRouter. Выберите модель и введите API-ключ.

### Получение API-ключа

1. Зарегистрируйтесь на [openrouter.ai](https://openrouter.ai)
2. Перейдите в [Keys](https://openrouter.ai/keys)
3. Создайте новый ключ

### Конфигурация

Конфиг хранится в `~/.claude/skills/review-homework/config.json` (не в git):

```json
{
  "openrouter_api_key": "sk-or-v1-...",
  "vision_model": "google/gemma-3-27b-it:free",
  "vision_model_name": "Google: Gemma 3 27B (free)"
}
```

### Fallback chain

```
PDF → pdfminer.six text extraction
  ├─ TEXT_OK → use text
  └─ NEED_VISION → render PNGs (pymupdf)
       → OpenRouter vision API (free model)
           ├─ VISION_OK → use extracted text
           ├─ VISION_PARTIAL → OpenRouter text + Claude vision for failed pages
           └─ VISION_FAIL / NO_CONFIG → Claude vision (Read tool)
```

Без настройки OpenRouter skill работает как раньше — Claude vision через Read tool.

## PDF Processing

Skill использует `pdfminer.six` для извлечения текста и `pymupdf` для вспомогательных операций:

1. `pymupdf` — быстрая проверка: есть ли шрифты в PDF? (0 шрифтов = image-only → сразу vision)
2. `pdfminer.six` `extract_text()` — основное извлечение текста (корректно обрабатывает ToUnicode CMap и Type3 шрифты из Figma/Canva экспорта)
3. Проверка качества — наличие кириллических символов
4. Если текст не извлекается (image-only PDF) → fallback на PyMuPDF рендеринг страниц в PNG + Claude vision

Большинство студенческих PDF (Google Slides, PowerPoint, Figma) содержат текстовый слой — vision не нужен. Fallback срабатывает только для сканов и image-only PDF.

## Excel-отчёт

Формат выходного файла: `review_<hw_name>_<YYYY-MM-DD>.xlsx`

Включает:
- Заголовок и метаданные (дата, версия методологии)
- Оценки по каждому критерию с комментариями
- Итоговую оценку и статус (зачтено/не зачтено)
- Общий комментарий к каждой работе
- Цветовую кодировку (красный → зелёный)
- Статистику по группе (среднее, медиана, мин, макс, % неудовл.)
