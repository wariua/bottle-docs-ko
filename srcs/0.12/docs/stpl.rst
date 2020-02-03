===================
SimpleTemplate 엔진
===================

.. currentmodule:: bottle

보틀에는 빠르고 강력하며 배우기 쉬운 내장 템플릿 엔진 *SimpleTemplate*\(줄여서 *stpl*)이 딸려 있다. :func:`view` 및 :func:`template` 헬퍼에서 쓰는 기본 엔진이기도 하고 단독 범용 템플릿 엔진으로도 쓸 수 있다. 이 문서에서는 템플릿 문법을 설명하고 흔한 사용 예를 살펴본다.

.. rubric:: 기본적인 API 사용:

:class:`SimpleTemplate`\은 :class:`BaseTemplate` API를 구현하고 있다. ::

   >>> from bottle import SimpleTemplate
   >>> tpl = SimpleTemplate('Hello {{name}}!')
   >>> tpl.render(name='World')
   u'Hello World!'

간결함을 위해 이 문서 예시에서는 :func:`template` 헬퍼를 쓴다. ::

   >>> from bottle import template
   >>> template('Hello {{name}}!', name='World')
   u'Hello World!'

:func:`template` 헬퍼에선 드러나지 않지만 템블릿을 컴파일 하는 것과 렌더링 하는 게 별도 동작이라는 것만 기억해 두자. 일반적으로 템플릿 컴파일은 한 번만 이뤄지고 내부적으로 캐싱 되지만 렌더링은 다양한 키워드 인자들로 여러 번 이뤄진다.

:class:`SimpleTemplate` 문법
============================

파이썬은 아주 강력한 언어지만 공백 문자에 민감한 문법 때문에 템플릿 언어로 쓰기 어렵다. SimpleTemplate은 그런 제약 몇 가지를 없애서 파이썬 언어의 기능과 라이브러리, 속도를 그대로 이용하면서도 깔끔하고 읽기 좋으며 유지하기 쉬운 템플릿을 작성할 수 있게 해 준다.

.. warning::

   :class:`SimpleTemplate` 문법이 그대로 파이썬 바이트코드로 컴파일 돼서 :meth:`SimpleTemplate.render` 호출 때마다 실행된다. 신뢰할 수 없는 템플릿은 렌더링 하지 말자! 해로운 파이썬 코드를 포함하거나 실행할 수 있다.

인라인 식
---------

위의 "Hello World!" 예에서 이미 ``{{...}}`` 문법 사용법을 배웠다. 하지만 그게 전부가 아니다. 그 중괄호 안에는 문자열이나 문자열 표현을 가진 뭔가로 평가되기만 하면 어떤 파이썬 식이든 허용된다. ::

  >>> template('Hello {{name}}!', name='World')
  u'Hello World!'
  >>> template('Hello {{name.title() if name else "stranger"}}!', name=None)
  u'Hello stranger!'
  >>> template('Hello {{name.title() if name else "stranger"}}!', name='mArC')
  u'Hello Marc!'

내부의 파이썬 식은 렌더링 시점에 실행되며 :meth:`SimpleTemplate.render` 메소드에 전달된 모든 키워드 인자에 접근할 수 있다. `XSS <http://en.wikipedia.org/wiki/Cross-Site_Scripting>`_ 공격을 방지하기 위해 HTML 특수 문자를 자동으로 이스케이프 한다. 식 앞에 느낌표를 붙여서 이스케이프 처리를 끌 수 있다. ::

  >>> template('Hello {{name}}!', name='<b>World</b>')
  u'Hello &lt;b&gt;World&lt;/b&gt;!'
  >>> template('Hello {{!name}}!', name='<b>World</b>')
  u'Hello <b>World</b>!'

파이썬 코드 내장
----------------

.. highlight:: html+django

템플릿 엔진에선 템플릿 안에 파이썬 코드 행이나 블록을 내장하는 걸 허용한다. 코드 행은 ``%``\로 시작하고 코드 블록은 ``<%`` 및 ``%>`` 토큰으로 감싼다. ::

  % name = "Bob"  # 파이썬 코드 행
  <p>Some plain text in between</p>
  <%
    # 파이썬 코드 블록
    name = name.title().strip()
  %>
  <p>More plain text</p>

내장된 파이썬 코드는 정규 파이썬 문법을 따르되 두 가지 추가 문법 규칙이 있다.

* **들여쓰기가 무시된다.** 문 앞에 원하는 만큼 공백을 넣을 수 있다. 그래서 코드를 위아래 마크업에 맞게 정렬해서 가독성을 크게 개선할 수 있다.
* 원래는 들여쓰기로 끝을 표시하던 블록을 명시적으로 ``end`` 키워드로 닫아야 한다.

::

  <ul>
    % for item in basket:
      <li>{{item}}</li>
    % end
  </ul>

``%``\와 ``<%`` 토큰 모두 그 행에 처음 있는 공백 아닌 문자인 경우에만 인식된다. 템플릿 마크업 중간에 등장하는 경우에는 이스케이프 해 줄 필요가 없다. 텍스트의 어느 행이 이 토큰들 중 하나로 시작하는 경우에만 백슬래시로 이스케이프 해 줘야 한다. 백슬래시 + 토큰 조합이 행 처음의 마크업에 등장하는 드문 경우도 인라인 식에 문자열 리터럴을 넣어서 어떻게든 처리할 수 있다. ::

  이 행에는 %와 <%가 있지만 파이썬 코드는 없다.
  \% 이 텍스트 행은 '%' 토큰으로 시작한다.
  \<% 토큰으로 시작하지만 텍스트로 렌더링 되는 또 다른 행.
  {{'\\%'}} 이 행은 이스케이프 된 토큰으로 시작한다.

이스케이프 할 게 많다면 :ref:`커스텀 토큰 <stpl-custom-tokens>` 사용을 고려해 보자.

공백 문자 제어
--------------

코드 블록과 코드 행이 항상 행 전체를 차지한다. 코드 조각 앞과 뒤의 공백 문자는 제거된다. 즉 내장 코드 때문에 템플릿 안에 빈 행이나 뜬금없는 공백이 생기지 않는다. ::

  <div>
   % if True:
    <span>content</span>
   % end
  </div>

이게 깔끔하고 간결한 HTML로 렌더링 된다. ::

  <div>
    <span>content</span>
  </div>

하지만 코드 내장으로 인해 항상 새 행이 시작되는데, 렌더링 된 템플릿에서 그렇게 되지 않는 걸 원할 수도 있다. 코드 조각 앞의 개행을 없애려면 텍스트 행을 백슬래시 두 개로 끝내면 된다. ::

  <div>\\
   %if True:
  <span>content</span>\\
   %end
  </div>

그럼 템플릿이 다음처럼 렌더링 된다. ::

  <div><span>content</span></div>

코드 조각 바로 앞에서만 이 방법이 통한다. 다른 곳에선 공백을 직접 제어할 수 있으므로 특별한 문법이 필요치 않다.

템플릿 함수
===========

각 템플릿에는 흔한 용도 대부분에 도움이 되는 함수들이 갖춰져 있다. 그 함수들을 언제나 사용 가능하다. 즉 임포트 하거나 직접 제공해 줄 필요가 없다. 그걸로 다룰 수 없는 일에는 사용 가능한 괜찮은 파이썬 라이브러리가 있을 것이다. 템플릿 내에서 원하는 뭐든 ``import`` 할 수 있다는 걸 기억하자. 결국은 파이썬 프로그램이다.

.. currentmodule:: stpl

.. versionchanged:: 0.12
   이 릴리스 전에서 :func:`include`\와 :func:`rebase`\는 함수가 아니라 문법적 키워드였다.

.. function:: include(sub_template, **variables)

  지정한 변수들로 하위 템플릿을 렌더링 하고 결과 텍스트를 현재 템플릿에 집어넣는다. 하위 템플릿으로 전달됐거나 그 안에서 정의된 지역 변수들을 담은 딕셔너리를 함수가 반환한다. ::

    % include('header.tpl', title='Page Title')
    Page Content
    % include('foother.tpl')

.. function:: rebase(name, **variables)

  현재 템플릿이 이후 다른 템플릿에 포함될 거라고 표시해 둔다. 현재 템플릿이 렌더링 된 후 그 결과 텍스트가 ``base``\라는 이름에 저장돼서 베이스 템플릿으로 전달되고, 베이스 템플릿이 렌더링 된다. 이를 이용해 템플릿을 다른 텍스트로 `감싸거나` 다른 템플릿 엔진에서 볼 수 있는 상속 기능을 흉내낼 수 있다. ::

    % rebase('base.tpl', title='Page Title')
    <p>Page Content ...</p>

  이를 다음 ``base.tpl``\과 결합할 수 있다. ::

    <html>
    <head>
      <title>{{title or 'No title'}}</title>
    </head>
    <body>
      {{base}}
    </body>
    </html>


템플릿 내에서 정의 안 된 변수에 접근하면 :exc:`NameError`\를 던지고 렌더링이
즉시 중단된다. 이건 표준적인 파이썬 동작이고 새로울 게 없다. 하지만 바닐라
파이썬에는 변수 사용 가능 여부를 확인할 수 있는 간단한 방법이 없다. 그래서
유연한 입력을 지원하려 하거나 같은 템플릿을 여러 상황에 이용하려 할 때
그 때문에 짜증이 나기 쉽다. 그럴 때 다음 함수들이 도움이 될 수 있다.

.. function:: defined(name)

    현재 템플릿 네임스페이스 내에 그 변수가 정의돼 있으면 True를 반환하고,
    아니면 False를 반환한다.

.. function:: get(name, default=None)

    변수를 반환하고, 없으면 기본값을 반환한다.

.. function:: setdefault(name, default)

    변수가 정의돼 있지 않으면 주어진 기본값으로 만든다.
    그 변수를 반환한다.

다음은 세 함수를 모두 사용해서 선택적 템플릿 변수를 다른 방식으로 구현한
예이다. ::

    % setdefault('text', 'No Text')
    <h1>{{get('title', 'No Title')}}</h1>
    <p> {{ text }} </p>
    % if defined('author'):
      <p>By {{ author }}</p>
    % end

.. currentmodule:: bottle


:class:`SimpleTemplate` API
==============================

.. autoclass:: SimpleTemplate
   :members:
