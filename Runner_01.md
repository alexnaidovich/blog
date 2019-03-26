# Runner - начало.

В процессе повседневной работы на нынешнем рабочем месте я сталкиваюсь с задачей создавать большое количество небольших однотипных веб-сайтов. По мере работы я с каждым следующим сайтом добавлял какой-то кусочек автоматизации процесса создания. 

> Менее года назад я стартовал карьеру веб-разработчика, и мое нынешнее место работы - первое в этом направлении. Прибыл я туда с довольно скудным базовым стеком скиллов, и самым востребованным из них оказался навык _копировать-вставлять_. И чтобы уйти от этого, в ход пошло освоение автоматизации. Мало-помалу в работу подтянулись Node.js, Webpack, ejs-шаблонизация, scss, headless Chrome и прочие плюшки.

Но вот уже третью неделю я ловлю себя на том, что я явно что-то делаю не так. У меня есть построенные мной утилиты для разных целей - от цикличного сбора необходимых данных и генерации любого количества контентных страниц за один запуск до тонких твиков `webpack.config.js`. И на настройку каждой из этих утилит уходило по ~10 минут довольно нудной работы (подстановка переменных, (рас-)комментирование (не-)нужных действий и т.д.). То есть, какая-то автоматизация есть, но она далека от совершенства. В силу большой привязки к SEO и необходимости отдавать специфичную статику, модные javascript-фреймворки типа React'а и его братии мне однозначно не подходят. Я даже невзначай поигрался с несколькими website builder'ами, авось наткнусь там на вариант, с которым выиграю и по времени, и по существу моей работы, пожертвовав отдалением от самого процесса кодинга. Не наткнулся, и очень рад этому. И сегодня меня внезапно осенило - 

### а что, если замутить Command Line Interface?

CLI может на выходе дать мне ряд существенных бенефитов. По моей задумке, CLI будет брать на себя рутину с кодогенерацией, созданием структуры проекта, подстановкой переменных в конфиг и утилиты, а также в процессе у меня есть неплохой шанс систематизировать все эти утилиты и добавить им реюзабельности. Поэтому интерфейс надо очень существенно продумать, ведь от него будет зависеть мой воркфлоу :).

Поскольку судьба меня свела именно с javascript'ом, соорудить свое детище я решил в окружении Node.js. За основу я взял [эту статью](https://codeburst.io/building-a-node-js-interactive-cli-3cb80ed76c86 "Building a Node JS interactive CLI - Codeburst.io"). В ней полностью рассмотрен путь создания примитивного CLI в окружении ноды.

Проект я начал со следующей тройки зависимостей:

  * `chalk` - стилизация в командной строке, для себя же делаю;
  * `inquirer` - коллекция базовых интерфейсов командной строки;
  * `shelljs` - Shell-команды для js.
  
  ```bash
  npm init
  npm install --save chalk inquirer shelljs
  ```
  
Поначалу я импортировал их все в основную точку входа - `index.js`. Но скорее всего, конкретно здесь мне ни одна из них не понадобится - проект уже является модульно-экспортным, и пока зависимости применяются только в импортируемых компонентах. Посмотрим, как будет по мере работы, но есть у меня предчувствие, что из `index.js` они уйдут.

Что мне понравилось в опорной статье - автор спланировал флоу своего приложения от приветствия до сообщения об успешном завершении. Это хорошая практика, которая мной применялась, к сожалению, очень редко. В данном случае хорошее планирование стратегически необходимо. Мое детище планирует выполнять на порядок больше действий, чем вышеупомянутый аналог автора, поэтому план будет пока что очень обобщенным (закомментирован в `main()`):

> `index.js`

```javascript
const inquirer = require('inquirer');
const chalk = require('chalk');
const shelljs = require('shelljs');
// Также я привлёк 2 стандартные библиотеки Node.js:
const fs = require('fs');      // <- либа для работы с файловой системой
const path = require('path');  // <- либа для работы с путями

(async function main() {

  // show intro
  // configure dependencies
  // configure webpack
  // configure ejs components
  // configure styles
  // write files
  // show success

})();
```

Кроме импорта зависимостей, есть смысл импортировать и собственные компоненты в процессе их создания. Также я подумал, что, скорее всего, мне пригодится файл настроек. А также обязательно у меня будет модуль с самописными функциями-помощниками. Всегда так бывает, когда изобретаешь велосипед :). Ну и сразу создаётся самый лёгкий компонент из нашего плана - функция показа приветствия.

На данном этапе проект имеет следующую структуру (исключая `node_modules` и прочую служебщину):

```
root
|--index.js         <- основная точка входа
|--package.json     <- зависимости проекта, инфа
|--settings.json    <- мой файл с настройками
|--src
|  |--helpers.js    <- помощники
|  |--intro.js      <- приветствие (компонент)
```

Импортируем модули в точку входа, сразу после импорта зависимостей. Более того, попробуем сделать это по-крутому, с помощью деструктуризации из es6:

> `index.js`

```javascript
// components
const [
  {
    rewriteSettings,  // у меня уже есть одна функция в хелперах, о ней позже
  },
  intro
] = [ 'helpers', 'intro' ].map(component => require(path.join('src', component)));

// settings
const SETTINGS = require('./settings.json');
```

Собственно, приветствие - самый простой компонент приложения с простой целью - оповестить меня в консоли, что приложение пытается запуститься. Зацикливаться на нем долго не буду, разве только отмечу, что изменил цвет строки на выходе с помощью `chalk`, и параметры `версия` и `автор` привязаны к `package.json`.

> `src/intro.js`

```javascript
const chalk = require('chalk');
const { version, author } = require('../package.json');

module.exports = () => {
  console.log(
    chalk.green(`Runner v.${version} by ${author}.\n`)
  );
  console.log(
    chalk.yellow(`Let's configure your boilerplate.`)
  );
}
```

И раз мы уже импортировали этот компонент в точку входа, именно там его и есть смысл вызвать.

> `index.js`

```javascript
/* imports */

(async function main() {

  // show intro
  intro();
  // ВЫПОЛНЕНО!

  // configure dependencies
  // ... остальные пункты плана

})();
```

В следующей заметке мы рассмотрим обработку входных зависимостей и работу с хелперами. А также проект будет залит на github.

Спасибо за внимание!