# Runner - здесь точно будет работа с зависимостями...

Метод проб и ошибок очень хорош. Но он не очень подходит в том случае, когда ты хочешь написать в блоге о том, как у тебя всё хорошо и ровно получается. Но это не про наш случай. Поэтому сейчас будет краткий разбор ошибок предыдущих частей.

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

Теперь к самому методу. Поскольку мы разделили наши настройки на отдельные файлы для каждого компонента, сохранятьфайлы придется теперь тоже раздельно. Согласно моему восприятию такого понятия, как naming convention, мои файлы настроек имеют вид **`${назначение_конфига}.config.json`**. И соответственно, функиця должна принимать это `назначение конфига` входным параметром. Назоыем его `typeOfConfig`, и с ним функция будет иметь следующий вид:
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

## А теперь к `lib/config-dependencies.js`

