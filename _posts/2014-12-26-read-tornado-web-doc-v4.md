---
layout: post
title: 读Tornado4.0文档Web framework之tornado.web模块笔记
description: Tornado, Web framework, tornado.web, document
---
几个月没写代码，发现[Tornado已经出v4.0版本](http://tornado.readthedocs.org/en/latest/web.html)了，时间过得真快，要把这几个月时间补回来喽。

#Web framework
- ###tornado.web - RequestHandler & Application classes

    * `tornado.web`提供一个简单的具有异步功能的web框架，允许扩展大量的连接，是长轮询的理想实现。
    * 线程安全：`RequestHandler`的方法都是非线程安全的，但是，`write()`/`finish()`/`flush()`这几个方法是只能在主线程中调用。如果用多线程，在结束request前用`IOLoop.add_callback`方法把控制转移给主线程。
    * RequestHandler
        + 在`SUPPORTED_METHODS`里定义了`GET`/`POST`/`HEAD`/`DELETE`/`PATCH`/`PUT`/`OPTIONS`这些标准方法，如果想要支持更多方法，需要自己在`RequestHandler`子类中重写`SUPPORTED_METHODS`。
        + Entry points是`RequestHandler.initialize()`。
        + `RequestHandler.prepare()`在request开始时，`get()`/`post()`等方法之前调用，通常这个方法用来实现一些初始化工作。不能用`asynchronous`来装饰此方法，可以用`gen.coroutine`或`return_future`来装饰此方法使它支持异步。如果此方法返回`Future`则不会执行，除非`Future`对象已完成。
        + `RequestHandler.on_finish()`在request结束时调用，对应于`RequestHandler.prepare()`。在response返回给客户端后，此方法被调用，亦可能没有任何输出。
        + `RequestHandler.get()`/`RequestHandler.post()`/`RequestHandler.put()`/`RequestHandler.delete()`/`RequestHandler.head()`/`RequestHandler.options()`可以使用`gen.coroutine`/`return_future`/`asynchronous`中的任一装饰器来实现异步。
    * 输入
        + `RequestHandler.get_argument()`返回给定参数名的值（通常是unicode），如果没有提供默认值，Tornado会认为此参数是必需参数，当缺少此参数时会抛出`MissingArgumentError`。如果此参数在url中出现不止一次，Tornado只返回最后一次值。
        + `RequestHandler.get_arguments()`返回给定参数名的一个list（通常是unicode），如果此参数不存在，Tornado返回一个空的list。
        + `RequestHandler.get_query_argument()`从request查询串中返回给定参数名的值（通常是unicode）。如果没有提供默认值，Tornado会认为此参数是必需参数，当缺少此参数时会抛出`MissingArgumentError`。如果此参数在url中出现不止一次，Tornado只返回最后一次值。
        + `RequestHandler.get_query_arguments()`返回在查询参数中给定参数名的一个list（通常是unicode），如果此参数不存在，Tornado返回一个空的list。
        + `RequestHandler.get_body_argument()`从request的body中返回给定参数名的值（通常是unicode）。如果没有提供默认值，Tornado会认为此参数是必需参数，当缺少此参数时会抛出`MissingArgumentError`。如果此参数在url中出现不止一次，Tornado只返回最后一次值。
        + `RequestHandler.get_body_arguments()`返回在body参数中给定参数名的一个list（通常是unicode），如果此参数不存在，Tornado返回一个空的list。
        + `RequestHandler.decode_argument()`被用来过滤从`get_argument()`和url中提取传递给`get()`/`post()`等方法的值。参数已经按比例解码，现在是byte string。此方法默认用utf8解码并返回unicode串，但是也可以在子类中重写改变。
        + `RequestHandler.request`：`tornado.httputil.HTTPServerRequest`对象包含额外的request参数例如`headers`和`body data`。
        + `RequestHandler.path_args`和`RequestHandler.path_kwargs`：包含位置的和被传递给HTTP方法的关键参数，在HTTP方法调用前设置属性，因此在`RequestHandler.prepare()`执行时属性值可用。
    * 输出
        + `RequestHandler.set_status()`是设置response的状态码。
        + `RequestHandler.set_header()`设置给定的response header的name和value。如果给定了datetime，Tornado自动按照HTTP要求将其格式化。如果value不是string，Tornado会将其转为string。所有的value都会按照utf8编码。
        + `RequestHandler.add_header()`添加给定的response header的name和value。不像`set_header`，`add_header`可以调用多次为同一个header返回多个值。
        + `RequestHandler.clear_header()`清除一个外发的header，撤销之前`set_header`的调用。此方法不应用于`add_header`设置的多值headers。
        + `RequestHandler.set_default_headers()`重写此方法在request开始时设置HTTP headers，可自定义服务器header，但是在正常的request流程中header可能不会执行前述设置。在错误处理期间，headers会被重置。
        + `RequestHandler.write(chunk)`：把给定的chunk写给输出缓存。如果给定的chunk是一个dictionary，Tornado把它当JSON，并设置`Content-Type`属性值为`application/json`（如果要输出JSON用一个不同的`Content-Type`属性，在`write()`调用后使用`set_header`）。需要注意的是，由于潜在的跨站安全隐患，list不会被转换为JSON，所有的JSON输出都会被封装在dictionary里。
        + `RequestHandler.flush(include_footers=False, callback=None)`：刷新当前的输出缓存到网络。如果给定了`callback`参数，当所有刷新的数据写给了Socket，它将会执行。一次只能执行一个`flush callback`，如果前一个`flush callback`还没完成，另一个`flush callback`就开始执行的话，前一个`flush callback`将会被丢弃。如果没有给定`callback`参数，Tornado会返回一个`Future`对象。
        + `RequestHandler.finish(chunk=None)`：完成response，结束HTTP request。
        + `RequestHandler.render(template_name, **kwargs)`：用给定的参数渲染template作为response。
        + `RequestHandler.render_string(template_name, **kwargs)`：根据给定的参数生成给定的template（utf8的byte string）。
        + `RequestHandler.get_template_namespace()`：返回一个dictionary作为默认的template命名空间，可以重写子类来添加或修改值。此方法的结果将会联合`tornado.template`模块中额外的默认值和`render`或`render_string`的关键参数。
        + `RequestHandler.redirect(url, permanent=False, status=None)`：定向到给定的URL(可以是相对路径)。如果指定了`status`参数，这个参数值会作为HTTP状态码；否则Tornado根据`permanent`参数值返回301状态码（永久）或302（临时）状态码，默认为302状态码。
        + `RequestHandler.send_error(status_code=500, **kwargs)`：发送HTTP错误码给浏览器。如果已经调用了`flush()`方法就不可能发送错误，所以此方法将结束这个response。如果已经写了输出但是还未调用`flush()`方法，Tornado将弃用此方法，用`error page`来代替。重写`write_error()`来自定义`error page`，额外的关键参数会被传递给`write_error()`。
        + `RequestHandler.write_error(status_code, **kwargs)`：重写实现自定义的`error page`，此方法可调用`write()`/`render()`/`set_header()`等方法来产生正常的输出。如果这个错误是被一个未捕获的异常（包括HTTPError）引起的，一个`exc_info`将在`kwargs["exc_info"]`中可用。需要注意的是，这个异常可能不是当前的异常，如`sys.exc_info()`或`traceback.format_exc()`这些方法的意图。
        + `RequestHandler.clear()`：重置这个response中的所有headers和content。
        + `RequestHandler.data_received(chunk)`：实现此方法来处理request流数据，需要`stream_request_body`装饰器。
    * Cookies
        + `RequestHandler.cookies`：是`self.request.cookies`的别名。
        + `RequestHandler.get_cookie()`：根据给定的cookie name获取对应的值，否则为默认值。
        + `RequestHandler.set_cookie()`：根据给定的options设置给定的cookie name/value对。
        + `RequestHandler.clear_cookie(name, path='/', domain=None)`：根据给定的cookie name删除cookie。由于cookie协议的限制，必须传递在设置cookie时的相同path和domain来清除cookie，但是在服务器上是无法找出哪些值是属于给定的cookie。
        + `RequestHandler.clear_all_cookies(path='/', domain=None)`：删除用户用request发送的所有cookie，查看`clear_cookie`方法有更多关于path和domain参数的信息。
        + `RequestHandler.get_secure_cookie(name, value=None, max_age_days=31, min_version=None)`：验证通过返回指定的cookie否则为空，解码的cookie值是一个byte string（不同于`get_cookie`）。
        + `RequestHandler.set_secure_cookie(name, value, expires_days=30, version=None, **kwargs)`：给cookie加签名和时间戳，以防伪造。必须在`Application`的setting中指定`cookie_secret`参数。此参数是一个长的、随机byte序列，用来制作HMAC加密签名。要读取用此方法设置的cookie，请使用`get_secure_cookie()`。注意，`expires_days`参数用来设置cookie在浏览器中的生命周期，但是它与`get_secure_cookie()`的`max_age_days`参数无关。加密的cookie可能包含随意的byte值，而不仅仅是unicode string（不同于普通的cookie）。
        + `RequestHandler.create_signed_value(name, value, version=None)`：给cookie加签名和时间戳，以防伪造。一般用在`set_secure_cookie()`，但是也作为非cookie用途的一种方法来使用。要解码不作为cookie存储的值，须使用`get_secure_cookie()`方法的可选值的参数。
        + `tornado.web.MIN_SUPPORTED_SIGNED_VALUE_VERSION = 1`：此版本的Tornado支持最旧版本的签名值的版本号。比此版本号还旧的签名值无法解码。
        + `tornado.web.MAX_SUPPORTED_SIGNED_VALUE_VERSION = 2`：此版本的Tornado支持最新版本的签名值的版本号。比此版本号还新的签名值无法解码。
        + `tornado.web.DEFAULT_SIGNED_VALUE_VERSION = 2`：用`RequestHandler.create_signed_value()`方法产生的签名值的版本号，可以传递`version`关键参数来重写。
        + `tornado.web.DEFAULT_SIGNED_VALUE_MIN_VERSION = 1`：`RequestHandler.get_secure_cookie()`方法接受的最旧版本的签名值，可以传递`min_version`关键参数来重写。
    * 其他
        + `RequestHandler.application`：`Application`对象服务于request。
        + `RequestHandler.check_etag_header()`：检查`Etag` header是否与request[`If-None-Match`](http://en.wikipedia.org/wiki/HTTP_ETag)不同，如果request的`Etag`匹配则返回`True`和304状态码。当request结束时该方法自动调用，但是应用也可更早调用来重写`compute_etag`方法，并在完成request前更早进行`If-None-Match`检查。在调用此方法前应设置`Etag` header（可以用`set_etag_header`方法）。
        + `RequestHandler.check_xsrf_cookie()`：验证`_xsrf` cookie与`_xsrf`参数的匹配。为阻止伪造跨站request，须设置`_xsrf` cookie和所有`POST` request里包含相同值的非cookie field。如果这两者不匹配，Tornado认为是潜在的伪造，拒绝表单提交。`_xsrf`值可以设置在name为`_xsrf`的任一表单field，或一个自定义的name为`X-XSRFToken`或`X-CSRFToken`的HTTP header（后者与Django兼容）。之前的Tornado 1.1.1版本，如果HTTP header中存在`X-Requested-With: XMLHTTPRequest`，此方法检查被忽略。这个异常显示为不安全的，已经被清除，更多信息请看[http://www.djangoproject.com/weblog/2011/feb/08/security/](http://www.djangoproject.com/weblog/2011/feb/08/security/)和[http://weblog.rubyonrails.org/2011/2/8/csrf-protection-bypass-in-ruby-on-rails](http://weblog.rubyonrails.org/2011/2/8/csrf-protection-bypass-in-ruby-on-rails)。
        + `RequestHandler.compute_etag()`：计算用在request的`Etag` header，默认用内容的hash来写。可以重写来自定义etag的实现方法，或返回空来禁用Tornado的默认etag支持。
        + `RequestHandler.create_template_loader(template_path)`：根据给定的路径来返回一个新的template加载器。可以用子类重写，默认根据给定的路径返回基于目录的加载器，在`Application`的setting里用`autoescape`参数设置。如果`Application`的setting设置了`template_loader`，则用它代替。
        + `RequestHandler.current_user`：request的已认证用户。这是`get_current_user()`方法的缓存版本，你可以重写方法来设置用户基于其他方式认证，例如cookie。如果没有重写前述方法，此方法通常返回空。第一次调用此方法时，Tornado懒加载当前用户，之后便缓存结果。
        + `RequestHandler.get_browser_locale(default='en_US')`：由`Accept-Language` header决定用户的位置。可见[14.4 Accept-Language](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4)。
        + `RequestHandler.get_current_user()`：重写方法来决定当前用户，例如从cookie里。
        + `RequestHandler.get_login_url()`：重写方法来自定义基于request的登录URL，默认用`Application`的setting里的`login_url`。
        + `RequestHandler.get_status()`：返回response的状态码。
        + `RequestHandler.get_template_path()`：重写方法来自定义每个handler的template路径。默认用`Application`的setting里的`template_path`，加载相对调用文件的template返回空。
        + `RequestHandler.get_user_locale()`：重写方法来决定已认证用户的位置。如果已经返回了空，Tornado返回到`get_browser_locale()`。此方法应该返回`tornado.locale.Locale`对象，最可能通过像调用`tornado.locale.get("en")`获得。
        + `RequestHandler.log_exception(typ, value, tb)`：重写方法来自定义未捕获异常的日志。默认记录`HTTPError`的实例作为警告，而不跟踪stack（在`tornado.general`日志）。同时所有其他的异常作为跟踪stack的错误（在`tornado.application`日志）。
        + `RequestHandler.on_connection_close()`：如果客户端关闭了连接，在异步的handler中调用，重写方法来清除与长连接相关的资源。注意调用此方法只在异步化期间连接被关闭的情况，如果需要在每个request后清除，可重写`on_finish()`。在客户端关闭后，代理可保持一个连接开启一段时间（也许是无限期），所以，用户最后关闭了连接后，此方法可能不会被及时调用。
        + `RequestHandler.require_setting(name, feature='this feature')`：如果给定的`Application`的setting没有定义，抛出一个异常。
        + `RequestHandler.reverse_url(name, *args)`：`Application.reverse_url`的别名。
        + `RequestHandler.set_etag_header()`：用`self.compute_etag()`设置response的`Etag` header。注意，如果没有设置header，`compute_etag()`返回空。当request结束时，此方法自动调用。
        + `RequestHandler.settings`：`self.application.settings`的别名。
        + `RequestHandler.static_url(path, include_host=None, **kwargs)`：根据给定的静态文件相对路径返回静态的URL。此方法需要在`Application`的setting设置`static_path`参数（指定静态文件的根目录）。此方法返回一个版本url（默认附加在`?v=<signature>`），这样允许静态文件无限期缓存，默认可通过传递`include_version=False`来禁用（其他静态文件实现不需要支持此方法，他们可能需要支持其他的选项）。默认此方法返回当前主机的相对路径，但是如果`include_host`为`true`，则将返回绝对路径。如果handler有`include_host`属性，属性值将被默认为所有`static_url`调用，而不会作为关键参数传递给`include_host`。
        + `RequestHandler.xsrf_form_html()`：一个HTML `input`元素包括在所有`POST`表单里。此方法定义了`_xsrf`的`input`值，Tornado检查所有`POST` request以防止伪造的跨站request。如果在`Application`的setting里设置了`xsrf_cookies`参数，所有的HTML表单必须要包括此HTML。在一个template里，此方法应用`{% module xsrf_form_html() %}`调用。更多信息见[`check_xsrf_cookie()`](http://tornado.readthedocs.org/en/stable/web.html#tornado.web.RequestHandler.check_xsrf_cookie)。
        + `RequestHandler.xsrf_token`：当前用户（会话）的XSRF-prevention token。为阻止伪造跨站request，须设置`_xsrf` cookie和所有`POST` request里包含相同`_xsrf`值的一个参数。如果这两者不匹配，Tornado认为是潜在的伪造，拒绝表单提交。可见[Cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery)。
    * Application configuration
        


--EOF--
