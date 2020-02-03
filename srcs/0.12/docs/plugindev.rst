.. module:: bottle


====================
플러그인 개발 안내서
====================

이 안내서에서는 플러그인 API와 새 플러그인 작성 방법을 설명한다. :ref:`plugins` 절을 아직 안 읽었으면 먼저 읽기를 권한다. :doc:`plugins/index`\에서 몇몇 실제 예를 보고 싶을 수도 있겠다.

.. note::

    작업 중인 문서다. 오류가 있거나 특정 부분의 설명이 명확하지 않다면 `메일링 리스트 <mailto:bottlepy@googlegroups.com>`_\로 알려 주거나 `버그 보고 <https://github.com/defnull/bottle/issues>`_\를 해 주길 바란다.


플러그인 기본 동작 방식
=======================

플러그인 API는 `데코레이터 <http://docs.python.org/glossary.html#term-decorator>`_  개념을 바탕으로 한다. 간단히 말해 플러그인이란 응용의 각 라우트 콜백에 적용되는 데코레이터다.

물론 단순화하면 그렇다는 것이고 플러그인은 라우트 콜백을 꾸미는 것보다 훨씬 많은 걸 할 수 있다. 하지만 시작점으로는 좋다. 코드를 좀 보자. ::

    from bottle import response, install
    import time

    def stopwatch(callback):
        def wrapper(*args, **kwargs):
            start = time.time()
            body = callback(*args, **kwargs)
            end = time.time()
            response.headers['X-Exec-Time'] = str(end - start)
            return body
        return wrapper

    install(stopwatch)

이 플러그인은 각 요청의 실행 시간을 측정해서 그에 따른 ``X-Exec-Time`` 헤더를 응답에 추가한다. 보다시피 플러그인은 래퍼를 반환하고 그 래퍼가 다시 원래 콜백을 호출한다. 데코레이터가 일반적으로 이렇게 동작한다.

마지막 행은 기본 응용에 플러그인을 설치하도록 한다. 그러면 그 응용의 모든 라우트에 자동으로 그 플러그인이 적용된다. 달리 말해 각 라우트 콜백마다 한 번씩 ``stopwatch()``\가 호출되고 그 반환 값이 원래 콜백 대신 쓰인다.

플러그인은 온디맨드 방식이다. 즉 라우트에 처음으로 요청이 있을 때 적용된다. 다중 스레드 환경에서 이게 제대로 동작하려면 플러그인이 스레드에 안전해야 한다. 대부분의 경우는 문제가 되지 않지만 유념하고 있어야 한다.

라우트에 플러그인들을 모두 적용하고 나면 포장된 콜백을 캐싱 해서 이후 요청은 캐싱 된 버전을 바로 사용해 처리한다. 즉 개별 라우트에 대해 일반적으로 플러그인이 한 번만 적용된다. 하지만 그 캐시는 설치 플러그인 목록이 바뀔 때마다 비워진다. 따라서 플러그인이 같은 라우트를 여러 번 꾸밀 수 있도록 해야 한다.

하지만 데코레이터 API는 상당히 제한적이다. 꾸미고 있는 라우트나 연관 응용 객체에 대해 아무것도 알 수 없으며 모든 라우트에 대해 공유하는 데이터를 저장할 효율적인 방법도 없다. 하지만 걱정 말자. 플러그인은 데코레이터 함수만 가능한 게 아니다. 호출 가능하거나 어떤 API를 구현하고 있기만 하다면 뭐든 보틀에서 플러그인으로 받아들인다. 아래 설명하는 그 API를 통해 전체 과정을 상당히 통제할 수 있다.


플러그인 API
============

:class:`Plugin`\은 진짜 클래스가 아니라 (:mod:`bottle`\로부터 임포트 할 수 없음) 플러그인이 구현해야 할 인터페이스다. 다음 API를 준수하기만 한다면 어떤 타입의 어떤 객체든 플러그인으로 받아들인다.

.. class:: Plugin(object)

    플러그인은 호출 가능하거나 :meth:`apply`\를 구현해야 한다. :meth:`apply`\가 정의돼 있으면 항상 그걸 플러그인 직접 호출보다 우선한다. 다른 메소드와 속성들은 모두 선택적이다.

    .. attribute:: name

        :meth:`Bottle.uninstall`\과 :meth:`Bottle.route()`\의 `skip` 매개변수는 플러그인 또는 그 타입을 나타내는 이름 문자열을 받는다. name 속성이 있는 플러그인에만 그게 동작한다.

    .. attribute:: api

        플러그인 API는 계속 진화하고 있다. 이 정수 속성은 어느 버전을 사용해야 할지를 보틀에게 알려 준다. 속성이 없으면 첫 버전을 상정한다. 최신 버전은 ``2``\다. 자세한 내용은 :ref:`plugin-changelog` 참고.

    .. method:: setup(self, app)

        응용에 플러그인이 설치되자마자 호출된다. (:meth:`Bottle.install` 참고.) 유일한 매개변수는 연계된 응용 객체다.

    .. method:: __call__(self, callback)

        :meth:`apply`\가 정의돼 있지 않으면 플러그인 자체를 데코레이터로 사용해서 각 라우트 콜백에 직접 적용한다. 유일한 매개변수는 꾸며야 할 콜백이다. 이 메소드가 뭘 반환하든 그게 원래 콜백을 대체한다. 받은 콜백을 포장하거나 대체할 필요가 없다면 callback 매개변수를 변경 없이 반환하면 된다.

    .. method:: apply(self, callback, route)

        정의돼 있으면 :meth:`__call__` 대신 이 메소드를 써서 라우트 콜백을 꾸민다. 추가로 있는 `route` 매개변수는 :class:`Route` 인스턴스로, 그 라우트에 대한 많은 메타 정보와 문맥을 제공한다. 자세한 내용은 :ref:`route-context` 참고.

    .. method:: close(self)

        플러그인이 제거되거나 응용이 끝나기 직전에 호출된다. (:meth:`Bottle.uninstall` 또는 :meth:`Bottle.close` 참고.)


:meth:`Bottle.route()` 데코레이터를 통해 라우트에 직접 적용된 플러그인에 대해선 :meth:`Plugin.setup`\과 :meth:`Plugin.close`\가 호출되지 *않는다*. 응용에 설치된 플러그인에 대해서만 호출된다.


.. _plugin-changelog:

플러그인 API 변경 사항
----------------------

플러그인 API는 계속 진화하고 있으며 보틀 0.10에서 라우트 문맥 딕셔너리 관련 이슈를 해결하기 위해 바뀐 적이 있다. 0.9 플러그인과 하위 호환성을 보장하기 위해 선택적인 :attr:`Plugin.api` 속성을 추가해서 어떤 API를 써야 하는지 알려 주도록 했다. API 차이를 요약하면 다음과 같다.

* **보틀 0.9 API 1** (:attr:`Plugin.api` 없음)

  * 0.9 문서에 기술돼 있던 초판 플러그인 API.

* **보틀 0.10 API 2** (:attr:`Plugin.api`\가 2)

  * :meth:`Plugin.apply` 메소드의 `context` 매개변수가 이제 딕셔너리가 아니라 :class:`Route` 인스턴스다.

.. _route-context:


라우트 문맥
===========

:meth:`Plugin.apply`\로 전달되는 :class:`Route` 인스턴스는 연관 라우트에 대한 자세한 정보를 제공한다. 중요한 속성들로 다음이 있다.

===========  =================================================================
속성         설명
===========  =================================================================
app          이 라우트가 설치된 응용 객체.
rule         규칙 문자열. (예: ``/wiki/:page``)
method       HTTP 메소드 문자열. (예: ``GET``)
callback     어떤 플러그인도 적용되지 않은 원래 콜백. 인트로스펙션에 유용함.
name         라우트의 이름. 지정돼 있지 않으면 ``None``.
plugins      라우트별 플러그인 목록. 응용 전역 플러그인들에 더해서
             이 플러그인들이 적용된다. (:meth:`Bottle.route` 참고.)
skiplist     이 라우트에 적용하지 않을 플러그인 목록. (:meth:`Bottle.route`
             참고.)
config       :meth:`Bottle.route` 데코레이터에 추가로 준 키워드 인자들이
             이 딕셔너리에 저장된다. 라우트별 설정 및 메타데이터에 쓰인다.
===========  =================================================================

아마 :attr:`Route.config`\가 가장 중요한 속성일 것이다. 그 딕셔너리가 라우트에 로컬이긴 하지만 모든 플러그인들이 공유한다는 걸 유념해야 한다. 고유 접두부를 붙이는 것도 당연히 좋은 생각이고, 플러그인에 설정이 많이 필요하다면 `config` 딕셔너리 안의 별도 네임스페이스에 저장하면 된다. 그러면 플러그인들 간의 이름 충돌을 피할 수 있다.


:class:`Route` 객체 바꾸기
--------------------------

:class:`Route` 속성 중 일부가 변경 가능하긴 하지만 다른 플러그인에 의도치 않은 영향을 줄 수도 있다. 이상 있는 라우트가 있을 때 유용한 오류 메시지를 제공해서 사용자가 문제를 고치게 하지 않고 그냥 몽키 패치 하는 건 안 좋은 생각일 가능성이 높다.

하지만 몇몇 드문 경우들에선 이 규칙을 깨는 게 정당화될 수도 있다. :class:`Route` 인스턴스를 변경한 후에는 :exc:`RouteReset`\을 예외로 던져야 한다. 그러면 현 라우트가 캐시에서 제거되고 모든 플러그인들이 다시 적용된다. 하지만 라우터는 갱신되지 않는다. `rule`\이나 `method` 값을 바꿔도 라우터에는 아무 영향이 없고 플러그인에만 효과가 있다. 다만 이는 향후에 바뀔 수도 있다.


런타임 최적화
=============

라우트에 모든 플러그인들이 적용되고 나면 포장된 라우트 콜백을 캐싱 해서 이후 요청들의 처리 속도를 높인다. 작성하려는 플러그인의 동작 방식이 설정에 따라 달라지는데 런타임에 그 설정을 바꿀 수 있게 하고 싶다면 매 요청마다 그 설정을 읽어야 한다. 아주 간단하다.

하지만 성능상의 이슈로 그때 그때 필요에 따라 다른 래퍼를 선택하거나, 클로저를 쓰거나, 런타임에 플러그인을 켜고 끄는 걸 할 만할 수도 있다. 내장된 HooksPlugin을 예로 들어 보자. 훅이 전혀 설치돼 있지 않으면 영향 받은 모든 라우트에서 플러그인 스스로를 제거해서 사실상 오버헤드가 전혀 없다. 그러다 첫 번째 훅을 설치하면 플러그인 스스로를 활성화해서 다시 효력이 생긴다.

이게 가능하려면 콜백 캐시를 통제할 수 있어야 한다. :meth:`Route.reset`\은 그 한 라우트에 대한 캐시를 지우고 :meth:`Bottle.reset`\은 응용의 모든 라우트에 대한 캐시 전체를 한꺼번에 지운다. 다음 요청에서 모든 플러그인들이 라우트에 처음 적용 요청한 것처럼 재적용된다.

당연하지만 두 메소드 모두 라우트 콜백 내에서 호출 시 현재 요청에는 영향을 주지 않는다. 현재 요청을 재시작하게 하고 싶으면 :exc:`RouteReset`\을 예외로 던지면 된다.


예시 플러그인: SQLitePlugin
===========================

이 플러그인은 포장되는 콜백에 sqlite3 데이터베이스 연결 핸들을 추가 키워드 인자로 제공하되, 콜백에서 받으려는 경우에만 그렇게 한다. 원치 않으면 그 라우트를 무시하며 어떤 오버헤드도 더해지지 않는다. 래퍼가 반환 값에는 영향을 주지 않지만 플러그인 관련 예외를 제대로 처리해 준다. :meth:`Plugin.setup`\으로 응용을 조사해서 충돌하는 플러그인을 찾아 본다.

::

    import sqlite3
    import inspect

    class SQLitePlugin(object):
        ''' 이 플러그인은 키워드 인자 `db`\를 받는 라우트 콜백에 sqlite3
        데이터베이스 핸들을 전달해 준다. 콜백에서 그런 매개변수를 받지
        않는 경우엔 연결을 만들지 않는다. 라우트별로 데이터베이스 설정을
        오버라이드 할 수 있다. '''

        name = 'sqlite'
        api = 2

        def __init__(self, dbfile=':memory:', autocommit=True, dictrows=True,
                     keyword='db'):
             self.dbfile = dbfile
             self.autocommit = autocommit
             self.dictrows = dictrows
             self.keyword = keyword

        def setup(self, app):
            ''' 설치된 다른 플러그인에서 같은 키워드 인자에 영향 주지
                않는지 확인한다.'''
            for other in app.plugins:
                if not isinstance(other, SQLitePlugin): continue
                if other.keyword == self.keyword:
                    raise PluginError("Found another sqlite plugin with "\
                    "conflicting settings (non-unique keyword).")

        def apply(self, callback, context):
            # 라우트별 값으로 전역 설정 오버라이드
            conf = context.config.get('sqlite') or {}
            dbfile = conf.get('dbfile', self.dbfile)
            autocommit = conf.get('autocommit', self.autocommit)
            dictrows = conf.get('dictrows', self.dictrows)
            keyword = conf.get('keyword', self.keyword)

            # 원래 콜백이 'db' 키워드를 받는지 확인한다.
            # 데이터베이스 핸들을 필요로 하지 않으면 무시.
            args = inspect.getargspec(context.callback)[0]
            if keyword not in args:
                return callback

            def wrapper(*args, **kwargs):
                # 데이터베이스에 연결
                db = sqlite3.connect(dbfile)
                # 이름으로 (row['column_name']) 컬럼 접근
                if dictrows: db.row_factory = sqlite3.Row
                # 키워드 인자에 연결 핸들 추가
                kwargs[keyword] = db

                try:
                    rv = callback(*args, **kwargs)
                    if autocommit: db.commit()
                except sqlite3.IntegrityError, e:
                    db.rollback()
                    raise HTTPError(500, "Database Error", e)
                finally:
                    db.close()
                return rv

            # 라우트 콜백을 포장된 버전으로 교체
            return wrapper

실제로 유용한 플러그인이며 보틀에 딸려 있는 것과 아주 비슷하다. 60행도 안 되는 코드 치고는 나쁘지 않다. 안 그런가? 다음이 사용례다. ::

    sqlite = SQLitePlugin(dbfile='/tmp/test.db')
    bottle.install(sqlite)

    @route('/show/:page')
    def show(page, db):
        row = db.execute('SELECT * from pages where name=?', page).fetchone()
        if row:
            return template('showpage', page=row)
        return HTTPError(404, "Page not found")

    @route('/static/:fname#.*#')
    def static(fname):
        return static_file(fname, root='/some/path')

    @route('/admin/set/:db#[a-zA-Z]+#', skip=[sqlite])
    def change_dbfile(db):
        sqlite.dbfile = '/tmp/%s.db' % db
        return "Switched DB to %s.db" % db

첫 번째 라우트에선 데이터베이스 연결이 필요하기 때문에 키워드 인자 ``db``\를 요청해서 플러그인에서 핸들을 만들게 한다. 두 번째 라우트에는 데이터베이스가 필요 없고, 그래서 플러그인에서 무시한다. 세 번째 라우트는 키워드 인자 'db'를 받긴 하지만 sqlite 플러그인을 명시적으로 건너뛴다. 이렇게 하면 플러그인에서 그 인자를 덮어 쓰지 않아서 같은 이름의 URL 인자 값이 그대로 오게 된다.

