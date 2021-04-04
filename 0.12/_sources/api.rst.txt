==============================
API 참조
==============================

.. module:: bottle
   :platform: Unix, Windows
   :synopsis: WSGI micro framework
.. moduleauthor:: Marcel Hellkamp <marc@gsites.de>

대부분 자동 생성된 API 문서다. 보틀이 처음이라면 서술식으로 된
:doc:`tutorial`\가 더 유용할 수 있다.




모듈들
=====================================

모듈에는 여러 함수, 상수, 예외가 정의돼 있다.

.. autofunction:: debug

.. autofunction:: run

.. autofunction:: load

.. autofunction:: load_app

.. autodata:: request

.. autodata:: response

.. autodata:: HTTP_CODES

.. function:: app()
              default_app()

    현재 :ref:`default-app`\을 반환. 실제로 이는 호출 가능한 :class:`AppStack` 인스턴스이며 스택스러운 API를 구현하고 있다.


라우팅
-------------------

보틀은 :class:`Bottle` 인스턴스들의 스택을 유지하며 (:func:`app` 및 :class:`AppStack` 참고) 스택 최상단 인스턴스를 일부 모듈 층위 함수와 데코레이터에서 *기본 응용*\으로 쓴다.


.. function:: route(path, method='GET', callback=None, **options)
              get(...)
              post(...)
              put(...)
              delete(...)

   현재 기본 응용에 라우트를 설치하는 데코레이터. 자세한 내용은 :meth:`Bottle.route` 참고.


.. function:: error(...)

   현재 기본 응용에 오류 핸들러를 설치하는 데코레이터. 자세한 내용은 :meth:`Bottle.error` 참고.


WSGI 및 HTTP 유틸리티
----------------------------

.. autofunction:: parse_date

.. autofunction:: parse_auth

.. autofunction:: cookie_encode

.. autofunction:: cookie_decode

.. autofunction:: cookie_is_encoded

.. autofunction:: yieldroutes

.. autofunction:: path_shift


자료 구조
----------------------

.. autoclass:: MultiDict
   :members:

.. autoclass:: HeaderDict
   :members:

.. autoclass:: FormsDict
   :members:

.. autoclass:: WSGIHeaderDict
   :members:

.. autoclass:: AppStack
   :members:

   .. method:: pop()

      현재 기본 응용을 반환하고 스택에서 제거.

.. autoclass:: ResourceManager
   :members:

.. autoclass:: FileUpload
   :members:

예외
---------------

.. autoexception:: BottleException
   :members:



:class:`Bottle` 클래스
=========================

.. autoclass:: Bottle
   :members:

.. autoclass:: Route
    :members:


:class:`Request` 객체
===================================================

:class:`Request` 클래스는 WSGI 환경을 포장한 것이며 양식 데이터, 쿠키, 파일 업로드, 기타 메타데이터를 파싱하고 접근하기 위한 유용한 메소드들을 제공한다. 대부분 속성이 읽기 전용이다.

.. autoclass:: Request
   :members:

.. autoclass:: BaseRequest
   :members:

모듈 층위의 :data:`bottle.request`\는 프록시 객체(:class:`LocalRequest`\에 구현)이며 항상 `현재` 요청, 즉 현재 스레드에서 현재 요청 핸들러가 처리 중인 요청을 가리킨다. 이 `스레드 로컬` 덕분에 다중 스레드 환경에서 전역 인스턴스를 안전하게 이용할 수 있다.

.. autoclass:: LocalRequest
   :members:


.. autodata:: request

:class:`Response` 객체
===================================================

:class:`Response` 클래스는 클라이언트에게 보낼 HTTP 상태 코드와 헤더 및 쿠키를 담는다. :data:`bottle.request`\와 비슷하게 스레드 로컬인 :data:`bottle.response` 인스턴스가 있어서 `현재` 응답을 조정하는 데 쓸 수 있다. 더불어 요청 핸들러 안에서 :class:`Response` 인스턴스를 만들어 반환할 수도 있다. 그 경우 전역 인스턴스 대신 새로 만든 인스턴스에 정의된 헤더와 쿠키가 쓰인다.

.. autoclass:: Response
   :members:

.. autoclass:: BaseResponse
   :members:

.. autoclass:: LocalResponse
   :members:


다음 두 클래스를 예외로 던질 수 있다. 둘의 중요한 차이는 :class:`HTTPError`\에 대해선 보틀이 오류 핸들러를 호출하지만 :class:`HTTPResponse`\나 다른 응답 타입에 대해선 하지 않는다는 점이다.

.. autoexception:: HTTPResponse
   :members:

.. autoexception:: HTTPError
   :members:




템플릿
======

:mod:`bottle`\에서 지원하는 모든 템플릿 엔진은 :class:`BaseTemplate` API를 구현하고 있다. 그래서 응용 코드를 전혀 바꾸지 않고도 템플릿 엔진을 바꾸거나 혼용하는 게 가능하다.

.. autoclass:: BaseTemplate
   :members:

   .. automethod:: __init__

.. autofunction:: view

.. autofunction:: template

좋아하는 템플릿 엔진을 위한 어댑터를 작성하거나 기존 어댑터를 이용할 수 있다. 제대로 지원하는 템플릿 엔진이 현재 네 가지 있다.

========================   ===============================   ====================   ========================
클래스                     URL                               데코레이터             렌더링 함수
========================   ===============================   ====================   ========================
:class:`SimpleTemplate`    :doc:`stpl`                       :func:`view`           :func:`template`
:class:`MakoTemplate`      http://www.makotemplates.org      :func:`mako_view`      :func:`mako_template`
:class:`CheetahTemplate`   http://www.cheetahtemplate.org/   :func:`cheetah_view`   :func:`cheetah_template`
:class:`Jinja2Template`    http://jinja.pocoo.org/           :func:`jinja2_view`    :func:`jinja2_template`
========================   ===============================   ====================   ========================

:class:`MakoTemplate`\을 기본 템플릿 엔진으로 쓰려면 해당 데코레이터와 렌더링 함수를 임포트하기만 하면 된다. ::

  from bottle import mako_view as view, mako_template as template

