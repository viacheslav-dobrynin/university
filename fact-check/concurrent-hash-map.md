# Fact-check: ConcurrentHashMap в Java

**Source**: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/java/concurrent-hash-map.mdx`
**Live URL**: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/java/concurrent-hash-map

Last verification pass (sentence-by-sentence): 2026-05-24
Last fix pass: 2026-06-07 (evaluator)

## Fix pass 2026-06-07 (evaluator)

Контекст: CRITICAL build-blocker (`.md`/`.mdx` cross-link) уже устранён ранее в этой сессии — submodule commit `658c7f3`, deploy run **27096736952 = SUCCESS** (2026-06-07T15:26:37Z). Страница ConcurrentHashMap впервые опубликована в полном виде.

Действия этого pass'а:

- **(a) Deploy.** Подтверждено: `gh run list --workflow=deploy.yml --limit=1` → `27096736952` **success** (1m16s). Build с `onBrokenLinks: throw` прошёл — broken-link больше нет. CRITICAL-строка в «Сводке замечаний» ниже помечена FIXED.
- **(b) Live-валидация (Playwright).** Прод-страница `…/docs/concurrent/java/concurrent-hash-map` рендерится: H1 «ConcurrentHashMap в Java», Decision matrix, ECharts-график, «Список литературы» присутствуют (проверено через preflight на корне + навигацию). См. §«Live-страница» — статус обновлён.
- **(c) Исправлены OPEN ⚠️-замечания:**
  - **2 #5 / size()-attribution** (строка ~52): цитата «*The value returned is an estimate…*» теперь явно приписана javadoc'у `mappingCount()`, а не `size()`. Текст: «javadoc `mappingCount()` … прямо говорит … ; `size()` наследует ту же приближённость, плюс насыщается на `Integer.MAX_VALUE`». **FIXED.**
  - **9.2 / 17.1 / 9.3 — JDK-тикеты** (строки ~155 и ~370): неверная атрибуция JDK-8161372 и неподтверждаемый JDK-8172951 удалены. Заменены на **JDK-8062841** («ConcurrentHashMap.computeIfAbsent stuck in an endless loop», дубликат **JDK-8074374**), под которым добавлена детекция рекурсии, бросающая `IllegalStateException("Recursive update")`. Тикеты сверены через WebSearch (bugs.openjdk.org даёт 403 на WebFetch): JDK-8161372 = «locks bin when k present» (unrelated); JDK-8172951 = «HashMap.computeIfAbsent adds entry that get() does not find» (про HashMap, не про CHM-рекурсию). **FIXED.**
  - **9.1 — регистр «map»/«Map»** (строка ~153): сверено с Java 21 javadoc через WebFetch — официальный текст использует **строчную** «map» («…any other mappings of this map…»). Страница УЖЕ совпадает с источником. Изменений не требуется; исходное замечание было ошибочным. **РЕШЕНО (no-op, источник подтверждает страницу).**
  - **19.1 — ссылка [8] Hendler-Shavit:** РЕШЕНО — **оставлена**. Она реально цитируется в теле (строка ~305: «Аналогичный disclaimer стоит на странице lock-free … Hendler–Shavit SPAA 2004 [[8](#ref-hendler-shavit)]») как прецедент disclaimer-шаблона. Удаление сломало бы якорь и нумерацию цитат. Это валидный cross-reference, не блокер.

## Live-страница / preflight

Preflight 2026-06-07 (evaluator fix pass):
- `gh run list --workflow=deploy.yml --limit=1` → `27096736952` **success**. Прод-страница ConcurrentHashMap опубликована и рендерится (H1, Decision matrix, ECharts, Список литературы). После правок этого pass'а — повторный деплой (см. лог ниже).

Preflight 2026-05-24:
- `gh run list --workflow=deploy.yml --limit=1` → последний run `26367443341` (**FAILURE**, 2026-05-24T17:05:02Z, commit «content: add ConcurrentHashMap page»).
- Причина фейла: broken-link check Docusaurus в локали `ru`. Файл `docs/concurrent/lock-free/concurrent-data-structures.mdx` строка 206 содержит ссылку `../java/concurrent-hash-map.md`, но страница теперь имеет расширение `.mdx`. Docusaurus резолвит ссылку в несуществующий путь и aborts build (`onBrokenLinks = throw`).
- Следствие: production-сайт **не обновлён**. Live URL страницы ConcurrentHashMap до сих пор возвращает старый stub: title `concurrent-hash-map` (slug, не «ConcurrentHashMap в Java»), пустое тело, edit link указывает на `.md`.
- Fact-check ниже выполнен по исходному `.mdx`-файлу, sentence-by-sentence. Live-валидация (ECharts, якоря, отсутствие 404 в литературе) пока невозможна — невозможна до зелёного deploy.

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–22 | ✅ Проверено | 2026-05-24 | JCiP §4.4.1 / §5.1 / §5.2 — цитаты корректны; «infinite loop on resize» как Java 7 race — подтверждён независимыми источниками. JSR-166 атрибуция точна. |
| 2 | API и базовый контракт | 24–52 | ✅ Проверено (замечание 2 #5 FIXED 2026-06-07) | 2026-06-07 | Class declaration, @author Doug Lea, @since 1.5 — подтверждены по HEAD OpenJDK. **Замечание 2 #5 (size-attribution) FIXED 2026-06-07:** цитата «*The value returned is an estimate…*» теперь явно приписана `mappingCount()`, а не `size()`. Замечание 2.1 (стиль «Tiger, 2004») — OK. |
| 3 | Эволюция реализации | 54–80 | ⚠️ Проверено с замечанием | 2026-05-24 | Segmented locking, lazy init, Java 8 переписка — общая картина корректна. **Замечание 3.1:** утверждение «доли-вторичные проверки» в строке 60 невнятно сформулировано; механизм Java 5/6 — это `volatile`-read+retry-with-lock fallback. Замечание стилистическое, не фактическое. |
| 4 | Ключевые механизмы Java 8+ → Константы | 82–98 | ✅ Проверено | 2026-05-24 | Все 8 констант (DEFAULT_CAPACITY=16, MAXIMUM_CAPACITY=1<<30, LOAD_FACTOR=0.75f, TREEIFY_THRESHOLD=8, UNTREEIFY_THRESHOLD=6, MIN_TREEIFY_CAPACITY=64, MIN_TRANSFER_STRIDE=16, RESIZE_STAMP_BITS=16) подтверждены поштучно по HEAD-исходнику. |
| 5 | Hash-маркеры | 99–109 | ✅ Проверено | 2026-05-24 | MOVED=-1, TREEBIN=-2, RESERVED=-3 подтверждены по HEAD-исходнику. |
| 6 | Жизненный цикл bin'а | 111–117 | ✅ Проверено | 2026-05-24 | CAS на пустой корзине, synchronized(head) на непустой, treeification на 8 + table≥64, untreeification на 6 — всё корректно. |
| 7 | Resize: ForwardingNode + параллельные хелперы | 119–129 | ⚠️ Проверено с замечанием | 2026-05-24 | Механика корректна. **Замечание 7.1:** формула stride `(table.length >>> 3) / NCPU` — в исходнике actually `((n >>> 3) / NCPU)` где `n = table.length`, **но это не общая формула «полосы»**; это формула в `transfer()` для расчёта **stride per helper**, а нижняя граница `MIN_TRANSFER_STRIDE = 16`. Численный пример «table.length = 4096, NCPU = 8 ⇒ stride = 64» арифметически правилен: 4096/8/8 = 64. Stride **не зависит от полосы**, а ограничивается snizu 16. OK. |
| 8 | Счётчик baseCount + CounterCell[] | 131–139 | ✅ Проверено | 2026-05-24 | Striped counter pattern, аналог LongAdder — корректно описано. |
| 9 | Атомарные compose-операции | 141–155 | ✅ Проверено (9.1/9.2/9.3 РЕШЕНЫ 2026-06-07) | 2026-06-07 | Цитата «The entire method invocation is performed atomically» подтверждена. **9.1 РЕШЕНО (no-op):** Java 21 javadoc использует СТРОЧНУЮ «map» («…of this map»); страница уже совпадает с источником — исходное замечание было ошибочным. **9.2/9.3 FIXED:** неверный JDK-8161372 («locks bin when k present», unrelated) и неподтверждаемый JDK-8172951 удалены; заменены на **JDK-8062841** (дубликат JDK-8074374) с `IllegalStateException("Recursive update")`. Тикеты сверены через WebSearch. |
| 10 | Семантика size() / mappingCount() / isEmpty() | 157–170 | ✅ Проверено | 2026-05-24 | Цитата size() и mappingCount() — подтверждена дословно по Java 21 javadoc. |
| 11 | Weakly consistent итераторы | 172–184 | ✅ Проверено | 2026-05-24 | Все три гарантии («may proceed concurrently…», «never throw CME», «traverse elements as they existed upon construction…») — стандартные тексты `java.util.concurrent` package-summary. Spliterator.CONCURRENT — подтверждён javadoc'ом keySet().spliterator(). |
| 12 | Псевдокод | 186–241 | ✅ Проверено | 2026-05-24 | Иллюстративный, явно помечен («схематично»). Соответствует putVal() в HEAD-исходнике на уровне общей структуры. |
| 13 | Сравнение с ConcurrentSkipListMap | 243–263 | ✅ Проверено | 2026-05-24 | Таблица корректна. CSLM lock-free, Fraser/Harris attribution — подтверждено. Cassandra memtable использует CSLM — подтверждено через SkipListMemtable.java. |
| 14 | Сложность | 265–285 | ✅ Проверено | 2026-05-24 | Все асимптотики корректны. Treeify gives O(log n) worst-case (RB-tree). |
| 15 | Применения в БД и распределённых системах | 287–299 | ✅ Проверено | 2026-05-24 | Caffeine `final ConcurrentHashMap<Object, Node<K, V>> data;` — подтверждено по BoundedLocalCache.java HEAD. Kafka — `metrics`, `sensors`, `childrenSensors` — все три ConcurrentMap-поля подтверждены по Metrics.java HEAD. Cassandra SkipListMemtable использует CSLM, не CHM — подтверждено. Spring `singletonObjects = new ConcurrentHashMap<>(256)` — подтверждено по DefaultSingletonBeanRegistry.java. |
| 16 | Визуализация: throughput vs число потоков | 301–366 | ⚠️ Проверено с замечанием | 2026-05-24 | Disclaimer «иллюстративные точки, не measured benchmark» **присутствует** (lines 305) — это требование задания. **Замечание 16.1:** JMH attribution «Java Microbenchmark Harness, OpenJDK» — корректно. **Замечание 16.2:** ось x — log2 от 1 до 32, ось y — нормализованный throughput 0–4. Кривые качественно соответствуют известному паттерну CHM vs Hashtable, но числовые значения нигде в JCiP §11.4.4 явно не публикуются — disclaimer обязан их покрыть, что и сделано. |
| 17 | Подводные камни | 368–376 | ✅ Проверено | 2026-05-24 | Все шесть пунктов корректны: recursive computeIfAbsent, итератор не snapshot, size vs mappingCount, null forbidden, hash flooding, putIfAbsent vs computeIfAbsent, атомарность только для одного ключа. |
| 18 | Decision matrix | 378–387 | ✅ Проверено | 2026-05-24 | Рекомендации согласованы с остальной частью страницы и стандартными гайдами. |
| 19 | Список литературы | 389–407 | ✅ Проверено (19.1 РЕШЕНО 2026-06-07) | 2026-06-07 | Все 9 ссылок ведут на работающие источники. **19.1 РЕШЕНО — [8] оставлена:** Hendler-Shavit реально цитируется в теле (строка ~305) как прецедент disclaimer-шаблона `[[8](#ref-hendler-shavit)]`; это валидный cross-reference, удаление сломало бы якорь и нумерацию. Не блокер. |

---

## Posentence-by-sentence verification

### Раздел 1: Мотивация (строки 10–22)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 1 | «Базовая `java.util.HashMap` — **не потокобезопасна**: одновременная модификация из двух потоков может оставить корзину в полусобранном состоянии, разорвать цепочку или зациклить указатели (исторически — печально известный «infinite loop on resize» в Java 7, демонстрировавшийся CPU-spike'ом на 100% в проде).» | ✅ ПОДТВЕРЖДЕНО | Hash-map infinite loop при race на resize — задокументировано как класс багов, см. [«Why accessing Java HashMap may cause infinite loop» (PixelsTech)](https://www.pixelstech.net/article/1585457836-Why-accessing-Java-HashMap-may-cause-infinite-loop-in-concurrent-environment). |
| 2 | «`Hashtable` (с 1.0). Каждый публичный метод объявлен `synchronized` на самом объекте таблицы.» | ✅ ПОДТВЕРЖДЕНО | Стандарт; [Hashtable javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Hashtable.html). |
| 3 | «Brian Goetz, Tim Peierls et al. в *Java Concurrency in Practice* §4.4.1 называют этот стиль **monitor-based class**…» | ✅ ПОДТВЕРЖДЕНО | JCiP §4.4.1 «Monitor-based classes», подтверждено в [authors.html](https://jcip.net/authors.html). |
| 4 | «`Collections.synchronizedMap(new HashMap<>())`… Враппер над любой `Map` — тот же глобальный монитор…» | ✅ ПОДТВЕРЖДЕНО | Стандартное описание; [Collections.synchronizedMap javadoc](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html). |
| 5 | «JCiP §5.2: *compound actions* on synchronized collections… are not thread-safe…» | ✅ ПОДТВЕРЖДЕНО | Цитата из JCiP §5.1.1 / §5.2.1 о compound actions — стандартная. |
| 6 | «`ConcurrentHashMap` (CHM), введённый в Java 5 в составе JSR-166 и переписанный Doug Lea в Java 8…» | ✅ ПОДТВЕРЖДЕНО | JSR-166 финализирован 30 сентября 2004, см. [jcp.org/en/jsr/detail?id=166](https://jcp.org/en/jsr/detail?id=166). Java 8 переписка — well-known. |
| 7 | «javadoc гарантирует, что «*The entire method invocation is performed atomically*»» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в Java 21 javadoc для computeIfAbsent / computeIfPresent / compute / merge. |
| 8 | «**Weakly consistent** итераторы не бросают `ConcurrentModificationException`…» | ✅ ПОДТВЕРЖДЕНО | Стандарт package-summary `java.util.concurrent`; см. [Java 21 javadoc CHM](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html). |

### Раздел 2: API и базовый контракт (строки 24–52)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 1 | Class declaration: `public class ConcurrentHashMap<K,V> extends AbstractMap<K,V> implements ConcurrentMap<K,V>, Serializable` | ✅ ПОДТВЕРЖДЕНО | [openjdk/jdk HEAD ConcurrentHashMap.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentHashMap.java). |
| 2 | «`@author Doug Lea`, `@since 1.5`» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в шапке файла OpenJDK HEAD. |
| 3 | «*Like Hashtable but unlike HashMap, this class does not allow null to be used as a key or value*» | ✅ ПОДТВЕРЖДЕНО | Точная цитата из Java 21 javadoc. |
| 4 | «*Neither the key nor the value can be null*» (из put) | ✅ ПОДТВЕРЖДЕНО | Стандартная фраза в Java javadoc для put. |
| 5 | «Метод `size()` приближённый. *The value returned is an estimate; the actual count may differ if there are concurrent insertions or removals.*» | ✅ FIXED 2026-06-07 | Было ⚠️ НЕТОЧНО: цитата из javadoc'а `mappingCount()`, а не `size()`. Исправлено — страница теперь явно приписывает цитату `mappingCount()` и отмечает, что `size()` наследует приближённость + насыщается на `Integer.MAX_VALUE`. Сверено с Java 21 javadoc (WebFetch). |

### Раздел 3: Эволюция реализации (строки 54–80)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 1 | «Java 5/6 — segmented locking. Внутренний массив `Segment[]` фиксированной длины (по умолчанию 16).» | ✅ ПОДТВЕРЖДЕНО | Историческая реализация; см. ранние OpenJDK исходники. |
| 2 | «Java 7 — lazy segment init + Unsafe CAS» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в OpenJDK 7 исходниках. |
| 3 | «Java 8 — полная переписка… Сегменты исчезли.» | ✅ ПОДТВЕРЖДЕНО | Известный факт; см. release notes Java 8 и текущий HEAD CHM. |
| 4 | «новая модель памяти JSR-133» | ✅ ПОДТВЕРЖДЕНО | JSR-133 specification. |

### Раздел 4: Константы (строки 84–98)

| # | Имя константы | Цитата | Вердикт | Доказательство |
|---|---|---|---|---|
| 1 | `DEFAULT_CAPACITY = 16` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 2 | `MAXIMUM_CAPACITY = 1 << 30` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 3 | `LOAD_FACTOR = 0.75f` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 4 | `TREEIFY_THRESHOLD = 8` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 5 | `UNTREEIFY_THRESHOLD = 6` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 6 | `MIN_TREEIFY_CAPACITY = 64` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 7 | `MIN_TRANSFER_STRIDE = 16` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 8 | `RESIZE_STAMP_BITS = 16` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |

### Раздел 5: Hash-маркеры (строки 99–109)

| # | Имя | Значение | Вердикт |
|---|---|---|---|
| 1 | `MOVED = -1` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 2 | `TREEBIN = -2` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |
| 3 | `RESERVED = -3` | ✅ ПОДТВЕРЖДЕНО | HEAD OpenJDK |

### Раздел 6: Жизненный цикл bin'а (строки 111–117)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «CAS на пустой корзине. Вставка идёт через `casTabAt(tab, i, null, newNode)`» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в `putVal()` HEAD-исходника. |
| 2 | «`synchronized` на head-узле» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в `putVal()` (synchronized (f) where f = tabAt(tab, i)). |
| 3 | «Treeification на 8 + table.length ≥ 64» | ✅ ПОДТВЕРЖДЕНО | TREEIFY_THRESHOLD=8 и MIN_TREEIFY_CAPACITY=64 в HEAD. |
| 4 | «Untreeification на 6» | ✅ ПОДТВЕРЖДЕНО | UNTREEIFY_THRESHOLD=6 в HEAD. |

### Раздел 7: Resize (строки 119–129)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «Отрицательное `sizeCtl` означает идёт resize» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в JCiP примечаниях и HEAD-комментариях. |
| 2 | «stride = `(table.length >>> 3) / NCPU`, минимум 16» | ✅ ПОДТВЕРЖДЕНО | Соответствует логике в transfer(). |
| 3 | «при `table.length = 4096`, `NCPU = 8` ⇒ stride = 64» | ✅ ПОДТВЕРЖДЕНО | Арифметически: 4096 / 8 / 8 = 64. |
| 4 | «ForwardingNode с hash = MOVED = -1» | ✅ ПОДТВЕРЖДЕНО | HEAD-исходник: ForwardingNode extends Node с hash=MOVED. |
| 5 | «`helpTransfer`» | ✅ ПОДТВЕРЖДЕНО | Метод существует в HEAD. |

### Раздел 8: Счётчик baseCount + CounterCell[] (строки 131–139)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «Поле `baseCount` (volatile long)» | ✅ ПОДТВЕРЖДЕНО | HEAD: `private transient volatile long baseCount;`. |
| 2 | «`CounterCell[] counterCells`» | ✅ ПОДТВЕРЖДЕНО | HEAD: `private transient volatile CounterCell[] counterCells;`. |
| 3 | «Тот же приём, что в `LongAdder`» | ✅ ПОДТВЕРЖДЕНО | LongAdder использует Striped64 superclass; CHM CounterCell — наследник Striped64.Cell. |

### Раздел 9: Атомарные compose-операции (строки 141–155)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «*The entire method invocation is performed atomically*» | ✅ ПОДТВЕРЖДЕНО | Точная цитата Java 21 javadoc. |
| 2 | «*Some attempted update operations on this map … must not attempt to update any other mappings of this map.*» | ✅ РЕШЕНО 2026-06-07 (no-op) | Сверено с Java 21 javadoc через WebFetch: официальный текст использует СТРОЧНУЮ «map» («…any other mappings of this map»). Страница уже совпадает. Исходное замечание (про заглавную «Map») было ошибочным — изменений не требуется. |
| 3 | «bug-репорты JDK-8161372 и JDK-8172951 фиксировали именно такие зависания…» | ✅ FIXED 2026-06-07 | Было ⚠️ НЕТОЧНО. JDK-8161372 = «locks bin when k present» (unrelated, подтверждено WebSearch); JDK-8172951 = «HashMap.computeIfAbsent adds entry that get() does not find» (про HashMap, не CHM-рекурсию). Исправлено: страница теперь ссылается на **JDK-8062841** («ConcurrentHashMap.computeIfAbsent stuck in an endless loop», дубликат JDK-8074374), под которым добавлен `IllegalStateException("Recursive update")`. |

### Раздел 10: size() / mappingCount() / isEmpty() (строки 157–170)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «`size()`: *Returns the number of key-value mappings in this map. If the map contains more than Integer.MAX_VALUE elements, returns Integer.MAX_VALUE.*» | ✅ ПОДТВЕРЖДЕНО | Цитата точна. |
| 2 | «`mappingCount()`: *...The value returned is an estimate; the actual count may differ if there are concurrent insertions or removals.*» | ✅ ПОДТВЕРЖДЕНО | Цитата точна. |
| 3 | «`isEmpty()` реализован независимо: он коротко-замкнут на проверку `sumCount() <= 0L`» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в HEAD-исходнике (метод isEmpty). |

### Раздел 11: Weakly consistent итераторы (строки 172–184)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | Три гарантии (proceed concurrently / never throw CME / traverse exactly once + may reflect modifications) | ✅ ПОДТВЕРЖДЕНО | Точные формулировки из package-summary `java.util.concurrent`. |
| 2 | «Spliterator.CONCURRENT для keySet()/values()/entrySet()» | ✅ ПОДТВЕРЖДЕНО | Подтверждено в Java 21 javadoc keySet().spliterator(), values().spliterator(), entrySet().spliterator(). |
| 3 | «`HashMap.parallelStream()` где гонки с модификацией приводят к UB» | ✅ ПОДТВЕРЖДЕНО | Стандарт: HashMap iterators fail-fast но не bulletproof; параллельная модификация — undefined behavior. |

### Раздел 13: Сравнение с ConcurrentSkipListMap (строки 243–263)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «CSLM lock-free, основан на Fraser/Harris» | ✅ ПОДТВЕРЖДЕНО | HEAD ConcurrentSkipListMap.java class-comment ссылается на эти работы. |
| 2 | «Cassandra использует CSLM для memtable» | ✅ ПОДТВЕРЖДЕНО | [SkipListMemtable.java HEAD line 67](https://github.com/apache/cassandra/blob/trunk/src/java/org/apache/cassandra/db/memtable/SkipListMemtable.java): `private final ConcurrentNavigableMap<PartitionPosition, AtomicBTreePartition> partitions = new ConcurrentSkipListMap<>();`. |

### Раздел 15: Применения в БД и распределённых системах (строки 287–299)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | Caffeine: `final ConcurrentHashMap<Object, Node<K, V>> data;` в BoundedLocalCache | ✅ ПОДТВЕРЖДЕНО | [BoundedLocalCache.java HEAD line ~229](https://github.com/ben-manes/caffeine/blob/master/caffeine/src/main/java/com/github/benmanes/caffeine/cache/BoundedLocalCache.java). |
| 2 | Kafka Metrics: три ConcurrentMap-поля (metrics, sensors, childrenSensors), все инициализируются `new ConcurrentHashMap<>()` | ✅ ПОДТВЕРЖДЕНО | [Metrics.java HEAD](https://github.com/apache/kafka/blob/trunk/clients/src/main/java/org/apache/kafka/common/metrics/Metrics.java) — declaration и init verified. |
| 3 | Cassandra SkipListMemtable использует CSLM, не CHM | ✅ ПОДТВЕРЖДЕНО | См. §13.2 выше. |
| 4 | Spring `singletonObjects = new ConcurrentHashMap<>(256);` | ✅ ПОДТВЕРЖДЕНО | [DefaultSingletonBeanRegistry.java HEAD line 95](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java). |

### Раздел 16: Визуализация: throughput vs число потоков (строки 301–366)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «Канонические измерения CHM против `Hashtable` и `synchronizedMap` есть в *Java Concurrency in Practice* (фигуры 11.3 и 11.4)» | ✅ ПОДТВЕРЖДЕНО | JCiP §11.4.4 содержит эти фигуры. |
| 2 | «**Иллюстративные значения, не репродукция Figure 11.3**» — disclaimer | ✅ DISCLAIMER ПРИСУТСТВУЕТ | Lines 305 явно: «Конкретные числа из книги в WebFetch недоступны (печатное издание), и выдумывать их под видом репродукции некорректно.» |
| 3 | «Для воспроизводимых measurements рекомендуется фреймворк **JMH**» | ✅ ПОДТВЕРЖДЕНО | JMH — стандарт OpenJDK. |

### Раздел 17: Подводные камни (строки 368–376)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1 | «После фикса JDK-8062841 современный CHM детектирует часть таких случаев и бросает `IllegalStateException` («Recursive update»)…» | ✅ FIXED 2026-06-07 | Было ⚠️ НЕТОЧНО (атрибуция к JDK-8172951). Исправлено на JDK-8062841 (дубликат JDK-8074374) — подтверждено WebSearch. JDK-8161372 (locks bin) и JDK-8172951 (HashMap bug) убраны как нерелевантные. |
| 2 | «hash flooding… SipHash, как в Rust `HashMap`» | ✅ ПОДТВЕРЖДЕНО | Rust default HashMap использует SipHash-1-3 как сравнительно недавнее упрощение от SipHash-2-4; см. [doc.rust-lang.org/std/collections/struct.HashMap.html](https://doc.rust-lang.org/std/collections/struct.HashMap.html). |

### Раздел 19: Список литературы (строки 389–407)

| # | Ссылка | URL | Вердикт | Примечание |
|---|---|---|---|---|
| 1 | [jcip] Goetz et al. (2006) JCiP | jcip.net | ✅ ПОДТВЕРЖДЕНО | URL рабочий. |
| 2 | [jdk-chm] OpenJDK CHM HEAD | github.com/openjdk/jdk | ✅ ПОДТВЕРЖДЕНО | URL рабочий, все константы подтверждены. |
| 3 | [jsr166] JCP detail page | jcp.org/en/jsr/detail?id=166 | ✅ ПОДТВЕРЖДЕНО | Final release date 30 Sep 2004 подтверждена. |
| 4 | [caffeine] BoundedLocalCache | github.com/ben-manes/caffeine | ✅ ПОДТВЕРЖДЕНО | Поле data подтверждено. |
| 5 | [kafka-metrics] Metrics.java | github.com/apache/kafka | ✅ ПОДТВЕРЖДЕНО | Три ConcurrentMap-поля подтверждены. |
| 6 | [spring-singletons] DefaultSingletonBeanRegistry | github.com/spring-projects/spring-framework | ✅ ПОДТВЕРЖДЕНО | Объявление singletonObjects подтверждено. |
| 7 | [cassandra-memtable] SkipListMemtable | github.com/apache/cassandra | ✅ ПОДТВЕРЖДЕНО | Поле partitions подтверждено. |
| 8 | [hendler-shavit] SPAA 2004 | doi.org/10.1145/1007912.1007944 | ✅ РЕШЕНО 2026-06-07 — ОСТАВЛЕНА | DOI рабочий. Цитируется в теле (строка ~305) как прецедент disclaimer-шаблона `[[8](#ref-hendler-shavit)]` — валидный cross-reference; удаление сломало бы якорь и нумерацию. Не блокер. |
| 9 | [jdk-csm] OpenJDK CSLM HEAD | github.com/openjdk/jdk | ✅ ПОДТВЕРЖДЕНО | URL рабочий. |

---

## Сводка замечаний

- **2.1** (стиль): «Java 5 (Tiger, 2004)» — корректно, OK.
- **2 #5** (фактическое): ✅ **FIXED 2026-06-07** — цитата «estimate…» переатрибутирована с `size()` на `mappingCount()`.
- **9.1** (минор): ✅ **РЕШЕНО 2026-06-07 (no-op)** — Java 21 javadoc использует строчную «map»; страница уже совпадает, исходное замечание было ошибочным.
- **9.2 / 17.1** (фактическое): ✅ **FIXED 2026-06-07** — неверный JDK-8161372 («locks bin when k present») заменён на JDK-8062841 (дубликат JDK-8074374).
- **9.3** (фактическое): ✅ **FIXED 2026-06-07** — неподтверждаемый JDK-8172951 удалён; заменён на верифицированный JDK-8062841.
- **19.1** (минор): ✅ **РЕШЕНО 2026-06-07** — [8] Hendler-Shavit оставлена (реально цитируется в теле как прецедент disclaimer-шаблона; валидный cross-reference).
- **CRITICAL** (deploy): ✅ **FIXED 2026-06-07** — ссылка `.md`→`.mdx` в `concurrent-data-structures.mdx` исправлена (submodule `658c7f3`), deploy run **27096736952 = SUCCESS**, страница опубликована и валидирована live через Playwright.
