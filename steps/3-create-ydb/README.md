# Yandex Database
## Создание базы

Создадим базу данных YDB с именем `ffmpeg` и типом serverless используя для этого флаг `--serverless`:

    yc ydb database create ffmpeg --serverless --folder-id $FOLDER_ID
    yc ydb database list

Сразу получим и сохраним document_api_endpoint в значение переменной `DOCAPI_ENDPOINT`

    yc ydb database get --name ffmpeg
    echo "export DOCAPI_ENDPOINT=<DOCAPI_ENDPOINT>" >> ~/.bashrc && . ~/.bashrc
    echo $DOCAPI_ENDPOINT

Как только база данных у нас была создана, воспользуемся ранее использованной утилитой AWS CLI для создания документной таблицы в нашей базе данных, при этом всю конфигурацию возьмем из файла `tasks.json`:

    aws dynamodb create-table --cli-input-json file://tasks.json \
    --endpoint-url $DOCAPI_ENDPOINT \
    --region ru-central1