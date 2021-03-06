---
layout: article
title: Начало с Altium Designer. Часть 2
modify_date: 2012-07-30
tags: altium manual old
image_path: /assets/images/2012-07-30-start-with-altium-designer-part2
---

Статья из моего старого блога: [https://kirik444.wordpress.com/2012/07/30/start-with-altium-designer-part-2/](https://kirik444.wordpress.com/2012/07/30/start-with-altium-designer-part-2/){:.info}

[Первая часть](/2012/05/28/start-with-altium-designer-part1.html)


![]({{page.image_path}}/00-logo.jpg){:.rounded}

еперь мы разведём плату под это суперсложное устройство.
Для упрощения создания заготовки, воспользуемся уже описанным здесь методом, который называется PCB Board Wizard.

Параметры заготовки должны быть следующие:

1. Форма -- прямоугольная, с размерами 50мм на 50 мм. Не должно быть легенды, размерных линий и штампа.
2. Сигнальных слоев -- 2, силовых -- 2.
3. Только сквозные отверстия.
4. Компоненты -- для дырочного монтажа. Между отверстиями -- максимум 2 дорожки.
5. Минимальный размер дорожки -- 0.5 мм. Размер контактной площадки -- 1.2 мм, отверстия -- 0.7 мм. Расстояние между соседними проводниками -- 0.5 мм

Теперь сохраним файл под именем board.PcbDoc, перетащим его в наш проект и откроем его.

<figure>
  <img src="{{page.image_path}}/01-new-board.png" alt="Рис 13. Заготовка для платы"/>
  <figcaption>Рис 13. Заготовка для платы</figcaption>
</figure>

Общий вид должен быть примерно такой. Чёрный квадрат и есть наша заготовка для платы. Но какая же плата без элементов? Добавим их. Делается это командой **Design → Import Changes From firstProject.PrjPcb**. Откроется следующее окно:

<figure>
  <img src="{{page.image_path}}/02-changes.png" alt="Рис 14. Список изменений"/>
  <figcaption>Рис 14. Список изменений</figcaption>
</figure>

В этом окне содержится список изменений, который будет применён к текущей плате. Легко видеть, что будут сделаны следующие изменения: будут добавлены 4 компонента и 4 связи между ними, 1 класс компонентов и 1 комната. В левой нижней части окна есть три кнопки:

1. Validate Changes -- кнопка для запуска процедуры проверки списка изменений. Может так быть, что на принципиальной схеме случайно будут 2 элемента с именами С1. На этапе проверки, это будет выявлено. И придётся исправлять ошибку. Нажмём на эту кнопку. Ошибок быть не должно и в колонке Check дожны появиться галки в зелёных кружках -- свидетельство отсутствия ошибок.
2. Execute Changes -- кнопка применения изменений. Нажмём на неё. На этот раз галки в кружка должны появиться в колонке Done. Если это так, то все изменения применены и элементы появились на плате.
3. Report Changes --нажатие генерирует лист для печати с табличкой, как на рисунке 14.

Теперь жмём **Close** и видим, что, помимо заготовки платы, на экране появилось еще что-то. Рассмотрим подробнее, что же появилось.

<figure>
  <img src="{{page.image_path}}/03-imported-changes.png" alt="Рис 15. Что добавилось"/>
  <figcaption>Рис 15. Что добавилось</figcaption>
</figure>

Итак, что мы имеем:

* 4 элемента. Они показаны контурами и названиями (R1, D1, …)
* Тонкие серые линии -- линии связи, показывающие, как должны быть соединены компоненты.
* Фиолетовая рамка и надпись schematic -- это граница и название комнаты элементов. О предназначении комнат будет рассказано в другой статье.

Расположим элементы следующим образом.

<figure>
  <img src="{{page.image_path}}/04-placement.png" alt="Рис 16. Расположение элементов"/>
  <figcaption>Рис 16. Расположение элементов</figcaption>
</figure>

И уже заметен тот факт, что заготовка для платы чересчур большая для таких элементов. Надо её уменьшить. Делается это просто, но перед этим произведём небольшую подготовку: установим начало координат и изменим шаг координатной сетки.

Для устновки начала координат последовательно нажимаем на клавиатуре буквы `E` → `O` → `S` (или выбираем в меню **Edit → Origin → Set**). После этого под указателем мышки появляется перекрестие, которое устанавливаем в левый нижний угол зелёного прямоугольника -- границы платы.

<figure>
  <img src="{{page.image_path}}/05-coordinate-system.png" alt="Рис 17. Начало СК"/>
  <figcaption>Рис 17. Начало СК</figcaption>
</figure>

Перекрестье и кружок говорят о том, что начало системы координат установлено. Теперь изменяем параметры координатной сетки. Если её не видно, то либо её шаг слишком большой, либо слишком мелкий. Для изменения шага нажимаем клавишу `G` и выбираем шаг в 1 мм. Согласитесь, что теперь гораздо удобнее.

<figure>
  <img src="{{page.image_path}}/06-grid.png" alt="Рис 18. Сетка"/>
  <figcaption>Рис 18. Сетка</figcaption>
</figure>

А вот теперь можем смело изменять размеры нашей платы. Для текущего расположения элементов она будет 26х17 мм. Начинаем.

Последовательно нажимаем на `D` → `S` → `R` (**Design → Board Shape → Redefine Board Shape**). Активируется режим переопределения размеров. Конечно, можно задавать размеры наугад, но у нас-то есть координатная сетка. Поэтому мы будем работать с ней. Нажимаем `J` → `L` (**Jump → Location**). Открывается окно указания точки.

<figure>
  <img src="{{page.image_path}}/07-point-coordinates.png" alt="Рис 19. Координаты точки"/>
  <figcaption>Рис 19. Координаты точки</figcaption>
</figure>

Вводим координаты (0;0), после чего нажимаем на ЛКМ -- фиксируем точку. Затем аналогично указываем еще три -- (0;26), (17;26) и (17;0). Смело жмём на **Esc**. Всё, плата изменила свои размеры.

Дорожки же прокладываются вообще влёт на такой простой плате. Выбираем **Auto Route → All**. В появившемся окне нажимаем Enter и всё, плата разведена.

<figure>
  <img src="{{page.image_path}}/08-result.png" alt="Рис 20. Результат"/>
  <figcaption>Рис 20. Результат</figcaption>
</figure>

Вот и всё. :)
