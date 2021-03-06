# bemhint [![Build Status](https://travis-ci.org/bem/bemhint.svg)](https://travis-ci.org/bem/bemhint) [![Dependency Status](https://david-dm.org/bem/bemhint.svg)](https://david-dm.org/bem/bemhint) [![devDependency Status](https://david-dm.org/bem/bemhint/dev-status.svg)](https://david-dm.org/bem/bemhint#info=devDependencies)

**bemhint** – это *плагинируемый* линтер БЭМ-проектов.

Данный модуль является ядром линтера, предоставляющим API для запуска и написания внешних плагинов, через которые реализуются проверки БЭМ-сущностей проекта.

## Установка

```bash
$ npm install bemhint
```

## Использование

```bash
$ bemhint --help

Использование:
    bemhint [ОПЦИИ] [АРГУМЕНТЫ]

Опции:
    -h, --help : Помощь
    -с CONFIGPATH, --config=CONFIGPATH : Путь до конфигурационного файла (по умолчанию: .bemhint.js)
    -r REPORTERS, --reporter=REPORTERS : Вид отчета с ошибками – flat и/или html (по умолчанию: flat)

Аргументы:
    TARGETS : Пути до БЭМ-сущностей для проверки (обязательный аргумент)
```

## Конфигурационный файл

JS/JSON-файл следующего формата:

```js
module.exports = {
    // cписок имен папок, которые являются уровнями переопределения (папками с блоками)
    levels: [
        '*.blocks',
        'blocks-*'
    ],

    // список путей, которые будут проигнорированы при проверке
    excludePaths: [
        'node_modules/**',
        'libs/**'
    ],

    // список подключаемых плагинов, плагины подключаются
    // относительно расположения конфигурационного файла
    plugins: {
        '<имя_плагина>': false, // плагин не будет подключен

        '<имя_плагина>': true, // обычное подключение плагина

        '<имя_плагина>': { // подключение плагина с конфигом
            // конфиг

            // список путей, которые будут проигнорированы плагином при проверке
            excludePaths: [
                'some.blocks/some-block/**',
                'some.blocks/another-block/_some-mod/*',
                'some.blocks/yet-another-block/yet-another-block.deps.js'
            ],

            techs: { // набор технологий, которые необходимо проверять плагином
                '*': false,
                'js|deps.js': true,
                'bemhtml.js': { // отдельный конфиг для технологии `bemhtml.js`
                    // конфиг
                }
            }

            // конфиг
        }
    }
};
```

**Замечание!** Конфиг для плагина может содержать не только зарезервированные поля `excludePaths` и `techs`, их наличие определяется реализацией плагина.

## Написание плагинов

JS-файл следующего формата:

```js
module.exports = {
    /**
     * Предопределенный конфиг для плагина,
     * данный конфиг будет объединен с пользовательским конфигом
     * @returns {Object}
     */
    configure: function() {
        return {
            // конфиг
        }
    },

    /**
     * @param {Entity[]} entities
     * @param {Config} config
     */
    forEntities: function(entities, config) {
        // Использование этой функции будет полезно,
        // если для реализации проверки необходимо знание
        // о всех БЭМ-сущностях проекта
    },

    /**
     * @param {Entity} entity
     * @param {Config} config
     */
    forEachEntity: function(entity, config) {
        // Использование этой функции будет полезно,
        // если для реализации проверки достаточно знание
        // о каждой из БЭМ-сущностей в отдельности
    },

    /**
     * @param {Object} tech
     * @param {Entity} entity
     * @param {Config} config
     */
    forEachTech: function(tech, entity, config) {
        // Использование этой функции будет полезно,
        // если для реализации проверки достаточно знание
        // о каждой из технологий БЭМ-сущностей в отдельности
    }
};
```

**Замечание!** Реализуемый плагин может содержать как одну из функций `forEntities`, `forEachEntity`, `forEachTech`, так и все три, при этом функция `configure` не является обязательной.

#### Entity

БЭМ-сущность проверяемого проекта (блок, элемент, модификатор).

**Entity.prototype.getTechs**<br>
**@returns** *{Object[]}* - список технологий, в которых реализована данная БЭМ-сущность

**Entity.prototype.getTechByName**<br>
**@param** *{String}* – имя технологии (css, js и т.д.)<br>
**@returns** *{Object}* – технология БЭМ-сущности

**Entity.prototype.addError**<br>
**@param** *{Object}* - объект, описывающий ошибку:
 * **msg** *{String}* – сообщение об ошибке
 * **tech** *{String}* – имя технологии, в которой найдена ошибка
 * **[value]** *{String|Object}* – значение ошибки

#### Config

Конфиг плагина.

**Config.prototype.getConfig**<br>
**@returns** *{Object}* – полный конфиг для плагина

**Config.prototype.getTechConfig**<br>
**@param** *{String}* – имя технологии (css, js и т.д.)<br>
**@returns** *{Object}* – конфиг для технологии

**Config.prototype.isExcludedPath**<br>
**@param** *{String}* – путь до технологии БЭМ-сущности<br>
**@returns** *{Boolean}*

## Примеры плагинов

* [bemhint-plugins-jshint](https://github.com/eGavr/bemhint-plugins-jshint)

* [bemhint-plugins-jscs](https://github.com/eGavr/bemhint-plugins-jscs)
