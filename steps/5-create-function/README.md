# Создание Функции

Ранее мы использовали рантайм `python37`, однако с версии рантайма `python39` библиотеки необходиомо указывать явно. Для работы с `requirements.txt` есть удобная python библиотека `pipreqs`. Для генерации `requirements.txt` с помощью `pipreqs` достаточно указать рабочий каталог. В большинстве интерпритаторов linux для указания текущего каталога есть переменная `$PWD`. Если файл `requirements.txt` уже есть и его нужно актуализировать то используется флаг `--force`, например:  

    pip install pipreqs
    pipreqs $PWD --print
    pipreqs $PWD --force

Для создания функции зададим ряд переменных:
* `SECRET_ID` идентификатор секрета (можно получить из таблицы со списком секретов)
* `YMQ_QUEUE_URL` URL очереди (можно получить на странице обзора)
* `DOCAPI_ENDPOINT` (можно получить на странице обзора БД, нужен именно Document API)
* `S3_BUCKET` имя бакета, в нашем случае это `storage-for-ffmpeg` 

Создадим и проверим все переменные:
 
    echo $SERVICE_ACCOUNT_FFMPEG_ID
    echo $SECRET_ID
    echo $YMQ_QUEUE_URL
    echo $DOCAPI_ENDPOINT
    echo $S3_BUCKET

Скачаем статический релизный бинарник для Linux amd64 на сайте `ffmpeg.org`^ обычно он в разделе `FFmpeg Static Builds` и называется примерно так 	`ffmpeg-release-amd64-static.tar.xz`, распакуйте архив из него вам понадобиться только файл `ffmpeg`. Поскольку есть ограничение на размер  файла который можно приложить через консоль, код наших функций и `ffmpeg` зальем в Object Storage. 

Находясь в директории с исходными файлами, упакуем все нужные нам файлы в zip-архив. В Object Storage для простоты используем тот же бакет, куда далее будем складывать видео. Открываем Объекты, вверху справа выбираем Загрузить.

    zip src.zip index.py requirements.txt ffmpeg

Создадим нашу функцию `ffmpeg-api` и `ffmpeg-converter`, при этом сразу зададим все необходимые переменные и сервисный аккаунт:

    yc serverless function create \
    --name  ffmpeg-api \
    --description "function for ffmpeg-api"

    yc serverless function create \
    --name  ffmpeg-converter \
    --description "function for ffmpeg-converter"

    yc serverless function version create \
    --function-name ffmpeg-api \
    --memory=256m \
    --execution-timeout=5s \
    --runtime=python37 \
    --entrypoint=index.handle_api \
    --service-account-id $SERVICE_ACCOUNT_FFMPEG_ID \
    --environment SECRET_ID=$SECRET_ID \
    --environment YMQ_QUEUE_URL=$YMQ_QUEUE_URL \
    --environment DOCAPI_ENDPOINT=$DOCAPI_ENDPOINT \
    --package-bucket-name $S3_BUCKET \
    --package-object-name src.zip

    yc serverless function version create \
    --function-name ffmpeg-converter \
    --memory=2048m \
    --execution-timeout=600s \
    --runtime=python37 \
    --entrypoint=index.handle_process_event \
    --service-account-id $SERVICE_ACCOUNT_FFMPEG_ID \
    --environment SECRET_ID=$SECRET_ID \
    --environment YMQ_QUEUE_URL=$YMQ_QUEUE_URL \
    --environment DOCAPI_ENDPOINT=$DOCAPI_ENDPOINT \
    --environment S3_BUCKET=$S3_BUCKET \
    --package-bucket-name $S3_BUCKET \
    --package-object-name src.zip

## Тестирование функции

В веб-консоли находясь в вашем рабочем каталоге перейдите в раздел `Cloud Functions`, выберете ранее созданную функцию `ffmpeg-api`. Перейдите в раздел `Тестирование` в боковом меню, выберите шаблон `Без шаблона` и добавьте во вводные данные JSON: 
 
    {"action":"convert", "src_url":"https://disk.yandex.ru/i/38RbVC0spb_jQQ"}    

Нажмите кнопку `Запустить тест`, этим самым мы загрузим файл в хранилище и создадим задачу в БД. Если вы всё сделали правильно, то увидите результат:

    {
        "task_id": "133e05c2-1b98-41cc-9aab-b816d71af343"
    }

Воспользуется полученным id для получения статуса из базы данных, поменяйте во вводных данных JSON:

    {"action":"get_task_status", "task_id":"133e05c2-1b98-41cc-9aab-b816d71af343"}

Нажмите кнопку `Запустить тест`, так как мы еще не обрабатывали задачи в очереди то и результат очевиден:

    {
        "ready": false
    }
