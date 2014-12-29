---
layout: post
title: 读Tornado4.0文档Web framework之tornado.web模块笔记
description: Tornado, Web framework, tornado.web, document
---
几个月没写代码，发现[Tornado已经出v4.0版本](http://tornado.readthedocs.org/en/latest/web.html)了，时间过得真快，要把这几个月时间补回来喽。

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
        + `RequestHandler.xsrf_form_html()`：一个HTML `input`元素包括在所有`POST`表单里。此方法定义了`_xsrf`的`input`值，Tornado检查所有`POST` request以防止伪造的跨站request。如果在`Application`的setting里设置了`xsrf_cookies`参数，所有的HTML表单必须要包括此HTML。在一个template里，此方法应用`{{%% module xsrf_form_html() %%}}`调用。更多信息见[`check_xsrf_cookie()`](http://tornado.readthedocs.org/en/stable/web.html#tornado.web.RequestHandler.check_xsrf_cookie)。
        + `RequestHandler.xsrf_token`：当前用户（会话）的XSRF-prevention token。为阻止伪造跨站request，须设置`_xsrf` cookie和所有`POST` request里包含相同`_xsrf`值的一个参数。如果这两者不匹配，Tornado认为是潜在的伪造，拒绝表单提交。可见[Cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery)。
    * 应用设置
        + `tornado.web.Application(handlers=None, default_host='', transforms=None, **settings)`：组成一个web应用的request handler集合。这个类的实例可以调用，直接传递给HTTP服务器来服务于web应用。
            * 这个类的构造器含有`URLSpec`对象的列表或（正则、request类）tuple，当Tornado接收到request，会按序迭代列表，并实例化正则与request路径匹配的第一个request类的实例。这个request类可指定为任一类对象或（全限定）名称。每个tuple包含额外的元素，对应`URLSpec`构造器的参数（在Tornado 3.2前的版本只允许2个或3个元素的tuple）。
            * 一个dictionary将是handler构造器和`RequestHandler.initialize`的一个关键参数，被当作第三个元素传递给tuple。此模式用在`StaticFileHandler`例子中，如果设置了static_path参数，`StaticFileHandler`将自动安装。
            * Tornado用`add_handlers`方法支持虚拟主机，第一个参数是主机的正则表达式。可以设置static_path这个关键参数来使用静态文件，Tornado通过`/static/` URI使用这些文件（用`static_url_prefix`参数配置），Tornado从相同的目录使用`/favicon.ico`和`/robots.txt`。一个自定义的`StaticFileHandler`子类可以指定`static_handler_class`的设置。
            * `settings`：传递给构造器的额外关键参数保存在`settings` dictionary里，在文档里通常称为"application settings"。`settings`用来自定义Tornado的各种变量（尽管一些例子中的丰富自定义设置可以通过重写`RequestHandler`子类实现）。一些应用也喜用`settings` dictionary这种方式给handler设定应用特殊的设置，而不用全局变量。Tornado的`settings`如下所述。
                1. 通用设置
                    * `autoreload`：如果此参数值为`True`，当任何资源文件改变时，服务器进程将会重启（详见[Debug mode and automatic reloading](http://tornado.readthedocs.org/en/stable/guide/running.html#debug-mode)）。此选项是Tornado 3.2的新特性，之前的功能实现是通过`debug`参数设定的。
                    * `debug`：一些debug模式设置的简写（详见[Debug mode and automatic reloading](http://tornado.readthedocs.org/en/stable/guide/running.html#debug-mode)）。设置`debug=True`等同于`autoreload=True`，`compiled_template_cache=False`，`static_hash_cache=False`，`serve_traceback=True`。
                    * `default_handler_class`和`default_handler_args`：如果没有找到其他的匹配，这个handler将使用。用来实现自定义的404页面（Tornado 3.2的新特性）。
                    * `compress_response`：如果此参数值为`True`，文本格式的response将自动压缩（Tornado 4.0的新特性）。
                    * `gzip`：`compress_response`的过时别名（从Tornado 4.0开始）。
                    * `log_function`：此函数在每个request结束时调用来记录结果（在`RequestHandler`对象，用一个参数）。默认的实现方法是记录在模块的根日志，也可以重写`Application.log_request`来实现自定义。
                    * `serve_traceback`：如果此参数值为`True`，默认的错误页将包含错误的traceback。这是Tornado 3.2的新特性，之前版本的此功能用`debug`参数控制。
                    * `ui_modules`和`ui_methods`：设定`UIModule`的映射或template可用的UI方法。也可设置给一个模块、dictionary，或模块的一个列表，或dicts。
                2. 认证和安全设置
                    * `cookie_secret`：用`RequestHandler.get_secure_cookie`和`set_secure_cookie`给cookie签名。
                    * `login_url`：如果用户未登录，`authenticated`装饰器将定向到此URL。可重写`RequestHandler.get_login_url`来自定义。
                    * `xsrf_cookies`：如果此参数值为`True`，Tornado将启用[防止伪造跨站request](http://tornado.readthedocs.org/en/stable/guide/security.html#xsrf)。
                    * `xsrf_cookie_version`：控制服务器产生的新的XSRF cookie的版本。通常是默认（支持的最高版本），但是在版本过渡时期，可能临时设置一个更低的值。这是Tornado 3.2.2的新特性。
                    * `twitter_consumer_key`/`twitter_consumer_secret`/`friendfeed_consumer_key`/`friendfeed_consumer_secret`/`google_consumer_key`/`google_consumer_secret`/`facebook_api_key`/`facebook_secret`：在`tornado.auth`模块用来认证不同的API。
                3. Template设置
                    * `autoescape`：控制template的自动转义。可设置为空来禁用转义，或者设定一个所有的输出都会传递的函数名。默认是`xhtml_escape`，可用在每个template的基础上用`{{%% autoescape %%}}`命令改变。
                    * `compiled_template_cache`：默认此参数值为`True`，如果为`False`，template将对每个request重编译。此选项是Tornado 3.2的新特性，之前的功能实现是通过`debug`参数设定的。
                    * `template_path`：目录包含template文件，也可以重写`RequestHandler.get_template_path`来实现自定义。
                    * `template_loader`：指定一个`tornado.template.BaseLoader`实例以实现自定义template的加载。如果此项设置已使用，可忽略`autoescape`和`template_path`设置，也可以重写`RequestHandler.create_template_loader`来实现自定义。
                4. 静态文件设置
                    * `static_hash_cache`：默认此参数值为`True`，如果为`False`，静态URL将对每个request重编译。此选项是Tornado 3.2的新特性，之前的功能实现是通过`debug`参数设定的。
                    * `static_path`：静态文件的目录。
                    * `static_url_prefix`：静态文件的URL前缀，默认为`"/static/"`。
                    * `static_handler_class`/`static_handler_args`：可设置使用不同的handler处理静态文件，而不是默认的`tornado.web.StaticFileHandler`。如果设置了`static_handler_args`，一个目录的关键参数可传递给handler的`initialize`方法。
            * `listen(port, address='', **kwargs)`：在给定的端口为应用启动HTTP服务器。这样为创建一个`HTTPServer`对象提供方便，调用它的监听方法。`HTTPServer.listen`不支持传递关键参数给`HTTPServer`构造器。高级用法（如多进程模式）不使用此方法，而采用创建一个`HTTPServer`和直接调用它的`TCPServer.bind`/`TCPServer.start`。注意在调用此方法后仍然需要调用`IOLoop.instance().start()`来启动服务器。
            * `add_handlers(host_pattern, host_handlers)`：给handler列表添加给定的handler。主机模式依次按它们的添加顺序进行处理，Tornado会考虑所有匹配的模式。
            * `reverse_url(name, *args)`：根据命名的`name`参数返回一个URL路径。handler作为命名的`URLSpec`被添加给`Application`。`URLSpec`正则的捕获组代替`args`参数，如果需要它们将被转化为string，用utf8编码，url转义。
            * `log_request(handler)`：把已完成的HTTP request写日志。默认写在python根日志，`Application`的任一子类重写此方法可改变日志路径，或传递一个函数给`Application`的settings作为`log_function`。
         + `tornado.web.URLSpec(pattern, handler, kwargs=None, name=None)`：指定URL和handler之间的映射。`tornado.web.url`可使用`URLSpec`类。
            * `pattern`：匹配正则表达式。正则中的任何组合作为参数将传递给handler的`GET`/`POST`等方法。
            * `handler`：被调用的`RequestHandler`的子类。
            * `kwargs`（可选）：传递额外参数的一个dictionary给handler构造器。
            * `name`（可选）：`Application.reverse_url`所使用的handler的名字。
    * 装饰器
         + `tornado.web.asynchronous(method)`：封装异步的request handler方法。使用`@gen.coroutine`而非`@asynchronous`是Tornado 3.1的新特性。
            * 如果此方法也用`@gen.coroutine`装饰，此装饰器为非必需的。两个装饰器一起使用是合法但非必需的，要把`@asynchronous`放在首位。
            * 此装饰器仅应用于HTTP动作方法，任何其他的方法未定义这种行为。此装饰器不会使一个方法异步，它告诉框架这个方法使异步的，方法有时的异步动作对于此装饰器是有用的。
            * 如果给定了装饰器，当方法返回时，response未完成。它决定了request handler调用`self.finish()`来结束HTTP request。无此装饰器，当`GET`/`POST`方法返回时，request自动结束。
         + `tornado.web.authenticated(method)`：用户已登录需要此装饰器来装饰方法，如果用户未登录将被定向到配置的`login url`。如果你用查询参数配置了`login url`，Tornado将假设你知道你在用它做什么；否则Tornado将加一个`next`参数，一旦你登录了，登录页知道发送哪个位置（页面）给你。
         + `tornado.web.addslash(method)`：使用此装饰器给request路径添加一个缺失的跟踪的斜杠。例如，一个request用此装饰器从`/foo`定向到`/foo/`，你的request handler映射使用一个正则表达式，如`r'/foo/?'`用此装饰器来连接。
         + `tornado.web.removeslash(method)`：使用此装饰器从request路径删除跟踪的斜杠。例如，一个request用此装饰器从`/foo`定向到`/foo/`，你的request handler映射使用一个正则表达式，如`r'/foo/*'`用此装饰器来连接。
         + `tornado.web.stream_request_body(cls)`：应用`RequestHandler`子类类支持stream body。在`data_received`和异步的`prepare`之间有着巧妙的交互：调用`prepare`返回或yield后，可以在任何点首次调用`data_recieved`。
            * `HTTPServerRequest.body`未定义，body参数不会包括在`RequestHandler.get_argument`里。
            * 当已读取request header而不是读取整个body后，调用`RequestHandler.prepare`。
            * 子类必须定义`data_received(self, data)`方法：数据可用时可被调用零至多次。注意，如果request body为空，则不能调用`data_received`。
            * `prepare`和`data_received`可返回`Future`对象。例如，`@gen.coroutine`的例子，不会调用下一个方法直到这些`Future`对象已完成。
            * 读取整个body后将调用正常的HTTP方法（`POST`/`PUT`，等等）。 
    * 其他的东西
         + `tornado.web.HTTPError(status_code, log_message=None, *args, **kwargs)`：一个异常将转化为HTTP错误response。一旦它自动结束了当前的函数，调用`RequestHandler.send_error`抛出一个`HTTPError`是方便的选择。用`HTTPError`自定义发送的response，可重写`RequestHandler.write_error`。
             * status_code(int)：HTTP状态码。必须在`httplib.responses`里列出来，除非给定了`reason`这个关键参数。
             * log_message(string)：用日志记录这个错误的信息（不会显示给用户除非`Application`的settings在`debug`模式）。也可能包含`%s`形式的占位符，将用剩余的位置参数填充。
             * reason (string)：关键参数。HTTP "reason"表达用`status_code`在状态行传递。一般从`status_code`自动决定，但是也可用一个非标准的数字码。
         + `tornado.web.Finish`：一个结束request但不用产生错误response的异常。当`RequestHandler`里抛出`Finish`，request将结束（如果还没有调用则调用`RequestHandler.finish`）。但是发出的response不会被修改，也不会调用处理错误的方法（包括`RequestHandler.write_error`）。实现自定义错误页比重写`write_error`更方便。
         + `tornado.web.MissingArgumentError(arg_name)`：`RequestHandler.get_argument`抛出的异常。这是`HTTPError`的子类，所以如果未捕获，一个400的响应码将代替500（并不会记录stack跟踪）。这是Tornado 3.1的新特性。
         + `tornado.web.UIModule(handler)`：在页面上可重用的、模块化的UI单元。UI模块通常执行额外的查询，并包括额外的CSS和JavaScript在输出页面上，在页面渲染上自动插入。
             * `render(*args, **kwargs)`：重写子类来返回此模块的输出。
             * `embedded_javascript()`：返回一个JavaScript string镶嵌在页面里。
             * `javascript_files()`：返回此模块需要的一个JavaScript文件的列表。
             * `embedded_css()`：返回一个CSS string镶嵌在页面里。
             * `css_files()`：返回此模块需要的一个CSS文件的列表。
             * `html_head()`：返回一个CSS string放置在<head>元素里。
             * `html_body()`：返回一个HTML string放置在<body>元素里。
             * `render_string(path, **kwargs)`：渲染一个template并当作一个string返回。
         + `tornado.web.ErrorHandler(application, request, **kwargs)`：为所有的request生成一个带`status_code`的错误reponse。
         + `tornado.web.FallbackHandler(application, request, **kwargs)`：一个封装了其他HTTP服务器回调的`RequestHandler`。这个fallback是接受一个`HTTPServerRequest`的可调用对象，例如一个`Application`或`tornado.wsgi.WSGIContainer`。一台服务器上同时用Tornado `RequestHandler`和WSGI，这样做最有用。
         + `tornado.web.RedirectHandler(application, request, **kwargs)`：为所有的request把客户端定向到给定的URL，应提供关键参数`url`给handler。
         + `tornado.web.StaticFileHandler(application, request, **kwargs)`：一个简单的处理目录里静态内容的handler。如果把关键参数`static_path`传递给`Application`，`StaticFileHandler`会自动配置。此handler可用来`static_url_prefix`/`static_handler_class`/`static_handler_args`实现自定义。此handler构造器需要一个`path`参数，这个参数指定来处理内容的根目录。注意，正则里匹配组合要解析`GET`方法里参数`path`的值（这里与前述的构造器参数不同，详见[`URLSpec`](http://tornado.readthedocs.org/en/stable/web.html#tornado.web.URLSpec)）。为最大化浏览器缓存的效率，这个类支持各版本的url，默认用参数`?v=`表示。如果给定了一个版本，Tornado告诉浏览器无限期缓存此文件。`make_static_url`用来构造带版本的url（也可用`RequestHandler.static_url`）。此handler主要用在开发中，以及服务于轻量级的文件处理；对于高流量的服务，使用一台专用的静态文件服务器更有效率（例如nginx或Apache）。推荐用HTTP `Accept-Ranges`机制返回部分的文件（因为一些浏览器需要此功能存在于HTML5的audio或video中），但是此handler不能用于处理占用太大内存的文件。
             * `子类注意事项`：此类可用子类扩展，但由于静态url是类方法生成的而非实例方法，继承模式也有些不同。重写类方法时确保用`@classmethod`装饰器，实例方法可用`self.path`、`self.absolute_path`和`self.modified`属性。子类只可重写此部分讨论的方法，重写其他方法容易产生错误，由于与`compute_etag`还有其他方法联系紧密，重写`StaticFileHandler.get`特别容易出问题。要改变静态url生成方式（例如匹配其他服务器的行为或CDN），重写`make_static_url`/`parse_url_path`/`get_cache_time`，抑或`get_version`。要替代文件系统的所有交互（例如处理数据库中的静态内容），重写`get_content`/`get_content_size`/`get_modified_time`/`get_absolute_path`/`validate_absolute_path`。Tornado 3.1的改变：添加了子类的许多方法。
             * `compute_etag()`：基于url版本号设置`Etag` header。允许对缓存的版本进行有效的`If-None-Match`的检查，向部分的response发送正确的`Etag`（例如同样的`Etag`作为完整的文件）。这是Tornado 3.1的新特性。
             * `set_headers()`：给response设置内容和缓存header。这是Tornado 3.1的新特性。
             * `should_return_304()`：如果header说明应返回304则返回`True`。这是Tornado 3.1的新特性。
             * `get_absolute_path(root, path)`：返回相对于`root`参数的`path`参数的绝对路径。`root`参数是`StaticFileHandler`配置的路径（大多数例子是`Application`的settings中`static_path`参数）。此类方法可用子类重写，默认返回文件系统路径，但是，也可用其他的string，只要子类重写的`get_content`是唯一且可理解的。这是Tornado 3.1的新特性。
             * `validate_absolute_path(root, absolute_path)`：验证并返回绝对路径。`root`参数是`StaticFileHandler`配置的路径，`path`参数是`get_absolute_path`的结果。这个实例方法在处理request期间调用，所有它可能抛出`HTTPError`或用类似`RequestHandler.redirect`的方法（重定向以阻止进一步的处理后返回空），如此为缺失的文件生成404错误。此方法可在返回前修改路径，但是，请注意，任何这样的更改都不会被`make_static_url`理解。在实例方法中，此方法的结果可用作`self.absolute_path`。这是Tornado 3.1的新特性。
             * `get_content(abspath, start=None, end=None)`：根据给定的绝对路径，提取请求的资源中的内容。此方法可被子类重写，注意，它的签名不同于其他可重写的方法（无`settings`设置）。这样做是为了确保`abspath`参数能够作为缓存键支持自身。此方法可返回一个byte string或byte string的迭代器，后者适用于大文件，有助于减少内存碎片。Tornado 3.1的新特性。
             * `get_content_version(abspath)`：根据给定的路径返回资源的版本string。此类方法可被子类重写，默认实现是文件内容的hash。这是Tornado 3.1的新特性。
             * `get_content_size()`：根据给定的路径返回资源的总大小。此方法可被子类重写，这是Tornado 3.1的新特性。Tornado 4.0的改变：通常不调用此方法，仅当请求了部分的结果。
             * `get_modified_time()`：返回`self.absolute_path`最近一次修改的时间。此方法可被子类重写，返回一个`datetime`对象或空。这是Tornado 3.1的新特性。
             * `get_content_type()`：返回request的`Content-Type` header。这是Tornado 3.1的新特性。
             * `set_extra_headers(path)`：子类给response添加额外的header。
             * `get_cache_time(path, modified, mime_type)`：重写以实现自定义缓存控制，返回正秒数，使结果可缓存，时间量或零到标记资源可缓存的未指定的时间量（视浏览器的heuristics算法）。默认用`v`参数为请求的资源返回缓存十年有效期。
             * `make_static_url(settings, path, include_version=True)`：用给定的路径返回一个带版本号的url。此方法可被子类重写（但是请注意，这是一个类方法而非实例方法）。子类仅需要实现签名`make_static_url(cls, settings, path)`，其他的根据参数可通过`static_url`传递（非标准）。`settings`参数是`Application.settings`的dictionary，`path`参数是请求的静态路径，返回的url是当前主机的相对路径。`include_version`参数决定了生成的url是否包含对应给定的路径的文件的版本号hash的query string。
             * `parse_url_path(url_path)`：将静态URL路径转化为文件系统路径。`url_path`参数用删除的`static_url_prefix`参数组成了URL，返回值是文件系统与`static_url`的相对路径。此方法与`make_static_url`可逆。
             * `get_version(settings, path)`：生成静态URL中的版本string。`settings`参数是`Application.settings`的dictionary，`path`参数是请求文件系统上资源位置的相对路径，如果未确定版本号，返回值应为string或空。Tornado 3.1的改变：之前推荐此方法用子类重写，现更推荐`get_content_version`，因为它允许基类处理结果的缓存。


--EOF--
