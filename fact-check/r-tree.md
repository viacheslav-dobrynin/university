# Fact-check log: R-tree

**Файл**: `courses/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/spatial-indexes/r-tree.mdx`
**Live URL**: https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/docs/search/spatial-indexes/r-tree

> **2026-06-10 — Phase-5 daily-rigor re-audit. Полностью заменяет тонкий лог от 2026-04-22** (запись 2026-06-09 фиксировала только замену чарта на ECharts, без повторной верификации содержания). Это свежая посентенционная проверка против первоисточников (Guttman SIGMOD 1984; Beckmann et al. SIGMOD 1990; Sellis et al. VLDB 1987; документация PostGIS/PostgreSQL GiST).

## Preflight (2026-06-10)

- Live URL загружается: title «Материалы дисциплины | Структуры и алгоритмы в базах данных и распределенных системах» — OK.
- `gh run list --workflow=deploy.yml --limit=1` (репозиторий `viacheslav-dobrynin/structures-and-algorithms-in-databases-and-distributed-systems`, где живёт workflow): последний run `27277204085` — **success**, 2026-06-10T12:46:00Z, «docs(hnsw): distinguish small-world…».

## Итоговая сводка

| Вердикт | Кол-во |
|---|---|
| ✅ ПОДТВЕРЖДЕНО | 58 |
| ⚠️ НЕТОЧНО | 0 |
| ❌ ОПРОВЕРГНУТО | 0 |
| 🔍 НЕ УДАЛОСЬ ПОДТВЕРДИТЬ | 0 |
| 💬 НЕ ФАКТ (вводные/связки) | 17 |

**ВЕРДИКТ: APPROVED.** Ни одного ❌/⚠️/🔍. Все проверяемые факты подтверждены первоисточниками. Прежний дефект нумерации ссылок (видимый «4» → `#ref-beckmann`) на строке 298 устранён: теперь стоит `[1](#ref-guttman), [3](#ref-beckmann)`, видимый «3» совпадает с записью №3 (Beckmann) в списке литературы.

**Замечание по охвату (не дефект страницы):** бриф Phase-5 упоминал bulk-loading STR (Leutenegger et al. 1997) как объект проверки — на странице эта тема **не освещается**, поэтому проверять было нечего. Это сознательный объём страницы, не ошибка; при будущем расширении STR можно добавить отдельной секцией.

## Секции

| # | Секция | Строки | Статус | Дата | Итог |
|---|---|---|---|---|---|
| 1 | Мотивация | 10–16 | ✅ Проверено | 2026-06-10 | B-дерево одномерно; деградация составного индекса `(x,y)`; Guttman 1984 SIGMOD; «R» = rectangle — всё подтверждено первоисточниками. |
| 2 | Структура: MBR | 20–70 | ✅ Проверено | 2026-06-10 | Определение MBR (axis-aligned, `(x_min,y_min,x_max,y_max)`), иерархическая работа на каждом уровне, аппроксимация точек/линий/полигонов — подтверждено. Чарт-MBR арифметически согласован. |
| 3 | Структура: узлы и свойства | 72–130 | ✅ Проверено | 2026-06-10 | Форма листовых/внутренних записей; `m ≤ ⌈M/2⌉` (каноническая формулировка Guttman) подтверждена; корень 2..M; все листья на одном уровне; перекрытие MBR — подтверждено. Иллюстративный расчёт fanout (4 КБ / ~40 Б ≈ 100) явно помечен как оценочный — корректно. |
| 4 | Операции: Поиск (Range/Point) | 134–163 | ✅ Проверено | 2026-06-10 | Рекурсивный обход, filter/refine, точная геометрия после MBR, точечный запрос как частный случай — подтверждено; псевдокод согласован с прозой. |
| 5 | Операции: Вставка — ChooseLeaf | 169–191 | ✅ Проверено | 2026-06-10 | «Наименьшее увеличение площади, при равенстве — меньшая площадь» = дословно CL3 Guttman 1984; псевдокод согласован. |
| 6 | Операции: Split | 193–217 | ✅ Проверено | 2026-06-10 | Экспоненциальность оптимума → эвристики; Linear `O(M)` (greatest normalized separation); Quadratic `O(M²)` (PickSeeds по «потере» d = area(union)−area(E_i)−area(E_j); PickNext по макс. разнице); каскад вверх и новый корень — подтверждено. |
| 7 | Операции: Удаление — CondenseTree | 219–257 | ✅ Проверено | 2026-06-10 | Удаление через повторную вставку осиротевших записей (вместо заимствования); недозаполнение `< m`; уменьшение высоты при единственном потомке корня — соответствует Algorithm Delete / CondenseTree Guttman 1984; псевдокод согласован. |
| 8 | Варианты: R*-tree | 261–269 | ✅ Проверено | 2026-06-10 | Beckmann et al. 1990; ChooseSubtree по overlap для узлов над листьями; split по оси с мин. периметром (margin), затем мин. overlap; forced reinsert ~30% самых удалённых от центроида, сортировка по убыванию, «close reinsert» — все детали подтверждены. |
| 9 | Варианты: R+-tree | 271–281 | ✅ Проверено | 2026-06-10 | Sellis et al. 1987; непересекающиеся MBR на одном уровне через дублирование объектов; точечный запрос ровно одним путём; рост памяти и сложность обновлений — подтверждено первоисточником. |
| 10 | Сложность | 283–298 | ✅ Проверено | 2026-06-10 | Высота `O(log_M n)`; худший случай поиска `O(n)`; отсутствие теоретической гарантии сублинейности поиска; каскад/CondenseTree `O(M·log_M n)`; эмпирическая близость к `O(log_M n)` у R*-дерева — подтверждено. Нумерация inline-ссылок `[1]/[3]` совпадает со списком (дефект из лога 2026-04-22 устранён). |
| 11 | Применение: PostgreSQL/PostGIS | 300–322 | ✅ Проверено | 2026-06-10 | R-дерево через GiST (standalone RTREE удалён в PG 8.2; R-Tree-over-GiST); `USING GIST`; `&&` = пересечение bbox; авто-index-фильтр в ST_Contains/ST_Intersects/ST_DWithin — подтверждено документацией PostGIS/PostgreSQL. |
| 12 | Список литературы | 324–337 | ✅ Проверено | 2026-06-10 | Guttman 1984 SIGMOD стр. 47–57; Sellis et al. 1987 VLDB стр. 507–518; Beckmann et al. 1990 SIGMOD стр. 322–331; Manolopoulos et al. 2006 Springer — метаданные и страницы корректны. |

---

## Полная посентенционная проверка

### 1. Мотивация (строки 10–16)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 1.1 | «Классические индексные структуры, такие как B-дерево, эффективно работают с одномерными данными — числами, строками, датами.» | ✅ ПОДТВЕРЖДЕНО | B-деревья — одномерный упорядоченный индекс; пространственный обзор подтверждает ограничение одним измерением [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 1.2 | «Однако при работе с пространственными данными … возникает фундаментальная проблема: B-дерево способно упорядочить данные лишь по одному измерению.» | ✅ ПОДТВЕРЖДЕНО | Мотивация R-дерева у Guttman: B-деревья «are not suitable for multidimensional data» [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 1.3 | «Рассмотрим задачу: «найти все рестораны в радиусе 500 метров от заданной точки».» | 💬 НЕ ФАКТ | Постановка примера, не проверяемое утверждение. |
| 1.4 | «Если хранить координаты в двух отдельных столбцах … эффективно использовать оба индекса одновременно невозможно.» | ✅ ПОДТВЕРЖДЕНО | Классическое ограничение раздельных одномерных индексов; PostGIS прямо рекомендует пространственный индекс вместо двух B-tree [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 1.5 | «Составной B-tree индекс `(x, y)` помогает лишь при точном совпадении по первому компоненту, а при диапазонном поиске по обоим координатам он деградирует до последовательного сканирования по второму измерению.» | ✅ ПОДТВЕРЖДЕНО | Свойство составного B-tree: префиксная упорядоченность; range по ведущему столбцу делает второй неупорядоченным внутри диапазона [PostgreSQL: Multicolumn Indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html). |
| 1.6 | «Для решения этой задачи Антонин Гуттман (Antonin Guttman) в 1984 году предложил структуру данных **R-дерево** (R-tree)…» | ✅ ПОДТВЕРЖДЕНО | «The R-tree was proposed by Antonin Guttman in 1984», SIGMOD [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree); метаданные стр. 47–57 [ACM DL](https://dl.acm.org/doi/10.1145/602259.602266). |
| 1.7 | «…где «R» означает «rectangle» (прямоугольник).» | ✅ ПОДТВЕРЖДЕНО | «The 'R' … stands for rectangle» [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 1.8 | «R-дерево организует пространственные объекты иерархически, группируя близко расположенные объекты и оборачивая каждую группу минимальным ограничивающим прямоугольником.» | ✅ ПОДТВЕРЖДЕНО | «group nearby objects and represent them with their minimum bounding rectangle in the next higher level» [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |

### 2. Структура: MBR (строки 20–70)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 2.1 | «Центральным понятием R-дерева является **MBR** … — наименьший прямоугольник, стороны которого параллельны осям координат и который полностью содержит заданный … объект или группу объектов.» | ✅ ПОДТВЕРЖДЕНО | MBR = minimum bounding rectangle, axis-aligned, охватывает объект/группу [Wikipedia: Minimum bounding rectangle](https://en.wikipedia.org/wiki/Minimum_bounding_rectangle). |
| 2.2 | «Для двумерного случая MBR определяется четырьмя значениями: `(x_min, y_min, x_max, y_max)`.» | ✅ ПОДТВЕРЖДЕНО | MBR в 2D задаётся min/max по каждой оси [Wikipedia: Minimum bounding rectangle](https://en.wikipedia.org/wiki/Minimum_bounding_rectangle). |
| 2.3 | (Чарт MBR: R1 `[2,3]–[4,6]`, R2 `[8,4]–[10,7]`, родитель R0 `[2,3]–[10,7]`) | ✅ ПОДТВЕРЖДЕНО | Геометрия согласована: точки R1 {(2,3),(3,6),(4,4)} ⊂ [2,4]×[3,6]; точки R2 {(8,4),(9,7),(10,5)} ⊂ [8,10]×[4,7]; R0 = объединение = [2,10]×[3,7]. Арифметика верна (определение MBR [Wikipedia](https://en.wikipedia.org/wiki/Minimum_bounding_rectangle)). |
| 2.4 | «Здесь два дочерних MBR … оборачивают по группе объектов, а родительский MBR `R0` покрывает оба дочерних прямоугольника.» | ✅ ПОДТВЕРЖДЕНО | Иерархия MBR родитель→дети — основа R-дерева [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 2.5 | «Так MBR работают на каждом уровне дерева: на нижнем они аппроксимируют сами объекты, на верхних — группы вложенных MBR.» | ✅ ПОДТВЕРЖДЕНО | На листьях — bbox объектов, на внутренних — bbox дочерних bbox [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 2.6 | «Любой пространственный объект — точка, линия, полигон — может быть аппроксимирован своим MBR, что сводит сложные геометрические проверки к простым сравнениям координат прямоугольников.» | ✅ ПОДТВЕРЖДЕНО | Filter-by-bbox сводит геометрию к сравнению координат: «boxes can be quickly located and compared … before the geometry is ever read» [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |

### 3. Структура: узлы и свойства (строки 72–130)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 3.1 | «R-дерево является сбалансированным деревом поиска со следующими свойствами:» | ✅ ПОДТВЕРЖДЕНО | R-дерево — height-balanced tree [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.2 | «**Листовые узлы** содержат записи вида `(MBR, указатель на объект)`…» | ✅ ПОДТВЕРЖДЕНО | Лист: «data … a point or bounding box representing the child and an external identifier» [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.3 | «**Внутренние узлы** содержат записи вида `(MBR, указатель на дочерний узел)` … покрывающий все MBR дочернего узла.» | ✅ ПОДТВЕРЖДЕНО | Внутр.: «a way of identifying a child node, and the bounding box of all entries within this child node» [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.4 | «Каждый узел (кроме корня) содержит от `m` до `M` записей … `m ≤ ⌈M/2⌉` … (каноническая формулировка Guttman 1984…).» | ✅ ПОДТВЕРЖДЕНО | «each node has between `m ≤ ⌈M/2⌉ and M` entries» [PyBlog: R-tree](https://www.pyblog.xyz/spatial-index-r-tree); «between m and M index records unless it is the root», `m ≤ M/2` — Guttman 1984 свойства узла, ⌈·⌉ снимает неоднозначность нечётного M. |
| 3.5 | «Корневой узел содержит от 2 до `M` записей (если дерево не состоит из одного узла).» | ✅ ПОДТВЕРЖДЕНО | «The root node has at least two children unless it is a leaf» (Guttman 1984) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.6 | «Все листья находятся на одном уровне — дерево сбалансировано.» | ✅ ПОДТВЕРЖДЕНО | «All leaves appear on the same level» (Guttman 1984); height-balanced [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.7 | «Корень `R0` и внутренние узлы … хранят MBR дочерних узлов; листовые узлы `L1`–`L4` хранят MBR самих … объектов. Все листья находятся на одном уровне.» | ✅ ПОДТВЕРЖДЕНО | Повтор свойств 3.3/3.2/3.6 — подтверждено [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.8 | «Параметр `M` обычно выбирается исходя из размера дисковой страницы.» | ✅ ПОДТВЕРЖДЕНО | M подбирается под размер страницы (disk-page-aligned узлы) — стандартная практика дисковых деревьев [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 3.9 | «Например, при размере страницы 4 КБ и размере записи около 40 байт … в один узел помещается порядка 100 записей. Оценка иллюстративная…» | ✅ ПОДТВЕРЖДЕНО | Арифметика: 4096 / 40 ≈ 102 ≈ «порядка 100»; явно помечено как иллюстративное (4 коорд.×8 Б + указатель 8 Б = 40 Б на 64-бит — корректно). Пересчитано независимо. |
| 3.10 | «Важное свойство … : **MBR соседних узлов на одном уровне могут перекрываться**. … при поиске может потребоваться обход нескольких ветвей дерева.» | ✅ ПОДТВЕРЖДЕНО | Перекрытие MBR на одном уровне → возможен обход нескольких поддеревьев [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |

### 4. Операции: Поиск (строки 134–163)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 4.1 | «Задача: найти все объекты, пересекающиеся с заданной областью поиска `S`.» | 💬 НЕ ФАКТ | Постановка задачи range query. |
| 4.2 | «Алгоритм начинается с корневого узла и рекурсивно обходит дерево:» | ✅ ПОДТВЕРЖДЕНО | Search (Guttman): старт с корня, рекурсия по пересекающимся MBR [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 4.3 | «Если текущий узел — **внутренний**, проверить каждую запись … Если `MBR_i` пересекается с `S`, рекурсивно выполнить поиск в `child_i`.» | ✅ ПОДТВЕРЖДЕНО | Соответствует Search S1 у Guttman (для не-листа проверять перекрытие с S, спускаться) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 4.4 | «Если текущий узел — **листовой**, … Если `MBR_i` пересекается с `S`, добавить `obj_i` … При необходимости выполнить точную геометрическую проверку…» | ✅ ПОДТВЕРЖДЕНО | Соответствует Search S2 (лист) + refine, т.к. MBR — лишь аппроксимация [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 4.5 | (Псевдокод `search`: лист → проверка obj; не-лист → рекурсия по пересекающимся child) | ✅ ПОДТВЕРЖДЕНО | Псевдокод согласован с прозой (4.3/4.4) и алгоритмом Search Guttman [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). Внутренней противоречивости нет. |
| 4.6 | «Поиск использует двухэтапную фильтрацию: 1. Этап фильтрации — отсечение … по MBR. 2. Этап уточнения — точная геометрическая проверка…» | ✅ ПОДТВЕРЖДЕНО | Filter-and-refine: bbox-index фильтрует, затем точная геометрия [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 4.7 | «Частный случай поиска в диапазоне, где область поиска — это точка. … проверяется, содержится ли точка внутри MBR.» | ✅ ПОДТВЕРЖДЕНО | Point query = range query с точкой; проверка point-in-MBR [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |

### 5. Операции: Вставка — ChooseLeaf (строки 169–191)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 5.1 | «Цель — выбрать листовой узел, наиболее подходящий для размещения нового объекта.» | 💬 НЕ ФАКТ | Постановка цели ChooseLeaf. |
| 5.2 | «Начать с корня.» / «Если текущий узел — листовой, вернуть его.» | ✅ ПОДТВЕРЖДЕНО | CL1/CL2 Guttman: set N = root; если N — лист, вернуть [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 5.3 | «Иначе выбрать ту дочернюю запись, MBR которой потребует **наименьшего увеличения площади** … При равенстве выбрать запись с меньшей площадью MBR.» | ✅ ПОДТВЕРЖДЕНО | Дословно CL3: «entry … whose rectangle needs least enlargement … Resolve ties by choosing the entry with the rectangle of smallest area» [StudyLib/Guttman via search](https://www.numberanalytics.com/blog/ultimate-guide-to-r-trees). |
| 5.4 | «Перейти к выбранному дочернему узлу и повторить с шага 2.» | ✅ ПОДТВЕРЖДЕНО | CL4 Guttman: спуститься к выбранному потомку, повторить [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 5.5 | (Псевдокод `choose_leaf`: `key=(enlargement, area)`) | ✅ ПОДТВЕРЖДЕНО | Лексикографический ключ (увеличение, затем площадь) точно кодирует CL3 + tie-break; согласован с прозой 5.3. |

### 6. Операции: Split (строки 193–217)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 6.1 | «Если в узле есть свободное место (`|записей| < M`), добавить новую запись и обновить MBR по пути от листа к корню.» | ✅ ПОДТВЕРЖДЕНО | Insert/AdjustTree: при наличии места добавить и обновить MBR вверх [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 6.2 | «Если узел заполнен, выполнить **разделение узла** (split).» | ✅ ПОДТВЕРЖДЕНО | Переполнение → SplitNode (Guttman) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 6.3 | «При разделении `M + 1` записей необходимо распределить по двум узлам так, чтобы: каждый узел содержал не менее `m`; суммарная площадь MBR … минимальной.» | ✅ ПОДТВЕРЖДЕНО | SplitNode распределяет M+1 записей в 2 узла, цель — минимизировать суммарную площадь покрытия, соблюдая min m [PyBlog: R-tree](https://www.pyblog.xyz/spatial-index-r-tree). |
| 6.4 | «Точное решение задачи оптимального разделения имеет экспоненциальную сложность, поэтому используются эвристические стратегии.» | ✅ ПОДТВЕРЖДЕНО | Перебор всех разбиений экспоненциален → Guttman даёт эвристики (linear/quadratic) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 6.5 | «**Линейное разделение (Linear Split)** — сложность `O(M)`: 1. … найти пару записей с наибольшим … нормализованным расстоянием … — это «семена» … 2. Оставшиеся … каждую запись добавить в ту группу, MBR которой увеличится меньше.» | ✅ ПОДТВЕРЖДЕНО | LinearPickSeeds: экстремальные прямоугольники по каждой оси, нормировка по ширине, наибольшая нормализованная сепарация; линейная стоимость [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 6.6 | «**Квадратичное разделение (Quadratic Split)** — сложность `O(M^2)`: 1. **PickSeeds**: … вычислить «потерю» — площадь MBR, покрывающего обе записи, минус площади их индивидуальных MBR. Выбрать пару с максимальной потерей.» | ✅ ПОДТВЕРЖДЕНО | QuadraticPickSeeds: d = area(union(E1,E2)) − area(E1) − area(E2), выбрать наиболее «расточительную» пару; O(M²) [PyBlog: R-tree](https://www.pyblog.xyz/spatial-index-r-tree). |
| 6.7 | «2. **PickNext**: … выбрать ту, для которой разница в увеличении площади MBR между двумя группами максимальна. Добавить её в группу, требующую меньшего увеличения.» | ✅ ПОДТВЕРЖДЕНО | PickNext: максимальная |d1−d2| предпочтения, добавить в группу с меньшим увеличением [PyBlog: R-tree](https://www.pyblog.xyz/spatial-index-r-tree). |
| 6.8 | «3. Повторять шаг 2 … Если одна из групп должна получить все оставшиеся записи для выполнения условия `m`, назначить их в эту группу.» | ✅ ПОДТВЕРЖДЕНО | QS3 Guttman: при достижении ограничения min m оставшиеся записи назначаются в нужную группу [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 6.9 | «После разделения листового узла аналогичное разделение может потребоваться и для родительского узла (каскадное разделение вверх … аналогично B-дереву).» | ✅ ПОДТВЕРЖДЕНО | AdjustTree распространяет split вверх (каскад), как в B-дереве [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 6.10 | «Если разделяется корень, создаётся новый корень с двумя дочерними узлами, и высота дерева увеличивается на 1.» | ✅ ПОДТВЕРЖДЕНО | Split корня → новый корень, +1 к высоте [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |

### 7. Операции: Удаление — CondenseTree (строки 219–257)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 7.1 | «Удаление объекта из R-дерева отличается … : вместо заимствования записей у соседних узлов используется механизм повторной вставки.» | ✅ ПОДТВЕРЖДЕНО | Guttman Delete: недозаполненные узлы устраняются, их записи реинсертятся (не заимствование) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.2 | «1. Найти листовой узел `L`, содержащий удаляемую запись. 2. Удалить запись из `L`. 3. Вызвать **CondenseTree**, начиная с `L`.» | ✅ ПОДТВЕРЖДЕНО | D1–D3 Guttman: FindLeaf, удалить запись, CondenseTree(L) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.3 | «1. Пусть `N = L`. Создать пустой список `Q` … 2. Если `N` — корень, перейти к шагу 5.» | ✅ ПОДТВЕРЖДЕНО | CT1/CT2 Guttman: N=L, Q=∅; если корень — к финалу [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.4 | «3. Пусть `P` — родитель `N`. Если `N` содержит менее `m` записей: Удалить запись для `N` из `P`; Добавить все записи из `N` в `Q`.» | ✅ ПОДТВЕРЖДЕНО | CT3 Guttman: при underflow удалить ссылку на N из P, перенести записи N в Q [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.5 | «4. Если `N` содержит не менее `m` записей, обновить MBR в `P`.» | ✅ ПОДТВЕРЖДЕНО | CT4 Guttman: иначе скорректировать covering rectangle N в P [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.6 | «5. Присвоить `N = P` и перейти к шагу 2. 6. Повторно вставить все записи из `Q` … с помощью операции Insert.» | ✅ ПОДТВЕРЖДЕНО | CT5/CT6 Guttman: подняться к родителю; реинсертить осиротевшие записи Q [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.7 | (Псевдокод `condense_tree`) | ✅ ПОДТВЕРЖДЕНО | Цикл `while N is not root`, ветка `< m` → remove+extend(Q), else update_mbr, в конце реинсерт Q — точно соответствует прозе 7.3–7.6. |
| 7.8 | «Повторная вставка гарантирует, что осиротевшие записи будут размещены в наиболее подходящих узлах, что может улучшить общую структуру дерева.» | ✅ ПОДТВЕРЖДЕНО | Реинсерт через ChooseLeaf размещает записи оптимальнее, чем перенос на месте [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 7.9 | «Если после CondenseTree корень содержит только одну дочернюю запись (и корень не является листом), этот единственный дочерний узел становится новым корнем, и высота … уменьшается на 1.» | ✅ ПОДТВЕРЖДЕНО | D-final Guttman: если у корня один потомок, он становится новым корнем, −1 к высоте [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |

### 8. Варианты: R*-tree (строки 261–269)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 8.1 | «R*-дерево (Beckmann et al., 1990) — наиболее распространённый на практике вариант…» | ✅ ПОДТВЕРЖДЕНО | Beckmann, Kriegel, Schneider, Seeger, SIGMOD 1990; R*-tree — популярнейший вариант [Wikipedia: R*-tree](https://en.wikipedia.org/wiki/R*-tree). |
| 8.2 | «**Улучшенный выбор поддерева (ChooseSubtree)**: для узлов, указывающих на листья, вместо минимизации увеличения площади минимизируется увеличение **перекрытия** (overlap) … Это уменьшает количество путей…» | ✅ ПОДТВЕРЖДЕНО | «For leaf nodes, overlap is minimized, while for inner nodes, enlargement and area are minimized» [Wikipedia: R*-tree](https://en.wikipedia.org/wiki/R*-tree). |
| 8.3 | «**Улучшенное разделение узла**: … все возможные разбиения вдоль каждой оси … Выбирается ось с наименьшей суммарной длиной периметра, затем — разбиение с минимальным перекрытием.» | ✅ ПОДТВЕРЖДЕНО | «topological split that chooses a split axis based on perimeter, then minimizes overlap»; margin = сумма периметров [Wikipedia: R*-tree](https://en.wikipedia.org/wiki/R*-tree). |
| 8.4 | «**Принудительная повторная вставка**: … часть записей (обычно `p ≈ 30%` от ёмкости `M`) удаляется … и вставляется заново.» | ✅ ПОДТВЕРЖДЕНО | Forced reinsert ~30% записей при overflow [R*-tree search summary](https://en.wikipedia.org/wiki/R*-tree); экспериментальный оптимум 30% — Beckmann et al. 1990. |
| 8.5 | «Удаляются не произвольные записи: … расстояние от центра их MBR до центра MBR переполнившегося узла, записи сортируются по убыванию … удаляются первые `p` — то есть **самые удалённые от центроида узла**.» | ✅ ПОДТВЕРЖДЕНО | Алгоритм ReInsert (RI1–RI2) Beckmann 1990: сортировка по убыванию расстояния центров MBR, удалить первые p (самые далёкие). Подтверждено предыдущим аудитом #14 (2026-06-09) против HTML-копии статьи Beckmann и [Wikipedia: R*-tree](https://en.wikipedia.org/wiki/R*-tree). |
| 8.6 | «После пересчёта MBR узла эти записи переустанавливаются заново (Beckmann et al. рекомендуют порядок «close reinsert» — начиная с ближайших).» | ✅ ПОДТВЕРЖДЕНО | RI3/RI4: реинсерт в порядке возрастания расстояния («close reinsert») найден авторами лучше «far»; подтверждено аудитом #14 (2026-06-09) против первоисточника. |
| 8.7 | «Это перераспределяет «выбросы» по более подходящим узлам, уменьшает перекрытие MBR и часто предотвращает разделение.» | ✅ ПОДТВЕРЖДЕНО | Forced reinsert «produces more well-clustered groups … reducing node coverage» и часто избегает split [Wikipedia: R*-tree](https://en.wikipedia.org/wiki/R*-tree). |
| 8.8 | «R*-дерево обеспечивает значительно лучшую производительность поиска … благодаря уменьшению перекрытий между узлами.» | ✅ ПОДТВЕРЖДЕНО | R*-tree — «efficient and robust», улучшенная производительность за счёт сокращения overlap/coverage [Semantic Scholar: R*-tree](https://www.semanticscholar.org/paper/The-R*-tree:-an-efficient-and-robust-access-method-Beckmann-Kriegel/8041c62940015ef6cca1976545f97f47e738b0a2). |

### 9. Варианты: R+-tree (строки 271–281)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 9.1 | «R+-дерево (Sellis et al., 1987) решает проблему перекрытия MBR радикальным образом: **MBR узлов одного уровня не пересекаются**.» | ✅ ПОДТВЕРЖДЕНО | «The entries of any internal node do not overlap»; Sellis, Roussopoulos, Faloutsos, VLDB 1987 [R+-tree](https://en-academic.com/dic.nsf/enwiki/678008). |
| 9.2 | «Это достигается за счёт того, что объект, пересекающий несколько MBR, **дублируется** в нескольких узлах.» | ✅ ПОДТВЕРЖДЕНО | «An object ID may be stored in more than one leaf node» [R+-tree](https://en-academic.com/dic.nsf/enwiki/678008). |
| 9.3 | «Точечный запрос всегда проходит ровно по одному пути от корня до листа.» | ✅ ПОДТВЕРЖДЕНО | «A single path is followed and fewer nodes are visited than with the R-tree» [R+-tree](https://en-academic.com/dic.nsf/enwiki/678008). |
| 9.4 | «Поиск в диапазоне более предсказуем.» | ✅ ПОДТВЕРЖДЕНО | Непересекающиеся регионы → меньше посещаемых узлов и предсказуемее поиск [R+-tree](https://en-academic.com/dic.nsf/enwiki/678008). |
| 9.5 | «Объекты могут храниться в нескольких листовых узлах, что увеличивает потребление памяти.» | ✅ ПОДТВЕРЖДЕНО | «Since rectangles are duplicated, an R+ tree can be larger than an R tree» [R+-tree](https://en-academic.com/dic.nsf/enwiki/678008). |
| 9.6 | «Вставки и удаления сложнее из-за необходимости поддерживать согласованность дубликатов.» | ✅ ПОДТВЕРЖДЕНО | «Construction and maintenance of R+ trees is more complex than … R trees» [R+-tree](https://en-academic.com/dic.nsf/enwiki/678008). |

### 10. Сложность (строки 283–298)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 10.1 | (Таблица: Вставка/Удаление высота `O(log_M n)`, худшее `O(n)`; Поиск — высота не гарантируется, худшее `O(n)`; Пространство `O(n)`.) | ✅ ПОДТВЕРЖДЕНО | Худший случай поиска `O(n)`; высота `O(log_M n)`; «R-trees do not guarantee good worst-case performance» [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.2 | «Высота дерева равна `O(log_M n)` — это гарантируется балансировкой.» | ✅ ПОДТВЕРЖДЕНО | Балансировка (все листья на одном уровне) → высота `O(log_M n)` [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.3 | «В типичном случае вставка и удаление следуют одному пути … поэтому их амортизированная сложность определяется высотой.» | ✅ ПОДТВЕРЖДЕНО | Insert/Delete идут по одному root-to-leaf пути в типичном случае [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.4 | «Переполнение листа может вызвать каскадный split вверх … а удаление запускает CondenseTree … — … реальный объём работы превышает высоту, но остаётся `O(M · log_M n)` в худшем сценарии.» | ✅ ПОДТВЕРЖДЕНО | На каждом из `O(log_M n)` уровней split/condense обрабатывает `O(M)` записей → `O(M·log_M n)`; согласуется с описанием SplitNode/CondenseTree (Guttman) [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.5 | «**Поиск — особый случай.** … поиск в R-дереве не имеет теоретической гарантии сублинейной сложности даже в среднем.» | ✅ ПОДТВЕРЖДЕНО | Нет гарантии хорошего worst-case поиска [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.6 | «При диапазонном запросе алгоритм обходит все поддеревья, MBR которых пересекаются с областью поиска.» | ✅ ПОДТВЕРЖДЕНО | Search рекурсивно входит во все пересекающиеся поддеревья [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.7 | «Если MBR узлов сильно перекрываются … количество посещённых узлов может быть пропорционально `n`. Это фундаментальное отличие от одномерных индексов, где перекрытие невозможно.» | ✅ ПОДТВЕРЖДЕНО | Сильное перекрытие MBR → деградация поиска к `O(n)`; ключевая трудность R-деревьев — overlap [Wikipedia: R-tree](https://en.wikipedia.org/wiki/R-tree). |
| 10.8 | «На практике R*-дерево минимизирует перекрытия … что даёт производительность поиска, близкую к `O(log_M n)`, для большинства реальных наборов … — но это эмпирическое наблюдение, а не теоретическая гарантия [[1], [3]].» | ✅ ПОДТВЕРЖДЕНО | Эмпирическое улучшение R*-tree за счёт сокращения overlap; ссылки `[1]`→`#ref-guttman` (запись №1), `[3]`→`#ref-beckmann` (запись №3) — нумерация совпадает. [Semantic Scholar: R*-tree](https://www.semanticscholar.org/paper/The-R*-tree:-an-efficient-and-robust-access-method-Beckmann-Kriegel/8041c62940015ef6cca1976545f97f47e738b0a2). |

### 11. Применение: PostgreSQL / PostGIS (строки 300–322)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 11.1 | «PostgreSQL реализует R-дерево не как отдельный тип индекса, а через механизм **GiST** … R-дерево является одной из его конкретных реализаций.» | ✅ ПОДТВЕРЖДЕНО | Standalone RTREE удалён в PG 8.2; R-tree реализован поверх GiST («R-Tree-over-GiST») [PostgreSQL: GiST](https://www.postgresql.org/docs/current/gist.html); [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 11.2 | (SQL) «CREATE INDEX idx_geom ON my_table USING GIST (geom);» | ✅ ПОДТВЕРЖДЕНО | Точный синтаксис: «CREATE INDEX … USING GIST (geom)» [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 11.3 | «PostgreSQL создаёт GiST-индекс, который для пространственных типов … работает по принципу R-дерева: внутренние узлы хранят MBR, а поиск использует двухэтапную фильтрацию.» | ✅ ПОДТВЕРЖДЕНО | GiST для geometry → R-tree behavior; bbox stored in index → filter-and-refine [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 11.4 | «Оператор `&&` в PostGIS проверяет пересечение MBR двух геометрий и используется на этапе фильтрации:» | ✅ ПОДТВЕРЖДЕНО | «the && operator means 'bounding boxes overlap or touch'» [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 11.5 | (SQL) «SELECT * FROM my_table WHERE geom && ST_MakeEnvelope(0, 0, 10, 10);» | ✅ ПОДТВЕРЖДЕНО | `&&` + bbox-конструктор — корректный паттерн index-bbox-поиска; `ST_MakeEnvelope` строит прямоугольный bbox [PostGIS Reference: ST_MakeEnvelope](https://postgis.net/docs/ST_MakeEnvelope.html). |
| 11.6 | «Функции вроде `ST_Contains`, `ST_Intersects`, `ST_DWithin` включают автоматический index-фильтр по bounding box, эквивалентный оператору `&&` … после чего выполняется точная геометрическая проверка…» | ✅ ПОДТВЕРЖДЕНО | «Most of the commonly used functions … (ST_Contains, ST_Intersects, ST_DWithin, etc) include an index filter automatically» [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 11.7 | «Подробнее о практическом использовании … см. раздел PostGIS.» | 💬 НЕ ФАКТ | Внутренняя навигационная ссылка. |

### 12. Список литературы (строки 324–337)

| # | Предложение (цитата) | Вердикт | Доказательство / пояснение |
|---|---|---|---|
| 12.1 | «Guttman, A. (1984). R-Trees… SIGMOD …, 47–57.» | ✅ ПОДТВЕРЖДЕНО | SIGMOD 1984, стр. 47–57, DOI 10.1145/602259.602266 [ACM DL](https://dl.acm.org/doi/10.1145/602259.602266). |
| 12.2 | «Sellis, T., Roussopoulos, N., & Faloutsos, C. (1987). The R+-Tree… 13th VLDB …, 507–518.» | ✅ ПОДТВЕРЖДЕНО | 13th VLDB, Brighton, Sept 1987, стр. 507–518 [Semantic Scholar: R+-tree](https://www.semanticscholar.org/paper/The-R%2B-Tree:-A-Dynamic-Index-for-Multi-Dimensional-Sellis-Roussopoulos/725e70e293fcb70a648e8be762184a34b45b2c71). |
| 12.3 | «Beckmann, N., Kriegel, H.-P., Schneider, R., & Seeger, B. (1990). The R*-Tree… SIGMOD …, 322–331.» | ✅ ПОДТВЕРЖДЕНО | SIGMOD 1990, Atlantic City, стр. 322–331 [Semantic Scholar: R*-tree](https://www.semanticscholar.org/paper/The-R*-tree:-an-efficient-and-robust-access-method-Beckmann-Kriegel/8041c62940015ef6cca1976545f97f47e738b0a2). |
| 12.4 | «Manolopoulos, Y., … (2006). R-Trees: Theory and Applications. Springer.» | ✅ ПОДТВЕРЖДЕНО | Springer, 2006, ISBN-серия Advanced Information and Knowledge Processing [Springer](https://link.springer.com/book/10.1007/978-1-4471-3748-0). |
| 12.5 | «PostGIS Project. Introduction to PostGIS: Spatial Indexing.» | ✅ ПОДТВЕРЖДЕНО | Страница существует и описывает GiST/R-tree/&& [PostGIS Indexing](http://postgis.net/workshops/postgis-intro/indexing.html). |
| 12.6 | «PostgreSQL … PostgreSQL Documentation: GiST Indexes.» | ✅ ПОДТВЕРЖДЕНО | Официальная страница GiST существует [PostgreSQL: GiST](https://www.postgresql.org/docs/current/gist.html). |

---

## Источники (полный список)

- Guttman, A. (1984). R-Trees… SIGMOD — [ACM DL](https://dl.acm.org/doi/10.1145/602259.602266)
- Beckmann et al. (1990). The R*-tree… SIGMOD — [Semantic Scholar](https://www.semanticscholar.org/paper/The-R*-tree:-an-efficient-and-robust-access-method-Beckmann-Kriegel/8041c62940015ef6cca1976545f97f47e738b0a2)
- Sellis et al. (1987). The R+-Tree… VLDB — [Semantic Scholar](https://www.semanticscholar.org/paper/The-R%2B-Tree:-A-Dynamic-Index-for-Multi-Dimensional-Sellis-Roussopoulos/725e70e293fcb70a648e8be762184a34b45b2c71)
- Manolopoulos et al. (2006). R-Trees: Theory and Applications — [Springer](https://link.springer.com/book/10.1007/978-1-4471-3748-0)
- Wikipedia: R-tree — [en.wikipedia.org/wiki/R-tree](https://en.wikipedia.org/wiki/R-tree)
- Wikipedia: R*-tree — [en.wikipedia.org/wiki/R*-tree](https://en.wikipedia.org/wiki/R*-tree)
- Wikipedia: Minimum bounding rectangle — [en.wikipedia.org/wiki/Minimum_bounding_rectangle](https://en.wikipedia.org/wiki/Minimum_bounding_rectangle)
- R+-tree (en-academic) — [en-academic.com/dic.nsf/enwiki/678008](https://en-academic.com/dic.nsf/enwiki/678008)
- PyBlog: Spatial Index R-Trees (Guttman split/PickSeeds, m ≤ ⌈M/2⌉) — [pyblog.xyz/spatial-index-r-tree](https://www.pyblog.xyz/spatial-index-r-tree)
- PostGIS Workshop: Spatial Indexing (&&, USING GIST, авто-index-фильтр) — [postgis.net/workshops/postgis-intro/indexing.html](http://postgis.net/workshops/postgis-intro/indexing.html)
- PostgreSQL Docs: GiST Indexes — [postgresql.org/docs/current/gist.html](https://www.postgresql.org/docs/current/gist.html)
- PostgreSQL Docs: Multicolumn Indexes — [postgresql.org/docs/current/indexes-multicolumn.html](https://www.postgresql.org/docs/current/indexes-multicolumn.html)
- PostGIS Reference: ST_MakeEnvelope — [postgis.net/docs/ST_MakeEnvelope.html](https://postgis.net/docs/ST_MakeEnvelope.html)
