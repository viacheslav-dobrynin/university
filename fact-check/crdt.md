# Fact-check: CRDT (бесконфликтные реплицируемые типы данных)

Source: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/distributed/crdt.mdx`
Live URL: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/distributed/crdt

Last verification pass (sentence-by-sentence): 2026-06-12

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–21 | ✅ Проверено | 2026-06-12 | Оптимистическая/multi-master репликация, AP-сторона CAP, слабости LWW-по-физ-времени и ручного разрешения (сиблинги Dynamo), определение CRDT/SEC — корректны; атрибуция SEC к Shapiro et al. 2011 верна. |
| 2 | Модель системы и определения | 23–36 | ✅ Проверено | 2026-06-12 | Локальные обновления без координации, eventual-доставка (задержки/реордеринг/дубли, без потерь), каузальная история, конкурентность, определение convergence и SEC — точные. |
| 3 | Две семьи CRDT (CvRDT/CmRDT) | 38–71 | ✅ Проверено | 2026-06-12 | State-based=convergent=CvRDT (полное состояние, merge=⊔=LUB, join-semilattice, монотонность, comm/assoc/idem), op-based=commutative=CmRDT (операции, конкурентные коммутируют, надёжная каузальная доставка exactly-once); эквивалентность по выразительности; таблица сравнения корректна. Терминология совпадает с RR-7506 (подтверждено независимо). |
| 4 | Теорема о сходимости (SEC) | 73–80 | ✅ Проверено | 2026-06-12 | Атрибуция Shapiro/Preguiça/Baquero/Zawirski 2011; достаточные условия CvRDT (join-semilattice + монотонность + merge=LUB) и CmRDT (конкурентные коммутируют + reliable causal delivery). Интуиция доказательства через comm/assoc/idem LUB — корректна. |
| 5 | Счётчики (G-Counter, PN-Counter) | 82–120 | ✅ Проверено | 2026-06-12 | Вектор покомпонентных счётчиков, inc только своей компоненты, value=sum, merge=поэлементный max=LUB. Арифметика пересчитана независимо python3 (см. §A): 11 и 9 — точно. PN=два G-Counter (P−N). |
| 6 | Множества (G-Set, 2P-Set, LWW-Element-Set, OR-Set) | 122–153 | ✅ Проверено | 2026-06-12 | G-Set (union), 2P-Set (once removed forever removed), LWW-Element-Set (bias на равных метках), OR-Set add-wins/observed-remove. OR-Set: remove убирает только наблюдённые теги, конкурентный add выживает через свежий тег. Числовой пример пересчитан python3 → {a1} (см. §A). |
| 7 | Регистры (LWW-Register, MV-Register) | 155–158 | ✅ Проверено | 2026-06-12 | LWW-Register (значение,timestamp), tie-break по id реплики; MV-Register хранит все конкурентные значения (сиблинги Dynamo/Riak), конкурентность через version vectors. Cassandra cell-level LWW = LWW-Register на ячейку — подтверждено. |
| 8 | Версионирование причинности | 160–173 | ✅ Проверено | 2026-06-12 | Vector clocks/version vectors частичный порядок u≤v ⇔ ∀i u[i]≤v[i], u∥v ⇔ ¬(u≤v)∧¬(v≤u) — корректно. DVV (Preguiça et al. 2010, arXiv:1011.5808), применяются в Riak, избегают sibling explosion при клиент-серверном доступе — подтверждено. |
| 9 | Sequence CRDT | 175–184 | ✅ Проверено | 2026-06-12 | WOOT (Oster 2006), Treedoc (Preguiça 2009), Logoot (Weiss 2009), RGA (Roh 2011), Yjs/YATA (Nicolaescu 2016), Automerge JSON (Kleppmann 2017) — все атрибуции, годы и описания подтверждены первоисточниками/dblp. |
| 10 | Delta-state CRDT | 186–190 | ✅ Проверено | 2026-06-12 | Атрибуция Almeida/Shoker/Baquero 2018 (JPDC 111:162–173, arXiv:1603.01529); δ-мутаторы — элементы той же полурешётки, сливаются тем же LUB, сохраняют SEC; трафик ∝ числу обновлений. Подтверждено по сырому PDF (см. §C). |
| 11 | Tombstones и сборка мусора | 192–198 | ✅ Проверено | 2026-06-12 | Надгробия нужны для корректного слияния удалений (иначе воскрешение), метаданные растут, GC через каузальную стабильность (отбросить, когда все реплики наблюдали), требует координации — корректно. |
| 12 | Реальные системы | 200–213 | ✅ Проверено | 2026-06-12 | Riak DT (counters/sets/maps/flags/registers + DVV — офиц. дока: «five data types: flags, registers, counters, sets, maps»), Redis Enterprise Active-Active/CRDB (CRDT-based, SEC — офиц. дока), Cosmos DB (LWW по полю default / custom merge stored procedure — офиц. дока, НЕ библиотека CRDT), Automerge/Yjs, Akka Distributed Data (CvRDT поверх gossip — офиц. дока), Cassandra (cell-level LWW, вырожденный случай). Дисклеймер «не приписывать CRDT, которого нет» корректен. |
| 13 | Когда CRDT НЕ подходят | 215–233 | ✅ Проверено | 2026-06-12 | Глобальные инварианты (неотрицательный баланс, уникальность, жёсткий лимит) требуют консенсуса/escrow; CRDT=AP, консенсус=CP; таблица сравнения и комбинированный подход — корректны. |
| 14 | Визуализация (ECharts) | 235–292 | ✅ Проверено | 2026-06-12 | Линейная модель наивный=10·раунд (100→10000), delta=10·обновл. (100→1000), ×10 разрыв; данные внутренне согласованы. Явно помечено «иллюстративная модель, не измеренный бенчмарк» дважды. |
| 15 | Список литературы | 294–326 | ✅ Проверено | 2026-06-12 | 16 источников; авторы/год/венью/тома/страницы/DOI сверены независимо — корректны. URL/DOI резолвятся (ACM/IEEE/Springer DOI отдают 403/418 publisher anti-bot, HAL отдаёт Anubis-challenge — это не битые ссылки, метаданные подтверждены через dblp/Semantic Scholar/Springer/ScienceDirect). ref [3] (Preguiça 2018 survey) процитирован inline в §«Две семьи CRDT» (ISSUES #30 FIXED). |

## §A. Независимая перепроверка арифметики (python3, 2026-06-12)

Запущен скрипт на python3, результаты сверены с текстом:

**G-Counter** (вектор `[A,B,C]`):
- A inc×2 → `[2,0,0]`=2 ✔
- B inc×5 → `[0,5,0]`=5 ✔
- C inc×1 → `[0,0,1]`=1 ✔
- merge([2,0,0],[0,5,0]) = `[2,5,0]` = 7 ✔
- A inc×3 ещё → `[5,5,0]` = 10 ✔
- merge([5,5,0],[0,0,1]) = `[5,5,1]` = 11 ✔
- суммарно инкрементов 2+5+3+1 = 11 ✔ — совпадает, потерь нет.

**PN-Counter**: P=[5,5,1] sum=11, N=[0,2,0] sum=2 → 11−2 = **9** ✔

**OR-Set add-wins**: added={a1,b1}, removed={b1}; живые = {a1,b1}−{b1} = **{a1}** ⇒ member(x)=true ✔

Все три примера совпадают с текстом до цифры.

## §B. Список литературы — поштучная проверка (2026-06-12)

| # | Источник | Вердикт | Проверка |
|---|---|---|---|
| 1 | Shapiro, Preguiça, Baquero, Zawirski 2011, RR-7506 INRIA | ✅ | inria.hal.science/inria-00555588 (метаданные через dblp/SemanticScholar; HAL PDF за Anubis, не битая); CvRDT/CmRDT/SEC формализм подтверждён. |
| 2 | Shapiro et al. 2011, SSS 2011, LNCS 6976, 386–400 | ✅ | doi 10.1007/978-3-642-24550-3_29; dblp ShapiroPBZ11 подтверждает том/страницы/венью. |
| 3 | Preguiça 2018, arXiv:1806.10254 | ✅ | arxiv.org/abs/1806.10254; inline `[[3]]` присутствует в §«Две семьи CRDT». |
| 4 | Almeida, Shoker, Baquero 2018, JPDC 111:162–173 (arXiv:1603.01529) | ✅ | сырой PDF: «J. Parallel Distrib. Comput. 111 (2018) 162–173», авторы/keywords/abstract подтверждают δ-мутаторы и LUB-слияние. |
| 5 | Preguiça, Baquero, Almeida, Fonte, Gonçalves 2010, DVV, arXiv:1011.5808 | ✅ | arxiv.org/abs/1011.5808 (опубл. 26.11.2010); применение в Riak (sibling explosion) подтверждено. |
| 6 | Roh, Jeon, Kim, Lee 2011, RGA, JPDC 71(3):354–368 | ✅ | doi 10.1016/j.jpdc.2010.12.006; dblp RohJKL11 — том 71(3), 354–368, 2011. |
| 7 | Oster, Urso, Molli, Imine 2006, WOOT, CSCW 2006, 259–268 | ✅ | doi 10.1145/1180875.1180916; dblp OsterUMI06 — CSCW 2006, 259–268. |
| 8 | Weiss, Urso, Molli 2009, Logoot, ICDCS 2009, 404–412 | ✅ | doi 10.1109/ICDCS.2009.75; ICDCS 2009 Montreal подтверждён. |
| 9 | Preguiça, Marquès, Shapiro, Letia 2009, Treedoc, ICDCS 2009, 395–403 | ✅ | doi 10.1109/ICDCS.2009.20; HAL inria-00445975 — авторы/венью подтверждены. |
| 10 | Nicolaescu, Jahns, Derntl, Klamma 2016, YATA, GROUP 2016, 39–49 | ✅ | doi 10.1145/2957276.2957310; dblp NicolaescuJDK16 — GROUP 2016, 39–49; основа Yjs. |
| 11 | Kleppmann, Beresford 2017, JSON CRDT, IEEE TPDS 28(10):2733–2746 | ✅ | doi 10.1109/TPDS.2017.2697382; том 28(10):2733–2746, 2017; основа Automerge. |
| 12 | Riak Data Types (CRDTs), офиц. дока | ✅ | docs.riak.com/.../developing/data-types/; «five data types: flags, registers, counters, sets, maps». |
| 13 | Redis Active-Active (CRDTs), офиц. дока | ✅ | redis.io/docs/latest/operate/rs/databases/active-active/; CRDB = CRDT-based, SEC подтверждены. |
| 14 | Akka Distributed Data, офиц. дока | ✅ | doc.akka.io/.../typed/distributed-data.html; CvRDT (GCounter/PNCounter/GSet/ORSet/registers/maps) поверх gossip. |
| 15 | Microsoft, Cosmos DB conflict resolution policies | ✅ | learn.microsoft.com/azure/cosmos-db/conflict-resolution-policies; LWW (default, _ts) / custom merge SP подтверждены. |
| 16 | Vogels 2009, Eventually Consistent, CACM 52(1):40–44 | ✅ | doi 10.1145/1435417.1435432; CACM Jan 2009, vol 52 no 1, 40–44. |

## §C. Сырой первоисточник (rigor: «fetch the raw source»)

Delta-CRDT PDF (members.loria.fr mirror) извлечён через pdftotext: подтверждены авторы (Almeida, Shoker, Baquero, Univ. Minho), венью «J. Parallel Distrib. Comput. 111 (2018) 162–173», даты (received 2016 / accepted 2017), и δ-мутаторы, возвращающие delta-state, сливаемый с локальным и удалённым состоянием тем же join — полностью соответствует тексту страницы. Формализм CvRDT/CmRDT (join-semilattice, merge=LUB comm/assoc/idem, монотонные update; op-based: конкурентные коммутируют + reliable/exactly-once causal delivery; эквивалентность; SEC) независимо сверен с авторитетным источником — расхождений с текстом нет.

## Live-страница

Preflight + проверка 2026-06-12:
- Live homepage загружается; CRDT live URL загружается, H1/title «CRDT (бесконфликтные реплицируемые типы данных)» корректны.
- `gh run list --workflow=deploy.yml` (submodule repo): run 27347984710 — `success` (2026-06-11).
- Все разделы присутствуют; ECharts-чарт присутствует в исходнике; «Список литературы» — 16 ref-якорей.

## Резюме

0 опровергнутых фактов, 0 неточностей, 0 неподтверждённых утверждений. Все рискованные атрибуции (SEC/CvRDT/CmRDT → Shapiro et al. 2011 RR-7506 + SSS 2011; join-semilattice/LUB comm/assoc/idem; OR-Set add-wins observed-remove; реальные системы Riak/Redis/Cosmos/Akka/Automerge-Yjs/Cassandra; delta-CRDT → Almeida/Shoker/Baquero; sequence-CRDT WOOT/Treedoc/Logoot/RGA/YATA/JSON-CRDT) и вся арифметика (G-Counter=11, PN-Counter=9, OR-Set={a1}) подтверждены независимо. Все 16 ссылок резолвятся (издательский anti-bot ≠ битая ссылка), метаданные верны. Вердикт: APPROVED.
