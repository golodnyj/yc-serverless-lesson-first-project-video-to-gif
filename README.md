# Использование Yandex Message Queue для конвертирования видео 

Цель урока разработать проект, который позволит пользователям конвертировать видео в GIF.  Эта задача хорошо подходит для Сloud Functions, потому что конвертирование отнимает немало ресурсов процессора, и чем более качественное видео, тем больше ресурсов требуется на его обработку.

Будет использованы следующие утилиты и сервисы: 
* Cloud Function
* Yandex Message Queue
* Yandex Database
* Yandex Object Storage`
* yc — Интерфейс командной строки, CLI [документаця](https://cloud.yandex.ru/docs/cli/quickstart#install).
* aws — Интерфейс командной строки, CLI.

В папке `steps` содержится последовательность шагов необходимых для настройки решения с соответствующими пояснениями.

# Почему в этом примере мы используем очереди
Представим, что мы попытались решить эту задачу «в лоб». Пользователь заходит на страницу и вводит ссылку на видео. Сервис скачивает предложенное видео, конвертирует его и отдает ссылку на GIF. Сразу возникают две серьёзные проблемы:
* Синхронное соединение не всегда стабильно. Чем больше вы его держите, тем больше вероятность, что оно разорвётся. В этом случае всё придётся сделать заново. А если соединение нестабильное, то пользователь может и не дождаться результата. 
* Задача ресурсоёмкая, если придёт одновременно 10 пользователей с большими видеороликами, мощностей может не хватить.

Чтобы избежать этих проблем, в архитектуру этого сервиса необходимо встроить очередь.
