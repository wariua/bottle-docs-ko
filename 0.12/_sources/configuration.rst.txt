==============
설정 (작업 중)
==============

.. currentmodule:: bottle

보틀 응용에서 :attr:`Bottle.config`\에 설정을 저장할 수 있다. dict 같은 객체로, 응용별 설정을 위한 중심 저장소다. 이 딕셔너리는 프레임워크의 많은 부분을 제어하고 (최신) 플러그인들에 할 일을 알려 주며 응용 자체 설정을 저장하는 데도 사용 가능하다.

설정 기본
=========

:attr:`Bottle.config` 객체는 일반 딕셔너리와 아주 비슷하게 동작한다. dict 공통 메소드들이 모두 기대처럼 동작한다. 몇 가지 예로 시작해 보자. ::

    import bottle
    app = bottle.default_app()             # 또는 취향에 따라 bottle.Bottle()

    app.config['autojson']    = False      # "autojson" 기능 끄기
    app.config['sqlite.db']   = ':memory:' # sqlite 플러그인에 사용할 DB 알려 주기
    app.config['myapp.param'] = 'value'    # 자체 설정 값 예시

    # 여러 값 한꺼번에 바꾸기
    app.config.update({
        'autojson': False,
        'sqlite.db': ':memory:',
        'myapp.param': 'value'
    })

    # 기본값 추가하기
    app.config.setdefault('myapp.param2', 'some default')

    # 값 얻기
    param  = app.config['myapp.param']
    param2 = app.config.get('myapp.param2', 'fallback value')

    # 설정 값 이용하는 예시 라우트
    @app.route('/about', view='about.rst')
    def about():
        email = app.config.get('my.email', 'nomail@example.com')
        return {'email': email}

앱 객체가 항상 사용 가능한 건 아니지만 요청 문맥 안에 있는 동안은 언제든 `request` 객체를 이용해 현재 응용과 그 설정을 얻을 수 있다. ::

    from bottle import request
    def is_admin(user):
        return user == request.app.config['myapp.admin_user']

이름 관행
=========

일을 편하게 만들기 위해 플러그인과 응용에서 설정 매개변수 이름에 관한 간단한 규칙 몇 가지를 따르는 게 좋다.

- 모든 키는 소문자 문자열이고 파이썬 식별자에 대한 규칙들을 따라야 한다. (특수 문자 안 되지만 밑줄은 가능.)
- 점으로 네임스페이스를 구분한다. (예: ``namespace.field``, ``namespace.subnamespace.field``)
- 보틀에서 자기 설정에 루트 네임스페이스를 쓴다. 플러그인에선 모든 변수들을 각자의 네임스페이스에 저장해야 한다. (예: ``sqlite.db``, ``werkzeug.use_debugger``)
- 응용에서도 별도의 네임스페이스를 써야 한다. (예: ``myapp.*``)


파일에서 설정 읽어 오기
=======================

.. versionadded 0.12

프로그래머 아닌 사람이 응용 설정을 할 수 있게 하고 싶은 경우, 아니면 데이터베이스
포트 바꾸려고 파이썬 모듈을 건드리는 건 싫을 때 설정 파일이 유용하다. 보다시피
설정 파일에 많이 쓰는 문법이다.

.. code-block:: ini

    [bottle]
    debug = True

    [sqlite]
    db = /tmp/test.db
    commit = auto

    [myapp]
    admin_user = defnull

:meth:`ConfigDict.load_config`\를 쓰면 디스크의 ``*.ini`` 방식 설정 파일을
읽어 들여서 기존 설정 안으로 값들을 가져올 수 있다. ::

    app.config.load_config('/etc/myapp.conf')

중첩 :class:`dict`\에서 설정 읽어 오기
======================================

.. versionadded 0.12

유용한 또 다른 메소드로 :meth:`ConfigDict.load_dict`\가 있다. 이 메소드는
중첩 딕셔너리 구조를 통째로 받아서 평탄한 키와 값 목록으로 바꿔 주며
키에는 네임스페이스를 붙여 준다. ::

    # dict 구조 통째로 읽어 들이기
    app.config.load_dict({
        'autojson': False,
        'sqlite': { 'db': ':memory:' },
        'myapp': {
            'param': 'value',
            'param2': 'value2'
        }
    })
    
    assert app.config['myapp.param'] == 'value'

    # json 파일에서 설정 읽어 들이기
    with open('/etc/myapp.json') as fp:
        app.config.load_dict(json.load(fp))


설정 변경 주시하기
==================

.. versionadded 0.12

:attr:`Bottle.config`\의 값이 바뀔 때마다 응용 객체의 ``config`` 훅이 실행된다. 이 훅을 이용해 런타임에 설정 변경에 반응할 수 있다. 예를 들어 새 데이터베이스에 재연결하거나, 백그라운드 서비스의 디버그 설정을 바꾸거나, 작업 스레드 풀 크기를 조정할 수 있다. 훅 콜백은 인자 두 개(key, new_value)를 받으며 딕셔너리에서 값이 실제로 바뀌기 전에 호출된다. 훅 콜백에서 예외를 던지면 그 변경이 취소되고 이전 값이 유지된다.

::

  @app.hook('config')
  def on_config_change(key, value):
    if key == 'debug':
        switch_own_debug_mode_to(value)

딕셔너리에 저장되려는 값을 훅 콜백에서 *바꾸지는* 못한다. 그건 필터의 역할이다.


.. conf-meta:

필터 및 기타 메타 데이터
========================

.. versionadded 0.12

:class:`ConfigDict`\에서는 설정 키별로 메타 데이터를 함께 저장할 수 있다. 현재 두 가지 메타 필드가 정의돼 있다.

help
    도움말 또는 설명 문자열. 디버깅이나 인트로스펙션에, 또는 관리 도구에서
    사이트 관리자의 응용 설정을 돕는 데 이용할 수 있다.

filter
    값 하나를 받고 반환하는 콜러블이다. 키에 필터가 정의돼 있으면 그 키에 저장되는 새 값이 먼저 필터 콜백을 거친다. 필터를 이용해 값을 다른 타입으로 바꾸거나, 값이 유효한지 검사하거나 (ValueError 던지기), 부대 효과를 일으킬 수 있다.

이 기능은 플러그인에서 특히 유용하다. 설정 필터를 이용해 매개변수를 검사하거나 부대 효과를 일으킬 수 있으며 ``help`` 필드를 통해 설정에 설명을 달 수 있다. ::

    class SomePlugin(object):
        def setup(app):
            app.config.meta_set('some.int', 'filter', int)
            app.config.meta_set('some.list', 'filter',
                lambda val: str(val).split(';'))
            app.config.meta_set('some.list', 'help',
                'A semicolon separated list.')

        def apply(self, callback, route):
            ...

    import bottle
    app = bottle.default_app()
    app.install(SomePlugin())

    app.config['some.list'] = 'a;b;c'     # 실제 저장되는 건 ['a', 'b', 'c']
    app.config['some.int'] = 'not an int' # ValueError 발생


API 문서
========

.. versionadded 0.12

.. autoclass:: ConfigDict
   :members:
