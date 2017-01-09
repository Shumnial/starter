# Starter #

Boilerplate для верстки проектов.

## Структура ##

* Таск раннер - [Gulp](http://gulpjs.com/)
* Сервер - [Express](http://expressjs.com/ru/guide/routing.html)
* Шаблонизатор [Nunjucks](https://mozilla.github.io/nunjucks/)
* Стили - [Less](http://lesscss.org/), [Autoprefixer](https://github.com/postcss/autoprefixer)
* Яваскрипт - [Babel.js](https://babeljs.io/)

## Config ##

Файл конфига лежит в корне проекта.
- devStatic  - string, Путь до папки с ассетами. по умолчнию равен '/'
- buildStatic  - string, Путь до папки с ассетами для готовой сборки. по умолчнию равен '' 
- buildDir  - string, Имя папки в которую будет все собираться. Если поменяли это значение, не забудьте добавить вашу новую директорию в гитигнор.
- port - number, порт на котором будет запущен сервер
- dpe - boolean, Если сайт создается под эту платформу, то обязательно аереключаем на true. тогда при экспорте все ссылки на фалы будут заменены на необходимую для этой платформы конструкцию типа: ``` @File('js/your_js_file.js') ```
- buildIgnore - array, список файлов/папок которые не должны попасть в билд сборку. По-умолчнию это папка с less -файлами и исходниками js из папки javascripts/source 
- commonData - array, список имен файлов, которые будут передавться во все компилируемы шаблоны. Файлы должны лежать в корне папки 'datasource'. В шаблон, данные из этих файлов, передаются в переменной common. 
	```
		//Это кусок конфига
		{
		...,
		commonData: ['somefile', somfile2] // .JSON only!!1111
		...,
		}


		//Это кусок somefile.json

		{
			"myVar": "Вжуххх!!!"
		}


		// Теперь из любого шаблона мы имеенм доступ к даннным поключенных файлов
		// Например так: 

		<div>{{ common.somefile.myVar }}</div> => <div>Вжуххх!!!</div>

	```

- pages - Список страниц проекта. В него передается объект следующего вида: 

	```
	{
		"name": "index",
		"route": "/",
		"pageData": [],
		"pageVars": {}
	}

	```
- name - string, Имя фала с шаблоном
- route - string, роут по которму будет отдаваться текущий шаблон
- pageData - array,  массив состоящий из имен фалов (.json) для вставки в текущий шаблон. Например, если мы указываем, что у данной страницы есть PageData = ['news', 'activity'] то это означает, что в шаблон будут переданы два json файла из папки ./datasource с указанными именами. Данные будут находится в ключах соответсвующих названию файла. Т.е. для отображения списка новостей можно использовать следующую конструкцию

```
<div class="news">
	{% for item in news %}
		<div class="news__item">
			<div class="news__imgbox">
				<img src="{{ item.img|img_asset }}" alt="">
			</div>
			<div class="news__title">{{ item.title }}</div>
			<div class="news__text">{{ item.text }}</div>
			<a href="{{ item.link }}">подробнее</a>
		</div>
	{% endfor %}
</div>

```  
- pageVars - object, объект хранящий в себе переменные (если такие необходимы) для теущего темплейта. Несмотря на то, что локальные переменные можно объявлять непосредственно в самом шаблоне, бывают случаи, что такие переменные должны быть у всех (или у многих) шаблонов, и чтобы, опять же, избавить себя от копипаста строки с объявлением переменной в каждом теплейте, и создан этот объект. в качетсвте значений можно передавать любой тип данных(массив, объект, число, булево и тп). Но обычно это строка.  Как это выглядит на практике

```
// есть конфиг с таким объектом
pages: [{
	"name": "index",
	"route": "/",
	"pageData": [],
	"pageVars": {
		"userName": "Василий Жопов",
		"greetings": "не рады"
	}
} ...]


// Теперь в шаблоне index.html можно добраться до этой переменной через объект locals

<p>
	Привет {{ locals.userName }}! <br> Мы {{ locals.greetings }} тебя видеть.


</p> 

// =>  <p>Привет Василий Жопов! <br> Мы не рады тебя видеть.
```

### Немного про фильтры ###

В шаблонизаторе есть набор [готовых фильтров](https://mozilla.github.io/nunjucks/templating.html#builtin-filters), под различные нужды. 
Т.к. данный boilerplate создавался, фактически, для одной-единственной цели, а именно — не трахаться с заменой статики при экспорте верстки под различные платформы, (+ избавиться от кучи ручного копипаста), то естественно в мозги шаблонизатора был вкорячен некоторый набор фильтров. Обязательны к использованию только asset фильтры. 

#### Лукморе asset, img_asset, css_asset ... etc ####

Применение img_asset в src атрибуте картинки: 

```
// такой src 
<img src="{{ 'logo.png'|img_asset }}" alt="">

// будет преобразован в =>
<img src="images/logo.png">

// или в такой (если config.dpe = true ) 
<img src="@File('images/logo.png')">
```


Еще кучка примеров использования:

```
<link href="{{ 'style.css'|css_asset }}" rel="stylesheet" type="text/css">

<script src="{{ 'common.js'|js_asset }}"></script>

<div style="background-image: url({{ 'pic.jpg'|img_asset }})"></div>

```

> Есть маленький нотис. Если мы используем фильтр на строковом значении (например logo.png), То, естественно мы должны заключить это значение в кавыки, чтобы пояснить за тип). Имя переменно в кавыки заключать не надо. 

```
{% set img = 'logo.png' %}

<img src="{{ img|img_asset }}" alt="">
```


#### Пояснялка за loop фильтр ;) ####

Иногда (на самом деле почти всегда), верстку делать надо, а контента еще нет. чтобы не копипастить всякие списочные итемы руками я написал небольшой фильтр. В него передается массив элементов, которые мы хотим итерировать, а так же единственный аргумент — количество итераций
Если в массиве всего 2 итема, а в дизайне список из 12 итемов, то нет необходимости засерать шаблон ненужным копипастом. 
Лучше используй фильтр loop. Ниже написано как это делать =)

```
{% set news = [
	{
		title: "Какой-то заголовок 1",
		text: "Немного букв 1"
	},
	{
		title: "Какой-то заголовок 2",
		text: "Немного букв 2"
	}
] %}


// Данный цикл будет итерировать уже не массив news а новый массив, 
// который будет состоять из элементов массива news c длинной массива = 12

{% for item in news|loop(12) %}
	<div>
		<div>{{ item.title }}</div>
		<div>{{ item.text }}</div>
	</div>
{% endfor %}

```