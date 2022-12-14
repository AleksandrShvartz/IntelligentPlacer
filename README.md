# Intelligent Placer - проект в рамках курса "Решение задач с семантическим разрывом"
**Постановка задачи**

Требуется по поданной на вход фотографии нескольких предметов на светлой горизонтальной поверхности и многоугольнику определить, можно вписать в многоугольник набор предметов из известного заранее множества. Предметы и горизонтальная поверхность, которые могут оказаться на фотографии, заранее известны. Также заранее известно что для каждого предмета зафиксировано изначальное  положение, обозначенное как вертикальное. Многоугольник задаётся фигурой, нарисованной темным маркером на белом листе бумаги, сфотографированной вместе с предметами. Поверхность может представлять собой любую плоскость имеющую однородный характер закраски и цвет близкий к белому. “Intelligent Placer” должен быть оформлен в виде python-библиотеки `intelligent_placer_lib`, которая поставляется каталогом `intelligent_placer_lib` с файлом `intelligent_placer.py`, содержащим функцию - точку входа:

```def check_image(<path_to_png_jpg_image_on_local_computer>[, <poligon_coordinates>])```

которая возвращает 1, если поданый на вход набор может быть вписан в многоуголник и 0, если нет.

```
 from intelligent_placer_lib import intelligent_placer
 def test_intelligent_placer():
	  assert intelligent_placer.check_image(“/path/to/my/image.png”)
```
Также требуется воспроизводимый `intelligent_placer.ipynb`, содержащий репрезентативные примеры работы алгоритма с оценками качества его работы и их визуализацией.

**Описание входных данных**
+  Допустимые форматы фотографий: png, jpg
+  Фотография делается на высоте от 40 до 60 сантиметров над поверхностью.
+  Угол наклона камеры: отклонение от плоскости паралельной листу не более 5 градусов
+  Tолщина линии границы не более 10 px;
+  Фигура задается  тёмным маркером на белом листе формата А4
+  Параметры фигуры: связный многоугольник с количеством вершин не более 10
+  Параметры фона: однородный, близкого к белому цвета
+  Лист должен быть повернут горизонтально, стороны отклоняются не более чем на 5 градусов от границ кадра
+  Прeдметы на фотографии не могут повторятся.
+  Размещение предметов: вне листа с многоугольником, вертикально(т.е. недопустимо подавать на вход предмет из множества, положение которого в пространстве повернуто относительно изначального не вокруг оси перпендикулярной плоскости листа), по правую сторону листа, без перекрытия, с интервалом около 2см между предметами
+  Ориентация входной фотографии важна (необходимо чтоб лист на фотографии располагался меньшими сторонами вниз и вверх)

**План решения задачи**
+  Перевести изображение из цветного в черно-белое.
+  Найти на изображении объекты границ листа, многоугольника и расположенных предметов с помощью фильтров.(Необходимо проверить несколько вариантов и выбрать в качестве решения лучший).
+  Определить для каждого из распознанных предметов чем он являеться (предварительно отфильтровав шум по слишком маленькой площади). Для этого будем использовать факт о том что предеты расположены слева от листа, не пересекаются. Лист  можно определить как объект имеющий наибольшую площадь boundbox-а, многоугольник  -объект лежащий внутри листа, все остальное - предметы.
+  Распознать входные предменты с помощью особенностей формы каждого (карандаш имеет очень большое отношение длины окружающего его boundbox-а к ширине, головоломка имеет маленькое отношение площади выделенного объекта к площади фигуры ограниченной его контуром, фигурка друида имеет описывающий многоугольник с большим количеством вершин, при этом не являясь круглым объектом и.т.д). Возможен вариант использования алгоритма распознования с помощью поиска ключевых точек.
+  Перевести распознаный многоугольник к "абстрактному" виду: найти все его вершиныи и расстояния между ними, затем скорректировать размер, домножив на коэффицент масштаба полученный из размеров распознанного листа.
+  Запустить обученную нейронную сеть на вход получающую на вход "чистое" изображение на котором на белом фоне нарисован абстрактный многоугольник и список из распознанных объектов (каждый объект предстовляет собой эталонную катринку вырезанную и вставленную на белый фон) которая выдает итоговый результат.
+  Сеть будем обучать на специально сгенерированных данных с помощью отрисовки большого количества случайных многоугольников. Построение многоугольников происходит вокруг случайных объектов из списка, расположенных случайным, не пересекающимся образом для разметки равной 1. Для разметки равной 0 многоугольник строиться как минимальный описывающий выборку предметов, касающихся друг друга со сдвинутыми одной или неколькими сторонами/вершинами.
