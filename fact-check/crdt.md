# Fact-check: CRDT (бесконфликтные реплицируемые типы данных)

Source: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/distributed/crdt.mdx`
Live URL: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/distributed/crdt

Last verification pass (sentence-by-sentence): 2026-06-08

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–21 | ✅ Проверено | 2026-06-08 | Оптимистическая/multi-master репликация, AP-сторона CAP, слабости LWW-по-физ-времени и ручного разрешения (сиблинги Dynamo), определение CRDT/SEC — корректны, атрибуция SEC к Shapiro et al. 2011 верна. |
| 2 | Модель системы и определения | 23–36 | ✅ Проверено | 2026-06-08 | Локальные обновления без координации, eventual-доставка (задержки/реордеринг/дубли, без потерь), каузальная история, конкурентность, определение convergence и SEC — точные. |
| 3 | Две семьи CRDT (CvRDT/CmRDT) | 38–71 | ✅ Проверено | 2026-06-08 | State-based=convergent=CvRDT (полное состояние, merge=⊔=LUB, join-semilattice, монотонность, comm/assoc/idem), op-based=commutative=CmRDT (операции, конкурентные коммутируют, надёжная каузальная доставка exactly-once). Таблица сравнения корректна. Терминология совпадает с RR-7506. |
| 4 | Теорема о сходимости (SEC) | 73–80 | ✅ Проверено | 2026-06-08 | Атрибуция Shapiro/Preguiça/Baquero/Zawirski 2011; достаточные условия CvRDT (join-semilattice + монотонность + merge=LUB) и CmRDT (конкурентные коммутируют + reliable causal delivery). Интуиция доказательства через comm/assoc/idem LUB — корректна. |
| 5 | Счётчики (G-Counter, PN-Counter) | 82–120 | ✅ Проверено | 2026-06-08 | Вектор покомпонентных счётчиков, inc только своей компоненты, value=sum, merge=поэлементный max=LUB. Арифметика пересчитана независимо (см. §A). PN-Counter = два G-Counter (P−N), пример 11−2=9 верен. |
| 6 | Множества (G-Set, 2P-Set, LWW-Element-Set, OR-Set) | 122–153 | ✅ Проверено | 2026-06-08 | G-Set (union), 2P-Set (once removed forever removed), LWW-Element-Set (bias на равных метках), OR-Set add-wins/observed-remove — все семантики совпадают с первоисточником. OR-Set: remove убирает только наблюдённые теги, конкурентный add выживает через свежий тег. Числовой пример пересчитан (см. §A). |
| 7 | Регистры (LWW-Register, MV-Register) | 155–158 | ✅ Проверено | 2026-06-08 | LWW-Register (значение,timestamp), tie-break по id реплики; MV-Register хранит все конкурентные значения (сиблинги Dynamo/Riak), конкурентность через version vectors. Cassandra cell-level LWW = LWW-Register на ячейку — корректно. |
| 8 | Версионирование причинности | 160–173 | ✅ Проверено | 2026-06-08 | Vector clocks/version vectors частичный порядок u≤v ⇔ ∀i u[i]≤v[i], u∥v ⇔ ¬(u≤v)∧¬(v≤u) — корректно. DVV в Riak (Preguiça et al. 2010), избегают sibling explosion при клиент-серверном доступе — подтверждено офиц. докой Riak. |
| 9 | Sequence CRDT | 175–184 | ✅ Проверено | 2026-06-08 | WOOT (Oster 2006), Treedoc (Preguiça 2009), Logoot (Weiss 2009), RGA (Roh 2011), Yjs/YATA (Nicolaescu 2016), Automerge JSON (Kleppmann 2017) — все атрибуции, годы и описания подтверждены. |
| 10 | Delta-state CRDT | 186–190 | ✅ Проверено | 2026-06-08 | Атрибуция Almeida/Shoker/Baquero 2018 (JPDC 111:162–173, arXiv:1603.01529); δ-мутаторы — элементы той же полурешётки, сливаются тем же LUB, сохраняют SEC; трафик ∝ числу обновлений — корректно. |
| 11 | Tombstones и сборка мусора | 192–198 | ✅ Проверено | 2026-06-08 | Надгробия нужны для корректного слияния удалений (иначе воскрешение), метаданные растут, GC через каузальную стабильность (отбросить, когда все реплики наблюдали), требует координации — корректно. |
| 12 | Реальные системы | 200–213 | ✅ Проверено | 2026-06-08 | Riak DT (counters/sets/maps/flags/registers + DVV), Redis Enterprise Active-Active (CRDB, CRDT-based), Cosmos DB (LWW по полю / custom merge stored procedure, НЕ библиотека CRDT), Automerge/Yjs, Akka Distributed Data (CvRDT поверх gossip), Cassandra (cell-level LWW, вырожденный случай) — все сверены с офиц. документацией. Дисклеймер «не приписывать CRDT, которого нет» корректен. |
| 13 | Когда CRDT НЕ подходят | 215–233 | ✅ Проверено | 2026-06-08 | Глобальные инварианты (неотрицательный баланс, уникальность, жёсткий лимит) требуют консенсуса/escrow; CRDT=AP, консенсус=CP; таблица сравнения и комбинированный подход — корректны. |
| 14 | Визуализация (ECharts) | 235–292 | ✅ Проверено | 2026-06-08 | Линейная модель наивный=10·раунд·..., delta=10·обновл.; данные внутренне согласованы (наивный 100→10000, delta 100→1000, ×10 разрыв). Явно помечено «иллюстративная модель, не измеренный бенчмарк» дважды. Канвас 748×380, непустой. |
| 15 | Список литературы | 294–326 | ⚠️ см. §B | 2026-06-08 | 16 источников; метаданные (год/венью/тома/страницы/DOI) сверены, корректны. URL/DOI резолвятся (ACM/IEEE/Springer DOI отдают 403/418 publisher anti-bot, но сам DOI резолвится 302→издатель — не битые ссылки). Единственное замечание: ref [3] (Preguiça 2018 survey) определён, но НЕ процитирован inline (см. ISSUES #30, P4, не блокирует). |

## §A. Независимая перепроверка арифметики

**G-Counter** (вектор `[A,B,C]`):
- A inc×2 → `[2,0,0]`=2 ✔
- B inc×5 → `[0,5,0]`=5 ✔
- C inc×1 → `[0,0,1]`=1 ✔
- merge([2,0,0],[0,5,0]) = [max(2,0),max(0,5),max(0,0)] = `[2,5,0]` = 7 ✔
- A inc×3 ещё → `[5,5,0]` = 10 ✔ (компонента A: 2+3=5)
- merge([5,5,0],[0,0,1]) = `[5,5,1]` = 11 ✔
- суммарно инкрементов 2+5+3+1 = 11 ✔ — совпадает, потерь нет, идемпотентность держит.

**PN-Counter**: P=[5,5,1] sum=11, N=[0,2,0] sum=2 → 11−2 = **9** ✔

**OR-Set add-wins**:
- A add(x)→a1: added={(x,a1)}, removed={} ✔
- B add(x)→b1 (конкурентно): added={(x,b1)} ✔
- B remove(x): убирает только наблюдённые теги {b1}; removed={b1}; a1 НЕ наблюдён B ✔
- merge: added={a1,b1}, removed={b1}; живые = {a1,b1}−{b1} = **{a1}** ✔
- {a1}≠∅ ⇒ member(x)=true, add побеждает ✔

Все три примера совпадают с текстом до цифры.

## §B. Список литературы — поштучная проверка

| # | Источник | Вердикт | Проверка |
|---|---|---|---|
| 1 | Shapiro, Preguiça, Baquero, Zawirski 2011, RR-7506 INRIA | ✅ | hal.inria.fr/inria-00555588 — 200; авторы/год/тип отчёта/CvRDT/CmRDT/SEC подтверждены. |
| 2 | Shapiro et al. 2011, SSS 2011, LNCS 6976, 386–400 | ✅ | doi 10.1007/978-3-642-24550-3_29 резолвится (Springer). |
| 3 | Preguiça 2018, arXiv:1806.10254 | ✅ метаданные + inline | arxiv.org/abs/1806.10254 — 200. FIXED (2026-06-09): inline `[[3]]` добавлен в `## Две семьи CRDT`, резолвится к `#ref-preguica-survey` (ISSUES #30 FIXED). |
| 4 | Almeida, Shoker, Baquero 2018, JPDC 111:162–173 (arXiv:1603.01529) | ✅ | doi 10.1016/j.jpdc.2017.08.003 — 200; том/страницы верны. |
| 5 | Preguiça, Baquero, Almeida, Fonte, Gonçalves 2010, DVV, arXiv:1011.5808 | ✅ | arxiv.org/abs/1011.5808 — 200; применение в Riak подтверждено. |
| 6 | Roh, Jeon, Kim, Lee 2011, RGA, JPDC 71(3):354–368 | ✅ | doi 10.1016/j.jpdc.2010.12.006 — 200. |
| 7 | Oster, Urso, Molli, Imine 2006, WOOT, CSCW 2006, 259–268 | ✅ | doi 10.1145/1180875.1180916 резолвится (ACM 403 anti-bot, не битая). |
| 8 | Weiss, Urso, Molli 2009, Logoot, ICDCS 2009, 404–412 | ✅ | doi 10.1109/ICDCS.2009.75 резолвится (IEEE 418 anti-bot). |
| 9 | Preguiça, Marquès, Shapiro, Letia 2009, Treedoc, ICDCS 2009, 395–403 | ✅ | doi 10.1109/ICDCS.2009.20 резолвится (IEEE 418 anti-bot). |
| 10 | Nicolaescu, Jahns, Derntl, Klamma 2016, YATA, GROUP 2016, 39–49 | ✅ | doi 10.1145/2957276.2957310 резолвится (ACM 403 anti-bot); основа Yjs. |
| 11 | Kleppmann, Beresford 2017, JSON CRDT, IEEE TPDS 28(10):2733–2746 | ✅ | doi 10.1109/TPDS.2017.2697382 резолвится (IEEE 418 anti-bot); основа Automerge. |
| 12 | Riak Data Types (CRDTs), офиц. дока | ✅ | docs.riak.com/.../developing/data-types/index.html — 200. |
| 13 | Redis Active-Active (CRDTs), офиц. дока | ✅ | redis.io/docs/latest/operate/rs/databases/active-active/ — 200. |
| 14 | Akka Distributed Data, офиц. дока | ✅ | doc.akka.io/.../typed/distributed-data.html — 200. |
| 15 | Microsoft, Cosmos DB conflict resolution policies | ✅ | learn.microsoft.com/azure/cosmos-db/conflict-resolution-policies — 200; LWW/custom merge SP подтверждены. |
| 16 | Vogels 2009, Eventually Consistent, CACM 52(1):40–44 | ✅ | doi 10.1145/1435417.1435432 резолвится (ACM 403 anti-bot). |

## Live-страница

Preflight + проверка 2026-06-08:
- Live URL загружается; H1 «CRDT (бесконфликтные реплицируемые типы данных)», заголовок вкладки корректен.
- `gh run list` (deploy.yml): run 27109927082 — `success`, commit «feat(distributed): add CRDT page, fix crdt slug».
- 15 H2-разделов присутствуют (соответствие плану).
- ECharts-чарт рендерится: canvas 748×380, непустой (getImageData).
- Раздел «Список литературы» присутствует; 16 ref-якорей, 15 уникальных inline `[[n]]`-ссылок, 0 висячих ref-ссылок, 0 висячих секционных якорей (Cyrillic anchors резолвятся после decodeURIComponent).
- Старый путь `/docs/distributed/cdrt` → 404 (удалён через git mv); новый `/docs/distributed/crdt` → 200.

## Резюме

0 опровергнутых фактов, 0 неточностей по существу, 0 неподтверждённых утверждений. 1 косметическое замечание: ref [3] определён, но не процитирован inline (P4, ISSUES #30, не блокирует). Все рискованные атрибуции (SEC/CvRDT/CmRDT → Shapiro et al. 2011; join-semilattice/LUB; OR-Set add-wins observed-remove; реальные системы Riak/Redis/Cosmos/Automerge-Yjs/Akka/Cassandra; delta-CRDT → Almeida/Shoker/Baquero) и вся арифметика (G-Counter, PN-Counter, OR-Set) подтверждены независимо.
