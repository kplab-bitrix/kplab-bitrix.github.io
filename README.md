
# Как за вечер создать модуль регистрации своих событий в Bitrix24

## 📘 Введение

В Bitrix Framework каждое важное действие (создание элемента, авторизация, обновление CRM-сущности и т.п.) сопровождается событиями. Эти события можно перехватывать, чтобы запускать свою бизнес-логику — не меняя ядро.

### Зачем регистрировать события?

- Подключение к логике ядра
- Реакция на действия пользователей и API
- Расширение стандартных сценариев без костылей

**В этом гайде ты создашь:**
- Свой модуль `kplab.events`
- Обработчик событий (например, лог входа)
- Установку и удаление модуля с регистрацией событий

---

## 🏗 Структура модуля

```
bitrix/modules/kplab.events/
├── install/
│   ├── index.php
│   ├── version.php
│   └── step.php (опционально)
├── lib/
│   └── events.php
├── include.php
└── lang/
    └── ru/install/index.php
```

---

## ⚙ Установка модуля

**install/index.php**

```php
use \Bitrix\Main\ModuleManager;
use \Bitrix\Main\EventManager;

class kplab_events extends CModule
{
    public function __construct() {
        $this->MODULE_ID = 'kplab.events';
        $this->MODULE_NAME = 'KPLab: Регистрация событий';
        $this->MODULE_DESCRIPTION = 'Модуль регистрации и обработки пользовательских событий';
        $this->MODULE_VERSION = '1.0.0';
        $this->PARTNER_NAME = 'KPLab';
    }

    public function DoInstall() {
        ModuleManager::registerModule($this->MODULE_ID);
        $this->InstallEvents();
    }

    public function DoUninstall() {
        $this->UnInstallEvents();
        ModuleManager::unRegisterModule($this->MODULE_ID);
    }

    public function InstallEvents() {
        EventManager::getInstance()->registerEventHandler(
            'main',
            'OnAfterUserAuthorize',
            $this->MODULE_ID,
            '\\KPLab\\Events\\Events',
            'onUserLogin'
        );
    }

    public function UnInstallEvents() {
        EventManager::getInstance()->unRegisterEventHandler(
            'main',
            'OnAfterUserAuthorize',
            $this->MODULE_ID,
            '\\KPLab\\Events\\Events',
            'onUserLogin'
        );
    }
}
```

---

## 📦 Логика обработчика

**lib/events.php**

```php
namespace KPLab\Events;

class Events
{
    public static function onUserLogin($arParams)
    {
        $userId = $arParams['user_fields']['ID'];
        $ip = $_SERVER['REMOTE_ADDR'];

        file_put_contents(
            $_SERVER['DOCUMENT_ROOT'].'/local/logs/login.log',
            "[".date('Y-m-d H:i:s')."] Пользователь $userId вошёл в систему с IP $ip\n",
            FILE_APPEND
        );
    }
}
```

---

## 📎 include.php и version.php

**include.php**

```php
\Bitrix\Main\Loader::registerAutoLoadClasses(
    'kplab.events',
    [
        '\\KPLab\\Events\\Events' => 'lib/events.php'
    ]
);
```

**version.php**

```php
$arModuleVersion = [
    'VERSION' => '1.0.0',
    'VERSION_DATE' => '2025-07-30',
];
```

---

## 🧪 Тестирование

1. Установи модуль через админку или консоль:

```php
$module = new kplab_events();
$module->DoInstall();
```

2. Выполни вход в систему

3. Проверь файл: `/local/logs/login.log`

---

## 🧩 Идеи для развития

- Добавить события CRM (например, `OnAfterCrmCompanyAdd`)
- Записывать в БД через ORM
- Интерфейс логов в админке
- Настройки модуля через .options.php

---

## 🔚 Заключение

Теперь у тебя есть:
- Свой модуль
- Обработка событий
- Автоматическая регистрация

**Это фундамент для любой автоматизации в Bitrix24.**

---

_KPLab | Лаборатория решений для бизнеса_

https://t.me/kplab_bitrix  
#bitrix24 #разработка #d7 #crm #kplab
