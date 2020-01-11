# alice-renderer

[![Actions Status](https://github.com/vitalets/alice-renderer/workflows/autotests/badge.svg)](https://github.com/vitalets/alice-renderer/actions)
[![Coverage Status](https://coveralls.io/repos/github/vitalets/alice-renderer/badge.svg?branch=master)](https://coveralls.io/github/vitalets/alice-renderer?branch=master)
[![Known Vulnerabilities](https://snyk.io/test/github/vitalets/alice-renderer/badge.svg?targetFile=package.json)](https://snyk.io/test/github/vitalets/alice-renderer?targetFile=package.json)
[![npm](https://img.shields.io/npm/v/alice-renderer.svg)](https://www.npmjs.com/package/alice-renderer)
[![license](https://img.shields.io/npm/l/alice-renderer.svg)](https://www.npmjs.com/package/alice-renderer)

Node.js библиотека для формирования [ответов](https://tech.yandex.ru/dialogs/alice/doc/protocol-docpage/#response) в навыках Яндекс Алисы.  

Позволяет:
* компактно записывать ответ
* разделять данные на текст и голос (там где нужно)
* добавлять паузы, звуки и аудио-эффекты
* вставлять изображения
* добавлять вариативность ответов
* вставлять кнопки-подсказки

Основана на использовании [Tagged templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates).

## Содержание

<!-- toc -->

- [Установка](#%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0)
- [Использование](#%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5)
  * [Базовый пример](#%D0%B1%D0%B0%D0%B7%D0%BE%D0%B2%D1%8B%D0%B9-%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80)
  * [Пример с модификаторами](#%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80-%D1%81-%D0%BC%D0%BE%D0%B4%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80%D0%B0%D0%BC%D0%B8)
  * [Пример с параметрами](#%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80-%D1%81-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D0%B0%D0%BC%D0%B8)
- [API](#api)
  * [reply](#reply)
  * [reply.end](#replyend)
  * [buttons(items, [defaults])](#buttonsitems-defaults)
  * [audio(name)](#audioname)
  * [effect(name)](#effectname)
  * [image(imageId, [options])](#imageimageid-options)
  * [pause([ms])](#pausems)
  * [br([count])](#brcount)
  * [text(value)](#textvalue)
  * [tts(value)](#ttsvalue)
  * [textTts(textValue, ttsValue)](#textttstextvalue-ttsvalue)
  * [plural(number, one, two, five)](#pluralnumber-one-two-five)
  * [userify(userId, target)](#userifyuserid-target)
  * [onceInRange(from, to, response)](#onceinrangefrom-to-response)
  * [configure(options)](#configureoptions)
- [Рецепты](#%D1%80%D0%B5%D1%86%D0%B5%D0%BF%D1%82%D1%8B)
  * [Вариативность через массивы](#%D0%B2%D0%B0%D1%80%D0%B8%D0%B0%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D1%81%D1%82%D1%8C-%D1%87%D0%B5%D1%80%D0%B5%D0%B7-%D0%BC%D0%B0%D1%81%D1%81%D0%B8%D0%B2%D1%8B)
  * [Модуль рендеринга ответов](#%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D1%8C-%D1%80%D0%B5%D0%BD%D0%B4%D0%B5%D1%80%D0%B8%D0%BD%D0%B3%D0%B0-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%BE%D0%B2)
  * [Обработка условий](#%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B0-%D1%83%D1%81%D0%BB%D0%BE%D0%B2%D0%B8%D0%B9)
- [Лицензия](#%D0%BB%D0%B8%D1%86%D0%B5%D0%BD%D0%B7%D0%B8%D1%8F)

<!-- tocstop -->

## Установка
```bash
npm i alice-renderer
```

## Использование
### Базовый пример
В простейшем варианте нужно применить тег-функцию [`reply`](#reply) к некоторой строке, заключённой в backticks `` ` ``:
```js
reply`строка`;
```
В результате получим объект с полями `text` и `tts`, в которых записана переданная строка:
```json5
{
  text: 'строка',
  tts: 'строка',
  end_session: false
}
```
При этом для текстового представления из строки вырезаются акценты (`+`):
```js
const { reply } = require('alice-renderer');

const response = reply`Как дел+а?`;

console.log(response);

/*
{
  text: 'Как дела?',
  tts: 'Как дел+а?',
  end_session: false
}
*/
```

### Пример с модификаторами
Функции-модификаторы позволяют обогащать ответ отдельно в текстовом и голосовом каналах.
Например, модификатор [`audio`](#audioname) добавляет звук - он запишется только в поле `tts`:
```js
const { reply, audio } = require('alice-renderer');

reply`${audio('sounds-game-win-1')} Как дел+а?`;

/*
{
  text: 'Как дела?',
  tts: '<speaker audio="alice-sounds-game-win-1.opus"> Как дел+а?',
  end_session: false
}
*/
```

Модификатор [`buttons`](#buttonsitems-defaults) позволяет добавить кнопки:
```js
const { reply, buttons } = require('alice-renderer');

reply`
  Как дел+а?
  ${buttons(['Отлично', 'Супер'])}
`;

/*
{
  text: 'Как дела?',
  tts: 'Как дел+а?',
  buttons: [
    {title: 'Отлично', hide: true},
    {title: 'Супер', hide: true},
  ],
  end_session: false
}
*/
```

Чтобы сделать ответ более разнообразным можно передавать в `reply` массивы значений:`${[item1, item2, ...]}`.
При рендеренге из массива выберется один случайный элемент:
```js
const { reply } = require('alice-renderer');

reply`
  ${['Привет', 'Здор+ово']}! Как дел+а?
`;

/*
{
  text: 'Здорово! Как дела?',
  tts: 'Здор+ово! Как дел+а?',
  end_session: false
}
*/
```

### Пример с параметрами
Для проброса параметров удобно использовать [`reply`](#reply) вместе со стрелочной функцией:
```js
const { reply, pause, buttons } = require('alice-renderer');

const welcome = username => reply`
  ${['Здравствуйте', 'Добрый день']}, ${username}! ${pause(500)} Как дел+а?
  ${buttons(['Отлично', 'Супер'])}
`;

const response = welcome('Виталий Пот+апов');

console.log(response);

/*
{
  text: 'Добрый день, Виталий Потапов! Как дела?',
  tts: 'Добрый день, Виталий Пот+апов! - - - - - - - Как дел+а?',
  buttons: [
    {title: 'Отлично', hide: true},
    {title: 'Супер', hide: true},
  ],
  end_session: false
}
*/
```
Переданные параметры также очищаются от акцентов при записи в поле `text`.

## API

### reply
Основная функция библиотеки. 
Используется как тег-функция для [template literal](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/template_strings): 
```js
 reply`строка`;
```
Формирует ответ для Алисы, раскладывая переданную строку на текст, голос и кнопки. 
По умолчанию строка записывается одновременно в оба поля `text` и `tts`.
Применяя модификаторы можно кастомизировать текстовую и голосовую часть:

```js
const { reply, pause, audio, buttons } = require('alice-renderer');

reply`
  ${audio('sounds-game-win-1')} ${['Привет', 'Здор+ово']}! ${pause(500)} Как дел+а?
  ${buttons(['Отлично', 'Супер'])}
`;

/*
{
  text: 'Здорово! Как дела?',
  tts: '<speaker audio="alice-sounds-game-win-1.opus"> Здор+ово! - - - - - - - Как дел+а?',
  buttons: [
    {title: 'Отлично', hide: true},
    {title: 'Супер', hide: true},
  ],
  end_session: false
}
*/
```

### reply.end
Формирует ответ ровно также, как и [`reply`](#reply), но завершает сессию:

```js
const { reply } = require('alice-renderer');

reply.end`До новых встреч!`;

/*
{
  text: 'До новых встреч!',
  tts: 'До новых встреч!',
  end_session: true
}
*/
```

### buttons(items, [defaults])
Добавляет в ответ кнопки.  
**Параметры:**
  * **items** `{Array<String|Object>}` - тайтлы/описания кнопок.
  * **defaults** `{?Object}` - дефолтные свойства создаваемых кнопок.

В простейшем варианте кнопки можно задавать текстом:
```js
const { reply, buttons } = require('alice-renderer');

reply`Хотите продолжить? ${buttons(['Да', 'Нет'])}`;

/*
{
  text: 'Хотите продолжить?',
  tts: 'Хотите продолжить?',
  buttons: [
    {title: 'Да', hide: true},
    {title: 'Нет', hide: true},
  ],
  end_session: false
}
*/
```

Если нужно изменить тип кнопок, то дополнительно выставляем `defaults`:
```js
const { reply, buttons } = require('alice-renderer');

reply`
  Хотите продолжить? 
  ${buttons(['Да', 'Нет'], {hide: false})}
`;

/*
{
  text: 'Хотите продолжить?',
  tts: 'Хотите продолжить?',
  buttons: [
    {title: 'Да', hide: false},
    {title: 'Нет', hide: false},
  ],
  end_session: false
}
*/
```

Для полной кастомизации можно задавать кнопки объектами:
```js
const { reply, buttons } = require('alice-renderer');

reply`
  Хотите продолжить? 
  ${buttons([
    {title: 'Да', payload: 'yes'},
    {title: 'Нет', payload: 'no'},
  ])}
`;

/*
{
  text: 'Хотите продолжить?',
  tts: 'Хотите продолжить?',
  buttons: [
    {title: 'Да', payload: 'yes', hide: true},
    {title: 'Нет', payload: 'no', hide: true},
  ],
  end_session: false
}
*/
```

### audio(name)
Добавляет звук в голосовой канал.  
**Параметры:**
  * **name** `{String}` - название звука из [библиотеки звуков](https://tech.yandex.ru/dialogs/alice/doc/sounds-docpage/).

```js
const { reply, audio } = require('alice-renderer');

reply`${audio('sounds-game-win-1')} Ура!`;

/*
{
  text: 'Ура!',
  tts: '<speaker audio="alice-sounds-game-win-1.opus"> Ура!',
  end_session: false
}
*/
```
  
### effect(name)
Добавляет голосовой эффект.  
**Параметры:**
  * **name** `{String}` - название эффекта из [библиотеки эффектов](https://tech.yandex.ru/dialogs/alice/doc/speech-effects-docpage/).

```js
const { reply, effect } = require('alice-renderer');

reply`${effect('hamster')} Я говорю как хомяк`;

/*
{
  text: 'Я говорю как хомяк',
  tts: '<speaker effect="hamster"> Я говорю как хомяк',
  end_session: false
}
*/
```

### image(imageId, [options])
Добавляет изображение BigImage в ответ.
 
**Параметры:**
  * **imageId** `{String}` - идентификатор изображения, [загруженного в навык](https://yandex.ru/dev/dialogs/alice/doc/resource-upload-docpage/)
  * **options.title** `{String}` - заголовок изображения
  * **options.description** `{String}` - описание изображения
  * **options.button** `{Object}` - действие по клику на изображение
  
Если не указывать `title / description` изображения, то эти поля автоматически заполняются из поля `response.text`.
Логика заполнения следующая: сначала заполняется `title`, если длина превышает максимальную длину тайтла (128 символов),
то заполняется `description`. Если и в `description` не помещается (256 символов) - 
то пробуем заполнить и `title` и `description`, остальное обрезаем.

Если изначально указано `title / description` - то они сохраняют исходное значение.

Пример с автозаполнением `title`:
```js
const { reply, image } = require('alice-renderer');

reply`
  Вот моя фотка.
  ${image('1234567/xxx')}
`;

/*
{
  text: 'Вот моя фотка.',
  tts: 'Вот моя фотка.',
  end_session: false,
  card: {
    type: 'BigImage',
    image_id: '1234567/xxx',
    title: 'Вот моя фотка.',
  }
}
*/
```

Пример с автозаполнением `description`:
```js
const { reply, image } = require('alice-renderer');

reply`
  Вот моя фотка.
  ${image('1234567/xxx', { title: 'Заголовок' })}
`;

/*
{
  text: 'Вот моя фотка.',
  tts: 'Вот моя фотка.',
  end_session: false,
  card: {
    type: 'BigImage',
    image_id: '1234567/xxx',
    title: 'Заголовок',
    description: 'Вот моя фотка.',
  }
}
*/
```

### pause([ms])
Добавляет паузу.  
**Параметры:**
  * **ms** `{?Number=500}` - время в милисекундах.

```js
const { reply, pause } = require('alice-renderer');

reply`Дайте подумать... ${pause()} Вы правы!`;

/*
{
  text: 'Дайте подумать. Вы правы!',
  tts: 'Дайте подумать. sil <[500]> Вы правы!',
  end_session: false
}
*/
```

### br([count])
Добавляет перенос строки в текстовый канал. Вставка `\n` не подходит,
т.к. исходные переносы строк вырезаются для удобства записи ответов в backticks `` `...` ``.  
**Параметры:**
  * **count** `{?Number=1}` - кол-во переносов строк.

```js
const { reply, br } = require('alice-renderer');

reply`
  Следующий вопрос: ${br()}
  "В каком году отменили крепостное право?"
`;

/*
{
  text: 'Следующий вопрос:\n"В каком году отменили крепостное право?"',
  tts: 'Следующий вопрос: "В каком году отменили крепостное право?"',
  end_session: false
}
*/
```

### text(value)
Добавляет строку только в текстовый канал.  
**Параметры:**
  * **value** `{String|Array<String>}` - строка текста (или массив строк, из которого будет выбран случайный элемент).

Например если фраза заканчивается многоточнием, то многоточние `...` лучше добавить только в текст.
Многоточние в голосе лишь создаст ненужную паузу в конце, из-за которой можно не "услышать" весь ответ пользователя.
```js
const { reply, text } = require('alice-renderer');

reply`Жизнь сложная штука${text('...')}`;

/*
{
  text: 'Жизнь сложная штука...',
  tts: 'Жизнь сложная штука',
  end_session: false
}
*/
```

### tts(value)
Добавляет строку только в голосовой канал.  
**Параметры:**
  * **value** `{String|Array<String>}` - строка для голоса (или массив строк, из которого будет выбран случайный элемент).

Полезно например для выражения эмоций:
```js
const { reply, tts } = require('alice-renderer');

reply`${tts('Йохохо!')} Вы угадали!`;

/*
{
  text: 'Вы угадали!',
  tts: ''Йохохо! Вы угадали!',
  end_session: false
}
*/
```

### textTts(textValue, ttsValue)
Добавляет первый аргумент в текстовую часть, а второй - в голосовую.  
**Параметры:**
  * **textValue** `{String|Array<String>}` - строка для поля `text`.
  * **ttsValue** `{String|Array<String>}` - строка для поля `tts`.

Это полезно при выводе значений предназначенных исключительно для экрана (например email):
```js
const { reply, textTts } = require('alice-renderer');

reply`
  Вы можете написать нам на емейл${textTts(': user1234@example.com', '. Он на вашем экране.')}
`;

/*
{
  text: 'Вы можете написать нам на емейл: user1234@example.com',
  tts: 'Вы можете написать нам на емейл. Он на вашем экране.',
  end_session: false
}
*/
```

Или при вставке emoji:
* в текст лучше их добавить без знаков препинания (так смотрится лучше)
* в голос наоборот вставить знак препинания без emoji (тогда Алиса прочитает с правильной интонацией)

```js
const { reply, textTts } = require('alice-renderer');

reply`Отлично${textTts(' 👌', '!')} Будет скучно - обращайтесь.`;

/*
{
  text: 'Отлично 👌 Будет скучно - обращайтесь.',
  tts: 'Отлично! Будет скучно - обращайтесь.',
  end_session: false
}
*/
```

### plural(number, one, two, five)
Подстановка форм слова для числительных. В строках можно использовать плейсхолдеры `$1`, `$2`, `$5`.  
**Параметры:**
  * **number** `{Number}` - число.
  * **one** `{String}` - строка для `1`.
  * **two** `{String}` - строка для `2`.
  * **five** `{String}` - строка для `5`.

```js
const { reply, plural } = require('alice-renderer');

const getResponse = count => reply`
  У вас ${plural(count, '$1 правильный ответ', '$2 правильных ответа', '$5 правильных ответов')}
`;

getResponse(1); // response.text = "У вас 1 правильный ответ"
getResponse(2); // response.text = "У вас 2 правильных ответа"
getResponse(5); // response.text = "У вас 5 правильных ответов"
getResponse(121); // response.text = "У вас 121 правильный ответ"
```

### userify(userId, target)
Персонализирует функции рендеринга под конкретного пользователя. 
Это позволяет избежать повторений при выборе ответов из массива.     
**Параметры:**
  * **userId** `{String}` - идентификатор пользователя.
  * **target** `{Function|Object}` - функция рендеринга или объект, ключи которого - функции рендеринга.

**Возвращает:**
  * `{Function|Object}` 

Например, без использования `userify`:
```js
const { reply } = require('alice-renderer');

const replySuccess = () => reply`
  ${['Отлично', 'Супер', 'Класс']}! 
  Это правильный ответ!
`;

replySuccess();
replySuccess();
replySuccess();

// может оказаться так:
// => "Супер! Это правильный ответ!"
// => "Супер! Это правильный ответ!"
// => "Супер! Это правильный ответ!"
```

С использованием `userify` ответы будут гарантированно разные:
```js
const { reply, userify } = require('alice-renderer');

const replySuccess = () => reply`
  ${['Отлично', 'Супер', 'Класс']}!
  Это правильный ответ!
`;

const userReplySuccess = userify(userId, replySuccess);

userReplySuccess();
userReplySuccess();
userReplySuccess();

// => "Супер! Это правильный ответ!"
// => "Отлично! Это правильный ответ!"
// => "Класс! Это правильный ответ!"
```

Также в `userify` можно передать объект - тогда будут обернуты все его методы:
```js
const { reply, userify } = require('alice-renderer');

const replies = {
  success: () => reply`${['Отлично', 'Супер', 'Класс']}! Это правильный ответ!`,
  fail: () => reply`${['Нет', 'Неверно']}! Это неправильный ответ!`,
};
const userReplies = userify(userId, replies);

userReplies.success();
```  

### onceInRange(from, to, response)
Возвращает заданный ответ один раз за `N` вызовов, где `N` лежит в диапазоне `(from, to)`.
Например, чтобы раз в 3 - 5 вызовов добавлять в ответ `"Вы классно отвечаете на вопросы!"`, можно написать так:
```js
reply`
  ${onceInRange(3, 5, 'Вы классно отвечаете на вопросы!')}
`;
```
Используется совместно с `userify` для повышения вариативности.
  
**Параметры:**
  * **from** `{Number}` - нижняя граница диапазона
  * **to** `{Number}` - верхняя граница диапазона
  * **response** `{String|Object}` - ответ

**Возвращает:**
  * `{String|Object}`

```js
const { reply, userify, rand } = require('alice-renderer');

const replySuccess = () => reply`
  Правильно!
  ${onceInRange(3, 5, 'Вы классно отвечаете на вопросы!')}
`;
const userReplySuccess = userify(userId, replySuccess);

// => "Правильно!"
// => "Правильно!"
// => "Правильно!"
// => "Правильно! Вы классно отвечаете на вопросы!"
// => "Правильно!"
// => ...
```

### configure(options)
Глобальная конфигурация модуля.  
**Параметры:**
  * **options.disableRandom** `{Boolean} = false` - отключает рандом. 
    Все ответы, содержащие массивы, всегда возвращают первый элемент.
    Это удобно для тестов.

**Возвращает:**
  * `{undefined}`

Например:
```js
const { reply, configure } = require('alice-renderer');

configure({disableRandom: true});

reply`${['Раз', 'Два', 'Три']}`;
reply`${['Раз', 'Два', 'Три']}`;
reply`${['Раз', 'Два', 'Три']}`;
reply`${['Раз', 'Два', 'Три']}`;

// всегда возвращает {text: "Раз", tts: "Раз"}
```

## Рецепты

### Вариативность через массивы
Вариативность в ответах удобно добавлять через массивы, которые выносить в отдельные переменные:
```js
const { reply } = require('alice-renderer');

const greetingText = [
  'Привет',
  'Здор+ово',
  'Здравствуйте',
  'Добр+о пожаловать',
  'Йох+анга',
];

const greetingSound = [
  audio('sounds-game-win-1'),
  audio('sounds-game-win-2'),
  audio('sounds-game-win-3'),
];

const welcome = () => reply`
  ${greetingSound} ${greetingText}! Я голосовой помощник Алиса.
`;
```

### Модуль рендеринга ответов
Всю работу по формированию финального текстового ответа удобно вынести в отдельный модуль. 
Это позволяет отделить логику навыка от рендеринга и менять эти части независимо:

**replies.js**
```js
const { reply, buttons } = require('alice-renderer');

exports.welcome = () => reply`
  Привет! Я голосовой помощник Алиса.
`;

exports.showMenu = username => reply`
  ${username}, вы можете сразу начать игру или узнать подробнее о правилах.
  ${buttons(['Начать игру', 'Правила'])}
`;

exports.goodbye = () => reply.end`
  Отлично! Будет скучно - обращайтесь.
`;
```

**logic.js**
```js
const replies = require('./replies');

function handleMenuRequest() {
  // логика формирования ответа...
  return replies.showMenu(session.username);
}
```

### Обработка условий
Можно вставлять обработку условных операторов прям в формируемую строку. Falsy значения в ответ не попадут:
```js
const { reply } = require('alice-renderer');

exports.correctAnswer = score => reply`
  Правильный ответ!
  ${score > 100 && 'Вы набрали больше 100 баллов и получаете приз!'}
`;
```

Также можно использовать вложенный `reply`, если требуется обработка строки в условии:
```js
const { reply, audio } = require('alice-renderer');

exports.correctAnswer = score => reply`
  Правильный ответ!
  ${score > 100 && reply`${audio('sounds-game-win-1')}Вы набрали больше 100 баллов и получаете приз!`}
`;
```

## Лицензия
MIT @ [Vitaliy Potapov](https://github.com/vitalets)
