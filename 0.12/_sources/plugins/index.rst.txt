.. module:: bottle

======================
사용 가능한 플러그인들
======================

다음은 보틀 핵심 기능을 확장해 주거나 다른 라이브러리를 보틀 프레임워크에 연결해 주는 서드파티 플러그인들이다.

플러그인에 대한 일반적인 질문(설치, 사용)은 :ref:`plugins` 절을 보라. 새 플러그인을 개발할 생각이라면 :doc:`/plugindev`\가 도움이 될 수 있다.

`Bottle-Cork <http://cork.firelet.net/>`_
    Cork은 보틀 기반 웹 응용에서 인증 및 인가를 구현하기 위한 간단한 메소드들을 제공한다.

`Bottle-Extras <http://pypi.python.org/pypi/bottle-extras/>`_
    보틀 플러그인 모음을 설치하기 위한 메타 패키지.

`Bottle-Flash <http://pypi.python.org/pypi/bottle-flash/>`_
    보틀 플래시 메시지 플러그인.

`Bottle-Hotqueue <http://pypi.python.org/pypi/bottle-hotqueue/>`_
    redis 기반의 보틀용 FIFO 큐.

`Macaron <http://nobrin.github.com/macaron/webapp.html>`_
    Macaron은 SQLite용 객체 관계 매퍼(ORM)다.

`Bottle-Memcache <http://pypi.python.org/pypi/bottle-memcache/>`_
    보틀에 Memcache 연결.

`Bottle-MongoDB <http://pypi.python.org/pypi/bottle-mongodb/>`_
    보틀에 MongoDB 연결.

`Bottle-Redis <http://pypi.python.org/pypi/bottle-redis/>`_
    보틀에 Redis 연결.

`Bottle-Renderer <http://pypi.python.org/pypi/bottle-renderer/>`_
    보틀을 위한 렌더러 플러그인.

`Bottle-Servefiles <http://pypi.python.org/pypi/bottle-servefiles/>`_
    보틀 앱에서 정적 파일을 처리해 주는 재사용 가능 앱.

`Bottle-Sqlalchemy <http://pypi.python.org/pypi/bottle-sqlalchemy/>`_
    보틀에 SQLAlchemy 연결.

`Bottle-Sqlite <http://pypi.python.org/pypi/bottle-sqlite/>`_
    보틀에 SQLite3 데이터베이스 연결.

`Bottle-Web2pydal <http://pypi.python.org/pypi/bottle-web2pydal/>`_
    보틀에 Web2py Dal 연결.

`Bottle-Werkzeug <http://pypi.python.org/pypi/bottle-werkzeug/>`_
    `werkzeug` 라이브러리 연결. (새로운 요청 및 응답 객체, 고급 디버깅 미들웨어 등)

여기 나열된 플러그인들은 보틀이나 보틀 프로젝트의 일부가 아니며 제삼자가 개발 및 유지한다.

.. toctree::
    :glob:
    :hidden:
    
    /plugins/*
