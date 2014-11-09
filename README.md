Cayman
======

Cayman позволяет быстро и просто создавать интерфейс командной строки 
для программ, написанных на базе Node.js.

## С чего начать?

  Достаточно просто подключить файл cayman.js к основному исполняемому 
  файлу вашего проекта:

```js
#!/usr/bin/env node

var cli = require( './cayman.js' );
```

## Синтаксис

  Вы получаете объект, позволяющий вызывать собственные методы 
  в виде логических цепочек, аналогично цепочкам вызовов в jQuery.
  Рассмотрим пример:

```js
#!/usr/bin/env node

var cli = require( './cayman.js' );

cli
	.meta( <...> )
	.meta( <...> )
	.option( <...> )

	.parse( process.argv );
```

## Описание программы

  Описание содержит основные сведения о программе, такие-как название, 
  версия, сведения об авторе, и всегда отображаться в самом начале вывода 
  программы. Любой элемент описания является полем метаданных и задается с 
  помощью функции meta:

```js
cli.meta( <свойство>, <значение> );
```
  Доступные свойства:

- **name** - название программы,
- **version** - версия,
- **usage** - синтаксис параметров,
- **copyright** - сведения об авторе,
- **license** - используемая лицензия,
- **url** - ссылка на сайт программы.

## Опции

  Опции позволяют при вызове программы передать ей некоторые параметры.
  Вы можете создавать как глобальные, так и локальные опции. Глобальные 
  опции обрабатываются всеми командами, локальные же только той командой, 
  к которой они привязаны. Опция, описанная до первого описания команды, 
  считается глобальной, остальные будут привязаны к своим командам.

  Синтаксис описания опции:

```js
cli
	.option({
		'short_name'  : <буква ключа, например: A, l>,
		'full_name'   : <название параметра для полной формы --name>,
		'access_name' : <название параметра в результирующем объекте>,
		'description' : <описание для справки>
	});
```

## Команды

  Одна программа может выполнять несколько различных действий. 
  Для каждого из них могут быть привязаны свои команды:

```js
cli
	.command( <название>, <описание для справки> )
		// Локальные опции //
		.option({
			'short_name'  : <буква ключа, например: A, l>,
			'full_name'   : <название параметра для полной формы --name>,
			'access_name' : <название параметра в результирующем объекте>,
			'description' : <описание для справки>
		})

		// Действия, выполняемые командой //
		.action(function( argv ){
			// Do something... //
		})
```
  Функция, переданная в качестве аргумента action, получит 
  объект параметров в формате:
```js
{
	'параметр 1': 'значение 1',
	'параметр 2': 'значение 2',
	// ... //
	'параметр N': 'значение N',
}
```
  Контекстом вызова функции будет являться объект cli.

## Метод parse
  
  Данный метод завершает описание интерфейса командной строки 
  и выполняет парсинг переданных программе опций в объект cli.argv. 
  Затем выполняется указанная при вызове программы команда. Синтаксис 
  метода parse:

```js
cli.parse( <массив переданных опций, обычно process.argv> );
```

## Инфраструктура объекта cli
  
  При выполнении определенной команды можно обратиться к объекту 
  cli с помощью ключевого сова this. Что здесь есть:

- **info** - информация о программе, описанная с помощью функции meta;
- **commands** - описанные команды и их локальные опции;
- **options** - массив глобальных опций;
- **argv** - результирующий объект парсинга;
- **header()** - выводит заголовок программы;
- **help()** - выводит справку.

## Справка программы

  По умолчанию в любой создаваемой программе уже определена одна 
  стандартная команда, которая выводит справку. Также справка 
  отображается в случае отсутствия команды при вызове программы.

  Справку можно вывести в любой момент, используя метод cli.help().
  Есть возможность изменить стандартное представление справки и выводить 
  ее в нужном формате без изменения исходных кодов cayman.
  Для этого необходимо до вызова функции parse переопределить метод 
  help:

```js
cli.help = function() {
	// New help function... //
};

// Something else... //

cli.parse( process.argv );
```

## Пример

```js
#!/usr/bin/env node

var cli = require( './cayman.js' );

cli
	// Метаданные //
	.meta( 'name',      'cherry' )
	.meta( 'version',   '0.0.1-beta' )
	.meta( 'copyright', '(C) 2014-2015 WMN' )
	.meta( 'url',       'https://github.com/axilirator/cherry' )
	.meta( 'license',   'This code is distributed under the GNU GPL v3.0' )

	// Глобальные опции //
	.option({
		'short_name'  : 's',
		'full_name'   : 'secret',
		'access_name' : 'server_secret',
		'description' : 'passphrase for authentication'
	})

	// Команды //
	.command( 'begin', 'do something' )
		// Локальные опции //
		.option({
			'short_name'  : 'i',
			'full_name'   : 'server-ip',
			'access_name' : 'server_ip',
			'description' : 'server\'s IP'
		})

		// Действие команды //
		.action(function( argv ){
			// Do something... //
			// this ссылается на cli //

			// argv.server_ip является параметром, который можно 
			// задать как в виде ключа -i <IP>, так и полностью
			// --server-ip <IP>

			// Параметр, не имеющий значения, получит значение true
		})
	.parse( process.argv );
```