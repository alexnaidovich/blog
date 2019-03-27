# Runner - инициализация, работа с зависимостями, хелперы.

В [первой части](https://github.com/alexnaidovich/blog/blob/master/Runner_01.md "Runner - начало") мы положили начало большому проекту, реализующему интерфейс командной строки для быстрой генерации проекта под сайтики для моей работы. Но на момент написания первой части я не удосужился хотя бы раз протестировать, как покажет себя запуск `node index.js` после того, как мы импортировали зависимости и добавили первый компонент. А сегодня первый запуск сразу встретил меня ошибкой:

> `terminal`

```
$ node index.js
internal/modules/cjs/loader.js:583
    throw err;
    ^

Error: Cannot find module 'src\helpers'
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:581:15)
    at Function.Module._load (internal/modules/cjs/loader.js:507:25)
    at Module.require (internal/modules/cjs/loader.js:637:17)
    at require (internal/modules/cjs/helpers.js:22:18)
    at map.component (C:\Alex-work\_inside-tools\runner\index.js:15:20)
    at Array.map (<anonymous>)
    at Object.<anonymous> (C:\Alex-work\_inside-tools\runner\index.js:15:3)
    at Module._compile (internal/modules/cjs/loader.js:689:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:700:10)
    at Module.load (internal/modules/cjs/loader.js:599:32)
```

Природа ошибки дала мне понять, что что-то не так с библиотекой `path` (или моим пониманием данной библиотеки). Поэтому в том месте, где я привлекаю свои компоненты, внутри функции `map` я заменил `require(path.join('src', component))` на простой относительный путь - `require('./src/' + component)`. После этого ошибка исчезла. Итак, точка входа на данный момент выглядит следующим образом:

> `index.js`

```javascript
const inquirer = require('inquirer');
const chalk = require('chalk');
const shell = require('shelljs');
const fs = require('fs');
const path = require('path'); // <- бросает ошибки...

// parts
const [ helpers, intro ] = [ 'helpers', 'intro' ].map(component => require(`./src/${component}`)); // <- здесь произошла замена

const { 
  rewriteSettings
} = helpers;

// settings
const SETTINGS = require('./settings.json');

(async function main() {

  // show intro
  intro();

  // configure dependencies  
  // configure webpack
  // configure ejs components
  // configure styles
  // write files
  // show success

})();

```

При запуске такого `index.js` командная строка, как и ожидалось, выводит приветствие (так как готово, пока что, только оно):

![Приветствие](https://raw.githubusercontent.com/alexnaidovich/blog/blog-images/01.JPG)

Все хорошо, но есть 2 момента: во-первых, хочется вызывать наш интерфейс командой `runner`, а не `node index.js`; во-вторых, сейчас мы можем вызвать этот интерфейс, только находясь в его папке. А хотелось бы иметь возможность вызывать отовсюду. Чтобы разрешить эти задачи, мы:

1. добавляем в файл `index.js` самой первой строчкой (перед импортами) следующий код:
> `index.js`
```javascript
#!/usr/bin/env node
```
Этой строкой мы сообщаем интерпретатору, в каком окружении должен исполняться наш интерфейс.
  
2. добавляем в файл `package.json` раздел `"bin"`:
> `package.json`
```json
{
  "bin" : {
    "runner": "./index.js"
  }
}
```
Этим действием мы привязываем запуск нашего исполняемого скрипта к команде `runner`.

3. Находясь в папке с нашим проектом, выполняем `npm link`. После этого наш интерфейс доступен к вызову по команде `runner` из любого места, любой папки и любого диска - он доступен глобально.

## Первый хелпер

Как было видно в моем коде, я уже запрашивал какую-то сущность из моего файла с помощниками `helpers.js`. Этой сущностью является функция `rewriteSettings`, и предназначена она для того, чтобы сохранять актуальные настройки в файле `settings.json`. На данный момент содержимое файла с помощниками следующее:

> [`src/helpers.js`](https://github.com/alexnaidovich/runner/blob/master/src/helpers.js#L1)

```javascript
// импорты
const fs = require('fs');
const chalk = require('chalk');
const { exec } = require('child_process');

/**
 * Локальный хелпер
 * Если ошибка - логируем, если нет - тихо идем дальше
 * Понадобится в rewriteSettings
 * @param {NodeJSException} error Объект ошибки
 */
function handleErrorWithoutSuccessCallback(error) {
  if (error) {
    console.log(chalk.red(error));
  }
}

/**
 * Тот самый rewriteSettings
 * 
 * @param {Object} json Объект настроек
 * @param {String} keyToReplace Параметр настроек, подлежащих изменению
 * @param {any} valueToReplace Значение изменяемого параметра
 */
module.exports.rewriteSettings = (json, keyToReplace, valueToReplace) => {
  // Хэндлер для замены параметра настроек
  function replaceValue(_key, _currentValue) {
    if (_key === keyToReplace) {
      return valueToReplace;
    }
    return _currentValue;
  }
  
  // Сохранение настроек на диск
  fs.writeFile(`settings.json`, JSON.stringify(json, replaceValue, 2), handleErrorWithoutSuccessCallback);
}

module.exports.npmInit = () => {
  // TODO
}

module.exports.npmInstall = () => {
  // TODO
}
```

> В дальнейшем в заметках я буду давать меньше комментариев. Вместо них я буду давать ссылку на [документированный исходный код](https://github.com/alexnaidovich/runner "Runner on Github").

А с самими настройками дело обстоит интереснее. Пока что `settings.json` абсолютно чист, но он будет заполняться по мере того, как мы будем продвигаться далее по общему плану нашего проекта. Есть даже опасения за то, чтобы этот файл не стал необоснованно большим.

## Работа с зависимостями

Итак, второй пункт нашего глобального плана - работа с зависимостями того сайта, который мы будем создавать с помощью CLI. Данный пункт включает в себя генерацию файла `package.json`, настройку в нем `dependencies` и `devDependencies` и выполнение команды `npm install`.

За всю эту работу будет отвечать отдельный компонент - `config-dependencies.js`, созданный в папке `src` и запрашиваемый к импорту в `index.js`. Из зависимостей этого компонента стоит отметить `inquirer` - с помощью этого модуля будет выстраиваться коммуникация между пользователем и интерфейсом. На компонент возлагается неслабый объем работы, поэтому его также необходимо детально спланировать.

> [`index.js`](https://github.com/alexnaidovich/runner/commit/8d5ddf7ce505dff49920ab37870a9a56b5217e8b#diff-168726dbe96b3ce427e7fedce31bb0bcR13)

```javascript
// parts
const [
  helpers, intro, configDependencies  // <- добавили последний
] = [
  'helpers', 'intro', 'config-dependencies'  // <- добавили последний
].map(component => require(`./src/${component}`));
```

_будет дополнено._
