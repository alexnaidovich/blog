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

## Составлен список всех вопросов компонента, и этот список вынесен отдельный файл.

## Подготовлены и импортированы необходимые хелперы.

## Выстроен костяк компонента.
