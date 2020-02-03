.. _flup: http://trac.saddi.com/flup
.. _gae: http://code.google.com/appengine/docs/python/overview.html
.. _wsgiref: http://docs.python.org/library/wsgiref.html
.. _cherrypy: http://www.cherrypy.org/
.. _paste: http://pythonpaste.org/
.. _rocket: http://pypi.python.org/pypi/rocket
.. _gunicorn: http://pypi.python.org/pypi/gunicorn
.. _fapws3: http://www.fapws.org/
.. _tornado: http://www.tornadoweb.org/
.. _twisted: http://twistedmatrix.com/
.. _diesel: http://dieselweb.org/
.. _meinheld: http://pypi.python.org/pypi/meinheld
.. _bjoern: http://pypi.python.org/pypi/bjoern
.. _gevent: http://www.gevent.org/
.. _eventlet: http://eventlet.net/
.. _waitress: http://readthedocs.org/docs/waitress/en/latest/
.. _apache: http://httpd.apache.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _pound: http://www.apsis.ch/pound



.. _tutorial-deployment:

================================================================================
배치
================================================================================

아무 인자 없이 보틀의 :func:`run` 함수를 호출하면 8080 포트로 로컬 개발 서버를 띄운다. 같은 호스트에 있으면 http://localhost:8080/\을 통해 응용에 접근해서 테스트 할 수 있다.

바깥 세상에서도 응용을 쓸 수 있게 하려면 서버가 리슨 할 인터페이스 IP를 지정하거나(예: ``run(host='192.168.0.1')``) 서버가 모든 인터페이스에 리슨 하게(예: ``run(host='0.0.0.0')``) 하면 된다. 리슨 할 포트도 비슷하게 바꿀 수 있지만 1024 아래 포트를 쓰려면 root 내지 관리자 권한이 필요하다. 80 포트가 HTTP 서버를 위한 표준이다. ::

  run(host='0.0.0.0', port=80) # 모든 인터페이스에서 HTTP 요청 받기

서버 선택지
================================================================================

기본 내장 서버는 `wsgiref WSGI 서버 <http://docs.python.org/library/wsgiref.html#module-wsgiref.simple_server>`_\를 기반으로 한 것이다. 스레드 안 쓰는 이 HTTP 서버가 개발 및 운용 초기에는 충분할 수 있지만 서버 부하가 커지면 성능 병목이 될 수 있다. 그 병목을 없앨 수 있는 방법이 세 가지 있다.

* 다중 스레드거나 비동기인 다른 서버 쓰기.
* 서버 프로세스를 여러 개 띄우고 로드 밸런서로 부하 나누기.
* 둘 다 하기.

**다중 스레드** 서버는 '전통적' 방법이다. 매우 견고하고 충분히 빠르며 관리하기 쉽다. 단점으로는, 동시에 처리할 수 있는 연결 수가 제한돼 있고 "전역 인터프리터 락(GIL)" 때문에 CPU 코어를 한 개만 활용한다. 대부분 응용에선 어차피 네트워크 IO에 대기하며 대부분의 시간을 보내기 때문에 문제가 되지 않지만 CPU 집약적인 작업(예: 이미지 처리)은 속도가 떨어질 수 있다.

**비동기** 서버는 아주 빠르고 실질적으로 무한대의 동시 연결을 처리할 수 있으며 관리하기 쉽지만 좀 까다로울 수 있다. 그 잠재력을 완전히 이용하려면 그에 맞도록 응용을 설계해야 하며 개별 서버의 개념들을 이해해야 한다.

**다중 프로세스**\(포크) 서버는 GIL의 제약을 받지 않아서 CPU 코어를 여러 개 활용할 수 있지만 서버 인스턴스 간 통신 비용이 커진다. 프로세스들 사이에 상태를 공유하려면 데이터베이스나 외부 메시지 큐가 필요하다. 아니면 아예 공유 상태가 필요 없도록 응용을 설계해야 한다. 구성 역시 좀 복잡하지만 잘 작성된 튜토리얼들이 있다.

서버 백엔드 바꾸기
================================================================================

성능을 높이는 가장 쉬운 방법은 paste_\나 cherrypy_ 같은 다중 스레드 서버 라이브러리를 설치하고 보틀에서 단일 스레드 서버 대신 그걸 쓰게 하는 것이다. ::

    bottle.run(server='paste')

보틀에는 많이 쓰는 WSGI 서버 대부분을 위한 어댑터가 딸려 있어서 구성 과정을 자동으로 대신 해 준다. 다음은 그 일부다.

========  ============  ======================================================
이름      홈페이지      설명
========  ============  ======================================================
cgi                     CGI 스크립트 실행
flup      flup_         FastCGI 프로세스로 실행
gae       gae_          구글 앱 엔진 도입 위한 헬퍼
wsgiref   wsgiref_      기본 단일 스레드 서버
cherrypy  cherrypy_     다중 스레드, 매우 안정적
paste     paste_        다중 스레드, 안정적, 경험으로 확인
rocket    rocket_       다중 스레드
waitress  waitress_     다중 스레드, Pyramid에서 사용
gunicorn  gunicorn_     사전 포크, 일부는 C로 작성
eventlet  eventlet_     WSGI 지원하는 비동기 프레임워크
gevent    gevent_       비동기 (greenlet)
diesel    diesel_       비동기 (greenlet)
fapws3    fapws3_       비동기 (네트워크 쪽만), C로 작성
tornado   tornado_      비동기, Facebook 일부에서 사용
twisted   twisted_      비동기, 충분히 검증됐지만... 뒤틀렸음
meinheld  meinheld_     비동기, 일부는 C로 작성
bjoern    bjoern_       비동기, 매우 빠르고 C로 작성
auto                    사용 가능한 서버 어댑터를 자동으로 선택
========  ============  ======================================================

전체 목록은 :data:`server_names`\를 통해 얻을 수 있다.

원하는 서버를 위한 어댑터가 없거나 서버 구성을 좀 더 제어해야 한다면 서버를 직접 띄우고 싶을 수 있다. 서버 문서에서 WSGI 응용을 실행하는 방법을 찾아 보면 된다. 다음 예는 paste_\를 띄우는 방법이다. ::

    application = bottle.default_app()
    from paste import httpserver
    httpserver.serve(application, host='0.0.0.0', port=80)



Apache mod_wsgi
--------------------------------------------------------------------------------

보틀 내에서 자체 HTTP 서버를 돌리지 않고 mod_wsgi_\를 쓰는 `Apache 서버 <apache>`_\에 보틀 응용을 붙일 수도 있다.

``application`` 객체를 제공하는 ``app.wsgi`` 파일만 만들어 주면 된다. mod_wsgi에서 그 객체를 써서 응용을 띄우는데 WSGI 호환 파이썬 콜러블이어야 한다.

``/var/www/yourapp/app.wsgi`` 파일::

    # 상대 경로가 (그리고 템플릿 탐색이) 동작하도록 작업 디렉터리 바꾸기
    os.chdir(os.path.dirname(__file__))
    
    import bottle
    # ... 여기서 보틀 응용을 구성 또는 임포트 ...
    # bottle.run()을 mod_wsgi과 함께 사용하지 말 것
    application = bottle.default_app()

Apache 설정은 다음처럼 된다. ::

    <VirtualHost *>
        ServerName example.com
        
        WSGIDaemonProcess yourapp user=www-data group=www-data processes=1 threads=5
        WSGIScriptAlias / /var/www/yourapp/app.wsgi
        
        <Directory /var/www/yourapp>
            WSGIProcessGroup yourapp
            WSGIApplicationGroup %{GLOBAL}
            Order deny,allow
            Allow from all
        </Directory>
    </VirtualHost>



구글 앱 엔진
--------------------------------------------------------------------------------

.. versionadded:: 0.9

서버 어댑터 ``gae``\를 써서 구글 앱 엔진에서 응용을 돌릴 수 있다. HTTP 서버를 새로 시작하지 않는다는 점에서 ``cgi`` 어댑터와 비슷하지만 더불어 구글 앱 엔진에 맞게 응용을 준비 및 최적화하고 그 API에 맞게 만들어 준다. ::

    bottle.run(server='gae') # host나 port 설정 필요 없음

정적 파일은 GAE에서 직접 제공하도록 하는 게 좋다. 다음이 ``app.yaml`` 예시다. ::

    application: myapp
    version: 1
    runtime: python
    api_version: 1

    handlers:
    - url: /static
      static_dir: static

    - url: /.*
      script: myapp.py


로드 밸런서 (수동 구성)
--------------------------------------------------------------------------------

파이썬 프로세스 하나는 가용 CPU 코어가 몇 개든 한 번에 한 CPU만 활용할 수 있다. 이때 가능한 기법은 독립된 파이썬 프로세스들로 부하를 분산해서 CPU 코어를 모두 활용하는 것이다.

보틀 응용 서버를 하나만 띄우는 게 아니라 사용 가능한 CPU 코어마다 로컬 포트를 다르게 해서 (localhost:8080, 8081, 8082, ...) 한 인스턴스씩 실행한다. 원하는 대로 서버 어댑터를 고를 수 있고 비동기 어댑터도 쓸 수 있다. 다음으로 고성능 로드 밸런서가 역방향 프록시 역할을 하며 각 요청을 무작위 포트로 전달하고, 그래서 동작 중인 모든 백엔드로 부하를 분산시킨다. 이렇게 하면 모든 CPU 코어를 쓸 수 있고 심지어 여러 물리적 서버로 부하를 나눌 수도 있다.

아주 빠른 로드 밸런서로 Pound_\가 있지만 많이 쓰는 웹 서버 대부분에도 그 일을 잘 해 주는 프록시 모듈이 있다.

Pound 예시::

    ListenHTTP
        Address 0.0.0.0
        Port    80

        Service
            BackEnd
                Address 127.0.0.1
                Port    8080
            End
            BackEnd
                Address 127.0.0.1
                Port    8081
            End
        End
    End

Apache 예시::

    <Proxy balancer://mycluster>
    BalancerMember http://192.168.1.50:80
    BalancerMember http://192.168.1.51:80
    </Proxy>
    ProxyPass / balancer://mycluster 

Lighttpd 예시::

    server.modules += ( "mod_proxy" )
    proxy.server = (
        "" => (
            "wsgi1" => ( "host" => "127.0.0.1", "port" => 8080 ),
            "wsgi2" => ( "host" => "127.0.0.1", "port" => 8081 )
        )
    )


옛 시절의 CGI
================================================================================

CGI 서버는 요청마다 새 프로세스를 시작한다. 이로 인한 오버헤드가 크지만 때로는, 특히 저렴한 호스팅 환경에선 유일한 선택지일 수 있다. 서버 어댑터 `cgi`\는 실제 CGI 서버를 띄우는 게 아니라 보틀 응용을 유효한 CGI 응용으로 바꿔 준다. ::

    bottle.run(server='cgi')



