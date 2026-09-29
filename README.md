# LLM Prompt Engineering Experiments: Risk Analysis with Qwen2.5-1.5B
# Эксперименты по промпт‑инжинирингу: анализ рисков с Qwen2.5-1.5B

---

## English

### Overview

A mini‑project demonstrating prompt engineering techniques to extract risks and mitigation measures from a chatbot project description, with a focus on **structured output reliability** and **failure handling**.

Instead of aiming for a “perfect answer,” the experiment intentionally tests how consistently an LLM can produce valid JSON under different prompting strategies (Zero‑shot, Few‑shot, CoT) and languages (RU/EN).

**Model:** `Qwen/Qwen2.5-1.5B-Instruct` (Hugging Face Transformers, Google Colab, T4 GPU).  
**Key artifact:** A validation pipeline that detects non‑JSON outputs and logs them as `PARTIAL` results.

### What we tested

| Technique | Description | Goal |
|----------|-------------|------|
| **Zero‑shot** | Minimal prompt: “list risks and measures” | Baseline behavior |
| **Few‑shot** | Prompt with 2 example pairs “Risk → Mitigation” | Improve structure via pattern |
| **CoT + JSON** | Step‑by‑step reasoning + explicit JSON request | Produce parseable JSON for automation |

Testing was performed in **RU and EN** to evaluate cross‑lingual stability.

### Results summary

| Technique | Lang | Format kept? | Parseable JSON? | Notes |
|--------|------|---------------|-----------------|-------|
| Zero‑shot | RU | No | No | Adds intro text (“Question:… Answer:…”) |
| Zero‑shot | EN | No | No | Extra framing phrases |
| Few‑shot | RU | Partial | No | Pattern partially followed; adds comments/code blocks |
| Few‑shot | EN | Partial | No | Better adherence, but still includes explanations |
| CoT + JSON | RU | Yes (in intent) | ❌ No | Model outputs narrative text instead of JSON |
| CoT + JSON | EN | Yes (in intent) | ❌ No | Same issue: no valid JSON block |

**Validation outcome:** Both RU and EN CoT responses failed JSON parsing (`Expecting value: line 1 column 1`). This indicates that **explicit instructions alone are insufficient** to guarantee structured output.

### Engineering insight: Failure is data

The lack of valid JSON is treated as a meaningful result, not a bug:

- **Added validation layer:** A Python function checks JSON validity and flags non‑parseable responses.
- **Fallback strategy:** Non‑valid JSON responses are marked as `status: "PARTIAL"` and stored with raw text for manual review.
- **Conclusion:** For production pipelines, relying only on prompts is risky. A combination of strict templates, post‑processing (regex extraction), and validation is required.

This approach aligns with **prompt safety** and **AI ethics** practices: predictable outputs reduce hallucinations and support safer integration with downstream systems.

### How to run

1. Open the notebook in [Google Colab](https://colab.research.google.com).
2. Set runtime to T4 GPU: `Runtime → Change runtime type → T4 GPU`.
3. Run cells sequentially: setup → model loading → prompts → validation.
4. Download `results.json` from the Files panel.

### Future work

- Implement regex‑based JSON extraction as a fallback when the model wraps JSON in text.
- Add severity normalization (map all values to: low/medium/high).
- Introduce guardrails (RAG‑based fact‑checking) to reduce hallucinated risks.
- Test larger models (Qwen2.5‑7B, Llama 3) for improved structural consistency.

---

## Русский

### Описание

Мини‑проект по промпт‑инжинирингу, где основной фокус — не просто «получить ответ», а **надёжность структурированного вывода** и **обработка сбоев**.

Вместо поиска «идеального ответа» эксперимент намеренно проверяет, насколько стабильно языковая модель выдаёт валидный JSON при разных техниках промптинга (Zero‑shot, Few‑shot, CoT) и на двух языках (RU/EN).

**Модель:** `Qwen/Qwen2.5-1.5B-Instruct` (Hugging Face Transformers, Google Colab, T4 GPU).  
**Ключевой артефакт:** пайплайн валидации, который обнаруживает не‑JSON ответы и помечает их как `PARTIAL`.

### Что тестировали

| Техника | Описание | Цель |
|---------|----------|------|
| **Zero‑shot** | Минимальный промпт: «выпиши риски и меры» | Проверить базовое поведение модели |
| **Few‑shot** | Промпт с 2 примерами пар «Риск → Мера» | Показать модели паттерн |
| **CoT + JSON** | Пошаговое рассуждение + явный запрос JSON | Получить машиночитаемые данные для автоматизации |

Тестирование проводилось на **русском и английском** для оценки кросс‑языковой стабильности.

### Итоги эксперимента

| Техника | Язык | Формат удержан? | JSON валиден? | Примечания |
|--------|------|------------------|---------------|------------|
| Zero‑shot | RU | Нет | Нет | Добавляет вступление («Вопрос:… Ответ:…») |
| Zero‑shot | EN | Нет | Нет | Лишние вводные фразы |
| Few‑shot | RU | Частично | Нет | Шаблон частично удержан, но добавлены комментарии и блоки кода |
| Few‑shot | EN | Частично | Нет | Лучше держится шаблона, но есть пояснения |
| CoT + JSON | RU | Да (по замыслу) | ❌ Нет | Модель выдаёт повествовательный текст вместо JSON |
| CoT + JSON | EN | Да (по замыслу) | ❌ Нет | Та же проблема: нет валидного JSON‑блока |

**Результат валидации:** и русский, и английский ответы в технике CoT+JSON не прошли валидацию JSON (`Expecting value: line 1 column 1`). Это показывает, что **одних только инструкций в промпте недостаточно**, чтобы гарантировать структурированный вывод.

### Инженерный вывод: сбой — это тоже данные

Отсутствие валидного JSON трактуется как осмысленный результат, а не ошибка:

- **Добавлен слой валидации:** функция на Python проверяет валидность JSON и помечает не‑валидные ответы.
- **Стратегия фоллбэка:** невалидные JSON‑ответы помечаются как `status: "PARTIAL"` и сохраняются с сырым текстом для ручной проверки.
- **Вывод:** для продакшн‑пайплайнов полагаться только на промпт рискованно. Нужна комбинация строгих шаблонов, пост‑обработки (извлечение JSON через regex) и валидации.

Такой подход соответствует практикам **prompt safety** и **этики ИИ**: предсказуемые выводы снижают галлюцинации и делают интеграцию с другими системами безопаснее.

### Как запустить

1. Открой ноутбук в [Google Colab](https://colab.research.google.com).
2. Включи GPU: `Runtime → Change runtime type → T4 GPU`.
3. Запусти ячейки по порядку: настройка → загрузка модели → промпты → валидация.
4. Скачай `results.json` через панель Files слева.

### Идеи для развития

- Добавить извлечение JSON через regex как фоллбэк, если модель оборачивает JSON в текст.
- Внедрить нормализацию severity (привести все значения к: low / medium / high).
- Добавить guardrails: проверку фактов через RAG, чтобы снизить галлюцинации в рисках.
- Протестировать более крупные модели (Qwen2.5‑7B, Llama 3) для лучшей структурной стабильности.

---

## Полезные ссылки / Useful links

- [Hugging Face Transformers](https://huggingface.co/docs/transformers)
- [Prompt Engineering Guide](https://www.promptingguide.ai/techniques)
- [JSON Schema](https://json-schema.org/)

---

**Author / Автор: Maksim
**Date / Дата:** 2026-09-30  
**Status / Статус:** Experiment completed, failure case documented, validation pipeline implemented. / Эксперимент завершён, случай сбоя задокументирован, пайплайн валидации реализован.

