---
layout: post
title: 读Tornado4.0文档阅读笔记一，Web framework
description: Tornado, doc
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
        
- ###tornado.template - Flexible output generation
- ###tornado.escape - Escaping & string manipulation
- ###tornado.locale - Internationalization support
- ###tornado.websocket - Bidirectional communication to the browser


--EOF--
