# Lamoda Seller Academy RAG Agent

RAG-консультант по публичной базе знаний Lamoda Seller Academy. Проект собирает статьи с сайта академии, строит гибридный поиск по текстам и подключает retrieval как инструмент в ReAct-like цикл агента.

Решение сделано в одном Jupyter Notebook: без внешней векторной БД, без LangGraph/ADK и без готовых агентных фреймворков. Индекс хранится в памяти Python.

## Что внутри

- Парсер публичной базы знаний Lamoda Seller Academy.
- Сохранение найденных статей в Markdown-формате.
- Разбиение документов на чанки с overlap.
- Sparse retrieval на `TfidfVectorizer`.
- Dense retrieval на `intfloat/multilingual-e5-small`.
- Смешивание результатов через Reciprocal Rank Fusion.
- ReAct-like агент с tool-вызовом `search_knowledge_base(query)`.
- Финальный ответ агента со ссылками на использованные источники.

## Результат сохраненного прогона

- Найдено 119 URL.
- Успешно обработано 37 статей.
- Построено 164 текстовых чанка.
- Sparse-матрица: 164 чанка x 16078 TF-IDF признаков.
- Dense embeddings: 164 чанка x 384 признака.

Эти числа относятся к сохраненному запуску ноутбука. При повторном запуске они могут отличаться, если структура сайта Lamoda Seller Academy изменилась.

## Архитектура

1. **Parsing**
   - старт с `https://academy.lamoda.ru/`;
   - сбор ссылок на разделы и статьи внутри `/articles/`;
   - извлечение заголовка и основного текстового блока статьи;
   - пропуск страниц без ожидаемого контейнера, чтобы парсер не падал на нерелевантных URL.

2. **Indexing**
   - очистка текста;
   - разбиение на чанки по 1000 символов с overlap 150;
   - подготовка metadata: title, source URL, chunk id.

3. **Retrieval**
   - sparse search: TF-IDF, n-grams `(1, 2)`;
   - dense search: multilingual E5 embeddings;
   - объединение выдачи sparse и dense поиска через Reciprocal Rank Fusion.

4. **Agent**
   - агент получает вопрос пользователя;
   - делает tool-вызов `search_knowledge_base(query)`;
   - формирует ответ только на основе найденных фрагментов;
   - добавляет ссылки на источники.

## Запуск

```bash
pip install -r requirements.txt
```

Дальше открыть ноутбук:

```text
notebooks/Lamoda_Seller_Academy_RAG_Agent.ipynb
```

Для агентной части нужен OpenRouter API key. Ключ вводится интерактивно через `getpass` и не сохраняется в ноутбуке.

## Стек

- Python
- requests
- BeautifulSoup
- scikit-learn
- sentence-transformers
- OpenAI SDK
- OpenRouter
- NumPy

## Ограничения и развитие

- Индекс строится как статический слепок сайта, без отслеживания изменений.
- Парсер завязан на текущую HTML-структуру страниц.
- Для улучшения качества retrieval можно добавить reranking, расширить multi-query поиск и собрать небольшой eval-набор вопросов.
- Для production-версии логично вынести индексацию в отдельный pipeline, добавить мониторинг качества и хранение индекса.
