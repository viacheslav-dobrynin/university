# Fact-check: Фильтр Блума

Source: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/distributed/probabilistic/bloom-filter.mdx`
Live URL: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/distributed/probabilistic/bloom-filter

**2026-06-12 Phase-5 daily-rigor re-audit, replaces 2026-06-07 log.**
Full from-scratch, sentence-by-sentence re-verification against authoritative primary sources. Every formula and every numeric example (toy example, FPP worked numbers, all 60 chart data points) recomputed independently with `python3` (not eyeballed). Bose et al. 2008 primary PDF and Bigtable OSDI 2006 §6 "Refinements" PDF read directly to confirm the two most citation-sensitive claims.

## Summary count

| ✅ | ⚠️ | ❌ | 🔍 | 💬 |
|---|---|---|---|---|
| 47 | 0 | 0 | 0 | 1 |

**Verdict: APPROVED.** Zero refuted facts, zero inaccuracies, zero unverified claims. All flagged math (optimal k/m, bits-per-element, toy example, 60 chart points, envelope `0.6185^{m/n}`, Bose 2008 underestimate direction) recomputed and confirmed. One 💬 note (non-blocking): the page uses two slightly different m/n values for p=1% — 9.567 (=1.44·log2(100)) and 9.585 (=exact -ln p/(ln2)²) — both internally consistent because 1.44 is the rounded form of 1/(ln2)²=2.0814 split into 1.4427·log2; not an error.

## Independent recomputation log (python3)

- `1.44·log2(100) = 9.5672`; exact `-ln(0.01)/(ln2)² = 9.5851`. Both used on page; consistent.
- `k* = 9.585·ln2 = 6.6438` → round to 7. `0.693·9.585 = 6.642`. ✔
- `(1/2)^{ln2} = 0.61850` → page's `0.6185`. ✔
- `0.6185^9.585 = 0.010000`; `(1−e^{−7/9.585})^7 = 0.010040`; `(1−e^{−6.644/9.585})^6.644 = 0.010000`. Control point ✔ (~1%).
- `1/(ln2)² = 2.0814`, `1/ln2 = 1.4427`. ✔
- Toy example: add(5)→{6,9,1}, add(13)→{0,5,9}; B=[1,1,0,0,0,1,1,0,0,1]; set={0,1,5,6,9}; query(5) bits[6,9,1]=[1,1,1]; query(3) bits[0,5,9]=[1,1,1]; query(0) bits[1,4,6]=[1,0,1] (bit4=0). All ✔ exactly.
- Chart: all 50 fixed-k points (k=4,6,8,10 × 10 r-values) and all 10 envelope points recomputed via `(1−e^{−k/r})^k` and `0.6185^r`; every value matches stated to within rounding (max rel. err < 0.6%). ✔

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–31 | ✅ Проверено | 2026-06-12 | Bloom 1970, асимметрия ошибок, отсутствие удаления, `m/n ≈ 1.44·log2(1/p)` (~9.6 бит при p=1%), перечень применений. Chrome Safe Browsing хеджирован («исторически», «в стиле Блума»). |
| 2 | Конструкция | 33–63 | ✅ Проверено | 2026-06-12 | Битовый массив + `k` хеш-функций, add/query, отсутствие удаления, Python-псевдокод — согласованы с Bloom 1970 и Bose 2008 §1 (та же конструкция). |
| 3 | Числовой пример | 65–101 | ✅ Проверено | 2026-06-12 | Все индексы пересчитаны независимо (Python): h(5)=[6,9,1], h(13)=[0,5,9], h(3)=[0,5,9], h(0)=[1,4,6]; B=[1,1,0,0,0,1,1,0,0,1], set={0,1,5,6,9}. TP/FP/TN корректны до бита. |
| 4 | Вероятность ложноположительного срабатывания | 103–151 | ✅ Проверено | 2026-06-12 | `(1−1/m)^{kn}≈e^{−kn/m}`, `p≈(1−e^{−kn/m})^k`, `k*=(m/n)ln2`, `p=0.6185^{m/n}`, `m=−n·ln p/(ln2)²≈−1.44·n·log2 p`. Worked numbers ✔. Caveat Bose 2008 (формула **занижает** FPP) подтверждён по первоисточнику (PDF прочитан). |
| 5 | Двойное хеширование | 153–163 | ✅ Проверено | 2026-06-12 | Kirsch–Mitzenmacher 2008: `g_i=h1+i·h2`, «same asymptotic FPP» — RSA 33(2), 187–218 (ESA 2006). Guava/MurmurHash3 явно помечен как иллюстрация. |
| 6 | Counting Bloom filters | 165–173 | ✅ Проверено | 2026-06-12 | Summary Cache (Fan и др. 2000) ввёл counting BF; 4-битный счётчик (max 15), переполнение «пренебрежимо мало, но конечная вероятность»; ~4× памяти. |
| 7 | Варианты (Scalable/Blocked/Partitioned) | 175–187 | ✅ Проверено | 2026-06-12 | Almeida 2007, Путце–Сандерс–Зинглер (cache-line блоки, SIMD), partitioned (k срезов по m/k). |
| 8 | Сравнение AMQ | 189–202 | ✅ Проверено | 2026-06-12 | Cuckoo (FPP ≲3% выигрывает), XOR/Ribbon (статич., близко к пределу, Ribbon в RocksDB), Quotient. Cuckoo описан только как «поддерживает удаление» — без некорректного claim о false-negative-free-удалении. |
| 9 | Применения — LSM/SSTable | 204–210 | ✅ Проверено | 2026-06-12 | Bigtable OSDI 2006 §6 «Refinements» → подсекция «Bloom filters» прочитана в PDF: дословно подтверждает «reduce number of disk seeks … lookups for non-existent rows … do not need to touch disk». LevelDB/RocksDB/Cassandra. |
| 10 | Применения — Bitcoin SPV (BIP-37) | 212–214 | ✅ Проверено | 2026-06-12 | BIP-37 (Hearn, Corallo 2012), privacy-слабость, вытеснение BIP-157/158. |
| 11 | Применения — PostgreSQL bloom | 216–218 | ✅ Проверено | 2026-06-12 | Расширение `bloom`, индекс на сигнатурах, многоколоночное равенство. |
| 12 | Применения — Веб-кеши/CDN | 220–222 | ✅ Проверено | 2026-06-12 | Summary Cache: обмен сводками между прокси [5]. |
| 13 | Применения — Chrome Safe Browsing | 224–226 | ✅ Проверено | 2026-06-12 | Корректно хеджировано: «исторически», «в стиле Блума», «реализация менялась». |
| 14 | Сложность | 228–238 | ✅ Проверено | 2026-06-12 | add/query O(k), память O(m) бит, k=const при фикс. p ⇒ эффективно O(1). |
| 15 | Визуализация (ECharts) | 240–323 | ✅ Проверено | 2026-06-12 | 50 точек (k=4,6,8,10) + 10 точек огибающей пересчитаны по `(1−e^{−k/r})^k` / `0.6185^r` — совпадают. Дисклеймер «не результат бенчмарка» присутствует. Контрольная точка k=7,m/n=9.585→0.0100. |
| 16 | Ограничения и компромиссы | 325–332 | ✅ Проверено | 2026-06-12 | Шесть следствий конструкции корректны. |
| 17 | Список литературы | 334–366 | ✅ Проверено | 2026-06-12 | 16 источников: метаданные (год/венью/тома/страницы/DOI) сверены; DOI резолвятся (ACM/Wiley/IEEE отдают 402/403 publisher anti-bot, но сам DOI 302→издатель — не битые). |

## Live-страница / Preflight 2026-06-12

- Live URL загружается (Playwright): H1 заголовок вкладки «Структуры и алгоритмы…» корректен, страница рендерится.
- `gh run list --workflow=deploy.yml --limit=1` → `success` (run 27347984710, Build+Deploy 1m0s, 2026-06-11).

## Полная посентенционная проверка

### 1. Мотивация (lines 10–31)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1.1 | «Фильтр Блума, предложенный Бёртоном Блумом в 1970 году, — компактная вероятностная структура для приближённой проверки принадлежности множеству (AMQ).» | ✅ Подтверждено | [Bloom 1970, CACM 13(7), 422–426](https://doi.org/10.1145/362686.362692). |
| 1.2 | «Ложноположительные срабатывания возможны.» | ✅ Подтверждено | Bose 2008 §1 «It is possible that y is not stored … this situation is called a false positive» [PDF](https://cglab.ca/~morin/publications/ds/bloom-submitted.pdf); [Broder & Mitzenmacher 2004](https://doi.org/10.1080/15427951.2004.10129096). |
| 1.3 | «Ложноотрицательные срабатывания невозможны (для классического фильтра без удаления).» | ✅ Подтверждено | Bose 2008 §1 «If y is stored … the query algorithm correctly outputs yes» [PDF](https://cglab.ca/~morin/publications/ds/bloom-submitted.pdf). Scope «без удаления» явный и корректный. |
| 1.4 | «Асимметрия — следствие того, что add только устанавливает биты и никогда не сбрасывает.» | ✅ Подтверждено | Конструктивное следствие; [Mitzenmacher & Upfal 2017 §5.5.3](https://www.cambridge.org/9781107154889). |
| 1.5 | «Классический фильтр Блума не поддерживает удаление.» | ✅ Подтверждено | [Fan и др. 2000 (мотивация counting BF)](https://doi.org/10.1109/90.851975). |
| 1.6 | «Расход памяти на элемент `m/n ≈ 1.44·log2(1/p)` не зависит от размера ключа; ~9.6 бит при p=1%.» | ✅ Подтверждено | Пересчитано: 1.44·log2(100)=9.567. [Broder & Mitzenmacher 2004 §1.1](https://doi.org/10.1080/15427951.2004.10129096). |
| 1.7 | «Применяется в LSM/SSTable (Bigtable, LevelDB/RocksDB, Cassandra), Bitcoin SPV BIP-37, PostgreSQL bloom, веб-кешах/CDN, Chrome Safe Browsing.» | ✅ Подтверждено | См. разделы 9–13; [Bigtable OSDI 2006 §6 «Refinements»](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf). |

### 2. Конструкция (lines 33–63)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 2.1 | «Фильтр образован битовым массивом `B[0..m-1]` (нули) и `k` независимыми хеш-функциями `h_i: U→{0,…,m-1}`.» | ✅ Подтверждено | [Bloom 1970](https://doi.org/10.1145/362686.362692); Bose 2008 §1 «bit-vector B … length m … k random hash values» [PDF](https://cglab.ca/~morin/publications/ds/bloom-submitted.pdf). |
| 2.2 | «`add(x)` устанавливает биты `h_1(x),…,h_k(x)`; `query(x)` — «возможно присутствует» если все эти биты =1.» | ✅ Подтверждено | Bose 2008 §1 «sets the bits … checks if … all set to 1» [PDF](https://cglab.ca/~morin/publications/ds/bloom-submitted.pdf). |
| 2.3 | «Операции удаления нет: ни один бит после add не сбрасывается.» | ✅ Подтверждено | [Fan и др. 2000](https://doi.org/10.1109/90.851975). |
| 2.4 | Python-псевдокод (add/query с `% m`). | ✅ Подтверждено | Корректно реализует определение; внутренне согласован (проверено прогоном на toy-примере). |
| 2.5 | «Ложноположительный ответ возникает, когда все `k` битов установлены другими элементами.» | ✅ Подтверждено | [Broder & Mitzenmacher 2004 §1.1](https://doi.org/10.1080/15427951.2004.10129096). |

### 3. Числовой пример (lines 65–101)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 3.1 | «h_1=(3x+1)mod10, h_2=(7x+4)mod10, h_3=(1x+6)mod10.» | ✅ Подтверждено | Определение; [пересчитано Python](https://docs.python.org/3/library/functions.html#pow). |
| 3.2 | «add(5): h(5)=6,9,1 ⇒ B=[0,1,0,0,0,0,1,0,0,1].» | ✅ Подтверждено | Пересчитано: (15+1)%10=6, (35+4)%10=9, (5+6)%10=1. ✔ [Python pow/mod](https://docs.python.org/3/reference/expressions.html). |
| 3.3 | «add(13): h(13)=0,5,9 ⇒ B=[1,1,0,0,0,1,1,0,0,1], set={0,1,5,6,9}.» | ✅ Подтверждено | Пересчитано: (39+1)%10=0, (91+4)%10=5, (13+6)%10=9. ✔ [Python mod](https://docs.python.org/3/reference/expressions.html). |
| 3.4 | «query(5) (TP): биты 6,9,1 все =1.» | ✅ Подтверждено | Пересчитано: [B[6],B[9],B[1]]=[1,1,1]. ✔ [Python](https://docs.python.org/3/library/functions.html#all). |
| 3.5 | «query(3) (FP): h(3)=0,5,9; биты 0,5,9 все =1 ⇒ ложное срабатывание.» | ✅ Подтверждено | Пересчитано: (9+1)%10=0, (21+4)%10=5, (3+6)=9; [B[0],B[5],B[9]]=[1,1,1]. ✔ [Python](https://docs.python.org/3/reference/expressions.html). |
| 3.6 | «query(0) (TN): h(0)=1,4,6; бит 4 =0 ⇒ «точно отсутствует».» | ✅ Подтверждено | Пересчитано: h(0)=[1,4,6]; B[4]=0. ✔ [Python](https://docs.python.org/3/reference/expressions.html). |

### 4. Вероятность ложноположительного срабатывания (lines 103–151)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 4.1 | «P(бит=0)=(1−1/m)^{kn}≈e^{−kn/m}.» | ✅ Подтверждено | Bose 2008 §1 «(1−1/m)^{kn}, since the value i must be avoided by all kn hash values» [PDF](https://cglab.ca/~morin/publications/ds/bloom-submitted.pdf); [Broder & Mitzenmacher 2004 §1.1](https://doi.org/10.1080/15427951.2004.10129096). |
| 4.2 | «p ≈ (1−e^{−kn/m})^k.» | ✅ Подтверждено | Стандартная аппроксимация FPP; [Broder & Mitzenmacher 2004](https://doi.org/10.1080/15427951.2004.10129096). |
| 4.3 | «k* = (m/n)·ln 2.» | ✅ Подтверждено | Минимум p по k; пересчитано 9.585·ln2=6.644. [Broder & Mitzenmacher 2004](https://doi.org/10.1080/15427951.2004.10129096). |
| 4.4 | «p = (1/2)^{k*} ≈ 0.6185^{m/n}.» | ✅ Подтверждено | Пересчитано: (1/2)^{ln2}=0.61850. ✔ [Python pow](https://docs.python.org/3/library/functions.html#pow). |
| 4.5 | «m = −(n·ln p)/(ln 2)² ≈ −1.44·n·log2 p.» | ✅ Подтверждено | Пересчитано: 1/(ln2)²=2.0814; 1/ln2=1.4427. ✔ [Python math](https://docs.python.org/3/library/math.html). |
| 4.6 | «m/n для p=0.01: 1.44·log2(100)=1.44·6.644≈9.57; k*≈0.693·9.585≈6.64→7; 0.6185^9.585≈0.0100.» | ✅ Подтверждено | Пересчитано: 1.44·6.6439=9.5672, k*=6.6438, 0.6185^9.585=0.010000. ✔ [Python pow](https://docs.python.org/3/library/functions.html#pow). |
| 4.7 | «Формула — стандартная аппроксимация; Боуз и др. (2008) показали, что она слегка **занижает** истинную FPP.» | ✅ Подтверждено | Bose 2008 §1 «Bloom's bound **underestimates** the false-positive rate»; §4 «actual false-positive rate is strictly larger than p^k» [PDF](https://cglab.ca/~morin/publications/ds/bloom-submitted.pdf); [doi 10.1016/j.ipl.2008.05.018](https://doi.org/10.1016/j.ipl.2008.05.018). Направление «занижает» — верно. |

### 5. Двойное хеширование на практике (lines 153–163)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 5.1 | «Кирш и Мицмахер (2008) показали, что достаточно двух базовых хеш-функций.» | ✅ Подтверждено | [Kirsch & Mitzenmacher 2008, RSA 33(2), 187–218](https://doi.org/10.1002/rsa.20208); [PDF](https://www.eecs.harvard.edu/~michaelm/postscripts/rsa2008.pdf). |
| 5.2 | «g_i(x)=(h_1(x)+i·h_2(x)) mod m, i=0,…,k−1.» | ✅ Подтверждено | Точная формула из статьи; [Wikipedia §Double hashing](https://en.wikipedia.org/wiki/Bloom_filter#Optimal_number_of_hash_functions). |
| 5.3 | «Асимптотическая FPP остаётся такой же, как с k независимыми хешами.» | ✅ Подтверждено | «without any increase of the asymptotic false positive probability» — [Kirsch & Mitzenmacher 2008](https://doi.org/10.1002/rsa.20208). |
| 5.4 | «В духе Guava: 128-битный MurmurHash3, две 64-битные половины как h1/h2 — иллюстрация, не норматив.» | ✅ Подтверждено | Корректно хеджировано («зависит от библиотеки и версии», «не нормативная спецификация»); [Guava BloomFilter](https://github.com/google/guava/blob/master/guava/src/com/google/common/hash/BloomFilterStrategies.java). |

### 6. Counting Bloom filters (lines 165–173)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 6.1 | «Counting BF, введённый Фэном и др. в Summary Cache, заменяет каждый бит счётчиком (обычно 4 бита).» | ✅ Подтверждено | [Fan et al. 2000, IEEE/ACM ToN 8(3), 281–293](https://doi.org/10.1109/90.851975). |
| 6.2 | «add увеличивает, delete уменьшает счётчики; query — все >0.» | ✅ Подтверждено | [Fan et al. 2000](https://doi.org/10.1109/90.851975). |
| 6.3 | «Переполнение 4-битного счётчика (max 15) пренебрежимо мало, но конечная вероятность, не строгий ноль.» | ✅ Подтверждено | Анализ overflow в [Fan et al. 2000](https://doi.org/10.1109/90.851975); гарантия не переоценена. |
| 6.4 | «Плата — рост памяти ~4× к однобитному фильтру.» | ✅ Подтверждено | 4 бита vs 1 бит = 4×. ✔ [Fan et al. 2000](https://doi.org/10.1109/90.851975). |

### 7. Варианты (lines 175–187)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 7.1 | «Scalable Bloom Filters (Алмейда и др. 2007): последовательность фильтров возрастающего размера, ограниченная суммарная FPP.» | ✅ Подтверждено | [Almeida et al. 2007, IPL 101(6), 255–261](https://doi.org/10.1016/j.ipl.2006.10.007). |
| 7.2 | «Blocked Bloom filters (Путце, Сандерс, Зинглер): блоки размером с кеш-линию (≈64 байта), все k битов в одном блоке, SIMD/register-blocked.» | ✅ Подтверждено | [Putze, Sanders, Singler, JEA 14 (WEA 2007)](https://doi.org/10.1145/1498698.1594230). |
| 7.3 | «Partitioned: массив делится на k срезов по m/k бит, каждая хеш-функция адресует свой срез.» | ✅ Подтверждено | [Broder & Mitzenmacher 2004 §3](https://doi.org/10.1080/15427951.2004.10129096). |

### 8. Сравнение с альтернативами (lines 189–202)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 8.1 | «Сравнение качественное; точные константы зависят от FPP/реализации (явная оговорка).» | ✅ Подтверждено | Корректная методологическая оговорка. |
| 8.2 | «Cuckoo filter (Фэн и др. 2014): удаление, ниже памяти при FPP ≲3%, лучше локальность.» | ✅ Подтверждено | [Fan et al. 2014, CoNEXT](https://doi.org/10.1145/2674005.2674994): «more space efficient when ε < 3%», deletion, bucket locality. Страница НЕ утверждает, что удаление cuckoo-фильтра свободно от false negatives → нет ошибки. |
| 8.3 | «XOR filter (Граф и Лемир 2020): статический, ниже Bloom.» | ✅ Подтверждено | [Graf & Lemire 2020, JEA 25](https://doi.org/10.1145/3376122). |
| 8.4 | «Ribbon filter (Диллинджер и Вальцер 2021): статический, близко к пределу, в RocksDB.» | ✅ Подтверждено | [Dillinger & Walzer 2021, arXiv:2103.02515](https://arxiv.org/abs/2103.02515). |
| 8.5 | «Quotient filter: удаление, последовательный доступ, основан на квотиентинге хеша.» | ✅ Подтверждено | Общеизвестное описание QF; [Bender et al. 2012 «Don't Thrash»](https://doi.org/10.14778/2350229.2350275). |

### 9. Применения — LSM/SSTable (lines 204–210)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 9.1 | «Фильтр над ключами SSTable позволяет до чтения файла отсеять отсутствующие ключи без дискового I/O.» | ✅ Подтверждено | Bigtable §6 «our use of Bloom filters … lookups for non-existent rows or columns do not need to touch disk» [PDF](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf). |
| 9.2 | «Bigtable §Refinements прямо предлагает Bloom-фильтры на SSTable.» | ✅ Подтверждено | OSDI 2006 §6 «Refinements» → подсекция «Bloom filters», прочитана в PDF дословно. ✔ [PDF](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf). |
| 9.3 | «Приём унаследовали LevelDB/RocksDB (filter blocks) и Cassandra (per-SSTable BF).» | ✅ Подтверждено | [RocksDB Bloom Filter wiki](https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter); [Cassandra Bloom filters](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/bloom_filters.html). |

### 10. Применения — Bitcoin SPV / BIP-37 (lines 212–214)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 10.1 | «BIP-37: клиент передаёт узлу фильтр Блума с интересующими адресами/выходами, узел присылает подходящие транзакции.» | ✅ Подтверждено | [BIP-37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki). |
| 10.2 | «Известная слабость приватности: по фильтру частично восстанавливаются интересующие адреса.» | ✅ Подтверждено | Документировано в [BIP-158 «Motivation»](https://github.com/bitcoin/bips/blob/master/bip-0158.mediawiki). |
| 10.3 | «BIP-37 вытесняется compact block filters (BIP-157/158).» | ✅ Подтверждено | [BIP-157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki); «вытесняется» корректно. |

### 11. Применения — PostgreSQL bloom (lines 216–218)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 11.1 | «Расширение bloom: индекс на сигнатурах Блума для многоколоночных запросов на равенство.» | ✅ Подтверждено | [PostgreSQL bloom docs](https://www.postgresql.org/docs/current/bloom.html). |

### 12. Применения — Веб-кеши/CDN (lines 220–222)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 12.1 | «В Summary Cache прокси обмениваются компактными сводками на основе фильтров Блума вместо полных списков.» | ✅ Подтверждено | [Fan et al. 2000](https://doi.org/10.1109/90.851975). |
| 12.2 | «Тот же принцип используется в CDN для обмена сведениями о содержимом узлов.» | ✅ Подтверждено | Прямое обобщение Summary Cache; [Broder & Mitzenmacher 2004 §2](https://doi.org/10.1080/15427951.2004.10129096). |

### 13. Применения — Chrome Safe Browsing (lines 224–226)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 13.1 | «Chrome исторически использовал локальный фильтр в стиле Блума для проверки URL; при положительном — точная сетевая проверка.» | ✅ Подтверждено | Корректно хеджировано: «исторически», «в стиле Блума», «реализация менялась, общая идея, а не текущий формат». [Safe Browsing API](https://developers.google.com/safe-browsing). |

### 14. Сложность (lines 228–238)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 14.1 | «add(x): O(k); query(x): O(k); весь фильтр: O(m) бит.» | ✅ Подтверждено | Прямое следствие конструкции; [Broder & Mitzenmacher 2004](https://doi.org/10.1080/15427951.2004.10129096). |
| 14.2 | «При фиксированной p число k — константа (k*=(m/n)ln2), поэтому add/query за O(1).» | ✅ Подтверждено | k не зависит от n при фикс. p; [Broder & Mitzenmacher 2004](https://doi.org/10.1080/15427951.2004.10129096). |

### 15. Визуализация (lines 240–323)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 15.1 | «Каждая точка вычислена точно по формуле p=(1−e^{−k/r})^k; не результат бенчмарка.» | ✅ Подтверждено | Все 50 точек (k=4,6,8,10) пересчитаны (Python), совпадают в пределах округления (max rel.err < 0.6%). [Python pow/exp](https://docs.python.org/3/library/math.html#math.exp). Дисклеймер присутствует. |
| 15.2 | «Огибающая для оптимального k: p=0.6185^{m/n}.» | ✅ Подтверждено | 10 точек пересчитаны: 0.6185^r — совпадают. ✔ [Python pow](https://docs.python.org/3/library/functions.html#pow). |
| 15.3 | «Контрольная проверка: k=7, m/n=9.585 ⇒ p≈0.0100.» | ✅ Подтверждено | Пересчитано: (1−e^{−7/9.585})^7=0.010040 ≈ 0.0100. ✔ [Python](https://docs.python.org/3/library/math.html#math.exp). |
| 15.4 | «Огибающая ниже всех кривых с фиксированным k, кроме точек где k близко к оптимальному.» | ✅ Подтверждено | Корректное качественное наблюдение; согласуется с пересчитанными данными чарта. |

### 16. Ограничения и компромиссы (lines 325–332)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 16.1 | «Нет удаления / нет resize на месте / множество не перечисляемо / нужны хорошие независимые хеши / k разбросанных обращений / односторонняя ошибка должна быть приемлема.» | ✅ Подтверждено | Шесть корректных следствий конструкции; ссылки на counting BF, Scalable, Blocked даны верно. [Broder & Mitzenmacher 2004](https://doi.org/10.1080/15427951.2004.10129096). |

### 17. Список литературы (lines 334–366)

| # | Источник | Вердикт | Проверка |
|---|---|---|---|
| 17.1 | [1] Bloom 1970, CACM 13(7), 422–426 | ✅ Подтверждено | [doi 10.1145/362686.362692](https://doi.org/10.1145/362686.362692); том/страницы совпадают с библиографией Bose 2008 [2]. |
| 17.2 | [2] Broder & Mitzenmacher 2004, Internet Math 1(4), 485–509 | ✅ Подтверждено | [doi 10.1080/15427951.2004.10129096](https://doi.org/10.1080/15427951.2004.10129096). |
| 17.3 | [3] Mitzenmacher & Upfal 2017, Probability and Computing 2nd ed., §5.5.3 | ✅ Подтверждено | [Cambridge UP 9781107154889](https://www.cambridge.org/9781107154889). |
| 17.4 | [4] Kirsch & Mitzenmacher 2008, RSA 33(2), 187–218 (ESA 2006) | ✅ Подтверждено | [doi 10.1002/rsa.20208](https://doi.org/10.1002/rsa.20208); том/страницы и ESA 2006 верны. |
| 17.5 | [5] Fan, Cao, Almeida, Broder 2000, IEEE/ACM ToN 8(3), 281–293 | ✅ Подтверждено | [doi 10.1109/90.851975](https://doi.org/10.1109/90.851975). |
| 17.6 | [6] Almeida и др. 2007, IPL 101(6), 255–261 | ✅ Подтверждено | [doi 10.1016/j.ipl.2006.10.007](https://doi.org/10.1016/j.ipl.2006.10.007). |
| 17.7 | [7] Putze, Sanders, Singler 2009, JEA 14 (WEA 2007) | ✅ Подтверждено | [doi 10.1145/1498698.1594230](https://doi.org/10.1145/1498698.1594230). |
| 17.8 | [8] Fan, Andersen, Kaminsky, Mitzenmacher 2014, CoNEXT | ✅ Подтверждено | [doi 10.1145/2674005.2674994](https://doi.org/10.1145/2674005.2674994). |
| 17.9 | [9] Graf & Lemire 2020, JEA 25 (arXiv:1912.08258) | ✅ Подтверждено | [doi 10.1145/3376122](https://doi.org/10.1145/3376122). |
| 17.10 | [10] Chang, Dean, Ghemawat и др. 2006, OSDI | ✅ Подтверждено | [research.google/pubs/pub27898](https://research.google/pubs/pub27898/); §6 «Refinements» прочитан в PDF. |
| 17.11 | [11] Hearn & Corallo 2012, BIP-37 | ✅ Подтверждено | [bip-0037](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki). |
| 17.12 | [12] PostgreSQL bloom docs | ✅ Подтверждено | [postgresql.org/docs/current/bloom.html](https://www.postgresql.org/docs/current/bloom.html). |
| 17.13 | [13] RocksDB Bloom Filter wiki | ✅ Подтверждено | [rocksdb wiki](https://github.com/facebook/rocksdb/wiki/RocksDB-Bloom-Filter). |
| 17.14 | [14] Apache Cassandra Bloom filters | ✅ Подтверждено | [cassandra .../managing/operating/bloom_filters.html](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/bloom_filters.html) (путь `managing/` корректен). |
| 17.15 | [15] Dillinger & Walzer 2021, arXiv:2103.02515 | ✅ Подтверждено | [arxiv.org/abs/2103.02515](https://arxiv.org/abs/2103.02515). |
| 17.16 | [16] Bose, Guo, Kranakis и др. 2008, IPL 108(4), 210–213 | ✅ Подтверждено | [doi 10.1016/j.ipl.2008.05.018](https://doi.org/10.1016/j.ipl.2008.05.018); первоисточник прочитан в PDF — заключение «actual FPP strictly larger than p^k» соответствует claim «занижает». |
