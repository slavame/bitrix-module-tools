bitrix-ws-tools
====================

Модуль WS-Tools это Инструментарий поддержки проектов компании "Рабочие Решения"


##Установка
```
/bitrix/admin/update_system_partner.php?addmodule=ws.tools
```

##Возможности модуля
###Интерфейс
* Конвертация строковых свойств в списочные или привязка по элементам без потери информации

###Функционал
* Автозагрузчик классов
* Обработка событий, вызов событий
* Простой механизм логирования данных
* Кэширование информации

##Использование функционала
Перед использованием функционала необходимо убедится в том, что модуль проинициализирован
```php
<?php
CModule::IncludeModule('ws.tools');
```

После этого становятся доступны все классы модуля
###Фасад модуля, далее модуль
Все сервисы модуля, такие как загрузчик классов и менеджер событий, доступны через главный объект "одиночку" модуля, доступен через вызов
```php
<?php
CModule::IncludeModule('ws.tools');
$toolsModule = WS\Tools\Module::getInstance();
```

Доступ ко всем сервисам происходит через метод модуля *getService($name)*
```php
<?php
CModule::IncludeModule('ws.tools');
$toolsModule = WS\Tools\Module::getInstance();
$classLoader = $toolsModule->getService('classLoader');
```

###Загрузка классов
Автоматическая загрузка классов происходит после регистрации каталога хранения классов в загрузчике модуля. Для получения загрузчика есть специальный метод *classLoader*
```php
<?php
CModule::IncludeModule('ws.tools');
$toolsModule = WS\Tools\Module::getInstance();
$classLoader = $toolsModule->classLoader();
$classLoader->registerFolder(__DIR__.'/classes');
```
Означает, что для поиска классов зарегистрирован каталог classes, находящийся в текущем каталоге файла

Классы подключаются по следующим правилам
* namespace класса помечается как путь к каталогу класса
* имя файла класса определяется как название

Таким образом класс *Custom\Network\Model* должен находится по следующему пути: *__DIR__.'Custom\Network\Model.php'*, где *__DIR__* - зарегистрированный каталог подключения классов

###Обработка событий, вызов событий
Доступ к менеджеру событий
```php
<?php
CModule::IncludeModule('ws.tools');
$toolsModule = WS\Tools\Module::getInstance();
$eventManager = $toolsModule->eventManager();
```
На данный момент не реализована возможность работы с данными по ссылке (модификация данных) 

####1. Инициализация типа события
Тип события определяется классом *\WS\Tools\Events\EventType*.
Типы событий главного модуля, инфоблоков и интернет магазина определены в виде констант, к примеру *\WS\Tools\Events\EventType::MAIN_PROLOG*  
```php
<?php
$eventManager = $toolsModule->eventManager();
$eventType = \WS\Tools\Events\EventType::create(\WS\Tools\Events\EventType::MAIN_PROLOG);
```

Для инициализаций других типов событий, необходимо прописать имя модуля и тип события при инициализации объекта, в методе *createByParams*  
```php
<?php
$eventManager = $toolsModule->eventManager();
$eventType = \WS\Tools\Events\EventType::createByParams("main", "OnPrologBefore");
$eventManager->subscribe($eventType, function ($arg1) {
    // код обработчика
});
```

####2. Регистрация обработчика события
Следующий код регистрирует обработчика события "OnProlog" модуля "main"
```php
<?php
$eventManager = $toolsModule->eventManager();
$eventType = \WS\Tools\Events\EventType::create(\WS\Tools\Events\EventType::MAIN_PROLOG);
$eventManager->subscribe($eventType, function ($arg1) {
    // код обработчика
});
```

####3. Собственный вызов события
Вызов события "OnProlog" модуля "main", так же будут вызваны все обработчики зарегистрированные не через модуль инструментов
```php
<?php
$eventManager = $toolsModule->eventManager();
$eventType = \WS\Tools\Events\EventType::create(\WS\Tools\Events\EventType::MAIN_PROLOG);
$eventManager->trigger($eventType);
```

### Логирование информации
Основой функционала логирования является [Журнал событий](https://dev.1c-bitrix.ru/community/webdev/user/11948/blog/2647/). Он позволяет журналировать собщения различного характера.
Модуль WS\Tools упрощает данную процедуру.
Пример помещения текста в Журнал событий:

```php
<?php
$log = $toolsModule->getLog("MY_CUSTOM_TYPE");
$log->severity(\WS\Tools\Log::SEVERITY_INFO)
    ->itemId(15)
    ->description("Обработка добавления элемента каталога в корзину")
    ->put();
```

Таким образом в примере выше в Журнал событий была добавлена запись произвольного типа "MY_CUSTOM_TYPE", с описанием "Обработка добавления элемента каталога в корзину", идентификатор элемента 15.

Теперь по поядку использование методов и их параметров:

* метод `getLog($type)` фасада модуля, осуществляет доступ к новому экземпляру класса логирования, где параметр `$type` определяет тип сообщения, к примеру для работы с добавлением в корзину элемента можно указать тип "ELEMENT_BASKET_ADD", его название произвольно в приложении. Зарание подготовленные типы доступны в константах `WS\Tools\Log::AUDIT_TYPE_..`
* метод `severity($value)` экземпляра логирования, задание уровня сообщения. Все уровни доступны через константы `WS\Tools\Log::SEVERITY_..`
* существуют вспомогательные методы более простого определения уровня сообщения:
+ `severityAsSecurity()` определяет сообщение как сообщение уровня безопасности. 
+ `severityAsError()` определяет сообщение как сообщение уровня ошибки. 
+ `severityAsWarning()` определяет сообщение как сообщение уровня предупреждения. 
+ `severityAsInfo()` определяет сообщение как сообщение информационного уровня. 
+ `severityAsDebug()` определяет сообщение как сообщение уровня отладки.
* `itemId($value)` устанавлевает идентификатор логируемого объекта, необязательный. Удобно использовать когда необходимо в журнале событий найти объект по идентификатору.
* `description($value)` установка описания записи, `$value` может принимать значения скаляра, массива или объекта
* `put()` отправляет сформированный объект на запись в Журнал событий

Функционал логирования поддерживает типы событий уровня ядра, например:

```php
<?php
$log = $toolsModule->getLog(\WS\Tools\Log::AUDIT_TYPE_IBLOCK_EDIT);
$log->severity(\WS\Tools\Log::SEVERITY_INFO)
    ->itemId(15)
    ->description("Обработка редактирования элемента инфоблока при вызове некого компонента")
    ->put();
```

Это означает то, что в Журнале событий колонки ITEM_ID появится запись элемента `15` со ссылкой на редактирование, а вместо кода `IBLOCK_EDIT` , будет сообщение `Изменен инфоблок`

Для получения описания типа сообщения существует событие 1С-Битрикс "OnEventLogGetAuditTypes" модуля "main". Обработчик должен фернуть массив пар ключ значение, где ключом будет являться код типа, а значением его описание. Можно также воспользоваться менеджером событий `WS\Toools`

```php
<?php
$eventManager = $toolsModule->eventManager();
$eventType = \WS\Tools\Events\EventType::create(\WS\Tools\Events\EventType::MAIN_EVENT_LOG_GET_AUDIT_TYPES);
$eventManager->subscribe($eventType, function () {
    return array(
        'MY_CUSTOM_TYPE' => "Пользовательский тип собщения"
    );
});
```

Пример, добавление сообщения об ошибке:

```php
<?php
// объект журналирования
$log = $toolsModule->getLog("WS_PROJECT_DEBUG_ERRORS");
// уровень сообщения
$log->severity(\WS\Tools\Log::SEVERITY_ERROR)
    // описание сообщения
    ->description("Обработка обработки почтового отправления, код {$postCode}")
    // сохранение сообщения
    ->put();
```

### Кэширование данных
Использование механизма кэширования из коробки Битрикса требует глубокого понимания использования.
Например практически невозможно интуитивно понять что точно делает метод `\Bitrix\Main\Data\Cache::startDataCache` с (внимание!) пятью параметрами.
На самом деле он проверяет есть ли буферизованный кэш и если его не оказалось включает буферизацию вывода. 
Тут же можно проинициализировать массив данных для использования в кеше. Такой клубок функционала программист разматывает каждый раз перед использованием кэширования данных.
`WS\Tools` предлагает упрощенную и более понятную схему кэширования данных. 

Пример (кэширование массива данных):

```php
<?php
CModule::IncludeModule('ws.tools');
$module = \WS\Tools\Module::getInstance();
// менеджер кэширования
$cm = $module->cacheManager();

//запись в кэш
$arrayWriteCache = $cm->getArrayCache('key_cache', 3000 /* cache time */);
$arrayWriteCache->set(array(
	'one', 'two', 'tree'
));

// получение данных 
$arrayReadCache = $cm->getArrayCache('key_cache', 3000);
var_dump($arrayReadCache->get() == array('one', 'two', 'tree'));
```

Кэширование вывода:

```php
<?php
CModule::IncludeModule('ws.tools');
$module = \WS\Tools\Module::getInstance();
// менеджер кэширования
$cm = $module->cacheManager();

// получение объекта кэширования вывода
$recordCache = $cm->getContentCache('testKeyCache', 200);
// проверка актуальности данных кэша
if ($recordCache->isExpire()) {
    // в случае если данные не актуальны, произведение записи новых данных
    $recordCache->record();
    echo "1222";
    // сохранение с запретом вывода данных в поток
    $recordCache->save(false);
}
// вывод в поток
echo $gettingCache->content();
```

Для реализации кэширования используются два класса: `\WS\Tools\Cache\ContentCache` и `\WS\Tools\Cache\ArrayCache`.
Класс `\WS\Tools\Cache\ContentCache` необходимо использовать для сохранения данных потока вывода, он доступен через метод менеджера кэширования `\WS\Tools\Cache\CacheManager::getContentCache`.
Класс `\WS\Tools\Cache\ArrayCache` используется для сохранения массива данных, доступен через метод `\WS\Tools\Cache\CacheManager::getArrayCache`.

При получении объектов кэширования необходимо указать параметры:

* `$key` - ключ кэширования
* `$timeLive` - время жизни кэша

#### Описание методов классов кэширования

###### Общие методы
* `setBxInitDir($value)` папка, в которой хранится кеш компонента, относительно /bitrix/cache/ (необязательный) [подробнее..](http://dev.1c-bitrix.ru/api_help/main/reference/cphpcache/initcache.php)
* `setBxBaseDir($value)` базовая директория кеша. По умолчанию равен cache (необязательный) [подробнее..](http://dev.1c-bitrix.ru/api_help/main/reference/cphpcache/initcache.php)
* `clear()` очистка хранимого кэша
* `isExpire()` проверка актуальности хранимого кэша

###### ArrayCache
* `get()` получение данных хранимого кэша
* `set($array)` сохранение данныз в кэш

###### ContentCache
* `record()` начало записи кэширования
* `abort()` прерывание записи кэширования, данные записи возвращяются в поток
* `abort()` прерывание записи кэширования, данные записи возвращяются в поток
* `stop($output = false)` сохранние записи кэширования в память, `$output` - параметр указывающий на необходимость направления данных в поток вывода, по умолчанию не распечатывается
* `content()` получение записи кэша ввиде строки 
