.. module:: bottle

.. _beaker: http://beaker.groovie.org/
.. _mod_python: http://www.modpython.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _werkzeug: http://werkzeug.pocoo.org/documentation/dev/debug.html
.. _paste: http://pythonpaste.org/modules/evalexception.html
.. _pylons: http://pylonshq.com/
.. _gevent: http://www.gevent.org/
.. _compression: https://github.com/defnull/bottle/issues/92
.. _GzipFilter: http://www.cherrypy.org/wiki/GzipFilter
.. _cherrypy: http://www.cherrypy.org
.. _heroku: http://heroku.com

간단 해법
=============

다음은 흔한 사용 방식을 위한 코드 조각과 예시 모음이다.

세션 추적하기
----------------------------

세션 내장 지원이 없다. (마이크로 프레임워크에선) 그걸 *제대로* 할 수 있는 방법이 없기 때문이다. 요구 사항과 환경에 따라서 beaker_ 미들웨어를 적당한 백엔드로 사용할 수도 있고 직접 구현할 수도 있다. 다음은 파일 기반 백엔드의 beaker 세션 예시다. ::

    import bottle
    from beaker.middleware import SessionMiddleware

    session_opts = {
        'session.type': 'file',
        'session.cookie_expires': 300,
        'session.data_dir': './data',
        'session.auto': True
    }
    app = SessionMiddleware(bottle.app(), session_opts)

    @bottle.route('/test')
    def test():
      s = bottle.request.environ.get('beaker.session')
      s['test'] = s.get('test',0) + 1
      s.save()
      return 'Test counter: %d' % s['test']

    bottle.run(app=app)

폼나게 디버깅 하기: 디버깅용 미들웨어
--------------------------------------------------------------------------------

보틀에선 WSGI 서버가 죽는 걸 방지하기 위해 앱 코드에서 던진 모든 예외를 잡는다. 내장 :func:`debug` 모드로는 충분치 않아서 예외가 디버깅용 미들웨어로 전파되기를 바란다면 그 동작을 끌 수 있다. ::

    import bottle
    app = bottle.app() 
    app.catchall = False # 이제 대부분의 예외를 보틀이 다시 던진다.
    myapp = DebuggingMiddleware(app) # 원하는 미들웨어를 여기에 (아래 참고)
    bottle.run(app=myapp)

그러면 보틀은 자체 예외(:exc:`HTTPError`, :exc:`HTTPResponse`, :exc:`BottleException`)만 잡고 미들웨어에서 나머지를 처리할 수 있다.

werkzeug_\와 paste_ 라이브러리에는 아주 강력한 디버깅용 WSGI 미들웨어가 딸려 있다. werkzeug_\에선 :class:`werkzeug.debug.DebuggedApplication`\를, paste_\에선 :class:`paste.evalexception.middleware.EvalException`\를 보라. 둘 모두 스택 검사를 지원할 뿐 아니라 그 스택 문맥에서 파이썬 코드를 실행할 수도 있다. 따라서 **운용 환경에서 사용해선 안 된다**.


보틀 응용 유닛 테스트
--------------------------------------------------------------------------------

일반적으로 WSGI 환경을 돌리지 않고 웹 응용에 정의된 메소드에 대해 유닛 테스트를 수행한다.

`Nose <http://readthedocs.org/docs/nose>`_\를 쓰는 간단한 예::

    import bottle
    
    @bottle.route('/')
    def index():
        return 'Hi!'

    if __name__ == '__main__':
        bottle.run()

테스트 스크립트::

    import mywebapp
    
    def test_webapp_index():
        assert mywebapp.index() == 'Hi!'

이 예에서 보틀 route() 메소드는 절대 실행되지 않는다. index()만 테스트 한다.


보틀 응용 기능 테스트
--------------------------------------------------------------------------------

동작 중인 WSGI 서버가 있으면 어떤 HTTP 기반 테스트 시스템이든 함께 사용 가능하지만 어떤 테스팅 프레임워크는 WSGI와 더 밀접하게 동작해서 통제된 환경에서 WSGI 응용을 호출할 수 있고, 트레이스백도 있고, 디버깅 도구들을 완전히 이용 가능하다. `WSGI 테스팅 도구들 <http://www.wsgi.org/en/latest/testing.html>`_\이 좋은 출발점이다.

`WebTest <http://webtest.pythonpaste.org/>`_\와 `Nose <http://readthedocs.org/docs/nose>`_\를 쓰는 예::

    from webtest import TestApp
    import mywebapp

    def test_functional_login_logout():
        app = TestApp(mywebapp.app)
        
        app.post('/login', {'user': 'foo', 'pass': 'bar'}) # 로그인 하고 쿠키 얻기

        assert app.get('/admin').status == '200 OK'        # 페이지 성공적으로 가져옴

        app.get('/logout')                                 # 로그아웃
        app.reset()                                        # 쿠키 버리기

        # 같은 페이지 가져오기, 이번엔 실패
        assert app.get('/admin').status == '401 Unauthorized'


다른 WSGI 앱 내장하기
--------------------------------------------------------------------------------

권장하는 방식은 아니지만 (보틀 앞에 미들웨어를 쓰는 게 맞다.) 보틀 앱 안에서 다른 WSGI 응용을 호출해서 보틀이 유사 미들웨어가 되게 할 수 있다. 다음이 예이다. ::

    from bottle import request, response, route
    subproject = SomeWSGIApplication()

    @route('/subproject/:subpath#.*#', method='ANY')
    def call_wsgi(subpath):
        new_environ = request.environ.copy()
        new_environ['SCRIPT_NAME'] = new_environ.get('SCRIPT_NAME','') + '/subproject'
        new_environ['PATH_INFO'] = '/' + subpath
        def start_response(status, headerlist):
            response.status = int(status.split()[0])
            for key, value in headerlist:
                response.add_header(key, value)
        return app(new_environ, start_response)

다시 말하지만 하위 프로젝트를 구현하기 위한 권장 방식이 아니다. 여러 사람들이 요청해서, 그리고 보틀이 어떻게 WSGI로 연결되는지 보여 주기 위해 있는 예시일 뿐이다.


마지막 슬래시 무시하기
--------------------------------------------------------------------------------

보틀에서 ``/example``\과 ``/example/``\은 서로 다른 라우트다. [1]_. 두 URL을 같은 걸로 취급하려면 ``@route`` 데코레이터를 두 개 붙여 주면 된다. ::

    @route('/test')
    @route('/test/')
    def test(): return 'Slash? no?'

아니면 모든 URL에서 끝의 슬래시를 없애 주는 WSGI 미들웨어를 추가할 수 있다. ::

    class StripPathMiddleware(object):
      def __init__(self, app):
        self.app = app
      def __call__(self, e, h):
        e['PATH_INFO'] = e['PATH_INFO'].rstrip('/')
        return self.app(e,h)
    
    app = bottle.app()
    myapp = StripPathMiddleware(app)
    bottle.run(app=myapp)

.. rubric:: 각주

.. [1] 왜냐면 다르기 때문이다. <http://www.ietf.org/rfc/rfc3986.txt> 참고.


킵얼라이브 요청
---------------

.. note::

    자세한 설명은 :doc:`async` 참고.

XHR multipart 같은 여러 "푸시" 메커니즘을 위해선 응답 헤더 "Connection: keep-alive"를 써서 연결을 닫지 않고 응답 데이터를 써 보내는 게 가능해야 한다. WSGI가 그런 동작에 잘 들어맞지는 않지만 보틀에서 gevent_ 비동기 프레임워크를 써서 그렇게 하는 게 가능하긴 하다. 다음은 gevent_ HTTP 서버나 paste_ HTTP 서버에서 동작하는 예시다. (다른 서버에서도 동작할 수 있겠지만 시도해 보진 않았다.) paste_ 서버를 쓰려면 ``server='gevent'``\를 ``server='paste'``\로 바꿔 주기만 하면 된다. ::

    from gevent import monkey; monkey.patch_all()

    import time
    from bottle import route, run
    
    @route('/stream')
    def stream():
        yield 'START'
        time.sleep(3)
        yield 'MIDDLE'
        time.sleep(5)
        yield 'END'
    
    run(host='0.0.0.0', port=8080, server='gevent')

브라우저로 ``http://localhost:8080/stream``\을 열면 'START', 'MIDDLE', 'END'가 (8초 후에 한번에 다 나오는 게 아니라) 하나씩 나오는 걸 볼 수 있다.

보틀에서 Gzip 압축하기
----------------------

.. note::
   자세한 논의는 compression_ 참고.

보틀에 자주 요청되는 기능 하나가 Gzip 압축 지원이다. 요청 처리 시 (CSS나 JS 파일 같은) 정적 자원을 압축해서 사이트 속도를 높여 준다.

Gzip 압축 지원은 여기 저기서 나오는 여러 예외 상황들 때문에 단순한 작업이 아니다. 제대로 된 Gzip 구현은

* 실시간으로 압축해야 하고 그것도 빠르게 해야 한다.
* 브라우저가 지원하지 않으면 압축하지 않아야 한다.
* 이미 압축된 파일(그림, 영상)은 압축하지 않아야 한다.
* 동적 파일을 압축하지 않아야 한다.
* 두 가지 다른 압축 알고리즘(gzip, deflate)을 지원해야 한다.
* 변화가 드문 파일의 압축 결과를 캐싱 해야 한다.
* 한쪽 파일이 어떻게든 바뀌면 캐시를 무효화해야 한다.
* 캐시가 너무 커지지 않도록 해야 한다.
* 실시간으로 압축하는 것보다 디스크 탐색 시간이 더 길 수 있으므로 작은 파일은 캐싱 하지 않아야 한다.

이런 요건들 때문에 보틀 프로젝트에서 권하는 건 보틀이 도는 WSGI 서버에서 Gzip 압축을 다루는 게 가장 좋다는 것이다. cherrypy_ 같은 WSGI 서버에서 이를 위해 쓸 수 있는 GzipFilter_ 미들웨어를 제공한다.


훅 플러그인 쓰기
----------------

예를 들어 모든 URL에서 반환하는 컨텐츠에 대해 교차 출처 리소스
공유(CORS)를 허용하고 싶다면 훅 데코레이터를 써서 콜백 함수를
설치할 수 있다. ::

    from bottle import hook, response, route

    @hook('after_request')
    def enable_cors():
        response.headers['Access-Control-Allow-Origin'] = '*'

    @route('/foo')
    def say_foo():
        return 'foo!'

    @route('/bar')
    def say_bar():
        return {'type': 'friendly', 'content': 'Hi!'}

``before_request``\를 써서 함수가 호출되기 전에 어떤 동작을
취할 수도 있다.


Heroku에서 보틀 쓰기
--------------------

인기 있는 클라우드 응용 플랫폼인 Heroku_\에선 이제 그 인프라
위에서 파이썬 응용을 돌리는 걸 지원한다.

이 방법은 `Heroku Quickstart
<http://devcenter.heroku.com/articles/quickstart>`_\를 바탕으로
해서 `Getting Started with Python on Heroku/Cedar
<http://devcenter.heroku.com/articles/python>`_ 안내서의
`Write Your App <http://devcenter.heroku.com/articles/python#write_your_app>`_
내용을 보틀에 맞는 코드로 바꾼 것이다. ::

    import os
    from bottle import route, run

    @route("/")
    def hello_world():
            return "Hello World!"

    run(host="0.0.0.0", port=int(os.environ.get("PORT", 5000)))

Heroku의 앱 스택에선 `os.environ` 딕셔너리를 이용해서 응용에서
요청을 받아야 하는 포트를 전달한다.
