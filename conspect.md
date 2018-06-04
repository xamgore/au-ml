**Baseline** — первый метод, который обычно делается. Зачем его делать? Чтобы потом сравниваться с baseline'ом, а потом уже применять что-то более сложное.

**Ленивое обучение** — мы не придумываем аппроксимирующую функцию, а просто запоминаем датасет и придумываем какое-то правило — отношение к датасету. Например, kNN.

**Ядро** — $K(r)$, не возрастает и положительно на $[0, 1]$.

### Метрические классификаторы. kNN. WkNN. Отбор эталонов. DROP5. Kdtree.

> [Воронцов](http://www.machinelearning.ru/wiki/images/c/c3/Voron-ML-Metric-slides.pdf)

Задача классификации: есть $X$ — множество описаний объектов, $Y$ — множество классов. Существует целевая функция $X \to Y$, значения которой известны на объектах обучающей выборки $D = \{(\vec x_1, y_1) .. (\vec x_n, y_n)\}$. Требуется построить алгоритм, способный классифицировать произвольный объект $\vec x \in X$.

Предположим гипотезу компактности — близкие объекты, как правило, лежат в одном классе. Тогда в метрических пространствах можно построить следующие классификаторы.

##### Метрический алгоритм классификации

Класс нового объекта определяется по суммарному весу каждого класса:

$$h(x, D) = \arg \max\limits_{y\ \in\ Y} \underbrace{\sum\limits_{x_i \in D} [y_i = y]\ w(x_i, x)}_{\Gamma_y(x)} $$

$w(x_i, x)$ — вес (степень важности) соседа $x_i$
$\Gamma_y(x)$ — _близость_ объекта $x$ к классу $y$

##### kNN

* Voting: $w(x_i, x) = 1 \Leftrightarrow x_i$ — один из $k$ ближайших соседей

  * простота реализации (lazy learning)
  * $k$ можно оптимизировать по критерию скользящего контроля (leave-one-out):

  $$ LOO(k, D) = \sum\limits_{x_i \in D} [\ h_k(x_i, D - x_i) \neq y_i\ ]$$
  * неоднозначность классификации при $\Gamma_y(x) = \Gamma_s(x)$ (в сторону наиболее вероятного класса, или наименее болезненный)
  * не учитываются расстояния
  <br>

* Radius Neighbours: $w(x_i, x) = 1 \Leftrightarrow \rho(x_i, x) < R$

##### Weighted kNN

$k$ _взвешенных_ ближайших соседей. Варианты $w$:

* $w(x_i, x) = 1 - \dfrac{\rho(x_i, x)}{const}$
  можно ограничить снизу нулём или рассматривать точки в окружности радиуса $const$.
  <br>

* $w(x_i, x) = q^{-\rho(x_i, x)}$ — экспоненциально убывающие веса
  <br>

* метод парзеновского окна фиксированной ширины: $K\bigg(\dfrac{\rho(x_i, x)}{h}\bigg)$
  <br>

* метод парзеновского окна переменной ширины: $K\bigg(\dfrac{\rho(x_i, x)}{\rho(x^{(k+1)}, x)}\bigg)$


##### Отбор эталонов (Prototype selection)

$$M(x_i) = \Gamma_{y_i}(x_i) - \max\limits_{y\ \in\ Y -\ y_i} \Gamma_y(x_i)$$

Margin (отступ) — оценка того, насколько хорошо точка лежит в своём классе. То есть это разница между близостью правильного класса $y_i$ и наибольшей близостью чужого класса.

![](https://i.imgur.com/vYenfpJ.png)

Шумы — окружены другими классами, эталонные — сидят глубоко в своём классе, или на границе. В качестве эталонов можно брать точки с наибольшим отступом. Тогда классификацию можно делать по эталонам:

$$h(x, \red\Omega) = \arg \max\limits_{y\ \in\ Y} \sum\limits_{x_i \in\ \red\Omega} [y_i = y]\ w(x_i, x) $$

Методы отбора эталонов:

* По навправлению поиска
  * incremental — начинаем с пустого множества эталонов, и добавляем в него
  * decremental — множество эталонов это весь датасет и мы выкидываем из него точки
  * batch (decremental) — когда делаем несколько точек за раз
  * mixed — и выкидываем, и добавляем
  * fixed (mixed) — стараемся сохранить фиксированное количество эталонов
  * <mark>replacement</mark> — выдумываем новую точку

* <mark>По типу выбора</mark>
  * condenstation — выбираем точку, которая хорошо лежит внутри своего класса
  * edition — когда на границе
  * hybrid

##### DROP5

_Decremental Reduction Optimization Procedure_ — метод отбора эталонов. [Псевдокод](https://i.imgur.com/u5RIXWg.png). <mark>Как относится к предыдущей классификации?</mark> (похоже на hybrid)

1. Начинаем с полного набора обучающих точек.
2. Для каждой определим $distance$ — расстояние до ближайшей точки другого класса (близость к врагам).
3. Идем от наименьшего $distance$ к наибольшему.
4. Для каждой точки $P$, смотрим у каких точек $S$ она ближайший сосед
   - $with \mathrel:=$ кол-во правильно классифицированных точек из $S$
   - $without \mathrel:=$ кол-во правильно классифицированных точек из $S$ после удаления $P$
   - если $without \geqslant with$, удаляем $P$

##### Kdtree

Если даже эталонов осталось очень много, можно использовать kdtree — структуру, которая быстро находит ближайших соседей.

* Разбиваем пространство гиперплоскостями, ортогональными одной из координатных осей, последовательно по медианным точкам.

* Поиск соседей начинаем с точек листа, в котором находится точка, если соседей недостаточно – поднимаемся на узел выше.

  <img src="https://i.imgur.com/IcoYW27.png" width="350">


### Кластеризация. kMeans, MeanShift, DBSCAN, Affinity Propagation.

Кластеризация — разделение множества точек на кластеры по приниципу похожести, кластеризация — обучение без учителя.

Цели кластеризации:
* Упростить дальнейшую обработку данных, разбить на группы схожих объектов и работать с каждой в отдельности
* Сократить объём хранимых данных, оставив по одному представителю от каждого кластера
* Построение иерархии множества объектов (например, таксономия Линнея)
* Выделение нетипичных объектов и аномалий
* Получение новых признаков (если объекты имеют кластерную структуру сами по себе)

Хорошо, когда есть внешняя метрика, например, человек, который может оценить аномалии, или классификатор, который начинает работать лучше, если ему передать признак-кластер. Но если такого нет, то минимизируют расстояния до центроид:

$$ \sum\limits_{x_i}{ \min\limits_{\mu} \big|| x_i - \mu |\big|^2_2 } \to \min $$

##### Алгоритм k-means

1. Инициализируем центры кластеров (случайно или более хитрым образом)
2. Припишем каждую точку к ближайшему центру.
3. Переместим центры кластеров в «центр масс» кластеров:
  $$ \mu_i = \dfrac{1}{|C_i|}\ {\sum\limits_{x\ \in\ C_i}}\ {x} $$
4. Повторяем шаги 2-3 до схождения.

Проблемы:

* неизвестно изначальное кол-во кластеров (а вдруг нужно больше / меньше)
* зависимость от начального положения центроид
* не работает с несферическими (ленточными) кластерами

### Препроцессинг. Масштабирование. Нормировка. Полиномиальные признаки. One-hot encoding.

<!-- Из-за неравномерных осей kNN в данном случае будет работать плохо.

<img src="https://i.imgur.com/OBVF65y.png" width="350"> -->

##### One-hot encoding

Хорошо описано [здесь](https://hackernoon.com/what-is-one-hot-encoding-why-and-when-do-you-have-to-use-it-e3c6186d008f). Коротко: была таблица с категорным признаком, а мы его превратили в фичи. Нужно, чтобы не было проблем, когда модель считает, например, среднее $(1+3)/2 = 2$ и получает, что среднее VW и Honda — Acura.

```
╔════════════╦═════════════════╦════════╗
║ CompanyName Categoricalvalue ║ Price  ║
╠════════════╬═════════════════╣════════║
║ VW         ╬      1          ║ 20000  ║
║ Acura      ╬      2          ║ 10011  ║
║ Honda      ╬      3          ║ 50000  ║
║ Honda      ╬      3          ║ 10000  ║
╚════════════╩═════════════════╩════════╝

╔════╦══════╦══════╦════════╦
║ VW ║ Acura║ Honda║ Price  ║
╠════╬══════╬══════╬════════╬
║ 1  ╬ 0    ╬ 0    ║ 20000  ║
║ 0  ╬ 1    ╬ 0    ║ 10011  ║
║ 0  ╬ 0    ╬ 1    ║ 50000  ║
║ 0  ╬ 0    ╬ 1    ║ 10000  ║
╚════╩══════╩══════╩════════╝
```