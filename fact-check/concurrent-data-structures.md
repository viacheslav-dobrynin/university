# Fact-check: Lock-free и wait-free структуры данных

**Source**: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/lock-free/concurrent-data-structures.mdx`
**Live URL**: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/concurrent/lock-free/concurrent-data-structures

**2026-06-11 Phase-5 daily-rigor re-audit, replaces 2026-05-08/18/23 log.**

Full from-scratch sentence-by-sentence re-verification against primary sources. Preflight 2026-06-11: production URL loads (title «Материалы дисциплины…» подтверждён через Playwright). Note: репозиторий не содержит Actions-workflow `deploy.yml` (`gh workflow list` пуст) — деплой идёт через GitHub Pages; критичная проверка (живая страница) пройдена.

## Summary count

- ✅ Подтверждено: 95
- ⚠️ Неточно / требует правки: 6
- ❌ Фактическая ошибка: 1
- 🔍 Не удалось подтвердить: 0
- 💬 Не факт (метатекст / суждение): 9

## VERDICT: NEEDS-FIXES

Один ❌ и шесть ⚠️. Самое серьёзное — **новая находка**, пропущенная всеми предыдущими пассами: утверждение, что PostgreSQL резервирует позицию WAL «через FAA», неверно — реальный код использует **спинлок** (`insertpos_lck`). Перечень внизу («Проблемные пункты»).

## Section log

| # | Раздел | Строки | Статус | Резюме |
|---|---|---|---|---|
| 1 | Мотивация | 10–20 | ⚠️ 1 ошибка | Pathfinder (ASI/MET держит IPC-семафор, blocked bc_dist), LWLock, Bw-tree, RCU — подтверждены. **WALInsertLock «через FAA» — НЕВЕРНО** (xlog.c использует спинлок). |
| 2 | Условия прогресса | 22–40 | ✅ | Иерархия wait-free⟹lock-free⟹obstruction-free подтверждена дословно; CLQ lock-free-не-wait-free, Kogan-Petrank PPoPP 2011 — верно. «monotonic strict total order» — нестандартная формулировка (💬). |
| 3 | Атомарные примитивы и память | 42–104 | ✅ | x86 `LOCK CMPXCHG`/`XADD`, ARMv8.1 LSE CAS/CASA/CASL/CASAL, LDXR/STXR (LL/SC), memory_order, JEP 188/JSR-133, publish-pattern — все подтверждены. |
| 4 | Проблема ABA | 106–116 | ⚠️ cosmetic | ABA-сценарий, HP (Michael 2004), EBR (Fraser 2004), RCU, GC-обоснование — верны. «DCAS на одном слове 128 бит» — терминологически DWCAS, не DCAS (💬/cosmetic). |
| 5 | Каноничные конструкции | 117–185 | ⚠️ 1 неточность | Treiber RJ 5118, M&S PODC 1996 (dummy node + двойной CAS-helping подтверждён по PDF), Harris DISC 2001, Michael SPAA 2002, Shalev-Shavit, Click, Fraser, Bw-tree — подтверждены. FutureTask.WaitNode «Treiber stack» подтверждён дословно. **CSLM: исходник атрибутирует базовые списки HM-алгоритму (Harris+Maged Michael), Fraser — related work** — страница говорит «Fraser и Harris», что неполно. |
| 6 | Безопасное освобождение памяти | 187–197 | ✅ | HP/EBR/RCU, C++26 `std::hazard_pointer`, crossbeam-epoch, NetBSD pserialize(9), liburcu — подтверждены. |
| 7 | Применения в БД | 198–207 | ❌ 1 ошибка | Bw-tree→Hekaton/Cosmos/Bing, MVCC, RCU, LeanStore/Umbra — верны. **«WALInsertLock … через FAA» — НЕВЕРНО** (спинлок). Disruptor — см. §+. |
| 8 | Визуализация | 208–277 | ✅ | Hendler-Shavit-Yerushalmi SPAA 2004 (14-CPU E6500), иллюстративный disclaimer, markLine x=14 — корректно. |
| 9 | Сравнение с альтернативами | 279–287 | ✅ | Все строки качественно корректны. |
| 10 | Подводные камни | 289–294 | ✅ | SPIN, CDSChecker, GenMC, jcstress — подтверждены. |
| 11 | Список литературы | 296–342 | ⚠️ 1 неточность | DOI/URL подтверждены. **Ref [19] описание «через FAA» — НЕВЕРНО**. Ref [15] (Disruptor) описание «не wait-free» — paper не содержит фразы. |
| + | Disruptor (строка 202, ref 15) | — | ⚠️ | Disruptor использует **CAS**, не FAA, для multi-producer claim (подтверждено по PDF); paper говорит «lock-free», но фразы «wait-free»/«не wait-free» в нём НЕТ. |

---

## Sentence-by-sentence verification

### Раздел 1: Мотивация (строки 10–20)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 1 | «Классический способ … — взять мьютекс.» | 💬 НЕ ФАКТ | Вводное обобщение. [Herlihy & Shavit, AoMP](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |
| 2 | «Семантика проста, доказательства линеаризуемости тривиальны…» | 💬 НЕ ФАКТ | Субъективное суждение. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |
| 3 | «…платой служит блокирующее поведение: если поток-владелец захвачен прерыванием, page fault'ом, GC-паузой или вытеснен — все ждущие встают.» | ✅ ПОДТВЕРЖДЕНО | Следствие mutual exclusion. [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 4 | «Lock convoy. Поток отпускает мьютекс, его подбирает другой; первый встаёт в конец очереди.» | ✅ ПОДТВЕРЖДЕНО | [Microsoft DevBlogs: Lock convoys](https://devblogs.microsoft.com/oldnewthing/20170307-00/?p=95775); [Wikipedia: Lock convoy](https://en.wikipedia.org/wiki/Lock_convoy) |
| 5 | «Под высокой нагрузкой пропускная способность падает в разы.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Lock convoy](https://en.wikipedia.org/wiki/Lock_convoy) |
| 6 | «Priority inversion. Низкоприоритетный держит замок, нужный высокоприоритетному; среднеприоритетный вытесняет низкоприоритетный, высокоприоритетный ждёт.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Priority inversion](https://en.wikipedia.org/wiki/Priority_inversion) |
| 7 | «Канонический пример — Mars Pathfinder (1997): в VxWorks бинарный IPC-семафор внутри `select()` удерживала низкоприоритетная задача метеорологии ASI/MET, на нём блокировалась высокоприоритетная `bc_dist`, а вытеснение среднеприоритетной communications-задачей затягивало блокировку настолько, что watchdog перезагружал систему.» | ✅ ПОДТВЕРЖДЕНО | Дословно совпадает с Reeves 1997: ASI/MET держал mutex внутри `select()`/`selNodeAdd()`; bc_dist блокировался в `pipeWrite()`; medium-priority задачи затягивали; fix — priority inheritance. [Reeves: What Really Happened on Mars](https://www.cs.unc.edu/~anderson/teach/comp790/papers/mars_pathfinder_long_version.html) |
| 8 | «Stop-the-world эффекты. Любая задержка владельца замка масштабируется на всех ждущих…» | ✅ ПОДТВЕРЖДЕНО | [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |
| 9 | «PostgreSQL имеет десятки LWLock для разделяемой памяти (буферный пул, WAL-инсерт, ProcArray)…» | ✅ ПОДТВЕРЖДЕНО | [PostgreSQL `lwlock.c`](https://github.com/postgres/postgres/blob/master/src/backend/storage/lmgr/lwlock.c) |
| 10 | «…под нагрузкой `WALInsertLock` исторически становится горлышком и потребовал шардирования на 8 «слотов».» | ✅ ПОДТВЕРЖДЕНО | `#define NUM_XLOGINSERT_LOCKS 8` + массив `WALInsertLocks` подтверждены в [xlog.c](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c). (Шардирование на 8 — верно; механизм резервации см. строку 80.) |
| 11 | «InnoDB использует latch'и на B-tree-страницы (latch coupling) — именно их избегает Bw-tree…» | ✅ ПОДТВЕРЖДЕНО | Bw-tree latch-free: [Levandoski et al. ICDE 2013 PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf) |
| 12 | «…хранящий состояние страниц как цепочку delta-записей и переключающий версии одним CAS.» | ✅ ПОДТВЕРЖДЕНО | [Bw-tree ICDE 2013 PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf) |
| 13 | «MVCC-движки (Postgres, Oracle) держат версионные цепочки и атомарно переключают snapshot-указатели, не блокируя читателей.» | ✅ ПОДТВЕРЖДЕНО | [PostgreSQL MVCC](https://www.postgresql.org/docs/current/mvcc-intro.html) |
| 14 | «Linux-ядро использует RCU в dcache, ip-routing и BPF-картах…» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html); [BPF maps](https://docs.kernel.org/bpf/maps.html) |
| 15 | «Lock-free и wait-free структуры данных — семейство алгоритмов, где блокирующий примитив заменён на атомарные RMW (`CAS`, `FAA`, `LL/SC`) с явной моделью памяти.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 16 | «Они дают прогресс системы при любой задержке отдельного потока…» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 17 | «Эта статья — о том, какие гарантии прогресса бывают…» | 💬 НЕ ФАКТ | Метатекст. [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |

### Раздел 2: Условия прогресса (строки 22–40)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 18 | «Иерархию условий прогресса формализовали Herlihy и Shavit.» | ✅ ПОДТВЕРЖДЕНО | Главы 3–4. [Herlihy & Shavit, AoMP](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |
| 19 | «Blocking. Прогресс отдельного потока зависит от других…» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 20 | «Obstruction-free. Любой поток в одиночку завершит операцию за конечное число шагов. Допускает live-lock.» | ✅ ПОДТВЕРЖДЕНО | Точное определение: прогресс гарантирован только при изоляции, возможен livelock. [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm); [DSTM PODC 2003](https://www.cs.brown.edu/people/mph/HerlihyLM03/main.pdf) |
| 21 | «Lock-free. … хотя бы один завершит операцию за конечное число шагов … конкретный поток может голодать (starvation).» | ✅ ПОДТВЕРЖДЕНО | «at least one thread makes progress … allows individual threads to starve». [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 22 | «Wait-free. Каждый поток завершает операцию за ограниченное число своих шагов, независимо от других. Сильнейшая гарантия.» | ✅ ПОДТВЕРЖДЕНО | «bound on the number of steps … no thread starves». [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 23 | «Свойства monotonic strict total order: wait-free ⟹ lock-free ⟹ obstruction-free. Lock-free автоматически obstruction-free, но не наоборот.» | ⚠️ НЕТОЧНО (формулировка) | Импликации **верны** дословно («all wait-free are lock-free; all lock-free are obstruction-free»). Но фраза «monotonic strict total order» — нестандартный/сбивающий с толку дескриптор для цепочки строгих импликаций (это просто линейная иерархия вложенности классов). Содержание корректно; ярлык — стилистический. [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 24 | Таблица условий прогресса (Treiber/M&S/Harris → lock-free; DSTM 2003 → obstruction-free; Kogan–Petrank 2011 → wait-free) | ✅ ПОДТВЕРЖДЕНО | DSTM: [Herlihy et al. PODC 2003](https://www.cs.brown.edu/people/mph/HerlihyLM03/main.pdf). Kogan-Petrank: [DOI 10.1145/1941553.1941585](https://doi.org/10.1145/1941553.1941585) |
| 25 | «`java.util.concurrent.ConcurrentLinkedQueue` — lock-free, но не wait-free: javadoc явно говорит, что это вариант алгоритма Майкла–Скотта PODC 1996, в котором поток-неудачник повторяет CAS-цикл.» | ⚠️ НЕТОЧНО | Содержательно верно (lock-free, не wait-free, M&S PODC 1996). Но ссылка на M&S — это **class-level комментарий в исходнике** CLQ.java (дословно: «based on one described in … Michael & Scott»), а не публичный API-javadoc. Идентично row 54. [CLQ.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java) |
| 26 | «Wait-free-варианты той же задачи существуют (Kogan–Petrank, PPoPP 2011), но дороже на низком контеншене.» | ✅ ПОДТВЕРЖДЕНО | Kogan & Petrank, *Wait-Free Queues With Multiple Enqueuers and Dequeuers*, PPoPP '11, 223–234. [DOI 10.1145/1941553.1941585](https://doi.org/10.1145/1941553.1941585) |

### Раздел 3: Атомарные примитивы и память (строки 42–104)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 27 | «CAS — универсальный примитив; принимает `addr`, `expected`, `new`; атомарно проверяет `*addr==expected`, пишет `new`, возвращает успех.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap) |
| 28 | «На x86-64 кодируется как `LOCK CMPXCHG`…» | ✅ ПОДТВЕРЖДЕНО | [Felix Cloutier x86: CMPXCHG](https://www.felixcloutier.com/x86/cmpxchg) |
| 29 | «…на ARMv8.1+ — единственная инструкция CAS/CASA/CASL/CASAL (acquire/release-варианты), часть LSE.» | ✅ ПОДТВЕРЖДЕНО | FEAT_LSE вводит CAS/CASP в ARMv8.1. [Arm: LSE](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/) |
| 30 | «На более старом AArch64 (до 8.1) и ARMv7 CAS реализуется парой LDXR/STXR (или LDREX/STREX), то есть LL/SC.» | ✅ ПОДТВЕРЖДЕНО | До LSE — LDXR/STXR цикл. [Arm: LSE](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/); [Wikipedia: LL/SC](https://en.wikipedia.org/wiki/Load-link/store-conditional) |
| 31 | «LL/SC — альтернативная пара примитивов, исторически на DEC Alpha, MIPS, PowerPC, ARM.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: LL/SC](https://en.wikipedia.org/wiki/Load-link/store-conditional) |
| 32 | «Преимущество перед CAS — естественная защита от ABA на уровне reservation set'а; недостаток — false-sharing срывает SC.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: LL/SC](https://en.wikipedia.org/wiki/Load-link/store-conditional) |
| 33 | «FAA — атомарное `*addr += delta` с возвратом старого значения; на x86 — `LOCK XADD`.» | ✅ ПОДТВЕРЖДЕНО | [Felix Cloutier x86: XADD](https://www.felixcloutier.com/x86/xadd); [Wikipedia: Fetch-and-add](https://en.wikipedia.org/wiki/Fetch-and-add) |
| 34 | «Дешевле CAS-цикла на конкурентных счётчиках: каждая операция прогрессирует за один шаг (FAA — wait-free по определению).» | ✅ ПОДТВЕРЖДЕНО | FAA завершается за ограниченное число шагов → wait-free. [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 35 | «Современные CPU и компиляторы переупорядочивают доступы; нужно явно навязать порядок.» | ✅ ПОДТВЕРЖДЕНО | [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 36 | «`relaxed` — атомарность без ограничений на порядок.» | ✅ ПОДТВЕРЖДЕНО | «no synchronization or ordering constraints … only atomicity». [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 37 | «`acquire` — обычные доступы после операции не уезжают перед ней.» | ✅ ПОДТВЕРЖДЕНО | «no reads/writes in current thread can be reordered before this load». [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 38 | «`release` — обычные доступы до операции не уезжают после неё.» | ✅ ПОДТВЕРЖДЕНО | «no reads/writes can be reordered after this store». [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 39 | «`acq-rel`/`seq-cst` — оба ограничения; `seq_cst` плюс глобальный полный порядок.» | ✅ ПОДТВЕРЖДЕНО | seq_cst = acq+rel + единый total order. [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 40 | «В Java — volatile-чтения/записи, `VarHandle.compareAndSet`, JEP 188 / JSR-133 определяют модель памяти, обновлённую в Java 5 и уточнённую в Java 9.» | ✅ ПОДТВЕРЖДЕНО | [JEP 188](https://openjdk.org/jeps/188); [Manson-Pugh-Adve POPL '05 (JSR-133)](https://doi.org/10.1145/1040305.1040336) |
| 41 | «В C++11 — `std::memory_order` поверх `std::atomic`.» | ✅ ПОДТВЕРЖДЕНО | [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 42 | «Без этих ограничений компилятор может переставить инициализацию узла после CAS — другой поток увидит мусор.» | ✅ ПОДТВЕРЖДЕНО | Публикационная гонка. [cppreference: memory_order (release-acquire)](https://en.cppreference.com/w/cpp/atomic/memory_order) |
| 43 | Псевдокод атомарного счётчика через CAS (`compare_exchange_weak`, acq_rel/relaxed). | ✅ ПОДТВЕРЖДЕНО | Корректная семантика. [cppreference: compare_exchange](https://en.cppreference.com/w/cpp/atomic/atomic/compare_exchange) |
| 44 | «Альтернатива через FAA — одна инструкция, без цикла, поэтому wait-free.» | ✅ ПОДТВЕРЖДЕНО | [cppreference: atomic::fetch_add](https://en.cppreference.com/w/cpp/atomic/atomic/fetch_add) |
| 45 | Псевдокод Treiber push (CAS `top` с release). | ✅ ПОДТВЕРЖДЕНО | Соответствует Treiber 1986. [Wikipedia: Treiber stack](https://en.wikipedia.org/wiki/Treiber_stack) |
| 46 | «`release` на CAS гарантирует, что инициализация `value` видна `pop` с `acquire` — publish-pattern.» | ✅ ПОДТВЕРЖДЕНО | release/acquire synchronizes-with. [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order) |

### Раздел 4: Проблема ABA (строки 106–116)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 47 | «Сценарий: T1 читает top=A, готовит CAS A→C, засыпает; T2 делает pop(A),pop(B),push(A) — top снова A, но другая жизнь узла; CAS T1 проходит, next указывает на освобождённый B.» | ✅ ПОДТВЕРЖДЕНО | Канонический ABA. [Wikipedia: ABA problem](https://en.wikipedia.org/wiki/ABA_problem) |
| 48 | «Tagged-pointer / counted-pointer. Пара (ptr, tag) атомарно; tag растёт на каждом CAS. На x86-64 — 16-байтная пара через `CMPXCHG16B` (DCAS на одном слове 128 бит).» | ⚠️ НЕТОЧНО (терминология) | Механизм верен; CMPXCHG16B — это **DWCAS** (double-width CAS), а не **DCAS** (double-compare-and-swap по двум разным адресам). Косметика. [Wikipedia: Double compare-and-swap](https://en.wikipedia.org/wiki/Double_compare-and-swap); [Felix Cloutier: CMPXCHG8B/16B](https://www.felixcloutier.com/x86/cmpxchg8b:cmpxchg16b) |
| 49 | «Hazard pointers (Michael 2004) — явный реестр «я читаю X»; reclaimer не освобождает X, пока кто-то его помечает.» | ✅ ПОДТВЕРЖДЕНО | Maged Michael, IEEE TPDS 15(6) 2004. [Wikipedia: Hazard pointer](https://en.wikipedia.org/wiki/Hazard_pointer); [DOI 10.1109/TPDS.2004.8](https://doi.org/10.1109/TPDS.2004.8) |
| 50 | «EBR (epoch-based) — Fraser 2004 — потоки помечают эпоху; reclaimer освобождает узел, ушедший на две эпохи назад.» | ✅ ПОДТВЕРЖДЕНО | [Fraser, UCAM-CL-TR-579 (Feb 2004)](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf) |
| 51 | «RCU — близкий механизм в ядре Linux: писатели делают копию + atomic swap + grace period; читатели без синхронизации.» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html) |
| 52 | «GC. В Java и Go ABA на уровне «узел переиспользован» невозможен; JDK-реализация Майкла–Скотта обходится без счётчика версии.» | ✅ ПОДТВЕРЖДЕНО | Дословно в комментарии CLQ.java: «in garbage collected systems, there is no possibility of ABA problems due to recycled nodes». [CLQ.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java) |

### Раздел 5: Каноничные конструкции (строки 117–185)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 53 | «R. Kent Treiber, IBM Research Report RJ 5118, апрель 1986.» | ✅ ПОДТВЕРЖДЕНО | Treiber 1986, «Systems Programming: Coping with Parallelism», RJ 5118. [Wikipedia: Treiber stack](https://en.wikipedia.org/wiki/Treiber_stack); [IBM RJ 5118 PDF](https://dominoweb.draco.res.ibm.com/reports/rj5118.pdf) |
| 54 | «Односвязный список, единственное общее поле — указатель на вершину; push и pop на одном CAS.» | ✅ ПОДТВЕРЖДЕНО | «lock-free stack … CAS on top pointer». [Wikipedia: Treiber stack](https://en.wikipedia.org/wiki/Treiber_stack) |
| 55 | «Идея/инвариант/стоимость: O(1) без конкуренции; под контеншеном серия CAS-промахов; latency деградирует линейно, throughput на плато.» | ✅ ПОДТВЕРЖДЕНО | Согласуется с измерениями. [Hendler-Shavit-Yerushalmi SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 56 | «Production user: `FutureTask` (внутренний класс `WaitNode` — комментарий «Simple linked list nodes to record waiting threads in a Treiber stack»); те же идеи в `Phaser` и `SynchronousQueue`. AQS использует CLH FIFO-очередь, не Treiber-стек.» | ✅ ПОДТВЕРЖДЕНО | Дословно: комментарий WaitNode «…in a Treiber stack. See other classes such as Phaser and SynchronousQueue…»; вставка — `WAITERS.weakCompareAndSet(this, q.next = waiters, q)`. AQS-уточнение тоже верно. [FutureTask.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/FutureTask.java) |
| 57 | «Maged M. Michael, Michael L. Scott. *Simple, Fast, and Practical Non-Blocking and Blocking Concurrent Queue Algorithms*, PODC '96.» | ✅ ПОДТВЕРЖДЕНО | Заголовок/авторы подтверждены по PDF. [M&S PODC 1996 PDF](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf); [DOI 10.1145/248052.248106](https://doi.org/10.1145/248052.248106) |
| 58 | «Односвязный список с двумя указателями (Head, Tail) и dummy-узлом в начале.» | ✅ ПОДТВЕРЖДЕНО | По PDF: «we keep a dummy node at the beginning of the list». [M&S PODC 1996 PDF](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf) |
| 59 | Псевдокод enqueue: snapshot tail/next, CAS `tail.next`, затем CAS `Tail` («помогли»); else CAS `Tail←next`. | ✅ ПОДТВЕРЖДЕНО | По PDF: `CAS(&tail.ptr->next, next, node)` затем `CAS(&Q->Tail, tail, node)`; отстающий Tail двигается `CAS(&Q->Tail, tail, next.ptr)`. [M&S PODC 1996 PDF](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf) |
| 60 | «`dequeue` симметричен: CAS Head←head.next, dummy-узел не даёт chain'у отвалиться. Алгоритм linearizable, lock-free, требует ABA-защиты в не-GC.» | ✅ ПОДТВЕРЖДЕНО | По PDF: `CAS(&Q->Head, head, next.ptr)`; «linearizable», «lock-free, non-blocking»; counted-pointers (tag) в не-GC версии. [M&S PODC 1996 PDF](https://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf) |
| 61 | «Production user: OpenJDK `ConcurrentLinkedQueue` — прямой потомок; внутрикодовый комментарий ссылается на M&S PODC 1996 и поясняет, что в GC-среде счётчики не нужны.» | ✅ ПОДТВЕРЖДЕНО | Комментарий CLQ.java дословно цитирует M&S и объясняет отсутствие counted pointers под GC. [CLQ.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java) |
| 62 | «Timothy L. Harris. *A Pragmatic Implementation of Non-Blocking Linked-Lists*, DISC '01, LNCS 2180.» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1007/3-540-45414-4_21](https://doi.org/10.1007/3-540-45414-4_21); [PDF](https://timharris.uk/papers/2001-disc.pdf) |
| 63 | «Двухшаговое удаление: (1) CAS метит next.marked false→true (логическое удаление); (2) CAS у предыдущего перебрасывает prev.next (физическое). Linearizability в точке (1); другой поток может «помочь».» | ✅ ПОДТВЕРЖДЕНО | Каноническое описание Harris-удаления. [Harris DISC 2001 PDF](https://timharris.uk/papers/2001-disc.pdf); [Wikipedia: Non-blocking linked list](https://en.wikipedia.org/wiki/Non-blocking_linked_list) |
| 64 | «Maged Michael (SPAA 2002) расширил конструкцию для lock-free hash-table chains, добавив hazard pointers.» | ✅ ПОДТВЕРЖДЕНО | [Michael SPAA '02, DOI 10.1145/564870.564881](https://doi.org/10.1145/564870.564881) |
| 65 | «Открытое хеширование (separate chaining) с lock-free-цепочками (Michael, SPAA 2002): каждый bucket — Harris-список; resize дороже, обычно блокирующий или incremental.» | ✅ ПОДТВЕРЖДЕНО | separate chaining ≡ open hashing; верно. [Michael SPAA '02](https://doi.org/10.1145/564870.564881); [Wikipedia: Hash table — separate chaining](https://en.wikipedia.org/wiki/Hash_table#Separate_chaining) |
| 66 | «Split-ordered lists (Shalev & Shavit, JACM 53(3), 2006): единый отсортированный список; resize — увеличение числа bucket'ов без перепланирования цепочек; O(1) ожидаемая стоимость, lock-free через Harris-маркеры.» | ✅ ПОДТВЕРЖДЕНО | [Shalev & Shavit, JACM 53(3) 2006, DOI 10.1145/1147954.1147958](https://doi.org/10.1145/1147954.1147958) |
| 67 | «Cliff Click (2007) NonBlockingHashMap для Java на принципах FSM-state-машины ячейки; peer-reviewed формального доказательства линеаризуемости и lock-free-свойств в литературе нет — оба свойства держатся на конструктивном описании автора.» | ✅ ПОДТВЕРЖДЕНО | Доклад 2007; нет рецензируемого формального доказательства. [Stanford EE380 2007](https://web.stanford.edu/class/ee380/Abstracts/070221_LockFreeHash.pdf) |
| 68 | «Keir Fraser, *Practical Lock-Freedom*, Cambridge UCAM-CL-TR-579 (2004); Fraser & Harris (2007). Каждый уровень skip-list — Harris-список; O(log n) ожидаемая стоимость.» | ✅ ПОДТВЕРЖДЕНО | [UCAM-CL-TR-579 PDF](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf); [Fraser & Harris TOCS 2007, DOI 10.1145/1233307.1233309](https://doi.org/10.1145/1233307.1233309) |
| 69 | «Production user: OpenJDK `ConcurrentSkipListMap` … class-level комментарий (Doug Lea) явно указывает, что реализация основана на работах Fraser и Harris.» | ⚠️ НЕТОЧНО | Комментарий CSLM.java атрибутирует **базовые списки HM-алгоритму** (дословно: «The base lists use a variant of the HM linked ordered set algorithm. See Tim Harris … and Maged Michael …»); Fraser назван среди thesis-работ с «features sharing … this one», т.е. как related work, не как основа. Страница говорит «основана на работах Fraser и Harris» — это неполно/смещает атрибуцию (опущен Maged Michael — соавтор «HM», и роль Fraser преувеличена). [CSLM.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentSkipListMap.java) |
| 70 | «Justin J. Levandoski, David B. Lomet, Sudipta Sengupta. *The Bw-Tree: A B-tree for New Hardware Platforms*, ICDE 2013.» | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1109/ICDE.2013.6544834](https://doi.org/10.1109/ICDE.2013.6544834); [MSR PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf) |
| 71 | «Latch-free B-tree под кеши многоядерных CPU и flash-хранилища.» | ✅ ПОДТВЕРЖДЕНО | [Bw-tree ICDE 2013 PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf) |
| 72 | «Bullets: PID/mapping table, delta-records prepended, единственный CAS на mapping table, ленивые splits/merges/consolidations с helping.» | ✅ ПОДТВЕРЖДЕНО | [Bw-tree ICDE 2013 PDF](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf) |
| 73 | «Production user: SQL Server Hekaton, Azure Cosmos DB / DocumentDB, Bing's index storage; open-source (CMU's BwTree, Peloton).» | ✅ ПОДТВЕРЖДЕНО | «shipping in three of Microsoft's products: SQL Server Hekaton, Azure DocumentDB, Bing ObjectStore». [SDC15 SNIA Bw-tree](https://www.snia.org/sites/default/files/SDC15_presentations/database/SudiptaSengupta_Bw-Tree-Key_Value.pdf); [CMU BwTree](https://github.com/wangziqi2013/BwTree) |
| 74 | «Каноническая ссылка на ICDE 2013 — paper и расширенная VLDB-версия 2013.» | ✅ ПОДТВЕРЖДЕНО | VLDB 2013 «The Bw-Tree: A Latch-Free B-Tree for Log-Structured Flash Storage». [VLDB 2013](http://www.vldb.org/pvldb/vol6.html) |

### Раздел 6: Безопасное освобождение памяти (строки 187–197)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 75 | «В средах без GC удаление узла порождает gap: узел отсоединён, но другие потоки держат ссылку. Преждевременное освобождение → use-after-free и ABA.» | ✅ ПОДТВЕРЖДЕНО | [Michael 2004, DOI 10.1109/TPDS.2004.8](https://doi.org/10.1109/TPDS.2004.8) |
| 76 | Таблица HP: «≤K защищённых указателей; reader — atomic-store + fence; reclamation O(N·K). Production: Folly, ConcurrencyFreaks, C++26 `std::hazard_pointer`.» | ✅ ПОДТВЕРЖДЕНО | [Michael 2004](https://doi.org/10.1109/TPDS.2004.8); C++26: [cppreference: hazard_pointer](https://en.cppreference.com/w/cpp/header/hazard_pointer.html); [P2530R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2530r3.pdf) |
| 77 | Таблица EBR: «глобальная эпоха; reader — atomic-store на критическую секцию; дешевле HP при многих ссылках. Production: crossbeam-epoch, NetBSD pserialize(9).» | ✅ ПОДТВЕРЖДЕНО | [crossbeam-epoch](https://docs.rs/crossbeam-epoch/); [NetBSD pserialize(9)](https://man.netbsd.org/pserialize.9) |
| 78 | Таблица RCU: «reader — ноль (только зависимостный read); reclamation — grace-period detection. Production: Linux dcache/IP routing/BPF, liburcu.» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html); [liburcu.org](https://liburcu.org/) |
| 79 | «HP и EBR применимы из user-space; RCU исторически kernel, но liburcu даёт userspace-вариант.» | ✅ ПОДТВЕРЖДЕНО | [LWN: User-space RCU](https://lwn.net/Articles/573424/) |
| 80 | «Выбор: RCU при горячих читателях и редких обновлениях; HP при ограниченном K; EBR при многих потоках и простоте.» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html) |

### Раздел 7: Применения в БД и распределённых системах (строки 198–207)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 81 | «Bw-tree → SQL Server Hekaton. Latch-free B-tree из ICDE 2013 работает в in-memory OLTP SQL Server, Azure Cosmos DB и Bing.» | ✅ ПОДТВЕРЖДЕНО | [SDC15 SNIA Bw-tree](https://www.snia.org/sites/default/files/SDC15_presentations/database/SudiptaSengupta_Bw-Tree-Key_Value.pdf) |
| 82 | «MVCC snapshot pointers. В PostgreSQL и Oracle переход к новой версии строки — атомарная установка указателя в tuple-header; читатели со snapshot'ом видят старую версию без блокировки.» | ✅ ПОДТВЕРЖДЕНО | [PostgreSQL MVCC](https://www.postgresql.org/docs/current/mvcc-intro.html) |
| 83 | «vacuum / undo играет роль reclamation с условиями, аналогичными EBR (нет читателей старше xmin).» | ✅ ПОДТВЕРЖДЕНО | [PostgreSQL VACUUM](https://www.postgresql.org/docs/current/routine-vacuuming.html) |
| 84 | «В PostgreSQL `WALInsertLock` исторически был bottleneck'ом и начиная с 9.4 шардирован на 8 слотов **с lock-free reservation позиции через FAA**.» | ❌ ФАКТИЧЕСКАЯ ОШИБКА | Шардирование на 8 слотов (`NUM_XLOGINSERT_LOCKS 8`) — верно. Но **резервация позиции НЕ через FAA**: `ReserveXLogInsertLocation()` в xlog.c использует **спинлок** `insertpos_lck` вокруг RMW по `Insert->CurrBytePos` (`SpinLockAcquire(&Insert->insertpos_lck); … CurrBytePos = endbytepos; SpinLockRelease(...)`), а не `pg_atomic_fetch_add`. Утверждение «через FAA» неверно. [xlog.c — ReserveXLogInsertLocation](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c) |
| 85 | «LMAX Disruptor — кольцевой буфер **с FAA-резервацией позиций**; сами авторы аккуратно отмечают, что Disruptor lock-free, но **не wait-free**: при коллизии producer'ы могут выполнять несколько CAS…» | ⚠️ НЕТОЧНО (две ошибки) | (a) **FAA неверно**: Disruptor 1.0 paper — multi-producer claim через **CAS**, не FAA («Contention on claiming the next available entry can be managed with a simple CAS operation on the sequence»; «an atomic counter updated using CAS operations in the case of multiple producers»). Сам же текст ниже корректно говорит «несколько CAS» — внутреннее противоречие. (b) **«сами авторы … не wait-free»**: paper содержит «lock-free», но фразы «wait-free»/«не wait-free» в нём **НЕТ** — атрибуция этой формулировки авторам — overreach. Качественный вывод (CAS-цикл producer'а может проигрывать) верен. [Disruptor 1.0 PDF](https://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf) |
| 86 | «RCU в ядре Linux. dcache, FIB, BPF-карты — read-side без атомарных операций; обновление через copy + swap + synchronize_rcu.» | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html); [BPF maps](https://docs.kernel.org/bpf/maps.html) |
| 87 | «Lock-free buffer pools. LeanStore, Umbra применяют CLOCK-replacement с атомарным reference-bit и lock-free-захватом frame'ов через CAS на состоянии страницы.» | ✅ ПОДТВЕРЖДЕНО | [LeanStore paper (TUM)](https://db.in.tum.de/~leis/papers/leanstore.pdf) |
| 88 | Ссылка на смежные страницы (Concurrent Hash Map, immutable/persistent). | 💬 НЕ ФАКТ | Внутренняя навигация. |

### Раздел 8: Визуализация (строки 208–277)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 89 | «Канонический эксперимент — продьюсер-консьюмер на стеке.» | 💬 НЕ ФАКТ | Постановочная фраза. [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 90 | «Hendler, Shavit и Yerushalmi (SPAA 2004) измерили 4 реализации на 14-узловом Sun Enterprise E6500 (UltraSPARC, Solaris 9), 50% push/50% pop, 500 000 операций на поток.» | ✅ ПОДТВЕРЖДЕНО | [Hendler-Shavit-Yerushalmi SPAA 2004, DOI 10.1145/1007912.1007944](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 91 | «Числа на графике — операций в секунду, считаны с Figure 7.» (с disclaimer-параграфом ниже) | ✅ ПОДТВЕРЖДЕНО (с disclaimer) | Disclaimer ниже явно помечает точки как иллюстративные, не репродукцию Figure 7. [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 92 | «Treiber stack (без backoff) — чистый CAS-цикл, пропускная способность падает (cache-line ping-pong на top).» | ✅ ПОДТВЕРЖДЕНО | [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 93 | «Treiber + exponential backoff — throughput держится почти на уровне одного потока.» | ✅ ПОДТВЕРЖДЕНО | [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 94 | «MCS lock + serial stack — стабильно, но не масштабируется (последовательное горлышко).» | ✅ ПОДТВЕРЖДЕНО | [Mellor-Crummey & Scott TOCS 1991, DOI 10.1145/103727.103729](https://doi.org/10.1145/103727.103729) |
| 95 | «Иллюстративные значения, не репродукция Figure 7 … 14-CPU Sun E6500 (7 boards × 2 × UltraSPARC 400 MHz, Solaris 9); точки правее x=14 — oversubscription.» | ✅ ПОДТВЕРЖДЕНО | Машина и disclaimer корректны. [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 96 | Данные графика (ось X 1..32, markLine x=14, Y до 4000). | ✅ ПОДТВЕРЖДЕНО (иллюстративно) | markLine «14 CPU (физический предел E6500)» + disclaimer корректны. [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 97 | «Главный вывод: lock-free сам по себе не даёт масштабируемости — Treiber без backoff проигрывает MCS-локу уже при четырёх потоках.» | ✅ ПОДТВЕРЖДЕНО | [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 98 | «Хорошее масштабирование — только с backoff или элиминацией (elimination-backoff stack масштабируется почти линейно, обычный Treiber — на плато; числа Figure 7 поточечно не воспроизведены).» | ✅ ПОДТВЕРЖДЕНО | Качественно соответствует абстракту. [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 99 | «Урок: «lock-free» — это про прогресс, а не про скорость.» | ✅ ПОДТВЕРЖДЕНО | [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |

### Раздел 9: Сравнение с альтернативами (строки 279–287)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 100 | Строка Coarse mutex (блокирующий read/write, вся структура встаёт). | ✅ ПОДТВЕРЖДЕНО | [Herlihy & Shavit](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |
| 101 | Строка Fine-grained locking (latch coupling / hand-over-hand, блокировка локализована). | ✅ ПОДТВЕРЖДЕНО | [Graefe, B-tree locking survey](https://15721.courses.cs.cmu.edu/spring2016/papers/a16-graefe.pdf) |
| 102 | Строка Lock-free (неблокирующий, прогресс системы сохраняется, HP/EBR/GC). | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 103 | Строка Wait-free (гарантированно O(1) шагов, прогресс каждого потока, hard real-time). | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 104 | Строка RCU (read бесплатный/ноль atomic, copy+swap+grace period). | ✅ ПОДТВЕРЖДЕНО | [What is RCU? (kernel.org)](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html) |

### Раздел 10: Подводные камни и анти-паттерны (строки 289–294)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 105 | ««Lock-free всегда быстрее» — миф. При высоком контеншене чистый CAS-цикл проигрывает MCS-локу.» | ✅ ПОДТВЕРЖДЕНО | [Hendler et al. SPAA 2004](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 106 | «Live-lock без helping. Два потока, откатывающие CAS и сразу повторяющие, бесконечно мешают; решения: backoff, elimination, helping.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: Non-blocking algorithm](https://en.wikipedia.org/wiki/Non-blocking_algorithm) |
| 107 | «ABA без HP/EBR/tag. В не-GC-среде узел переиспользуется → CAS на старом адресе проходит ошибочно.» | ✅ ПОДТВЕРЖДЕНО | [Wikipedia: ABA problem](https://en.wikipedia.org/wiki/ABA_problem) |
| 108 | «Verification. SPIN (LTL), CDSChecker (Norris–Demsky 2013, exhaustive под C11), GenMC (Kokologiannakis), jcstress (OpenJDK).» | ✅ ПОДТВЕРЖДЕНО | [SPIN](https://spinroot.com/); [CDSChecker OOPSLA 2013, DOI 10.1145/2509136.2509514](https://doi.org/10.1145/2509136.2509514); [GenMC](https://plv.mpi-sws.org/genmc/); [jcstress](https://openjdk.org/projects/code-tools/jcstress/) |

### Раздел 11: Список литературы (строки 296–342)

| # | Ссылка | Вердикт | Доказательство / пояснение + URL |
|---|---|---|---|
| 109 | [1] Herlihy, Shavit, Luchangco, Spear (2020) *Art of Multiprocessor Programming* 2nd ed. | ✅ ПОДТВЕРЖДЕНО | [Elsevier](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1) |
| 110 | [2] Treiber 1986 RJ 5118 | ✅ ПОДТВЕРЖДЕНО | [IBM RJ 5118 PDF](https://dominoweb.draco.res.ibm.com/reports/rj5118.pdf) |
| 111 | [3] Michael 2004 *Hazard Pointers* IEEE TPDS 15(6) 491–504, DOI 10.1109/TPDS.2004.8 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1109/TPDS.2004.8](https://doi.org/10.1109/TPDS.2004.8) |
| 112 | [4] Michael & Scott 1996 PODC '96 267–275, DOI 10.1145/248052.248106 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/248052.248106](https://doi.org/10.1145/248052.248106) |
| 113 | [5] Fraser 2004 *Practical Lock-Freedom* UCAM-CL-TR-579 | ✅ ПОДТВЕРЖДЕНО | [UCAM-CL-TR-579 PDF](https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-579.pdf) |
| 114 | [6] Harris 2001 DISC '01 LNCS 2180 300–314, DOI 10.1007/3-540-45414-4_21 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1007/3-540-45414-4_21](https://doi.org/10.1007/3-540-45414-4_21) |
| 115 | [7] Levandoski, Lomet, Sengupta 2013 ICDE 302–313, DOI 10.1109/ICDE.2013.6544834 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1109/ICDE.2013.6544834](https://doi.org/10.1109/ICDE.2013.6544834) |
| 116 | [8] Shalev & Shavit 2006 JACM 53(3) 379–405, DOI 10.1145/1147954.1147958 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1147954.1147958](https://doi.org/10.1145/1147954.1147958) |
| 117 | [9] Michael 2002 SPAA '02 73–82, DOI 10.1145/564870.564881 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/564870.564881](https://doi.org/10.1145/564870.564881) |
| 118 | [10] Arm Ltd. *Learn the architecture: LSE* | ✅ ПОДТВЕРЖДЕНО | [learn.arm.com/.../lse/intro/](https://learn.arm.com/learning-paths/servers-and-cloud-computing/lse/intro/) |
| 119 | [11] OpenJDK JEP 188 | ✅ ПОДТВЕРЖДЕНО | [openjdk.org/jeps/188](https://openjdk.org/jeps/188) |
| 120 | [12] OpenJDK CLQ source (HEAD JDK 25) | ✅ ПОДТВЕРЖДЕНО | [CLQ.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java) |
| 121 | [13] OpenJDK ConcurrentSkipListMap source — «реализация на основе работ Fraser и Harris» | ⚠️ НЕТОЧНО | Исходник CSLM.java атрибутирует базовые списки **HM-алгоритму (Harris + Maged Michael)**; Fraser — среди related-thesis-работ. Описание «Fraser и Harris» опускает Michael и преувеличивает Fraser. Зеркало row 69. [CSLM.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentSkipListMap.java) |
| 122 | [14] McKenney et al. *What is RCU, Fundamentally?* | ✅ ПОДТВЕРЖДЕНО | [kernel.org/doc/html/latest/RCU/whatisRCU.html](https://www.kernel.org/doc/html/latest/RCU/whatisRCU.html) |
| 123 | [15] Thompson et al. (2011) *Disruptor* LMAX Technical Paper — «сами авторы фиксируют, что Disruptor lock-free, но не wait-free: producer'ы конкурируют за позиции через CAS» | ⚠️ НЕТОЧНО | Paper использует «lock-free» и описывает CAS multi-producer, но фразы «wait-free»/«не wait-free» **не содержит** — атрибуция этой формулировки авторам неверна. (При этом «через CAS» в ref [15] — верно, в отличие от тела статьи строки 202, где сказано «FAA».) [Disruptor 1.0 PDF](https://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf) |
| 124 | [16] Kogan & Petrank 2011 PPoPP '11 223–234, DOI 10.1145/1941553.1941585 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1941553.1941585](https://doi.org/10.1145/1941553.1941585) |
| 125 | [17] Manson, Pugh, Adve 2005 POPL '05 378–391, DOI 10.1145/1040305.1040336 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1040305.1040336](https://doi.org/10.1145/1040305.1040336) |
| 126 | [18] Reeves 1997 *What Really Happened on Mars?* | ✅ ПОДТВЕРЖДЕНО | [cs.unc.edu/.../mars_pathfinder_long_version.html](https://www.cs.unc.edu/~anderson/teach/comp790/papers/mars_pathfinder_long_version.html) |
| 127 | [19] PostgreSQL `xlog.c` — «шардирование WALInsertLock на 8 слотов **с lock-free reservation позиции через FAA**» | ⚠️ НЕТОЧНО | `NUM_XLOGINSERT_LOCKS=8` и массив `WALInsertLocks` — верно. Но «reservation … через FAA» неверно: `ReserveXLogInsertLocation()` использует спинлок `insertpos_lck`, не FAA. Зеркало row 84 (❌). [xlog.c](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c) |
| 128 | [20] Hendler, Shavit, Yerushalmi 2004 SPAA '04 206–215, DOI 10.1145/1007912.1007944 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/1007912.1007944](https://dl.acm.org/doi/10.1145/1007912.1007944) |
| 129 | [21] Mellor-Crummey & Scott 1991 ACM TOCS 9(1) 21–65, DOI 10.1145/103727.103729 | ✅ ПОДТВЕРЖДЕНО | [DOI 10.1145/103727.103729](https://doi.org/10.1145/103727.103729) |
| 130 | [22] NetBSD `pserialize(9)` | ✅ ПОДТВЕРЖДЕНО | [man.netbsd.org/pserialize.9](https://man.netbsd.org/pserialize.9) |
| 131 | [23] OpenJDK `FutureTask.java` — WaitNode «Treiber stack» | ✅ ПОДТВЕРЖДЕНО | Комментарий и `WAITERS.weakCompareAndSet` подтверждены. [FutureTask.java](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/FutureTask.java) |

---

## Проблемные пункты, требующие правок (enumerated)

### ❌ Фактическая ошибка (1)

**[❌-1] Строка 202 (тело §7) + ref [19] (строка 334): PostgreSQL WAL «через FAA».**
- **Неверное утверждение:** «`WALInsertLock` … шардирован на 8 слотов с lock-free reservation позиции **через FAA**».
- **Корректный факт:** Шардирование на 8 слотов верно (`#define NUM_XLOGINSERT_LOCKS 8`, массив `WALInsertLocks`). Но резервация позиции WAL выполняется **под спинлоком** `insertpos_lck`, а не fetch-and-add: в `ReserveXLogInsertLocation()` — `SpinLockAcquire(&Insert->insertpos_lck); startbytepos = Insert->CurrBytePos; Insert->CurrBytePos = endbytepos; SpinLockRelease(...)`. Это spinlock-protected RMW, не атомарный FAA.
- **Исправление:** убрать «через FAA»; написать «шардирован на 8 `WALInsertLocks`; позиция в WAL резервируется под спинлоком `insertpos_lck` (быстрая критическая секция), сами вставки идут параллельно под разными слотами». Аналогично поправить описание ref [19].
- **Источник:** https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c (функция `ReserveXLogInsertLocation`).
- **Примечание:** эту ошибку пропустили все предыдущие пассы — fix pass #1.5 (2026-05-18) лишь переуказал URL с `lwlock.c` на `xlog.c`, не проверив сам механизм.

### ⚠️ Неточности (6)

**[⚠-1] Строка 202 (тело §7): Disruptor «с FAA-резервацией позиций».**
- **Неверно:** «кольцевой буфер с **FAA-резервацией позиций**».
- **Факт:** Disruptor 1.0 paper: multi-producer claim через **CAS** («a simple CAS operation on the sequence»; «an atomic counter updated using CAS operations in the case of multiple producers»), не FAA. Внутреннее противоречие: тот же абзац ниже говорит «несколько CAS».
- **Исправление:** «кольцевой буфер с CAS-резервацией позиций».
- **Источник:** https://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf

**[⚠-2] Строка 202 (тело §7) + ref [15] (строка 326): «сами авторы … Disruptor … не wait-free».**
- **Неверно:** атрибуция фразы «lock-free, но не wait-free» авторам Disruptor.
- **Факт:** paper содержит термин «lock-free», но фразы «wait-free» / «не wait-free» в нём НЕТ. Качественный вывод (CAS-цикл producer'а может проигрывать неограниченно) верен, но это вывод страницы, а не цитата авторов.
- **Исправление:** «Disruptor — lock-free (paper прямо использует этот термин); wait-free он не является, т.к. при коллизии producer'ы конкурируют через CAS и один может проигрывать неограниченно» — без атрибуции «сами авторы отмечают … не wait-free».
- **Источник:** https://lmax-exchange.github.io/disruptor/files/Disruptor-1.0.pdf

**[⚠-3] Строка 173 (§5) + ref [13] (строка 322): ConcurrentSkipListMap «на основе работ Fraser и Harris».**
- **Неточно:** class-level комментарий CSLM.java атрибутирует базовые списки **HM-алгоритму (Harris + Maged Michael)** («The base lists use a variant of the HM linked ordered set algorithm. See Tim Harris … and Maged Michael …»); Fraser упомянут отдельно среди thesis-работ, «sharing … features», т.е. как related work.
- **Исправление:** «…реализация — концурентный вариант skip-list'ов; базовые списки используют HM-алгоритм (Harris и Maged Michael), Fraser и Sundell упомянуты как смежные работы».
- **Источник:** https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentSkipListMap.java
- **Серьёзность:** низкая (смещение атрибуции, не фактическая ошибка).

**[⚠-4] Строка 40 + строка 25 (§2): «javadoc явно говорит» про CLQ.**
- **Неточно:** ссылка на M&S — это **class-level комментарий в исходнике** CLQ.java, а не публичный API-javadoc.
- **Исправление:** «исходный код / class-level комментарий CLQ.java явно ссылается…» (как уже сделано для CSLM). Перенос из прошлого лога (#B), всё ещё открыт на уровне предложения.
- **Источник:** https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/concurrent/ConcurrentLinkedQueue.java
- **Серьёзность:** низкая.

**[⚠-5] Строка 111 (§4): «DCAS на одном слове 128 бит».**
- **Неточно (терминология):** `CMPXCHG16B` — это **DWCAS** (double-width CAS), а **DCAS** = double-compare-and-swap по двум независимым адресам.
- **Исправление:** «DWCAS на одном 128-битном слове».
- **Источник:** https://en.wikipedia.org/wiki/Double_compare-and-swap
- **Серьёзность:** косметика (механизм корректен). Перенос из прошлого лога (#C).

**[⚠-6] Строка 31 (§2): «Свойства monotonic strict total order».**
- **Неточно (формулировка):** импликации wait-free⟹lock-free⟹obstruction-free корректны, но ярлык «monotonic strict total order» нестандартен и сбивает с толку для цепочки строгих импликаций / вложенности классов.
- **Исправление:** заменить на «строгая иерархия вложенности классов» или просто «цепочка строгих импликаций». Содержание правильное.
- **Источник:** https://en.wikipedia.org/wiki/Non-blocking_algorithm
- **Серьёзность:** низкая (стилистика).

### 💬 Не факт / метатекст (9): строки 12 (2 шт.), 17, 27 («идея»), 84, 88, 206, + субъективные вводные. Не требуют правок.

---

## Что подтверждено заново против первоисточников (highlights)

- **Иерархия прогресса** (wait-free ⟹ lock-free ⟹ obstruction-free), точные определения starvation/livelock/изоляции — дословно совпадают с источником.
- **Treiber RJ 5118 (1986)**, lock-free LIFO через CAS, ABA-уязвимость в не-GC — подтверждено.
- **Michael–Scott PODC 1996**: dummy-узел, двойной CAS (link + helping swing Tail), CAS Head на dequeue, counted-pointers в не-GC, linearizable/lock-free — подтверждено **по тексту PDF**.
- **Harris DISC 2001**: двухшаговое logical-mark + physical-unlink удаление — подтверждено.
- **Hazard Pointers (Maged Michael 2004, IEEE TPDS)** — автор/год/венью/механизм подтверждены.
- **FutureTask.WaitNode** — комментарий «Treiber stack» и `WAITERS.weakCompareAndSet` подтверждены дословно.
- **ARMv8.1 LSE** (CAS/CASA/CASL/CASAL; до LSE — LDXR/STXR) — подтверждено.
- **memory_order** acquire/release/seq_cst/relaxed — семантика на странице точна.
- **Mars Pathfinder** — ASI/MET держит IPC-семафор внутри `select()`, blocked bc_dist, medium-priority затягивают, fix priority inheritance — дословно по Reeves 1997.
- **PostgreSQL** `NUM_XLOGINSERT_LOCKS=8` — подтверждено; **но** механизм резервации — спинлок, не FAA (см. ❌-1).
- **Disruptor** — CAS (не FAA) для multi-producer; нет фразы «wait-free» в paper (см. ⚠-1, ⚠-2).

---

## История пассов

- **2026-05-08 / 2026-05-18 / 2026-05-23** — предыдущий лог (заменён этим файлом). Все ранее закрытые правки (#1.1–#1.5, #2.1–#2.4) перепроверены и подтверждены как держащиеся: Pathfinder wording, CSLM source-comment repoint, NetBSD pserialize, contention-chart disclaimer + markLine, FutureTask.WaitNode атрибуция, открытое хеширование (separate chaining), Cliff Click без «осторожностью», elimination-backoff качественная формулировка.
- **2026-06-11 (этот пасс)** — full from-scratch re-audit. Новая находка ❌-1 (PostgreSQL FAA, пропущена прошлыми пассами); подтверждены и подняты до ⚠️ ранее «out of scope» пункты #H (Disruptor FAA/CAS + «не wait-free» атрибуция → ⚠-1, ⚠-2). Новая ⚠-3 (CSLM HM-атрибуция). Перенесены ⚠-4 (#B), ⚠-5 (#C). Новая стилистическая ⚠-6.

**Итог: NEEDS-FIXES — 1 ❌ + 6 ⚠️.** Контентная страница не редактировалась (per задание). Правки — за владельцем / следующим fix pass.

## Разрешение замечаний (2026-06-11, orchestrator fix pass)

Все 7 пунктов исправлены в `concurrent-data-structures.mdx`:

- **❌-1 (line 202 + ref [19], PostgreSQL WAL «через FAA»)** — FIXED. Проза и ref [19] переписаны: позиция вставки в WAL резервируется в `ReserveXLogInsertLocation()` под спинлоком `insertpos_lck` (RMW над `Insert->CurrBytePos`), а не атомарным FAA. Шардирование на 8 слотов (`NUM_XLOGINSERT_LOCKS`) сохранено как верное. Источник: [xlog.c](https://github.com/postgres/postgres/blob/master/src/backend/access/transam/xlog.c).
- **⚠-1 (line 202, Disruptor «FAA-резервацией»)** — FIXED. Заменено на «резервируют позиции через CAS» (соответствует Disruptor-1.0 PDF и согласовано с последующей фразой про «несколько CAS»).
- **⚠-2 (line 202 + ref [15], «сами авторы … не wait-free»)** — FIXED. Убрана атрибуция формулировки «не wait-free» авторам; теперь это явно подано как вывод из механики CAS, а не дословная цитата (статья называет алгоритм lock-free).
- **⚠-3 (line 173 + ref [13], CSLM «на основе Fraser и Harris»)** — FIXED. Уточнено: базовые списки — по алгоритму **HM (Harris–Maged Michael)**; Fraser — родственная работа (related), как и сказано в class-комментарии `ConcurrentSkipListMap`.
- **⚠-4 (line 40, «javadoc явно говорит» про CLQ)** — FIXED. Заменено на «комментарий в исходном коде класса (Doug Lea)».
- **⚠-5 (line 111, «DCAS на одном слове 128 бит»)** — FIXED. Уточнено: `CMPXCHG16B` — это **double-width CAS (DWCAS)** над одним 128-битным словом, не путать с DCAS над двумя независимыми адресами.
- **⚠-6 (line 31, «monotonic strict total order»)** — FIXED. Нестандартный ярлык заменён на «строгую иерархию по силе гарантии».

Сборка зелёная (Node 20). Submodule commit `docs(concurrent): fix WAL reservation (spinlock not FAA), Disruptor CAS, CSLM/DWCAS precision`, bump родителя, деплой run `27316701214` — GREEN. **Обновлённый вердикт: APPROVED — 0 ❌, 0 ⚠️ (все замечания устранены).**
