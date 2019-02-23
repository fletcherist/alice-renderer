# alice-renderer
Библиотека для формирования [ответов](https://tech.yandex.ru/dialogs/alice/doc/protocol-docpage/#response) в навыках Алисы. 
Основана на [Tagged Template Literals](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/template_strings), 
что позволяет в компактной форме записывать текстово-голосовой ответ с кнопками, аудио-эффектами, 
вариативностью и другими кастомизациями.

## Содержание
* [Установка](#)
* [Примеры](#)
  * [базовый](#)
  * [с модификаторами](#)
  * [с параметрами](#)
* [API](#)
  * [reply](#)
  * [reply.end](#)
  * [buttons](#)
  * [audio](#)
  * [effect](#)
  * [pause](#)
  * [br](#)
  * [text](#)
  * [tts](#)
* [Рецепты](#)
  * [вариативность через массивы](#)
  * [модуль рендеринга ответов](#)
  * [обработка условий](#)

## Установка
```bash
npm i alice-renderer
```

## Примеры
### Базовый
В простейшем случае применение функции [`reply`](#reply) к некоторой строке
записывает эту строку в объект с полями `text` и `tts`. При этом из текста вырезаются акценты (`+`):
```js
const {reply} = require('alice-renderer');

const response = reply`Привет! Как дел+а?`;

console.log(response);

/*
{
  text: 'Привет! Как дела?',
  tts: 'Привет! Как дел+а?',
  end_session: false
}
*/
```

### С модификаторами
Функции-модификаторы позволяют добавлять в ответ вариативность, кнопки, аудио эффекты итд:
```js
const {reply, audio, pause, buttons} = require('alice-renderer');

const response = reply`
  ${audio('sounds-game-win-1')} ${['Привет', 'Здор+ово']}! ${pause(500)} Как дел+а?
  ${buttons(['Отлично', 'Супер'])}
`;

console.log(response);

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
При передаче массива в виде `${[item1, item2, ...]}` из него в ответ попадет один случайный элемент. 
В `buttons([...])` переданные значения разворачиваются в кнопки. Подробнее про [buttons()](#buttons).

### С параметрами
Для проброса параметров удобно использовать стрелочную функцию вместе с `reply`:
```js
const {reply, pause, buttons} = require('alice-renderer');

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

### reply\`...\`
Основная функция библиотеки. Формирует ответ для Алисы, раскладывая переданную строку на текст, голос и кнопки. 
По умолчанию строка записывается одновременно в оба поля `text` и `tts`.
Применяя долполнитиельные хелперы можно кастомизировать текстовую и голосовую часть:

```js
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

### reply.end\`...\`
Формирует ответ ровно также, как и [`reply`](), но завершает сессию:

```js
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
reply`${effect('hamster')} Я говорю как хомяк`;

/*
{
  text: 'Я говорю как хомяк',
  tts: '<speaker effect="hamster"> Я говорю как хомяк',
  end_session: false
}
*/
```

### pause([ms])
Добавляет паузу.  
**Параметры:**
  * **ms** `{?Number=500}` - примерное время в милисекундах.

```js
reply`Дайте подумать... ${pause()} Вы правы!`;

/*
{
  text: 'Дайте подумать. Вы правы!',
  tts: 'Дайте подумать. - - - Вы правы!',
  end_session: false
}
*/
```

### br([count])
Добавляет перенос строки в текстовый канал. Вставка `\n` не подходит,
т.к. исходные переносы строк вырезаются для удобства записи ответов.  
**Параметры:**
  * **count** `{?Number=1}` - кол-во переносов строк.

```js
reply`Следующий вопрос: ${br()} "В каком году отменили крепостное право?"`;

/*
{
  text: 'Следующий вопрос:\n"В каком году отменили крепостное право?"',
  tts: 'Следующий вопрос: "В каком году отменили крепостное право?"',
  end_session: false
}
*/
```

### text(str)
Добавляет строку только в текстовый канал. Удобно использовать совместно с `tts()`:
```js
reply`Вы можете написать нам на емейл${text(' user1234@example.com')}. ${tts('Он на вашем экране.')}`;

/*
{
  text: 'Вы можете написать нам на емейл user1234@example.com',
  tts: 'Вы можете написать нам на емейл. Он на вашем экране.',
  end_session: false
}
*/
```

### tts(str)
Добавляет строку только в голосовой канал. Удобно использовать совместно с `text()`:
```js
reply`Вы можете написать нам на емейл${text(' user1234@example.com')}. ${tts('Он на вашем экране.')}`;

/*
{
  text: 'Вы можете написать нам на емейл user1234@example.com',
  tts: 'Вы можете написать нам на емейл. Он на вашем экране.',
  end_session: false
}
*/
```

## Рецепты

### Вариативность через массивы
Вариативность в ответах удобно добавлять через массивы, которые выносить в отдельные переменные:
```js
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
  ...
  return replies.showMenu(session.username);
}
```

### Обработка условий
Можно вставлять обработку условных операторов прям в формируемую строку. Falsy значения в ответ не попадут:
```js
exports.correctAnswer = score => reply`
  Правильный ответ!
  ${score > 100 && 'Вы набрали больше 100 баллов и получаете приз!'}
`;
```

Также можно использовать вложенный `reply`, если требуется обработка строки в условии:
```js
exports.correctAnswer = score => reply`
  Правильный ответ!
  ${score > 100 && reply`${audio('sounds-game-win-1')}Вы набрали больше 100 баллов и получаете приз!`}
`;
```

## Лицензия
MIT @ [Vitaliy Potapov](https://github.com/vitalets)