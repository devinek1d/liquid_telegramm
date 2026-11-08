# Glassgram environment audit

Audit date: 2026-11-08 (local Windows environment). Scope: read-only prerequisite inventory for Stage 0. No software was installed and no system-wide settings were changed.

## Result

The current host is **not ready for a Telegram Desktop Windows build**. Available: 64-bit Windows PowerShell environment, Git 2.56.0.windows.4, Python 3.12.10. Not found: Visual Studio / MSVC, `vswhere`, Windows 10/11 SDK, CMake, Qt tools, Python 3.10. The project plan identifies VS 2026, Windows SDK 10.0.26100.0 and Python 3.10 as the expected toolchain; those prerequisites were not observed. Disk space is 42.4 GiB free on C: (475.9 GiB total), which should not be treated as proof that a full build fits; check the current upstream build guide and expected build footprint before provisioning.

This is an environment inventory only. No configure or build command was attempted, and no binary was produced.

## Evidence

| Component | Observed result |
|---|---|
| OS/runtime | PowerShell reports Python as `WindowsPE`, 64-bit. CIM OS query was denied by the environment, so Windows edition/build was not verified here. |
| Git | `git version 2.56.0.windows.4`, executable at `C:\Users\Admin\.cache\codex-runtimes\codex-primary-runtime\dependencies\native\git\cmd\git.exe` |
| Python | `C:\Users\Admin\AppData\Local\Programs\Python\Python312\python.exe`, version 3.12.10, 64-bit |
| Python launcher | `py -0 -p` returned “No installed Pythons found!”; `py.exe` is available at `C:\Users\Admin\AppData\Local\Microsoft\WindowsApps\py.exe` |
| Visual Studio / MSVC | `vswhere.exe`, `cl.exe`, `msbuild.exe` not found on PATH; standard Visual Studio install directories absent; Visual Studio SxS registry keys absent |
| Windows SDK | `C:\Program Files (x86)\Windows Kits\10\Include` absent; `HKLM:\SOFTWARE\Microsoft\Windows Kits\Installed Roots` absent. SDK therefore not detected. |
| CMake | `cmake` not found on PATH; `C:\Program Files\CMake` absent |
| Qt | `qmake` and `qtpaths` not found on PATH; `C:\Qt` absent. Qt may be provisioned by upstream scripts later, but no local Qt installation was detected. |
| Disk | C: NTFS, 475.9 GiB total, 42.4 GiB available at audit time |

## Commands and outcomes

Commands run in PowerShell from the workspace:

```powershell
Get-Content -LiteralPath 'C:\Users\Admin\Downloads\CODEX_TASK_00.md'
Get-Content -LiteralPath 'C:\Users\Admin\Downloads\GLASSGRAM_PLAN.md'
Get-ChildItem -Force
 git status --short --branch
 git branch --show-current
 git remote -v
where.exe git
where.exe cmake
where.exe python
where.exe py
where.exe cl
where.exe msbuild
where.exe qmake
where.exe qtpaths
& 'C:\Program Files (x86)\Microsoft Visual Studio\Installer\vswhere.exe' -all -products '*' -format json
python --version
py -0 -p
python -c "import sys,platform; print(sys.executable); print(platform.python_version()); print(platform.architecture())"
[System.IO.DriveInfo]::new('C:\') | Select-Object Name,DriveFormat,TotalSize,AvailableFreeSpace
```

`where.exe` confirmed only Git and Python; the remaining executables were not found. `vswhere` and queried Visual Studio / SDK registry locations were absent. `Get-CimInstance Win32_OperatingSystem` and `Win32_LogicalDisk` failed with a CIM access error; disk figures above came from .NET `DriveInfo` instead.

## Limitations and next action

- No build was attempted because the required compiler/SDK/CMake toolchain is missing and the expected Python minor version is not present.
- Do not start visual implementation before the baseline gate. Next environment task: provision or select an approved x64 Visual Studio Developer environment and SDK, install/prepare the documented Python/CMake prerequisites through an approved route, then rerun this inventory against the actual checked-out upstream build guide. Confirm sufficient free disk space before fetching/building dependencies.
- This file records only environment evidence. It does not establish repository source paths, credentials, or a successful baseline build.

## Обновление 2026-11-08 после разрешённой установки

Пользователь разрешил установку, затем уточнил: тяжёлую сборку по возможности выполнять в облаке.

- Python 3.10.11 установлен рядом с 3.12, PATH не заменялся. Проверено `Python310/python.exe --version`; installer exit 0.
- Visual Studio Community 2026: bootstrapper завершился с code 0 (23:03 MSK); vswhere isComplete=1. Проверены MSVC 14.44.35207, Windows SDK include 10.0.26100.0, CMake 4.3.1-msvc1. NativeDesktop workload также установил свой текущий MSVC 14.51.36231.
- Подписи скачанных установщиков проверены: Python Software Foundation и Microsoft Corporation, Valid.
- Свободно около 32.2 GiB против 42.3 GiB до установки (изменение включает кэш и временные файлы, это не точный размер установленного ПО).
- Qt, библиотеки Telegram, configure и сборка локально не запускались. Перезагрузка не запускалась.
- Проверка облачной среды GitHub Actions успешно выполнена: https://github.com/devinek1d/liquid_telegramm/actions/runs/42517586623 . VS2026/MSVC14.44, SDK26100, CMake и Python обнаружены. Это не проверка полноценной сборки или Python 3.10.
- Следующий путь: зависимости и сборку готовить в облаке; локально проверять итоговый бинарник после изоляции профиля.
