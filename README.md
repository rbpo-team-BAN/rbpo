# ConfigRelease

Учебный проект команды «БАН» для курса «Разработка безопасного программного обеспечения». Основа — тема №24: публикация конфигураций приложений.

Редактор готовит YAML-конфигурацию, публикатор рассматривает и выпускает версию, сервисный клиент получает текущую конфигурацию своей пары «приложение + окружение».

- [Паспорт проекта: граница, роли, сценарии и правила](PROJECT.md).
- [Использование генеративного ИИ](AI_USAGE.md).

## Текущая реализация

Минимальная запускаемая основа: ASP.NET Core API с одним открытым endpoint `GET /health`. Он возвращает `200 OK` и JSON `{"status":"ok","service":"ConfigRelease"}`.

SQLite, токены, обработка YAML и предметные операции пока планируются. Их граница и правила описаны в паспорте. Для текущего запуска не нужны база данных, токены или внешние сервисы; сторонних NuGet-пакетов нет.

## Локальный запуск

Нужен стабильный .NET SDK 10, включая ASP.NET Core. `global.json` допускает стабильные SDK 10.0 начиная с 10.0.100. [Установка .NET SDK](https://dotnet.microsoft.com/download/dotnet/10.0).

Команды выполняются из корня репозитория. Сервер запускается только на `127.0.0.1:5080`; порт должен быть свободен. Остановка — `Ctrl+C`.

### Windows / PowerShell

Если SDK распакован в `.dotnet` в корне проекта, команды выберут его; иначе используется системный `dotnet`. Локальный SDK не хранится в Git и не является частью репозитория.

```powershell
$dotnetPath = if (Test-Path -LiteralPath '.\.dotnet\dotnet.exe') { '.\.dotnet\dotnet.exe' } else { 'dotnet' }
& $dotnetPath --version
& $dotnetPath restore .\src\ConfigRelease.Api\ConfigRelease.Api.csproj --configfile .\NuGet.Config
& $dotnetPath build .\src\ConfigRelease.Api\ConfigRelease.Api.csproj --no-restore
& $dotnetPath run --project .\src\ConfigRelease.Api\ConfigRelease.Api.csproj --no-build --no-launch-profile --urls http://127.0.0.1:5080
```

В другом окне PowerShell:

```powershell
$healthResponse = Invoke-WebRequest -Uri 'http://127.0.0.1:5080/health'
$healthResponse.StatusCode
$healthResponse.Content
```

Ожидаются код `200` и тело:

```json
{"status":"ok","service":"ConfigRelease"}
```

### Linux / macOS

При установленном .NET SDK 10:

```sh
dotnet --version
dotnet restore src/ConfigRelease.Api/ConfigRelease.Api.csproj --configfile NuGet.Config
dotnet build src/ConfigRelease.Api/ConfigRelease.Api.csproj --no-restore
dotnet run --project src/ConfigRelease.Api/ConfigRelease.Api.csproj --no-build --no-launch-profile --urls http://127.0.0.1:5080
```

В другом терминале:

```sh
curl -i http://127.0.0.1:5080/health
```

Ожидаются HTTP `200 OK`, `Content-Type: application/json` и то же JSON-тело. Проверка текущего блока выполняется на Windows; отдельный запуск на Linux/macOS пока не выполнялся.

## Структура

```text
PROJECT.md                          паспорт продукта
AI_USAGE.md                         сведения об использовании ИИ
global.json                         выбор .NET SDK
NuGet.Config                        источник будущих пакетов
src/ConfigRelease.Api/
  ConfigRelease.Api.csproj           проект ASP.NET Core
  Program.cs                        HTTP API
  Properties/launchSettings.json    профиль локального запуска
```

Сборочные `bin/`, `obj/`, локальный SDK, рабочие базы данных и локальные настройки исключены из Git через `.gitignore`.
