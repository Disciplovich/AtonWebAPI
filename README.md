
Веб-сервис для управления пользователями с ролевой системой доступа (Администратор/Пользователь).

## Требования

* **.NET 9 или выше**
* Visual Studio 2022 или выше / .NET CLI
* Postman или другой HTTP-клиент для тестирования (опционально)
* Для работы со Swagger UI достаточно браузера

## Установка и запуск

1. Клонирование репозитория.
   Откройте Git Bash и введите команду для клонирования репозитория:
   git clone https://github.com/username/aton-webapi.git
   Перейдите в папку с проектом:
   cd aton-webapi

2. Убедитесь, что у вас установлен .NET 9 SDK или выше.
   Для проверки выполните:
   dotnet --version
   Если .NET SDK не установлен, скачайте его с официального сайта: https://dotnet.microsoft.com/download

3. Компиляция и запуск программы.
   Откройте терминал и введите команду:
   dotnet restore
   dotnet run

4. Сервис запустится на:
   - HTTP: http://localhost:5074
   - HTTPS: https://localhost:7129

5. Откройте браузер и перейдите по адресу:
   https://localhost:7129/swagger
   для доступа к Swagger UI с документацией и тестированием API.
<img width="1055" height="824" alt="image" src="https://github.com/user-attachments/assets/468a7f97-49f7-4ef4-882b-e367b0d9ba95" />


## Начальная настройка

При первом запуске автоматически создается пользователь-администратор:

* **Логин:** admin
* **Пароль:** admin123

Используйте эти данные для базовой аутентификации в Swagger или при прямых HTTP-запросах.

## Аутентификация

Сервис использует **Basic Authentication**. При каждом запросе передавайте заголовок:

Authorization: Basic YWRtaW46YWRtaW4xMjM=

где YWRtaW46YWRtaW4xMjM= — это base64-кодированная строка admin:admin123

В Swagger UI: Нажмите кнопку "Authorize" вверху, введите:
- Username: admin
- Password: admin123

## Методы API

### 1. Создание пользователя (Create)

POST /api/Users/register

Создание пользователя с указанием логина, пароля, имени, пола, даты рождения и статуса администратора.

*Доступно:* Администраторам

**Параметры (form-data):**
- Login (обязательный) - только латинские буквы и цифры
- Password (обязательный) - только латинские буквы и цифры
- Name (обязательный) - только латинские и русские буквы
- Gender (обязательный) - 0 (женщина), 1 (мужчина), 2 (неизвестно)
- Birthday (опциональный) - дата в прошлом
- Admin (опциональный) - true/false, по умолчанию false

**Пример запроса в Swagger:**
Login: john_doe
Password: password123
Name: John
Gender: 1
Birthday: 1990-01-01
Admin: false

### 2. Изменение профиля (Update-1)

PUT /api/Users/change/profile-data/{selectedUserLogin}

Изменение имени, пола или даты рождения пользователя.

*Доступно:* Администраторам ИЛИ самому пользователю (если он активен)

**Параметры запроса:**
- name (query) - новое имя
- gender (query) - новый пол (0, 1, 2)
- birthday (query) - новая дата рождения

**Пример:**
PUT /api/Users/change/profile-data/john_doe?name=Jonathan&gender=1

### 3. Изменение пароля (Update-1)

PUT /api/Users/change/password/{selectedUserLogin}

Изменение пароля пользователя.

*Доступно:* Администраторам ИЛИ самому пользователю (если он активен)

**Параметры:**
- password (query, обязательный) - новый пароль

**Пример:**
PUT /api/Users/change/password/john_doe?password=newpassword456

### 4. Изменение логина (Update-1)

PUT /api/Users/change/login/{selectedUserLogin}

Изменение логина пользователя. Логин должен оставаться уникальным.

*Доступно:* Администраторам ИЛИ самому пользователю (если он активен)

**Параметры:**
- newUserLogin (query, обязательный) - новый логин

**Пример:**
PUT /api/Users/change/login/john_doe?newUserLogin=johndoe_new

### 5. Получение активных пользователей (Read)

GET /api/Users/request/all-active-users

Получение списка всех активных пользователей, отсортированных по дате создания.

*Доступно:* Только администраторам

### 6. Поиск пользователя по логину (Read)

GET /api/Users/request/by-login?login={login}

Получение информации о пользователе: имя, пол, дата рождения, статус активности.

*Доступно:* Только администраторам

**Пример:**
GET /api/Users/request/by-login?login=admin

### 7. Получение личного профиля (Read)

GET /api/Users/request/personal-profile?login={login}&password={password}

Получение полной информации о собственном профиле.

*Доступно:* Только самому пользователю (если он активен)

**Пример:**
GET /api/Users/request/personal-profile?login=john_doe&password=password123

### 8. Поиск пользователей старше возраста (Read)

GET /api/Users/request/users-over-specified-age?age={age}

Получение пользователей старше указанного возраста.

*Доступно:* Только администраторам

**Пример:**
GET /api/Users/request/users-over-specified-age?age=30

### 9. Удаление пользователя (Delete)

DELETE /api/Users/delete?login={login}&isHardDelete={true/false}

Удаление пользователя:
- isHardDelete=true - полное удаление из БД
- isHardDelete=false - мягкое удаление (установка RevokedOn/RevokedBy)

*Доступно:* Только администраторам

**Пример мягкого удаления:**
DELETE /api/Users/delete?login=john_doe&isHardDelete=false

### 10. Восстановление пользователя (Update-2)

PUT /api/Users/restore/user/{login}

Восстановление мягко удаленного пользователя (очистка RevokedOn/RevokedBy).

*Доступно:* Только администраторам

**Пример:**
PUT /api/Users/restore/user/john_doe

### 11. Специальные методы (только в режиме DEBUG)

GET /api/Users/debug/get-all-users - получение всех пользователей (активных и неактивных)

OPTIONS /api/Users/options - информация о поддерживаемых методах

HEAD /api/Users/head - проверка доступности ресурса

## Правила валидации

1. **Логин и пароль:** только латинские буквы и цифры
2. **Имя:** только латинские и русские буквы
3. **Пол:** 0 (женщина), 1 (мужчина), 2 (неизвестно)
4. **Дата рождения:** не может быть в будущем
5. **Логин:** должен быть уникальным

## Тестирование

1. Через Swagger UI: Самый простой способ - все методы доступны с авторизацией
2. Через Postman:
   - Установите заголовок Authorization: Basic YWRtaW46YWRtaW4xMjM=
   - Тестируйте все методы по указанным выше URL
