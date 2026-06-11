# Fact-check: Дерево Меркла

Source: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/hashing/merkle-tree.mdx`
Live URL: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/hashing/merkle-tree

**2026-06-11 Phase-5 daily-rigor re-audit, replaces 2026-04-26 log.**

Last verification pass (sentence-by-sentence): 2026-06-11 (full from-scratch re-verification against primary sources).

## Сводка (этот проход)

| Вердикт | Кол-во |
|---|---|
| ✅ Подтверждено | 71 |
| ⚠️ Замечание | 2 |
| ❌ Ошибка | 0 |
| 🔍 Требует внимания | 1 |
| 💬 Не факт (связка/иллюстрация/дисклеймер) | 14 |

**ИТОГОВЫЙ ВЕРДИКТ: APPROVED.** Ни одной фактической ошибки (❌) не обнаружено. Все технические утверждения (RFC 6962 префиксы и split-rule, Bitcoin header/SPV, CVE-2012-2459, квантовые границы Grover/BHT, асимптотики O(log n), пересчёт чарта и числовых примеров) подтверждены первичными источниками. Остаются 2 минорных ⚠️ (стилистика/контекст) и 1 🔍 (тонкость RFC-границы), не влияющие на корректность. Подробности ниже.

### Что изменилось относительно 2026-04-26 лога

- **§7 (Безопасность, line 200) — ИСПРАВЛЕНО.** Прошлый лог фиксировал ОШИБКУ: коллизионная граница ошибочно приписывалась Grover. Текущий текст корректно атрибутирует коллизионную границу алгоритму **BHT (Brassard–Høyer–Tapp)** с `2^(k/3)` (~85 бит для SHA-256), а Grover — только preimage `k/2` (~128 бит). Добавлена ссылка [[16](#ref-bht)]. Проверено и подтверждено в этом проходе.
- **§13/§21 (Cassandra repair URL) — ИСПРАВЛЕНО.** Прошлый лог фиксировал 404 на старом URL. Текущая ссылка [[13]] использует `managing/operating/repair.html` — URL загружается и содержит описание Merkle-tree anti-entropy. Подтверждено.
- **§10 (Git, дата миграции) — ИСПРАВЛЕНО.** Прошлый лог оспаривал «2018». Текущий текст корректно даёт: проект выбрал SHA-256 в late 2018 (design); `--object-format=sha256` experimental с Git 2.29 (2020); stable с Git 2.42 (август 2023). Все три даты подтверждены.
- **§19/§21 (Kuszmaul год) — ИСПРАВЛЕНО.** Текущий текст и библиография согласованы: Kuszmaul (2018), PRIMES paper, представлен на PRIMES Conference 2019. Файл `materials/2018/Kuszmaul.pdf` соответствует.

## Section log

| # | Раздел | Строки | Статус | Дата | Резюме |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–30 | ✅ Проверено | 2026-06-11 | Все факты подтверждены (RFC 6962, Yellow Paper App. D, Dynamo §4.7, IPFS arXiv 1407.3561, Cassandra repair). Btrfs-нюанс корректно оговорён в §16. |
| 2 | Определение и структура | 32–69 | ✅ Проверено | 2026-06-11 | RFC 6962 §2.1 цитируется дословно: `0x00`/`0x01` префиксы и `k` = largest power of two strictly less than `n` (k < n ≤ 2k). Схема 8 листьев структурно корректна (высота 3). |
| 3 | Числовой пример | 71–85 | ✅ Проверено | 2026-06-11 | Плейсхолдеры явно помечены как иллюстративные; арифметика корня (4 листа → 2 пары → root) согласована с определением. |
| 4 | Свойства и инварианты | 87–95 | ✅ Проверено | 2026-06-11 | proof size 30·32=960 пересчитан; n=2^20 → 20 хешей (точно для степени двойки); SHAttered 2017. |
| 5 | Доказательства включения | 97–159 | ✅ Проверено | 2026-06-11 | Псевдокод build/verify согласован с RFC 6962: лист `H(0x00‖d)`, узел `H(0x01‖…)`, перенос «лишнего» хеша без хеширования. Audit path для d_3 (H_4 R, H_12 L) пересчитан — корректен. |
| 6 | Алгоритмы построения и обновления | 161–192 | ✅ Проверено | 2026-06-11 | Bottom-up O(n); RFC-6962 split-rule и Bitcoin duplicate-rule описаны точно. |
| 7 | Безопасность | 194–201 | ✅ Проверено | 2026-06-11 | **Прошлая ОШИБКА устранена.** Grover→preimage k/2 (~128 бит); BHT→collision 2^(k/3) (~85 бит). Оба значения пересчитаны и подтверждены. Префиксы 0x00/0x01 и CVE-2012-2459 корректны. |
| 8 | Сложность | 203–213 | ✅ Проверено | 2026-06-11 | Все асимптотики корректны (build O(n); audit/verify/update/append O(log n)). |
| 9 | Визуализация: рост размера proof | 215–276 | ✅ Проверено | 2026-06-11 | Все 14 точек чарта пересчитаны программно — совпадают до байта (`⌈log₂ n⌉·32` и `n·32`). n=2^20 → 640 B / 32 MB; n=2^30 → 960 B / 32 GB — точно. |
| 10 | Применения — Git | 278–289 | ✅ Проверено | 2026-06-11 | **Прошлое замечание устранено.** Merkle DAG, типы объектов, SHA-256 timeline (late 2018 design, 2.29/2020 experimental, 2.42/авг 2023 stable) — всё подтверждено по git docs и RelNotes 2.42. |
| 11 | Применения — Bitcoin | 291–298 | ⚠️ Проверено с замечанием | 2026-06-11 | Header 80 B, SPV §8, ~3000 tx → 12 хешей (⌈log₂3000⌉=12, 384 B) пересчитан. CVE-2012-2459, BIP-37/157/158 корректны. **Замечание:** «вытесняется compact block filters» — slightly overstated, но допустимо. |
| 12 | Применения — Ethereum | 300–308 | ✅ Проверено | 2026-06-11 | Три корня stateRoot/transactionsRoot/receiptsRoot, MPT, Yellow Paper App. D — подтверждено. |
| 13 | Cassandra и Dynamo | 310–319 | ✅ Проверено | 2026-06-11 | **Прошлый 404 устранён.** Dynamo §4.7 + Cassandra repair (managing/operating/) загружается, описывает Merkle-tree anti-entropy. O(diff·log n) корректно. |
| 14 | Certificate Transparency | 321–330 | ✅ Проверено | 2026-06-11 | STH, inclusion vs consistency proof (append-only расширение), SCT, RFC 9162 hash agility — подтверждено по RFC 6962 §2.1.1/2.1.2. |
| 15 | IPFS | 332–334 | ✅ Проверено | 2026-06-11 | Merkle DAG, CID (content-addressing), dag-pb/dag-cbor IPLD codecs — корректно. |
| 16 | ZFS, Btrfs, dm-verity | 336–340 | ⚠️ Проверено с замечанием | 2026-06-11 | ZFS uberblock Merkle tree и dm-verity — корректно. Btrfs CRC32C-нюанс (B-дерево, не классический Merkle; SHA-256/BLAKE2 опционально с kernel 5.5+) **корректно оговорён в самом тексте**. Остаточное ⚠️ — мелкая точность формулировки в §1 (line 29). |
| 17 | Варианты — MPT | 342–352 | ✅ Проверено | 2026-06-11 | leaf/extension/branch, 17 ссылок в branch, RLP+Keccak-256, stateProof/non-membership — точно. |
| 18 | Варианты — SMT | 354–362 | ✅ Проверено | 2026-06-11 | Dahlberg/Pulls/Peeters NordSec 2016 (eprint 2016/683), фикс. глубина, предвычисленные empty-хеши, non-membership, zkSync/Plasma Cash — подтверждено. |
| 19 | Варианты — Verkle | 364–368 | 🔍 Проверено | 2026-06-11 | Vector commitment (KZG/IPA), multi-proof ~O(1), Ballet&Feist 2021, Kuszmaul 2018 — корректно. **🔍:** «Hegota (2H 2026)» — прогноз roadmap; на 2026-06-11 ещё не зафиксирован hard-fork. Помечать к ревалидации. |
| 20 | Сравнение с альтернативами | 370–380 | ✅ Проверено | 2026-06-11 | Качественные сравнения корректны. |
| 21 | Список литературы | 382–414 | ✅ Проверено | 2026-06-11 | **Прошлые замечания устранены.** Cassandra URL (`managing/operating/`) и Kuszmaul (2018/файл 2018) согласованы. Все 16 источников — DOI/URL валидны. |

## Live-страница

Preflight 2026-06-11:
- Live URL загружается (`https://viacheslav-dobrynin.github.io/...`, title «Материалы дисциплины»).
- `gh run list --workflow=deploy.yml --limit=1` — последний run `27277204085` (`success`, 2026-06-10T12:46:00Z).

---

## Полная посентенционная проверка

### 1. Мотивация (lines 10–30)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 1.1 | «Криптографическая хеш-функция `H` сжимает … `k` бит и обладает свойствами collision-resistance, second-preimage и preimage.» | ✅ Подтверждено | [NIST FIPS 180-4 §1](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf); [Cryptographic hash function](https://en.wikipedia.org/wiki/Cryptographic_hash_function). |
| 1.2 | «Этого достаточно, чтобы убедиться в целостности **одного** объекта…» | 💬 Не факт | Описательная мотивация. [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962). |
| 1.3 | «Однако в реальных системах объект чаще всего составной…» | 💬 Не факт | Связка. [Merkle 1987](https://doi.org/10.1007/3-540-48184-2_32). |
| 1.4 | «Для таких сценариев одной плоской хеш-функции мало — три практические проблемы.» | 💬 Не факт | Связка. [RFC 6962 §1](https://datatracker.ietf.org/doc/html/rfc6962#section-1). |
| 1.5 | «Если хранить «один большой хеш» … не позволяет указать, что именно изменилось.» | ✅ Подтверждено | Прямое следствие свойств хеша; мотивация Merkle trees в [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962). |
| 1.6 | «Лёгкому клиенту нужно убедиться, что `data_i` входит в агрегат, не скачивая весь набор.» | ✅ Подтверждено | [RFC 6962 §1](https://datatracker.ietf.org/doc/html/rfc6962#section-1); [Bitcoin whitepaper §8](https://bitcoin.org/bitcoin.pdf). |
| 1.7 | «Простой список чексумм … требует `O(n)` доверенных байт.» | ✅ Подтверждено | Тривиальное следствие. [Merkle CRYPTO '87](https://doi.org/10.1007/3-540-48184-2_32). |
| 1.8 | «При сравнении двух копий … локализовать различающиеся подмножества.» | ✅ Подтверждено | Anti-entropy в [Dynamo §4.7](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf). |
| 1.9 | «Поэлементное сравнение даёт `O(n)` обменов; хочется `O(diff · log n)`.» | ✅ Подтверждено | [Cassandra repair docs](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/repair.html). |
| 1.10 | «Бинарное дерево Меркла, предложенное Ральфом Мерклом в докторской диссертации 1979 года и развитое в CRYPTO '87…» | ✅ Подтверждено | [Merkle 1979 PhD thesis, Stanford](https://www.ralphmerkle.com/papers/Thesis1979.pdf) (гл. о tree authentication); [Merkle CRYPTO '87, LNCS 293, 369–378](https://doi.org/10.1007/3-540-48184-2_32). |
| 1.11 | «Один корневой хеш … доказательство включения имеет размер `O(log n) · k` бит.» | ✅ Подтверждено | [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1) — audit path `≤ ⌈log₂ n⌉(+1)` узлов. |
| 1.12 | «Bitcoin — `merkle root` транзакций в заголовке блока.» | ✅ Подтверждено | [Bitcoin whitepaper §7-§8](https://bitcoin.org/bitcoin.pdf); [Block header (80 B, hashMerkleRoot)](https://en.bitcoin.it/wiki/Block_hashing_algorithm). |
| 1.13 | «Ethereum — три корня (stateRoot, transactionsRoot, receiptsRoot) на основе MPT.» | ✅ Подтверждено | [Yellow Paper §4.3.3 & Appendix D](https://ethereum.github.io/yellowpaper/paper.pdf). |
| 1.14 | «Git — Merkle DAG из коммитов, деревьев и блобов.» | ✅ Подтверждено | [Pro Git Ch. 10 «Git Objects»](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects). |
| 1.15 | «Certificate Transparency — публичные append-only логи согласно RFC 6962.» | ✅ Подтверждено | [RFC 6962 abstract](https://datatracker.ietf.org/doc/html/rfc6962). |
| 1.16 | «IPFS — content-addressable хранилище на Merkle DAG.» | ✅ Подтверждено | [Benet 2014, arXiv:1407.3561](https://arxiv.org/abs/1407.3561); [IPFS Merkle DAG docs](https://docs.ipfs.tech/concepts/merkle-dag/). |
| 1.17 | «ZFS, Btrfs, dm-verity — проверяемое чтение и verified boot.» | ⚠️ Неточно | ZFS/dm-verity корректно ([dm-verity](https://source.android.com/security/verifiedboot/dm-verity)). **Btrfs** по умолчанию CRC32C (не крипто-хеш), checksum tree = B-дерево; Merkle-семантика только опц. с SHA-256/BLAKE2. Это **корректно оговорено в §16 (line 338)**, поэтому summary-строка line 29 слегка упрощает. См. [Btrfs Checksumming](https://btrfs.readthedocs.io/en/latest/Checksumming.html). |
| 1.18 | «Apache Cassandra, Amazon DynamoDB — anti-entropy при репликации.» | ✅ Подтверждено | [Dynamo §4.7](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf); [Cassandra repair](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/repair.html). |

### 2. Определение и структура (lines 32–69)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 2.1 | «Дерево Меркла — это полное (или почти полное) бинарное дерево … внутренний узел — хеш конкатенации хешей детей.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1); [Merkle 1987](https://doi.org/10.1007/3-540-48184-2_32). |
| 2.2 | «Корень (Merkle root) криптографически привязывает весь набор листьев к одному значению фиксированной длины.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 2.3 | «`H : {0,1}^* → {0,1}^k` (SHA-256 → k=256, BLAKE3 → 256, Keccak-256 → 256).» | ✅ Подтверждено | [FIPS 180-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf); [BLAKE3 spec](https://github.com/BLAKE3-team/BLAKE3-specs/blob/master/blake3.pdf); [FIPS 202 (SHA-3/Keccak)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf). |
| 2.4 | «Листья `L_i = H(d_i)`; узлы `N(left,right)=H(left‖right)`; корень — единственный узел без родителя.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1) (с поправкой на префиксы далее). |
| 2.5 | «RFC 6962 §2.1 фиксирует: `MTH({d}) = SHA-256(0x00 ‖ d)`; `MTH(D[n]) = SHA-256(0x01 ‖ MTH(D[0:k]) ‖ MTH(D[k:n]))`.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1) дословно: «MTH({d(0)}) = SHA-256(0x00 ‖ d(0))» и «MTH(D[n]) = SHA-256(0x01 ‖ MTH(D[0:k]) ‖ MTH(D[k:n]))». |
| 2.6 | «где `k` — наибольшая степень двойки, строго меньшая `n`.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1): «let k be the largest power of two smaller than n (i.e., k < n <= 2k)». |
| 2.7 | «Без префиксов внутренний узел и лист … могут совпасть по биту, что открывает атаки на коллизии.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1); [Bitcoin Optech merkle-tree-vulnerabilities](https://bitcoinops.org/en/topics/merkle-tree-vulnerabilities/). |
| 2.8 | «Дерево над `n` листьями имеет высоту `h = ⌈log_2 n⌉`.» | ✅ Подтверждено | Стандартное свойство; для RFC split-rule — индукцией. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 2.9 | «Если `n` — степень двойки, дерево сбалансировано; иначе правое поддерево короче, форма определяется `n`.» | ✅ Подтверждено | Следствие split-rule [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1) (k < n ≤ 2k). |
| 2.10 | Схема дерева для 8 листьев. | 💬 Не факт | Иллюстрация; структурно корректна (каждый родитель = хеш двух детей; высота 3 для n=8). [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 2.11 | «`L_i = H(d_i)` — листья; `A,B,C,D,N12,N34,root` — внутренние узлы; root — k-битное значение для 8 блоков.» | ✅ Подтверждено | Согласовано с 2.4. [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |

### 3. Числовой пример (lines 71–85)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 3.1 | «Четыре блока … значения **иллюстративны**, не результаты реального SHA-256.» | 💬 Не факт | Дисклеймер о плейсхолдерах. [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 3.2 | Таблица H_1..H_4, H_{12}, H_{34}, root. | 💬 Не факт | Иллюстрация; логика (4 листа → 2 пары → корень) согласована с §2. [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 3.3 | «Корень — единственное k-битное значение … имея сам блок и `O(log n)` промежуточных хешей.» | ✅ Подтверждено | [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |

### 4. Свойства и инварианты (lines 87–95)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 4.1 | «Tamper-evidence. Любая модификация бита в листе пересчитывает все хеши на пути до корня; collision-resistance гарантирует невозможность подбора другой последовательности с тем же root.» | ✅ Подтверждено | [Merkle 1987](https://doi.org/10.1007/3-540-48184-2_32); [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 4.2 | «Локальность audit path: достаточно `⌈log_2 n⌉` хешей-сиблингов; для n=2^20 — ровно 20 хешей.» | ✅ Подтверждено | Для степени двойки audit path = ровно ⌈log₂ n⌉ = 20. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 4.3 | «Размер доказательства: `⌈log_2 n⌉ · k` бит; SHA-256, n=2^30 → 30·32=960 байт.» | ✅ Подтверждено | Пересчитано: ⌈log₂ 2^30⌉=30, 30·32=960. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 4.4 | «Чувствительность к порядку листьев: перестановка двух транзакций в блоке Bitcoin даёт другой root.» | ✅ Подтверждено | Прямое следствие упорядоченной конкатенации. [Bitcoin whitepaper §7](https://bitcoin.org/bitcoin.pdf). |
| 4.5 | «Caveat: безопасность не выше хеш-функции; SHA-1 после SHAttered (2017) теряет защиту; мигрируют на SHA-256/BLAKE3/Keccak-256.» | ✅ Подтверждено | [SHAttered, Stevens et al., CRYPTO '17, eprint 2017/190](https://eprint.iacr.org/2017/190). |

### 5. Доказательства включения (lines 97–159)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 5.1 | «Audit path / Merkle proof — упорядоченный список хешей-сиблингов от листа к корню; проверяющий пересчитывает корень снизу вверх.» | ✅ Подтверждено | [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 5.2 | Псевдокод `build_audit_path` (side L/R, перенос «лишнего» хеша без хеширования, `H(0x01‖…)`). | ✅ Подтверждено | Соответствует split/duplicate-free правилу [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1); «лишний» хеш промотируется без хеширования — как в RFC. |
| 5.3 | Псевдокод `verify` (`H(0x00‖leaf)`, затем `H(0x01‖…)` с учётом side). | ✅ Подтверждено | Префиксы 0x00/0x01 точно по [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 5.4 | «Audit path для d_3: H_4 (R) на уровне листьев, H_{12} (L) на уровне узлов; пересчёт H_3 → H(H_3‖H_4)=H_{34} → H(H_{12}‖H_{34})=root.» | ✅ Подтверждено | Пересчитано вручную для n=4: путь корректен, sibling order верный. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 5.5 | Таблица сложности: audit path `⌈log_2 n⌉` хешей; биты `⌈log_2 n⌉·k`; время `⌈log_2 n⌉` хеш-вычислений. | ✅ Подтверждено | [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1) (worst-case bound `⌈log₂ n⌉+1`; для степеней двойки = ⌈log₂ n⌉). |
| 5.6 | «k=256, n=2^30: 30 хешей, 960 байт, 30 вызовов H — единицы микросекунд.» | ✅ Подтверждено | Арифметика пересчитана: 30·32=960 B. Время — реалистичная оценка для SHA-256 (~10⁷-10⁸ H/с на CPU). |

### 6. Алгоритмы построения и обновления (lines 161–192)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 6.1 | «Каноническое построение — bottom-up; время O(n), память O(n) или O(log n) потоково.» | ✅ Подтверждено | Стандартный bottom-up. [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 6.2 | Псевдокод `build_root` (листья `H(0x00‖b)`, узлы `H(0x01‖…)`, split-rule перенос). | ✅ Подтверждено | Согласован с [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 6.3 | «Bitcoin: при нечётном числе узлов последний **дублируется** и хешируется сам с собой.» | ✅ Подтверждено | [Bitcoin Optech](https://bitcoinops.org/en/topics/merkle-tree-vulnerabilities/); [Bitcoin Core merkle.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/consensus/merkle.cpp). |
| 6.4 | «CVE-2012-2459: подбор дублирующей транзакции → разные деревья дают один root → DoS; закрыто в Bitcoin Core, но дизайн-уязвимость осталась.» | ✅ Подтверждено | [CVE-2012-2459 (NVD)](https://nvd.nist.gov/vuln/detail/CVE-2012-2459) (DoS, block-processing outage); механизм дублей — [Bitcoin Optech](https://bitcoinops.org/en/topics/merkle-tree-vulnerabilities/). |
| 6.5 | «RFC 6962 §2.1: дерево разбивается асимметрично, `k` < n ≤ 2k, левое поддерево сбалансировано, правое рекурсивно; никакого дублирования.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1) дословно. |
| 6.6 | «Современные системы (CT, IPFS, Trillian) используют RFC-6962-подобные правила; дублирующее правило Bitcoin — ради обратной совместимости.» | ✅ Подтверждено | [Trillian (Google CT log)](https://github.com/google/trillian); [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962). |
| 6.7 | «Изменение одного листа: пересчёт только пути до корня — O(log n) при кэше.» | ✅ Подтверждено | Прямое следствие; [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 6.8 | «Добавление листа (append-only CT): амортизированно O(log n); стек «правых» хешей по аналогии с двоичным счётчиком.» | ✅ Подтверждено | Стандартный merkle-mountain-range/append подход. [RFC 6962 §2.1.2](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.2). |

### 7. Безопасность (lines 194–201)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 7.1 | «Second-preimage / domain confusion: без префикса атакующий строит «параллельное» поддерево с тем же корнем; RFC 6962 §2.1 предписывает 0x00/0x01.» | ✅ Подтверждено | [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1); [second-preimage в Merkle trees](https://bitcoinops.org/en/topics/merkle-tree-vulnerabilities/). |
| 7.2 | «CVE-2012-2459: дублирование «лишнего» листа → разные наборы дают один root → block-malleability и DoS; закрыто проверкой валидатора; split-rule RFC 6962 строже.» | ✅ Подтверждено | [CVE-2012-2459 (NVD)](https://nvd.nist.gov/vuln/detail/CVE-2012-2459); [Bitcoin Optech](https://bitcoinops.org/en/topics/merkle-tree-vulnerabilities/). |
| 7.3 | «SHA-1 после SHAttered (2017) небезопасна; Git переходит на SHA-256 (`extensions.objectFormat = sha256`); рекомендуются SHA-256/SHA-3/Keccak-256/BLAKE3.» | ✅ Подтверждено | [SHAttered eprint 2017/190](https://eprint.iacr.org/2017/190); [git hash-function-transition](https://git-scm.com/docs/hash-function-transition). |
| 7.4 | «Постквантово: Grover снижает preimage с `k` до `k/2`.» | ✅ Подтверждено | Grover даёт `2^(k/2)` для поиска прообраза. [Grover's algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm). |
| 7.5 | «Алгоритм BHT (Brassard–Høyer–Tapp) снижает collision-стойкость с `k/2` до `k/3` бит.» | ✅ Подтверждено | BHT: `O(2^(k/3))` запросов к k-битной функции. [BHT algorithm](https://en.wikipedia.org/wiki/BHT_algorithm) («O(n^(1/3)) queries»); [BHT, arXiv:quant-ph/9705002](https://arxiv.org/abs/quant-ph/9705002). |
| 7.6 | «Для SHA-256 это ~128 бит preimage и ~85 бит collision; последнее достаточно для практики.» | ✅ Подтверждено | Пересчитано: 256/2=128; 256/3≈85.3. Запас в 85 бит коллизионной стойкости — стандартная оценка. [BHT algorithm](https://en.wikipedia.org/wiki/BHT_algorithm). |
| 7.7 | «Утечка структуры через timing: при приватных данных (zk-proof) проверять audit path в постоянное время.» | ✅ Подтверждено | Общая практика side-channel-устойчивости. [Timing attack](https://en.wikipedia.org/wiki/Timing_attack). |

### 8. Сложность (lines 203–213)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 8.1 | «Построение root: время O(n), память O(n)/O(log n) потоково, артефакт k бит.» | ✅ Подтверждено | n листьев → n-1 внутренних узлов = O(n). [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 8.2 | «Audit path: время O(log n), память O(log n), артефакт `⌈log_2 n⌉·k` бит.» | ✅ Подтверждено | [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 8.3 | «Верификация audit path: время O(log n), память O(1).» | ✅ Подтверждено | Итеративный пересчёт по пути. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 8.4 | «Обновление одного листа (с кэшем): время O(log n), память O(n) для кэша всех узлов.» | ✅ Подтверждено | Пересчёт пути; кэш всех узлов = O(n). [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 8.5 | «Append (амортизированно): время O(log n), память O(log n) для side-stack.» | ✅ Подтверждено | Двоичный счётчик / MMR. [RFC 6962 §2.1.2](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.2). |

### 9. Визуализация: рост размера proof (lines 215–276)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 9.1 | «`proof_size(n) = ⌈log_2 n⌉ · k / 8` байт; наивный список — `n · k / 8`.» | ✅ Подтверждено | Прямая формула. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 9.2 | Все 14 точек обоих рядов чарта (n=2^4…2^30). | ✅ Подтверждено | Программно пересчитано — все 14 точек совпадают до байта (`⌈log₂ n⌉·32` и `n·32`). [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |
| 9.3 | «n=2^20 → Merkle 640 байт, список 32 МБ; n=2^30 → proof 960 байт, список 32 ГБ.» | ✅ Подтверждено | Пересчитано: 20·32=640 B; 2^20·32=32 MiB; 30·32=960 B; 2^30·32=32 GiB. [RFC 6962 §2.1.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.1). |

### 10. Применения — Git (lines 278–289)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 10.1 | «Объектный граф Git — Merkle DAG; объекты blob/tree/commit/tag идентифицируются хешем содержимого и ссылаются друг на друга через хеши.» | ✅ Подтверждено | [Pro Git Ch. 10 «Git Objects»](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects). |
| 10.2 | «blob — содержимое файла; tree — отображение имя→(mode,тип,hash); commit — tree + родители + автор; tag — подписанная ссылка на коммит.» | ✅ Подтверждено | [Pro Git Ch. 10](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects). |
| 10.3 | «Коммит фиксирует всё состояние одной строкой-хешем: изменение файла каскадирует вверх до commit hash.» | ✅ Подтверждено | Свойство content-addressing. [Pro Git Ch. 10](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects). |
| 10.4 | «Исторически SHA-1; с 2020 Git поддерживает SHA-256 (experimental с 2.29; стабильно с 2.42, август 2023), мотивировано SHAttered (2017).» | ✅ Подтверждено | Проект выбрал SHA-256 в late 2018 ([git hash-function-transition](https://git-scm.com/docs/hash-function-transition)); experimental с [Git 2.29 (2020)](https://github.com/git/git/blob/master/Documentation/RelNotes/2.29.0.adoc); «tone down the warning … experimental curiosity» — [Git 2.42.0 RelNotes (авг 2023)](https://github.com/git/git/blob/master/Documentation/RelNotes/2.42.0.adoc). |

### 11. Применения — Bitcoin (lines 291–298)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 11.1 | «Каждый блок содержит в заголовке `merkle_root` — корень дерева над всеми транзакциями.» | ✅ Подтверждено | [Bitcoin whitepaper §7](https://bitcoin.org/bitcoin.pdf); [Block header (hashMerkleRoot 32 B)](https://en.bitcoin.it/wiki/Block_hashing_algorithm). |
| 11.2 | «SPV-кошелёк хранит только заголовки (~80 байт каждый) и запрашивает audit path; восстанавливает merkle root без скачивания блока.» | ✅ Подтверждено | Header = 80 B ([Block header](https://en.bitcoin.it/wiki/Block_hashing_algorithm)); SPV — [Bitcoin whitepaper §8](https://bitcoin.org/bitcoin.pdf). |
| 11.3 | «Для блока с ~3000 транзакций audit path содержит ~12 хешей (~384 байта).» | ✅ Подтверждено | Пересчитано: ⌈log₂ 3000⌉=12; 12·32=384 B. [Bitcoin whitepaper §8](https://bitcoin.org/bitcoin.pdf). |
| 11.4 | «Bitcoin использует «дублирующее» правило → CVE-2012-2459, закрытая явной проверкой невозможности дублей.» | ✅ Подтверждено | [CVE-2012-2459 (NVD)](https://nvd.nist.gov/vuln/detail/CVE-2012-2459); [Bitcoin Core merkle.cpp](https://github.com/bitcoin/bitcoin/blob/master/src/consensus/merkle.cpp). |
| 11.5 | «Bloom-фильтрация для SPV описана в BIP-37, признана уязвимой к корреляции, вытесняется compact block filters (BIP-157/158).» | ⚠️ Slightly overstated | BIP-37 действительно имеет privacy-проблемы; BIP-157/158 — альтернатива, но в проде неполностью «вытеснила» BIP-37. Формулировка допустима. [BIP-37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki); [BIP-157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki). |

### 12. Применения — Ethereum (lines 300–308)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 12.1 | «Заголовок Ethereum содержит три корня: stateRoot, transactionsRoot, receiptsRoot.» | ✅ Подтверждено | [Yellow Paper §4.3.3 (block header) & Appendix D](https://ethereum.github.io/yellowpaper/paper.pdf). |
| 12.2 | «stateRoot — состояние аккаунтов/storage; transactionsRoot — транзакции; receiptsRoot — receipts (события, gas used, статус).» | ✅ Подтверждено | [Yellow Paper §4.3.1, Appendix D](https://ethereum.github.io/yellowpaper/paper.pdf). |
| 12.3 | «Все три используют Merkle Patricia Trie (MPT) — гибрид трая и дерева Меркла для разреженного key-value с криптодоказательствами.» | ✅ Подтверждено | [Yellow Paper Appendix D](https://ethereum.github.io/yellowpaper/paper.pdf); [Ethereum MPT docs](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/). |

### 13. Cassandra и DynamoDB anti-entropy (lines 310–319)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 13.1 | «Поэлементное сравнение нереалистично; Cassandra и Dynamo строят над диапазонами ключей merkle tree и обмениваются корнями.» | ✅ Подтверждено | [Dynamo §4.7](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf); [Cassandra repair](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/repair.html) («compares the data with merkle trees»). |
| 13.2 | Шаги 1–4 repair: координатор инициирует на диапазоне токенов; реплики строят дерево над SSTable; обмен корнями; рекурсивный спуск только в различающиеся ветви. | ✅ Подтверждено | [Cassandra repair](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/repair.html); [Dynamo §4.7](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf). |
| 13.3 | «Сложность по обмену — `O(diff · log n)`, где diff — число различающихся листьев.» | ✅ Подтверждено | Стандартная оценка Merkle-anti-entropy. [Dynamo §4.7](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf). |

### 14. Certificate Transparency (lines 321–330)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 14.1 | «CT — публичный append-only лог TLS-сертификатов; любой может убедиться, что сертификат попал в лог.» | ✅ Подтверждено | [RFC 6962 abstract/§1](https://datatracker.ietf.org/doc/html/rfc6962#section-1). |
| 14.2 | «Лог = append-only дерево Меркла; центральный объект — Signed Tree Head (STH): подписанная пара (размер дерева, корень).» | ✅ Подтверждено | [RFC 6962 §3.5](https://datatracker.ietf.org/doc/html/rfc6962#section-3.5). |
| 14.3 | «Inclusion proof — audit path для одного сертификата; consistency proof — последовательность хешей, демонстрирующая append-only-расширение старого STH.» | ✅ Подтверждено | [RFC 6962 §2.1.1 (audit) & §2.1.2 (consistency)](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1.2). |
| 14.4 | «Браузеры (Chrome, Safari) требуют SCT (Signed Certificate Timestamp).» | ✅ Подтверждено | [RFC 6962 §3.3 SCT](https://datatracker.ietf.org/doc/html/rfc6962#section-3.3); [Chrome CT policy](https://googlechrome.github.io/CertificateTransparency/ct_policy.html). |
| 14.5 | «RFC 9162 уточняет CT v2.0 с поддержкой нескольких хеш-функций и улучшенной структурой записей.» | ✅ Подтверждено | [RFC 9162 (CT v2.0)](https://datatracker.ietf.org/doc/html/rfc9162). |

### 15. IPFS и content-addressable storage (lines 332–334)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 15.1 | «IPFS строит файловую систему как Merkle DAG; объект идентифицируется CID — мультихешем содержимого.» | ✅ Подтверждено | [IPFS Merkle DAG](https://docs.ipfs.tech/concepts/merkle-dag/); [Benet 2014, arXiv:1407.3561](https://arxiv.org/abs/1407.3561). |
| 15.2 | «Большие файлы разбиваются на чанки в дерево; директории — ноды-карты имя→CID; манипуляция изменит CID.» | ✅ Подтверждено | [IPFS UnixFS docs](https://docs.ipfs.tech/concepts/file-systems/); [IPFS CID](https://docs.ipfs.tech/concepts/content-addressing/). |
| 15.3 | «Сериализация — dag-pb (исторический) и dag-cbor (новый, на CBOR).» | ✅ Подтверждено | [IPLD dag-pb codec](https://ipld.io/specs/codecs/dag-pb/spec/); [IPLD dag-cbor codec](https://ipld.io/specs/codecs/dag-cbor/spec/). |

### 16. ZFS, Btrfs, dm-verity (lines 336–340)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 16.1 | «ZFS хранит хеши (Fletcher4 по умолчанию, опц. SHA-256) в указателях родительских блоков, образуя дерево Меркла с корнем uberblock — детектирование bit-rot и целостный snapshot.» | ✅ Подтверждено | [OpenZFS checksums](https://openzfs.github.io/openzfs-docs/Basic%20Concepts/Checksums.html); [ZFS data integrity](https://en.wikipedia.org/wiki/ZFS#Data_integrity). |
| 16.2 | «Btrfs хранит CRC32C (по умолчанию) или SHA-256/BLAKE2 (опц., kernel 5.5+) в отдельном checksum tree — это B-дерево, не дерево Меркла в классическом смысле, и CRC32C — не криптографический хеш.» | ✅ Подтверждено | [Btrfs Checksumming](https://btrfs.readthedocs.io/en/latest/Checksumming.html) (CRC32C default; xxhash/sha256/blake2 с kernel 5.5). Корректное уточнение, снимающее упрощение из line 29. |
| 16.3 | «dm-verity располагает над read-only-разделом дерево Меркла поверх блоков фикс. размера; ядро сверяет хеш блока вверх до подписанного root hash; целостность при загрузке (Android verified boot, Chrome OS).» | ✅ Подтверждено | [dm-verity (kernel docs)](https://docs.kernel.org/admin-guide/device-mapper/verity.html); [Android dm-verity](https://source.android.com/security/verifiedboot/dm-verity). |

### 17. Варианты — Merkle Patricia Trie (lines 342–352)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 17.1 | «MPT — модификация бинарного дерева Меркла под разреженные key-value с переменной длиной ключей.» | ✅ Подтверждено | [Yellow Paper Appendix D](https://ethereum.github.io/yellowpaper/paper.pdf). |
| 17.2 | «Узлы трёх типов: leaf (nibble-путь, значение); extension (nibble-prefix, ссылка); branch (массив 17 ссылок: 16 nibble-веток + значение).» | ✅ Подтверждено | [Ethereum MPT docs](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/). |
| 17.3 | «Каждый узел сериализуется RLP и хешируется Keccak-256; значение в родителе — хеш дочернего узла.» | ✅ Подтверждено | [Yellow Paper Appendix B (RLP) & D](https://ethereum.github.io/yellowpaper/paper.pdf). |
| 17.4 | «MPT даёт доказательства наличия (stateProof) и отсутствия (отсутствие пути) — полезно для stateless-клиентов.» | ✅ Подтверждено | [Ethereum MPT docs (proofs)](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/). |

### 18. Варианты — Sparse Merkle Trees (lines 354–362)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 18.1 | «SMT — дерево фиксированной глубины (напр., 256 уровней при 32-байтных ключах), почти все листья пусты.» | ✅ Подтверждено | [Dahlberg, Pulls, Peeters, NordSec 2016, eprint 2016/683](https://eprint.iacr.org/2016/683). |
| 18.2 | «Ключ адресует лист по битам — путь длины H уровней; пустые поддеревья имеют предвычисленные хеши; non-membership proof тривиален (лист = 0/nil).» | ✅ Подтверждено | [eprint 2016/683 §3 (caching empty subtrees, (non-)membership)](https://eprint.iacr.org/2016/683). |
| 18.3 | «SMT используются в zk-rollup'ах (zkSync, Plasma Cash), накоплении состояния (IBC), privacy-preserving credentials.» | ✅ Подтверждено | [zkSync state tree](https://docs.zksync.io/); [Plasma Cash](https://ethresear.ch/t/plasma-cash-plasma-with-much-less-per-user-data-checking/1298). |

### 19. Варианты — Verkle Trees (lines 364–368)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 19.1 | «Verkle Tree заменяет хеш-функцию на векторное полиномиальное обязательство; коммитмент к вектору многих дочерних хешей; multi-proof почти за O(1) по числу листьев.» | ✅ Подтверждено | [Ballet & Feist 2021, EF Blog](https://blog.ethereum.org/2021/12/02/verkle-tree-structure); [Kuszmaul 2018](https://math.mit.edu/research/highschool/primes/materials/2018/Kuszmaul.pdf). |
| 19.2 | «Мотив для Ethereum — stateless-клиенты; при ширине 256 witness падает с десятков МБ (бинарный MPT/Keccak) до сотен КБ.» | ✅ Подтверждено | [Ballet & Feist 2021](https://blog.ethereum.org/2021/12/02/verkle-tree-structure). |
| 19.3 | «EF описывает структуру в Ballet & Feist (2021); академическое введение — Kuszmaul (2018); внедрение в Verge-этапе через EIP-6800/4762; hard-fork Hegota (2H 2026), тестнет Verkle Gen Devnet 6.» | 🔍 Требует ревалидации | EF blog/Kuszmaul/EIP-6800/4762 — корректны ([EIP-6800](https://eips.ethereum.org/EIPS/eip-6800)). **🔍:** «Hegota (2H 2026)» — прогноз roadmap, на 2026-06-11 не зафиксирован. Перепроверить при следующем проходе. [Ethereum roadmap](https://ethereum.org/en/roadmap/verkle-trees/). |

### 20. Сравнение с альтернативами (lines 370–380)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 20.1 | Таблица: один хеш / список чексумм / HMAC-цепочка / дерево Меркла / per-item подпись по столбцам proof-size, inclusion, anti-entropy, append. | ✅ Подтверждено | Качественные сравнения корректны; Merkle = `O(log n)` proof, `O(diff·log n)` anti-entropy, append амортиз. `O(log n)`. [RFC 6962 §2.1](https://datatracker.ietf.org/doc/html/rfc6962#section-2.1). |
| 20.2 | «Дерево Меркла — единственный метод, дающий одновременно компактное доказательство, эффективный anti-entropy и append без перестройки.» | ✅ Подтверждено | Следствие таблицы. [Merkle 1987](https://doi.org/10.1007/3-540-48184-2_32); [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962). |

### 21. Список литературы (lines 382–414)

| # | Предложение | Вердикт | Доказательство |
|---|---|---|---|
| 21.1 | [1] Merkle 1979 PhD thesis Stanford, гл. о authentication tree. | ✅ Подтверждено | [Thesis1979.pdf](https://www.ralphmerkle.com/papers/Thesis1979.pdf). |
| 21.2 | [2] Merkle (1988) CRYPTO '87, LNCS 293, 369–378, DOI. | ✅ Подтверждено | [doi.org/10.1007/3-540-48184-2_32](https://doi.org/10.1007/3-540-48184-2_32). |
| 21.3 | [3] RFC 6962 (CT), §2.1 префиксы 0x00/0x01 + split-rule. | ✅ Подтверждено | [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962). |
| 21.4 | [4] RFC 9162 (CT v2.0). | ✅ Подтверждено | [RFC 9162](https://datatracker.ietf.org/doc/html/rfc9162). |
| 21.5 | [5] Nakamoto 2008 Bitcoin whitepaper (§8 SPV). | ✅ Подтверждено | [bitcoin.org/bitcoin.pdf](https://bitcoin.org/bitcoin.pdf). |
| 21.6 | [6] Wood, Ethereum Yellow Paper, Appendix D (MPT). | ✅ Подтверждено | [yellowpaper/paper.pdf](https://ethereum.github.io/yellowpaper/paper.pdf). |
| 21.7 | [7] Chacon & Straub, Pro Git, Ch. 10. | ✅ Подтверждено | [git-scm.com/book Ch. 10](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects). |
| 21.8 | [8] DeCandia et al. 2007 Dynamo, SOSP '07, §4.7. | ✅ Подтверждено | [amazon-dynamo-sosp2007.pdf](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf). |
| 21.9 | [9] Benet 2014 IPFS, arXiv:1407.3561. | ✅ Подтверждено | [arxiv.org/abs/1407.3561](https://arxiv.org/abs/1407.3561). |
| 21.10 | [10] Dahlberg/Pulls/Peeters 2016 SMT, eprint 2016/683. | ✅ Подтверждено | [eprint.iacr.org/2016/683](https://eprint.iacr.org/2016/683). |
| 21.11 | [11] Ballet & Feist 2021 Verkle, EF Blog. | ✅ Подтверждено | [blog.ethereum.org/2021/12/02](https://blog.ethereum.org/2021/12/02/verkle-tree-structure). |
| 21.12 | [12] CVE-2012-2459 NVD. | ✅ Подтверждено | [nvd.nist.gov/vuln/detail/CVE-2012-2459](https://nvd.nist.gov/vuln/detail/CVE-2012-2459). |
| 21.13 | [13] Apache Cassandra Repair (anti-entropy), URL `managing/operating/repair.html`. | ✅ Подтверждено | [Cassandra repair](https://cassandra.apache.org/doc/latest/cassandra/managing/operating/repair.html) — загружается, описывает Merkle-tree repair. (Прошлый 404 устранён.) |
| 21.14 | [14] Stevens et al. 2017 SHAttered, CRYPTO '17, eprint 2017/190. | ✅ Подтверждено | [eprint.iacr.org/2017/190](https://eprint.iacr.org/2017/190). |
| 21.15 | [15] Kuszmaul 2018, PRIMES paper MIT (PRIMES Conf. 2019). | ✅ Подтверждено | [Kuszmaul.pdf](https://math.mit.edu/research/highschool/primes/materials/2018/Kuszmaul.pdf). Год в тексте и файле согласованы. |
| 21.16 | [16] Brassard, Høyer, Tapp 1997 BHT, arXiv:quant-ph/9705002. | ✅ Подтверждено | [arxiv.org/abs/quant-ph/9705002](https://arxiv.org/abs/quant-ph/9705002). |

---

## Перечень открытых пунктов (для top-university bar)

Фактических ошибок (❌) нет. Открытые минорные пункты:

1. **⚠️ line 29 / 1.17** — summary-строка «ZFS, Btrfs, dm-verity — проверяемое чтение и verified boot» упрощает Btrfs (по умолчанию CRC32C, не криптографический хеш; checksum tree — B-дерево). Нюанс **корректно раскрыт** в §16 (line 338), так что это лишь лёгкое упрощение в обзорном списке, не ошибка. Источник: [Btrfs Checksumming](https://btrfs.readthedocs.io/en/latest/Checksumming.html).
2. **⚠️ line 298 / 11.5** — «Bloom-фильтрация… вытесняется compact block filters (BIP-157/158)» — slightly overstated; BIP-157/158 — альтернатива, но не полностью вытеснила BIP-37 в проде. Формулировка допустима. Источник: [BIP-157](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki).
3. **🔍 line 368 / 19.3** — «Hegota (запланирован на 2H 2026)» — прогноз roadmap; на дату аудита (2026-06-11) hard-fork не зафиксирован. Ревалидировать при следующем проходе. Источник: [Ethereum Verkle roadmap](https://ethereum.org/en/roadmap/verkle-trees/).

Ни один из пунктов не является блокирующим. Страница держит планку top-university по точности.

## Разрешение замечаний (2026-06-11, orchestrator)

- **Пункт 1 (⚠️ line 29, ZFS/Btrfs/dm-verity)** — ACCEPTED AS-IS. Обзорная строка описывает *функцию* («проверяемое чтение и verified boot»), а не утверждает, что все три системы — деревья Меркла; нюанс Btrfs (CRC32C по умолчанию, checksum tree = B-дерево) полностью раскрыт в §16 (line 338). Правка не требуется.
- **Пункт 2 (⚠️ line 298, BIP-157/158)** — FIXED. Формулировка «вытесняется compact block filters» переписана: «в качестве более приватной альтернативы предложены compact block filters (BIP-157/158), на которые постепенно переходят многие лёгкие кошельки» — BIP-157/158 теперь поданы как альтернатива, а не как полная замена BIP-37.
- **Пункт 3 (🔍 line 368, Hegota/2H 2026)** — FIXED. Неподтверждённый прогноз («Hegota, 2H 2026, Verkle Gen Devnet 6») удалён; заменён на хедж «конкретный hard-fork и его сроки на момент написания окончательно не зафиксированы и могут меняться; работа ведётся на экспериментальных devnet'ах».

Сборка зелёная (Node 20). Submodule commit `a8d0c3c`, bump родителя, деплой run `27316099300` — GREEN. **Обновлённый вердикт: APPROVED — 0 ❌, открытые ⚠️/🔍 устранены или приняты с обоснованием.**
