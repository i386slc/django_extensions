# Использование django-charts-dashboard

## Графики как представления

Если вы хотите отображать только одну диаграмму, вы можете наследовать доступные **ChartViews**. Требуется, чтобы вы определили два основных метода: **generate\_labels** и **generate\_values**.

Доступны BarChartView, PieChartView, DoughnutChartView, RadarChartView, HorizontalBarChartView, PolarAreaChartView, LineChartView, GroupChartView

в диаграмме типа импорта `views.py`, которую вы хотите использовать:

```python
from django.views.generic.base import TemplateView
from charts.views import BarChartView

class ExampleChart(BarChartView, TemplateView):
    ...
    title = "Index of ..."

    def generate_labels(self):
        return ["Africa","Brazil","Japan","EUA"]

    def generate_values(self):
        return [1,10,15,8]
```

в вашем шаблоне, в котором вы хотите отобразить диаграмму, используйте этот тег:

```django
{% raw %}
{% load charts %}
<html>
<head></head>
<body>

{% render_chart chart %}
{% endraw %}

</body>
</html>
```

### Параметры представлений графиков

ниже значения по умолчанию

```python
title = ""
legend = False
beginAtZero = False
aspectRatio = True
width = 100
height = 100
tooltip = None
```

* **title**: определяет заголовок для диаграммы
* **legend**: включает или отключает легенду на диаграмме
* **beginAtZero**: определяет **yAxis** init с нуля
* **aspectRatio**: включает изменение размера диаграммы, если параметр определен как `False`
* **stepSize**: определяет интервал по оси y
* **width**: определяет диаграмму ширины (когда аспектное соотношение равно `False`)
* **height**: определяет диаграмму высоты (когда аспектное соотношение равно `False`)
* **tooltip**: определяет строковую подсказку при наведении курсора мыши на диаграмму
* **colors**: определяет список цветов (строковое шестнадцатеричное представление), чтобы переопределить случайные цвета

Если вы хотите изменить размер диаграммы, просто определите свойства ширины и высоты и установите для параметра **aspectRatio** значение `False`:

```python
from django.views.generic.base import TemplateView
from charts.views import BarChartView

class ExampleChart(BarChartView, TemplateView):
    ...
    title = "Index of ..."
    aspectRatio = False
    width = 300
    height = 250

    def generate_labels(self):
        return ["Africa","Brazil","Japan","EUA"]

    def generate_values(self):
        return [1,10,15,8]
```

### RadarChartView

Чтобы использовать **RadarChartView**, вам нужно создать специальный узел для добавления набора данных. Используя метод **create\_node**, вы можете передать `«label»`, данные (список) и необязательный параметр `«color»`, если вы не передадите цвет, для узла будет сгенерирован случайный цвет. Используйте это в методе **generate\_values**.

Пример ниже:

```python
from django.views.generic.base import TemplateView
from charts.views import RadarChartView

class ExampleChart(RadarChartView, TemplateView):
    ...
    title = "Index of ..."

    def generate_labels(self):
        return ["Africa","Brazil","Japan","EUA"]

    def generate_values(self):
        dataset = []
        # вы можете создать множество узлов для просмотра на диаграмме
        nodeOne = self.create_node("Example 1", [15,5,2,50])
        ....
        dataset.append(nodeOne)

        return dataset
```

### LineChartView

Если вы хотите использовать **LineChartView**, это тот же метод, что и **RadarChartView**, но единственное отличие заключается в параметре **fill**, который по умолчанию равен `False`. Линейная диаграмма также имеет метод **create\_node** для создания специального узла для диаграммы.

Для создания **AreaChart** определите **fill** как `True` в методе **create\_node**. Вы тоже можете передать цвет в качестве параметра этому методу.

Цвет должен быть передан в виде строки `«#606060»`.

Пример: `self.create_node("Test", [1,2,3,4,5], "#606060")`

### GroupChartView

Также есть метод **crete\_node** и тот же метод создания диаграмм выше.

## Диаграммы как объекты

в ваших `views.py`:

```python
from django.views.generic import TemplateView
from charts.objects import BarChart

class ExampleView(TemplateView):

    template_name = "core/example.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)

        chart = BarChart()
        chart.title = "Example charts title"
        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [2,3,10,6]
        chart.data_label = "Test"

        context["chart"] = chart.build_chart()

        return context
```

И в вашем шаблоне `«example.html»` используйте это:

```html
<canvas id="mychart"></canvas>
```

в разделе **script**:

```javascript
$(function(){
    var dataset = JSON.parse('{{ chart|safe }}');
    new Chart(document.getElementById("mychart"), dataset);
})
```

Вы можете использовать объект диаграммы в любой функции в вашем `views.py`, например:

```python
class ExampleView(TemplateView):

    template_name = "core/example.html"

    def my_method(self):
        chart = BarChart()
        chart.title = "Example charts title"
        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [2,3,10,6]
        chart.data_label = "Test"

        return chart.build_chart()

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["chart"] = self.my_method() # любой ключ в context

        return context
```

Диаграммы, доступные в пакете: BarChart, PieChart, HorizontalBarChart, DoughnutChart, PolarAreaChart, RadarChart, LineChart, GroupChart

Можно определить параметры для диаграммы объекта, например:

```python
barchart.title = "..."
barchart.legend = True
```

## Определение фиксированных цветов для диаграммы

Для определения фиксированных вместо случайных цветов используйте это:

```python
class ExampleView(TemplateView):

    template_name = "core/example.html"

    def my_method(self):
        chart = BarChart()
        chart.set_colors(["#fff","#B4edf",...]) # установите свой список цветов здесь
```

### Множество диаграмм в представлениях

Здесь вы можете отобразить более одной диаграммы в вашем HTML-шаблоне, просто вызовите экземпляры диаграмм и определите ключ в контексте.

```python
from django.views.generic import TemplateView
from charts.objects import BarChart, PieChart

class ExampleView(TemplateView):

    template_name = "core/example.html"

    def my_barchart(self):
        chart = BarChart()
        chart.title = "Example charts title"
        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [2,3,10,6]
        chart.data_label = "Test"

        return chart.build_chart()

    def my_piechart(self):
        chart = PieChart()
        chart.title = "Example charts title"
        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [2,3,10,6]
        chart.data_label = "Test"

        return chart.build_chart()


    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["barchart"] = self.my_barchart()
        context["piechart"] = self.my_piechart()

        return context
```

В теле вашего шаблона:

Пример использования **Bootstrap**:

```html
<div class="row">
    <div class="col-6">
        <canvas id="mybarchart"></canvas>
    </div>
    <div class="col-6">
        <canvas id="mypiechart"></canvas>
    </div>
</div>
```

и в разделе script:

```javascript
$(function(){
    var bardata = JSON.parse('{{ barchart|safe }}');
    new Chart(document.getElementById("mybarchart"), bardata);

    var piedata = JSON.parse('{{ piechart|safe }}');
    new Chart(document.getElementById("mypiechart"), piedata);
});
```

### RadarChart

Чтобы использовать радарные диаграммы в качестве объекта в вашем представлении, сделайте следующее:

```python
from django.views.generic import TemplateView
from charts.objects import RadarChart

class ExampleView(TemplateView):

    template_name = "core/example.html"

    def my_method(self):
        chart = RadarChart()
        chart.title = "Example charts title"

        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [
            chart.create_node("Example 1", [5,8,9,64,3]),
            chart.create_node("Example 2", [10,1,19,6,13])
        ]

        return chart.build_chart()

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["chart"] = self.my_method()

        return context
```

### LineChart

```python
from django.views.generic import TemplateView
from charts.charts import LineChart

class ExampleView(TemplateView):

    template_name = "core/example.html"

    def my_method(self):
        chart = LineChart()
        chart.title = "Example charts title"

        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [
            chart.create_node("Example 1", [5,8,9,64,3])
        ]

        return chart.build_chart()

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["chart"] = self.my_method()

        return context
```

### AreaChart

Просто используйте **LineChart** и определите параметр **fill** как `True`, вы можете определить цвет для узла, если хотите.

```python
from django.views.generic import TemplateView
from charts.charts import LineChart

class ExampleView(TemplateView):

    template_name = "core/example.html"

    def my_method(self):
        chart = LineChart()
        chart.title = "Example charts title"

        chart.labels = ["test 1","test 2", "test 3", "test 4"]
        chart.data = [
            # создаем узел для линейного графика
            chart.create_node("Example 1", [5,8,9,64,3], True),
        ]

        # Параметр True в строках выше - это опция заполнения (False или True)

        return chart.build_chart()

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["chart"] = self.my_method()

        return context
```

### GroupChart

Тот же метод, что и на графике выше, с той лишь разницей, что метод **create\_node** имеет параметр цвета.
