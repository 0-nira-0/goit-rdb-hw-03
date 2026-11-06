# goit-rdb-hw-03

Домашнє завдання 3 дисципліни Relational Databases for Data Science. Повний цикл роботи з даними в PostgreSQL: завантаження через `COPY FROM STDIN`, staging і типізована таблиця, аудит якості, розподіли, аномалії, 12 DQL/EDA-запитів і Reflection.

Автор: Mykyta Oliinyk.

## Датасет

- Назва: NYC TLC Yellow Taxi Trip Records, жовтень 2025 року.
- Офіційне джерело: [TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).
- Файл поїздок: https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2025-10.parquet
- Довідник зон: https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
- Словник даних: [data_dictionary_trip_records_yellow.pdf](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf), редакція від 18 березня 2025 року.

## Вибірка

- У файлі за місяць 4 428 699 рядків і 20 колонок, усі колонки збережено.
- Вибірка: 100 000 рядків, `month_df.sample(n=100_000, random_state=42).sort_index()`, це 2,26% файлу.
- Випадкова вибірка замість `head(100_000)`, бо перші рядки файлу охоплюють лише одну добу й не містять рядків із `payment_type = 0`. Порівняння наведено в розділі 1.3 notebook.
- Після очищення в `hw3_taxi_trips` 100 000 рядків, у `hw3_taxi_zones` 265 зон.

Вибірка зберігається в `data/hw3_taxi_sample.csv` (10,3 МБ, менше за ліміт 100 МБ), довідник зон у `data/hw3_taxi_zones.csv`. Місячний Parquet до репозиторію не входить: notebook завантажує його з офіційного джерела в тимчасову папку системи й під час запуску формує CSV заново. Той самий seed дає той самий файл, доки TLC не опублікує нову версію даних за цей місяць.

## Запуск

Google Colab: відкрийте `hw_t3.ipynb` і виконайте Runtime → Run all. Перша клітинка встановить залежності. У Colab потрібна версія середовища 2026.07 (Python 3.12): Runtime → Change runtime type → Runtime version, бо `pgserver` не має збірок для Python 3.13.

Локально потрібен Python 3.12, бо `pgserver` не має wheel для новіших версій, а в середовищі має бути `pip`:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install jupyter
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1800 hw_t3.ipynb
```

PostgreSQL запускається через `pgserver` із даними в `/tmp/hw3_pg`. Повторний запуск безпечний: таблиці створюються заново через `DROP TABLE IF EXISTS`.

## Структура репозиторію

```text
goit-rdb-hw-03/
├── hw_t3.ipynb   notebook з outputs
├── README.md
├── .gitignore
└── data/
    ├── hw3_taxi_sample.csv   вибірка 100 000 поїздок
    └── hw3_taxi_zones.csv    довідник зон TLC
```

## Таблиці

| Таблиця | Призначення |
|---|---|
| `hw3_taxi_staging` | сирі дані поїздок, усі колонки `TEXT` |
| `hw3_taxi_zones_staging` | сирий довідник зон |
| `hw3_taxi_trips` | типізовані очищені поїздки |
| `hw3_taxi_zones` | типізований довідник зон |
