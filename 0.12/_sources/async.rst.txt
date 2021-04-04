비동기 응용 입문
================

비동기 설계 패턴은 `WSGI <http://www.python.org/dev/peps/pep-3333/>`_\의 동기적 성격과 잘 어울리지 않는다. 그래서 대부분의 비동기 프레임워크들(tornado, twisted, ...)에선 비동기 기능을 드러내기 위한 특별한 API를 구현하고 있다. 보틀은 WSGI 프레임워크이고 WSGI의 동기적 성격을 공유하긴 하지만 정말 멋진 `gevent 프로젝트 <http://www.gevent.org/>`_ 덕분에 보틀로도 비동기 응용을 작성하는 게 가능하다. 이 글에선 비동기식 WSGI로 보틀을 사용하는 방법을 다룬다.

동기식 WSGI의 한계
------------------

`WSGI 명세 (pep 3333) <http://www.python.org/dev/peps/pep-3333/>`_\에서 규정하는 요청/응답 사이클을 간략하게 말하면 이렇다. 각 요청마다 응용 콜러블이 호출되고 그 콜러블은 바디 이터레이터를 반환해야 한다. 그러면 서버가 바디를 돌면서 각 덩어리를 소켓에 써넣는다. 바디 이터레이터가 끝나면 클라이언트 연결을 닫는다.

간단명료하지만 문제가 있다. 이 모든 게 동기적으로 이뤄진다는 점이다. 응용에서 데이터를 기다려야 한다면 (IO, 소켓, 데이터베이스, ...) 빈 문자열을 내놓거나 (바쁜 대기), 아니면 현재 스레드를 블록시켜야 한다. 두 해법 모두 처리 스레드를 점유하므로 새 요청에 답할 수 없게 된다. 그래서 스레드별로 동시에 한 요청만 처리할 수 있다.

대부분 서버에서는 스레드의 비교적 높은 오버헤드를 피하기 위해 그 수를 제한한다. 20개나 그보다 적은 스레드로 이뤄진 풀을 많이 쓴다. 모든 스레드가 점유되면 새 연결은 모두 정지 상태가 되고, 밖에서 보기에 실질적으로 서버가 죽은 게 된다. 실시간 업데이트를 위해 오래 폴링하는 AJAX 요청을 써서 채팅을 구현하려 한다면 동시 연결 20개가 한계가 될 것이다. 꽤 소규모의 채팅이다.

해결사 Greenlet
---------------

대부분 서버에서는 스레드 간 전환과 새 스레드 생성의 높은 오버헤드 때문에 동시 스레드 수가 상당히 적도록 작업 스레드 풀 크기를 제한한다. 스레드가 프로세스(fork)에 비해 싸긴 하지만 새 연결마다 만들기엔 여전히 비용이 크다.

`gevent <http://www.gevent.org/>`_ 모듈은 거기에 *그린렛(greenlet)*\을 추가한다. 그린렛은 전통적인 스레드와 비슷하게 동작하지만 생성 비용이 아주 작다. gevent 기반 서버에서는 거의 오버헤드 없이 (연결마다 하나씩) 그린렛 수천 개를 만들 수 있다. 그리고 개별 그린렛이 블록돼도 서버가 새 요청을 받는 데 아무 영향도 주지 않는다. 동시 연결 수에 실질적으로 제한이 없다.

그래서 비동기식 응용을 만드는 게 엄청나게 쉬워지는데, 모양새가 동기식 응용과 비슷하기 때문이다. 사실 gevent 기반 서버는 비동기가 아니라 초다중 스레드다. 다음이 예이다. ::

    from gevent import monkey; monkey.patch_all()

    from time import sleep
    from bottle import route, run

    @route('/stream')
    def stream():
        yield 'START'
        sleep(3)
        yield 'MIDDLE'
        sleep(5)
        yield 'END'

    run(host='0.0.0.0', port=8080, server='gevent')

첫 번째 행이 중요하다. gevent에서 파이썬의 블로킹 API 대부분을 몽키패치해서 현재 스레드를 블록시키지 않고 CPU를 다음 그린렛으로 넘기게 만든다. 실제로 파이썬의 threading 모듈을 gevent 기반의 유사 스레드로 교체한다. 그래서 원래는 스레드 전체를 블록시킬 ``time.sleep()``\을 사용할 수 있는 것이다. 만약 파이썬 내장 모듈들을 몽키패치하는 게 마음에 들지 않는다면 대응하는 gevent 함수(이 경우 ``gevent.sleep()``)를 쓸 수 있다.

이 스크립트를 돌리고 브라우저로 ``http://localhost:8080/stream``\을 열면 `START`, `MIDDLE`, `END`\가 (8초 후에 한번에 다 나오는 게 아니라) 하나씩 나오는 걸 볼 수 있다. 정확히 일반 스레드처럼 동작하는데도 이제는 서버가 수천 개의 동시 요청을 문제없이 처리할 수 있다.

.. note::

    일부 브라우저에선 페이지 렌더링을 시작하기 전에 데이터를 어느 정도
    버퍼링한다. 그런 브라우저에선 효과를 눈으로 확인하려면 몇 바이트 더
    yield 해야 할 수도 있다. 더불어 많은 브라우저에선 URL당 동시 연결이
    1개로 제한돼 있다. 해당하는 경우 브라우저를 하나 더 띄우거나
    벤치마크 도구(예: `ab`, `httperf`)를 써서 성능을 측정할 수 있다.

이벤트 콜백
-----------

비동기 프레임워크(tornado, twisted, node.js 등)에서 아주 흔한 설계 패턴은 논블로킹 API를 쓰면서 비동기 이벤트에 콜백을 결속시키는 것이다. 명시적으로 닫기 전까진 소켓 객체를 계속 열어 두기 때문에 콜백에서 이후에 소켓에 쓰기를 할 수 있다. 다음은 `tornado 라이브러리 <http://www.tornadoweb.org/documentation#non-blocking-asynchronous-requests>`_\를 기반으로 한 예이다. ::

    class MainHandler(tornado.web.RequestHandler):
        @tornado.web.asynchronous
        def get(self):
            worker = SomeAsyncWorker()
            worker.on_data(lambda chunk: self.write(chunk))
            worker.on_finish(lambda: self.finish())

이렇게 하면 좋은 점은 요청 핸들러가 빨리 끝난다는 것이다. 콜백에서 계속 이전 요청의 소켓에 쓰기를 하는 동안 처리 스레드는 다음으로 넘어가서 새 요청을 받을 수 있다. 그런 프레임워크들은 이런 식으로 해서 적은 OS 스레드만으로 많은 동시 요청을 처리해 낸다.

Gevent+WSGI에서는 사정이 다르다. 첫째로, 새 연결을 받아들일 수 있는 무한한 (유사) 스레드 풀이 있기 때문에 콜백을 일찍 끝내는 게 좋은 점이 없다. 둘째로, 일찍 끝낼 수가 없기도 한데, 그러면 (WSGI에서 요구하는 대로) 소켓을 닫게 될 것이기 때문이다. 셋째로, WSGI를 준수하려면 이터러블을 반환해야 한다.

WSGI 표준을 준수하기 위해서 우리가 해야 할 일은 비동기적으로 쓰기를 할 수 있는 바디 이터러블을 반환하는 것이다. `gevent.queue <http://www.gevent.org/gevent.queue.html>`_\의 도움으로 분리된 소켓을 *흉내내서* 앞의 예를 다음처럼 작성할 수 있다. ::

    @route('/fetch')
    def fetch():
        body = gevent.queue.Queue()
        worker = SomeAsyncWorker()
        worker.on_data(body.put)
        worker.on_finish(lambda: body.put(StopIteration))
        worker.start()
        return body

서버 관점에서 이 큐 객체는 이터러블이다. 비어 있으면 블록하고 ``StopIteration``\을 만나면 바로 멈춘다. 이는 WSGI을 준수한다. 응용에서 보기에 그 큐 객체는 논블로킹 소켓처럼 동작한다. 언제든 거기에 쓰기를 할 수 있고 이리저리 전달할 수 있으며, 심지어 거기에 비동기적으로 쓰기를 하는 새 (유사) 스레드를 시작할 수도 있다. 대부분의 경우 이런 식으로 긴 폴링을 구현한다.


마지막으로: 웹소켓
------------------

저수준 세부 사항에 대해선 잠시 잊고서 웹소켓에 대해 얘기해 보자. 이 글을 읽고 있다면 아마 웹소켓이 뭔지 알고 있을 텐데, 브라우저(클라이언트)와 웹 응용(서버) 사이의 양방향 통신 채널이다.

고맙게도 `gevent-websocket <http://pypi.python.org/pypi/gevent-websocket/>`_ 패키지가 어려운 일을 다 해 준다. 다음은 메시지를 받아서 그냥 클라이언트로 돌려 보내는 단순한 웹소켓 종점이다. ::

    from bottle import request, Bottle, abort
    app = Bottle()

    @app.route('/websocket')
    def handle_websocket():
        wsock = request.environ.get('wsgi.websocket')
        if not wsock:
            abort(400, 'Expected WebSocket request.')

        while True:
            try:
                message = wsock.receive()
                wsock.send("Your message was: %r" % message)
            except WebSocketError:
                break

    from gevent.pywsgi import WSGIServer
    from geventwebsocket import WebSocketHandler, WebSocketError
    server = WSGIServer(("0.0.0.0", 8080), app,
                        handler_class=WebSocketHandler)
    server.serve_forever()

클라이언트가 연결을 닫을 때까지 while 루프가 돈다. 그게 전부다. :)

클라이언트 측 자바스크립트 API 역시 정말 단순명료하다. ::

    <!DOCTYPE html>
    <html>
    <head>
      <script type="text/javascript">
        var ws = new WebSocket("ws://example.com:8080/websocket");
        ws.onopen = function() {
            ws.send("Hello, world");
        };
        ws.onmessage = function (evt) {
            alert(evt.data);
        };
      </script>
    </head>
    </html>

