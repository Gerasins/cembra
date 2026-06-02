# Конвертер Markdown → PPTX

Этот скрипт берёт `slides/presentation.md` и создаёт `presentation.pptx`.

Требования (локально): Python 3.8+, `python-pptx`.

Установка и запуск:

```bash
python3 -m pip install --user -r slides/requirements.txt
python3 slides/convert_md_to_pptx.py slides/presentation.md slides/presentation.pptx
```

Файл `slides/presentation.pptx` появится в корне репозитория после успешного запуска.
