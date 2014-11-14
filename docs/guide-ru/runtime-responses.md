Ответы
=========

Когда приложение заканчивает обработку [запроса](runtime-requests.md), оно генерирует объект [[yii\web\Response|ответа]]
и отправляет его конечному пользователю. Объект ответа содержит такие данные, как HTTP-код состояния, HTTP-заголовки и тело ответа.
По существу, конечная цель разработки Web-приложения — создание таких объектов ответа на различные запросы.

В большинстве случаев вам придется иметь дело в основном с [компонентом приложения](structure-application-components.md) `response`,
который по умолчанию является экземпляром класса [[yii\web\Response]]. Однако Yii также позволяет вам создавать собственные объекты ответа
и отправлять их конечным пользователям, как будет рассмотрено ниже.

В данном разделе мы опишем, как составлять ответы и отправлять их конечным пользователям. 


## Код состояния <a name="status-code"></a>

Одна из первых вещей, которые вы делаете при построении ответа, — это определение того, был ли успешно обработан запрос.
Это делается путём установки свойства [[yii\web\Response::statusCode]], которое может принимать значение одного из валидных 
[HTTP-кодов состояния](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html). Например, чтобы показать, что запрос был
успешно обработан, вы можете установить значение кода состояния равным 200, вот так:

```php
Yii::$app->response->statusCode = 200;
```

Однако в большинстве случаев вам не нужно явно устанавливать значение кода состояния. Дело в том, что значение свойства [[yii\web\Response::statusCode]] по умолчанию равно 200. А если вам нужно показать, что запрос не имел успеха, вы можете
выбросить соответствующее HTTP-исключение, как показано здесь:

```php
throw new \yii\web\NotFoundHttpException;
```

Когда [обработчик ошибок](runtime-handling-errors.md) поймает исключение, он извлечёт код состояния 
из исключения и назначит его ответу. Исключение [[yii\web\NotFoundHttpException]] в коде выше 
связано с HTTP-кодом состояния 404. В Yii предопределены следующие HTTP-исключения:

* [[yii\web\BadRequestHttpException]]: код состояния 400.
* [[yii\web\ConflictHttpException]]: код состояния 409.
* [[yii\web\ForbiddenHttpException]]: код состояния 403.
* [[yii\web\GoneHttpException]]: код состояния 410.
* [[yii\web\MethodNotAllowedHttpException]]: код состояния 405.
* [[yii\web\NotAcceptableHttpException]]: код состояния 406. 
* [[yii\web\NotFoundHttpException]]: код состояния 404.
* [[yii\web\ServerErrorHttpException]]: код состояния 500.
* [[yii\web\TooManyRequestsHttpException]]: код состояния 429.
* [[yii\web\UnauthorizedHttpException]]: код состояния 401.
* [[yii\web\UnsupportedMediaTypeHttpException]]: код состояния 415.

Если в вышеприведённом списке нет исключения, которое вы хотите выбросить, вы можете создать его, расширив класс
[[yii\web\HttpException]], или выбросить его напрямую с кодом состояния, например:
 
```php
throw new \yii\web\HttpException(402);
```


## HTTP-заголовки <a name="http-headers"></a> 

Вы можете отправлять HTTP-заголовки, управляя [[yii\web\Response::headers|набором заголовков]] компонента `response`.
Например:

```php
$headers = Yii::$app->response->headers;

// добавить заголовок Pragma. Уже имеющиеся Pragma-заголовки НЕ будут перезаписаны.
$headers->add('Pragma', 'no-cache');

// установить заголовок Pragma. Любые уже имеющиеся Pragma-заголовки будут сброшены.
$headers->set('Pragma', 'no-cache');

// удалить заголовок (или заголовки) Pragma и вернуть значения удалённых Pragma-заголовков в массиве
$values = $headers->remove('Pragma');
```

> Информация: названия заголовков регистронезависимы. Заново зарегистрированные заголовки не отсылаются пользователю до тех пор, пока
  не будет вызван метод [[yii\web\Response::send()]].


## Тело ответа <a name="response-body"></a>

Большинство ответов должны иметь тело, which gives the content that you want to show to end users.

Если у вас уже имеется отформатированная строка тела, вы можете назначить её свойству [[yii\web\Response::content]] 
объекта запроса. Например:

```php
Yii::$app->response->content = 'hello world!';
```

Если ваши данные нужно отформатировать перед отправкой конечным пользователям, вам следует установить значения двух свойств:
[[yii\web\Response::format|формат]] и [[yii\web\Response::data|данные]]. Свойство [[yii\web\Response::format|формат]]
определяет, в каком формате следует возвращать [[yii\web\Response::data|данные]]. Например:

```php
$response = Yii::$app->response;
$response->format = \yii\web\Response::FORMAT_JSON;
$response->data = ['message' => 'hello world'];
```

Yii из коробки имеет поддержку следующих форматов, каждый из которых реализован классом [[yii\web\ResponseFormatterInterface|форматтера]].
Вы можете настроить эти форматтеры или добавить новые путём настройки свойства [[yii\web\Response::formatters]].

* [[yii\web\Response::FORMAT_HTML|HTML]]: реализуется классом [[yii\web\HtmlResponseFormatter]].
* [[yii\web\Response::FORMAT_XML|XML]]: реализуется классом [[yii\web\XmlResponseFormatter]].
* [[yii\web\Response::FORMAT_JSON|JSON]]: реализуется классом [[yii\web\JsonResponseFormatter]].
* [[yii\web\Response::FORMAT_JSONP|JSONP]]: реализуется классом [[yii\web\JsonResponseFormatter]].

Хотя тело запроса может быть явно установлено показанным выше способом, в большинстве случаев вы можете устанавливать его неявно через возвращаемое значение методов 
[действий](structure-controllers.md). Типичный пример использования:
 
```php
public function actionIndex()
{
    return $this->render('index');
}
```

Действие `index` в коде выше возвращает результат рендеринга представления `index`. Возвращаемое значение будет взято 
компонентом `response`, отформатировано и затем отправлено конечным пользователям.

Так как по умолчанию форматом ответа является [[yii\web\Response::FORMAT_HTML|HTML]], вам нужно лишь вернуть строку
в методе действия. Если вы хотите использовать другой формат ответа, вам нужно сначала установить его перед отправкой данных.
Например:

```php
public function actionInfo()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    return [
        'message' => 'hello world',
        'code' => 100,
    ];
}
```

Как было сказано выше, кроме использования компонента приложения `response`  по умолчанию вы также можете создавать свои
объекты ответа и отправлять их конечным пользователям. Вы можете сделать это, возвращая такой объект в методе действия, например:

```php
public function actionInfo()
{
    return \Yii::createObject([
        'class' => 'yii\web\Response',
        'format' => \yii\web\Response::FORMAT_JSON,
        'data' => [
            'message' => 'hello world',
            'code' => 100,
        ],
    ]);
}
```

> Примечание: создавая собственные объекты ответов, вы не сможете воспользоваться конфигурацией компонента `response`, настроенной вами в конфигурации приложения. Тем не менее, вы можете воспользоваться 
  [внедрением зависимости](concept-di-container.md), чтобы применить общую конфигурацию к вашим новым объектам ответа.


## Перенаправление браузера <a name="browser-redirection"></a>

Перенаправление браузера основано на отправке HTTP-заголовка `Location`. Так как данная возможность широко используется, Yii предоставляет
некоторые специальные средства её поддержки.

Вы можете перенаправить браузер пользователя на URL-адрес, вызвав метод [[yii\web\Response::redirect()]]. Этот метод
устанавливает в соответствующий заголовок `Location` указанный URL-адрес и возвращает сам объект ответа. В методе действия
вы можете вызвать короткую версию, [[yii\web\Controller::redirect()]]. Например:

```php
public function actionOld()
{
    return $this->redirect('http://example.com/new', 301);
}
```

В коде выше метод действия возвращает результат вызова метода `redirect()`. Как описано ранее, объект ответа,
возвращаемый методом действия, будет использоваться как ответ, отправляемый конечным пользователям.

В местах, отличных от методов действий, следует напрямую вызывать [[yii\web\Response::redirect()]] и сразу после него — 
метод [[yii\web\Response::send()]] для уверенности в том, что никакое дополнительное содержимое не будет добавлено к ответу.

```php
\Yii::$app->response->redirect('http://example.com/new', 301)->send();
```

> Информация: по умолчанию метод [[yii\web\Response::redirect()]] устанавливает код состояния ответа равным 302, сообщая тем самым браузеру,
  что запрошенный ресурс *временно* находится по другому URI-адресу. Вы можете передать значение
  301 в коде состояния, чтобы сообщить браузеру, что ресурс был перемещен *навсегда*.

Если текущий запрос является AJAX-запросом, отправка заголовка `Location` не приведёт к автоматическому перенаправлению 
браузера. Чтобы решить эту задачу, метод [[yii\web\Response::redirect()]] устанавливает значение заголовка `X-Redirect`  
равным URL-адресу перенаправления. На стороне клиента вы можете написать JavaScript-код, который считывает значение этого заголовка и соответствующим образом
перенаправляет браузер.

> Информация: Yii поставляется с JavaScript-файлом `yii.js`, который предоставляет набор наиболее часто используемых JavaScript-инструментов,
  включая и перенаправление браузера с использованием заголовка `X-Redirect`. Следовательно, если вы используете этот JavaScript-файл
  (зарегистрировав пакет ресурсов [[yii\web\YiiAsset]]), вам не нужно писать никакой код для поддержки AJAX-перенаправления.


## Sending Files <a name="sending-files"></a>

Like browser redirection, file sending is another feature that relies on specific HTTP headers. Yii provides
a set of methods to support various file sending needs. They all have built-in support for HTTP range header.

* [[yii\web\Response::sendFile()]]: sends an existing file to client.
* [[yii\web\Response::sendContentAsFile()]]: sends a text string as a file to client.
* [[yii\web\Response::sendStreamAsFile()]]: sends an existing file stream as a file to client. 

These methods have the same method signature with the response object as the return value. If the file
to be sent is very big, you should consider using [[yii\web\Response::sendStreamAsFile()]] because it is more
memory efficient. The following example shows how to send a file in a controller action:

```php
public function actionDownload()
{
    return \Yii::$app->response->sendFile('path/to/file.txt');
}
```

If you are calling the file sending method in places other than an action method, you should also call
the [[yii\web\Response::send()]] method afterwards to ensure no extra content will be appended to the response.

```php
\Yii::$app->response->sendFile('path/to/file.txt')->send();
```

Some Web servers have a special file sending support called *X-Sendfile*. The idea is to redirect the
request for a file to the Web server which will directly serve the file. As a result, the Web application
can terminate earlier while the Web server is sending the file. To use this feature, you may call
the [[yii\web\Response::xSendFile()]]. The following list summarizes how to enable the `X-Sendfile` feature
for some popular Web servers:

- Apache: [X-Sendfile](http://tn123.org/mod_xsendfile)
- Lighttpd v1.4: [X-LIGHTTPD-send-file](http://redmine.lighttpd.net/projects/lighttpd/wiki/X-LIGHTTPD-send-file)
- Lighttpd v1.5: [X-Sendfile](http://redmine.lighttpd.net/projects/lighttpd/wiki/X-LIGHTTPD-send-file)
- Nginx: [X-Accel-Redirect](http://wiki.nginx.org/XSendfile)
- Cherokee: [X-Sendfile and X-Accel-Redirect](http://www.cherokee-project.com/doc/other_goodies.html#x-sendfile)


## Sending Response <a name="sending-response"></a>

The content in a response is not sent to the user until the [[yii\web\Response::send()]] method is called.
By default, this method will be called automatically at the end of [[yii\base\Application::run()]]. You can, however,
explicitly call this method to force sending out the response immediately.

The [[yii\web\Response::send()]] method takes the following steps to send out a response:

1. Trigger the [[yii\web\Response::EVENT_BEFORE_SEND]] event.
2. Call [[yii\web\Response::prepare()]] to format [[yii\web\Response::data|response data]] into 
   [[yii\web\Response::content|response content]].
3. Trigger the [[yii\web\Response::EVENT_AFTER_PREPARE]] event.
4. Call [[yii\web\Response::sendHeaders()]] to send out the registered HTTP headers.
5. Call [[yii\web\Response::sendContent()]] to send out the response body content.
6. Trigger the [[yii\web\Response::EVENT_AFTER_SEND]] event.

After the [[yii\web\Response::send()]] method is called once, any further call to this method will be ignored.
This means once the response is sent out, you will not be able to append more content to it.

As you can see, the [[yii\web\Response::send()]] method triggers several useful events. By responding to
these events, it is possible to adjust or decorate the response.
