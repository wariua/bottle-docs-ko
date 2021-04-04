.. highlight:: python
.. currentmodule:: bottle

.. _mako: http://www.makotemplates.org/
.. _cheetah: http://www.cheetahtemplate.org/
.. _jinja2: http://jinja.pocoo.org/
.. _paste: http://pythonpaste.org/
.. _fapws3: https://github.com/william-os4y/fapws3
.. _bjoern: https://github.com/jonashaag/bjoern
.. _flup: http://trac.saddi.com/flup
.. _cherrypy: http://www.cherrypy.org/
.. _WSGI: http://www.wsgi.org/
.. _파이썬: http://python.org/
.. _testing: https://github.com/defnull/bottle/raw/master/bottle.py
.. _issue_tracker: https://github.com/defnull/bottle/issues
.. _PyPI: http://pypi.python.org/pypi/bottle

==========================
보틀: 파이썬 웹 프레임워크
==========================

보틀(Bottle)은 파이썬_\을 위한 빠르고 단순하며 경량인 WSGI_ 마이크로 웹 프레임워크다. 한 파일로 된 모듈로 배포되며 `파이썬 표준 라이브러리 <http://docs.python.org/library/>`_ 외에 다른 의존성이 없다.


* **라우팅:** 요청을 함수 호출로 연결해 준다. 단순한 URL과 동적인 URL을 지원한다.
* **템플릿:** 빠르고 파이썬스러운 :ref:`내장 템플릿 엔진 <tutorial-templates>`\이 있으며 mako_, jinja2_, cheetah_ 템플릿을 지원한다.
* **유틸리티:** 폼 데이터, 파일 업로드, 쿠키, 헤더, 기타 HTTP 관련 메타데이터에 편리하게 접근할 수 있다.
* **서버:** 개발용 HTTP 서버가 내장돼 있으며 paste_, fapws3_, bjoern_, `구글 앱 엔진 <http://code.google.com/intl/en-US/appengine/>`_, cherrypy_, 기타 WSGI_ 지원 HTTP 서버와 쓸 수 있다.

.. rubric:: 예시: 보틀에서 "Hello World"

::

  from bottle import route, run, template

  @route('/hello/<name>')
  def index(name):
      return template('<b>Hello {{name}}</b>!', name=name)

  run(host='localhost', port=8080)

이 스크립트를 실행하거나 파이썬 콘솔에 붙여 넣은 다음 브라우저로 `<http://localhost:8080/hello/world>`_ 주소를 열어 보자. 이게 전부다.

.. rubric:: 내려받아 설치하기

.. _download:

.. __: https://github.com/defnull/bottle/raw/master/bottle.py

PyPI_\를 통해 (``easy_install -U bottle``) 최신 안정 버전을 설치하거나 `bottle.py`__ 파일(불안정 버전)을 프로젝트 디렉터리로 내려받으면 된다. 파이썬 표준 라이브러리 외 모듈에 대한 필수 의존성이 전혀 없다 [1]_. 보틀은 **파이썬 2.5+ 및 3.x**\에서 돈다.

사용자 안내서
=============
웹 개발에 보틀 프레임워크를 이용하는 방법을 배우고 싶다면 여기서 시작하면 된다. 여기서 답을 얻지 못한 질문이 있다면 `메일링 리스트 <mailto:bottlepy@googlegroups.com>`_\에서 마음껏 물어 보면 된다.

.. toctree::
   :maxdepth: 2

   tutorial
   configuration
   routing
   stpl
   api
   plugins/index


지식 베이스
===========
글, 지침, 하우투 모음.

.. toctree::
   :maxdepth: 2

   tutorial_app
   async
   recipes
   faq


개발 및 기여
============

보틀 개발 및 릴리스 절차에 관심 있는 개발자들을 위한 장이다.

.. toctree::
   :maxdepth: 2

   changelog
   development
   plugindev


.. toctree::
   :hidden:

   plugins/index

라이선스
========

MIT 라이선스에 따라 코드와 문서를 사용할 수 있다.

.. include:: ../LICENSE
  :literal:

하지만 보틀 로고는 그 라이선스에 포함되지 *않는다*. 보틀 홈페이지에
대한 링크로 쓰거나 변경 안 된 라이브러리와 함께 직접 사용하는 건 허용된다.
그 외 경우에는 먼저 확인을 구하길 바란다.

.. rubric:: 각주

.. [1] 템플릿이나 서버 어댑터 클래스 사용을 위해선 당연히 해당 템플릿이나 서버 모듈이 필요하다.

