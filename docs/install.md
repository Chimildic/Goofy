# Установка

Вы создаете собственную копию библиотеки. Только вы имеете доступ ко всему, что происходит в этой копии. 

## Настройка goofy

Выполняется один раз. 
> [Видео инструкция](https://drive.google.com/file/d/1yhI8sfBVAyhn5RUUtOxKLiYSgdIF9Ts9/view) с новым интерфейсом платформы Apps Script. 
> 
> [Видео инструкция](https://drive.google.com/file/d/13I_E9g5x_Gb-G-KANmzUxLgDv-bPkQsu/view) для старого интерфейса.

1. Перейдите в [Spotify Dashboard](https://developer.spotify.com/dashboard/). Нажмите `Log in` и авторизуйтесь в Spotify. Примите условия использования.

   ![Условия использования](/img/install-step-dashboard-term.png ':size=40%')

2. Нажмите кнопку `Create an app`. Введите произвольное имя и описание. Поставьте галочки. Нажмите кнопку `Create`.

   ![Создание приложения](/img/install-step-create-app.png ':size=40%')

3. Перейдите к [библиотеке в Apps Script](https://script.google.com/d/1DnC4H7yjqPV2unMZ_nmB-1bDSJT9wQUJ7Wq-ijF4Nc7Fl3qnbT0FkPSr/edit?usp=sharing). Войдите в Google аккаунт, если потребуется.

4. Выберите слева в раскрывающемся меню `Общие сведения`. 

   ![Открыть меню](/img/general-property.gif ':size=60%')

   На открывшейся странице, справа `Создать копию`. Откроется копия, созданная на вашем аккаунте. Переименуйте, если нужно (нажать на имя вверху страницы).
   
    ![Создать копию](/img/install-step-copy.png)

5. Перейдите в файл `config.gs`. Вставьте `CLIENT_ID` и `CLIENT_SECRET` вместо слов `вашеЗначение`. Значения брать в созданном приложении Spotify на шаге 2.

   Сохраните изменение <kbd>Ctrl</kbd><kbd>S</kbd> или иконка дискеты на панели действий

   ![Client ID и Client Secret](/img/install-step-client-id2.png)

6. Запустите в редакторе выполнение функции `setProperties`. 

   ![run setProperties](/img/install-run-setProperties.png)

   Увидите всплывающее сообщение с необходимость предоставить права доступа. Согласитесь на выдачу.

   ![запрос прав](/img/install-permission-request.png ':size=50%')

   Выберите Google аккаунт, на котором создали копию библиотеки.

   ![Выбор аккаунта](/img/install-step-account.png)

   Нажмите `Дополнительные настройки`, затем `Перейти на страницу "Копия Goofy (Ver. 1)"`

   ![Выбор аккаунта](/img/install-step-warning.png ':size=50%')

   Нажмите кнопку `Разрешить` внизу окна.

   ![Выбор аккаунта](/img/install-step-grant-permissions.png)

7. Окно закроется. Выберите `Начать развертывание` - `Пробные развертывания`

   ![Развернуть веб-приложение](/img/install-step-webapp.png ':size=40%')

   В появившимся окне скопируйте ссылку. Должно заканчиваться на `dev`.

   ![Скопировать ссылку](/img/install-step-link.png)

8. В **новой вкладке** перейдите по скопированной ссылке. На открывшийся странице скопируйте ссылку внизу страницы. Оканчивается на `usercallback`.

   ![Callback-ссылка](/img/install-step-callback-link.png)

9. Вернитесь в [Spotify Dashboad](https://developer.spotify.com/dashboard/). Нажмите кнопку `EDIT SETTINGS` у вашего приложения.
    
    Вставьте в поле `Redirect URIs` скопированную ссылку из шага 8. Нажмите кнопку `ADD`. Затем внизу кнопку `Save`.
    
    ![Добавить callback](/img/install-step-dashboard-redirect.png ':size=50%')

10. Перейдите на вкладку из шага 8 и обновите эту страницу (`F5`).

    Нажмите кнопку `Authorize`.

    ![Callback-ссылка](/img/install-step-callback-link.png)

    Разрешите доступ к аккаунту Spotify.

    ![Разрешения Spotify](/img/install-step-grant-spotify.png)

    Должны появиться слова о успешном выполнении.

    Первая установка и настройка завершены.

## Настройка last.fm

Если не используется, выполнять не нужно. 

1. Подключите Spotify к Last.fm [здесь](https://www.last.fm/settings/applications)
2. Создайте точку входа [здесь](https://www.last.fm/api/account/create). Заполните название и описание произвольно. Остальное пропустить, оставить пустым.
3. Полученный `API key` присвоить параметру `LASTFM_API_KEY`. Также укажите свой логин в `LASTFM_LOGIN`. И задайте `true` для `ON_LASTFM_RECENT_TRACKS`.
4. Запустить в редакторе выполнение функции `setProperties`.

![Lastfm account api](/img/lastfm_account_api3.png)

# Обновление

Далее описываются способы обновления библиотеки. При первой установке это не нужно.

## Обновить библиотеку

Замените все содержимое файла `library.gs` на новое <kbd>Ctrl</kbd><kbd>A</kbd> и <kbd>Ctrl</kbd><kbd>V</kbd>, которое берется [здесь](https://github.com/Chimildic/goofy/blob/main/library.js) или [здесь](https://script.google.com/d/1DnC4H7yjqPV2unMZ_nmB-1bDSJT9wQUJ7Wq-ijF4Nc7Fl3qnbT0FkPSr/edit?usp=sharing) <kbd>Ctrl</kbd><kbd>A</kbd> и <kbd>Ctrl</kbd><kbd>C</kbd> и сохраните файл.

## Обновить параметры

1. Измените требуемый параметр в файле `config.gs` (актуальный список находится [здесь](https://github.com/Chimildic/goofy/blob/main/config.js))
2. Запустите в редакторе функцию `setProperties`

## Обновить права доступа

При расширении функций библиотеки, могут потребоваться дополнительные права доступа. Например, на загрузку изображения или доступ к любимым трекам. При такой необходимости, в [списке изменений](/changelog.md) очередного обновления будут указания на обновление прав доступа.

1. Вставьте следующую функцию и запустите в редакторе один раз. После можно ее удалить.
    ```js
    function resetAuth(){
        Auth.reset();
    }
    ```
2. Нажмите в верхнем меню редактора `начать развертывание` - `пробные развертывания`
3. Скопируйте ссылку из открывшегося окна и перейдите по ней в новой вкладке
4. Нажмите `Authorize` и подтвердите новые права доступа
