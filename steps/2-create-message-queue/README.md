# Yandex Message Queue
## Создание очереди
Для создания очереди Yandex Message Queue вы можете использовать три разных способа:
* UI;
* консольную утилиту aws;
* terraform.

## Создание очереди с помощью UI

1. Откройте раздел Message Queue. 
2. Нажмите кнопку `Создать очередь`. 
3. Укажите имя очереди `ffmpeg`. 
4. Выберите тип очереди `Стандартная`. 
5. Нажмите кнопку `Создать`.

## Создание очереди с помощью утилиты aws

Для альтернативного способа создания очереди можете воспользоваться  AWS CLI. Для начала, задайте конфигурацию с помощью команды `aws configure`. При этом от вас потребуется ввести:
* `AWS Access Key ID` — это идентификатор ключа доступа `key_id` сервисного аккаунта, полученный на предыдущем шаге.
* `AWS Secret Access Key` — это секретный ключ `secret` сервисного аккаунта, полученный на предыдущем шаге.
* `Default region name` — используйте значение `ru-central1`.

По завершению конфигурации вы сможете создать очередь: 

    aws configure
    aws sqs create-queue --queue-name ffmpeg --endpoint https://message-queue.api.cloud.yandex.net/

В результате успешного выполнения предыдущей команды в ответ вы получите URL: 
    
    {
        "QueueUrl": "https://message-queue.api.cloud.yandex.net/b1ga4gj7agij03ln6aov/dj6000000003kv2t02b3/ffmpeg"
    }

Запишем значения URL в переменную `YMQ_QUEUE_URL` она нам потребуется при создании функции:

    echo "export YMQ_QUEUE_URL=<YMQ_QUEUE_URL>" >> ~/.bashrc && . ~/.bashrc
    echo $YMQ_QUEUE_URL

Еще вам потребует значение атрибута `QueueArn` получим его:

    aws sqs get-queue-attributes \
    --endpoint https://message-queue.api.cloud.yandex.net \
    --queue-url $YMQ_QUEUE_URL \
    --attribute-names QueueArn

В результате вы получите ответ вида:

    {
        "Attributes": {
            "QueueArn": "yrn:yc:ymq:ru-central1:b1gl21bkgss4msekt08i:ffmpeg"
        }
    }

Сохраним значение `yrn:yc:ymq:ru-central1:b1gl21bkgss4msekt08i:ffmpeg` в переменную `YMQ_QUEUE_ARN`:

    echo "export YMQ_QUEUE_ARN=<YMQ_QUEUE_ARN>" >> ~/.bashrc && . ~/.bashrc
    echo $YMQ_QUEUE_ARN

