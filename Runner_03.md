# Runner - здесь точно будет работа с зависимостями...

### Вместо вступления:
![Test Runner CLI Stage 2](https://raw.githubusercontent.com/alexnaidovich/blog/blog-images/deps_01.gif)

Данная заметка переписана с нуля. Если вдруг интересно, [ее контент доступен в предыдущем ее коммите](https://github.com/alexnaidovich/blog/blob/436faf9915886b97ca1210733597c179d362909c/Runner_03.md). Но с момента ее написания на проект обвалилась целая гора оптимизации и рефактора (хотя то, что там есть сейчас - тоже далеко не идеал). 

Пункт большого плана касательно работы с зависимостями выполнен. Что изменилось:

1. [Общий конфиг `settings.json` разбит на отдельные файлы для каждого компонента](https://github.com/alexnaidovich/blog/blob/master/Runner_03.md#%D0%BE%D0%B1%D1%89%D0%B8%D0%B9-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3-settingsjson-%D1%80%D0%B0%D0%B7%D0%B1%D0%B8%D1%82-%D0%BD%D0%B0-%D0%BE%D1%82%D0%B4%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D1%84%D0%B0%D0%B9%D0%BB%D1%8B-%D0%B4%D0%BB%D1%8F-%D0%BA%D0%B0%D0%B6%D0%B4%D0%BE%D0%B3%D0%BE-%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%B0).
2. [Составлен список всех вопросов компонента, и этот список вынесен отдельный файл](https://github.com/alexnaidovich/blog/blob/master/Runner_03.md#%D1%81%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD-%D1%81%D0%BF%D0%B8%D1%81%D0%BE%D0%BA-%D0%B2%D1%81%D0%B5%D1%85-%D0%B2%D0%BE%D0%BF%D1%80%D0%BE%D1%81%D0%BE%D0%B2-%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%B0-%D0%B8-%D1%8D%D1%82%D0%BE%D1%82-%D1%81%D0%BF%D0%B8%D1%81%D0%BE%D0%BA-%D0%B2%D1%8B%D0%BD%D0%B5%D1%81%D0%B5%D0%BD-%D0%BE%D1%82%D0%B4%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9-%D1%84%D0%B0%D0%B9%D0%BB).
3. [Подготовлены и импортированы необходимые хелперы](https://github.com/alexnaidovich/blog/blob/master/Runner_03.md#%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D1%8B-%D0%B8-%D0%B8%D0%BC%D0%BF%D0%BE%D1%80%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D1%8B-%D0%BD%D0%B5%D0%BE%D0%B1%D1%85%D0%BE%D0%B4%D0%B8%D0%BC%D1%8B%D0%B5-%D1%85%D0%B5%D0%BB%D0%BF%D0%B5%D1%80%D1%8B).
4. [Выстроен костяк компонента](https://github.com/alexnaidovich/blog/blob/master/Runner_03.md#%D0%B2%D1%8B%D1%81%D1%82%D1%80%D0%BE%D0%B5%D0%BD-%D0%BA%D0%BE%D1%81%D1%82%D1%8F%D0%BA-%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%BD%D0%B5%D0%BD%D1%82%D0%B0).

## Общий конфиг `settings.json` разбит на отдельные файлы для каждого компонента.

`settings.json` больше в рабочем проекте не существует. Физически он остался в качестве черновика. Все файлы настроек будут храниться в папке `config`, и в этой заметке мы уделим внимание файлу `deps.config.json`. [Здесь можно взглянуть на его содержимое](https://github.com/alexnaidovich/runner/blob/master/config/deps.config.json), и все его поля нам пригодятся в работе.

## Составлен список всех вопросов компонента, и этот список вынесен отдельный файл.

В данном компоненте (как, впрочем, и во всех остальных), будет тесно использоваться зависимость `inquirer`. Это удобный интерфейс для опросов в командной строке. Проблема с ним состоит в том, что сами вопросы для `inquirer`'a - объекты в массивах. И выглядят они массивно. Когда я их описывал в основном файле компонента, файл (в котором преимущественно должна быть логика) сильно раздулся. Поэтому я всем скопом вынес их в [отдельный файл](https://github.com/alexnaidovich/runner/blob/master/lib/inquirer-questions.js). В нем будут также храниться вопросы для остальных компонент. Вид имеет примерно следующий:

```javascript
const path = require('path');
const dir_config = path.resolve(__dirname, '..', 'config');
const [ deps_config ] = [
  'deps.config.json'
].map(config => require(path.join(dir_config, config)));

module.exports = {
  config_dependencies: {
    author_pick: [
      {
        type: "list",
        message: "Pick an author",
        name: "_author",
        choices: deps_config.defaultAuthors.concat("input manually")
      }
    ],
    author_input: [
      {
        type: "input",
        message: "Input your name",
        name: "author"
      },
      {
        type: "confirm",
        message: "Do you want your name to be saved?",
        name: "storeName",
        default: true
      }
    ],
    /* целая куча остальных вопросов */
  } // конец вопросов для компонента
}
```

## Подготовлены и импортированы необходимые хелперы.

Файл с помощниками `lib/helpers.js` претерпел, пожалуй, самые серьезные изменения. В нем появились три важных для работы метода:

  * `scanCachedDevDependencies()` - на случай, если определенный набор дев-зависимостей установленн где-нибудь на HDD/SSD (мой случай) - можно просто ссылаться на путь, где они установлены. Функция принимает на себя путь к установленным зависимостям, а возвращает промис с полем `devDependencies` для создаваемого `package.json`;
  
```javascript
/**
 * Gets path to pre-installed devDependencies and and returns 'devDependencies' object
 * linked to those packages for new 'package.json' output.
 * 
 * @param {String} pathToDependencies Path to installed npm packages
 * @returns {Promise<Object | String<Error>>} Resolved mapped devDeps or rejected error message 
 */
module.exports.scanCachedDevDependencies = pathToDependencies => {
  //  превращаем входной параметр (строку с путем) в 100% абсолютный путь.
  const _path = path.resolve(pathToDependencies);

  //  создаем новый объект поля devDependencies, в котором прописываем ссылочные пути к зависимостям.
  //  хелпер для основной части.
  const mapDeps = devDeps => {
    const returnObj = {};
    Object.keys(devDeps).forEach(dep => {
      returnObj[dep] = `file:${_path.replace('\\', '/')}/node_modules/${dep}`;
    });
    return returnObj;
  }

  //  основная часть. считывает директорию, указанную в пути. 
  //  если там нет файла package.json, реджектится. при других ошибках тоже.
  //  в обратном случае резловится в готовый объект devDependencies.
  const getDeps = () => {
    return new Promise((resolve, reject) => {
      fs.readdir(_path, (err, files) => {   
        if (err) {
          reject(err);
        } else if (!files.find(file => file === 'package.json')) {
          reject(`No 'package.json' found at the folder ${_path}.`);
        } else {
          const { devDependencies } = require(path.join(_path, 'package.json'));
          resolve(mapDeps(devDependencies));
        }
      });
    });
  }

  //  возвращаем немедленный запуск основной части.
  return getDeps();
}
```
  
  * `savePackageJson()` - эта функция сохраняет файл `package.json` в текущей рабочей директории. Принимает на себя объект-содержимое файла, а возвращает промис, который разрешается в сообщение о сохранении при успехе или в строку с ошибкой, если вдруг неуспех.
  
```javascript
/**
 * Stores 'package.json' file on disc
 * 
 * @param {Object} package_json 'package.json' content
 * @returns {Promise<String>} Promise containing state of file saving
 */
module.exports.savePackageJson = package_json => {
  //  получаем путь к текущей рабочей директории
  const cwd = process.cwd();
  
  return new Promise((resolve, reject) => {
    fs.writeFile(`${cwd}/package.json`, JSON.stringify(package_json, null, 2), error => {
      if (error) {
        reject(error);
      }
      resolve(`"package.json" saved.`);
    });
  });
}
```

  * `npmInstall()` - функция с говорящим названием. Первым аргументом идет булевое значение `isDevDeps` - если приходит `true`, функция будет знать, что выполнять команду надо с флагом `--save-dev`, в обратном случае - `--save`. Вторым аргументом принимается массив зависимостей. Если он пустой и в сохраненном (уже к этому времени) `package.json` ничего в полях `dependencies`/`devDependencies` не прописано, установка происходить не должна. Здесь внутри функции мы собираемся выполнить `bash`-команду, и для этого нам пригодится метод `exec()`, который мы импортируем из модуля `child_process`, входящего в состав базовой комплектации Node.js. Этот метод создает дополнительный поток для исполнения bash/shell команд, поскольку основной поток занят выполнением нашей CLI. Также, поскольку все знают, что сам процесс выполнения `npm install` может быть очень долгим, понадобится какая-то индикация процесса, чтобы не возникло ощущение, что "компьютер завис". Для этой цели я выбрал спиннер от библиотеки `ora` (ещё одна внешняя зависимость в проект). По сложившейся традиции, для работы в среде `async/await`, возвращаем промис.
  
```javascript
// спиннер - индикатор процесса
const ora = require('ora');
// метод exec() стандартного модуля child_process
const { exec } = require('child_process')

/**
* Performs 'npm install'.
* Returns resolved promise on success or rejected promise on failure.
* 
* @param {Boolean} isDevDeps defines if the deps are dev (true) or prod (false).
* @param {Array} deps Array of packages
* @returns {Promise<Boolean> | Promise<String>} 
*/
module.exports.npmInstall = (isDevDeps, deps = []) => {
  //  соединяем массив зависимостей в строку, разделенную пробелом
  let _deps = deps.length ? deps.join(' ') : '';
  //  определяем, будем ли мы устанавливать дев- или прод-зависимости
  let _savedev = isDevDeps ? '--save-dev' : '--save';
  
  return new Promise((resolve, reject) => {
    //  запускаем спиннер
    const spinner = ora('Installing dependencies via npm install. It may take a while...').start();
    
    //  запускаем параллельный поток shell-оболочки, в котором исполняются команды
    const npminstall = exec(`npm install ${_savedev} ${_deps}`, (error, stdout, stderr) => {
      //  если ошибка на запуске - останавливаем спиннер и реджектимся
      if (error) {
        spinner.stop();
        console.log(error);
        reject(error);
      }
    });
    
    //  подписываемся на событие 'message' параллельного потока для трансляции его сообщений в консоль основного потока
    //  должен признать, это не сработало, хотя пытался добиться разными способами.
    npminstall.on('message', message => process.stdout.write(message));
    
    //  подписываемся на событие выхода из параллельного потока при завершении в нём команды 'npm install'
    npminstall.on('exit', (code, signal) => {
      //  если в процессе что-то пошло не так (код выхода из потока - не ноль)
      if (code !== 0) {
        //  останавливаем спиннер и реджектимся
        spinner.stop();
        reject(`Exit child process with non-zero code: ${code};\n${signal}`);
      } else {
        //  в случае успеха - показываем сообщение об успехе и резолвимся. и тоже останавливаем спиннер :)
        console.log(chalk.yellow(`\nnpm install finished with code ${code} and status ${signal}`));
        spinner.stop();
        resolve(true);
      }
    })
  });
}
```

## Выстроен костяк компонента.

Проще и последовательнее всего будет объяснить комментариями в коде.

> `lib/config-dependencies.js`

```javascript
//  импортируем inquirer - он будет мучить нас вопросами
const inquirer = require('inquirer');

//  импортируем наш конфиг, ...
const CONFIG = require('../config/deps.config.json');

//  ...наши хелперы, ...
const { 
  rewriteSettings,
  scanCachedDevDependencies,
  savePackageJson,
  npmInstall
} = require('./helpers');

// ... и наши вопросы для inquirer'а.
const {
  author_pick,
  author_input,
  name_and_description,
  is_dev_deps_cached,
  choose_dev_deps_kit,
  input_dev_deps,
  choose_dev_deps_cached_path,
  input_dev_deps_cache_path,
  choose_deps_kit,
  input_deps
} = require('./inquirer-questions').config_dependencies;

//  создаем пустой объект - содержимое будущего файла 'package.json'.
//  по ходу исполнения кода будем наполнять его информацией.
const package_json = {};

/**
 * локальный хелпер - для тех случаев, когда пользователь выбрал вариант "ввести вручную".
 * после ввода должно быть предложение сохранить ввод для возможности выбрать его в следующий раз
 * @param {String} choice Ответ на предыдущий вопрос
 * @param {Array} questions Список следующих вопросов - массив из двух вопросов
 * @param {String} configArray Параметр конфига, который подлежит перезаписи
 * @param {String} inputKey 
 * @param {String} confirmKey 
 * @returns {String} Final answer
 */
async function ifInputManually(choice, questions, configArray, inputKey, confirmKey) {
  //  если ответ на предыдущий вопрос - "ввести вручную"
  if (choice === "input manually") {
    //  функция, которая задаёт вопросы
    const INPUT = () => inquirer.prompt(questions);
    
    //  ответы = это результат выполнения этой функции. 
    //  это будет объект с двумя полями - inputKey (строка, результат ввода)
    //  и confirmKey (булевое, сохранять ли результат ввода в конфиге)
    const answers = await INPUT();
    
    //  если решено сохранять результат ввода
    if (answers[confirmKey]) {
      //  добваляем результат ввода в соответствующее поле импортированного конфига
      CONFIG[configArray].push(answers[inputKey]);
      //  и переписываем файл конфига
      rewriteSettings(CONFIG, 'deps');
    }
    //  возвращаем результат ввода
    return answers[inputKey];
  }
  
  //  а если ответ на предыдущий вопрос - не "ввести вручную" - возвращаем результат предыдущего вопроса
  return choice;
}

//  Функция для выбора/заполнения автора
async function defineAuthor() {
  const PICK = () => inquirer.prompt(author_pick);
  const { _author } = await PICK();
  
  //  поскольку есть вероятность ответа "ввести вручную",
  //  возвращаем немедленное выполнение хелпера ifInputManually()
  return await ifInputManually(
    _author,
    author_input,
    'defaultAuthors',
    'author',
    'storeName'
  );
}

//  Функция для заполнения названия и описания проекта
async function setNameAndDescription() {
  const INPUT = () => inquirer.prompt(name_and_description);
  const { name, description } = await INPUT();
  
  //  добавляем в объект package_json полученные ответы
  Object.assign(package_json, { name, description });
  
  //  возвращаем полученные ответы (хотя это и необязательно)
  return [ name, description ];
}

//  Функция, которая определит, установлены ли дев-зависимости где-либо локально
async function defineIfDevDepsCached() {
  const INPUT = () => inquirer.prompt(is_dev_deps_cached);
  const { isDevDepsCached } = await INPUT();
  return isDevDepsCached;
}

//  Функция, в которой указываешь, где установлены дев-зависимости.
async function _chooseDevDepsCachedPath() {
  const CHOOSE = () => inquirer.prompt(choose_dev_deps_cached_path);
  const { chooseDevDepsCachePath } = await CHOOSE();
  return await ifInputManually(
    chooseDevDepsCachePath, 
    input_dev_deps_cache_path,
    'devDepsCachePaths',
    'inputDevDepsCachePath',
    'saveDevDepsCachePath'
  );
}

//  Функция, в которой выбираешь, какой комплект дев-зависимостей устанавливать (или ввести)
async function chooseDevDepsKit() {
  const CHOOSE = () => inquirer.prompt(choose_dev_deps_kit);
  const { chooseDevDepsKit } = await CHOOSE();
  return await ifInputManually(
    chooseDevDepsKit,
    input_dev_deps,
    'devDepsKits',
    'inputDevDeps',
    'saveDevDeps'
  );
}

//  Функция, в которой выбираешь, какой комплект прод-зависимостей устанавливать (или ввести)
async function chooseDepsKit() {
  const CHOOSE = () => inquirer.prompt(choose_deps_kit);
  const { chooseDepsKit } = await CHOOSE();
  return await ifInputManually(
    chooseDepsKit,
    input_deps,
    'depsKits',
    "inputDeps",
    "saveDepsKit"
  );
}

//  Костяк главной функции, который экспортируется в точку входа
//  Для мягкого сглаживания ошибок оборачиваем всё её тело в блок try-catch
module.exports = async function main() {
  try {    
    // 1. Определяем автора и сохраняем его в переменной package_json
    const author = await defineAuthor();
    Object.assign(package_json, { author });
    
    // 2. Даём проекту имя и описание
    await setNameAndDescription();

    //   2.1. ...а также версию и баш-скрипты для webpack'а
    let { version, scripts } = CONFIG.examplePackageJson;
    Object.assign(package_json, { version, scripts });
    
    // 3. Работаем с дев-зависимостями
    //   3.1. Определяем, установлены ли они где-нибудь на компе
    let isDevDepsCached = await defineIfDevDepsCached();
    
    //     3.1.1. Если да, указываем путь к ним, и формируем объект devDependencies для package_json
    if (isDevDepsCached) {
      let cachePath = await _chooseDevDepsCachedPath();
      let devDeps = await scanCachedDevDependencies(cachePath);
      
      //  присваиваем поле devDependencies в переменную package_json
      Object.assign(package_json, {devDependencies: devDeps});
      
      //  сохраняем файл package.json в текущей рабочей директории
      await savePackageJson(package_json);
      
      //  и там же выполняем npm install
      await npmInstall(true);
    } else {
      //  3.2. В противном случае - 
      //  сохраняем файл package.json в текущей рабочей директории
      await savePackageJson(package_json);
      
      //   выбираем/вводим дев-зависимости
      let _devDepsKit = await chooseDevDepsKit();
      let devDepsKit = _devDepsKit.split(' ');

      //  Если выбор не пуст - выполняем 'npm install'
      if (devDepsKit.length > 0) {
        await npmInstall(false, devDepsKit);
      } else {
        console.log(`No dev dependencies installed. You may want to install them manually.`);
      }
    }

    // 4. выбираем/вводим прод-зависимости
    //   все так же, как и с предыдущим пунктом
    let _depsKit = await chooseDepsKit();
    let depsKit = _depsKit !== '' ? _depsKit.split(' ') : new Array(0);

    if (depsKit.length > 0) {
      await npmInstall(false, depsKit);
    } else {
      console.log(`No prod dependencies installed. You may want to install them manually.`);
    }

    //  возвращаем сообщение об успешной установке зависимостей в точку входа
    return 'Dependencies Configured Successfully.';
  } catch (error) {
    //  или сообщение об ошибке, если что-то пошло не так.
    return `Failed to configure dependencies: \n${error}`;
  }
}
```

На этом работу надо компонентом `config-dependencies.js` можно считать завершенной.

Спасибо за внимание.
