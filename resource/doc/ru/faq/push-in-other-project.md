# Использование Workerman для отправки данных клиентам в других проектах

**Вопрос:**

У меня есть обычный веб-проект, и я хочу вызывать интерфейсы Workerman для отправки данных клиентам в этом проекте.


**Ответ:**

**На основе Workerman вы можете обратить внимание на следующие ссылки:**

  - [Пример отправки через компонент Channel](../components/channel-examples.md) (поддерживает многопроцессорные/кластерные серверы, требуется загрузка компонента Channel)

  - [Отправка на основе Worker](https://www.workerman.net/q/508) (однопроцессорный режим, самый простой)

**Ссылки на основе webman**

  - [Плагин для отправки вебмана](https://www.workerman.net/plugin/2)


**Ссылки на основе GatewayWorker**

  - [Отправка через GatewayWorker в других проектах](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (поддерживает многопроцессорные/кластерные серверы, поддерживает групповые, мультигрупповые отправки, отправку одному получателю)


**Ссылки на основе PHPSocket.IO**

  - [Веб-отправка сообщений](https://www.workerman.net/web-sender) (по умолчанию однопроцессорный режим, основанный на socket.io, лучшая совместимость с браузером)
