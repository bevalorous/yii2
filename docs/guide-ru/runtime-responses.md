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


## Response Body <a name="response-body"></a>

Most responses should have a body which gives the content that you want to show to end users.

If you already have a formatted body string, you may assign it to the [[yii\web\Response::content]] property
of the response. For example,

```php
Yii::$app->response->content = 'hello world!';
```

If your data needs to be formatted before sending it to end users, you should set both of the
[[yii\web\Response::format|format]] and [[yii\web\Response::data|data]] properties. The [[yii\web\Response::format|format]]
property specifies in which format the [[yii\web\Response::data|data]] should be formatted. For example,

```php
$response = Yii::$app->response;
$response->format = \yii\web\Response::FORMAT_JSON;
$response->data = ['message' => 'hello world'];
```

Yii supports the following formats out of the box, each implemented by a [[yii\web\ResponseFormatterInterface|formatter]] class.
You can customize these formatters or add new ones by configuring the [[yii\web\Response::formatters]] property.

* [[yii\web\Response::FORMAT_HTML|HTML]]: implemented by [[yii\web\HtmlResponseFormatter]].
* [[yii\web\Response::FORMAT_XML|XML]]: implemented by [[yii\web\XmlResponseFormatter]].
* [[yii\web\Response::FORMAT_JSON|JSON]]: implemented by [[yii\web\JsonResponseFormatter]].
* [[yii\web\Response::FORMAT_JSONP|JSONP]]: implemented by [[yii\web\JsonResponseFormatter]].

While response body can be set explicitly as shown above, in most cases you may set it implicitly by the return value
of [action](structure-controllers.md) methods. A common use case is like the following:
 
```php
public function actionIndex()
{
    return $this->render('index');
}
```

The `index` action above returns the rendering result of the `index` view. The return value will be taken
by the `response` component, formatted and then sent to end users.

Because by default, the response format is as [[yii\web\Response::FORMAT_HTML|HTML]], you should only return a string
in an action method. If you want to use a different response format, you should set it first before returning the data.
For example,

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

As aforementioned, besides using the default `response` application component, you can also create your own
response objects and send them to end users. You can do so by returning such object in an action method, like the following,

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

> Note: If you are creating your own response objects, you will not be able to take advantage of the configurations
  that you set for the `response` component in the application configuration. You can, however, use 
  [dependency injection](concept-di-container.md) to apply common configuration to your new response objects.


## Browser Redirection <a name="browser-redirection"></a>

Browser redirection relies on sending a `Location` HTTP header. Because this feature is commonly used, Yii provides
some special supports for it.

You can redirect the user browser to a URL by calling the [[yii\web\Response::redirect()]] method. The method
sets the appropriate `Location` header with the given URL and returns the response object itself. In an action method,
you can call its shortcut version [[yii\web\Controller::redirect()]]. For example,

```php
public function actionOld()
{
    return $this->redirect('http://example.com/new', 301);
}
```

In the above code, the action method returns the result of the `redirect()` method. As explained before, the response
object returned by an action method will be used as the response sending to end users.

In places other than an action method, you should call [[yii\web\Response::redirect()]] directly followed by 
a call to the [[yii\web\Response::send()]] method to ensure no extra content will be appended to the response.

```php
\Yii::$app->response->redirect('http://example.com/new', 301)->send();
```

> Info: By default, the [[yii\web\Response::redirect()]] method sets the response status code to be 302 which instructs
  the browser that the resource being requested is *temporarily* located in a different URI. You can pass in a status
  code 301 to tell the browser that the resource has been *permanently* relocated.

When the current request is an AJAX request, sending a `Location` header will not automatically cause the browser
redirection. To solve this problem, the [[yii\web\Response::redirect()]] method sets an `X-Redirect` header with 
the redirection URL as its value. On the client side you may write JavaScript code to read this header value and
redirect the browser accordingly.

> Info: Yii comes with a `yii.js` JavaScript file which provides a set of commonly used JavaScript utilities,
  including browser redirection based on the `X-Redirect` header. Therefore, if you are using this JavaScript file
  (by registering the [[yii\web\YiiAsset]] asset bundle), you do not need to write anything to support AJAX redirection.


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
