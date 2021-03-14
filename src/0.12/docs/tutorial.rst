.. module:: bottle

.. _Apache Server:
.. _Apache: http://www.apache.org/
.. _cherrypy: http://www.cherrypy.org/
.. _decorator: http://docs.python.org/glossary.html#term-decorator
.. _flup: http://trac.saddi.com/flup
.. _http_code: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
.. _http_method: http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html
.. _json: http://de.wikipedia.org/wiki/JavaScript_Object_Notation
.. _lighttpd: http://www.lighttpd.net/
.. _mako: http://www.makotemplates.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _Paste: http://pythonpaste.org/
.. _Pound: http://www.apsis.ch/pound/
.. _`WSGI Specification`: http://www.wsgi.org/
.. _issue: http://github.com/defnull/bottle/issues
.. _Python: http://python.org/
.. _SimpleCookie: http://docs.python.org/library/cookie.html#morsel-objects
.. _testing: http://github.com/defnull/bottle/raw/master/bottle.py

========
따라하기
========

여기선 보틀 웹 프레임워크의 개념과 기능을 소개하고 기초 주제와 고급 주제를 함께 다룬다. 처음부터 끝까지 읽어 나가도 되고 나중에 참고 용도로 써도 된다. 자동으로 생성되는 :doc:`api`\도 흥미로울 수 있다. 더 자세한 내용을 다루지만 여기보다 설명이 적다. 흔한 질문들에 대한 해결책은 :doc:`recipes` 모음이나 :doc:`faq` 페이지에서 찾을 수 있다. 혹시 도움이 필요하면 `메일링 리스트 <mailto:bottlepy@googlegroups.com>`_\에 가입하거나 `IRC 채널 <http://webchat.freenode.net/?channels=bottlepy>`_\에 가 보면 된다.

.. _installation:

설치
==============================================================================

보틀은 다른 외부 라이브러리에 의존하지 않는다. `bottle.py </bottle.py>`_\를 프로젝트 디렉터리로 내려받아서 바로 코딩을 시작할 수 있다.

.. code-block:: bash

    $ wget http://bottlepy.org/bottle.py

이렇게 하면 신규 기능이 모두 포함된 최신 개발 스냅샷을 받게 된다. 더 안정적인 환경을 선호한다면 안정 릴리스를 쓰는 게 좋다. `PyPI <http://pypi.python.org/pypi/bottle>`_\에서 받을 수 있으며 :command:`pip` (권장)나 :command:`easy_install`, 또는 패키지 관리자를 통해 설치할 수 있다.

.. code-block:: bash

    $ sudo pip install bottle              # 권장
    $ sudo easy_install bottle             # pip 없는 경우 대안
    $ sudo apt-get install python-bottle   # 데비안, 우분투 등에서

어느 쪽이든 보틀 응용을 돌리려면 파이썬 2.5 또는 그 이상(3.x 포함)이 필요하다. 패키지를 시스템 전역으로 설치할 권한이 없거나 그러고 싶지 않다면 `virtualenv <http://pypi.python.org/pypi/virtualenv>`_\로 가상 환경을 만들자.

.. code-block:: bash

    $ virtualenv develop              # 가상 환경 생성
    $ source develop/bin/activate     # 가상 환경의 파이썬 사용
    (develop)$ pip install -U bottle  # 가상 환경에 보틀 설치

시스템에 virtualenv가 설치돼 있지 않다면:

.. code-block:: bash

    $ wget https://raw.github.com/pypa/virtualenv/master/virtualenv.py
    $ python virtualenv.py develop    # 가상 환경 생성
    $ source develop/bin/activate     # 가상 환경의 파이썬 사용
    (develop)$ pip install -U bottle  # 가상 환경에 보틀 설치



빨리 해 보기: "Hello World"
==============================================================================

보틀이 :ref:`설치돼 있거나 <installation>` 프로젝트 디렉터리로 복사해 뒀다고 가정한다. 아주 기본적인 "Hello World" 예시로 시작해 보자. ::

    from bottle import route, run

    @route('/hello')
    def hello():
        return "Hello World!"

    run(host='localhost', port=8080, debug=True)

이게 전부다. 이 스크립트를 실행하고서 http://localhost:8080/hello 주소를 열면 브라우저에 "Hello World!"가 보이게 된다. 어떻게 동작하는지 살펴보자.

:func:`route` 데코레이터는 URL 경로에 코드 조각을 결속시킨다. 이 경우에는 ``/hello`` 경로를 ``hello()`` 함수로 연결한다. 이걸 (데코레이터 이름 그대로) `라우트`\라고 하는데 이 프레임워크에서 가장 중요한 개념이다. 원하는 만큼 마음대로 라우트를 정의할 수 있다. 브라우저가 URL을 요청하면 연계된 함수가 호출되고 그 반환 값이 브라우저로 간다. 그게 핵심이다.

마지막 행의 :func:`run` 호출은 내장된 개발용 서버를 돌린다. ``localhost`` 포트 ``8080``\에서 돌면서 :kbd:`Control-c`\를 누르기 전까지 요청을 처리한다. 이후 서버 백엔드를 교체할 수 있지만 일단은 개발용 서버만으로 충분하다. 구성 작업이 전혀 필요 없기 때문에 성가신 과정 없이 로컬 테스트를 위해 응용을 올려서 돌려볼 수 있다.

초기 개발 과정에는 :ref:`tutorial-debugging`\가 아주 유용하지만 공개된 응용에서는 꺼 둬야 한다. 잊지 말자.

분명 아주 간단한 예시지만 보틀을 이용해 어떻게 응용을 만드는지 기본적인 개념을 잘 보여 준다. 그럼 또 어떤 것들이 가능한지 살펴보자.

.. _tutorial-default:

기본 응용
------------------------------------------------------------------------------

단순함을 위해 여기 있는 예시 대부분에선 모듈 층위의 :func:`route` 데코레이터를 써서 라우트를 정의한다. 그러면 전역 "기본 응용"에 라우트가 추가되며 :func:`route`\를 처음 호출할 때 :class:`Bottle` 인스턴스가 자동으로 생성된다. 몇 가지 다른 모듈 층위 데코레이터와 함수들도 그 기본 응용과 관련돼 있다. 하지만 객체 지향적인 방식을 선호하고 키보드를 더 두드리는 게 괜찮다면 따로 응용 객체를 만들어서 전역 객체 대신 쓸 수도 있다. ::

    from bottle import Bottle, run

    app = Bottle()

    @app.route('/hello')
    def hello():
        return "Hello World!"

    run(app, host='localhost', port=8080)

:ref:`default-app` 절에서 객체 지향 방식을 더 설명한다. 일단은 선택지가 있다는 것만 기억해 두자.




.. _tutorial-routing:

요청 라우팅
==============================================================================

앞 장에서 라우트가 한 개만 있는 아주 간단한 웹 응용을 만들어 봤다. 다음은 "Hello World" 예시의 라우팅 부분이다. ::

    @route('/hello')
    def hello():
        return "Hello World!"

:func:`route` 데코레이터가 URL 경로를 콜백 함수로 연결하고 :ref:`기본 응용 <tutorial-default>`\에 새 라우트를 추가해 준다. 하지만 라우트가 하나 뿐인 응용은 별로 재미가 없으니 좀 더 추가해 보자. ::

    @route('/')
    @route('/hello/<name>')
    def greet(name='Stranger'):
        return template('Hello {{name}}, how are you?', name=name)

이 예에서 두 가지를 볼 수 있다. 콜백 하나에 라우트를 여러 개 결속시킬 수 있고, URL에 와일드카드를 추가해서 키워드 인자를 통해 접근할 수 있다.



.. _tutorial-dynamic-routes:

동적 라우트
-------------------------------------------------------------------------------

와일드카드를 포함한 라우트를 `동적 라우트`\라고 하며 (반대는 `정적 라우트`), 동시에 여러 URL에 걸린다. 간단한 와일드카드는 이름을 꺾쇠괄호로 감싼 것이며 다음 슬래시(``/``) 전까지 하나 이상의 문자를 받는다. 예를 들어 라우트 ``/hello/<name>``\은 ``/hello/alice``\와 ``/hello/bob`` 요청은 받지만 ``/hello``\나 ``/hello/``, ``/hello/mr/smith`` 요청은 받지 않는다.

URL에서 각 와일드카드가 걸린 부분이 요청 콜백에 키워드 인자로 전달된다. 이를 바로 이용하면 RESTful이고 보기 좋으면서 유의미한 URL을 간단히 구현할 수 있다. 다음은 몇 가지 다른 예시들와 걸리는 URL이다. ::

    @route('/wiki/<pagename>')            # /wiki/Learning_Python 걸림
    def show_wiki_page(pagename):
        ...

    @route('/<action>/<user>')            # /follow/defnull 걸림
    def user_api(action, user):
        ...

.. versionadded:: 0.10

필터를 사용해 와일드카드를 더 구체적으로 정의하고 URL의 걸린 부분이 콜백으로 전달되기 전에 변환을 할 수 있다. 필터 와일드카드는 ``<name:filter>`` 또는 ``<name:filter:config>`` 형식으로 선언한다. 선택적인 config 부분의 문법은 사용 필터에 따라 다르다.

다음 필터들이 기본으로 구현돼 있으며 더 추가할 수도 있다.

* **:int** (부호 있는) 숫자에만 걸리고 값을 정수로 변환한다.
* **:float** :int와 비슷하되 부동소수점수다.
* **:path** 슬래시 문자를 포함한 모든 문자들에 비탐욕 방식으로 걸리며 하나 이상의 경로 분절이 걸리게 하는 데 쓸 수 있다.
* **:re** config 필드에 원하는 정규 표현식을 지정할 수 있다. 걸린 값이 변환되지는 않는다.

현실적인 예시를 몇 가지 살펴보자. ::

    @route('/object/<id:int>')
    def callback(id):
        assert isinstance(id, int)

    @route('/show/<name:re:[a-z]+>')
    def callback(name):
        assert name.isalpha()

    @route('/static/<path:path>')
    def callback(path):
        return static_file(path, ...)

새로운 필터를 추가할 수도 있다. 자세한 건 :doc:`routing` 참고.

.. versionchanged:: 0.10

흔한 사용 방식들을 단순화하기 위해 **보틀 0.10**\에서 새로운 규칙 문법이 도입됐다. 하지만 이전 문법도 여전히 동작하며 많은 코드 예시에서 이전 문법을 쓰는 걸 볼 수 있다. 예시를 보면 차이를 잘 알 수 있다.

=================== ====================
이전 문법           새 문법
=================== ====================
``:name``           ``<name>``
``:name#regexp#``   ``<name:re:regexp>``
``:#regexp#``       ``<:re:regexp>``
``:##``             ``<:re>``
=================== ====================

향후 프로젝트에선 되도록 이전 문법을 피하자. 현재는 폐기 예정이 아니지만 언젠간 폐기될 것이다.


HTTP 요청 메소드
------------------------------------------------------------------------------

.. __: http_method_

HTTP 프로토콜에서는 다양한 작업을 위한 몇 가지 `요청 메소드`__\("동사(verb)"라고도 함)를 규정하고 있다. 따로 메소드를 지정하지 않으면 모든 라우트에 GET이 기본이다. 그 라우트는 GET 요청에만 걸리게 된다. POST나 PUT, DELETE 같은 다른 메소드를 처리하고 싶으면 :func:`route` 데코레이터에 키워드 인자 ``method``\를 추가하거나 :func:`get`, :func:`post`, :func:`put`, :func:`delete` 데코레이터를 대신 쓰면 된다.

HTML 양식 제출에는 POST 메소드를 흔히 쓴다. 다음 예는 POST를 이용한 로그인 양식 처리를 보여 준다. ::

    from bottle import get, post, request # 또는 route

    @get('/login') # 또는 @route('/login')
    def login():
        return '''
            <form action="/login" method="post">
                Username: <input name="username" type="text" />
                Password: <input name="password" type="password" />
                <input value="Login" type="submit" />
            </form>
        '''

    @post('/login') # 또는 @route('/login', method='POST')
    def do_login():
        username = request.forms.get('username')
        password = request.forms.get('password')
        if check_login(username, password):
            return "<p>Your login information was correct.</p>"
        else:
            return "<p>Login failed.</p>"

이 예에선 ``/login`` URL이 두 가지 다른 콜백에 연결돼 있다. GET 요청 콜백은 사용자에게 HTML 양식을 표시한다. POST 요청 콜백은 양식 제출 시 호출되어 사용자가 양식에 입력한 로그인 정보를 검사한다. :ref:`tutorial-request` 절에서 :attr:`Request.forms` 사용법을 더 설명한다.

.. rubric:: 특수 메소드: HEAD와 ANY

HEAD 메소드는 GET 요청 응답과 동일하되 바디는 없는 응답을 요청하는 데 쓴다. 문서 전체를 내려받을 필요 없이 자원에 대한 메타 정보를 얻는 데 유용하다. 보틀에선 그 요청을 받았는데 맞는 라우트가 없으면 자동으로 대응하는 GET 라우트로 간 다음 응답 바디가 있으면 잘라내는 방식으로 처리한다. 즉 HEAD 라우트를 따로 지정해 줄 필요가 없다.

그리고 낮은 우선순위로 동작하는 비표준 ANY 메소드를 쓸 수도 있다. ANY에 대한 라우트는 HTTP 메소드와 상관없이 요청을 잡아채는데, 다른 구체적 라우트가 정의돼 있지 않은 경우에만 그렇게 한다. 요청을 더 구체적인 하위 응용으로 돌리는 *프록시 라우트*\에 유용하다.

요약하면 원래 요청 메소드에 맞는 라우트가 없는 경우에서, HEAD 요청이 GET 라우트로 가고 모든 요청이 ANY 라우트로 간다. 아주 간단하다.



정적 파일 라우팅
------------------------------------------------------------------------------

이미지나 CSS 파일 같은 정적 파일이 자동으로 처리되지 않는다. 라우트와 콜백을 추가해서 어떤 파일을 제공하고 어디서 얻을 수 있는지 지정해 줘야 한다. ::

  from bottle import static_file
  @route('/static/<filename>')
  def server_static(filename):
      return static_file(filename, root='/path/to/your/static/files')

:func:`static_file` 함수는 안전하고 편리하게 파일들을 제공할 수 있는 헬퍼다. (:ref:`tutorial-static-files` 참고.) ``<filename>`` 와일드카드가 슬래시를 포함한 경로에는 걸리지 않기 때문에 이 예는 ``/path/to/your/static/files`` 디렉터리 바로 안에 있는 파일들만 제공할 수 있다. 하위 디렉터리 안의 파일도 제공하려면 와일드카드에 `path` 필터를 쓰면 된다. ::

  @route('/static/<filepath:path>')
  def server_static(filepath):
      return static_file(filepath, root='/path/to/your/static/files')

``root='./static/files'``\처럼 상대적으로 루트 경로를 지정할 때는 조심해야 한다. 작업 디렉터리(``./``)와 프로젝트 디렉터리가 항상 같지는 않다.




.. _tutorial-errorhandling:

오류 페이지
------------------------------------------------------------------------------

뭔가 잘못되면 정보는 있지만 꽤 밋밋한 오류 페이지를 보틀에서 표시한다. :func:`error` 데코레이터로 특정 HTTP 상태 코드에 대한 기본 페이지를 바꿀 수 있다. ::

  from bottle import error
  @error(404)
  def error404(error):
      return 'Nothing here, sorry'

그러면 `404 File Not Found` 오류 발생 시 따로 만든 오류 페이지가 사용자에게 표시된다. 오류 핸들러로 전달되는 유일한 매개변수는 :exc:`HTTPError` 인스턴스다. 그 외에는 오류 핸들러와 일반 요청 콜백이 꽤 비슷하다. :data:`request`\에서 읽을 수 있고 :data:`response`\로 쓸 수 있으며 :exc:`HTTPError` 인스턴스를 빼고 지원하는 어떤 데이터 타입이든 반환할 수 있다.

응용에서 :exc:`HTTPError` 예외를 반환하거나 던진 (:func:`abort`) 경우에만 오류 핸들러가 쓰인다. :attr:`Response.status`\를 바꾸거나 :exc:`HTTPResponse`\를 반환하는 걸로는 오류 핸들러가 실행되지 않는다.






.. _tutorial-output:

내용물 생성하기
==============================================================================

순수한 WSGI에서는 응용에서 반환할 수 있는 타입 종류가 아주 제한돼 있다. 바이트열을 내놓는 이터러블을 응용에서 반환해야 한다. (문자열도 이터러블이니까) 문자열을 반환할 수도 있지만, 그러면 대부분 서버는 내용물을 한 글자씩 전송하게 된다. 유니코드열은 아예 허용되지 않는다. 이래선 실용성이 너무 떨어진다.

보틀은 훨씬 더 유연하게 다양한 타입들을 지원한다. 게다가 가능한 경우 ``Content-Length`` 헤더를 추가해 주기도 하며, 유니코드를 자동으로 인코딩하기에 직접 해 줄 필요가 없다. 다음은 응용 콜백에서 반환할 수 있는 데이터 타입 목록과 프레임워크에서 처리하는 방식에 대한 간단한 설명이다.

딕셔너리
    앞서 언급한 것처럼 파이썬 딕셔너리(와 그 서브클래스)는 자동으로 JSON 문자열로 변환돼서 ``Content-Type`` 헤더를 ``application/json``\으로 하고 브라우저로 반환된다. 그래서 JSON 기반 API를 쉽게 구현할 수 있다. JSON 외 다른 데이터 형식들도 지원한다. 자세한 내용은 :ref:`tutorial-output-filter` 참고.

빈 문자열, ``False``, ``None``, 기타 참 아닌 값:
    빈 출력을 만들고 ``Content-Length`` 헤더를 0으로 설정한다.

유니코드열
    유니코드열(또는 유니코드열을 내놓는 이터러블)은 자동으로 ``Content-Type`` 헤더에 지정된 코덱(기본은 utf8)으로 인코딩돼서 일반 바이트열처럼 (아래 참고) 처리된다.

바이트열
    보틀에선 열을 (글자 하나씩 돌지 않고) 통째로 반환하며 열 길이를 가지고 ``Content-Length`` 헤더를 추가해 준다. 바이트열 리스트는 먼저 하나로 합친다. 바이트열을 내놓는 다른 이터러블은 합치지 않는데, 너무 커져서 메모리에 안 들어갈 수도 있기 때문이다. 그 경우 ``Content-Length`` 헤더를 설정하지 않는다.

:exc:`HTTPError` 또는 :exc:`HTTPResponse` 인스턴스
    이걸 반환하는 건 예외로 던지는 것과 효과가 같다. :exc:`HTTPError`\인 경우 오류 핸들러를 사용한다. 자세한 건 :ref:`tutorial-errorhandling` 참고.

파일 객체
    ``.read()`` 메소드가 있는 모든 걸 파일 내지 파일스러운 객체로 보고 WSGI 서버 프레임워크에 정의돼 있는 콜러블 ``wsgi.file_wrapper``\로 전달한다. 일부 WSGI 서버 구현에선 최적화된 시스템 호출(sendfile)을 이용해 더 효율적으로 파일을 전송할 수 있다. 다른 경우엔 그냥 메모리에 들어가는 크기의 덩어리만큼씩 돈다. ``Content-Length``\나 ``Content-Type`` 같은 선택적인 헤더들이 자동으로 설정되지 *않는다*. 되도록 :func:`send_static`\을 쓰는 게 좋다. 자세한 건 :ref:`tutorial-static-files` 참고.

이터러블과 제너레이터
    콜백 내에서 ``yield``\를 쓰거나 이터러블을 반환할 수도 있다. 단 그 이터러블은 바이트열이나 유니코드열, :exc:`HTTPError`\나 :exc:`HTTPResponse` 인스턴스를 내놓아야 한다. 미안하지만 중첩 이터러블은 안 된다. 이터러블이 비어 있지 않은 값을 내놓는 즉시 HTTP 상태 코드와 헤더들을 브라우저로 보낸다는 점에 유의하자. 이후에 바꿔도 효과가 없다.

이 목록의 순서도 중요하다. 예를 들어 :class:`str`\의 서브클래스를 반환하는데 그 객체에 ``read()`` 메소드도 있을 수 있다. 문자열을 먼저 처리하기 때문에 파일이 아니라 문자열로 처리한다.

.. rubric:: 기본 인코딩 바꾸기

보틀에선 ``Content-Type`` 헤더의 `charset` 매개변수를 이용해 유니코드열을 어떻게 인코딩 할지 정한다. 그 헤더의 기본값은 ``text/html; charset=UTF8``\이고 :attr:`Response.content_type` 속성으로 바꿀 수 있다. 아니면 :attr:`Response.charset` 속성을 바로 설정할 수도 있다. (:class:`Response` 객체는 :ref:`tutorial-response` 절에서 설명한다.)

::

    from bottle import response
    @route('/iso')
    def get_iso():
        response.charset = 'ISO-8859-15'
        return u'This will be sent with ISO-8859-15 encoding.'

    @route('/latin9')
    def get_latin():
        response.content_type = 'text/html; charset=latin9'
        return u'ISO-8859-15 is also known as latin9.'

일부 드문 경우에 파이썬 인코딩 이름이 HTTP 명세에서 지원하는 이름과 다르다. 그때는 먼저 (클라이언트에게 그대로 전송되는) :attr:`Response.content_type` 헤더를 설정한 다음 (유니코드 인코딩에 쓰는) :attr:`Response.charset` 속성을 설정해야 한다.

.. _tutorial-static-files:

정적 파일
--------------------------------------------------------------------------------

파일 객체를 바로 반환할 수도 있지만 정적 파일을 제공하는 권장하는 방식은 :func:`static_file`\이다. 자동으로 마임 타입을 추측하고, ``Last-Modified`` 헤더를 추가해 주며, 보안을 위해 경로를 ``root`` 디렉터리로 제한해 주고, 적절한 오류 응답(권한 오류에는 403, 파일이 없으면 404)을 만들어 준다. ``If-Modified-Since`` 헤더도 지원하며 ``304 Not Modified`` 응답까지 생성할 수 있다. 따로 마임 타입을 줘서 추측 동작을 끌 수도 있다.

::

    from bottle import static_file
    @route('/images/<filename:re:.*\.png>')
    def send_image(filename):
        return static_file(filename, root='/path/to/image/files', mimetype='image/png')

    @route('/static/<filename:path>')
    def send_static(filename):
        return static_file(filename, root='/path/to/static/files')

정말 필요한 경우에는 :func:`static_file`\의 반환 값을 예외로 던질 수도 있다.

.. rubric:: 내려받기 대화창

대부분의 브라우저에서는 마임 타입을 알고 있고 어떤 응용에 할당돼 있으면 (예: PDF 파일) 내려받은 파일을 열려고 한다. 그걸 원하는 게 아니라면 내려받기 대화창을 꼭 표시하게 하고 사용자에게 파일 이름 제안까지 할 수 있다. ::

    @route('/download/<filename:path>')
    def download(filename):
        return static_file(filename, root='/path/to/static/files', download=filename)

``download`` 매개변수가 그냥 ``True``\면 원본 파일 이름을 쓴다.

.. _tutorial-error:

HTTP 오류와 재지향
--------------------------------------------------------------------------------

:func:`abort` 함수를 써서 간단하게 HTTP 오류 페이지를 생성할 수 있다.

::

    from bottle import route, abort
    @route('/restricted')
    def restricted():
        abort(401, "Sorry, access denied.")

클라이언트를 다른 URL로 보내기 위해 ``Location`` 헤더를 새 URL로 설정하고 ``303 See Other`` 응답을 보낼 수 있다. :func:`redirect`\가 그걸 대신 해 준다. ::

    from bottle import redirect
    @route('/wrong/url')
    def wrong():
        redirect("/right/url")

두 번째 매개변수로 다른 HTTP 상태 코드를 줄 수도 있다.

.. note::
    두 함수 모두 :exc:`HTTPError` 예외를 던져서 콜백 코드 실행을 중단시킨다.

.. rubric:: 다른 예외

:exc:`HTTPResponse`\와 :exc:`HTTPError`\를 제외한 다른 예외들은 ``500 Internal Server Error`` 응답을 만들게 되고 그래서 WSGI 서버가 죽지 않는다. 미들웨어의 이런 예외 처리 동작을 끄려면 ``bottle.app().catchall``\을 ``False``\로 설정하면 된다.


.. _tutorial-response:

:class:`Response` 객체
--------------------------------------------------------------------------------

HTTP 상태 코드, 응답 헤더, 쿠키 등의 응답 메타데이터가 :data:`response`\라는 객체에 저장됐다가 브라우저로 전송된다. 그 메타데이터를 직접 조작할 수도 있고 미리 정의된 헬퍼 메소드를 이용할 수도 있다. 전체 API와 기능들이 API 절에 설명돼 있지만 (:class:`Response` 참고) 여기서도 주요 사용법과 기능들을 살펴본다.

.. rubric:: 상태 코드

`HTTP 상태 코드 <http_code>`_\는 브라우저의 동작을 제어하며 기본은 ``200 OK``\다. 대부분 상황에선 :attr:`Response.status` 속성을 직접 설정할 필요가 없고 :func:`abort` 헬퍼를 쓰거나 :exc:`HTTPResponse` 인스턴스를 반환하면서 적당한 상태 코드를 주면 된다. 어떤 정수든 허용되지만 `HTTP 명세 <http_code>`_\에서 규정한 것 외의 코드는 브라우저를 혼란시킬 뿐이며 표준을 깨는 것이다.

.. rubric:: 응답 헤더

:meth:`Response.set_header`\를 통해 ``Cache-Control``\이나 ``Location`` 같은 응답 헤더들을 지정한다. 이 메소드는 매개변수를 두 개 받는데, 각각 헤더 이름과 값이다. 이름에서 대소문자를 구별하지 않는다. ::

  @route('/wiki/<page>')
  def wiki(page):
      response.set_header('Content-Language', 'en')
      ...

대부분 헤더는 유일하다. 즉 이름별로 한 헤더만 클라이언트에게 보낸다. 하지만 일부 특수 헤더는 응답에 여러 번 등장할 수 있다. 그런 헤더를 추가하려면 :meth:`Response.set_header` 대신 :meth:`Response.add_header`\를 쓰면 된다. ::

    response.set_header('Set-Cookie', 'name=value')
    response.add_header('Set-Cookie', 'name2=value2')

이는 예시일 뿐이다. 쿠키 처리를 하고 싶다면 :ref:`다음 절 <tutorial-cookies>`\을 읽어 보자.


.. _tutorial-cookies:

쿠키
-------------------------------------------------------------------------------

쿠키란 사용자의 브라우저 프로파일에 저장되는 이름 있는 텍스트 조각이다. :meth:`Request.get_cookie`\를 통해 앞서 정의된 쿠키에 접근할 수 있고 :meth:`Response.set_cookie`\로 새 쿠키를 설정할 수 있다. ::

    @route('/hello')
    def hello_again():
        if request.get_cookie("visited"):
            return "Welcome back! Nice to see you again"
        else:
            response.set_cookie("visited", "yes")
            return "Hello there! Nice to meet you"

:meth:`Response.set_cookie` 메소드는 쿠키 수명과 동작을 제어하는 여러 추가 키워드 인자들을 받는다. 자주 쓰는 설정은 다음과 같다.

* **max_age:**    초 단위 최대 수명. (기본값: ``None``)
* **expires:**    datetime 객체 또는 유닉스 타임스탬프. (기본값: ``None``)
* **domain:**     쿠키를 읽는 게 허용되는 도메인. (기본값: 현재 도메인)
* **path:**       쿠키를 지정된 경로로 제한. (기본값: ``/``)
* **secure:**     쿠키를 HTTPS 연결로 제한. (기본값: 꺼짐)
* **httponly:**   클라이언트 측 자바스크립트에서 이 쿠키를 읽지 못하게 하기. (기본값: 꺼짐. 파이썬 2.6 이상 필요.)

`expires`\와 `max_age` 어느 쪽도 설정하지 않으면 브라우저 세션이 끝날 때, 즉 브라우저 창이 닫힐 때 쿠키가 만료된다. 쿠키 사용 시 고려해야 할 사항들이 몇 가지 있다.

* 대부분 브라우저에서 쿠키가 4 KB 텍스트로 제한된다.
* 일부 사용자들은 쿠키를 전혀 허용하지 않도록 브라우저를 설정한다. 검색 엔진에서도 대부분 쿠키를 무시한다. 쿠키 없이도 응용이 동작하게 해야 한다.
* 쿠키는 클라이언트에 저장되며 어떤 식으로도 암호화되지 않는다. 즉 쿠키에 저장한 모든 걸 사용자가 읽을 수 있다. 게다가 공격자가 서버의 `XSS <http://en.wikipedia.org/wiki/HTTP_cookie#Cookie_theft_and_session_hijacking>`_ 취약성을 통해 사용자 쿠키를 훔치는 게 가능할 수도 있고 어떤 바이러스는 브라우저 쿠키를 읽어 간다고 한다. 즉 절대 비밀 정보를 쿠키에 저장해선 안 된다.
* 악의적인 공격자가 손쉽게 쿠키를 위조할 수 있다. 쿠키를 신뢰해선 안 된다.

.. _tutorial-signed-cookies:

.. rubric:: 쿠키 서명

앞서 언급한 것처럼 악의적 클라이언트가 손쉽게 쿠키를 위조할 수 있다. 이런 조작을 막기 위해 쿠키에 암호학적 서명을 할 수 있다. 쿠키를 읽거나 설정할 때마다 `secret` 키워드 인자를 통해 서명 키를 제공하고 그 키를 비밀로 유지하기만 하면 된다. 그렇게 하면 쿠키에 서명이 돼 있지 않거나 서명이 일치하지 않는 경우 :meth:`Request.get_cookie`\가 ``None``\을 반환하게 된다. ::

    @route('/login')
    def do_login():
        username = request.forms.get('username')
        password = request.forms.get('password')
        if check_login(username, password):
            response.set_cookie("account", username, secret='some-secret-key')
            return template("<p>Welcome {{name}}! You are now logged in.</p>", name=username)
        else:
            return "<p>Login failed.</p>"

    @route('/restricted')
    def restricted_area():
        username = request.get_cookie("account", secret='some-secret-key')
        if username:
            return template("Hello {{name}}. Welcome back.", name=username)
        else:
            return "You are not logged in. Access denied."

이에 더해 보틀에선 서명된 쿠키에 저장된 데이터를 자동으로 pickle 및 unpickle 해 준다. 즉 pickle 한 데이터가 4 KB 제한을 넘지 않는 한 (문자열만이 아니라) pickle 가능한 어떤 객체든 쿠키에 저장할 수 있다.

.. warning:: 서명된 쿠키가 암호화되는 건 아니다. (클라이언트가 여전히 내용물을 볼 수 있다.) 그리고 복사 방지가 되는 것도 않다. (클라이언트가 이전 쿠키를 재사용할 수 있다.) 주된 의도는 pickle 처리를 안전하게 만들고 조작을 막는 것이지, 클라이언트 쪽에 비밀 정보를 저장하는 게 아니다.









.. _tutorial-request:

요청 데이터
==============================================================================

전역 :data:`request` 객체를 통해 쿠키, HTTP 헤더, HTML ``<form>`` 필드, 기타 요청 데이터를 이용할 수 있다. 이 특수 객체는 여러 클라이언트 연결을 동시에 처리하는 다중 스레드 환경에서도  항상 *현재* 요청을 가리킨다. ::

  from bottle import request, route, template
  
  @route('/hello')
  def hello():
      name = request.cookies.username or 'Guest'
      return template('Hello {{name}}', name=name)

:data:`request` 객체는 :class:`BaseRequest`\의 서브클래스이며 데이터 접근을 위한 풍부한 API가 있다. 여기선 자주 쓰는 것들만 다루는데, 처음엔 이걸로 충분할 것이다.



:class:`FormsDict` 소개
--------------------------------------------------------------------------------

보틀에선 특별한 딕셔너리를 이용해 양식 데이터와 쿠키를 저장한다. :class:`FormsDict`\는 보통 딕셔너리처럼 동작하면서 거기 더해 개발을 편하게 해 주는 기능이 몇 가지 있다.

**속성 접근:** 딕셔너리의 모든 값들을 속성으로도 접근할 수 있다. 그 가상 속성들은 유니코드열을 반환하는데, 값이 없거나 유니코드 디코딩이 실패한 경우도 마찬가지다. 그 경우 문자열이 비어 있지만 존재는 한다. ::

  name = request.cookies.name
  
  # 풀어 쓰면 다음과 같고:
  
  name = request.cookies.getunicode('name') # encoding='utf-8' (기본)

  # 기본적으로 다음을 하는 것이다:

  try:
      name = request.cookies.get('name', '').decode('utf-8')
  except UnicodeError:
      name = u''

**키별로 여러 값:**: :class:`FormsDict`\가 :class:`MultiDict`\의 서브클래스이므로 키별로 여러 값을 저장할 수 있다. 표준 딕셔너리 접근 메소드는 값 한 개만 반환하지만 :meth:`~MultiDict.getall` 메소드는 해당 키에 대한 모든 값들의 (비어 있을 수도 있는) 리스트를 반환한다. ::

  for choice in request.forms.getall('multiple_choice'):
      do_something(choice)

**WTForms 지원:** 어떤 라이브러리(예: `WTForms <http://wtforms.simplecodes.com/>`_)는 입력으로 유니코드로만 된 딕셔너리를 원한다. :meth:`FormsDict.decode`\가 일을 대신 해 준다. 즉 모든 값들을 디코딩해서 각각의 사본들을 반환하면서 키별 여러 값 및 기타 모든 특성을 유지한다.

.. note::

    **파이썬 2**\에선 키와 값이 모두 바이트열이다. 유니코드가 필요하면 :meth:`FormsDict.getunicode`\를 호출하거나 속성 접근을 통해 값을 얻을 수 있다. 두 방법 모두 문자열 디코딩(기본값: utf8)을 시도하며 실패 시 빈 문자열을 반환한다. :exc:`UnicodeError`\를 잡을 필요가 없다. ::

      >>> request.query['city']
      'G\xc3\xb6ttingen'  # utf8 바이트열
      >>> request.query.city
      u'Göttingen'        # 같은 내용의 유니코드열

    **파이썬 3**\에선 모든 문자열이 유니코드지만 HTTP는 바이트 기반 프로토콜이다. 응용으로 전달하기 전에 서버에서 어떻게든 바이트열을 디코딩해야 한다. WSGI에선 ISO-8859-1(즉 latin1)을 제안하는데, 이후 다른 인코딩으로 재인코딩할 수도 있는 가역적 단일 바이트 코덱이다. 보틀에선 :meth:`FormsDict.getunicode`\와 속성 접근 때는 재인코딩을 하지만 딕셔너리 접근 메소드에서는 하지 않는다. 그 경우엔 서버 구현에서 제공한 값을 그대로 반환하는데, 아마 원하는 값이 아닐 것이다.

      >>> request.query['city']
      'GÃ¶ttingen' # 서버가 임시로 ISO-8859-1로 디코딩한 utf8 문자열
      >>> request.query.city
      'Göttingen'  # 보틀이 utf8로 제대로 재인코딩한 동일 문자열

    값들이 올바로 디코딩된 딕셔너리 전체가 필요한 경우 (가령 WTForms에 사용) :meth:`FormsDict.decode`\를 호출해서 재인코딩된 사본을 얻을 수 있다.


쿠키
--------------------------------------------------------------------------------

쿠키는 클라이언트 브라우저에 저장되는 작은 텍스트 조각이며 요청마다 서버로 다시 전송된다. 단일 요청을 넘어 어떤 상태를 유지하는 데 유용하지만 (HTTP 자체는 무상태 프로토콜이다.) 보안 관련 문제에는 쓰지 않는 게 좋다. 클라이언트에서 손쉽게 위조할 수 있다.

:attr:`BaseRequest.cookies`\(:class:`FormsDict` 인스턴스)를 통해 클라이언트가 보낸 모든 쿠키를 이용할 수 있다. 다음 예는 간단한 쿠키 기반 조회수 카운터다. ::

  from bottle import route, request, response
  @route('/counter')
  def counter():
      count = int( request.cookies.get('counter', '0') )
      count += 1
      response.set_cookie('counter', str(count))
      return 'You visited this page %d times' % count

:meth:`BaseRequest.get_cookie` 메소드는 쿠키에 접근하는 또 다른 방법이다. 다른 절에서 설명한 것처럼 :ref:`서명된 쿠키 <tutorial-signed-cookies>` 디코딩을 지원한다.

HTTP 헤더
--------------------------------------------------------------------------------

클라이언트가 보낸 모든 HTTP 헤더(예: ``Referer``, ``Agent``, ``Accept-Language``)가 :class:`WSGIHeaderDict`\에 저장되어 :attr:`BaseRequest.headers` 속성을 통해 접근할 수 있다. :class:`WSGIHeaderDict`\는 기본적으로 키에 대소문자 구별이 없는 딕셔너리다. ::

  from bottle import route, request
  @route('/is_ajax')
  def is_ajax():
      if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
          return 'This is an AJAX request'
      else:
          return 'This is a normal request'


질의 변수
--------------------------------------------------------------------------------

서버로 키/값 짝 몇 개를 보내는 데 (``/forum?id=1&page=5`` 같은) 질의 문자열을 쓰는 경우가 많다. :attr:`BaseRequest.query` 속성(:class:`FormsDict` 인스턴스)으로 그 값들에 접근할 수 있고 :attr:`BaseRequest.query_string` 속성으로 문자열 전체를 얻을 수 있다.

::

  from bottle import route, request, response, template
  @route('/forum')
  def display_forum():
      forum_id = request.query.id
      page = request.query.page or '1'
      return template('Forum ID: {{id}} (page {{page}})', id=forum_id, page=page)


HTML `<form>` 처리
------------------

기본부터 시작해 보자. HTML에서 ``<form>``\은 보통 다음처럼 생겼다.

.. code-block:: html

    <form action="/login" method="post">
        Username: <input name="username" type="text" />
        Password: <input name="password" type="password" />
        <input value="Login" type="submit" />
    </form>

``action`` 속성이 양식 데이터를 받을 URL을 나타낸다. ``method``\는 사용할 HTTP 메소드(``GET`` 또는 ``POST``)를 지정한다. ``method="get"``\이면 양식 값들이 URL에 덧붙고 앞서 설명한 것처럼 :attr:`BaseRequest.query`\를 통해 이용할 수 있다. 하지만 이 방식은 안전하지 않은 데다 다른 제약들이 있고, 그래서 여기선 ``method="post"``\를 쓴다. 긴가민가하면 ``POST``\를 쓰면 된다.

``POST``\를 통해 전송된 필드는 :class:`FormsDict`\인 :attr:`BaseRequest.forms`\에 저장된다. 서버 쪽 코드에서 다음처럼 값을 볼 수 있다. ::

    from bottle import route, request

    @route('/login')
    def login():
        return '''
            <form action="/login" method="post">
                Username: <input name="username" type="text" />
                Password: <input name="password" type="password" />
                <input value="Login" type="submit" />
            </form>
        '''

    @route('/login', method='POST')
    def do_login():
        username = request.forms.get('username')
        password = request.forms.get('password')
        if check_login(username, password):
            return "<p>Your login information was correct.</p>"
        else:
            return "<p>Login failed.</p>"

양식 데이터에 접근하는 데 쓰는 속성들이 여러 가지 있다. 일부는 편리한 접근을 위해 여러 출처의 값들을 합친다. 요약하면 다음 표와 같다.

===========================   ===============   ================   =============
속성                          GET 양식 필드     POST 양식 필드     파일 업로드
===========================   ===============   ================   =============
:attr:`BaseRequest.query`     yes               no                 no
:attr:`BaseRequest.forms`     no                yes                no
:attr:`BaseRequest.files`     no                no                 yes
:attr:`BaseRequest.params`    yes               yes                no
:attr:`BaseRequest.GET`       yes               no                 no
:attr:`BaseRequest.POST`      no                yes                yes
===========================   ===============   ================   =============


파일 업로드
-----------

파일 업로드를 지원하려면 ``<form>`` 태그를 살짝 바꿔야 한다. 먼저 ``<form>`` 태그에 ``enctype="multipart/form-data"`` 속성을 추가해서 브라우저가 양식 데이터를 다른 방식으로 인코딩하게 한다. 다음으로 ``<input type="file" />`` 태그를 추가해서 사용자가 파일을 선택할 수 있게 한다. 예는 다음과 같다.

.. code-block:: html

    <form action="/upload" method="post" enctype="multipart/form-data">
      Category:      <input type="text" name="category" />
      Select a file: <input type="file" name="upload" />
      <input type="submit" value="Start upload" />
    </form>

업로드된 파일을 보틀에서 몇 가지 메타데이터와 함께 :attr:`BaseRequest.files`\에 :class:`FileUpload` 인스턴스로 저장한다. 그 파일을 그냥 디스크에 저장하고 싶다고 해 보자. ::

    @route('/upload', method='POST')
    def do_upload():
        category   = request.forms.get('category')
        upload     = request.files.get('upload')
        name, ext = os.path.splitext(upload.filename)
        if ext not in ('.png','.jpg','.jpeg'):
            return 'File extension not allowed.'

        save_path = get_save_path_for_category(category)
        upload.save(save_path) # upload.filename을 자동으로 덧붙임
        return 'OK'

:attr:`FileUpload.filename`\은 클라이언트 파일 시스템 상의 파일 이름을 담는데, 파일명 안의 지원 안 되는 문자 내지 경로 분절로 인한 버그를 막기 위해 정리 및 정규화가 돼 있다. 클라이언트가 보낸 변경 안 된 이름이 필요하다면 :attr:`FileUpload.raw_filename`\을 보면 된다.

파일을 디스크에 저장하려는 거라면 :attr:`FileUpload.save` 메소드를 적극 권한다. 몇 가지 흔한 오류들을 방지해 주며 (가령 따로 명시하지 않으면 기존 파일을 덮어 쓰지 않는다.) 메모리에서 파일을 효율적인 방식으로 저장한다. :attr:`FileUpload.file`\을 통해 파일 객체에 바로 접근할 수 있다. 단 조심하자.


JSON 내용물
-----------

일부 자바스크립트나 REST 클라이언트는 서버로 ``application/json`` 내용물을 보낸다. 파싱 가능한 경우 :attr:`BaseRequest.json` 속성에 파싱된 자료 구조가 들어 있다.


비가공 요청 바디
----------------

:attr:`BaseRequest.body`\를 통해 가공 안 된 바디 데이터를 파일스러운 객체로 접근할 수 있다. 내용물 길이 및 :attr:`BaseRequest.MEMFILE_MAX` 설정에 따라서 :class:`BytesIO` 버퍼거나 임시 파일이다. 두 경우 모두 바디가 버퍼에 완전히 저장된 다음에 그 속성에 접근할 수 있다. 데이터가 거대할 것으로 예상되는데 스트림에 버퍼 없이 바로 접근하고 싶다면 ``request['wsgi.input']``\을 살펴보라.



WSGI 환경
--------------------------------------------------------------------------------

각 :class:`BaseRequest` 인스턴스는 WSGI 환경 딕셔너리를 감싸고 있다. 원본은 :attr:`BaseRequest.environ`\에 저장되지만 요청 객체 자체가 딕셔너리처럼 동작하기도 한다. 많이 쓰는 데이터 대부분이 특수한 메소드나 속성을 통해 노출돼 있지만 `WSGI 환경 변수 <WSGI specification>`_\에 직접 접근하고 싶다면 그럴 수도 있다. ::

  @route('/my_ip')
  def show_ip():
      ip = request.environ.get('REMOTE_ADDR')
      # 또는 ip = request.get('REMOTE_ADDR')
      # 또는 ip = request['REMOTE_ADDR']
      return template("Your IP is: {{ip}}", ip=ip)







.. _tutorial-templates:

템플릿
================================================================================

보틀에는 :doc:`stpl`\이라는 빠르고 강력한 내장 템플릿 엔진이 딸려 있다. :func:`template` 함수나 :func:`view` 데코레이터를 써서 템플릿을 렌더링할 수 있다. 템플릿 이름을 적고 템플릿에 전달하고 싶은 변수들을 키워드 인자로 주기만 하면 된다. 다음은 간단한 템플릿 렌더링 예시다. ::

    @route('/hello')
    @route('/hello/<name>')
    def hello(name='World'):
        return template('hello_template', name=name)

템플릿 파일 ``hello_template.tpl``\을 읽어 들여서 ``name`` 변수를 설정해서 렌더링한다. ``./views/`` 폴더 또는 ``bottle.TEMPLATE_PATH`` 목록에 지정된 모든 폴더에서 템플릿을 찾게 된다.

:func:`view` 데코레이터를 쓰면 :func:`template`\을 호출하는 게 아니라 템플릿 변수들을 담은 딕셔너리를 반환하는 방식이 가능하다. ::

    @route('/hello')
    @route('/hello/<name>')
    @view('hello_template')
    def hello(name='World'):
        return dict(name=name)

.. rubric:: 문법

.. highlight:: html+django

템플릿 문법은 파이썬 언어 위에 씌운 아주 얇은 층이다. 주된 목적은 블록 들여쓰기가 올바로 이뤄지게 해서 들여쓰기에 신경쓰지 않고 템플릿을 작성할 수 있게 하는 것이다. 전체 문법 설명은 :doc:`stpl` 참고.

다음은 템플릿 예시다. ::

    %if name == 'World':
        <h1>Hello {{name}}!</h1>
        <p>This is a test.</p>
    %else:
        <h1>Hello {{name.title()}}!</h1>
        <p>How are you?</p>
    %end

.. rubric:: 캐싱

템플릿은 컴파일 후 메모리 내에 캐싱된다. 템플릿 파일에 변경을 해도 템플릿 캐시를 비우기 전에는 효과가 없게 된다. 비우려면 ``bottle.TEMPLATES.clear()``\를 호출하면 된다. 디버그 모드에서는 캐싱이 꺼진다.

.. highlight:: python




.. _plugins:

플러그인
================================================================================

.. versionadded:: 0.9

보틀의 핵심 기능들이 흔한 사용 방식을 대부분 포괄하지만 마이크로 프레임워크라서 한계가 있다. 이때 "플러그인"이 역할을 할 수 있다. 플러그인은 프레임워크에 빠진 기능성을 더하거나, 서드파티 라이브러리를 연결하거나, 어떤 반복 작업을 자동화해 준다.

:doc:`/plugins/index`\이 점점 늘어나고 있으며 대다수 플러그인들이 응용들 간에 이식성 있고 재사용 가능하도록 설계돼 있다. 뭔가 해결할 문제가 있다면 이미 해결돼서 바로 쓸 수 있는 플러그인이 존재할 가능성이 높다. 아니라면 :doc:`/plugindev`\가 도움이 될 수 있다.

플러그인의 효과와 API는 다종다양하고 플러그인에 따라 다르다. 예를 들어 ``SQLitePlugin`` 플러그인은 ``db`` 키워드 인자가 있는 콜백들을 탐지해서 콜백이 호출될 때마다 새로운 데이터베이스 연결 객체를 만들어 준다. 그래서 데이터베이스 사용이 아주 편리해진다. ::

    from bottle import route, install, template
    from bottle_sqlite import SQLitePlugin

    install(SQLitePlugin(dbfile='/tmp/test.db'))

    @route('/show/<post_id:int>')
    def show(db, post_id):
        c = db.execute('SELECT title, content FROM posts WHERE id = ?', (post_id,))
        row = c.fetchone()
        return template('show_post', title=row['title'], text=row['content'])

    @route('/contact')
    def contact_page():
        ''' 이 콜백에선 db 연결이 필요치 않다. 'db' 키워드 인자가
            없으므로 sqlite 플러그인에서 이 콜백을 완전히 무시한다. '''
        return template('contact')

다른 플러그인에서는 스레드에 안전한 :data:`local` 객체를 만들어 주거나, :data:`request` 객체의 세부 내용을 바꾸거나, 콜백이 반환한 데이터를 필터링하거나, 콜백을 완전히 건너뛸 수도 있다. 예를 들어 "auth" 플러그인에서 유효한 세션인지 검사해서 원래 콜백을 호출하지 않고 로그인 페이지를 반환할 수도 있다. 정확히 어떤 일이 일어나는지는 플러그인에 달려 있다.


응용 전체 설치
--------------------------------------------------------------------------------

플러그인을 응용 전체에 설치할 수도 있고 추가 기능이 필요한 특정 라우트에만 설치할 수도 있다. 대부분의 플러그인은 모든 라우트에 설치해도 안전하며 그 기능이 필요치 않은 콜백에 오버헤드를 더하지 않을 만큼 똑똑하다.

``SQLitePlugin`` 플러그인을 예로 들어 보자. 이 플러그인은 데이터베이스 연결이 필요한 라우트 콜백에만 영향을 준다. 다른 라우트는 그대로 둔다. 그래서 추가 오버헤드 없이 응용 전체에 플러그인을 설치할 수 있다.

플러그인을 설치하려면 플러그인을 첫 인자로 해서 :func:`install`\을 호출하면 된다. ::

    from bottle_sqlite import SQLitePlugin
    install(SQLitePlugin(dbfile='/tmp/test.db'))

아직 라우트 콜백에 플러그인이 적용돼 있지 않다. 놓치는 라우트가 없도록 하기 위해 적용을 늦춘다. 그래서 원한다면 플러그인들을 먼저 설치하고 나서 라우트를 추가할 수 있다. 그런데 설치되는 플러그인들의 순서는 중요하다. 어떤 플러그인에서 데이터베이스 연결이 필요하다면 데이터베이스 플러그인을 먼저 설치해야 한다.


.. rubric:: 플러그인 제거

이름이나 클래스, 인스턴스를 써서 앞서 설치한 플러그인을 :func:`uninstall` 할 수 있다. ::

    sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')
    install(sqlite_plugin)

    uninstall(sqlite_plugin) # 특정 플러그인 제거
    uninstall(SQLitePlugin)  # 그 타입의 모든 플러그인 제거
    uninstall('sqlite')      # 그 이름의 모든 플러그인 제거
    uninstall(True)          # 모든 플러그인 한꺼번에 제거

언제든 플러그인을 설치하고 제거할 수 있다. 요청을 처리 중인 런타임에도 가능하다. 이를 이용하면 멋진 기술(느린 디버깅 내지 프로파일링 플러그인을 필요할 때만 설치하기)이 가능하긴 하지만 남용하진 말자. 플러그인 목록이 바뀔 때마다 라우트 캐시가 비워지고 모든 플러그인들이 재적용된다.

.. note::
    모듈 층위의 :func:`install` 및 :func:`uninstall` 함수는 :ref:`default-app`\에 효력이 있다. 특정 응용의 플러그인을 관리하려면 :class:`Bottle` 응용 객체의 대응 메소드를 쓰면 된다.


라우트 한정 설치
--------------------------------------------------------------------------------

일부 라우트에만 플러그인을 설치하고 싶다면 :func:`route` 데코레이터의 ``apply`` 매개변수를 쓸 수 있다. ::

    sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')

    @route('/create', apply=[sqlite_plugin])
    def create(db):
        db.execute('INSERT INTO ...')


플러그인 차단 목록
--------------------------------------------------------------------------------

일부 라우트에서 어떤 플러그인을 명시적으로 비활성화하고 싶을 수도 있다. 그런 용도로 :func:`route` 데코레이터에 ``skip`` 매개변수가 있다. ::

    sqlite_plugin = SQLitePlugin(dbfile='/tmp/test1.db')
    install(sqlite_plugin)

    dbfile1 = '/tmp/test1.db'
    dbfile2 = '/tmp/test2.db'

    @route('/open/<db>', skip=[sqlite_plugin])
    def open_db(db):
        # 이때는 플러그인이 'db' 키워드 인자를 건드리지 않는다.

        # 플러그인 핸들을 써서 런타임 설정을 할 수도 있다.
        if db == 'test1':
            sqlite_plugin.dbfile = dbfile1
        elif db == 'test2':
            sqlite_plugin.dbfile = dbfile2
        else:
            abort(404, "No such database.")

        return "Database File switched to: " + sqlite_plugin.dbfile

``skip`` 매개변수는 단일 값 또는 값들의 리스트를 받는다. 이름, 클래스, 인스턴스를 사용해 건너뛸 플러그인을 지정할 수 있다. 모든 플러그인을 한꺼번에 건너뛰려면 ``skip=True``\로 설정하면 된다.

플러그인과 하위 응용
--------------------------------------------------------------------------------

대부분의 플러그인은 설치된 그 응용에서만 쓰기 위한 것이다. 따라서 :meth:`Bottle.mount`\로 붙인 하위 응용에는 영향을 주지 않아야 한다. 다음 예를 보자. ::

    root = Bottle()
    root.mount('/blog', apps.blog)

    @root.route('/contact', template='contact')
    def contact():
        return {'email': 'contact@example.com'}

    root.install(plugins.WTForms())

응용을 마운트할 때마다 보틀에선 모든 요청을 하위 응용으로 전달해 주는 프록시 라우트를 주 응용에 만든다. 그런 프록시 라우트에 대해선 기본적으로 플러그인이 비활성화된다. 그래서 (가상의) `WTForms` 플러그인은 ``/contact`` 라우트에는 효과가 있지만 ``/blog`` 하위 응용의 라우트에는 영향을 주지 않는다.

이 적당한 기본 동작 방식을 바꿀 수도 있다. 다음 예는 특정 프록시 라우트에서 모든 플러그인을 재활성화한다. ::

    root.mount('/blog', apps.blog, skip=None)

하지만 조심할 게 있다. 플러그인에겐 하위 응용 전체가 위의 프록시 라우트 하나로 보이게 된다. 하위 응용의 개별 라우트에 영향을 주려면 마운트된 응용에서 따로 플러그인을 설치해야 한다.



개발
================================================================================

기초를 배웠으니 이제 응용을 작성하고 싶을 것이다. 아래 내용은 더 생산적으로
작업하는 데 도움이 될 수 있는 몇 가지 팁이다.

.. _default-app:

기본 응용
--------------------------------------------------------------------------------

보틀에선 전역으로 :class:`Bottle` 인스턴스들의 스택을 유지하고 최상단 인스턴스를 몇 가지 모듈 층위 함수와 데코레이터의 기본 인스턴스로 쓴다. 예를 들어 :func:`route` 데코레이터는 기본 응용에 대한 :meth:`Bottle.route` 호출을 간략히 줄인 것이다. ::

    @route('/')
    def hello():
        return 'Hello World'

작은 응용에서 아주 편리하고 키보드 입력을 좀 줄여 준다. 하지만 작성한 모듈을 임포트하면 전역 응용에 바로 라우트가 설치된다는 뜻이기도 하다. 그런 류의 임포트 부작용을 피하기 위해 보틀에선 응용을 구성하는 더 명시적인 방법을 제공한다. ::

    app = Bottle()

    @app.route('/')
    def hello():
        return 'Hello World'

응용 객체를 분리하면 재사용성이 크게 향상되기도 한다. 다른 개발자가 이 모듈의 ``app`` 객체를 안전하게 임포트해서 :meth:`Bottle.mount`\를 이용해 응용들을 합칠 수 있다.

아니면 응용 스택을 직접 사용해서 간편하게 작성하면서도 라우트들을 격리하는 것도 가능하다. ::

    default_app.push()

    @route('/')
    def hello():
        return 'Hello World'

    app = default_app.pop()

:func:`app`\과 :func:`default_app` 모두 :class:`AppStack` 인스턴스이고 스택스러운 API를 구현하고 있다. 필요한 대로 그 스택에 응용을 밀어 넣거나 꺼낼 수 있다. 별도의 응용 객체를 제공하지 않는 서드파티 모듈을 임포트하려 할 때도 도움이 된다. ::

    default_app.push()

    import some.module

    app = default_app.pop()


.. _tutorial-debugging:


디버그 모드
--------------------------------------------------------------------------------

개발 초기에는 디버그 모드가 큰 도움이 될 수 있다.

.. highlight:: python

::

    bottle.debug(True)

이 모드에선 보틀에서 찍는 내용이 더 많아지고 오류가 발생했을 때 도움 되는 디버깅 정보를 제공한다. 방해가 될 수도 있는 몇몇 최적화를 끄고 설정 오류 가능성을 검사해서 경고하기도 한다.

다음은 디버그 모드에서 바뀌는 점들을 몇 가지 나열한 것이다.

* 기본 오류 페이지에서 트레이스백을 보여 준다.
* 템플릿을 캐싱하지 않는다.
* 플러그인이 즉시 적용된다.

운용 서버에서는 디버그 모드를 쓰지 않도록 하자.

자동 재적재
--------------------------------------------------------------------------------

개발 중에는 최근 바꾼 내용을 확인하기 위해 서버를 여러 번 재시작해야
한다. 자동 재적재 기능이 일을 덜어 줄 수 있다. 모듈 파일을 편집할
때마다 재적재 기능이 서버 프로세스를 재시작하고 코드 최신 버전을
적재한다.

::

    from bottle import run
    run(reloader=True)

어떻게 동작하는 걸까. 주 프로세스에서는 서버를 시작하는 게 아니라
주 프로세스 시작에 쓴 것과 같은 명령행 인자로 자식 프로세스를 새로
만든다. 모든 모듈 층위 코드가 최소 두 번 실행된다! 조심하자.

자식 프로세스는 ``os.environ['BOTTLE_CHILD']``\를 ``True``\로 해서
재적재 동작 없는 일반 앱 서버로 시작한다. 적재된 모듈 어디에라도
변경이 발생하면 주 프로세스가 자식 프로세스를 끝내고 다시 만든다.
템플릿 파일 변경은 재적재를 일으키지 않는다. 템플릿 캐싱을
비활성화하고 싶으면 디버그 모드를 쓰면 된다.

재적재를 위해선 자식 프로세스를 중단시킬 수 있어야 한다. 윈도우나 기타
(파이썬에서 ``KeyboardInterrupt``\를 일으키는) ``signal.SIGINT``\를
지원하지 않는 운영 체제에서 돌고 있으면 ``signal.SIGTERM``\으로
자식을 죽인다. 참고로 ``SIGTERM`` 수신 후에는 종료 핸들러와 finally
구문 등이 실행되지 않는다.


명령행 인터페이스
--------------------------------------------------------------------------------

.. versionadded: 0.10

버전 0.10부터 보틀을 명령행 도구처럼 쓸 수 있다.

.. code-block:: console

    $ python -m bottle

    Usage: bottle.py [options] package.module:app

    Options:
      -h, --help            show this help message and exit
      --version             show version number.
      -b ADDRESS, --bind=ADDRESS
                            bind socket to ADDRESS.
      -s SERVER, --server=SERVER
                            use SERVER as backend.
      -p PLUGIN, --plugin=PLUGIN
                            install additional plugin/s.
      --debug               start server in debug mode.
      --reload              auto-reload on file changes.

`ADDRESS` 필드는 IP 주소나 IP:PORT 짝을 받으며 기본은 ``localhost:8080``\이다. 다른 매개변수들은 따로 설명이 필요 없을 것이다.

플러그인과 응용 모두 임포트 식을 통해 지정한다. 임포트 경로(예: ``package.module``)와 그 모듈 네임스페이스에서 평가할 식을 콜론으로 구분해서 쓴다. 자세한 건 :func:`load` 참고. 다음이 예이다.

.. code-block:: console

    # 'myapp.controller' 모듈에서 'app' 객체를 잡고,
    # paste 서버를 전체 인터페이스 80 포트에서 시작.
    python -m bottle -server paste -bind 0.0.0.0:80 myapp.controller:app

    # 자동 재적재하는 개발용 서버를 시작하고 전역 기본 응용 제공.
    # 'test.py'에 라우트 정의돼 있음.
    python -m bottle --debug --reload test

    # 매개변수를 좀 줘서 자체 디버그 플러그인 설치.
    python -m bottle --debug --reload --plugin 'utils:DebugPlugin(exc=True)' test

    # 요청에 따라 'myapp.controller.make_app()'으로 생성한 응용 제공.
    python -m bottle 'myapp.controller:make_app()'


배치
================================================================================

보틀은 기본적으로 내장 `wsgiref WSGI 서버 <http://docs.python.org/library/wsgiref.html#module-wsgiref.simple_server>`_ 위에서 돈다. 스레드 없는 이 HTTP 서버가 개발 및 운용 초기에는 충분할 수 있겠지만 서버 부하가 커지면 성능 병목이 될 수 있다.

성능을 높이는 가장 쉬운 방법은 paste_\나 cherrypy_ 같은 다중 스레드 서버 라이브러리를 설치하고 보틀에서 단일 스레드 서버 대신 쓰게 하는 것이다. ::

    bottle.run(server='paste')

이 방식과 다른 배치 방식들을 :doc:`deployment` 글에서 따로 설명한다.




.. _tutorial-glossary:

용어
====

.. glossary::

   콜백(callback)
      어떤 외부 행위가 일어났을 때 호출할 프로그래머가 작성한 코드.
      웹 프레임워크에서는 각 URL에 콜백 함수를 지정하는 방식으로
      URL 경로와 응용 코드 간에 매핑을 하는 경우가 많다.

   데코레이터(decorator)
      다른 함수를 반환하는 함수. 일반적으로 ``@decorator`` 문법으로 함수를 변환하는 데 쓴다. 데코레이터에 대한 자세한 내용은 `파이썬 함수 정의 문서 <http://docs.python.org/reference/compound_stmts.html#function>`_ 참고.

   환경(environ)
      루트 아래 모든 문서들에 대한 정보가 저장돼 있는 구조체이며
      상호 참조에 사용한다. 파싱 단계 후에 환경을 pickle 하며,
      그래서 이후 동작에선 새로 추가되거나 바뀐 문서만 읽어서
      파싱하면 된다.

   핸들러 함수
      특정 사건이나 상황을 처리하는 함수. 웹 프레임워크에선 응용을
      구성하는 각 URL에 핸들러 함수를 콜백으로 붙이는 방식으로
      응용을 개발한다.

   소스 디렉터리
      스핑크스 프로젝트를 위한 모든 파일들을 담은 디렉터리 (하위 디렉터리 포함).

