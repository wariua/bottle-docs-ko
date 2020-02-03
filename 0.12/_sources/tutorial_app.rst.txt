.. _Bottle: http://bottle.paws.org
.. _Python: http://www.python.org
.. _SQLite: http://www.sqlite.org
.. _Windows: http://www.sqlite.org/download.html
.. _PySQLite: http://pypi.python.org/pypi/pysqlite/
.. _`데코레이터 문`: http://docs.python.org/glossary.html#term-decorator
.. _`Python DB API`: http://www.python.org/dev/peps/pep-0249/
.. _`WSGI 참조 서버`: http://docs.python.org/library/wsgiref.html#module-wsgiref.simple_server
.. _Cherrypy: http://www.cherrypy.org/
.. _Fapws3: http://github.com/william-os4y/fapws3
.. _Flup: http://trac.saddi.com/flup
.. _Paste: http://pythonpaste.org/
.. _Apache: http://www.apache.org
.. _`보틀 문서`: http://bottlepy.org/docs/dev/tutorial.html
.. _`mod_wsgi`: http://code.google.com/p/modwsgi/
.. _`json`: http://www.json.org

========================
따라하기: 할일 목록 응용
========================

.. note::

   작업 중인 문서이고 작성자는 `noisefloor <http://github.com/noisefloor>`_\다.


이 따라하기에선 Bottle_ WSGI 프레임워크를 간단히 소개할 것이다. 이 따라하기를 읽은 다음 보틀을 이용해 프로젝트를 만들 수 있게 하는 게 주된 목표다. 이 문서 안에서 모든 기능들을 보여 주지는 못하지만 적어도 라우팅, 보틀 템플릿 이용 출력 포매팅, GET / POST 매개변수 다루기 같은 중요한 것들은 살펴볼 것이다.

내용을 이해하기 위해 WSGI에 대한 기본 지식이 있어야 하는 건 아니다. 보틀이 하려는 게 바로 WSGI를 사용자에게서 떨어뜨리는 거다. 하지만 Python_ 프로그래밍 언어는 적당히 이해하고 있어야 한다. 그리고 이 따라하기의 예시에서 SQL 데이터베이스에 데이터를 저장하고 읽어 오니까 SQL에 대한 기본 지식이 도움이 되긴 하지만 보틀 개념을 이해하는 데 필수는 아니다. 여기선 SQLite_\를 쓴다. 일부 예시에선 보틀이 브라우저로 보내는 출력에 HTML으로 서식을 준다. 따라서 많이 쓰는 HTML 태그에 대한 기본 지식 역시 도움이 된다.

보틀을 소개한다는 취지에 따라 집중을 위해 내용 중간 중간의 파이썬 코드는 짧게 한다. 그리고 따라하기 내의 모든 코드는 동작은 잘 하지만 "야생에서" (가령 공개 웹 서버에서) 사용하기에 꼭 괜찮은 건 아니다. 그렇게 하려면 오류 처리 등을 더 추가하고, 데이터베이스를 패스워드로 보호하고, 입력을 검사하고 이스케이프 하는 것 등을 할 수 있다.

.. contents:: 차례
   :local:

목표
===========

이 따라하기가 끝나면 간단한 웹 기반 할일 목록 앱이 생기게 된다. 그 목록에는 항목마다 텍스트(최대 100글자)와 상태(0은 완료, 1은 미완료)가 있다. 웹 기반 사용자 인터페이스를 통해 미완료 항목을 보고 편집할 수 있으며 새 항목을 추가할 수 있다.

개발 중에는 모든 페이지를 ``localhost``\에서만 볼 수 있지만 이후에 Apache mod_wsgi 사용법을 포함해 응용을 "진짜" 서버에 얹는 방법을 보게 될 것이다.

보틀이 라우팅을 하고 템플릿을 이용해 서식 있는 출력을 만든다. 목록 항목들은 SQLite 데이터베이스에 저장된다. 데이터베이스를 읽고 쓰는 건 파이썬 코드로 한다.

최종적으로 다음 페이지와 기능을 가진 응용이 생기게 된다.

 * 시작 페이지 ``http://localhost:8080/todo``
 * 목록에 새 항목 추가하기: ``http://localhost:8080/new``
 * 항목 편집 페이지: ``http://localhost:8080/edit/:no``
 * @validate 데코레이터로 동적 라우트에 할당된 데이터 검증하기
 * 오류 잡기

시작하기 전에...
====================


.. rubric:: 보틀 설치

꽤 최근 파이썬(2.5 또는 이상 버전)이 설치돼 있다고 하면 거기에 보틀만 설치해 주면 된다. 보틀은 파이썬 자체 외의 다른 의존성이 없다.

직접 또는 파이썬 easy_install을 이용해 (``easy_install bottle``) 보틀을 설치할 수 있다.


.. rubric:: 추가 소프트웨어 요건

데이터베이스로 SQLite3을 쓰니까 설치가 돼 있어야 한다. 리눅스 시스템은 대부분 배포판에 기본으로 SQLite3가 설치돼 있다. 윈도우와 MacOS X에서도 SQLite3를 사용 가능하며 `sqlite3` 모듈이 파이썬 표준 라이브러리에 포함돼 있다.

.. rubric:: SQL 데이터베이스 만들기

일단 이후에 사용할 데이터베이스를 만들어야 한다. 다음 스크립트를 프로젝트 디렉터리에 저장하고 파이썬으로 실행하면 된다. 대화형 인터프리터를 쓸 수도 있다. ::

    import sqlite3
    con = sqlite3.connect('todo.db') # 경고: 현재 디렉터리에 파일이 생성된다
    con.execute("CREATE TABLE todo (id INTEGER PRIMARY KEY, task char(100) NOT NULL, status bool NOT NULL)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Read A-byte-of-python to get a good introduction into Python',0)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Visit the Python website',1)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Test various editors for and check the syntax highlighting',1)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Choose your favorite WSGI-Framework',0)")
    con.commit()

데이터베이스 `todo.db`\가 생기고 그 안의 ``todo``\라는 테이블에 컬럼 ``id``, ``task``, ``status``\가 있다. ``id``\는 각 행의 고유 ID이고 이후 행을 참조하는 데 쓴다. ``task`` 컬럼은 작업을 설명하는 텍스트를 담으며 최대 100글자 길이다. 마지막의 ``status`` 컬럼은 작업 미완료(1) 또는 완료(0)를 표시하는 데 쓴다.

보틀 이용해 웹 기반 할일 목록 만들기
================================================

이제 웹 기반 응용을 만들기 위해 보틀을 등장시킬 차례다. 하지만 먼저 보틀의 기본 개념 하나를 살펴볼 필요가 있다. 바로 라우트다.


.. rubric:: 라우트란

기본적으로 브라우저에 보이는 각 페이지는 페이지 주소가 호출될 때 동적으로 생성된다. 즉 정적 콘텐츠가 전혀 없다. 그게 바로 보틀에서 "라우트"라고 부르는 것, 즉 서버 상의 특정 주소다. 그래서 예를 들어 브라우저에서 페이지 ``http://localhost:8080/todo``\를 요청하면 보틀이 그 요청을 "잡아채서" 그 "todo"라는 라우트에 대해 정의된 (파이썬) 함수가 있는지 확인한다. 있으면 대응하는 파이썬 코드를 실행하고 그 결과를 반환하게 된다.


.. rubric:: 첫 단계 - 미완료 항목 모두 표시하기

라우트 개념을 이해했으니 첫 번째 라우트를 만들어 보자. 목표는 할일 목록에서 미완료 항목 전체를 보는 거다. ::

    import sqlite3
    from bottle import route, run

    @route('/todo')
    def todo_list():
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT id, task FROM todo WHERE status LIKE '1'")
        result = c.fetchall()
        return str(result)

    run()

위 코드를 ``todo.py``\로 저장하자. 이왕이면 ``todo.db`` 파일과 같은 디렉터리가 좋다. 아니라면 ``sqlite3.connect()`` 문의 ``todo.db`` 앞에 경로를 붙여 줘야 한다.

뭘 하는지 살펴보자. SQLite 데이터베이스에 접근하기 위해 필요한 모듈 ``sqlite3``\를 임포트 하고 보틀에서 ``route``\와 ``run``\을 임포트 한다. ``run()`` 문은 보틀에 포함된 웹 서버를 시작하는 거다. 기본으로 그 웹 서버는 localhost 8080 포트에서 페이지를 제공한다. 또 ``route``\를 임포트 했는데, 이건 보틀의 라우팅을 맡는 함수다. 보다시피 ``todo_list()``\라는 함수를 하나 정의해서 데이터베이스에서 읽어 오는 코드를 몇 줄 작성했다. 중요한 건 ``def todo_list()`` 문 바로 위에 있는 `데코레이터 문`_ ``@route('/todo')``\다. 그렇게 함수를 라우트 ``/todo``\에 결속시켜서 브라우저가 ``http://localhost:8080/todo``\를 호출할 때마다 보틀이 ``todo_list()`` 함수의 결과를 반환하게 된다. 이게 보틀 내에서 라우팅이 동작하는 방식이다.

한 함수에 라우트를 여러 개 결속시킬 수도 있다. 그래서 다음 코드도 잘 동작한다. ::

    @route('/todo')
    @route('/my_todo_list')
    def todo_list():
        ...

하지만 한 라우트를 여러 함수에 결속시키는 건 안 된다.

브라우저에서 보게 되는 건 반환된 내용, 즉 ``return`` 문이 내놓은 값이다. 이 예에선 ``result``\를 ``str()``\로 문자열로 변환해 줘야 한다. 보틀은 return 문에서 문자열 또는 문자열 리스트를 기대하는데 여기서 데이터베이스 질의의 결과물은 `Python DB API`_\에서 규정하는 대로 튜플의 리스트이기 때문이다.

이제 위 스크립트를 좀 이해했으니 직접 실행해서 결과를 살펴보자. 리눅스/유닉스 기반 시스템에서 ``todo.py`` 파일이 실행 가능해야 한다는 걸 잊지 말자. 그럼 ``python todo.py``\를 실행하고 브라우저에서 ``http://localhost:8080/todo`` 페이지를 호출하자. 스크립트를 입력하며 실수를 하지 않았다면 다음처럼 나올 것이다. ::

    [(2, u'Visit the Python website'), (3, u'Test various editors for and check the syntax highlighting')]

이렇게 나왔다면, 축하한다! 이제 엄연한 보틀 사용자가 됐다. 잘 동작하지 않아서 스크립트에 뭔가를 변경해야 한다면 페이지를 제공하는 보틀을 멈추는 걸 잊지 말자. 안 그러면 바뀐 버전이 올라가지 않는다.

그런데 출력이 그렇게 흥미롭지도 않고 읽기에 좋지도 않다. SQL 질의가 반환한 그대로의 결과라 그렇다.

그럼 다음 단계로 좀 보기 좋게 출력에 서식을 주자. 하지만 그 전에 일을 좀 쉽게 만들어 두자.


.. rubric:: 디버깅과 자동 재적재

스크립트에 뭔가 잘못된 게 있으면, 가령 데이터베이스 연결이 동작하지 않으면 보틀이 브라우저로 짧은 오류 메시지를 보낸다는 걸 벌써 알아챘을지도 모르겠다. 디버깅을 위해선 그보다 자세한 내용을 얻을 수 있으면 상당히 도움이 된다. 스크립트에 다음 문을 추가해 주기만 하면 된다. ::

    from bottle import run, route, debug
    ...
    # 끝에 이렇게 추가:
    debug(True)
    run()

"debug"를 켜면 파이썬 인터프리터의 전체 스택트레이스를 얻게 되는데, 일반적으로 버그를 찾는 데 필요한 유용한 정보가 담겨 있다. 또한 템플릿(아래 참고)을 캐싱 하지 않으므로 서버를 멈추지 않아도 템플릿 변경 효과가 바로 나타난다.

.. warning::

   ``debug(True)``\는 개발 용도로만 쓰기 위한 것이다. 운용 환경에서 사용해선 *안 된다*.



또 다른 멋진 기능으로 자동 재적재가 있는데, ``run()`` 문을 다음처럼 바꿔서 켤 수 있다.

::

    run(reloader=True)

스크립트 변화를 자동으로 탐지해서 새 버전을 다시 올려 준다. 그래서 한 번만 호출해 두면 서버를 멈추고 시작할 필요가 없다.

마찬가지로 이 기능은 기본적으로 개발 중에 쓰기 위한 것이지 운용 시스템을 위한 게 아니다.


.. rubric:: 보틀 템플릿으로 출력에 서식 주기

이제 스크립트 출력을 올바른 서식으로 바꾸는 걸 살펴보자.

사실 보틀이 하는 건 함수에게서 문자열 또는 문자열 리스트를 받아서 내장 서버의 도움을 받아 브라우저로 반환하는 것이다. 문자열 내용에는 신경쓰지 않으며, 그래서 HTML 마크업으로 서식을 준 텍스트도 가능하다.

보틀에는 쓰기 쉬운 템플릿 엔진이 딸려 있다. 템플릿은 ``.tpl`` 확장자의 별도 파일로 저장한다. 그러면 함수 안에서 템플릿을 호출할 수 있다. 템플릿에는 어떤 텍스트도 들어갈 수 있다. (대부분 HTML 마크업에 파이썬 문이 섞인 형태일 것이다.) 그리고 템플릿에 인자를 줄 수 있다. 가령 데이터베이스 질의 결과를 주면 템플릿 안에서 멋지게 서식이 더해진다.

이제 미완료 할일 항목들을 보여 주는 질의 결과를 두 열짜리 간단한 테이블로 바꿀 것이다. 첫 번째 열에는 항목 ID, 두 번째 열에는 텍스트가 들어간다. 결과 집합은 위에서 본 것처럼 튜플들의 리스트고 각 튜플이 항목 하나씩을 담고 있다.

예시에 템플릿을 포함시키려면 다음 행을 추가해 주기만 하면 된다. ::

    from bottle import route, run, debug, template
    ...
    result = c.fetchall()
    c.close()
    output = template('make_table', rows=result)
    return output
    ...

두 가지를 하고 있다. 첫째로 템플릿을 사용할 수 있도록 보틀에서 ``template``\을 임포트 한다. 둘째로 템플릿 ``make_table``\의 출력을 변수 ``output``\에 할당해서 반환한다. 템플릿을 호출할 때는 데이터베이스 질의로 받은 ``result``\를 변수 ``rows``\로 할당하는데, 템플릿 내에서 그 변수를 사용한다. 필요하면 템플릿에 여러 변수/값을 할당할 수도 있다.

템플릿은 항상 문자열의 리스트를 반환하며, 따라서 어떤 변환도 필요치 않다. 당연히 ``return template('make_table', rows=result)``\라고 작성해서 한 행을 줄일 수도 있고, 위와 정확히 같은 결과가 나온다.

이제 해당 템플릿을 작성할 차례다. ::

    %# 튜플의 리스트(또는 리스트의 리스트, 튜플의 튜플, ...)로 HTML 테이블을 생성하는 템플릿
    <p>The open items are as follows:</p>
    <table border="1">
    %for row in rows:
      <tr>
      %for col in row:
        <td>{{col}}</td>
      %end
      </tr>
    %end
    </table>

``todo.py``\와 같은 디렉터리에 위 코드를 ``make_table.tpl``\로 저장하자.

코드를 살펴보자. %로 시작하는 행은 모두 파이썬 코드로 해석한다. 물론 유효한 파이썬 문만 허용되며, 아니면 여느 파이썬 코드와 마찬가지로 템플릿에서 예외를 던진다. 다른 행들은 평범한 HTML 마크업이다.

보다시피 ``rows``\를 순회하기 위해 파이썬 ``for`` 문을 두 번 사용하고 있다. 앞서 보았듯 ``rows``\는 데이터베이스 질의 결과를 담고 있는 변수고, 그래서 튜플들의 리스트다. 첫 번째 ``for`` 문은 리스트 내의 튜플에 접근하는 것이고 두 번째는 튜플 내의 항목에 접근하는 것이다. 그렇게 해서 각 항목을 테이블 셸에 집어넣는다. ``for``, ``if``, ``while`` 등의 문을 모두 ``%end``\로 닫아 주는 걸 잊지 말자. 안 그러면 원하는 결과가 나오지 않을 수 있다.

템플릿의 파이썬 코드 아닌 행에서 변수에 접근해야 한다면 이중 중괄호로 감싸면 된다. 그러면 템플릿에서 그 변수의 실제 값을 대신 집어넣어 준다.

스크립트를 다시 실행해서 결과를 보자. 아직 아주 멋진 건 아니지만 적어도 튜플 리스트보단 읽기에 좋다. 당연히 위의 초간단 HTML 마크업에다가 가령 인라인 스타일로 양념을 쳐서 더 보기 좋은 출력을 얻을 수도 있다.


.. rubric:: GET과 POST의 값 사용하기

미완료 항목 전체를 제대로 볼 수 있게 됐으니 다음 단계로 가 보자. 할일 목록에 새 항목을 추가하는 거다. 일반 HTML 기반 양식으로 새 항목을 받게 되는데 GET 메소드로 데이터를 받는다.

먼저 스크립트에 새 라우트를 설치하고 GET 데이터를 받을 거라고 라우트에 지정하자. ::

    from bottle import route, run, debug, template, request
    ...
    return template('make_table', rows=result)
    ...

    @route('/new', method='GET')
    def new_item():

        new = request.GET.get('task', '').strip()

        conn = sqlite3.connect('todo.db')
        c = conn.cursor()

        c.execute("INSERT INTO todo (task,status) VALUES (?,?)", (new,1))
        new_id = c.lastrowid

        conn.commit()
        c.close()

        return '<p>The new task was inserted into the database, the ID is %s</p>' % new_id

GET (또는 POST) 데이터에 접근하려면 보틀에서 ``request``\를 임포트 해야 한다. 실제 데이터를 변수에 할당하기 위해 ``request.GET.get('task', '').strip()`` 문을 쓰는데, 여기서 ``task``\는 접근하려는 GET 데이터의 이름이다. 이게 전부다. GET 데이터 값이 여러 개라면 ``request.GET.get()`` 문을 여러 번 써서 다른 변수에 할당할 수도 있다.

이 코드 조각 나머지 부분에선 얻은 데이터를 처리한다. 데이터베이스에 기록하고, 대응하는 ID를 데이터베이스에서 얻고, 출력을 만들어 낸다.

그런데 GET 데이터는 어디서 오는 걸까? 일단, 양식이 들어 있는 고정된 HTML 페이지를 이용할 수 있다. 아니면 지금 우리가 할 것처럼 GET 데이터 없이 ``/new`` 라우트를 호출했을 때 템플릿으로 출력할 수도 있다.

다음처럼 코드를 확장해야 한다. ::

    ...
    @route('/new', method='GET')
    def new_item():

        if request.GET.get('save','').strip():

            new = request.GET.get('task', '').strip()
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()

            c.execute("INSERT INTO todo (task,status) VALUES (?,?)", (new,1))
            new_id = c.lastrowid

            conn.commit()
            c.close()

            return '<p>The new task was inserted into the database, the ID is %s</p>' % new_id
        else:
            return template('new_task.tpl')


``new_task.tpl``\은 다음과 같다. ::

    <p>Add a new task to the ToDo list:</p>
    <form action="/new" method="GET">
    <input type="text" size="100" maxlength="100" name="task">
    <input type="submit" name="save" value="save">
    </form>

이게 전부다. 보다시피 이번 템플릿은 그냥 HTML이다.

이제 할일 목록을 늘여 나갈 수 있다.

한편으로 POST 데이터를 선호한다면, 돌아가는 방식은 동일하니까 ``request.POST.get()``\으로 써 주기만 하면 된다.


.. rubric:: 기존 항목 편집하기

마지막은 기존 항목을 편집할 수 있게 하는 거다.

우리가 알고 있는 라우트만 이용해도 가능하긴 하지만 꽤 까다로울 수 있다. 하지만 보틀에 "동적 라우트"라는 게 있어서 이 작업이 상당히 쉬워진다.

동적 라우트의 기본 형식은 다음과 같다. ::

    @route('/myroute/:something')

여기서 중요한 건 콜론이다. ``:something``\에 대해 다음 슬래시까지 어떤 문자열이든 받도록 한다. 그리고 ``something``\의 값이 그 라우트에 할당된 함수로 전달돼서 그 데이터를 함수 안에서 처리할 수 있다.

할일 목록을 위해 ``@route('/edit/:no)`` 라우트를 만들 것이다. 여기서 ``no``\는 편집할 항목의 ID다.

코드는 다음과 같다. ::

    @route('/edit/:no', method='GET')
    def edit_item(no):

        if request.GET.get('save','').strip():
            edit = request.GET.get('task','').strip()
            status = request.GET.get('status','').strip()

            if status == 'open':
                status = 1
            else:
                status = 0

            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("UPDATE todo SET task = ?, status = ? WHERE id LIKE ?", (edit, status, no))
            conn.commit()

            return '<p>The item number %s was successfully updated</p>' % no
        else:
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("SELECT task FROM todo WHERE id LIKE ?", (str(no)))
            cur_data = c.fetchone()

            return template('edit_task', old=cur_data, no=no)

``GET`` 데이터를 얻는 것을 포함해, 기본적으로 앞서 새 항목을 추가할 때와 거의 동일하다. 여기 추가된 건 동적 라우트 ``:no``\를 쓰는 것인데, 그 번호가 대응하는 함수로 전달된다. 보다시피 함수 안에서 ``no``\를 사용해 데이터베이스의 해당 데이터 행에 접근한다.

함수에서 호출하는 ``edit_task.tpl`` 템플릿은 다음과 같다. ::

    %# 할일 편집용 템플릿
    %# 이 템플릿은 "no" 값뿐 아니라 해당 할일 항목의 텍스트인 "old"도 받는다.
    <p>Edit the task with ID = {{no}}</p>
    <form action="/edit/{{no}}" method="get">
    <input type="text" name="task" value="{{old[0]}}" size="100" maxlength="100">
    <select name="status">
    <option>open</option>
    <option>closed</option>
    </select>
    <br/>
    <input type="submit" name="save" value="save">
    </form>

앞서 설명한 것처럼 이번에도 템플릿에 파이썬 문과 HTML이 섞여 있다.

동적 라우트에 대해 하나만 더 얘기하자면, 좀 있다 보겠지만 동적 라우트에 정규 표현식까지 쓸 수 있다.


.. rubric:: 동적 라우트 검사하기

동적 라우트를 쓰는 건 좋은데 많은 경우 라우트에서 동적인 부분을 검사할 필요가 있다. 예를 들어 위의 편집용 라우트에선 정수를 기대한다. 그런데 부동소수점수나 문자 등을 수신하면 파이썬 인터프리터가 예외를 던지게 되는데, 이는 바라는 방식이 아니다.

그런 경우를 위해 보틀에서 ``@validate`` 데코레이터를 제공하는데, "입력"을 검사한 다음 함수로 전달한다. 검사를 적용하려면 코드를 다음처럼 확장하면 된다. ::

    from bottle import route, run, debug, template, request, validate
    ...
    @route('/edit/:no', method='GET')
    @validate(no=int)
    def edit_item(no):
    ...

먼저 보틀 프레임워크에서 ``validate``\를 임포트 한 다음 @validate 데코레이터를 적용하면 된다. 여기선 ``no``\가 정수인지 검사한다. 기본적으로 부동소수점수, 리스트 등의 모든 데이터 타입을 검사할 수 있다.

코드를 저장하고 ``:no``\에 "403 forbidden" 유발 값(가령 부동소수점수)을 줘서 다시 페이지를 호출해 보자. 그러면 예외가 아니라 정수가 필요하다는 "403 - Forbidden" 오류를 받게 된다.

.. rubric:: 정규 표현식을 이용한 동적 라우트

보틀에선 라우트의 "동적인 부분"이 정규 표현식인 동적 라우트도 처리할 수 있다.

예를 들어서 할일 목록의 각 항목을 "item1" 같은 형식의 번호로 접근할 수 있어야 한다고 해 보자. 일단 항목마다 라우트를 만들고 싶진 않을 것이다. 그리고 단순한 동적 라우트로도 안 된다. 라우트에 고정으로 "item"이 포함돼 있기 때문이다.

이미 얘기한 것처럼 정규 표현식이 해결책이다. ::

    @route('/item:item#[0-9]+#')
    def show_item(item):
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT task FROM todo WHERE id LIKE ?", (item))
        result = c.fetchall()
        c.close()
        if not result:
            return 'This item number does not exist!'
        else:
            return 'Task: %s' %result[0]

물론 이 예는 좀 작위적이다. 단순 동적 라우트에 검사를 추가해서 쓰는 게 더 쉬울 것이다. 단지 정규 표현식 라우트가 어떻게 동작하는지 보이려는 것이다. ``@route('/item:item#[0-9]+#')`` 행 앞쪽은 보통 라우트와 비슷하지만 #로 감싼 부분이 정규 표현식으로 해석돼서 라우트의 동적인 부분이 된다. 그래서 이 경우엔 0에서 9까지 숫자가 걸리게 하려는 것이다. 그 아래의 함수 "show_item"에서는 해당 항목이 데이터베이스에 있는지 여부를 확인한다. 있으면 그 할일의 텍스트를 반환한다. 보다시피 라우트의 정규 표현식 부분만 전달된다. 그리고 항상 문자열로 전달된다. 이 경우처럼 단순 정수인 경우도 마찬가지다.


.. rubric:: 정적 파일 반환하기

라우트를 파이썬 함수에 연계시킬 필요는 없고 정적 파일만 반환하면 되는 경우가 있을 수 있다. 예를 들어 응용의 도움말 페이지가 있다면 그 페이지를 HTML 그대로 반환하고 싶을 수 있다. 다음처럼 하면 된다. ::

    from bottle import route, run, debug, template, request, validate, static_file

    @route('/help')
    def help():
        return static_file('help.html', root='/path/to/file')

일단 보틀에서 ``static_file`` 함수를 임포트 해야 한다. 그리고 보다시피 ``return`` 문이 ``return static_file`` 문으로 바뀐다. 적어도 두 인자가 필요한데, 반환할 파일의 이름과 파일 경로다. 파일이 응용과 같은 디렉터리에 있더라도 경로를 지정해야 한다. 하지만 그 경우 경로로 ``'.'``\를 쓸 수도 있다. 파일의 MIME 타입을 보틀에서 추측하지만 명시적으로 지정하고 싶다면 ``static_file``\에 ``mimetype='text/html'``\처럼 세 번째 인자를 더해 주면 된다. 동적 라우트를 포함한 어떤 종류의 라우트에도 ``static_file``\을 쓸 수 있다.


.. rubric:: JSON 데이터 반환하기

응용에서 출력 페이지를 직접 생성하는 게 아니라 이후 자바스크립트 등에서 처리할 데이터를 반환하고 싶은 경우가 있을 수 있다. 그런 경우를 위해 보틀에선 웹 응용 간 데이터 교환의 거의 표준인 JSON 객체를 반환하는 걸 지원한다. 파이썬을 포함한 여러 프로그래밍 언어에서 JSON을 처리할 수 있다.

정규 표현식 라우트 예시에서 생성하는 데이터를 JSON 객체로 반환하고 싶다고 해 보자. 코드가 다음처럼 된다. ::

    @route('/json:json#[0-9]+#')
    def show_json(json):
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT task FROM todo WHERE id LIKE ?", (json))
        result = c.fetchall()
        c.close()

        if not result:
            return {'task':'This item number does not exist!'}
        else:
            return {'Task': result[0]}

보다시피 상당히 간단하다. 그냥 파이썬 딕셔너리를 반환하면 보틀이 자동으로 JSON 객체로 변환해서 보낸다. 그래서 가령 "http://localhost/json1"을 호출하면 보틀이 JSON 객체 ``{"Task": ["Read A-byte-of-python to get a good introduction into Python"]}``\을 반환하게 된다.



.. rubric:: 오류 잡기

다음 단계는 보틀 자체에서 오류를 잡아서 응용 사용자에게 어떤 오류 메시지도 보이지 않게 하는 것이다. 이를 위해 보틀에는 HTTP 오류에 할당할 수 있는 "오류 라우트"가 있다.

우리는 403 오류를 잡고 싶다. 코드는 다음과 같다. ::

    from bottle import error

    @error(403)
    def mistake(code):
        return 'The parameter you passed has the wrong format!'

즉 먼저 보틀에서 ``error``\를 임포트 하고 ``error(403)``\으로 라우트를 정의하면 "403 forbidden" 오류를 모두 잡아 준다. 그 오류에 "mistake" 함수가 할당됐다. 참고로 ``error()``\는 필요하든 말든 항상 오류 코드를 전달해 준다. 따라서 함수에서 항상 인자를 한 개 받아야 한다. 안 그러면 동작하지 않는다.

마찬가지로 한 함수에 여러 오류 라우트를 할당할 수도 있고 여러 오류를 각각의 함수로 잡을 수도 있다. 즉 다음 코드도 잘 동작하고, ::

    @error(404)
    @error(403)
    def mistake(code):
        return 'There is something wrong!'

다음 코드도 잘 동작한다. ::

    @error(403)
    def mistake403(code):
        return 'The parameter you passed has the wrong format!'

    @error(404)
    def mistake404(code):
        return 'Sorry, this page does not exist!'


.. rubric:: 정리

지금까지 내용을 모두 읽었으면 보틀이라는 WSGI 프레임워크가 어떻게 동작하는지 대략 이해할 수 있을 것이다. 그리고 각자의 응용에 보틀을 이용하기 위해 필요한 지식을 모두 얻었다.

다음 장에선 더 큰 프로젝트에서 보틀을 쓰는 법을 간단히 소개한다. 그리고 높은 부하, 즉 대량의 웹 트래픽 하에서 더 잘 동작하는 웹 서버들과 함께 보틀을 돌리는 방법을 살펴볼 것이다.

서버 구성
================================

지금까지 보틀의 표준 서버를 사용했는데, 파이썬에 딸려 있는 `WSGI 참조 서버`_\다. 개발 용도에는 잘 맞지만 큰 응용에는 적합하지 않다. 하지만 다른 방법들을 보기 전에 표준 서버의 설정을 바꾸는 방법부터 살펴보자.


.. rubric:: 다른 포트와 IP로 보틀 돌리기

기본적으로 보틀은 ``localhost``\라고도 하는 IP 주소 127.0.0.1에서 ``8080`` 포트로 페이지를 제공한다. 그 설정을 바꾸는 건 아주 간단하다. 보틀의 ``run()`` 함수에 추가 매개변수를 줘서 포트와 주소를 바꿀 수 있다.

포트를 바꾸려면 run 명령에 ``port=포트번호``\를 추가하면 된다. 예를 들어 다음처럼 하면 보틀이 80 포트에 리슨 하게 된다. ::

    run(port=80)

보틀이 리슨 하는 IP 주소를 바꾸려면 다음처럼 한다. ::

    run(host='123.45.67.89')

물론 두 매개변수를 함께 쓸 수도 있다. ::

   run(port=80, host='123.45.67.89')

다음 절의 설명처럼 다른 서버로 보틀을 돌릴 때도 ``port``\와 ``host`` 매개변수를 적용할 수 있다.


.. rubric:: 다른 서버로 보틀 돌리기

앞서 얘기한 것처럼 개발 용도나 개인 내지 소수 사람들만 보틀 기반 응용을 쓸 때는 표준 서버가 잘 맞는다. 하지만 규모가 커지면 표준 서버가 병목이 될 수 있다. 단일 스레드라서 한 번에 한 요청만 처리할 수 있기 때문이다.

보틀에는 높은 부하에서 잘 동작하는 다중 스레드 서버들에 대한 어댑터가 이미 다양하게 갖춰져 있다. Cherrypy_, Fapws3_, Flup_, Paste_\를 지원한다.

예를 들어 Paste 서버로 보틀을 돌리고 싶으면 다음 코드를 쓰면 된다. ::

    from bottle import PasteServer
    ...
    run(server=PasteServer)

``FlupServer``, ``CherryPyServer``, ``FapwsServer``\도 마찬가지로 동작한다.


.. rubric:: Apache에서 mod_wsgi로 보틀 돌리기

이미 Apache_\를 쓰고 있을 수도 있고 보틀 기반 응용을 큰 규모로 돌리고 싶을 수도 있다. 그렇다면 Apache와 mod_wsgi_\를 고려해 볼 차례다.

Apache 서버가 올라가 있고 mod_wsgi도 잘 동작하고 있다고 가정한다. 여러 리눅스 배포판에서 각각의 패키지 관리 시스템을 통해 mod_wsgi를 쉽게 설치할 수 있다.

보틀에 mod_wsgi를 위한 어댑터가 딸려 있으므로 쉽게 응용을 올릴 수 있다.

아래 예에서는 "할일 목록" 응용을 ``http://www.mypage.com/todo``\를 통해 접근할 수 있도록 하고 코드와 템플릿, SQLite 데이터베이스를 ``/var/www/todo`` 경로에 저장한다고 가정한다.

mod_wsgi를 통해 응용을 돌릴 때 꼭 해야 할 일이 코드에서 ``run()`` 문을 빼는 것이다. 안 그러면 제대로 동작하지 않는다.

그 다음엔 다음 내용으로 ``adapter.wsgi``\라는 파일을 만들자. ::

    import sys, os, bottle

    sys.path = ['/var/www/todo/'] + sys.path
    os.chdir(os.path.dirname(__file__))

    import todo # 응용 적재

    application = bottle.default_app()

그리고 같은 경로 ``/var/www/todo``\에 저장하자. 사실 확장자가 ``.wsgi``\기만 하면 파일 이름은 뭐든 괜찮다. 가상 호스트에서 파일을 가리키는 데만 이름이 쓰인다.

마지막으로 Apache 설정에 다음처럼 가상 호스트 설정을 추가해 줘야 한다. ::

    <VirtualHost *>
        ServerName mypage.com

        WSGIDaemonProcess todo user=www-data group=www-data processes=1 threads=5
        WSGIScriptAlias / /var/www/todo/adapter.wsgi

        <Directory /var/www/todo>
            WSGIProcessGroup todo
            WSGIApplicationGroup %{GLOBAL}
            Order deny,allow
            Allow from all
        </Directory>
    </VirtualHost>

서버를 재시작하고 나면 ``http://www.mypage.com/todo``\에서 할일 목록을 볼 수 있을 것이다.

끝으로
=========================

보틀 소개 및 따라하기가 이제 끝났다. 보틀의 기본 개념들을 배우고 보틀 프레임워크를 이용해 첫 번째 응용을 작성했다. 그리고 큰 작업에 맞게 보틀을 조정하는 방법과 Apache 웹 서버에서 mod_wsgi로 보틀을 돌리는 방법을 살펴봤다.

시작할 때 말한 것처럼 이 따라하기는 보틀의 모든 면과 가능성을 보여 주지 못한다. 가령 파일 객체 및 스트림 받기와 인증 데이터 다루기를 여기선 건너뛰었다. 또한 템플릿 안에서 다른 템플릿을 호출할 수 있다는 것도 보여 주지 않았다. 그런 사항에 대한 소개는 `보틀 문서`_\를 참고하면 된다.

전체 예시 코드
=========================

조금씩 조금씩 개발한 할일 목록 예시의 전체 코드다.

주된 응용 코드 ``todo.py``::

    import sqlite3
    from bottle import route, run, debug, template, request, validate, static_file, error

    # mod_wsgi 위에서 돌릴 때만 필요
    from bottle import default_app

    @route('/todo')
    def todo_list():

        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT id, task FROM todo WHERE status LIKE '1';")
        result = c.fetchall()
        c.close()

        output = template('make_table', rows=result)
        return output

    @route('/new', method='GET')
    def new_item():

        if request.GET.get('save','').strip():

            new = request.GET.get('task', '').strip()
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()

            c.execute("INSERT INTO todo (task,status) VALUES (?,?)", (new,1))
            new_id = c.lastrowid

            conn.commit()
            c.close()

            return '<p>The new task was inserted into the database, the ID is %s</p>' % new_id

        else:
            return template('new_task.tpl')

    @route('/edit/:no', method='GET')
    @validate(no=int)
    def edit_item(no):

        if request.GET.get('save','').strip():
            edit = request.GET.get('task','').strip()
            status = request.GET.get('status','').strip()

            if status == 'open':
                status = 1
            else:
                status = 0

            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("UPDATE todo SET task = ?, status = ? WHERE id LIKE ?", (edit,status,no))
            conn.commit()

            return '<p>The item number %s was successfully updated</p>' %no

        else:
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("SELECT task FROM todo WHERE id LIKE ?", (str(no)))
            cur_data = c.fetchone()

            return template('edit_task', old = cur_data, no = no)

    @route('/item:item#[0-9]+#')
    def show_item(item):

            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("SELECT task FROM todo WHERE id LIKE ?", (item))
            result = c.fetchall()
            c.close()

            if not result:
                return 'This item number does not exist!'
            else:
                return 'Task: %s' %result[0]

    @route('/help')
    def help():

        static_file('help.html', root='.')

    @route('/json:json#[0-9]+#')
    def show_json(json):

        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT task FROM todo WHERE id LIKE ?", (json))
        result = c.fetchall()
        c.close()

        if not result:
            return {'task':'This item number does not exist!'}
        else:
            return {'Task': result[0]}


    @error(403)
    def mistake403(code):
        return 'There is a mistake in your url!'

    @error(404)
    def mistake404(code):
        return 'Sorry, this page does not exist!'


    debug(True)
    run(reloader=True)
    # 응용을 개발 환경에서 운용 환경으로 옮길 때 reloader=True와 debug(True)를 반드시 제거할 것

템플릿 ``make_table.tpl``::

    %# 튜플의 리스트(또는 리스트의 리스트, 튜플의 튜플, ...)로 HTML 테이블을 생성하는 템플릿
    <p>The open items are as follows:</p>
    <table border="1">
    %for row in rows:
      <tr>
      %for col in row:
        <td>{{col}}</td>
      %end
      </tr>
    %end
    </table>

템플릿 ``edit_task.tpl``::

    %# 할일 편집용 템플릿
    %# 이 템플릿은 "no" 값뿐 아니라 해당 할일 항목의 텍스트인 "old"도 받는다.
    <p>Edit the task with ID = {{no}}</p>
    <form action="/edit/{{no}}" method="get">
    <input type="text" name="task" value="{{old[0]}}" size="100" maxlength="100">
    <select name="status">
    <option>open</option>
    <option>closed</option>
    </select>
    <br/>
    <input type="submit" name="save" value="save">
    </form>

템플릿 ``new_task.tpl``::

    %# 새 작업을 위한 양식 템플릿
    <p>Add a new task to the ToDo list:</p>
    <form action="/new" method="GET">
    <input type="text" size="100" maxlength="100" name="task">
    <input type="submit" name="save" value="save">
    </form>
