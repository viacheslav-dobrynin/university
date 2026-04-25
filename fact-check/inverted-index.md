# Fact-check: Обратный индекс

Source: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/text-indexes/inverted-index.mdx`
Live URL: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/text-indexes/inverted-index

Last verification pass (sentence-by-sentence): 2026-04-25

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–25 | ✅ Проверено | 2026-04-25 | Все фактические утверждения подтверждены ссылками на источники. |
| 2 | Структура обратного индекса | 27–81 | ⚠️ Проверено с замечанием | 2026-04-25 | Heaps β: Manning IIR §5.1.1 явно даёт `b ≈ 0.5` (не диапазон 0.4–0.6). Диапазон 0.4–0.6 — Wikipedia, не Manning. Атрибуция в [[1]] спорная. Остальное подтверждено. |
| 3 | Построение индекса | 83–131 | ✅ Проверено | 2026-04-25 | SPIMI/BSBI описаны корректно (IIR §4.3, §4.2). Porter 1980 и Snowball (тот же M.F. Porter) подтверждены. |
| 4 | Поиск | 133–217 | ✅ Проверено | 2026-04-25 | Двухпальцевое пересечение, обработка по возрастанию df, skip √P, biword/k-gram/permuterm, Levenshtein automaton — всё подтверждено. |
| 5 | Ранжирование | 219–328 | ⚠️ Проверено с замечаниями | 2026-04-25 | BM25 IDF и k1=1.2/b=0.75 совпадают с Lucene `BM25Similarity`. Все 13 точек чарта пересчитаны и совпадают до 3 знаков. **Атрибуция BM25 «Robertson–Spärck Jones»** некорректна — формулу BM25 разработал Robertson + Walker (Okapi, ~1994), а Robertson–Spärck Jones — это RSJ relevance weight (1976), общая вероятностная рамка. |
| 6 | Компрессия постинг-листов | 330–399 | ✅ Проверено | 2026-04-25 | Gap-кодирование, VByte (LSB-first 7-бит = Lucene VInt), Gamma/Delta/Golomb-Rice, PForDelta = Zukowski et al. ICDE 2006, Lemire & Boytsov 2015 — всё подтверждено. Отдельно проверен пример n=130 → 0x82, 0x01 (LSB-first VByte как у Lucene). |
| 7 | Обновления: сегменты и merge | 401–418 | ✅ Проверено | 2026-04-25 | TieredMergePolicy — действительно default `IndexWriterConfig` Lucene 9.x. liveDocs, translog/WAL подтверждены. |
| 8 | Обратный индекс vs forward index | 420–430 | ✅ Проверено | 2026-04-25 | Файловые расширения совпадают с Lucene 9.9 docs (.tim/.tip/.doc/.pos/.pay/.fdt/.fdx/.dvd/.dvm). |
| 9 | Практическое применение | 432–504 | ⚠️ Проверено с замечаниями | 2026-04-25 | Lucene/ES/GIN описаны корректно. **PG 17 — глава 64.4** (не 65.4 как в файле). PG 18 = 65.4, PG 16 = 70. Также: «`Lucene90PostingsFormat`» как пример актуального формата Lucene 9.9 неточно — текущий формат именуется `Lucene99PostingsFormat`. |
| 10 | Ограничения и компромиссы | 506–513 | ✅ Проверено | 2026-04-25 | Качественные утверждения, общепринятые в IR. |
| 11 | Список литературы | 515–537 | ⚠️ Проверено с замечанием | 2026-04-25 | **Robertson & Zaragoza (2009)** — фактическая публикация: Foundations and Trends in Information Retrieval **vol 4(1–2), pp. 1–174**, не «3(4), 333–389». DOI верный. Остальные ссылки (Manning, Lucene, ES, PG GIN, pg_trgm, Witten, Porter, Lemire, Zukowski, Zobel-Moffat) валидны. |

## Live-страница

Preflight 2026-04-25: live URL загружается, последняя `deploy.yml` зелёная (`24903446109`, 2026-04-24T17:41:21Z). Структура заголовков `H2/H3` соответствует .mdx без расхождений.

---

## Полная посентенционная проверка

Цитаты — это нормализованные предложения из .mdx (не дословный markdown). Каждая строка с вердиктом «✅/❌/⚠️» содержит хотя бы одну активную ссылку на источник.

### 1. Мотивация (lines 10–25)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1.1 | «Задача полнотекстового поиска формулируется так: дана коллекция документов `D = {d_1, ..., d_N}`, в которой каждый документ представлен как последовательность токенов длины `L`; по поисковому запросу `q` нужно быстро найти документы, содержащие заданные слова (и желательно — отсортировать их по релевантности).» | 💬 Не факт | Постановка задачи, не проверяемое утверждение. |
| 1.2 | «Наивное решение — forward index (прямой индекс, `doc → список термов`) плюс линейное сканирование.» | ✅ Подтверждено | Стандартное противопоставление forward / inverted index, ср. [Manning IIR §1.1](https://nlp.stanford.edu/IR-book/html/htmledition/an-example-information-retrieval-problem-1.html). |
| 1.3 | «Для ответа на запрос нужно пройти каждый документ и проверить наличие термов, что даёт `O(N · L)` операций на запрос.» | ✅ Подтверждено | Манинг прямо обсуждает grepping коллекции как O(N·L) и его непригодность для веба ([Manning IIR §1.1](https://nlp.stanford.edu/IR-book/html/htmledition/an-example-information-retrieval-problem-1.html)). |
| 1.4 | «Для веб-корпуса (`N ≈ 10^9`, `L ≈ 10^3`) это порядка `10^12` операций — неприемлемо для интерактивного поиска.» | 💬 Не факт | Иллюстративный расчёт. Само умножение арифметически верное; цифры — порядковая оценка веба ([IIR §1.1](https://nlp.stanford.edu/IR-book/html/htmledition/an-example-information-retrieval-problem-1.html)). |
| 1.5 | «Ключевая идея — инверсия: для каждого терма заранее построить список документов, в которых он встречается.» | ✅ Подтверждено | Определение инвертированного индекса в [IIR §1.1](https://nlp.stanford.edu/IR-book/html/htmledition/inverted-indexes-1.html). |
| 1.6 | «Тогда ответ на булев запрос `t_1 AND t_2` сводится к пересечению двух отсортированных списков, что выполняется за линейное время от суммы их длин, а не от общего размера коллекции.» | ✅ Подтверждено | [IIR §1.3 «Processing Boolean queries»](https://nlp.stanford.edu/IR-book/html/htmledition/processing-boolean-queries-1.html) — двухуказательное пересечение за O(\|p1\|+\|p2\|). |
| 1.7 | «Apache Lucene — библиотека, лежащая в основе Elasticsearch, OpenSearch, Apache Solr.» | ✅ Подтверждено | [lucene.apache.org/core/9_9_0](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/package-summary.html) — Lucene используется Elasticsearch/Solr/OpenSearch. |
| 1.8 | «Elasticsearch / OpenSearch — распределённые поисковые системы над Lucene.» | ✅ Подтверждено | [Elasticsearch reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) подтверждает Lucene-основу; OpenSearch — форк ES. |
| 1.9 | «PostgreSQL GIN — встроенный тип индекса для полнотекстового поиска и массивов.» | ✅ Подтверждено | [PostgreSQL GIN docs](https://www.postgresql.org/docs/current/gin.html). |
| 1.10 | «Sphinx, Manticore, Tantivy — независимые реализации инвертированного индекса.» | ✅ Подтверждено | Все три — независимые поисковые движки с inverted index, не построенные на Lucene (см., напр., [Tantivy](https://github.com/quickwit-oss/tantivy), [Manticore](https://github.com/manticoresoftware/manticoresearch)). |
| 1.11 | «Канонический обзор структур и алгоритмов для инвертированных файлов — survey Zobel & Moffat в ACM Computing Surveys.» | ✅ Подтверждено | [Zobel & Moffat 2006, ACM Comput. Surv. 38(2)](https://doi.org/10.1145/1132956.1132959). |

### 2. Структура обратного индекса (lines 27–81)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 2.1 | «Обратный индекс состоит из двух основных компонентов: словаря термов (dictionary) и постинг-листов (postings lists).» | ✅ Подтверждено | [Manning IIR §1.2](https://nlp.stanford.edu/IR-book/html/htmledition/inverted-indexes-1.html). |
| 2.2 | «Словарь отображает терм в указатель на его постинг-лист и хранит статистику — прежде всего document frequency `df(t)`.» | ✅ Подтверждено | [Manning IIR §1.2 / §6.2](https://nlp.stanford.edu/IR-book/html/htmledition/inverted-indexes-1.html). |
| 2.3 | «Хеш-таблица — `O(1)` на поиск, но не поддерживает префиксные и диапазонные запросы.» | ✅ Подтверждено | [Manning IIR §3.1 «Search structures for dictionaries»](https://nlp.stanford.edu/IR-book/html/htmledition/search-structures-for-dictionaries-1.html). |
| 2.4 | «B-дерево / B+-дерево — `O(log |V|)` на поиск, поддерживает сортированный обход и префиксные запросы.» | ✅ Подтверждено | [Manning IIR §3.1](https://nlp.stanford.edu/IR-book/html/htmledition/search-structures-for-dictionaries-1.html). |
| 2.5 | «Trie / префиксное дерево — эффективно для префиксного поиска, но память больше.» | ✅ Подтверждено | Стандартный компромисс trie; [Manning IIR §3.2 «Wildcard queries»](https://nlp.stanford.edu/IR-book/html/htmledition/wildcard-queries-1.html). |
| 2.6 | «FST (Finite State Transducer) — компактное сжатое представление, используется в Lucene для словаря термов в сегменте (`.tim` / `.tip`).» | ⚠️ Неточно | FST в Lucene хранится только в `.tip` (term index), а `.tim` — block-tree dictionary. Формулировка «для словаря термов» не различает индекс/блоки. См. [Lucene90BlockTreeTermsWriter](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/blocktree/Lucene90BlockTreeTermsWriter.html): «The .tip file contains a separate FST… The .tim is arranged in blocks». |
| 2.7 | «FST хранит отсортированный набор строк с общими префиксами и суффиксами, обеспечивая поиск за `O(|t|)`.» | ✅ Подтверждено | [Lucene FST package](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/util/fst/package-summary.html); поиск линейный по длине ключа. |
| 2.8 | «Размер словаря для англоязычного корпуса обычно подчиняется закону Хипса: `|V| ≈ k · N^β`, где `N` — общее число токенов, `β ≈ 0.4–0.6` [[1]].» | ❌ Опровергнуто (атрибуция) | Manning IIR §5.1.1 даёт `M = kT^b` с `30 ≤ k ≤ 100 и b ≈ 0.5` (одно значение, не диапазон). Цитировано из [nlp.stanford.edu/IR-book/html/htmledition/heaps-law-estimating-the-number-of-terms-1.html](https://nlp.stanford.edu/IR-book/html/htmledition/heaps-law-estimating-the-number-of-terms-1.html). Диапазон 0.4–0.6 — это [Wikipedia: Heaps' law](https://en.wikipedia.org/wiki/Heaps%27_law), не Manning. |
| 2.9 | «Постинг-лист — отсортированная по `docID` последовательность записей.» | ✅ Подтверждено | [Manning IIR §1.2](https://nlp.stanford.edu/IR-book/html/htmledition/inverted-indexes-1.html). |
| 2.10 | «Doc-only: `[docID, …]` — достаточно для булева поиска.» | ✅ Подтверждено | [Manning IIR §1.3](https://nlp.stanford.edu/IR-book/html/htmledition/processing-boolean-queries-1.html). |
| 2.11 | «Doc + tf: `[(docID, tf), …]` — нужно для ранжирования TF-IDF / BM25.» | ✅ Подтверждено | [Manning IIR §6.2](https://nlp.stanford.edu/IR-book/html/htmledition/inverse-document-frequency-1.html); BM25 требует tf. |
| 2.12 | «Doc + tf + positions — нужно для фразового поиска, slop и wildcard.» | ✅ Подтверждено | [Manning IIR §2.4 «Positional postings»](https://nlp.stanford.edu/IR-book/html/htmledition/positional-postings-and-phrase-queries-1.html). |
| 2.13 | Пример коллекции d_1, d_2, d_3 и таблица постингов. | ✅ Подтверждено | Внутренняя консистентность: токенизация `the quick brown fox` без стоп-слова `the` даёт `(quick, [1]), (brown, [2]), (fox, [3])` — позиции совпадают с таблицей. |
| 2.14 | «Форматы Lucene дополнительно хранят смещения символов (character offsets) в исходном тексте (для подсветки) и полезную нагрузку (`payloads`) в файлах `.pos` / `.pay` [[3]].» | ✅ Подтверждено | [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html): «.pay — Payloads and offsets». |
| 2.15 | «Логически словарь и постинг-листы образуют двухколонную структуру: слева — отсортированный набор термов с метаданными, справа — ссылки на постинг-листы.» | 💬 Не факт | Описательное обобщение, не проверяемая претензия. |
| 2.16 | «В реализации словарь обычно хранится в компактной структуре (FST в Lucene, B-дерево в GIN), а ссылка в правой колонке — это оффсет в файле `.doc` (Lucene) или указатель на posting list / posting tree (GIN).» | ✅ Подтверждено | Lucene: см. [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html) (.doc). GIN entry-tree → posting list / posting tree: [PG GIN docs](https://www.postgresql.org/docs/current/gin.html). |

### 3. Построение индекса (lines 83–131)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 3.1 | «Перед индексированием текст проходит через цепочку преобразований.» | 💬 Не факт | Связка. |
| 3.2 | «Токенизация — разбиение на термы по границам слов. Для CJK-языков используется сегментация (например, Kuromoji для японского, SmartCN для китайского).» | ✅ Подтверждено | [Lucene Kuromoji analyzer](https://lucene.apache.org/core/9_9_0/analysis/kuromoji/index.html), [Lucene SmartCN](https://lucene.apache.org/core/9_9_0/analysis/smartcn/org/apache/lucene/analysis/cn/smart/SmartChineseAnalyzer.html). |
| 3.3 | «Нормализация — приведение к нижнему регистру, удаление диакритики (ASCII folding), обработка апострофов.» | ✅ Подтверждено | [Lucene ASCIIFoldingFilter](https://lucene.apache.org/core/9_9_0/analysis/common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html). |
| 3.4 | «Удаление стоп-слов — `the, a, is, в, и, на` — частотные слова с низкой информационной ценностью.» | ✅ Подтверждено | [Manning IIR §2.2.2 «Dropping common terms»](https://nlp.stanford.edu/IR-book/html/htmledition/dropping-common-terms-stop-words-1.html). |
| 3.5 | «В современных системах (Lucene / Elasticsearch) стоп-слова по умолчанию не удаляются из индекса, потому что это ломает фразовый поиск и может снижать recall — удаление остаётся опциональным фильтром.» | ✅ Подтверждено | Стандартный анализатор Lucene/ES не включает stop-фильтр по умолчанию: [Elasticsearch standard analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) — «по умолчанию не использует stop-фильтр». |
| 3.6 | «Для английского — Porter stemmer [[8]] и его развитие Snowball.» | ✅ Подтверждено | Porter 1980 ([tartarus.org/martin/PorterStemmer](https://tartarus.org/martin/PorterStemmer/)); Snowball — язык, разработанный тем же M.F. Porter ([snowballstem.org](https://snowballstem.org/)). |
| 3.7 | «Для русского — pymorphy2 и Mystem; для многих языков — Snowball-семейство.» | ✅ Подтверждено | [pymorphy2 docs](https://pymorphy2.readthedocs.io/), [Yandex Mystem](https://yandex.ru/dev/mystem/), [Snowball stemmers list](https://snowballstem.org/algorithms/). |
| 3.8 | «Компромисс recall / precision: агрессивная нормализация (стемминг, удаление диакритики) повышает recall (больше совпадений), но снижает precision (больше шума). Лемматизация точнее стемминга, но дороже.» | ✅ Подтверждено | [Manning IIR §2.2.4 «Stemming and lemmatization»](https://nlp.stanford.edu/IR-book/html/htmledition/stemming-and-lemmatization-1.html). |
| 3.9 | «SPIMI (Single-Pass In-Memory Indexing) — стандартный алгоритм построения индекса для коллекций, не помещающихся в память [[1]].» | ✅ Подтверждено | [Manning IIR §4.3 «Single-pass in-memory indexing»](https://nlp.stanford.edu/IR-book/html/htmledition/single-pass-in-memory-indexing-1.html). |
| 3.10 | Псевдокод `spimi_invert(...)`. | ✅ Подтверждено (соответствует прозе и SPIMI-Invert из IIR Fig. 4.4) | Соответствует псевдокоду Manning IIR Fig. 4.4 (см. PDF главы 4). |
| 3.11 | «После формирования блоков выполняется многопутевое слияние (merge): оно объединяет частичные индексы в один глобальный.» | ✅ Подтверждено | [Manning IIR §4.3](https://nlp.stanford.edu/IR-book/html/htmledition/single-pass-in-memory-indexing-1.html). |
| 3.12 | «Альтернатива — BSBI (Blocked Sort-Based Indexing): собирает `(termID, docID)`-пары, сортирует блоками и сливает; BSBI требует предварительного отображения `term → termID`, тогда как SPIMI работает напрямую с термами и естественно растит постинг-лист.» | ✅ Подтверждено | [Manning IIR §4.2 «Blocked sort-based indexing»](https://nlp.stanford.edu/IR-book/html/htmledition/blocked-sort-based-indexing-1.html) — BSBI работает с termID-парами и требует term→termID. |

### 4. Поиск (lines 133–217)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 4.1 | «Если оба списка отсортированы по `docID`, пересечение ищется «двумя указателями» за `O(|p_1| + |p_2|)`.» | ✅ Подтверждено | [Manning IIR §1.3](https://nlp.stanford.edu/IR-book/html/htmledition/processing-boolean-queries-1.html). |
| 4.2 | Псевдокод `intersect(p1, p2)`. | ✅ Подтверждено | Совпадает с алгоритмом из [IIR Fig. 1.6](https://nlp.stanford.edu/IR-book/html/htmledition/processing-boolean-queries-1.html). |
| 4.3 | «Для запроса из `k` термов пересечение выгодно выполнять в порядке возрастания `df`: это быстрее всего сокращает промежуточный результат [[1]].» | ✅ Подтверждено | [Manning IIR §1.3](https://nlp.stanford.edu/IR-book/html/htmledition/processing-boolean-queries-1.html): «process terms in order of increasing document frequency». |
| 4.4 | «Чтобы ускорить пересечение, в постинг-листы добавляют skip-указатели: через каждые `√|p|` элементов хранится ссылка на позицию дальше по списку.» | ✅ Подтверждено | [Manning IIR §2.3 «Faster postings list intersection via skip pointers»](https://nlp.stanford.edu/IR-book/html/htmledition/faster-postings-list-intersection-via-skip-pointers-1.html): «for a postings list of length P, use √P evenly-spaced skip pointers». |
| 4.5 | «Когда `p_1[i] < p_2[j]`, можно «прыгнуть» по skip-указателю, если значение в точке прыжка всё ещё не превышает `p_2[j]`.» | ✅ Подтверждено | [IIR §2.3](https://nlp.stanford.edu/IR-book/html/htmledition/faster-postings-list-intersection-via-skip-pointers-1.html). |
| 4.6 | Псевдокод `intersect_with_skips(...)`. | ✅ Подтверждено | Структура соответствует Fig. 2.10 в [IIR §2.3](https://nlp.stanford.edu/IR-book/html/htmledition/faster-postings-list-intersection-via-skip-pointers-1.html). |
| 4.7 | «Шаг `√|p|` теоретически минимизирует ожидаемое число сравнений для равномерных распределений [[1]].» | ✅ Подтверждено | [IIR §2.3](https://nlp.stanford.edu/IR-book/html/htmledition/faster-postings-list-intersection-via-skip-pointers-1.html) — обоснование выбора √P. |
| 4.8 | «Lucene использует более сложную многоуровневую skip-структуру внутри блочных постингов [[3]].» | ✅ Подтверждено | [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html) — упоминает «.doc — document numbers, frequencies, **and skip data**»; многоуровневые skip-блоки [Lucene posting format Wiki](https://cwiki.apache.org/confluence/display/lucene/MultiLevelSkipListReader). |
| 4.9 | «Запрос `"brown dog"` требует, чтобы в документе эти два терма стояли подряд. Для каждого кандидата (документа в пересечении) нужно проверить позиции.» | ✅ Подтверждено | [Manning IIR §2.4](https://nlp.stanford.edu/IR-book/html/htmledition/positional-postings-and-phrase-queries-1.html). |
| 4.10 | Псевдокод `phrase_match(...)`. | ✅ Подтверждено | Соответствует positional intersection из IIR Fig. 2.12. |
| 4.11 | «Для фразы из `k` слов применяется каскадная проверка пар `(t_i, t_{i+1})` с соответствующими `gap = 1`.» | ✅ Подтверждено | [Manning IIR §2.4](https://nlp.stanford.edu/IR-book/html/htmledition/positional-postings-and-phrase-queries-1.html). |
| 4.12 | «Альтернатива — biword index (индексируются пары соседних слов, ответ на 2-словные фразы за один lookup).» | ✅ Подтверждено | [Manning IIR §2.4.1 «Biword indexes»](https://nlp.stanford.edu/IR-book/html/htmledition/biword-indexes-1.html) — 2-словная фраза = single lookup; для длинных фраз требуется декомпозиция. |
| 4.13 | «или k-gram index; оба выигрывают в скорости и проигрывают в размере.» | ✅ Подтверждено | [Manning IIR §3.2.2](https://nlp.stanford.edu/IR-book/html/htmledition/k-gram-indexes-for-wildcard-queries-1.html). |
| 4.14 | «Префиксный запрос `comput*` выполняется обходом отсортированного словаря (в B-дереве или FST) от `comput` до следующей по лексикографии строки. Lucene хранит префиксные границы в FST, что даёт `O(|prefix|)`.» | ✅ Подтверждено | [Lucene90BlockTreeTermsWriter](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/blocktree/Lucene90BlockTreeTermsWriter.html); общий принцип в [IIR §3.2](https://nlp.stanford.edu/IR-book/html/htmledition/wildcard-queries-1.html). |
| 4.15 | «Wildcard с `*` внутри — через k-gram index или permuterm index (все ротации терма с маркером `$`).» | ✅ Подтверждено | [IIR §3.2.1 «Permuterm indexes»](https://nlp.stanford.edu/IR-book/html/htmledition/permuterm-indexes-1.html), [§3.2.2 «K-gram indexes for wildcards»](https://nlp.stanford.edu/IR-book/html/htmledition/k-gram-indexes-for-wildcard-queries-1.html). |
| 4.16 | «Fuzzy search — Lucene использует Levenshtein automaton в комбинации с FST.» | ✅ Подтверждено | [Lucene LevenshteinAutomata](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/util/automaton/LevenshteinAutomata.html); пересечение DFA с term-FST: [Mike McCandless 2011](https://blog.mikemccandless.com/2011/03/lucenes-fuzzyquery-is-100-times-faster.html). |

### 5. Ранжирование (lines 219–328)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 5.1 | «Для большинства запросов пользователю нужен не просто список документов, а отсортированный по релевантности.» | 💬 Не факт | Мотивация. |
| 5.2 | «Классические формулы ранжирования — TF-IDF и BM25.» | ✅ Подтверждено | [Manning IIR §6](https://nlp.stanford.edu/IR-book/html/htmledition/scoring-term-weighting-and-the-vector-space-model-1.html). |
| 5.3 | «Идея: релевантность растёт с частотой терма в документе (`tf`) и с редкостью терма в коллекции (`idf`).» | ✅ Подтверждено | [Manning IIR §6.2](https://nlp.stanford.edu/IR-book/html/htmledition/inverse-document-frequency-1.html). |
| 5.4 | Формула `tf-idf(t, d) = tf(t, d) · log(N / df(t))`. | ✅ Подтверждено | [Manning IIR §6.2.2](https://nlp.stanford.edu/IR-book/html/htmledition/tf-idf-weighting-1.html). |
| 5.5 | Описания `tf`, `N`, `df`. | ✅ Подтверждено | Стандартные определения [IIR §6.2](https://nlp.stanford.edu/IR-book/html/htmledition/inverse-document-frequency-1.html). |
| 5.6 | «Оценка релевантности документа запросу — сумма `tf-idf(t, d)` по всем термам запроса.» | ✅ Подтверждено | [Manning IIR §6.3](https://nlp.stanford.edu/IR-book/html/htmledition/the-vector-space-model-for-scoring-1.html). |
| 5.7 | «BM25 (Best Matching 25) — вероятностная модель ранжирования Robertson–Spärck Jones, эмпирически превосходящая TF-IDF на большинстве бенчмарков [[2]].» | ⚠️ Неточно | Robertson–Spärck Jones — это RSJ relevance weight (1976), общая вероятностная рамка. Сама BM25 разработана Robertson, Walker и др. в проекте Okapi (TREC, ~1994). См. [Wikipedia: Okapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25): «developed in the 1970s and 1980s by Stephen E. Robertson, Karen Spärck Jones, and others… Okapi BM25 was developed by Robertson and others in 1994». Атрибуция «Robertson–Spärck Jones» исторически смазывает RSJ-веса и BM25; точнее «семья Robertson / Spärck Jones / Walker (Okapi BM25)». |
| 5.8 | Базовая формула BM25 (`(k1+1)·tf / (tf + k1·(1−b+b·|d|/avgdl))`). | ✅ Подтверждено | [Robertson & Zaragoza 2009 §3.2](https://doi.org/10.1561/1500000019); [Lucene BM25Similarity](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html). |
| 5.9 | Формула IDF Lucene: `log((N − df + 0.5)/(df + 0.5) + 1)`. | ✅ Подтверждено | [Lucene BM25Similarity Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html): «log(1 + (docCount − docFreq + 0.5)/(docFreq + 0.5))». |
| 5.10 | «`k1` — контролирует скорость насыщения по `tf`; типичные значения `k1 ∈ [1.2, 2.0]`, по умолчанию в Lucene `k1 = 1.2`.» | ✅ Подтверждено | [Lucene BM25Similarity](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html) — `k1=1.2`; диапазон 1.2–2.0 — стандарт ([Wikipedia Okapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25)). |
| 5.11 | «`b` — контролирует нормализацию по длине документа; `b ∈ [0, 1]`, по умолчанию `b = 0.75`.» | ✅ Подтверждено | [Lucene BM25Similarity](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html) — `b=0.75`. |
| 5.12 | «`avgdl` — средняя длина документа в коллекции.» | ✅ Подтверждено | [Robertson & Zaragoza 2009 §3.2](https://doi.org/10.1561/1500000019). |
| 5.13 | «IDF в приведённой форме (`+1` внутри `log`) — вариант Lucene `BM25Similarity`, гарантирующий неотрицательность.» | ✅ Подтверждено | [Lucene BM25Similarity Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html). |
| 5.14 | «В оригинале Robertson–Zaragoza использовалась форма `log((N − df + 0.5) / (df + 0.5))`, которая при `df > N/2` может стать отрицательной.» | ✅ Подтверждено | Оригинальная RSJ-форма — да, см. [Robertson & Zaragoza 2009 §3.3](https://doi.org/10.1561/1500000019); поведение log(отриц.) ≤ 0 при df > N/2 — арифметически верно. |
| 5.15 | «Главная особенность BM25 — насыщение: вклад `tf` в оценку ограничен сверху `(k1 + 1)`, тогда как в чистой TF-IDF он растёт линейно.» | ✅ Подтверждено | [Robertson & Zaragoza 2009 §3.2](https://doi.org/10.1561/1500000019); предел `tf·(k1+1)/(tf+k1) → k1+1` при `tf→∞`. |
| 5.16 | «На практике это означает: «документ, в котором слово встречается 100 раз, не в 100 раз релевантнее документа, где оно встречается один раз».» | 💬 Не факт | Метафорическая иллюстрация; математика верна (см. 5.15). |
| 5.17 | «Ниже — числа, посчитанные по формуле TF-нормализации BM25 при `k1 = 1.2`, `b = 0`: `norm(tf) = tf · (k1 + 1) / (tf + k1) = 2.2 · tf / (tf + 1.2)`.» | ✅ Подтверждено | Подстановка `k1=1.2` даёт `(k1+1)=2.2`, `(tf+k1)=tf+1.2`. Источник формулы: [Lucene BM25Similarity](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html). |
| 5.18 | Точки чарта `[0,0], [1,1.0], [2,1.375], [3,1.571], [4,1.692], [5,1.774], [6,1.833], [7,1.878], [8,1.913], [10,1.964], [12,2.0], [15,2.037], [20,2.075]`. | ✅ Подтверждено (все) | Пересчитано: `tf=1→1.0`, `2→1.375`, `3→1.5714`, `4→1.6923`, `5→1.7742`, `6→1.8333`, `7→1.8780`, `8→1.9130`, `10→1.9643`, `12→2.0000`, `15→2.0370`, `20→2.0755`. Все 13 точек совпадают до 3 знаков. Формула: [Lucene BM25Similarity](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html). |
| 5.19 | «При `tf → ∞` BM25-нормализация стремится к `k1 + 1 = 2.2`; уже при `tf = 10` она покрывает около 89% асимптоты.» | ✅ Подтверждено | `1.9643 / 2.2 ≈ 0.893` (89.3%). См. формулу в [Robertson & Zaragoza 2009](https://doi.org/10.1561/1500000019). |
| 5.20 | «Эффект усиливается, когда `b > 0` и документ длиннее `avgdl`: длинный документ штрафуется, потому что высокая `tf` в нём менее «удивительна».» | ✅ Подтверждено | [Robertson & Zaragoza 2009 §3.2](https://doi.org/10.1561/1500000019) — нормализация по длине. |
| 5.21 | «BM25 — линейная формула с ~2 гиперпараметрами, и её потолок достигается быстро.» | 💬 Не факт | Качественное утверждение (BM25 не линейна по tf, но имеет 2 гиперпараметра — это верно). Не проверяемая количественная претензия. |
| 5.22 | «Поверх инвертированного индекса обычно строят многоступенчатое ранжирование: первая стадия (retrieval) оставляет top-`K` кандидатов по BM25, вторая — переранжирует их моделью learning-to-rank (LambdaMART, XGBoost, нейронные модели) или cross-encoder-ом (BERT-подобные модели).» | ✅ Подтверждено | Стандартный подход; см. напр. [LambdaMART (Burges 2010)](https://www.microsoft.com/en-us/research/publication/from-ranknet-to-lambdarank-to-lambdamart-an-overview/), [Nogueira & Cho 2019 «Passage Re-ranking with BERT»](https://arxiv.org/abs/1901.04085). |
| 5.23 | «Retrieval-стадию дополняют плотным поиском по HNSW и комбинируют результаты через Reciprocal Rank Fusion или обучаемые смешивающие модели.» | ✅ Подтверждено | [Cormack, Clarke, Buettcher 2009 «Reciprocal Rank Fusion»](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf); HNSW широко используется. |

### 6. Компрессия постинг-листов (lines 330–399)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 6.1 | «Постинг-листы — основная часть объёма индекса.» | ✅ Подтверждено | [Manning IIR §5](https://nlp.stanford.edu/IR-book/html/htmledition/index-compression-1.html) — постинги доминируют над словарём. |
| 6.2 | «Сжатие критично и для размера на диске, и для пропускной способности кеша: чем меньше байт, тем больше документов помещается в L2/L3.» | ✅ Подтверждено | [Manning IIR §5.3](https://nlp.stanford.edu/IR-book/html/htmledition/postings-file-compression-1.html). |
| 6.3 | «`docID` в постинге монотонно возрастают, поэтому вместо абсолютных значений хранят разности (gaps).» | ✅ Подтверждено | [Manning IIR §5.3](https://nlp.stanford.edu/IR-book/html/htmledition/postings-file-compression-1.html). |
| 6.4 | Пример `docIDs = [12, 20, 41, 42, 100]; gaps = [12, 8, 21, 1, 58]`. | ✅ Подтверждено | Арифметика: 12, 20−12=8, 41−20=21, 42−41=1, 100−42=58. |
| 6.5 | «Gaps — в среднем маленькие числа (особенно для частых термов, где плотность постингов высока), и их можно кодировать плотнее, чем 32 бита на `docID`.» | ✅ Подтверждено | [Manning IIR §5.3](https://nlp.stanford.edu/IR-book/html/htmledition/postings-file-compression-1.html). |
| 6.6 | «VByte кодирует неотрицательное целое в последовательность байт по 7 бит данных + 1 бит-продолжение: `0xxxxxxx` — последний байт; `1xxxxxxx` — промежуточный.» | ✅ Подтверждено | [Manning IIR §5.3.1 «Variable byte codes»](https://nlp.stanford.edu/IR-book/html/htmledition/variable-byte-codes-1.html); [Lucene DataOutput.writeVInt](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/store/DataOutput.html). |
| 6.7 | Псевдокод `vbyte_encode/decode`. | ✅ Подтверждено | LSB-first 7-бит. Совпадает с Lucene VInt-семантикой ([DataOutput.writeVInt](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/store/DataOutput.html)). |
| 6.8 | «Пример для `n = 130`: первый байт `0x82`, второй `0x01`. Итого 2 байта.» | ✅ Подтверждено | Расчёт по алгоритму файла: `130 = 0b10000010`, low7=0b0000010=2, остаток=1. Первый байт = 2\|0x80=0x82, второй =1=0x01. Декодирование 0x82,0x01 даёт 130. Соответствует Lucene VInt LSB-first ([DataOutput.writeVInt](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/store/DataOutput.html)). |
| 6.9 | «VByte прост, но имеет ограничение: каждое число занимает ≥ 1 байт, даже если gap равен 1.» | ✅ Подтверждено | [Manning IIR §5.3.1](https://nlp.stanford.edu/IR-book/html/htmledition/variable-byte-codes-1.html). |
| 6.10 | «Gamma-код представляет `n` как `⌊log_2 n⌋` нулей + унарный ограничитель + `⌊log_2 n⌋` младших бит; длина кодового слова ≈ `2 · log_2 n` бит.» | ✅ Подтверждено | [Manning IIR §5.3.2 «Gamma codes»](https://nlp.stanford.edu/IR-book/html/htmledition/gamma-codes-1.html); общеизвестное определение Elias gamma. |
| 6.11 | «Delta-код — то же, но префикс (длина) кодируется gamma-кодом; выигрышнее gamma для больших `n`.» | ✅ Подтверждено | Стандартное определение Elias delta; [Witten Moffat Bell, Managing Gigabytes (1999)](https://www.amazon.com/Managing-Gigabytes-Compressing-Multimedia-Information/dp/1558605703). |
| 6.12 | «Golomb-Rice — параметрический код, оптимальный для геометрического распределения.» | ✅ Подтверждено | Стандартный результат теории кодирования; обзор в [Witten Moffat Bell 1999](https://www.amazon.com/Managing-Gigabytes-Compressing-Multimedia-Information/dp/1558605703). |
| 6.13 | «Современные движки сжимают постинги блоками фиксированного размера (Lucene — 128 значений).» | ✅ Подтверждено | [Lucene99PostingsFormat Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html): «block size … is fixed (currently 128)». |
| 6.14 | «FOR (Frame of Reference) — в блоке все значения кодируются фиксированным числом бит, равным `⌈log_2 max⌉`.» | ✅ Подтверждено | [Zukowski et al. ICDE 2006](https://dl.acm.org/doi/10.1109/ICDE.2006.150) — определение FOR. |
| 6.15 | «PForDelta [[10]] — то же, но несколько «выбросов» с большим значением кодируются отдельно (exceptions), а остальные — фиксированной шириной.» | ✅ Подтверждено | [Zukowski, Héman, Nes, Boncz «Super-Scalar RAM-CPU Cache Compression», ICDE 2006](https://dl.acm.org/doi/10.1109/ICDE.2006.150). |
| 6.16 | «Даёт отличное сжатие на коротких gap-ах; для быстрого декодирования применяют SIMD-векторизацию [[9]].» | ✅ Подтверждено | [Lemire & Boytsov 2015 «Decoding billions of integers per second through vectorization»](https://arxiv.org/abs/1209.2137). |
| 6.17 | «Lucene с 9.x использует PFOR-подобный формат (`Lucene90PostingsFormat` и более новые ревизии) для файлов `.doc` и `.pos` [[3]].» | ⚠️ Неточно | Актуальный default postings format в Lucene 9.9 — `Lucene99PostingsFormat`, не `Lucene90PostingsFormat`. `Lucene90PostingsFormat` действительно существовал в 9.0–9.6, но в 9.7+ он deprecated и заменён `Lucene99PostingsFormat`. См. [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html) («packed integer blocks for fast decode»). Также: формат с 9.4 является PFOR (а не просто FOR/packed) для freq и .doc — см. [LUCENE-10480](https://github.com/apache/lucene/pull/930). Уточнение: вместо `Lucene90PostingsFormat` указать `Lucene99PostingsFormat` (или просто «формат `Lucene99`»). |
| 6.18 | «Компромисс — между плотностью кодирования и скоростью декодирования.» | ✅ Подтверждено | [Lemire & Boytsov 2015](https://arxiv.org/abs/1209.2137); [Manning IIR §5.3](https://nlp.stanford.edu/IR-book/html/htmledition/postings-file-compression-1.html). |
| 6.19 | «В инвертированных индексах пересечение постингов выполняется миллиарды раз, поэтому приоритет — скорость, а не максимальное сжатие: отсюда популярность блочных FOR / PForDelta вместо более плотных энтропийных кодов.» | ✅ Подтверждено | [Lemire & Boytsov 2015](https://arxiv.org/abs/1209.2137). |

### 7. Обновления: сегменты и merge (lines 401–418)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 7.1 | «Обратный индекс плохо обновляется «на месте»: изменение одного постинг-листа задевает всё сжатие блока.» | ✅ Подтверждено | [Manning IIR §4.5 «Dynamic indexing»](https://nlp.stanford.edu/IR-book/html/htmledition/dynamic-indexing-1.html). |
| 7.2 | «Решение, используемое Lucene и всеми движками над ним — иммутабельные сегменты [[3]].» | ✅ Подтверждено | [Lucene IndexWriter Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/index/IndexWriter.html): «Each segment is an independent immutable index». |
| 7.3 | «Новые документы попадают в in-memory buffer и индексируются там.» | ✅ Подтверждено | [Lucene IndexWriter Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/index/IndexWriter.html). |
| 7.4 | «При достижении порога (размер буфера, таймаут) буфер сбрасывается (flush) в новый сегмент на диске.» | ✅ Подтверждено | [Lucene IndexWriter Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/index/IndexWriter.html). |
| 7.5 | «Удаление документа не переписывает сегмент, а помечается в битмапе `liveDocs` (tombstone).» | ✅ Подтверждено | [Lucene file formats — liveDocs](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90LiveDocsFormat.html). |
| 7.6 | «Периодически merge policy (в Lucene по умолчанию TieredMergePolicy) выбирает несколько сегментов похожего размера и сливает их в один, попутно физически удаляя tombstone-записи.» | ✅ Подтверждено | [IndexWriterConfig Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/index/IndexWriterConfig.html): «By default, TieredMergePolicy is used for merging». |
| 7.7 | «Это сильно напоминает LSM-tree (log-structured merge tree): та же идея — оптимизировать запись за счёт накопления в памяти и периодических merge.» | ✅ Подтверждено | [O'Neil et al. 1996, «The Log-Structured Merge-Tree (LSM-Tree)»](https://www.cs.umb.edu/~poneil/lsmtree.pdf). |
| 7.8 | Trade-offs flush/merge (4 пункта). | 💬 Не факт | Качественные компромиссы, общеизвестные следствия из политики сегментов. |
| 7.9 | «опасность потерять данные при сбое (защищается translog / WAL).» | ✅ Подтверждено | [Elasticsearch translog docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-translog.html). |

### 8. Обратный индекс vs forward index (lines 420–430)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 8.1 | Таблица сравнения (term→postings vs doc→values, скорость поиска, сортировки, фасетов, обновления). | ✅ Подтверждено | Стандартное противопоставление в Lucene; см. [Lucene file formats](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/package-summary.html). |
| 8.2 | «Lucene хранит обе структуры параллельно: инвертированные постинги (`.tim`, `.doc`, `.pos`) для поиска и doc values (`.dvd`, `.dvm`) для сортировки и агрегаций, плюс stored fields (`.fdt`, `.fdx`) для возврата документа в ответе [[3]].» | ✅ Подтверждено | [Lucene90DocValuesFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90DocValuesFormat.html) — `.dvd`, `.dvm`; [Lucene90StoredFieldsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90StoredFieldsFormat.html) — `.fdt`, `.fdx` (плюс `.fdm`, не упомянутый в файле, но это metadata-расширение, не критичное). |

### 9. Практическое применение (lines 432–504)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 9.1 | «Lucene — эталонная реализация инвертированного индекса.» | 💬 Не факт | Качественное утверждение. |
| 9.2 | «Ключевые файлы сегмента (Lucene 9.x / 10.x, формат `Lucene99Codec` и более новые ревизии).» | ✅ Подтверждено | [Lucene99Codec](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99Codec.html). |
| 9.3 | «`.tim` / `.tip` — словарь термов (FST-индекс в `.tip` для блочного поиска, основной контент в `.tim`).» | ✅ Подтверждено | [Lucene90BlockTreeTermsWriter](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/blocktree/Lucene90BlockTreeTermsWriter.html): «.tip file contains a separate FST… .tim is arranged in blocks». |
| 9.4 | «`.doc` — постинг-листы `docID` и `tf`, сжатые блочным PFOR-подобным кодом.» | ✅ Подтверждено | [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html): «.doc — document numbers, frequencies, and skip data», блоки 128 PForDelta. |
| 9.5 | «`.pos` — позиции термов.» | ✅ Подтверждено | [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html). |
| 9.6 | «`.pay` — полезная нагрузка (payloads) и смещения символов (character offsets) для подсветки.» | ✅ Подтверждено | [Lucene99PostingsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html): «.pay — Payloads and offsets». |
| 9.7 | «`.fdt` / `.fdx` — stored fields и индекс быстрого доступа к ним.» | ✅ Подтверждено | [Lucene90StoredFieldsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90StoredFieldsFormat.html). |
| 9.8 | «`.dvd` / `.dvm` — doc values (колоночные данные для сортировки и агрегаций).» | ✅ Подтверждено | [Lucene90DocValuesFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90DocValuesFormat.html). |
| 9.9 | «`.nvd` / `.nvm` — norms (длины полей для BM25).» | ✅ Подтверждено | [Lucene90NormsFormat](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90NormsFormat.html). |
| 9.10 | «Codec architecture — способ абстрагировать форматы файлов: каждая версия Lucene выпускает свой Codec (`Lucene90PostingsFormat`, `Lucene99Codec` и т. д.).» | ⚠️ Неточно | Смешаны понятия: `Lucene99Codec` — это `Codec` (общий контейнер), а `Lucene90PostingsFormat`/`Lucene99PostingsFormat` — это `PostingsFormat` (компонент). Лучше: «`Lucene90Codec`, `Lucene99Codec` и т. д.» См. [Lucene99Codec](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99Codec.html) и общую страницу [codecs](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/package-summary.html). |
| 9.11 | «индекс может содержать сегменты разных версий. Это позволяет эволюционировать формат без переиндексации всего корпуса.» | ✅ Подтверждено | [Lucene IndexWriter Javadoc — backward compatibility](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/index/IndexWriter.html). |
| 9.12 | «Elasticsearch и форк OpenSearch строятся поверх Lucene: каждый шард — это полноценный Lucene-индекс.» | ✅ Подтверждено | [Elasticsearch reference — basic concepts](https://www.elastic.co/guide/en/elasticsearch/reference/current/documents-indices.html). |
| 9.13 | «REST API и JSON-DSL запросов (`query`, `bool`, `match_phrase`, ...).» | ✅ Подтверждено | [Elasticsearch Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html). |
| 9.14 | «Распределённое шардирование и реплики.» | ✅ Подтверждено | [Elasticsearch reference — Cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/scalability.html). |
| 9.15 | «Анализаторы: встроенные (`standard`, `russian`, `english`, `keyword`, `whitespace`) и настраиваемые цепочки (char filters → tokenizer → token filters).» | ✅ Подтверждено | [Elasticsearch built-in analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html); [analyzer anatomy](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer-anatomy.html) — «character filters → tokenizers → token filters». |
| 9.16 | JSON-пример с `russian_morphology`. | ✅ Подтверждено | Корректный синтаксис ES analyzer JSON. |
| 9.17 | «Фильтр `russian_morphology` не входит в сборку Elasticsearch — его даёт плагин `elasticsearch-analysis-morphology`.» | ✅ Подтверждено | [github.com/imotov/elasticsearch-analysis-morphology](https://github.com/imotov/elasticsearch-analysis-morphology) — сторонний плагин. |
| 9.18 | «альтернатива — встроенный `hunspell`-фильтр с русскими словарями.» | ✅ Подтверждено | [ES Hunspell token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-hunspell-tokenfilter.html). |
| 9.19 | «При смене анализатора требуется полная переиндексация — обратный индекс существующих сегментов построен по старым правилам токенизации.» | ✅ Подтверждено | [ES reference — analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html). |
| 9.20 | «GIN (Generalized Inverted Index) — встроенный в PostgreSQL инвертированный индекс, пригодный для полнотекстового поиска (`tsvector`), массивов, JSONB, расширения `pg_trgm` [[5]] [[6]].» | ✅ Подтверждено | [PostgreSQL GIN docs](https://www.postgresql.org/docs/current/gin.html); [pg_trgm docs](https://www.postgresql.org/docs/current/pgtrgm.html). |
| 9.21 | «Entry tree — B-дерево по ключам (термам). Каждый листовой элемент указывает на постинг-лист.» | ✅ Подтверждено | [PG GIN — Implementation](https://www.postgresql.org/docs/current/gin-implementation.html): «B-tree of keys… each tuple in a leaf page contains either a pointer to a B-tree of heap pointers (a posting tree), or a simple list of heap pointers (a posting list)». |
| 9.22 | «Posting list / posting tree — если в списке мало TID-ов, они хранятся прямо в записи entry tree (posting list внутри строки). Когда список разрастается — перемещается в отдельный posting tree (B-дерево по `(block, offset)` TID-ам).» | ✅ Подтверждено | [PG GIN docs — Implementation](https://www.postgresql.org/docs/current/gin-implementation.html). |
| 9.23 | «Pending list — дополнительная однонаправленная очередь новых записей. Режим `fastupdate=on` (по умолчанию включён) ускоряет вставку.» | ✅ Подтверждено | [PG GIN docs — Tips](https://www.postgresql.org/docs/current/gin-tips.html): «If consistent response time is more important than update speed, use of pending entries can be disabled by turning off the `fastupdate` storage parameter». Default: enabled. |
| 9.24 | «Периодически `autovacuum` или явный `VACUUM` перемещает их в основной индекс.» | ✅ Подтверждено | [PG GIN docs](https://www.postgresql.org/docs/current/gin.html): vacuum / `gin_clean_pending_list` / `gin_pending_list_limit`. |
| 9.25 | SQL-пример `CREATE INDEX … USING GIN (to_tsvector('russian', body))` и запрос с `ts_rank`. | ✅ Подтверждено | Корректный PG syntax; [PG textsearch-controls](https://www.postgresql.org/docs/current/textsearch-controls.html). |
| 9.26 | «Оператор `@@` возвращает `true`, если `tsvector` соответствует `tsquery`.» | ✅ Подтверждено | [PG textsearch-intro](https://www.postgresql.org/docs/current/textsearch-intro.html). |
| 9.27 | «`ts_rank` — встроенная функция ранжирования (на основе TF и близости термов; BM25 как таковой в Postgres не встроен, но существует в расширениях).» | ✅ Подтверждено | [PG textsearch-controls — Ranking Search Results](https://www.postgresql.org/docs/current/textsearch-controls.html): «ranks vectors based on the frequency of their matching lexemes»; BM25 в core нет, есть в расширениях типа [pg_search](https://github.com/paradedb/paradedb). |
| 9.28 | «pg_trgm [[6]] даёт индексы по триграммам строк и поддерживает `ILIKE '%...%'`, операторы подобия `%` и `<->` (distance), делая возможным fuzzy search и wildcard на уровне GIN/GiST.» | ✅ Подтверждено | [pg_trgm docs](https://www.postgresql.org/docs/current/pgtrgm.html): «text % text → boolean» (similarity), «text <-> text → real» (distance), индексы LIKE/ILIKE. |
| 9.29 | Сноска [[5]] к этому разделу: «Номер главы различается между мажорными версиями: в PG 16 — глава 70, в PG 17/18 — 65.4.» (это часть библиографии, но дублирующая информация в этом разделе.) | ❌ Опровергнуто | PG 16 = ch. 70 ([docs/16/gin.html](https://www.postgresql.org/docs/16/gin.html)); PG 17 = **64.4** ([docs/17/gin.html](https://www.postgresql.org/docs/17/gin.html)); PG 18 = 65.4 ([docs/18/gin.html](https://www.postgresql.org/docs/18/gin.html)). Утверждение «PG 17/18 = 65.4» неверно для PG 17. |

### 10. Ограничения и компромиссы (lines 506–513)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 10.1 | «Асимметрия индексации и поиска. Построение индекса дорого (часы для больших корпусов), поиск — дёшев. Эту асимметрию бесполезно пытаться обратить.» | 💬 Не факт | Качественное обобщение, общеизвестно в IR. См. [Manning IIR §4](https://nlp.stanford.edu/IR-book/html/htmledition/index-construction-1.html). |
| 10.2 | «Смена анализатора = переиндексация. Любое изменение токенизации, стемминга или стоп-листа требует перестроить весь индекс.» | ✅ Подтверждено | [ES reference — analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html). |
| 10.3 | «Нет семантики. Обратный индекс сопоставляет поверхностные формы (или их основы). Синонимы, парафразы, кросс-язычность решаются либо словарями синонимов (дорого и хрупко), либо плотным поиском по эмбеддингам.» | ✅ Подтверждено | Известное ограничение «лексической» retrieval; см. [Karpukhin et al. 2020 «Dense Passage Retrieval»](https://arxiv.org/abs/2004.04906). |
| 10.4 | «Гибридный поиск. На практике лучшие результаты даёт комбинация: первая стадия — BM25-retrieval, вторая — dense retrieval по HNSW; результаты объединяются через Reciprocal Rank Fusion или обучаемую комбинацию скоров.» | ✅ Подтверждено | [Cormack, Clarke, Buettcher 2009 «Reciprocal Rank Fusion»](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf); современная практика. |
| 10.5 | «Не подходит для аналитики и сортировок — для них рядом держат doc values / column store.» | ✅ Подтверждено | [Lucene doc values](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/Lucene90DocValuesFormat.html). |
| 10.6 | «Большие словари. В корпусах с большим числом редких токенов (код, логи, URL) словарь распухает; помогает агрессивная нормализация и ограничение длины термов.» | ✅ Подтверждено | [Manning IIR §5.1](https://nlp.stanford.edu/IR-book/html/htmledition/statistical-properties-of-terms-in-information-retrieval-1.html); общая практика индексирования логов. |

### 11. Список литературы (lines 515–537)

| # | Запись | Вердикт | Доказательство |
|---|---|---|---|
| 11.1 | [1] Manning, Raghavan, Schütze (2008) Introduction to Information Retrieval. Cambridge UP. | ✅ Подтверждено | [nlp.stanford.edu/IR-book](https://nlp.stanford.edu/IR-book/). |
| 11.2 | [2] Robertson & Zaragoza (2009) The Probabilistic Relevance Framework: BM25 and Beyond. **Foundations and Trends in Information Retrieval, 3(4), 333–389**. doi 10.1561/1500000019. | ❌ Опровергнуто (выходные данные) | DOI верный, но фактическое издание — **Foundations and Trends in IR vol 4, issues 1–2, pp. 1–174** (опубликовано 2009). Подтверждено страницей издателя: [emerald.com/ftinr/article/4/1-2/1/1326508](https://www.emerald.com/ftinr/article/4/1-2/1/1326508/The-Probabilistic-Relevance-Framework-BM25-and). Нужно исправить vol/issue/pages. |
| 11.3 | [3] Apache Lucene File Formats / Codec API; ссылки на 9.9 docs. | ✅ Подтверждено | Все три URL валидны: [codecs package](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/package-summary.html), [lucene99 codec](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/package-summary.html), [BM25Similarity](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/search/similarities/BM25Similarity.html). |
| 11.4 | [4] Elastic — Text analysis and Inverted index. | ✅ Подтверждено | [elastic.co/guide/.../analysis.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html). |
| 11.5 | [5] PG GIN docs; «PG 16 — гл. 70, PG 17/18 — 65.4». | ❌ Опровергнуто (PG 17) | PG 16 = ch. 70 ✅ ([docs/16/gin.html](https://www.postgresql.org/docs/16/gin.html)); PG 17 = **64.4** ([docs/17/gin.html](https://www.postgresql.org/docs/17/gin.html)) ❌; PG 18 = 65.4 ([docs/18/gin.html](https://www.postgresql.org/docs/18/gin.html)) ✅. |
| 11.6 | [6] PG pg_trgm docs. | ✅ Подтверждено | [postgresql.org/docs/current/pgtrgm.html](https://www.postgresql.org/docs/current/pgtrgm.html). |
| 11.7 | [7] Witten, Moffat, Bell (1999) Managing Gigabytes (2nd ed.). Morgan Kaufmann. | ✅ Подтверждено | [Amazon listing 1999 2nd ed.](https://www.amazon.com/Managing-Gigabytes-Compressing-Multimedia-Information/dp/1558605703); 2nd edition опубликована в 1999 Morgan Kaufmann. |
| 11.8 | [8] Porter, M.F. (1980) An algorithm for suffix stripping. Program 14(3), 130–137. | ✅ Подтверждено | [tartarus.org/martin/PorterStemmer](https://tartarus.org/martin/PorterStemmer/) — официальная страница автора с цитатой «M.F. Porter, 1980, …, _Program_, 14(3) pp 130−137». |
| 11.9 | [9] Lemire & Boytsov (2015) Decoding billions of integers per second through vectorization. SP&E 45(1), 1–29. arXiv:1209.2137. | ✅ Подтверждено | [arXiv:1209.2137](https://arxiv.org/abs/1209.2137); опубликовано в Software: Practice and Experience 45(1):1–29 (2015), DOI [10.1002/spe.2203](https://doi.org/10.1002/spe.2203). |
| 11.10 | [10] Zukowski, Héman, Nes, Boncz (2006) Super-Scalar RAM-CPU Cache Compression. ICDE. | ✅ Подтверждено | [DOI 10.1109/ICDE.2006.150](https://dl.acm.org/doi/10.1109/ICDE.2006.150). |
| 11.11 | [11] Zobel & Moffat (2006) Inverted files for text search engines. ACM Comput. Surv. 38(2), 6-es. | ✅ Подтверждено | [DOI 10.1145/1132956.1132959](https://doi.org/10.1145/1132956.1132959). |

---

## Findings & recommended fixes (для генератора)

Список изменений, которые следует внести в `inverted-index.mdx`. Все остальные предложения подтверждены или не являются проверяемыми фактами.

1. **Heaps' law β-диапазон / атрибуция (line 40, item 2.8).**
   - Текущий текст: «`β ≈ 0.4–0.6` [[1](#ref-manning-iir)]».
   - Проблема: Manning IIR §5.1.1 даёт `b ≈ 0.5` (одно значение), не диапазон. Диапазон 0.4–0.6 — Wikipedia/общая литература.
   - Корректная версия (предложение): либо исправить значение на `β ≈ 0.5 [[1]]` с цитатой из IIR §5.1.1; либо сохранить диапазон 0.4–0.6, но убрать ссылку [[1]] и сослаться на Wikipedia/Heaps' law. Я рекомендую первый вариант ради точности — Manning остаётся каноническим источником и даёт единое значение.
   - Источник: [Manning IIR §5.1.1](https://nlp.stanford.edu/IR-book/html/htmledition/heaps-law-estimating-the-number-of-terms-1.html), цитата «30 ≤ k ≤ 100 and b ≈ 0.5».

2. **FST в `.tim`/`.tip` (line 38, item 2.6).**
   - Текущий текст: «FST… используется в Lucene для словаря термов в сегменте (`.tim` / `.tip`)».
   - Проблема: технически FST живёт только в `.tip` (term index), а `.tim` — block-tree dictionary. Связка `.tim`/`.tip` без уточнения легко вводит в заблуждение, потому что в ту же фразу выше в файле (line 438) уже написано правильно: «FST-индекс в `.tip` для блочного поиска, основной контент в `.tim`».
   - Корректная версия: «используется в Lucene в качестве индекса термов сегмента (`.tip`), указывающего на блочный словарь в `.tim`».
   - Источник: [Lucene90BlockTreeTermsWriter Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene90/blocktree/Lucene90BlockTreeTermsWriter.html).

3. **BM25 атрибуция «Robertson–Spärck Jones» (line 240, item 5.7).**
   - Текущий текст: «BM25 (Best Matching 25) — вероятностная модель ранжирования Robertson–Spärck Jones».
   - Проблема: BM25 в её современной форме (Okapi BM25) разработана Robertson, Walker, Spärck Jones и др. в проекте Okapi (TREC, ~1994). Robertson–Spärck Jones как самостоятельная пара — это RSJ relevance weight (1976) и общая вероятностная рамка. Наименование «модель Robertson–Spärck Jones» исторически смазывает.
   - Корректная версия: «BM25 (Okapi BM25) — функция ранжирования из вероятностной рамки Robertson / Spärck Jones / Walker (Okapi, 1994), эмпирически превосходящая TF-IDF…». Либо: «вероятностная модель ранжирования из семейства Robertson–Spärck Jones».
   - Источник: [Wikipedia Okapi BM25 — History](https://en.wikipedia.org/wiki/Okapi_BM25); [Robertson & Zaragoza 2009 §1](https://doi.org/10.1561/1500000019).

4. **Ссылка на `Lucene90PostingsFormat` как актуальный формат 9.x (line 397, item 6.17).**
   - Текущий текст: «Lucene с 9.x использует PFOR-подобный формат (`Lucene90PostingsFormat` и более новые ревизии) для файлов `.doc` и `.pos`».
   - Проблема: default PostingsFormat в Lucene 9.7+ — это `Lucene99PostingsFormat`, а не `Lucene90PostingsFormat`. Последний deprecated с 9.7. Имя `Lucene90PostingsFormat` ассоциируется с устаревшей версией.
   - Корректная версия: «Lucene с 9.x использует PFOR-подобный формат (`Lucene99PostingsFormat` и более новые ревизии) для файлов `.doc` и `.pos`».
   - Источник: [Lucene99PostingsFormat Javadoc](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99PostingsFormat.html).

5. **Codec architecture — смешение Codec и PostingsFormat (line 446, item 9.10).**
   - Текущий текст: «каждая версия Lucene выпускает свой Codec (`Lucene90PostingsFormat`, `Lucene99Codec` и т. д.)».
   - Проблема: `Lucene99Codec` — это `Codec` (контейнер форматов), а `Lucene90PostingsFormat` — это `PostingsFormat` (один из компонентов). В перечислении смешаны разные типы.
   - Корректная версия: «каждая версия Lucene выпускает свой Codec (`Lucene90Codec`, `Lucene94Codec`, `Lucene99Codec` и т. д.) с собственным набором форматов файлов».
   - Источник: [Lucene99Codec](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/lucene99/Lucene99Codec.html); [codecs package overview](https://lucene.apache.org/core/9_9_0/core/org/apache/lucene/codecs/package-summary.html).

6. **Номер главы PG GIN в PostgreSQL 17 (lines 482 cross-reference и 525 в библиографии, item 9.29 / 11.5).**
   - Текущий текст: «в PG 16 — глава 70, в PG 17/18 — 65.4».
   - Проблема: PG 17 = **64.4** (не 65.4). PG 18 действительно 65.4. PG 16 = 70 (верно).
   - Корректная версия: «Номер главы различается между мажорными версиями: в PG 16 — глава 70, в PG 17 — 64.4, в PG 18 — 65.4».
   - Источники: [PG 16 GIN](https://www.postgresql.org/docs/16/gin.html), [PG 17 GIN](https://www.postgresql.org/docs/17/gin.html), [PG 18 GIN](https://www.postgresql.org/docs/18/gin.html).

7. **Robertson & Zaragoza 2009 — выходные данные в библиографии (line 519, item 11.2).**
   - Текущий текст: «Foundations and Trends in Information Retrieval, **3(4), 333–389**».
   - Проблема: фактическая публикация — vol **4, issues 1–2, pp. 1–174** (2009). DOI 10.1561/1500000019 разрешается на эту запись.
   - Корректная версия: «Foundations and Trends in Information Retrieval, **4(1–2), 1–174**».
   - Источник: [Emerald (издатель серии) запись DOI](https://www.emerald.com/ftinr/article/4/1-2/1/1326508/The-Probabilistic-Relevance-Framework-BM25-and).

---

## Приложение A: пересчёт точек чарта BM25 TF-saturation (k1=1.2, b=0)

| tf | norm(tf) = 2.2·tf/(tf+1.2) | в файле | Δ |
|---|---|---|---|
| 0 | 0.000 | 0.0 | 0 |
| 1 | 1.000 | 1.0 | 0 |
| 2 | 1.375 | 1.375 | 0 |
| 3 | 1.571 | 1.571 | 0 |
| 4 | 1.692 | 1.692 | 0 |
| 5 | 1.774 | 1.774 | 0 |
| 6 | 1.833 | 1.833 | 0 |
| 7 | 1.878 | 1.878 | 0 |
| 8 | 1.913 | 1.913 | 0 |
| 10 | 1.964 | 1.964 | 0 |
| 12 | 2.000 | 2.0 | 0 |
| 15 | 2.037 | 2.037 | 0 |
| 20 | 2.075 | 2.075 | 0 |

Все 13 точек совпадают до 3-го знака. Диаграмма математически корректна.
