# Glassgram: аудит официальной сборки

Дата аудита: 2026-11-08. Источник: `upstream/`, отдельный клон Telegram Desktop `https://github.com/telegramdesktop/tdesktop.git`; аудит публикуется в devinek1d/liquid_telegramm, исходники закреплены подмодулем.

## Состояние исходников

- Ветка `dev`, `HEAD 7381fb1df75d863eb3463fdddef57a2ac51f1d56` (2026-11-06, `[ai] Track widget dependencies and QuickJS upstream fixes`), remote `origin` указывает на официальный репозиторий.
- `git status --short --branch`: `## dev...origin/dev` — рабочее дерево upstream чистое.
- По индексу `git ls-files -s` содержит gitlink-записи подмодулей с закреплёнными SHA; `.gitmodules` задаёт их официальные URL. Полный `git submodule status` не завершился: встроенный MSYS Git упал с `NtCreateDirectoryObject ... 0xC0000022`. Поэтому наличие/инициализацию каждого рабочего дерева подмодуля этим аудитом не подтверждаем. `git ls-files -s` подтвердил закреплённые ссылки, не состояние checkout подмодулей.
- `README.md` описывает Telegram Desktop, GPLv3 с исключением OpenSSL, перечисляет Windows 7+ x86/x64 как поддерживаемые upstream платформы и направляет к `docs/building-win.md`.
- В корне upstream присутствует `AGENTS.md`; для `Telegram/SourceFiles` есть локальный `AGENTS.md`. Учтены их указания о разделении ответственности UI, стилей, хранения и updater. Руководство репозитория говорит о VS 2022 в общей платформенной заметке, но текущий конкретный Windows рецепт в `docs/building-win.md` требует VS 2026; для данного pinned checkout источник истины — конкретный build guide.

## Официальные требования и команды (не запускались)

`upstream/docs/building-win.md` требует пустую build-папку, `ThirdParty` и `Libraries`, Visual Studio 2026, Windows SDK `10.0.26100.0`, Python 3.10 в PATH, Git и x64 Native Tools environment. Для x86/x64 задан toolset `-vcvars_ver=14.44`; x64 и ARM64 имеют отдельные команды и настройки. Рекомендуемый путь — x64. Нужны личные `api_id`/`api_hash`, получаемые через Telegram API; их нельзя помещать в репозиторий или лог.

Официальная последовательность из документации (команды ниже приведены для пояснения и не исполнялись):

```bat
git clone --recursive https://github.com/telegramdesktop/tdesktop.git
tdesktop\Telegram\build\prepare\win.bat
cd tdesktop\Telegram
configure.bat x64 -D TDESKTOP_API_ID=<api_id> -D TDESKTOP_API_HASH=<api_hash>
```

Далее открыть `tdesktop\out\Telegram.slnx` в Visual Studio 2026, выбрать проект Telegram и собрать Debug/Release. Ожидаемый официальный артефакт — `tdesktop\out\Debug\Telegram.exe` и Release аналог. Qt Visual Studio Tools в гайде необязательны. Подробные локальные сведения о Windows SDK, Visual Studio, Python и диске находятся в [GLASSGRAM_ENVIRONMENT.md](GLASSGRAM_ENVIRONMENT.md).

## Результат сборки

Сборка, подготовка библиотек, конфигурация, запуск и тесты **не выполнялись**. Бинарник не создан, baseline launch не подтверждён. Главный установленный blocker по соседнему отчёту среды: Visual Studio/MSVC и требуемый Windows SDK отсутствуют; Python — 3.12 вместо указанной 3.10. Даже после установки инструментов останется получить и безопасно передать API credentials. Задание допускает сборку при наличии предпосылок; здесь они не выполнены. Никакие build команды не выдаются за проверенные.

## Проверенные точки входа исходного кода

Пути модулей ниже относительно `upstream/Telegram/SourceFiles/`, кроме явно начинающихся с `Telegram/` (они относительно `upstream/`).

| Подсистема | Пути / роль |
|---|---|
| Главное окно и композиция | `Telegram/SourceFiles/mainwidget.h`, `mainwidget.cpp`; контроллеры окна — `window/window_controller.*`, `window/window.*` |
| Список чатов | `dialogs/dialogs_widget.h`, `.cpp`, `.style`; раскладка/элементы также в `dialogs/dialogs.*` |
| Просмотр переписки | `history/history_widget.h`, `.cpp`, `history/history_inner_widget.*`, `history/view/history_view_element.*` |
| Композер сообщения | `history/view/controls/history_view_compose_controls.h`, `.cpp`; поле — `chat_helpers/message_field.h`, `.cpp` |
| Тема и стили | `ui/td_common.style`, `ui/color_indices.style`, `settings/settings.style`, а также локальные `.style` модулей `dialogs/`, `window/`, `history/view/` и `boxes/`. Генерация стилей конфигурируется CMake; расширять набор следует после изучения соответствующего AGENTS. |
| Меню / popup | `window/window_main_menu.h`, `.cpp`, `.style`; peer/context menus — `window/window_peer_menu.*`, `history/view/history_view_context_menu.*`, общие меню — `menu/` |
| Анимации | UI использует специализированные классы/компоненты; область перехода чатов находится в `window/window_chat_switch_process.*`; анимации и эффекты переписки в `history/view/` и `ui/effects/`. Ресурсные анимации — `Telegram/Resources/animations/`. |
| Настройки | `settings/sections/settings_main.h`, `.cpp`, `settings/settings.style`; точка интеграции настроек главного окна в `settings/sections/`. |
| Идентичность / запуск Windows | `core/launcher.cpp` задаёт Qt app name `TelegramDesktop`; Windows интеграция — `platform/win/launcher_win.cpp`, `platform/win/specific_win.cpp`, а также AppUserModelID/shortcut код в `platform/win/`. |
| Локальное хранилище | `storage/localstorage.h`, `.cpp`; аккаунтные данные — `storage/storage_account.*`; Windows путь `%APPDATA%\<AppName>\` формирует `platform/win/specific_win.cpp` (`psAppDataPath`). Имя приложения само по себе не является достаточным доказательством полной изоляции. |
| Updater | `core/update_checker.h`, `.cpp`, `_other/updater.h`, `_other/updater_win.cpp`; Windows target сборки определяется в `Telegram/CMakeLists.txt`. Upstream Windows updater использует `Updater.exe`, папку `Telegram Desktop` для fallback app-data и собственный update flow. |

## План изоляции профиля и идентичности

До входа в аккаунт и до одновременного запуска с официальным клиентом отдельным следующим изменением определить Glassgram identity в одном централизованном месте сборки/старта: отдельное имя бинарника и продукта, отдельный Qt/Windows AppUserModelID, отдельный каталог данных `%APPDATA%\Glassgram\` с явной миграцией выключенной по умолчанию, собственные ярлыки/иконка и notification identity. Проверить single-instance mutex, protocol/file associations, автозапуск, shell integration, логи, crash dumps и временные файлы — смены `QApplication::setApplicationName` недостаточно. Обновления отключить до собственного канала: отдельный updater/update URL, подпись/проверка целостности и rollback; нельзя направлять Glassgram на upstream Telegram канал. До проверки путей/реестра отдельный профиль пустой, production account не использовать. Изоляцию доказать параллельным запуском и проверкой фактических файлов/идентификаторов без переноса данных.

## Узкий in-app glass POC

Сначала одна поверхность (заголовок чата или панель списка), без прозрачности окна и без изменений отправки сообщений. Оставить штатный Qt paint lifecycle и нарисовать scrim + скруглённый fill с границей. Сравнить на той же поверхности и машине три метода: (A) непрозрачный fill как обязательный fallback; (B) кэшируемый downsampled snapshot только внутриприкладного фонового слоя с blur и ограниченной инвалидизацией; (C) `QGraphicsBlurEffect`/Qt effect как быстрый reference, не предполагая, что он производителен. Сначала выяснить фактический renderer/layer ownership, чтобы не снимать окно/desktop и не создавать циклический repaint. GPU shader — лишь дальнейший вариант после измерений и подтверждения доступности текущему Qt/rendering backend.

Сравнить baseline/варианты скриншотами в 1440×900 и 1920×1080, текст/контраст, resize, движущийся контент, 100%/150% DPI, reduced-motion irrelevant для blur. На релизной сборке (только после baseline) профилировать CPU/GPU, repaint count, время кадра p50/p95/p99 и dropped frames на idle, прокрутке и переключении чатов; таргет 60 FPS при 60 Hz, без регулярных пропусков кадра. Кэш обновлять только при изменении подложки/геометрии; при сбое или измеримом jank переключаться на opaque. Критерий POC: читаемость без ухудшения, изолированное семантически безопасное рисование, повторяемые замеры и мгновенный Classic/opaque fallback.

## Риски и следующий шаг

- Клон upstream не является ещё полноценным форком Glassgram; текущая папка `upstream/` — pinned source mirror/clone. Релиз приложения требует отдельной идентичности, notices и собственной API регистрации. Пользователь явно разрешил публикацию материалов аудита.
- Смена имени может не изолировать Windows shortcut, notification, updater, single-instance и протокол; требуется трассировка путей и проверка.
- Размытая поверхность может увеличить repaint и ухудшить контраст; не использовать desktop transparency или непрерывную анимацию.
- Самый маленький следующий шаг: подготовить отдельный build-host/VM с требованиями текущего `docs/building-win.md`, обеспечить приватную подачу API credentials и выполнить официальный неизменённый x64 baseline Debug/Release с полными логами и фиксацией путей артефактов. До прохождения gate не менять UI.

## Что проверено здесь

Прочитаны product plan, upstream README/LICENSE/build guide и оба найденных AGENTS.md; получены branch/SHA/status/remote, проверены submodule gitlinks и существование перечисленных source paths. Не запускались configure/build/test, не устанавливались зависимости, не открывался Telegram и не затрагивались учётные записи.


## Передача API credentials

Получить собственные значения по https://core.telegram.org/api/obtaining_api_id через my.telegram.org → API development tools. Ввести их только локально в приватном терминале сборки; не присылать в чат, не включать в Git и публикуемые логи. CMake cache и build-каталог считать чувствительными. Показанные выше параметры — placeholders, не реальные ключи.

