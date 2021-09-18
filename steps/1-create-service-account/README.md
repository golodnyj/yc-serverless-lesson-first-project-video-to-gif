# Service Account
## Создание аккаунта

Создайте сервисный аккаунт с именем `ffmpeg-account-for-cf`: 

    export SERVICE_ACCOUNT=$(yc iam service-account create --name ffmpeg-account-for-cf \
    --description "service account for serverless" \
    --format json | jq -r .)

Проверьте текущий список сервисных аккаунтов:
     
    yc iam service-account list

После проверки запишите ID, созданного сервисного аккаунта, в переменную `SERVICE_ACCOUNT_ID`:

    echo "export SERVICE_ACCOUNT_FFMPEG_ID=<ID>" >> ~/.bashrc && . ~/.bashrc  
    echo $SERVICE_ACCOUNT_FFMPEG_ID

## Назначение роли сервисному аккаунту

Добавим вновь созданному сервисному аккаунту роли `storage.viewer`, `storage.uploader`, `ymq.reader`, `ymq.writer`, `ydb.admin`, `serverless.functions.invoker`, и `lockbox.payloadViewer`:

    echo "export FOLDER_ID=$(yc config get folder-id)" >> ~/.bashrc && . ~/.bashrc 
    echo $FOLDER_ID

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role storage.viewer 

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role storage.uploader

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role ymq.reader

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role ymq.writer 

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role ydb.admin 

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role serverless.functions.invoker

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role lockbox.payloadViewer

    yc resource-manager folder add-access-binding $FOLDER_ID \
    --subject serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --role editor

Вы можете назначить несколько ролей с помощью команды `set-access-binding`. Но команда `set-access-binding` полностью перезаписывает права доступа к ресурсу! Все текущие роли на ресурс будут удалены. Убедитесь, что на ресурс не назначено ролей, которые вы не хотите потерять: 

    yc resource-manager folder list-access-bindings $FOLDER_ID

    yc resource-manager folder set-access-bindings $FOLDER_ID \
    --access-binding role=storage.viewer,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=storage.uploader,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=ymq.reader,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=ymq.writer,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=ydb.admin,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=serverless.functions.invoker,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=lockbox.payloadViewer,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID \
    --access-binding role=editor,subject=serviceAccount:$SERVICE_ACCOUNT_FFMPEG_ID

## Создание ключа доступа для сервисного аккаунта

Этот этап нужен для получения идентификатора ключа доступа и секретного ключа, которые будут использованы для загрузки файлов в Object Storage, работы с Yandex Message Queue и т.д. Для создания ключа доступа необходимо вызвать следующую команду:

    yc iam access-key create --service-account-name ffmpeg-account-for-cf

В результате вы получите примерно следующее

    access_key:
        id: ajefraollq5puj2tir1o
        service_account_id: ajetdv28pl0a1a8r41f0
        created_at: "2021-08-23T21:13:05.677319393Z"
        key_id: BTPNvWthv0ZX2xVmlPIU
    secret: cWLQ0HrTM0k_qAac43cwMNJA8VV_rfTg_kd4xVPi

Тут `key_id` — это идентификатор ключа доступа `ACCESS_KEY_ID`. А `secret` — это секретный ключ `SECRET_ACCESS_KEY`. Переменные `ACCESS_KEY_ID` и `SECRET_ACCESS_KEY` могут быть использованы для задания соответствующих значений `aws_access_key_id` и `aws_secret_access_key` при использовании библиотеки boto3.

## Создание элемента в сервисе Lockbox

В сервисе Lockbox cоздайте ваш первый секрет. Секрет состоит из набора версий, в которых хранятся ваши данные. Версия содержит наборы ключей и значений:
* Ключ — это несекретное название для значения, по которому вы будете его идентифицировать.
* Значение — это секретные данные.

Версия не изменяется. Для любого изменения количества пар ключей-значений или их содержимого необходимо создать новую версию. Создадим секрет с именем `ffmpeg-sa-key` и парой ключей `ACCESS_KEY_ID` и `SECRET_ACCESS_KEY`: 

    yc lockbox secret create --name ffmpeg-sa-key \
    --folder-id $FOLDER_ID \
    --description "keys for serverless" \
    --payload '[{"key": "ACCESS_KEY_ID", "text_value": <ACCESS_KEY_ID>}, {"key": "SECRET_ACCESS_KEY", "text_value": "SECRET_ACCESS_KEY"}]'

Получим и запишем значения `SECRET_ID` оно нам потребуется при создании функции:
    
    yc lockbox secret list
    yc lockbox secret get --name ffmpeg-sa-key

    echo "export SECRET_ID=<SECRET_ID>" >> ~/.bashrc && . ~/.bashrc
    echo SECRET_ID
