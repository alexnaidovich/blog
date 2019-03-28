# Runner - здесь точно будет работа с зависимостями...

Метод проб и ошибок очень хорош. Но он не очень подходит в том случае, когда ты хочешь написать в блоге о том, как у тебя всё хорошо и ровно получается. Но это не про наш случай. Поэтому сейчас будет краткий разбор ошибок [предыдущих частей](https://github.com/alexnaidovich/blog/blob/master/Runner_02.md).

### 1. Naming conventions best practices & структура проекта

Касательно **naming conventions** - это чисто косметическая правка, которая свелась к тому, что я переименовал папку `src` в `lib`. Мог бы об этом и не упоминать, но тогда была бы путаница. 

А поистину серьезные изменения коснулись файла настроек. Глобальный `settings.json`, который лежит в корне, я оставил себе как черновик про запас, а вот конфиги, которые будут реально использоваться в проекте, будут жить в папке `config`. И структура проекта на данный момент такова:

```
root
|--index.js
|--package.json
|--settings.json
|--lib
|  |--intro.js
|  |--config-dependencies.js
|  |--helpers.js
|--config
|  |--deps.config.json
```

### 2. path

Всё с библиотекой хорошо. Точнее, будет хорошо, если на инициализации сохранить основные пути в переменные, делая их абсолютными при помощи метода `path.resolve()`. 
> [`index.js`](https://github.com/alexnaidovich/runner/blob/master/index.js#L12)

```javascript
// paths
const dirname = __dirname;
const dir_lib = path.resolve(dirname, 'lib');
const dir_config = path.resolve(dirname, 'config');

// components
const [
  helpers, intro, configDependencies
] = [
  'helpers', 'intro', 'config-dependencies'
].map(component => require(path.join(dir_lib, component)));
```

> Прелесть глобальной общедоступной переменной `__dirname` заключается в том, что она всегда указывает на директорию **файла, в котором она вызвана**. А понадобится это нам для того, чтобы, к примеру, сохранять настройки в исходной папке CLI, а не в папке сайтика, который мы с помощью CLI генерируем.

Теперь при импорте наших компонент метод `require(path.join(dir_lib, component))` отрабатывает на отлично и не выбрасывает ошибок.

### 3. `lib/helpers.js`, а точнее - `rewriteSettings()`

Разобрать ошибку с путями было необходимо для решения проблемы этого метода - места сохранения файла настроек. В предыдущем исполнении данного метода файл `settings.json` сохранялся не в папке, где находится CLI, а в текущей рабочей директории (папке, откуда мы запустили runner), так как путь сохранения был указан относительный. Чтобы это решить, нам в данном компоненте понадобится библиотека `path`, а также сохраним путь к папке настроек в переменную.
> [`lib/helpers.js`](https://github.com/alexnaidovich/runner/blob/master/lib/helpers.js#L4)
```javascript
const path = require('path');
const dir_config = path.resolve(__dirname, '..', 'config');
```

Теперь к самому методу. Поскольку мы разделили наши настройки на отдельные файлы для каждого компонента, сохранять файлы придется теперь тоже раздельно. Согласно моему восприятию такого понятия, как naming convention, мои файлы настроек имеют вид **`${назначение_конфига}.config.json`**. И соответственно, функиця должна принимать это `назначение конфига` входным параметром. Назовем его `typeOfConfig`, и с ним функция будет иметь следующий вид:
> [`lib/helpers.js`](https://github.com/alexnaidovich/runner/blob/master/lib/helpers.js#L24)
```javascript
module.exports.rewriteSettings = (json, typeOfConfig, keyToReplace, valueToReplace) => {  
  function replaceValue(_key, _currentValue) {
    if (_key === keyToReplace) {
      return valueToReplace;
    }
    return _currentValue;
  }
  fs.writeFile(`${dir_config}/${typeOfConfig}.config.json`, JSON.stringify(json, replaceValue, 2), handleErrorWithoutSuccessCallback);
}
```

А обратить внимание здесь стоит на то, что первым параметром в `fs.writeFile()` мы передаем путь, который включает в себя переменную `dir_config`, определенную ранее. Теперь файл сохранится там, где нужно.

## А теперь к `lib/config-dependencies.js`

Первое. **Импортируем свежий, специально для этого компонента созданный файл настроек**. Про `settings.json` забываем.
Второе. Импортируем то, чем будем пользоваться (зависимость `inquirer` и наш хелпер `rewriteSettings`).

> `lib/config-dependencies.js`
```javascript
const inquirer = require('inquirer'); 
const CONFIG = require('../config/deps.config.json');
const { rewriteSettings } = require('./helpers');
const package_json = {}; // <- про это сейчас расскажу.
```

Основная цель данного компонента - сформировать `package.json` и на его основе выполнить `npm install`. Поэтому (отсылка к последней строчке предыдущего кода) заведем переменную `package_json`, хранящую (пока что) пустой объект.

Вспомним наш план:
```javascript
async function main() {
  // Plan: 
  // 1. Define author by default
  //   1.1. Select from list of default authors or 'input manually'
  //   1.2. If 'input manually' - offer to store in defaults.
  // 2. Give name and description to the project
  // 3. Define devDeps
  //   3.1. Define if they are cahced or not 
  //     3.1.1. If devDeps are cahced, choose directory to install from (with 'input manually' value and offer to store)
  //     3.1.2. If not, go to 3.2
  //   3.2. Select devDeps kit to install (with 'input manually' value)
  //     3.2.1. If inputted manually, offer to save input as a devDeps kit
  // 4. Define deps
  //   4.1. Select deps kit to install (with 'input manually' value)
  //     4.1.1. If inputted manually, offer to save input as a deps kit
  // 5. Save 'package.json'
  // 6. Perform 'npm install'
}
```

### Автор

Первым делом, определим автроа. Поскольку изначально наш массив авторов по умолчанию пуст, интерфейс должен предложить ввести имя вручную и сохранить для выбора в следующий раз. Это из разряда тех вещей, которые после установки заполняются только однажды. В процессе будем использовать библиотеку `inquirer`, которая будет "опрашивать" пользователя в командной строке.

> `lib/config-dependencies.js`
```javascript
async function defineAuthor() {

  // вопрос #1: Выбрать автора из списка или ввести вручную
  const questions_PICK = [
    {
      type: "list",
      message: "Pick an author",
      name: "_author",
      choices: CONFIG.defaultAuthors.concat("input manually") // Вариант "ввести вручную" будет в любом случае, даже если массив авторов пуст
    }
  ]
  
  // вопрос #2 - ввод имени вручную
  const questions_INPUT = [
    {
      type: "input",
      message: "Input your name",
      name: "author"
    }
    , // вопрос #3 - предлагаем сохранить свое имя в настройках по умолчанию
    {
      type: "confirm",
      message: "Do you want your name to be saved?",
      name: "storeName",
      default: true
    }
  ]

  // функции, задающие вопросы и возвращающие промисы с ответами
  const PICK = () => inquirer.prompt(questions_PICK);
  const INPUT = () => inquirer.prompt(questions_INPUT);

  //   1.1. Select from list of default authors or 'input manually'
  const { _author } = await PICK();

  //   1.2. If 'input manually' - offer to store in defaults.
  // если "ввести вручную"
  if (_author === "input manually") { 

    const { author, storeName } = await INPUT();

    if (storeName) {
      CONFIG.defaultAuthors.push(author); // <- добавляем автроа в список
      rewriteSettings(CONFIG, 'deps'); // <- переписываем конфиг
    }
    package_json['author'] = author; // <- создаем поле "author" в переменной package_json
    return author;
  }
  package_json['author'] = _author; // <- создаем поле "author" в переменной package_json
  return _author;
}

async function main() {
  // Plan: 
  // 1. Define author by default
  let author = await defineAuthor();
  console.log(`Author (from MAIN): ${author}`);
  
  // в качестве теста вернем автора в точку входа (index.js)
  return author;
  // ...
}

module.exports = main;
```

![Cold Run](https://raw.githubusercontent.com/alexnaidovich/blog/blog-images/Runner_03-01.JPG)

Как видим из скриншота, интерфейс добавил в (изначально пустой) массив `defaultAuthors` мое имя. При следующем запуске интерфейс уже предложит мне его выбрать.

![Pick an author](https://raw.githubusercontent.com/alexnaidovich/blog/blog-images/Runner_03-02.JPG)

Первый пункт плана можно считать выполненным. Идем дальше.
> Главное - не забыть в README.md указать, чтобы пользователь не набирал себе имя "input manually" :)

### `"name"` и `"description"`

Эти данные будут вводиться каждый раз при генерации каждого нового проекта. Принцип действия такой же - здесь `inquirer` задаст 2 вопроса типа `input`. На что здесь стоит обратить внимание - это на поле `"validate"` в вопросе про название проекта. Это функция-валидатор, которая принимает на себя введенное значение и не пропускает его, если оно пустое или в нем есть какие-либо символы, кроме латинских букв, цифр или дефиса.
> `lib/config-dependencies.js`
```javascript
async function setNameAndDescription() {
  const questions = [
    {
      type: "input",
      message: "Input the name of your project",
      name: "name",
      validate: value => value !== "" && !(/[^a-zA-Z0-9\-]/g.test(value))
    },
    {
      type: "input",
      message: "Input the description of your project",
      name: "description"
    }
  ]

  const INPUT = () => inquirer.prompt(questions);
  const { name, description } = await INPUT();
  Object.assign(package_json, { name, description }); // <- добавляем поля "name" и "description" в объект package_json

  return [ name, description ];
}

module.exports = async function main() {
  // Plan: 
  // 1. Define author by default
  let author = await defineAuthor();
  
  // 2. Give name and description to the project
  let [ name, description ] = await setNameAndDescription();

  // test
  return [ author, name, description, JSON.stringify(package_json, null, 2) ];
  // ...
}

module.exports = main;
```

Из главной функции в точку входа возвращаем все имеющиеся поля по отдельности, и объект `package_json`, пропущенный через `JSON.stringify()`. И как видно из скрина, всё приходит как надо.

![Author, name and description](https://github.com/alexnaidovich/blog/blob/blog-images/Runner_03-03.JPG?raw=true)
