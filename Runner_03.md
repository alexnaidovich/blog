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

## Выстроен костяк компонента.
