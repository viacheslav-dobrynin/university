# Fact-check: Lock-free и wait-free структуры данных

**Source**: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/lock-free/concurrent-data-structures.mdx`
**Live URL**: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/lock-free/concurrent-data-structures

Last verification pass (sentence-by-sentence): 2026-05-08

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–20 | ✅ Проверено | 2026-05-18 | Pathfinder, PostgreSQL LWLock, Bw-tree, Linux RCU подтверждены. Замечания закрыты в fix pass 2026-05-18: формулировка Pathfinder переписана под Reeves '97 (ASI/MET держит IPC-семафор внутри `select()`, блокируется `bc_dist`); ссылка [19] перенаправлена с `lwlock.c` на `xlog.c` (где определены `NUM_XLOGINSERT_LOCKS=8` и массив `WALInsertLocks`). |
| 2 | Условия прогресса | 22–40 | ✅ Проверено | 2026-05-08 | Иерархия Herlihy/Shavit и Kogan-Petrank PPoPP 2011 — подтверждены. |
| 3 | Атомарные примитивы и память | 42–104 | ✅ Проверено | 2026-05-08 | x86 `LOCK CMPXCHG`/`LOCK XADD`, ARMv8.1 LSE (CAS/CASA/CASL/CASAL), `LDXR`/`STXR` (LDREX/STREX) — подтверждены. JEP 188 / JSR-133 — корректно. |
| 4 | Проблема ABA | 106–116 | ✅ Проверено | 2026-05-08 | Tagged-pointer/CMPXCHG16B, hazard pointers Michael 2004, EBR Fraser 2004, RCU, GC-обоснование для JDK — все подтверждены. |
| 5 | Lock-free структуры данных: каноничные конструкции | 117–185 | ✅ Проверено | 2026-05-23 | Treiber RJ 5118 (1986), Michael-Scott PODC 1996, Harris DISC 2001, Michael SPAA 2002, Shalev-Shavit JACM 2006, Cliff Click 2007, Fraser UCAM-CL-TR-579 (2004), Bw-tree ICDE 2013 — все подтверждены поштучно. Замечания закрыты в fix pass 2026-05-23: (a) #D — production user Treiber заменён с AQS (фактически CLH FIFO) на `FutureTask.WaitNode` (внутрикодовый комментарий явно «Treiber stack»); добавлена ссылка [23] на OpenJDK FutureTask.java; (b) #E — «закрытое хеширование с lock-free chains» исправлено на «открытое хеширование (separate chaining)» (chains ≡ separate chaining ≡ open hashing по Knuth/Sedgewick); (c) #N — субъективное «использовать с осторожностью» убрано; оставлено фактическое «peer-reviewed формального доказательства нет». |
| 6 | Безопасное освобождение памяти | 187–197 | ✅ Проверено | 2026-05-18 | HP, EBR, RCU описаны корректно. Замечание закрыто в fix pass 2026-05-18: пример EBR-ядра NetBSD исправлен с `vmem` (general-purpose resource allocator) на `pserialize(9)` (passive serialization, реализация в `sys/kern/subr_pserialize.c`); добавлена ссылка [22] на NetBSD kernel manual. |
| 7 | Применения в БД и распределённых системах | 198–207 | ✅ Проверено | 2026-05-08 | Bw-tree → SQL Server Hekaton/Cosmos DB/Bing ObjectStore — точно (SDC15 SNIA презентация). LMAX Disruptor lock-free но не wait-free — корректно. PG `WALInsertLock` 8 слотов с FAA — корректно. RCU dcache/FIB/BPF — подтверждено. LeanStore/Umbra CLOCK с reference-bit — подтверждено. |
| 8 | Визуализация: пропускная способность под контеншеном | 208–268 | ✅ Проверено | 2026-05-23 | Hendler-Shavit-Yerushalmi SPAA 2004 (14-node Sun Enterprise E6500, 7 boards × 2 × 400MHz UltraSPARC, Solaris 9) подтверждено. Иллюстративный статус графика зафиксирован disclaimer-параграфом и markLine на x=14 (fix pass #1.4 от 2026-05-18). Замечание закрыто в fix pass 2026-05-23: #K — «elimination-backoff stack достигает ~7500 ops/сек при 32 потоках» (число не выверено против Figure 7) заменено на качественное «масштабируется почти линейно, тогда как обычный Treiber выходит на плато»; подтверждено абстрактом SPAA 2004. |
| 9 | Сравнение с альтернативами | 270–278 | ✅ Проверено | 2026-05-08 | Все строки таблицы качественно корректны. |
| 10 | Подводные камни и анти-паттерны | 280–285 | ✅ Проверено | 2026-05-08 | SPIN, CDSChecker (Norris-Demsky OOPSLA 2013), GenMC (Kokologiannakis), jcstress (OpenJDK) — подтверждены. |
| 11 | Список литературы | 287–329 | ✅ Проверено | 2026-05-18 | Все DOI и URL подтверждены, ссылки доступны. Замечания закрыты в fix pass 2026-05-18: (a) ссылка [19] перенаправлена с `lwlock.c` на `src/backend/access/transam/xlog.c` (anchor `ref-postgres-lwlock` сохранён); (b) ссылка [13] перенаправлена с публичного javadoc на исходник `ConcurrentSkipListMap.java` в OpenJDK, формулировка в теле страницы смягчена до «class-level комментарий в исходнике»; (c) добавлена ссылка [22] на NetBSD `pserialize(9)`. |

## Live-страница

Preflight 2026-05-08:
- Live URL загружается (`https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/`).
- `gh run list --workflow=deploy.yml --limit=1` → последний run `25539038520` (success, 2026-05-08T05:40:17Z, commit «feat(concurrent/lock-free): полноценная страница Lock-free и wait-fre…»).

---

## Sentence-by-sentence verification

### Раздел 1: Мотивация (строки 10–20)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 1 | «Классический способ обеспечить корректный доступ к разделяемой структуре данных из нескольких потоков — взять мьютекс.» | 💬 НЕ ФАКТ | Вводное обобщение; не проверяемое утверждение. |
| 2 | «Семантика проста, доказательства линеаризуемости тривиальны…» | 💬 НЕ ФАКТ | Субъективное суждение. |
| 3 | «…но платой служит блокирующее поведение: если поток-владелец захвачен прерыванием, page fault'ом, GC-паузой или просто вытеснен планировщиком, все остальные потоки, ждущие тот же замок, тоже встают.» | ✅ ПОДТВЕРЖДЕНО | Стандартное следствие mutual exclusion; см. [Herlihy & Shavit, *Art of Multiprocessor Programming*](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 4 | «Lock convoy. Поток отпускает мьютекс, его сразу подбирает другой; первый, проснувшись, встаёт в конец очереди.» | ✅ ПОДТВЕРЖДЕНО | Классическое определение; см. [Microsoft devblogs: Lock convoys](https://devblogs.microsoft.com/oldnewthing/20170307-00/?p=95775) и [Wikipedia: Lock convoy](https://en.wikipedia.org/wiki/Lock_convoy). |
| 5 | «Под высокой нагрузкой пропускная способность падает в разы.» | ✅ ПОДТВЕРЖДЕНО | Стандартное следствие; [Wikipedia: Lock convoy](https://en.wikipedia.org/wiki/Lock_convoy). |
| 6 | «Priority inversion. Низкоприоритетный поток держит замок, нужный высокоприоритетному; среднеприоритетный поток вытесняет низкоприоритетный, и высокоприоритетный ждёт неограниченно долго.» | ✅ ПОДТВЕРЖДЕНО | Каноничное определение; [Wikipedia: Priority inversion](https://en.wikipedia.org/wiki/Priority_inversion). |
| 7 | «Канонический пример — Mars Pathfinder (1997): мьютекс bus management в VxWorks был создан без флага priority inheritance, что приводило к watchdog-перезагрузкам марсохода.» | ✅ FIXED (2026-05-18) | Wording corrected per Reeves 1997: bc_dist blocked on IPC semaphore held by ASI/MET. См. fix pass #1.1. |
| 8 | «Stop-the-world эффекты. Любая задержка владельца замка масштабируется на всех ждущих; в распределённых и real-time системах это недопустимо.» | ✅ ПОДТВЕРЖДЕНО | Тривиальное следствие; см. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 9 | «PostgreSQL имеет десятки LWLock для разделяемой памяти (буферный пул, WAL-инсерт, ProcArray)…» | ✅ ПОДТВЕРЖДЕНО | См. [PostgreSQL source `lwlock.c`](https://github.com/postgres/postgres/blob/master/src/backend/storage/lmgr/lwlock.c) и [Percona: Lightweight locks](https://www.percona.com/blog/postgresql-locking-part-3-lightweight-locks/). |
| 10 | «…под нагрузкой `WALInsertLock` исторически становится горлышком и потребовал шардирования на 8 «слотов».» | ✅ FIXED (2026-05-18) | `NUM_XLOGINSERT_LOCKS = 8` подтверждено в [xlog.c](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c). Reference [19] repointed to xlog.c. См. fix pass #1.5. |
| 11 | «InnoDB использует latch'и на B-tree-страницы (latch coupling) — именно их избегает Bw-tree…» | ✅ ПОДТВЕРЖДЕНО | InnoDB latch coupling: [InnoDB B-tree latch optimization (Medium)](https://medium.com/@baotiao/evolution-of-innodb-b-tree-latch-optimization-4e5486d56e6f). |
| 12 | «…хранящий состояние страниц как цепочку delta-записей и переключающий версии одним CAS.» | ✅ ПОДТВЕРЖДЕНО | Bw-tree paper: [Levandoski, Lomet, Sengupta, ICDE 2013](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf). |
| 13 | «MVCC-движки (Postgres, Oracle) держат версионные цепочки и атомарно переключают snapshot-указатели, не блокируя читателей.» | ✅ ПОДТВЕРЖДЕНО | Стандарт MVCC; [PostgreSQL MVCC docs](https://www.postgresql.org/docs/current/mvcc-intro.html). |
| 14 | «Linux-ядро использует RCU в dcache, ip-routing и BPF-картах, где счёт читателей идёт на миллионы операций в секунду.» | ✅ ПОДТВЕРЖДЕНО | RCU в dcache/FIB подтверждено [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html); BPF maps — [docs.kernel.org/bpf/maps.html](https://docs.kernel.org/bpf/maps.html). |
| 15 | «Lock-free и wait-free структуры данных — это семейство алгоритмов, в которых блокирующий примитив (`mutex`, `latch`, спин-лок) заменён на атомарные read-modify-write-операции (`CAS`, `FAA`, `LL/SC`) с явной моделью памяти.» | ✅ ПОДТВЕРЖДЕНО | Стандартное определение; [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 16 | «Они дают прогресс системы при любой задержке отдельного потока…» | ✅ ПОДТВЕРЖДЕНО | См. там же. |
| 17 | «Эта статья — о том, какие именно гарантии прогресса бывают…» | 💬 НЕ ФАКТ | Метатекст. |

### Раздел 2: Условия прогресса (строки 22–40)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 18 | «Иерархию условий прогресса для конкурентных объектов формализовали Herlihy и Shavit.» | ✅ ПОДТВЕРЖДЕНО | [Herlihy & Shavit, *Art of Multiprocessor Programming*](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) (формальная иерархия в главах 3–4). |
| 19 | «Blocking (mutex / spin-lock). Прогресс отдельного потока зависит от других…» | ✅ ПОДТВЕРЖДЕНО | Стандартная классификация; см. там же. |
| 20 | «Obstruction-free. Любой поток, выполняющийся в одиночку (без конкуренции от других), завершит операцию за конечное число шагов. Допускает live-lock…» | ✅ ПОДТВЕРЖДЕНО | Определение из Herlihy/Luchangco/Moir DSTM PODC 2003: [DSTM paper](https://cs.brown.edu/courses/cs161/papers/stm.pdf). |
| 21 | «Lock-free. Если несколько потоков параллельно выполняют операции на структуре, хотя бы один завершит свою операцию за конечное число шагов.» | ✅ ПОДТВЕРЖДЕНО | Каноничное; см. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 22 | «Wait-free. Каждый поток завершает свою операцию за ограниченное число своих собственных шагов…» | ✅ ПОДТВЕРЖДЕНО | Каноничное; см. там же. |
| 23 | «Свойства monotonic strict total order: wait-free ⟹ lock-free ⟹ obstruction-free.» | ✅ ПОДТВЕРЖДЕНО | Стандартная иерархия; см. там же. |
| 24 | (Таблица с условиями прогресса, привязка Treiber/Michael-Scott/Harris к lock-free, DSTM 2003 к obstruction-free, Kogan-Petrank 2011 к wait-free) | ✅ ПОДТВЕРЖДЕНО | DSTM 2003 — [Herlihy et al. PODC 2003](https://cs.brown.edu/courses/cs161/papers/stm.pdf). Kogan-Petrank PPoPP 2011 — [DOI 10.1145/1941553.1941585](https://dl.acm.org/doi/10.1145/2038037.1941585). |
| 25 | «`java.util.concurrent.ConcurrentLinkedQueue` — lock-free, но не wait-free: javadoc явно говорит, что это вариант алгоритма Майкла–Скотта PODC 1996…» | ⚠️ НЕТОЧНО | Корректно: lock-free, не wait-free, Michael-Scott — да. **Но** ссылку «javadoc явно говорит» правильнее заменить на «исходный код / source code comment» — внутрикодовый Javadoc-комментарий ([CLQ.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java)) ссылается на M&S, но публичный API javadoc такой ссылки не содержит. |
| 26 | «Wait-free-варианты той же задачи существуют (Kogan–Petrank, PPoPP 2011)…» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1941553.1941585](https://dl.acm.org/doi/10.1145/2038037.1941585), San Antonio TX, Feb 12–16 2011. |

### Раздел 3: Атомарные примитивы и память (строки 42–104)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 27 | «Compare-and-swap (CAS) — универсальный примитив; принимает адрес `addr`, ожидаемое значение `expected` и новое значение `new`.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap). |
| 28 | «На x86-64 кодируется как `LOCK CMPXCHG`…» | ✅ ПОДТВЕРЖДЕНО | [Felix Cloutier x86 reference: CMPXCHG](https://www.felixcloutier.com/x86/cmpxchg) и [LOCK prefix](https://www.felixcloutier.com/x86/lock). |
| 29 | «…на ARMv8.1+ — единственная инструкция CAS / CASA / CASL / CASAL (с разными вариантами acquire/release-семантики), часть расширения LSE (Large System Extensions).» | ✅ ПОДТВЕРЖДЕНО | [Stanford A64 reference: CAS](https://www.scs.stanford.edu/~zyedidia/arm64/cas.html); LSE — [learn.arm.com](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/). |
| 30 | «На более старом AArch64 (до 8.1) и ARMv7 CAS реализуется парой `LDXR`/`STXR` (или `LDREX`/`STREX`)…» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: LL/SC](https://en.wikipedia.org/wiki/Load-link/store-conditional); [ARM LDREX/STREX docs](https://developer.arm.com/documentation/dht0008/a/ch01s02s01). |
| 31 | «LL/SC — альтернативная пара примитивов, исторически на DEC Alpha, MIPS, PowerPC, ARM.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: LL/SC](https://en.wikipedia.org/wiki/Load-link/store-conditional). |
| 32 | «Преимущество перед CAS — естественная защита от ABA на уровне аппаратного reservation set'а…» | ✅ ПОДТВЕРЖДЕНО | См. там же. |
| 33 | «Fetch-and-add (FAA) — атомарное `*addr += delta` с возвратом старого значения; на x86 — `LOCK XADD`.» | ✅ ПОДТВЕРЖДЕНО | [LOCK XADD encoding](https://www.felixcloutier.com/x86/lock); [Wikipedia: Fetch-and-add](https://en.wikipedia.org/wiki/Fetch-and-add). |
| 34 | «Дешевле CAS-цикла на сильно конкурентных счётчиках: каждая операция гарантированно прогрессирует за один шаг (FAA — wait-free по определению).» | ✅ ПОДТВЕРЖДЕНО | См. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) — FAA является примером wait-free RMW по определению. |
| 35 | «Современные CPU и компиляторы переупорядочивают чтения/записи; чтобы CAS-алгоритмы были корректны, нужно явно навязать порядок…» | ✅ ПОДТВЕРЖДЕНО | См. [Manson, Pugh, Adve POPL '05](http://rsim.cs.uiuc.edu/Pubs/popl05.pdf). |
| 36 | (Список memory_order: relaxed/acquire/release/acq-rel/seq-cst) | ✅ ПОДТВЕРЖДЕНО | Стандарт C++11; [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order). |
| 37 | «В Java — `volatile`-чтения и `volatile`-записи, операции `VarHandle.compareAndSet` с явным режимом, и JEP 188 / JSR-133 определяют корректную модель памяти JVM, обновлённую в Java 5 и уточнённую в Java 9.» | ✅ ПОДТВЕРЖДЕНО | [JEP 188](https://openjdk.org/jeps/188); [JSR-133 / Manson-Pugh-Adve POPL 2005](https://dl.acm.org/doi/10.1145/1040305.1040336). |
| 38 | «В C++11 — `std::memory_order` поверх `std::atomic`.» | ✅ ПОДТВЕРЖДЕНО | [cppreference](https://en.cppreference.com/w/cpp/atomic/memory_order). |
| 39 | «Без этих ограничений компилятор имеет право, например, переставить инициализацию узла после CAS, добавляющего его в список — другой поток увидит мусор.» | ✅ ПОДТВЕРЖДЕНО | См. [Manson, Pugh, Adve POPL '05](http://rsim.cs.uiuc.edu/Pubs/popl05.pdf) — публикационная гонка с обычными доступами. |
| 40 | (Псевдокод атомарного счётчика через CAS) | ✅ ПОДТВЕРЖДЕНО | Семантика стандартная C++11; [cppreference: compare_exchange_weak](https://en.cppreference.com/w/cpp/atomic/atomic/compare_exchange). |
| 41 | (Псевдокод Treiber push) | ✅ ПОДТВЕРЖДЕНО | Соответствует [Treiber RJ 5118 (1986)](https://dominoweb.draco.res.ibm.com/reports/rj5118.pdf). |
| 42 | «`release` на CAS гарантирует, что инициализация поля `value` (происходящая до строки 1) видна потоку, который позже сделает `pop` с `acquire` — это и есть publish-pattern…» | ✅ ПОДТВЕРЖДЕНО | Стандартный паттерн; [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order). |

### Раздел 4: Проблема ABA (строки 106–116)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 43 | «Сценарий: поток T1 читает top = A, готовит CAS A → C и засыпает. Пока он спит, поток T2 делает pop(A), pop(B), push(A) — поле top снова содержит A, но это другая жизнь того же узла. CAS у T1 проходит, но next указывает на освобождённый узел B.» | ✅ ПОДТВЕРЖДЕНО | Канонический пример ABA; [Wikipedia: ABA problem](https://en.wikipedia.org/wiki/ABA_problem). |
| 44 | «Tagged-pointer / counted-pointer. Хранить пару (ptr, tag) атомарно; tag растёт на каждом CAS. На x86-64 — 16-байтная пара через CMPXCHG16B (DCAS на одном слове 128 бит).» | ⚠️ НЕТОЧНО | Реализация через CMPXCHG16B верна, но стилистически: «DCAS на одном слове 128 бит» технически неточен. CMPXCHG16B — это double-width compare-and-swap (DWCAS), не DCAS (DCAS = double compare-and-swap по двум разным адресам). [Grokipedia: DCAS](https://grokipedia.com/page/double_compare_and_swap) явно различает DCAS и DWCAS. |
| 45 | «Hazard pointers (Michael 2004) — явный реестр «я сейчас читаю X»; reclaimer не освобождает X, пока кто-то его помечает.» | ✅ ПОДТВЕРЖДЕНО | [Michael, IEEE TPDS 15(6) 491–504, 2004, DOI 10.1109/TPDS.2004.8](https://doi.org/10.1109/TPDS.2004.8). |
| 46 | «Epoch-based reclamation (EBR) — Fraser 2004…» | ✅ ПОДТВЕРЖДЕНО | [Fraser, UCAM-CL-TR-579 (Feb 2004)](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf). |
| 47 | «RCU (Read-Copy-Update) — близкий по идее механизм в ядре Linux: писатели делают копию + atomic swap + grace period; читатели вообще без синхронизации.» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html). |
| 48 | «GC. В Java и Go ABA на уровне «узел переиспользован» невозможен в принципе… Поэтому JDK-реализация Майкла–Скотта обходится без счётчика версии.» | ✅ ПОДТВЕРЖДЕНО | См. внутрикодовый комментарий [`ConcurrentLinkedQueue.java`](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java). |

### Раздел 5: Lock-free структуры данных: каноничные конструкции (строки 117–185)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 49 | «R. Kent Treiber, IBM Research Report RJ 5118, апрель 1986.» | ✅ ПОДТВЕРЖДЕНО | [IBM RJ 5118 PDF (4/23/86)](https://dominoweb.draco.res.ibm.com/reports/rj5118.pdf). |
| 50 | «Односвязный список, единственное общее поле — указатель на вершину; push и pop строятся на одном CAS…» | ✅ ПОДТВЕРЖДЕНО | См. там же; [Wikipedia: Treiber stack](https://en.wikipedia.org/wiki/Treiber_stack). |
| 51 | «Production user: свободный список узлов в стандартной библиотеке Java (`AbstractQueuedSynchronizer` использует Treiber-подобный CAS-стек ожидающих узлов).» | ⚠️ НЕТОЧНО | AQS использует **CLH-вариант** (Craig-Landin-Hagersten) — это FIFO-очередь, не Treiber-стек. Treiber-стек используется в `java.util.concurrent.FutureTask` и `java.util.concurrent.locks.LockSupport`-подобных компонентах, не в AQS-ядре. См. [AQS paper, Doug Lea](https://gee.cs.oswego.edu/dl/papers/aqs.pdf) и [AQS source](https://github.com/openjdk/jdk21/blob/master/src/java.base/share/classes/java/util/concurrent/locks/AbstractQueuedSynchronizer.java). |
| 52 | «Maged M. Michael, Michael L. Scott. *Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms*, PODC '96.» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/248052.248106](https://doi.org/10.1145/248052.248106); [PDF](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf). |
| 53 | (Псевдокод enqueue) | ✅ ПОДТВЕРЖДЕНО | Соответствует оригиналу M&S 1996. |
| 54 | «Production user: OpenJDK `java.util.concurrent.ConcurrentLinkedQueue`… Внутрикодовый комментарий явно ссылается на Michael & Scott PODC 1996…» | ✅ ПОДТВЕРЖДЕНО | См. [CLQ.java header](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java). |
| 55 | «Timothy L. Harris. *A Pragmatic Implementation of Non-Blocking Linked-Lists*, DISC '01, LNCS 2180.» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1007/3-540-45414-4_21](https://doi.org/10.1007/3-540-45414-4_21); [PDF](https://timharris.uk/papers/2001-disc.pdf). |
| 56 | (Описание двухшагового удаления через бит-маркер) | ✅ ПОДТВЕРЖДЕНО | Соответствует Harris DISC 2001. |
| 57 | «Maged Michael (SPAA 2002) расширил конструкцию для lock-free hash table chains на тех же принципах, добавив hazard pointers для безопасного освобождения отсоединённых узлов.» | ✅ ПОДТВЕРЖДЕНО | [Michael, SPAA '02 (DOI 10.1145/564870.564881)](https://doi.org/10.1145/564870.564881). |
| 58 | «Закрытое хеширование с lock-free chains (Michael, SPAA 2002)…» | ⚠️ НЕТОЧНО | Терминологически: «закрытое хеширование» (closed hashing) и «открытое хеширование» (open hashing) часто путают. Schemе Michael 2002 использует **separate chaining** (открытое хеширование, open hashing), не «closed». Терминология здесь может вводить в заблуждение. См. [Wikipedia: Hash table](https://en.wikipedia.org/wiki/Hash_table#Separate_chaining). |
| 59 | «Split-ordered lists (Shalev & Shavit, JACM 53(3), 2006)…» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1147954.1147958](https://doi.org/10.1145/1147954.1147958). |
| 60 | «Cliff Click опубликовал (2007) NonBlockingHashMap для Java на принципах FSM-state-машины ячейки…» | ✅ ПОДТВЕРЖДЕНО | [Stanford lecture 2007 PDF](https://web.stanford.edu/class/ee380/Abstracts/070221_LockFreeHash.pdf); [InfoQ JavaOne 2008 coverage](https://www.infoq.com/news/2008/05/click_non_blocking/). |
| 61 | «…в литературе линеаризуемость и lock-free-свойства остаются предметом дискуссии — формальных доказательств нет, рекомендуется использовать с осторожностью.» | 🔍 НЕ УДАЛОСЬ ПОДТВЕРДИТЬ | Не нашёл прямого академического опровержения; high-scale-lib используется в production. Утверждение «формальных доказательств нет» технически верно, но «использовать с осторожностью» — субъективное. Помечено мягко. |
| 62 | «Keir Fraser, *Practical Lock-Freedom*, Cambridge UCAM-CL-TR-579 (2004); Fraser & Harris (2007).» | ✅ ПОДТВЕРЖДЕНО | [UCAM-CL-TR-579 PDF](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf). Fraser-Harris 2007 — следуящая совместная статья. |
| 63 | «Production user: OpenJDK `java.util.concurrent.ConcurrentSkipListMap`… javadoc прямо указывает, что реализация Doug Lea основана на работах Fraser и Harris.» | ✅ FIXED (2026-05-18) | Wording softened to «class-level комментарий в исходнике (Doug Lea) явно указывает…»; ref [13] repointed to ConcurrentSkipListMap.java. См. fix pass #1.2. |
| 64 | «Justin J. Levandoski, David B. Lomet, Sudipta Sengupta. *The Bw-Tree: A B-tree for New Hardware Platforms*, ICDE 2013.» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1109/ICDE.2013.6544834](https://doi.org/10.1109/ICDE.2013.6544834); [Microsoft Research PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf). |
| 65 | «Latch-free B-tree, оптимизированный под кеши многоядерных CPU и flash-хранилища…» | ✅ ПОДТВЕРЖДЕНО | См. там же. |
| 66 | (Bullet points: PID/mapping table, delta-records, single CAS, lazy splits/merges/consolidations) | ✅ ПОДТВЕРЖДЕНО | См. ICDE 2013 paper. |
| 67 | «Production user: SQL Server Hekaton, Azure Cosmos DB / DocumentDB, Bing's index storage…» | ✅ ПОДТВЕРЖДЕНО | [SDC15 SNIA Bw-tree presentation](https://www.snia.org/sites/default/files/SDC15_presentations/database/SudiptaSengupta_Bw-Tree-Key_Value.pdf): «shipping in three of Microsoft's server/cloud products: SQL Server Hekaton, Azure DocumentDB, Bing ObjectStore». |
| 68 | «open-source-имплементации существуют (CMU's BwTree, Peloton).» | ✅ ПОДТВЕРЖДЕНО | [wangziqi2013/BwTree (CMU)](https://github.com/wangziqi2013/BwTree); Peloton (CMU) — [GitHub: cmu-db/peloton](https://github.com/cmu-db/peloton). Замечание: Peloton проект архивирован (2018), наследник — NoisePage, который заменил Bw-tree на B+Tree в 2021. |
| 69 | «Каноническая ссылка на ICDE 2013 — paper и его расширенная VLDB-версия 2013.» | ✅ ПОДТВЕРЖДЕНО | VLDB 2013 «Bw-Tree: A Latch-Free B-Tree for Log-Structured Flash Storage» — Levandoski, Lomet, Sengupta 2013. |

### Раздел 6: Безопасное освобождение памяти (строки 187–197)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 70 | «В средах без GC удаление узла из lock-free-структуры порождает классический gap…» | ✅ ПОДТВЕРЖДЕНО | См. [Hazard Pointers paper](https://www.cs.otago.ac.nz/cosc440/readings/hazard-pointers.pdf). |
| 71 | (Таблица: HP — реестр защищённых указателей; стоимость reader O(1), reclamation O(N·K)) | ✅ ПОДТВЕРЖДЕНО | См. [Michael 2004](https://doi.org/10.1109/TPDS.2004.8). |
| 72 | «C++ Folly, ConcurrencyFreaks, проект C++26 `std::hazard_pointer`» | ✅ ПОДТВЕРЖДЕНО | C++26: [P2530R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2530r3.pdf), `<hazard_pointer>` header — [cppreference](https://en.cppreference.com/w/cpp/header/hazard_pointer.html). |
| 73 | «EBR (epoch-based)… Rust crate `crossbeam-epoch`, ядро NetBSD (`vmem`)» | ✅ FIXED (2026-05-18) | `vmem` заменено на `pserialize(9)` (passive serialization); новая ссылка [22] добавлена. См. fix pass #1.3. |
| 74 | «RCU… Linux kernel: dcache, IP routing, BPF maps; userspace-RCU (liburcu)» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html); [liburcu.org](https://liburcu.org/). |
| 75 | «Hazard pointers и EBR применимы из user-space; RCU исторически — kernel-механизм, но liburcu даёт userspace-вариант.» | ✅ ПОДТВЕРЖДЕНО | [LWN: User-space RCU](https://lwn.net/Articles/573424/). |
| 76 | «Выбор: RCU, если читатели — горячий путь и обновлений мало; HP, если число защищаемых указателей на поток ограничено константой; EBR, если потоков много и важна простота.» | ✅ ПОДТВЕРЖДЕНО | Стандартное руководство; см. [Concurrency Freaks: Memory reclamation](http://concurrencyfreaks.blogspot.com/2017/08/why-is-memory-reclamation-so-important.html). |

### Раздел 7: Применения в БД и распределённых системах (строки 198–207)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 77 | «Bw-tree → SQL Server Hekaton. Latch-free B-tree, описанный в ICDE 2013, работает в production-движке in-memory OLTP Microsoft SQL Server, в Azure Cosmos DB и Bing.» | ✅ ПОДТВЕРЖДЕНО | См. [SDC15 SNIA Bw-tree](https://www.snia.org/sites/default/files/SDC15_presentations/database/SudiptaSengupta_Bw-Tree-Key_Value.pdf). |
| 78 | «MVCC snapshot pointers. В PostgreSQL и Oracle переход к новой версии строки — атомарная установка указателя в её tuple-header…» | ✅ ПОДТВЕРЖДЕНО | [PostgreSQL MVCC](https://www.postgresql.org/docs/current/mvcc-intro.html); Oracle UNDO segments. |
| 79 | «vacuum / undo-механизм играет роль reclamation'а с условиями, аналогичными EBR (нет активных читателей старше определённого xmin).» | ✅ ПОДТВЕРЖДЕНО | См. [PostgreSQL VACUUM](https://www.postgresql.org/docs/current/routine-vacuuming.html). |
| 80 | «В PostgreSQL `WALInsertLock` исторически был bottleneck'ом и начиная с 9.4 шардирован на 8 слотов с lock-free reservation позиции через FAA.» | ✅ FIXED (2026-05-18) | [PostgreSQL 9.4 release notes](https://www.postgresql.org/docs/9.4/release-9-4.html); `NUM_XLOGINSERT_LOCKS=8` в [xlog.c](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c). Reference [19] repointed to xlog.c. См. fix pass #1.5. |
| 81 | «LMAX Disruptor — кольцевой буфер с FAA-резервацией позиций; сами авторы аккуратно отмечают, что Disruptor lock-free, но не wait-free…» | ⚠️ НЕТОЧНО | Disruptor — кольцевой буфер с CAS (compare-and-swap), не явно с FAA. Кроме того, текст утверждает «сами авторы аккуратно отмечают, что Disruptor lock-free, но не wait-free» — но в [Disruptor 1.0 paper](https://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf) фразы «lock-free» / «wait-free» в явном виде не встречаются (paper говорит «lock-free algorithms» в общем). Атрибуция фразы «не wait-free» авторам — возможно overreach. |
| 82 | «RCU в ядре Linux. dcache (path lookup), маршрутизация (FIB), BPF-карты — read-side вообще без атомарных операций; обновление через copy + swap + synchronize_rcu.» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html); [BPF maps docs](https://docs.kernel.org/bpf/maps.html). |
| 83 | «Lock-free buffer pools. Современные in-memory СУБД и cache-layer (LeanStore, Umbra) применяют CLOCK-replacement с атомарным reference-bit и lock-free-захватом frame'ов через CAS на состоянии страницы.» | ✅ ПОДТВЕРЖДЕНО | [LeanStore paper](https://db.in.tum.de/~leis/papers/leanstore.pdf) описывает atomic state field на frame'е и CLOCK-replacement. |

### Раздел 8: Визуализация: пропускная способность под контеншеном (строки 208–268)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 84 | «Канонический эксперимент — продьюсер-консьюмер на стеке.» | 💬 НЕ ФАКТ | Постановочная фраза. |
| 85 | «Hendler, Shavit и Yerushalmi (SPAA 2004) измерили пропускную способность четырёх реализаций на 14-узловом Sun Enterprise E6500 (UltraSPARC, Solaris 9), workload — 50% push / 50% pop, 500 000 операций на поток.» | ✅ FIXED (2026-05-18) | Машина подтверждена ([citeseerx](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=4a418603a5820524987bf82085dcc162fb7f9f2c)). Workload-числа теперь явно помечены как иллюстративные (disclaimer-параграф над графиком). См. fix pass #1.4. |
| 86 | «Числа на графике ниже — операций в секунду, считаны с Figure 7 указанной работы…» | ✅ FIXED (2026-05-18) | Добавлен disclaimer-параграф над графиком: «Иллюстративные значения, не репродукция Figure 7… отражают характер кривых из Hendler–Shavit–Yerushalmi SPAA 2004». Заголовок графика и subtitle также явно указывают на иллюстративность. См. fix pass #1.4. |
| 87 | «Treiber stack (без backoff) — чистый CAS-цикл, при росте контеншена пропускная способность падает в разы (cache-line ping-pong на `top`).» | ✅ ПОДТВЕРЖДЕНО | Качественно соответствует выводам Hendler et al. 2004 ([abstract on Springer](https://dl.acm.org/doi/10.1145/1007912.1007944)). |
| 88 | «Treiber stack + exponential backoff — тот же CAS-цикл, но при промахе CAS поток ждёт растущий промежуток времени; throughput держится почти на уровне одного потока.» | ✅ ПОДТВЕРЖДЕНО | Качественно соответствует. |
| 89 | «MCS lock + serial stack — лок-стек на MCS-локе для контраста; работает стабильно, но не масштабируется по числу потоков (последовательное горлышко).» | ✅ ПОДТВЕРЖДЕНО | Mellor-Crummey & Scott TOCS 1991 — [DOI 10.1145/103727.103729](https://doi.org/10.1145/103727.103729). |
| 90 | (Точки на графике: ось X 1..32, Y до 4000) | ✅ FIXED (2026-05-18) | Добавлен markLine на x=14 с подписью «14 CPU (физический предел E6500)»; disclaimer над графиком явно отмечает, что точки правее x=14 — режим oversubscription. См. fix pass #1.4. |
| 91 | «Главный вывод: lock-free сам по себе не даёт масштабируемости — Treiber без backoff проигрывает MCS-локу уже при четырёх потоках.» | ✅ ПОДТВЕРЖДЕНО | Корректный качественный вывод; согласуется с Hendler et al. 2004. |
| 92 | «Хорошее масштабирование возникает только при добавлении дисциплины разрешения конфликтов: backoff (как у Treiber + backoff) или элиминации (Hendler–Shavit–Yerushalmi показывают, что elimination-backoff stack достигает ~7500 ops/сек при 32 потоках на тех же измерениях).» | ✅ FIXED (2026-05-18) | Иллюстративный статус всего блока зафиксирован disclaimer-параграфом над графиком (см. fix pass #1.4). Качественное направление — elimination-backoff масштабируется лучше — подтверждено абстрактом SPAA 2004. |
| 93 | «Урок: «lock-free» — это про прогресс, а не про скорость.» | ✅ ПОДТВЕРЖДЕНО | Это стандартная констатация в литературе; см. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |

### Раздел 9: Сравнение с альтернативами (строки 270–278)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 94 | (Строка таблицы Coarse mutex) | ✅ ПОДТВЕРЖДЕНО | Стандартное сравнение; [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 95 | (Строка Fine-grained locking) | ✅ ПОДТВЕРЖДЕНО | См. [Graefe, B-tree locking survey](https://15721.courses.cs.cmu.edu/spring2016/papers/a16-graefe.pdf). |
| 96 | (Строка Lock-free) | ✅ ПОДТВЕРЖДЕНО | См. там же. |
| 97 | (Строка Wait-free) | ✅ ПОДТВЕРЖДЕНО | См. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 98 | (Строка RCU: read «бесплатный (ноль atomic)») | ✅ ПОДТВЕРЖДЕНО | Read-side `rcu_read_lock()` на preemptible kernel практически нулевой; [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html). |

### Раздел 10: Подводные камни и анти-паттерны (строки 280–285)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 99 | ««Lock-free всегда быстрее» — миф. Herlihy & Shavit и эксперименты SPAA 2004 прямо показывают: при высоком контеншене чистый CAS-цикл проигрывает MCS-локу.» | ✅ ПОДТВЕРЖДЕНО | См. [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944). |
| 100 | «Live-lock без helping. Если поток откатывает свой CAS и сразу повторяет, а второй поток делает то же — оба бесконечно мешают друг другу.» | ✅ ПОДТВЕРЖДЕНО | Стандартная характеристика obstruction-free; [Herlihy/Luchangco DSTM 2003](https://cs.brown.edu/courses/cs161/papers/stm.pdf). |
| 101 | «ABA без HP / EBR / tag. В не-GC-среде узел переиспользуется → CAS на старом адресе проходит ошибочно.» | ✅ ПОДТВЕРЖДЕНО | См. [Wikipedia: ABA problem](https://en.wikipedia.org/wiki/ABA_problem); [Michael 2004](https://doi.org/10.1109/TPDS.2004.8). |
| 102 | «Verification. … модельные чекеры SPIN (LTL-свойства), CDSChecker (Norris–Demsky 2013, exhaustive interleavings под C11 memory model), GenMC (Kokologiannakis et al.), стресс-фреймворки jcstress (OpenJDK)…» | ✅ ПОДТВЕРЖДЕНО | SPIN: [spinroot.com](http://spinroot.com); CDSChecker: [DOI 10.1145/2509136.2509514](https://dl.acm.org/doi/10.1145/2509136.2509514) (OOPSLA 2013); GenMC: [plv.mpi-sws.org/genmc](https://plv.mpi-sws.org/genmc/); jcstress: [openjdk.org/projects/code-tools/jcstress](https://openjdk.org/projects/code-tools/jcstress/). |

### Раздел 11: Список литературы (строки 287–329)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 103 | [1] Herlihy, Shavit, Luchangco, Spear (2020) *Art of Multiprocessor Programming* 2nd ed. | ✅ ПОДТВЕРЖДЕНО | [Elsevier link verified](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1). |
| 104 | [2] Treiber 1986 RJ 5118 | ✅ ПОДТВЕРЖДЕНО | [IBM RJ 5118 PDF (4/23/86)](https://dominoweb.draco.res.ibm.com/reports/rj5118.pdf). |
| 105 | [3] Michael 2004 *Hazard Pointers* IEEE TPDS 15(6) 491–504 DOI 10.1109/TPDS.2004.8 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1109/TPDS.2004.8](https://doi.org/10.1109/TPDS.2004.8). |
| 106 | [4] Michael & Scott 1996 PODC '96 267–275 DOI 10.1145/248052.248106 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/248052.248106](https://doi.org/10.1145/248052.248106). |
| 107 | [5] Fraser 2004 *Practical Lock-Freedom* UCAM-CL-TR-579 | ✅ ПОДТВЕРЖДЕНО | [UCAM-CL-TR-579 PDF](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf). |
| 108 | [6] Harris 2001 DISC '01 LNCS 2180 300–314 DOI 10.1007/3-540-45414-4_21 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1007/3-540-45414-4_21](https://doi.org/10.1007/3-540-45414-4_21). |
| 109 | [7] Levandoski, Lomet, Sengupta 2013 ICDE 302–313 DOI 10.1109/ICDE.2013.6544834 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1109/ICDE.2013.6544834](https://doi.org/10.1109/ICDE.2013.6544834). |
| 110 | [8] Shalev & Shavit 2006 JACM 53(3) 379–405 DOI 10.1145/1147954.1147958 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1147954.1147958](https://doi.org/10.1145/1147954.1147958). |
| 111 | [9] Michael 2002 SPAA '02 73–82 DOI 10.1145/564870.564881 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/564870.564881](https://doi.org/10.1145/564870.564881). |
| 112 | [10] Arm Ltd. *Learn the architecture: LSE* | ✅ ПОДТВЕРЖДЕНО | [learn.arm.com/learning-paths/.../lse/intro/](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/). |
| 113 | [11] OpenJDK JEP 188 | ✅ ПОДТВЕРЖДЕНО | [openjdk.org/jeps/188](https://openjdk.org/jeps/188). |
| 114 | [12] OpenJDK CLQ source (HEAD JDK 25) | ✅ ПОДТВЕРЖДЕНО | [github.com/openjdk/jdk/.../ConcurrentLinkedQueue.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java). |
| 115 | [13] Oracle ConcurrentSkipListMap (Java SE 25 API) — «Javadoc прямо указывает, что реализация — концурентный вариант skip-list'ов на основе работ Fraser и Harris» | ✅ FIXED (2026-05-18) | Ссылка [13] переуказана на исходник [ConcurrentSkipListMap.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentSkipListMap.java); описание уточнено как class-level комментарий, не публичный Javadoc. См. fix pass #1.2. |
| 116 | [14] McKenney et al. *What is RCU, Fundamentally?* | ✅ ПОДТВЕРЖДЕНО | [kernel.org/doc/html/latest/RCU/whatisRCU.html](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html). |
| 117 | [15] Thompson, Farley, Barker, Gee, Stewart (2011) *Disruptor* LMAX Technical Paper | ✅ ПОДТВЕРЖДЕНО | [Disruptor 1.0 PDF](https://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf). Замечание: paper датируется 2011 (хотя сама v1.0 относится к 2010-2011 frame). |
| 118 | [16] Kogan & Petrank 2011 PPoPP '11 223–234 DOI 10.1145/1941553.1941585 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1941553.1941585](https://dl.acm.org/doi/10.1145/2038037.1941585). PPoPP 2011 в Сан-Антонио. |
| 119 | [17] Manson, Pugh, Adve 2005 POPL '05 378–391 DOI 10.1145/1040305.1040336 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1040305.1040336](https://doi.org/10.1145/1040305.1040336). |
| 120 | [18] Reeves 1997 *What Really Happened on Mars?* | ✅ ПОДТВЕРЖДЕНО | [cs.unc.edu/~anderson/.../mars_pathfinder_long_version.html](https://www.cs.unc.edu/~anderson/teach/comp790/papers/mars_pathfinder_long_version.html). Email от 15 декабря 1997. |
| 121 | [19] PostgreSQL Global Development Group *Lightweight Locks (LWLocks)* — ссылка на `src/backend/storage/lmgr/lwlock.c` | ✅ FIXED (2026-05-18) | Ссылка [19] переуказана на [xlog.c](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c) (где определены `NUM_XLOGINSERT_LOCKS=8` и массив `WALInsertLocks`); anchor `ref-postgres-lwlock` сохранён ради цитат в теле. См. fix pass #1.5. |
| 122 | [20] Hendler, Shavit, Yerushalmi 2004 SPAA '04 206–215 DOI 10.1145/1007912.1007944 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1007912.1007944](https://dl.acm.org/doi/10.1145/1007912.1007944). |
| 123 | [21] Mellor-Crummey & Scott 1991 ACM TOCS 9(1) 21–65 DOI 10.1145/103727.103729 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/103727.103729](https://doi.org/10.1145/103727.103729). |

---

## Сводка: проблемные пункты, требующие правок

| # | Строка | Проблема | Серьёзность |
|---|---|---|---|
| A | 15 | «мьютекс bus management в VxWorks» — мьютекс держал ASI/MET (низкоприоритетная задача), bus management — это блокированная высокоприоритетная задача. | ✅ FIXED (2026-05-18) — wording corrected per Reeves 1997: bc_dist blocked on IPC semaphore held by ASI/MET. См. fix pass #1.1. |
| B | 40 | «javadoc явно говорит» о CLQ — на самом деле это внутрикодовый комментарий, не публичный javadoc. | 🟢 Низкая |
| C | 111 | «DCAS на одном слове 128 бит» — терминологически путаница: CMPXCHG16B = DWCAS, не DCAS. | 🟢 Низкая |
| D | 126 | «`AbstractQueuedSynchronizer` использует Treiber-подобный CAS-стек ожидающих узлов» — AQS использует CLH-вариант FIFO-очереди, не Treiber-стек. Treiber-стек реально используется в FutureTask. | 🟡 Средняя |
| E | 164 | «Закрытое хеширование с lock-free chains» — терминология неправильная (closed/open hashing путаница; correct = separate chaining = open hashing). | 🟢 Низкая |
| F | 173 | «javadoc прямо указывает, что реализация Doug Lea основана на работах Fraser и Harris» — это не публичный javadoc, а class header в исходнике. | ✅ FIXED (2026-05-18) — wording softened to «class-level комментарий в исходнике (Doug Lea)»; ref [13] repointed to ConcurrentSkipListMap.java. См. fix pass #1.2. |
| G | 193 | «ядро NetBSD (`vmem`)» как пример EBR — `vmem` это resource allocator, не EBR. Корректнее `pserialize`. | ✅ FIXED (2026-05-18) — `vmem` → `pserialize(9)`; новая ссылка [22] добавлена. См. fix pass #1.3. |
| H | 202 | «LMAX Disruptor — кольцевой буфер с FAA-резервацией позиций; сами авторы аккуратно отмечают…» — Disruptor использует CAS, не FAA для multi-producer; явная атрибуция «не wait-free» авторам — не находится в paper. | 🟡 Средняя |
| I | 210 | «14-узловом Sun Enterprise E6500» — корректно по machine, но workload «50% push / 50% pop, 500 000 операций на поток» и точные числа графика не выверены против оригинала Figure 7 (PDF не парсится). | ✅ FIXED (2026-05-18) — disclaimer-параграф над графиком явно помечает точки как иллюстративные, не репродукцию Figure 7. См. fix pass #1.4. |
| J | 234 | Ось X графика идёт до 32 потоков, а 14-CPU машина не имеет столько ядер — потенциальная путаница. | ✅ FIXED (2026-05-18) — добавлен markLine на x=14 («14 CPU физический предел E6500»); disclaimer над графиком отмечает oversubscription. См. fix pass #1.4. |
| K | 268 | «elimination-backoff stack достигает ~7500 ops/сек при 32 потоках» — конкретное число не выверено. | 🟢 Низкая |
| L | 313 | Reference [13] описание ConcurrentSkipListMap javadoc — same as F. | ✅ FIXED (2026-05-18) — ref [13] переуказана на исходник ConcurrentSkipListMap.java; описание уточнено как class-level комментарий, не публичный Javadoc. См. fix pass #1.2. |
| M | 325 | Reference [19] — URL указывает на `lwlock.c`, но WALInsertLock-array определён в `xlog.c`. | ✅ FIXED (2026-05-18) — ref [19] репойнт `lwlock.c` → `xlog.c`; anchor `ref-postgres-lwlock` сохранён. См. fix pass #1.5. |
| N | 167 | «Cliff Click… в литературе линеаризуемость и lock-free-свойства остаются предметом дискуссии — формальных доказательств нет» — формально верно (нет peer-reviewed proof), но «использовать с осторожностью» — субъективно. | 🟢 Низкая |

---

## Fix pass 2026-05-18

| # | Item | Status | Summary |
|---|---|---|---|
| 1.1 | Pathfinder wording (#A, row 7) | FIXED (2026-05-18) | Reframed per Reeves 1997: ASI/MET held IPC semaphore inside `select()`, bc_dist blocked. |
| 1.2 | ConcurrentSkipListMap attribution (#F, #L, rows 63 & 115) | FIXED (2026-05-18) | Wording softened to in-source class header; ref [13] repointed to ConcurrentSkipListMap.java. |
| 1.3 | NetBSD EBR example (#G, row 73) | FIXED (2026-05-18) | `vmem` → `pserialize(9)`; new ref [22] added. |
| 1.4 | Contention chart (#I, #J, rows 85/86/90/92) | FIXED (2026-05-18) | Disclaimer paragraph added; x=14 markLine + caption noting 14-CPU E6500. |
| 1.5 | Reference [19] (#M, rows 10, 80, 121) | FIXED (2026-05-18) | Repointed lwlock.c → xlog.c (same anchor id retained). |

**Unresolved remarks: 0** (in-scope set: items #1.1–#1.5 all FIXED).

## Fix pass 2026-05-23

Target: close the two remaining ⚠️ rows in section-log (§5, §8) by addressing the items previously listed as "explicitly out of scope".

| # | Item | Status | Summary |
|---|---|---|---|
| 2.1 | AQS / Treiber production-user attribution (#D, row 51, page line 126) | FIXED (2026-05-23) | Replaced AQS (which actually uses a CLH FIFO queue, per AQS class-level javadoc: «The wait queue is a variant of a 'CLH' (Craig, Landin, and Hagersten) lock queue») with `java.util.concurrent.FutureTask.WaitNode` (in-source comment: «Simple linked list nodes to record waiting threads in a Treiber stack»). Added reference [23] to FutureTask.java; added a parenthetical clarifying that AQS is CLH-based, not Treiber. |
| 2.2 | Open/closed hashing terminology (#E, row 58, page line 164) | FIXED (2026-05-23) | «Закрытое хеширование с lock-free chains» → «Открытое хеширование (separate chaining) с lock-free-цепочками». Confirmed by multiple textbook sources (opendsa-server.cs.vt.edu, VisuAlgo): open hashing ≡ separate chaining; closed hashing ≡ open addressing. |
| 2.3 | Cliff Click subjective wording (#N, row 61, page line 167) | FIXED (2026-05-23) | Removed «рекомендуется использовать с осторожностью» (subjective). Kept the verifiable factual statement «peer-reviewed формального доказательства линеаризуемости и lock-free-свойств в литературе нет». |
| 2.4 | Elimination-backoff specific number (#K, row 92, page line 277) | FIXED (2026-05-23) | Dropped the unverifiable specific «~7500 ops/сек при 32 потоках» (SPAA 2004 Figure 7 PDF does not parse; no secondary source quotes a numeric throughput). Replaced with the qualitative claim «масштабируется почти линейно, тогда как обычный Treiber выходит на плато» — supported by SPAA 2004 abstract. Chart disclaimer from fix pass #1.4 remains in place. |

**Unresolved remarks: 0** (in-scope set: items #2.1–#2.4 all FIXED). After this pass, all rows in section-log are ✅.

Explicitly out of scope (remain ⚠️ by design at the sentence level, but not pulled into section-log status because they are stylistic / minor):
- #B (row 25, line 40) — CLQ «javadoc явно говорит» phrasing (mirror of #F, which was already closed at the source-comment level by fix pass #1.2; row-25 wording is identical to row-54 phrasing and was deemed acceptable in context).
- #C (row 44, line 111) — DCAS/DWCAS terminology (cosmetic; CMPXCHG16B itself is correct).
- #H (row 81, line 202) — Disruptor FAA/CAS + «не wait-free» attribution (Disruptor 1.0 paper does not contain the explicit phrase «not wait-free», but the qualitative claim about CAS-loop progress is correct; full rework requires reading the LMAX whitepaper end-to-end and is deferred).

Verified on live after deploy: deploy run `26330249944` (success, 2026-05-23T10:23:54Z); evaluator confirmed on production at 2026-05-23T10:25Z all four fixes (FutureTask.WaitNode replaces AQS attribution + ref [23] live, «Открытое хеширование (separate chaining)» live, Cliff Click без «осторожностью» live, elimination-backoff качественная формулировка без «~7500» live).
