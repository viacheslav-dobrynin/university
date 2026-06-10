# Fact-check log: HNSW

**Файл**: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/vector-indexes-and-quantization/hnsw.mdx`

**Аудит**: 2026-06-10 — Phase-5 daily-rigor re-audit «для лучшего университета». Полный пересмотр с нуля, предложение за предложением, против первоисточников. Предыдущий аудит (2026-04-23/24) был тонким (~6 КБ) и устарел; этот аудит заменяет его.

## Сводка (2026-06-10)

- Всего проверяемых утверждений: ~95.
- ✅ подтверждено: подавляющее большинство (атрибуции, формулы, параметры реальных систем, данные LUCENE-9937, арифметика числового примера — всё сверено с первоисточниками).
- ⚠️ неточно: **1** — раздел «Small-world графы»: свойство **navigable** (эффективная жадная маршрутизация, polylog) приписано модели Уоттса–Строгаца, но в W–S навигабельность отсутствует (жадная маршрутизация экспоненциальна); навигабельность — вклад Клейнберга (2000).
- ❌ опровергнуто: **0**.
- 🔍 не подтверждено: **0** (все утверждения нашли источник).
- **Вердикт: NEEDS-FIXES (1 ⚠️)** — единственное содержательное замечание относится к атрибуции навигабельности Клейнбергу. **→ устранено 2026-06-10, см. раздел «Разрешение ⚠️-замечаний» ниже; текущий статус APPROVED.**

Примечание: устаревшее замечание прошлого аудита про «ref[1] = 2018» более не применимо — текущий текст указывает 2020 / TPAMI vol 42 / issue 4 / pp 824–836, что корректно (DOI 10.1109/TPAMI.2018.2889473, опубликовано в апреле 2020).

## Секции

| # | Секция | Строки | Статус | Дата | Итог |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–26 | ✅ Проверено | 2026-06-10 | Размерности эмбеддингов (128 SIFT, 384 MiniLM, 768 BERT, 1536/3072 OpenAI) подтверждены. Curse of dimensionality / деградация kd/R-tree до ~линейного скана выше ~10 измерений + distance concentration — подтверждено. Атрибуция HNSW (Малков & Яшунин, 2016 arXiv) и NSW (2014) — подтверждено. Список систем (Qdrant, Weaviate, pgvector, Milvus, Elasticsearch, Redis, FAISS) — все реально реализуют HNSW. |
| 2 | Навигация по графу: предшественник NSW | 28–44 | ⚠️ Неточно | 2026-06-10 | NSW (M ближайших + случайные дальние рёбра через порядок вставок), жадный спуск, локальный минимум — подтверждено. **⚠️ Замечание**: «navigable small-world… впервые формально описаны в модели Уоттса–Строгаца» — W–S даёт small-world (низкий диаметр + высокая кластеризация), но НЕ navigable: жадная маршрутизация в W–S экспоненциальна. Навигабельность (polylog жадный routing) — модель Клейнберга (2000). |
| 3 | Иерархическая структура HNSW | 46–78 | ✅ Проверено | 2026-06-10 | Многослойный граф, экспоненциальное распределение уровней, `floor(-ln(U)·mL)`, `mL = 1/ln(M)`, `M_max0 = 2M`, инвариант связности, точка входа на максимальном уровне — всё подтверждено оригинальной статьёй (arXiv:1603.09320). Таблица параметров с типичными значениями корректна. |
| 4 | Схема иерархии (визуализация) | 79–172 | ✅ Проверено | 2026-06-10 | Иллюстративная диаграмма; проза согласована (L2=2 верш., L1=4, L0=12; спуск L2→L1→L0). Фактических ошибок нет. |
| 5 | Алгоритм поиска: SEARCH-LAYER, K-NN-SEARCH, числовой пример | 174–271 | ✅ Проверено | 2026-06-10 | Псевдокод SEARCH-LAYER (beam search, min/max-кучи, условие остановки) и K-NN-SEARCH соответствуют Algorithm 2/5 статьи. Арифметика числового примера пересчитана независимо: a=11.45, b=6.05, c=2.65, d=0.05, e=0.65, f=4.05, g=11.45 — точно совпадает; ответ d корректен. |
| 6 | Алгоритм вставки: INSERT, SELECT-NEIGHBORS | 273–330 | ✅ Проверено | 2026-06-10 | INSERT соответствует Algorithm 1; два варианта SELECT-NEIGHBORS — Algorithms 3/4. Связь эвристики с RNG корректно охарактеризована: в самой статье термин RNG не используется, эквивалентность отмечается в последующих исследованиях — это аккуратно отражено в тексте. |
| 7 | Сложность | 332–342 | ✅ Проверено | 2026-06-10 | O(log n) поиска как эмпирическое наблюдение + теоретическая оценка при small-world/равномерных данных, с явным caveat «не гарантия в худшем случае» — корректно и хорошо захеджировано (статья заявляет polylog scaling). Стоимость вставки и оценка памяти `~ n·2M` (геометрическая сумма по уровням) — корректно. |
| 8 | Компромиссы и настройка: Recall–QPS + LUCENE-9937 | 344–408 | ✅ Проверено | 2026-06-10 | Пять точек `(ef, recall, QPS)` сверены поштучно с тикетом: (10,0.713,69662.756), (50,0.950,28021.582), (100,0.985,16108.538), (500,1.000,4115.886), (800,1.000,2729.680) — округления корректны. M=16, efConstruction=500 — подтверждено. Disclaimer тикета («benchmark contained an error that made hnswlib perform too well») теперь честно отражён в прозе. Рамка illustrative/measured честная. |
| 9 | Выбор M, efConstruction vs ef, оценка памяти | 410–434 | ✅ Проверено | 2026-06-10 | Правило `efConstruction ≥ 2·ef_target`, арифметика памяти (граф 128 MB при n=10^6,M=16; 30.7 GB векторов + 1.28 GB граф ≈ 32 GB при 10^7×768; ~3.2 KB/вектор) — пересчитано, верно. |
| 10 | Ограничения | 436–442 | ✅ Проверено | 2026-06-10 | Tombstones, delete+reinsert, плохая переносимость на диск (random page access), DiskANN/Vamana как SSD-альтернатива, чувствительность к распределению, недетерминизм построения — подтверждено. |
| 11 | Применение в продакшене | 444–473 | ✅ Проверено | 2026-06-10 | Qdrant (m=16, ef_construct=100, ef=ef_construct, filterable HNSW в обходе) ✅. Milvus (HNSW + IVF_FLAT/SQ8/PQ + DiskANN) ✅. Weaviate (HNSW default, flat+dynamic, switch на 10k, ef=-1 → clamp(dynamicEfFactor·limit,…)) ✅. pgvector 0.5.0 (28 авг 2023), синтаксис, m=16/ef_construction=64/hnsw.ef_search=40 ✅. Lucene 9.0 (ANN via HNSW graphs) / 9.5 (KnnFloatVectorQuery+Field) ✅. Redis (M=16, EF_CONSTRUCTION=200, EF_RUNTIME=10, EPSILON) ✅. Vespa (HNSW на tensor-полях, attribute filtering) ✅. FAISS (IndexHNSWFlat/SQ/PQ/2Level) ✅. |
| 12 | Список литературы | 475–499 | ✅ Проверено | 2026-06-10 | [1] Malkov&Yashunin 2020 TPAMI 42(4):824–836, arXiv:1603.09320 ✅ (год исправлен на 2020). [2] NSW Information Systems 2014 vol45 pp61–68 ✅. [3] Watts–Strogatz Nature 1998 393:440–442, DOI 10.1038/30918 ✅. [4]–[12] (ann-benchmarks, hnswlib, FAISS, pgvector, Qdrant, Milvus, Weaviate, Lucene, LUCENE-9937) — URL/метаданные корректны. |

## Источники (первоисточники, использованные в этом аудите)

- HNSW arXiv: https://arxiv.org/abs/1603.09320
- HNSW IEEE TPAMI 2020: https://dl.acm.org/doi/abs/10.1109/TPAMI.2018.2889473
- NSW (Information Systems 2014): https://scholar.google.com/scholar_lookup?title=Approximate+nearest+neighbor+algorithm+based+on+navigable+small+world+graphs&author=Y.+Malkov&publication_year=2014&journal=Information+Systems&volume=45&pages=61-68
- Watts–Strogatz Nature 1998: https://www.nature.com/articles/30918 (DOI 10.1038/30918)
- Kleinberg navigable small-world / greedy routing: https://www.kth.se/social/upload/514c7450f276547cb33a1992/2-kleinberg.pdf
- LUCENE-9937: https://issues.apache.org/jira/browse/LUCENE-9937
- pgvector: https://github.com/pgvector/pgvector ; релиз 0.5.0: https://www.postgresql.org/about/news/pgvector-050-released-2700/
- Qdrant indexing: https://qdrant.tech/documentation/concepts/indexing/
- Weaviate vector index: https://docs.weaviate.io/weaviate/concepts/vector-index
- Milvus index types: https://milvus.io/docs/index.md
- Redis vector (HNSW): https://redis.io/docs/latest/develop/ai/search-and-query/vectors/
- Vespa HNSW: https://docs.vespa.ai/en/querying/approximate-nn-hnsw.html
- FAISS indexes: https://github.com/facebookresearch/faiss/wiki/Faiss-indexes
- Lucene 9.0 release notes: https://cwiki.apache.org/confluence/display/LUCENE/Release+Notes+9.0
- Lucene 9.5 release notes: https://cwiki.apache.org/confluence/display/LUCENE/Release+Notes+9.5

## Разрешение ⚠️-замечаний (2026-06-10, orchestrator)

**Замечание 1 (секция 2, строка 32): «navigable small-world… впервые формально описаны в модели Уоттса–Строгаца» — FIXED (2026-06-10).**

Модель Уоттса–Строгаца (1998) даёт small-world (низкий диаметр + высокая кластеризация), но НЕ navigable: жадная маршрутизация при наличии лишь топологии малого мира экспоненциальна. Навигабельность (polylog жадный routing) требует особого распределения длинных рёбер и формально установлена Клейнбергом (2000).

Исправление в `hnsw.mdx`:
- строка 32 переписана: small-world (Уоттс–Строгац: низкий диаметр + высокая кластеризация) теперь явно отделён от navigable (Клейнберг 2000: polylog жадный routing требует особого распределения дальних рёбер); добавлено уточнение «Именно навигируемость — а не просто принадлежность классу small-world — обеспечивает быстрый поиск; NSW и HNSW конструируют граф так, чтобы это свойство выполнялось» с inline-ссылкой `[[13](#ref-kleinberg)]`;
- добавлен источник [13] Kleinberg, J. (2000). *The Small-World Phenomenon: An Algorithmic Perspective*. STOC '00, 163–170. DOI 10.1145/335305.335325.

Сборка зелёная (Node 20). Submodule commit `7ac6953` («docs(hnsw): distinguish small-world (Watts-Strogatz) from navigable (Kleinberg)»), bump родителя `f95322b`, деплой run `27277204085` — GREEN.

**Обновлённый вердикт: APPROVED — 0 ❌, 0 ⚠️ (единственное ⚠️ устранено).**
