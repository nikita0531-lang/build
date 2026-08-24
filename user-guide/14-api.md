---
title: 14. Приложение: API и интеграции
description: Для интеграторов
published: true
date: 2026-08-24T14:26:42.502Z
tags: руководство, api, интеграции
editor: markdown
dateCreated: 2026-08-24T14:26:38.934Z
---

Раздел для интеграторов и технических специалистов. Обычному пользователю системы
он не нужен.

---

## 14.1. Общие сведения

- Протокол: **HTTP/JSON**, REST-подобный.
- Все прикладные методы начинаются с **`/api/`**.
- Интерактивная документация: **`/docs`** (ReDoc), спецификация OpenAPI —
  **`/swagger/v1/swagger.json`**.
- Проверка живости: **`GET /api/health`**, состояние БД — **`GET /api/dbHealth`**.

---

## 14.2. Аутентификация

| Метод | Назначение |
|---|---|
| `POST /api/auth/auth` | Авторизация: логин и пароль → access token |
| `POST /api/auth/refresh` | Обновление access token по refresh token из HttpOnly cookie |
| `POST /api/auth/logout` | Выход с отзывом refresh token |
| `GET /api/auth/sessions` | Сессии текущего пользователя |
| `GET /api/auth/activeSessions` | Все активные сессии (для системного администратора) |
| `POST /api/auth/revokeSession` | Отзыв сессии — сразу освобождает лицензионное место |

**Схема:** access token передаётся в заголовке `Authorization: Bearer <token>`.
Refresh token хранится в **HttpOnly cookie** и в JavaScript недоступен — это
защита от кражи токена через XSS.

**Смена пароля.** Если администратор выдал временный пароль, API отвечает
**428** на прикладные запросы, пока пароль не заменён:
`POST /api/user/changePassword`.

Политика пароля: `GET /api/security/passwordPolicy`.

---

## 14.3. Рабочее пространство

Все прикладные запросы выполняются в контексте пространства.
Идентификатор передаётся заголовком:

```
X-Workspace-Id: <id>
```

Заголовок обязателен и для веб-сокетов (чат и примечания). Без него запись
попадёт не в тот контур или не будет видна. Доступные пространства:
`GET /api/workspace/my`.

---

## 14.4. Соглашения

**Списки** — `POST /api/<entity>/getAll` с телом, содержащим постраничность,
сортировку, фильтры и поиск. Часть простых списков доступна через `GET`
(например, `GET /api/project/getAll`).

**Карточка** — `GET /api/<entity>/{id}`.

**Создание** — `POST /api/<entity>`, **изменение** — `PUT /api/<entity>/{id}`,
**удаление** — `DELETE /api/<entity>/{id}`.

**Смена статуса** — `POST /api/<entity>/changeStatus/{id}`.
Допустимые переходы: `GET /api/workflow/workflowMap`.

**Экспорт** — `POST /api/<entity>/export` — весь набор по переданным фильтрам.

**Импорт** — `GET /api/<entity>/importTemplate` (шаблон) и
`POST /api/<entity>/importExcel` (загрузка).

---

## 14.5. Карта эндпоинтов

Всего в API **277 операций**. Основные группы:

| Группа | Операций | Ключевые методы |
|---|---|---|
| **Issue** | 17 | `getAll`, `getAllAsTasks`, `changeStatus`, `changePriority`, `setResponsibleUser`, `bulk/*`, `export`, `importExcel`, `getAssignableUsers` |
| **Report** | 22 | Шаблоны и генерация документов по предписаниям и инспекциям |
| **Administration-User** | 14 | CRUD, `activate`, `block`, `changePassword`, `delegate`, `getProfile`, `culture` |
| **Work** | 13 | CRUD, `changeStatus`, `progress`, `timesheet`, `setMain`, `exportKs2` |
| **ProjectObject** | 11 | CRUD, `getPlanInfo`, `uploadPlanByAsset`, `changeStatus`, экспорт дерева |
| **Notification** | 10 | Список, `unreadCount`, `readAll`, `preferences`, mobile-push токены и устройства |
| **Provider** | 10 | Конфигурация каналов, `test`, `sendTestMessage`, `link` |
| **Inspection** | 9 | CRUD, `changeStatus`, `checklistItems/{itemId}/answer`, `inspectorSchedule`, `export` |
| **Project** | 9 | CRUD, `changeStatus`, `hierarchy/meta`, импорт |
| **ChecklistTemplate** | 9 | CRUD, `copy`, экспорт и импорт Excel |
| **Asset** | 9 | Загрузка файлов, `download`, `preview`, `thumbnail`, `getAssetsToEntity` |
| **Contract / Contragent / IssueType / IssueCategory / InspectionCategory / WorkType / UnitOfMeasure** | по 8 | Типовой CRUD + экспорт/импорт |
| **Administration-Role** | 8 | CRUD, пользователи роли |
| **Workspace** | 7 | CRUD, `my` |
| **ExternalField / UserProject / RoleCategory** | по 6 | Пользовательские поля и назначения |
| **Prescription** | 5 | `generate`, `getAll`, `changeStatus`, карточка, удаление |
| **WorkExecution** | 5 | CRUD, `changeStatus`, `getAll` |
| **Analytics** | 4 | `getStatistic`, `getStatisticByProject`, `getStatisticByProjectObject`, `getShakhmatSummary` |
| **UserWorkspace** | 4 | Назначение пользователей на пространства |
| **Communication-Message / Note** | по 4 | Чат и примечания |
| **Administration-Permission** | 3 | Список прав, `self`, назначение |
| **ProjectObjectZone / ProjectObjectMarker** | по 3 | Зоны и метки на планах |
| **Administration-SystemSettings** | 3 | Чтение и изменение настроек, пароль System |
| **License / Issues.WebApi** | 3 | `features`, `license-status`, `license/upload` |
| **Audit** | 1 | `getShort` — история изменений |
| **Common-Workflow** | 1 | `workflowMap` — карта переходов по статусам |

Полный список с описаниями — в `/docs`.

---

## 14.6. Файлы и вложения

Файлы — отдельная сущность **Asset**:

1. `POST /api/asset` — загрузка.
2. `GET /api/asset/getAssetsToEntity` — файлы конкретной записи.
3. `GET /api/asset/{id}/download` — оригинал,
   `/preview` и `/thumbnail` — уменьшенные версии.

Планы объектов привязываются через `POST /api/projectObject/{id}/uploadPlanByAsset`.

---

## 14.7. Реальное время

Чат и примечания работают через веб-сокеты (SignalR). При подключении необходимо
передать `X-Workspace-Id` — иначе сообщения уйдут не в тот контур.

---

## 14.8. Ошибки

| Код | Значение | Как показывается в интерфейсе |
|---|---|---|
| **400** | Запрос отклонён сервером | «Запрос отклонён сервером» |
| **401** | Не аутентифицирован или токен истёк | «Сессия истекла. Войдите снова» |
| **403** | Нет прав на ресурс | «У вас нет доступа к этому ресурсу» |
| **404** | Данные не найдены | «Данные не найдены» |
| **428** | Требуется смена выданного пароля | Редирект на экран смены пароля |
| **429** | Слишком много запросов | «Слишком много запросов. Повторите позже» |
| **500** | Ошибка на сервере | «Ошибка на сервере. Попробуйте позже» |

Бизнес-ошибки возвращаются с человекочитаемым текстом, например
«Переход в статус "Закрыто" из "Новый" не допускается» или
«Значение поля "Наименование" не уникально».

---

## 14.9. Обмен через Excel

Для массовой загрузки без программирования используйте импорт: формат выгрузки
реестра совпадает с форматом импорта, поэтому цикл «выгрузить → поправить →
загрузить» рабочий.

**Осторожно:** для **работ и договоров** импорт всегда создаёт новые записи —
повторная загрузка того же файла приведёт к дублям. Справочники, как правило,
обновляют знакомые строки и создают незнакомые.

Дальше: [15. Справочник статусов и прав](/user-guide/15-reference)
