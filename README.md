# censor-user2

Бинарный классификатор запросов на русском: пропустить запрос или заблокировать.
Дообученная модель [`deepvk/USER2-base`](https://huggingface.co/deepvk/USER2-base)
(ModernBERT, 150M параметров), `ModernBertForSequenceClassification`, 2 класса.

| Класс | Значение |
|---|---|
| `0` | нейтральный запрос — пропустить |
| `1` | нежелательный запрос — заблокировать |

## Файлы

| Файл | Что это |
|---|---|
| `model.safetensors`, `config.json` | веса и конфиг модели (веса — через Git LFS) |
| `tokenizer.json`, `tokenizer_config.json`, `special_tokens_map.json` | токенизатор |
| `config_censor.json` | порог и настройки инференса |
| `SHA256SUMS` | контрольные суммы файлов |

Веса лежат в Git LFS: перед клонированием — `git lfs install`, в уже склонированном репозитории — `git lfs pull`.

## Инференс

- запрос приводится к **нижнему регистру** (`text.lower()`);
- длина — до **128 токенов**, дальше обрезается;
- блок, если вероятность класса `1` ≥ **0.869**.

vLLM 0.8.5 или новее:

```bash
vllm serve ./censor-user2 --served-model-name censor --runner pooling --convert classify \
  --max-model-len 512 --gpu-memory-utilization 0.05
```

```
POST /classify
{"model": "censor", "input": ["текст запроса в нижнем регистре"], "truncate_prompt_tokens": 128}

→ {"data": [{"index": 0, "probs": [0.05, 0.95], ...}], ...}
```

`data[0].probs[1]` — вероятность класса `1`.

## Лицензия

Apache 2.0, как у базовой модели `deepvk/USER2-base`.
