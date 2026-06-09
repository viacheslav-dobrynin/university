# Fact-check: Неизменяемые и персистентные структуры данных

Source: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/immutable/immutable-vs-persistent.md`
Live URL: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/immutable/immutable-vs-persistent

Last verification pass (sentence-by-sentence): 2026-06-08
Submodule commit: 6428957e1338ce5a4d6a651d34f44ff26a76cfd8 — Parent: a6460048d8cb2c3d2c20d88de5a99dabf2ec3bf8 — Deploy run 27110276803 SUCCESS.

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 8–19 | ✅ Проверено | 2026-06-08 | Определение immutability, четыре выгоды (lock-free чтение, time-travel/снимки, MVCC-снимки, ФП-чистота), центральное противоречие O(n)-копирования и тезис о структурном разделении — корректны. Разграничение с lock-free (RMW над изменяемой структурой vs «никогда не мутируют») верно; дублирования CAS/HP/RCU нет — отослано на lock-free страницу. |
| 2 | Иммутабельность и персистентность: определения | 21–39 | ✅ Проверено | 2026-06-08 | Immutability (свойство объекта) vs persistence (свойство структуры, DSST) разграничены корректно; уточнение «не дисковая персистентность» верно. Таблица таксономии partial/full/confluent/functional (запрос/обновление/топология версий) совпадает с DSST 1989 и литературой по confluent persistence (Fiat–Kaplan): partial=линейная цепочка, full=дерево версий, confluent=DAG (merge), functional=частный случай full без destructive assignment. |
| 3 | Структурное разделение | 41–50 | ✅ Проверено | 2026-06-08 | Path copying O(d) узлов на пути, поддеревья вне пути общие — корректно. Разграничение с наивным COW (весь объект O(n) до первой записи) vs структурное разделение на уровне узлов O(log n) с сохранением разделения после записи — точно. Ремарка «умный node/page-level COW ≈ path copying» корректна. |
| 4 | Методы реализации персистентности | 52–60 | ✅ Проверено | 2026-06-08 | Fat node (список (версия,значение), запрос O(log m) бин.поиском, память O(1)), path copying (O(p) обновление, p=O(log n) для деревьев, O(1) на запрос), node copying/splitting (гибрид, при ограниченной in-degree O(1) аморт. память + O(1) замедление запроса) — все три метода и главный результат DSST верны (подтверждено сверкой с JCSS 1989). |
| 5 | Персистентные списки и стеки | 62–91 | ✅ Проверено | 2026-06-08 | Cons-список = неизменяемый стек, push/pop/peek O(1), хвост никогда не копируется, разделяемый spine — корректно. Арифметика примера пересчитана независимо (см. §A): 4 физических узла хранят 3 версии. |
| 6 | Персистентные сбалансированные деревья | 93–103 | ✅ Проверено | 2026-06-08 | RB/weight-balanced деревья через path copying, O(log n) копируемых узлов, старая версия валидна. Атрибуция OCaml `Map` (сбалансированное двоичное дерево, AVL-стиль) и Haskell `Data.Map` (size-balanced BST) корректна — это стандартные реализации. Таблица lookup/insert/delete O(log n) верна. |
| 7 | HAMT | 105–125 | ✅ Проверено | 2026-06-08 | Bagwell 2001 «Ideal Hash Trees», 5-битные куски хеша, ветвление 32, bitmap присутствия + popcount-индексация компактного массива, глубина ≈ ceil(log32 n)=ceil(log2 n/5). Арифметика n=10^6 пересчитана (см. §A): log2≈19.93, /5≈3.99, ceil=4. Основа map в Clojure/Scala — подтверждено. |
| 8 | Персистентные векторы (bit-partitioned tries) | 127–139 | ✅ Проверено | 2026-06-08 | Bit-partitioned trie по индексу, ветвление 32, O(log32 n) доступ/обновление. Атрибуция Clojure-вектора Ричу Хикки «на основе идей Багвелла, отдельной статьи нет» — корректна и аккуратно сформулирована (Hickey сделал непостоянные структуры Bagwell персистентными). 32^4=1 048 576 ≥ 10^6 ⇒ глубина 4, ~4 копируемых узла (см. §A). RRB-Trees (Bagwell & Rompf 2011): relaxed radix balancing, таблица размеров, O(log n) конкатенация/расщепление при сохранении O(log32 n) доступа — подтверждено. |
| 9 | Finger trees | 141–149 | ✅ Проверено | 2026-06-08 | Hinze & Paterson 2006 (JFP 16(2):197–217, DOI резолвится). 2-3 дерево с пальцами, моноидальная мера → deque/priority-queue/random-access/interval tree. Таблица: концы O(1) аморт., конкатенация/расщепление O(log n) — совпадает с первоисточником. |
| 10 | Амортизация и персистентность: проблема и решение Окасаки | 151–163 | ✅ Проверено | 2026-06-08 | Метод банкира (credits) / физика (потенциал Φ), аморт.=факт.+ΔΦ. Проблема «multiple futures» ломает амортизацию (кредиты тратятся многократно; пример очереди из двух стеков). Решение Окасаки [2]: ленивость + мемоизация (suspension форсируется раз, повторно O(1)), анализ через дебеты, banker's/real-time queue, implicit recursive slowdown. Совпадает с гл. «Amortization and Persistence via Lazy Evaluation» книги 1998 / тезиса CMU-CS-96-177 (1996). |
| 11 | Связь с базами данных и распределёнными системами | 165–173 | ✅ Проверено | 2026-06-08 | MVCC/snapshot isolation (без дублирования вывода — отослано на lock-free); CouchDB append-only B+дерево (новый корень в конец файла, старое дерево валидно, crash-resistant MVCC — подтверждено офиц. докой); LMDB COW B+дерево, single-writer MVCC, читатели и писатель не блокируют друг друга (подтверждено докой Symas/lmdb.tech, дословно); LSM SSTable неизменяемы, compaction создаёт новые файлы. Все три — path-copying на диске. |
| 12 | Сборка мусора и подсчёт ссылок | 175–181 | ✅ Проверено | 2026-06-08 | Разделяемый узел жив пока достижим из любой версии; функциональные структуры = DAG без циклов ⇒ reference counting корректен (нет проблемы циклов). HAMT/векторы на трассирующем GC (JVM/CLR), в Rust (`im`)/C++ — Arc/shared_ptr или EBR. Корректно. |
| 13 | Сложность | 183–197 | ✅ Проверено | 2026-06-08 | Сводная таблица внутренне согласована с разделами выше: список/стек O(1)/O(1); дерево O(log n)/O(log n); HAMT/вектор O(log32 n)≈const; RRB-Tree конкат/расщ O(log n); finger tree концы O(1) аморт., конкат/расщ O(log n). Пояснение «≈ const» корректно отослано на §HAMT/§Векторы. |
| 14 | Список литературы | 199–215 | ✅ Проверено | 2026-06-08 | 8 источников, метаданные сверены (см. §B). Все 8 процитированы inline в стиле bloom-filter (`[[n](#ref-...)]` + `<a id>`). DOI [1] и [5] резолвятся (302 → Elsevier / Cambridge). EPFL-записи [3]/[4] существуют (record/64398, record/169879; transient 429 при автоматическом fetch — не битая ссылка). |

## §A. Независимая перепроверка арифметики

**Cons-список (§Персистентные списки):**
- v1 = [2,1] = 2 узла (`2`, `1`).
- v2 = push(3, v1): новый узел `3` → хвост v1 разделяется.
- v3 = push(4, v1): новый узел `4` → тот же хвост v1 разделяется.
- Физические узлы: {2,1}=2 (общие) + {3}=1 + {4}=1 = **4 узла**, хранящие **3 версии**. ✔ Совпадает с текстом.

**HAMT глубина (n=10^6):**
- log2(10^6) = 6·log2(10) = 6·3.321928 = 19.9316 ≈ 19.93 ✔
- 19.93 / 5 = 3.9863 ≈ 3.99 ✔
- ceil(3.99) = 4 ✔ ⇒ log32(10^6) ≈ 4. Совпадает.

**Персистентный вектор (n=10^6):**
- 32^4 = (2^5)^4 = 2^20 = 1 048 576 ≥ 10^6 ✔ ⇒ глубина 4 ⇒ ~4 копируемых узла на обновление. Совпадает.

## §B. Проверка списка литературы

| Ref | Источник | Статус | Заметка |
|---|---|---|---|
| [1] | Driscoll, Sarnak, Sleator, Tarjan (1989) «Making Data Structures Persistent», JCSS 38(1):86–124, DOI 10.1016/0022-0000(89)90034-2 | ✅ | Метаданные точны: prelim. STOC 1986 (Berkeley, 28–30 мая), full в JCSS Feb 1989, том 38 вып. 1 стр. 86–124. Источник fat node/path copying/node copying + partial/full persistence. DOI резолвится 302→Elsevier. |
| [2] | Okasaki (1998) «Purely Functional Data Structures», CUP, ISBN 0-521-66350-4, основано на CMU-CS-96-177 (1996) | ✅ | Подтверждено: книга 1998 CUP; CMU отчёт Sept 1996 (cs.cmu.edu/~rwh/students/okasaki.pdf). Каноничный источник ленивой амортизации/дебетов/banker's-real-time queue/implicit recursive slowdown. |
| [3] | Bagwell (2001) «Ideal Hash Trees», EPFL LAMP-REPORT-2001-001, infoscience.epfl.ch/record/64398 | ✅ | Tech report (не рецензируемый) — формулировка верна. Вводит HAMT, O(log32) bound, основа Clojure/Scala. Запись record/64398 существует (PDF также lampwww.epfl.ch/papers/idealhashtrees.pdf). |
| [4] | Bagwell & Rompf (2011) «RRB-Trees: Efficient Immutable Vectors», EPFL record/169879 | ✅ | Запись существует, авторы/год/тема верны: relaxed radix balanced, O(log n) конкат/insert/split при сохранении index/update скорости. |
| [5] | Hinze & Paterson (2006) «Finger Trees: A Simple General-purpose Data Structure», JFP 16(2):197–217, DOI 10.1017/S0956796805005769 | ✅ | DOI резолвится 302→Cambridge Core. Каноничная метадата JFP 16(2):197–217, 2006. |
| [6] | Apache CouchDB «The Power of B-trees» + append-only хранилище, docs.couchdb.org/en/stable/intro/overview.html | ✅ (косметика) | Append-only B-tree / новый корень / старое дерево валидно / crash-resistant MVCC — подтверждено офиц. Technical Overview. Косметика: заголовок «The Power of B-trees» физически живёт на guide.couchdb.org/.../btree.html, но процитированный overview-URL поддерживает заявленный факт и резолвится. Не блокирует → ISSUES #31 (P4). |
| [7] | Chu (Symas) «LMDB Technical Information», www.symas.com/lmdb | ✅ | COW B+дерево, single-writer MVCC, «write transactions do not block readers, nor do readers block writers» — дословно подтверждено докой Symas/lmdb.tech. |
| [8] | Clojure «Data Structures», clojure.org/reference/data_structures | ✅ | Персистентные map (HAMT) и vector (bit-partitioned trie) — офиц. дока. Атрибуция Хикки/Багвелл верна. |

## Итог

- Опровергнуто (❌): 0
- Неточно (⚠️): 0
- Не удалось подтвердить (🔍): 0
- Косметика (не блокирует): 1 — ref [6] заголовок «The Power of B-trees» vs процитированный overview-URL → ISSUES #31 (P4). FIXED (2026-06-09): ref [6] переименован в «Technical Overview — … (раздел «ACID Properties»)», URL docs.couchdb.org/en/stable/intro/overview.html совпадает с заголовком (ISSUES #31 FIXED).

Все 14 содержательных разделов + список литературы проверены sentence-by-sentence; вся арифметика (cons-список, HAMT, вектор) пересчитана независимо и совпадает; все 8 ссылок с корректными метаданными и резолвящимися DOI/URL; house-style цитирования совпадает с bloom-filter.mdx. CI зелёный, live-страница и обратная ссылка с lock-free страницы проверены.
