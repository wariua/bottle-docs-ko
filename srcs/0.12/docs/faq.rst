.. module:: bottle

.. _paste: http://pythonpaste.org/modules/evalexception.html
.. _pylons: http://pylonshq.com/
.. _mod_python: http://www.modpython.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/

==============
자주 묻는 질문
==============

보틀에 대해
===========

복잡한 응용에 보틀이 적합한가?
-------------------------------

보틀은 작은 웹 응용 및 서비스를 시험하고 만들기 위해 설계된 *마이크로* 프레임워크다. 걸리적거리지 않고 일을 빨리 끝낼 수 있게 해 주지만 다른 프레임워크엔 있는 일부 고급 기능과 준비된 솔루션들(MVC, ORM, 양식 검사, 스캐폴딩, XML-RPC)이 빠져 있다. 보틀에 그런 기능들을 덧붙여서 복잡한 응용을 만드는 게 분명 가능하긴 하지만 그보단 pylons_\나 paste_\처럼 모든 걸 갖춘 웹 프레임워크 사용을 고려하는 게 좋다.


흔한 문제와 함정
================





mod_wsgi/mod_python에서 "Template Not Found"
--------------------------------------------------------------------------------

보틀은 ``./``\와 ``./views/``\에서 템플릿을 찾는다. mod_python_ 내지 mod_wsgi_ 환경에서는 Apache 설정에 따라 현재 디렉터리(``./``)가 달라진다. 템플릿 탐색 경로에 절대 경로를 추가해서 보틀이 올바른 경로를 탐색하도록 하면 된다. ::

    bottle.TEMPLATE_PATH.insert(0,'/absolut/path/to/templates/')

동적 라우트와 슬래시
--------------------------------------------------------------------------------

:ref:`동적 라우트 문법 <tutorial-dynamic-routes>`\에서 위치 토큰(``:name``)은 다음 슬래시 전의 전부에 걸린다. 정규 표현식 문법으로는 ``[^/]+``\와 동등하다. 슬래시도 받으려면 위치 토큰에 따로 정규 표현식 패턴을 추가해 줘야 한다. 예: ``/images/:filepath#.*#``\에는 ``/images/icons/error.png``\가 걸리지만 ``/images/:filename``\에는 안 걸린다.

역방향 프록시 문제
--------------------------------------------------------------------------------

보틀이 공개 주소와 응용 위치를 알고 있는 경우에만 재지향과 URL 만들기가 동작한다. 보틀을 역방향 프록시나 로드 밸런서 뒤에서 돌리면 도중에 일부 정보가 없어질 수 있다. 예를 들어 ``wsgi.url_scheme`` 값이나 ``Host`` 헤더가 클라이언트의 실제 요청이 아니라 프록시의 로컬 요청을 반영할 수가 있다. 다음은 그 값들을 정정하는 걸 도와 주는 작은 WSGI 미들웨어 코드다. ::

  def fix_environ_middleware(app):
    def fixed_app(environ, start_response):
      environ['wsgi.url_scheme'] = 'https'
      environ['HTTP_X_FORWARDED_HOST'] = 'example.com'
      return app(environ, start_response)
    return https_app

  app = bottle.default_app()    
  app.wsgi = fix_environ_middleware(app.wsgi)
  






