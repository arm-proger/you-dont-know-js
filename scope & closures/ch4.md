# Вы не знаете JS: Область видимости и замыкания
# Глава 4: Поднятие переменных (Hoisting)

Теперь вы должно быть вполне уверенно разбираетесь в идее области видимости, и в том, как переменные присоединяются к различным уровням области видимости в зависимости от того, где и как они объявлены. Обе области видимости, как функции, так и блока работают по одинаковым правилам в таком виде: любая переменная, объявленная в области видимости, присоединяется к этой области видимости.

Но есть маленький нюанс в том, как работает присоединение к области видимости с объявлениями, которые появляются в различных местах области видимости, и это именно тот нюанс, который мы тут и исследуем.

## Курица или яйцо?

Есть искушение подумать, что весь код, который вы видите в программе на JavaScript, интерпретируется строка за строкой. Несмотря на то, что по сути это правда, есть одна часть этого предположения, которая может привести к  некорректному пониманию вашей программы.

Представьте такой код:

```js
a = 2;

var a;

console.log( a );
```

Что вы ожидаете увидеть в выводе оператора `console.log(..)`?

Многие разработчики ожидают увидеть `undefined`, поскольку оператор `var a` идет после `a = 2`, и было бы естественным предположить, что переменная переопределена и потому ей присвоено значение по умолчанию `undefined`. Однако, результат будет `2`.

Представьте еще один код:

```js
console.log( a );

var a = 2;
```

Вы можете склониться к предположению, что поскольку в предыдущем показанном коде есть некоторое поведение в стиле "немного с ног на голову", то возможно в этом коде также будет выведено `2`. Другие могут подумать, что поскольку переменная `a` используется раньше, чем объявлена, то это приведет к выбросу `ReferenceError`.

К сожалению, оба предположения неверны. Будет выведено `undefined`.

**Так что же здесь происходит?** Похоже тут вопрос сродни "что раньше: курица или яйцо?". Что идет первым: объявление ("яйцо") или присваивание ("курица")?

## Компилятор снова наносит удар

Чтобы ответить на этот вопрос, нам нужно вернуться в главу 1 и нашу дискуссию о компиляторах. Вспомните, что *Движок* на самом деле скомпилирует ваш код JavaScript до того, как начнет интерпретировать его. Частью фазы компиляции является нахождение и ассоциация всех объявлений с их соответствующими областями видимости. Глава 2 показала нам, что это и есть сердце лексической области видимости.

Поэтому, лучший путь думать об этих вещах — что все объявления как переменных, так и функций, обрабатываются в первую очередь, до того как будет выполнена любая часть вашего кода.

Когда видите `var a = 2;`, вы наверное думаете о нем как об одном операторе. Но JavaScript на самом деле думает о нем как о двух операторах: `var a;` и `a = 2;`. Первый оператор, объявление, обрабатывается во время фазы компиляции. Второй оператор, присваивание, остается **на своем месте** в фазе исполнения.

Следовательно о нашем первом коде следует думать как об обрабатываемом следующим образом:

```js
var a;
```
```js
a = 2;

console.log( a );
```

...где первая часть — компиляция, а вторая — выполнение.

Аналогично, наш второй код в действительности будет обработан так:

```js
var a;
```
```js
console.log( a );

a = 2;
```

Получается, один из путей представить это, в какой-то степени образно, что эти объявления переменной и функции "переехали" с того места, где они появились в коде в начало кода. Это дало начало названию "Поднятие (Hoisting)".

Другими словами, **яйцо (объявление) появилось до курицы (присваивания)**.

**Примечание:** Поднимаются только сами объявления, тогда как любые присваивания или другая логика выполнения остается *на месте*. Если поднятие намеренно используется, чтобы перестроить логику выполнения вашего кода, то это может вызвать хаос.

```js
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

Объявление функции `foo` (которое в этом случае *включает в себя* соответствующее значение как актуальную функцию) поднимается, благодаря чему ее вызов в первой строке может быть выполнен.

Также важно отметить, что каждое поднятие соотносится **с областью видимости**. Поэтому несмотря на то, что наши предыдущие примеры кода были упрощенными в том, что были включены только в глобальную область видимости, функция `foo(..)`, которую мы сейчас изучаем, показывает, что `var a` поднимается наверх `foo(..)` (а не, очевидно, наверх всей программы). Так что программу можно было бы прочесть более точно как:

```js
function foo() {
	var a;

	console.log( a ); // undefined

	a = 2;
}

foo();
```

Объявления функций поднимаются, как мы только что видели. А функциональные выражения — нет.

```js
foo(); // не ReferenceError, но TypeError!

var foo = function bar() {
	// ...
};
```

Идентификатор переменной `foo` поднимается и присоединяется к включающей его области видимости (глобальной) этой программы, поэтому `foo()` не вызовет ошибки `ReferenceError`. Но в `foo` пока еще нет значения (как если бы это было объявление обычной функции вместо выражения). Поэтому, `foo()` пытается вызвать значение `undefined`, которое является неправильной операцией с вызовом ошибки `TypeError`.

Также помните, что даже если это именованное функциональное выражение, идентификатор имени недоступен в окружающей области видимости:

```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
	// ...
};
```

Этот код более точно интерпретируется (с учетом поднятия) как:

```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
	var bar = ...self...
	// ...
}
```

## Сначала функции

Как объявления функций, так и переменных поднимаются. Но тонкость (которая *поможет* объяснить множественные объявления "дубликатов" в коде) в том, что сперва поднимаются функции, а затем уже переменные.

Представим:

```js
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

`1` выводится вместо `2`! Этот код интерпретируется *Движком* так:

```js
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```

Обратите внимание, что `var foo` является дублем объявления (и поэтому игнорируется), даже несмотря на то, что идет до объявления `function foo()...`, потому что объявления функций поднимаются до обычных переменных.

В то время как множественные/дублирующие объявления `var` фактически игнорируются, последовательные объявления функции *перекрывают* предыдущие.

```js
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

Несмотря на то, что всё это может прозвучать как не более чем любопытный академический факт, это привлекает тем, что дубли определений в одной и той же области видимости — в самом деле плохая идея и часто приводит к странным результатам.

Объявления функций, которые появляются внутри обычных блоков, обычно поднимаются в окружающей области видимости, вместо того чтобы быть условными как показывает код ниже:

```js
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```

Однако, важно отметить, что такое поведение небезопасно и может измениться в будущих версиях JavaScript, поэтому лучше избегать объявления функций в блоках.

## Обзор

У нас есть соблазн смотреть на `var a = 2;` как на один оператор, но  *Движок* JavaScript видит это по-другому. Он видит `var a` и `a = 2` как два отдельных оператора, первый — как задачу фазы компиляции, а второй — как задачу фазы выполнения.

Это приводит к тому, что все объявления в области видимости, независимо от того где они появляются, обрабатываются *первыми*, до того, как сам код будет выполнен. Можно мысленно представить себе это как объявления (переменных и функций), "переезжающие" в начало их соответствующих областей видимости, что мы называем "поднятие (hoisting)".

Сами объявления поднимаются, а присваивания, даже присваивания функциональных выражений, *не* поднимаются.

Остерегайтесь дублей объявлений, особенно смешанных обычных объявлений var и объявлений функций — вас будут поджидать неприятности!