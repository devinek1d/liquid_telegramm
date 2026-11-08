# Glassgram: план облачной Windows-сборки

Дата проверки: 2026-11-08. Цель — собрать x64 Windows-версию на временном облачном Windows runner, сохраняя локально только исходники и небольшой итоговый артефакт. Сборка не запускалась; workflow и облачный ресурс не создавались.

## Вывод

Технически выполнимо через GitHub Actions в публичном репозитории `devinek1d/liquid_telegramm`: стандартный публичный Windows runner бесплатен, имеет 4 CPU, 16 GB RAM, но всего 14 GB SSD. Для Telegram Desktop подготовка многочисленных сторонних библиотек и Qt плюс VS Build Tools делает этот диск главным риском. Сначала разумен один x64 пробный запуск на стандартном `windows-2025-vs2026`; перед ним workflow должен измерять свободное место и завершаться с понятным сообщением, если его недостаточно. Более надёжный вариант — GitHub-hosted larger Windows runner (150 GB SSD на базовой конфигурации), доступный организациям/enterprise с Team или Enterprise Cloud. Не включать его без проверки плана и стоимости. Альтернатива — временная Windows VM с удалённым запуском и удалением после выгрузки артефакта.

## Основание

- `upstream/docs/building-win.md` задаёт Visual Studio 2026, SDK `10.0.26100.0`, Python 3.10 и для x64 MSVC `-vcvars_ver=14.44`. Подготовка выполняется `Telegram/build/prepare/win.bat`, затем `configure.bat x64` и build решения `out/Telegram.slnx`.
- `upstream/.github/workflows/win.yml` уже демонстрирует подход upstream: checkout с recursive submodules, отдельные `TBuild/ThirdParty` и `Libraries`, SDK читается из build guide, toolset 14.44, кэш третьих библиотек/библиотек/Qt, затем сборка Debug и загрузка `Telegram.exe`/`Updater.exe`. Workflow имеет матрицу нескольких архитектур/Qt/генераторов и шаг очистки диска; для Glassgram следует взять только x64-путь, не копировать матрицу целиком.
- Workflow upstream в некоторых событиях использует специализированный `depot-windows-latest-16`, а в обычном пути — `windows-latest`; это не доказывает достаточность 14 GB для чистой полной сборки. Их очистка части временных файлов и кэширование показывают, что место важно.
- По GitHub Docs: стандартный Windows runner публичного репозитория — 4 CPU, 16 GB RAM, 14 GB SSD; публичные стандартные hosted runners бесплатны. Larger runners требуют GitHub Team или Enterprise Cloud; таблица размеров включает Windows 4 CPU/16 GB/150 GB. Состав образов `windows-2025-vs2026` документирует компонент MSVC 14.44 и Visual Studio 2026. Версии образов меняются, поэтому перед запуском сверить runner image README.

## Минимальный план реализации

1. Добавить отдельный `workflow_dispatch` workflow в Glassgram, ограниченный x64 и ручным запуском. `runs-on: windows-2025-vs2026`; не запускать автоматически на каждый push. Checkout текущего репозитория с submodules recursive. Учитывать, что сторонние actions в upstream (`actions/checkout@v7`, `actions/cache@v6`, `actions/upload-artifact@v7`) могут требовать версий, доступных в аккаунте/момент запуска; при необходимости использовать актуальные стабильные версии.
2. В начале логировать `Get-PSDrive`/свободное место и версии Python, CMake, Visual Studio/SDK, MSVC. До тяжёлых операций проверить порог места. Резервировать достаточно места под checkout, зависимости, сборочную папку, кэш и артефакт; если стандартный runner не проходит проверку — остановиться до загрузки крупных зависимостей и перейти к larger runner или VM.
3. Использовать `upstream/Telegram/build/prepare/win.bat`, configure x64 и Release build по upstream guide. Работать в `$RUNNER_TEMP`/локальном диске runner, чтобы данные исчезали вместе с VM. Использовать изолированный кэш зависимостей с ключом от версии prepare script, SDK, архитектуры и toolset; учесть, что холодный запуск всё равно скачивает/собирает библиотеки.
4. Передавать Telegram API ID/hash как GitHub Actions secrets только после получения собственных credentials. Прокинуть значения в configure аккуратно, не печатать команду с раскрытыми значениями; не сохранять build directory/CMake cache как artifact/cache. В опубликованном upstream workflow присутствуют встроенные общие API значения — не переносить их в Glassgram и не считать заменой личным credentials.
5. Загрузить только нужные Release EXE(шники) как Actions artifact с коротким сроком хранения; скачать артефакт локально. Не публиковать Releases и не менять репозиторий/настройки без отдельной задачи.

## Блокеры и ограничения

- Личные API credentials не предоставлены; облачный build ещё не настроен и не запускался. Поэтому невозможно проверить доступность runner, свободное место, завершение сборки и точный размер артефакта.
- 14 GB SSD на стандартном hosted runner — неподтверждённая достаточность для чистой сборки; стартовать полный build без предварительной оценки/проверки диска рискованно. GitHub larger runners решают ограничение места, но требуют поддерживаемого плана и оплачиваемого ресурса.
- Кэш Actions ускоряет повторные прогоны, но имеет лимиты, может быть вытеснен и не снимает потребность в месте на холодном прогоне. Не обещать, что он позволит уложиться в обычный runner.
- `windows-2025-vs2026` содержит VS 2026 и компонент MSVC 14.44 по опубликованному image inventory, но точный snapshot обновляется; workflow должен печатать реальные версии и падать, если нужный toolset/SDK отсутствует.
- Пользователь проекта предпочитает облако и минимум локальных действий: это сохраняет локально checkout и скачанный результат, однако первые зависимости всё равно нужно скачать в эфемерный runner, а результат проверить после загрузки.

## Официальные ссылки

- [GitHub Docs: выбрать runner для job](https://docs.github.com/en/actions/how-tos/write-workflows/choose-where-workflows-run/choose-the-runner-for-a-job) — labels, характеристики, бесплатность публичных стандартных runners.
- [GitHub Docs: Larger runners reference](https://docs.github.com/en/actions/reference/runners/larger-runners) — размеры CPU/RAM/SSD.
- [GitHub Docs: доступность larger runners](https://docs.github.com/en/actions/how-tos/manage-runners/larger-runners/use-larger-runners) — требование GitHub Team/Enterprise Cloud.
- [GitHub Actions runner images: Windows Server 2025 VS2026 inventory](https://github.com/actions/runner-images/blob/main/images/windows/Windows2025-VS2026-Readme.md) — актуальный состав образа, VS/MSVC.
- [GitHub runner image labels и миграции](https://github.com/actions/runner-images) — актуальные labels и изменения образов.

