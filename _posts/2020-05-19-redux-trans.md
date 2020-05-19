---
title: "Redux란 무엇인가?"
last_modified_at: 2020-05-19T19:15:00-05:00
category:
    - Project

tags:
    - Redux
    - flutter
---

# flutter와 Redux

이 글은 [이 문서](https://gitlab.com/brianegan/flutter_architecture_samples/-/blob/master/example/redux/README.md)를 번역한 것입니다. 제가 이때까지 본 문서 중에 flutter에서 redux를 사용하는 방법에 대해서 가장 간명하게 서술해놓은 문서라 한국어로 번역을 해봅니다. 다만 널리 쓰이는 개발 관련 용어는 영어로 그대로 쓰고 설명을 첨부했습니다.

## 핵심 개념

- App State는 ```Store``` 속의 위젯 계층에서 가장 위에 위치하고 있는 immutable(변경할 수 없는)한 객체이다.
- ```Store```는 ```StoreProvider```라 불리는 ```InheritedWidget```을 통해 모든 후손 위젯에게 전달 된다.
- ```State``` 객체는 immutable하다. 그러므로 ```State```를 업데이트하기 위해서는 ```Action```을 발송해야(dispatch) 한다.
- ```Reducer```는 ```Action```을 취한다. ```Reducer```는 이전의 ```State```와 발송된 ```Action```에 기반하여 새로운 ```State```를 빌드하고 반환한다.
- Reducer는 순수한 함수(Pure Functions)이다.
- App State가 업데이트 되면, ```StoreConnector```를 통해서 ```Store```에 연결되어 있던, 모든 위젯들은 다시 빌드된다.
- ```StoreConnector```를 사용하는 위젯들을 ```Container``` 위젯이라고 부른다. 이들은 오직 이전의 App State를 뷰모델로 전환하는 역할만 맡는다.
- 데이터를 화면에 표시하는 위젯들을 ```presentation``` 위젯이라고 부른다. ```Text``` 위젯이나 ```FloatingActionButton``` 같은 위젯을 생각해보면 된다.
- ```State```에서 데이터를 읽어오려면 ```selector``` 함수를 이용하라. 이것은 마치 *State 데이터베이스*에 질의를 하는 것과 같이 작동한다.
- DB나 웹에서 데이터를 불러오는 것 같은 동작을 다루려면 ```Middleware```를 사용한다.

## App State 싱글톤

Redux에서의 아이디어는 앱의 상태를 루트 수준의 싱글톤에 보관하는 것입니다.

만약 당신이 간단한 예제를 보았다면 이것이 *Lifting State up*이라는 핵심 법칙을 활용한 것이란 것을 알 수 있습니다. 이 경우 우리는 앱의 상태를 앱의 최상위까지 끌어올려 모든 후손들이 앱의 상태에 접근할 수 있게 합니다.

이를 위해서는 Redux ```Store```를 만들고 ```StoreProvider```에게 넘겨주어야 합니다. 이렇게 하면 ```StoreProvider```의 모든 후손들은 ```StoreProvider.of(context).store``` 코드를 쓰거나 ```StoreConnector```를 통해 store에 접근할 수 있을 것입니다.

### App State 업데이트 하기

App State를 업데이트 하기 위해서는 ```Action```을 발송해야 합니다.

당신의 ```Reducer``` 함수가 ```Action```을 붙잡아 ```Action``` 안에 보관되어 있는 데이터를 사용하여 App State를 업데이트 할 것입니다.

```Reducer``` 함수는 순수한 함수입니다. ```Reducer```는 오직 이전의 state와 발송된 action을 가지고 새로운 App State를 반환하는 역할만 맡습니다.

다시 말해 ```Reducer```는 API 호출이나 로그 기록과 같은 부가 효과를 만들어서는 안 됩니다. 이런 일을 하기 위해서는 ```Middleware```를 사용하십시오.

처음 Redux를 사용할 때 이것이 "상용구 코드(Boilerplate)" 같이 느껴질 수도 있지만 [이 문서](http://redux.js.org/)를 보면 Redux를 사용할 동기부여가 될 것입니다.

<pre>
만약 모델이 다른 모델을 업데이트할 수 있다면 뷰가 모델을 업데이트 할 수 있으며 이것은 또 다른 모델을 업데이트 할 수 있고 이것은 결국 또 다른 뷰의 업데이트를 초래할지도 모른다.

언젠가는 당신은 state가 바뀌어야 할 이유, 바뀌어야 할 때 그리고 바뀌는 방법을 통제할 수 없게 됨으로써 당신 앱에서 무슨 일이 일어나고 있는지에 대해 이해하지 못하게 될 것이다.

시스템이 불투명하고 결정론적이지 못하다면 버그를 재현하고 새로운 기능을 추가하기가 매우 힘들다.
</pre>

당신의 앱에서 위와 같은 고충이 있었다면 Redux나 다른 상태 관리 패턴에 대해 생각해볼 좋은 기회입니다.

이렇게 강력한 방식으로 ```Actions```를 발송하고 App State를 업데이트 하면 다음과 같은 사항을 명확하게 파악할 수 있습니다.

1. 어떤 Action이 State를 변경했는지
2. 어떤 Reducer가 그 변경 사항에 책임이 있는지
3. Action에 대한 반응으로 State가 왜 작동하지 않게 됐는지
4. State의 변경에 대하여 View가 언제 업데이트 될 필요가 있는지

## UI 업데이트 하기

```Action```에 대하여 App State가 바뀔 때마다 당신은 아마 UI를 어떠한 방식으로든 업데이트하고 싶을 겁니다.

그러기 위해서는 ```StoreConnector``` 위젯을 사용하여 ```StoreProvider```에 연결하세요. ```StoreConnector```가 하는 일은 단순합니다. store의 가장 최근 state를 받아 ```ViewModel```로 전환하죠. 그런 뒤 ViewModel을 이용하여 위젯 트리를 빌드합니다.

App State가 변경될 때마다 ```StoreConnector```는 ```ViewModel```과 ```Widget```트리를 다시 빌드합니다.

위젯을 쉽게 테스트하고 기능을 공유하기 위해서는 다음 두 가지의 위젯을 구분하여 만드는 것을 추천합니다.

- ```container``` 위젯 - 이 위젯은 당신의 ```presentation```위젯을 위해 ```StoreConnector``` 위젯을 사용하여 ViewModel을 구축합니다.
- ```presentation```위젯은 UI를 구축하는 데 필요한 모든 데이터를 받는 ```StatelessWidget```입니다.

이것은 ```presentation``` 위젯을 테스트하기 쉽게 합니다. 왜냐하면 이 위젯이 요구하는 데이터를 그저 렌더링 시점에서 던져주기만 하면 되기 때문입니다. 그리고 assertion들을 렌더링된 결과물에 작성해주세요. 이것은 마치 UI의 '순수한 함수'와 같습니다.

## Selector 함수

```Selector``` 함수는 App State에 어떤 한 시점에서 접근할 수 있게 해주는 단순한 함수입니다. 좀 더 자세한 설명을 위해서는 [이 문서(영어임)](https://pub.dartlang.org/packages/reselect)을 참조해주세요.

## Middleware를 사용하여 데이터를 불러오고 저장하기

우리의 할 일들을 웹이나 DB에서 불러오기 위해서는 비동기 호출을 사용해야 합니다. ```Reducer``` 함수는 순수하기 때문에 대신에 우리는 ```Middleware```를 사용해야 합니다.

```Middleware```는 발송된 ```Action```에 대응하여 작동합니다. 그리고 ```Reducer``` 이전에 실행됩니다. 당신은 ```Action```을 중간에 가로채어 데이터를 불러올 수 있습니다.

이 앱에서 우리는 "Store Todos Middleware"를 가지고 있습니다. 이것은 ```LoadTodos```와 ```SaveTodos``` 타입의 action에 반응하여 웹이나 DB에서 데이터를 불러오고 저장합니다.

## 테스트하기

전반적으로 이 앱은 "테스팅 피라미드"를 형성하고 있습니다. 수많은 유닛 테스트, 좀 더 적은 위젯 테스트, 좀 더 적은 통합 테스트로 이루어져 있죠.

- 유닛 테스트
  - ```Reducer``` 함수는 순수한 함수이기 때문에 아주 쉬운 유닛 테스트입니다.
  - API를 호출하는 ```Middleware``` 함수는 Mock 구현을 사용해 테스트 될 수 있습니다. 이것은 Mockito 라이브러리를 사용하여 이루어집니다.
  - ```selector``` 함수는 또한 순수한 함수이기 때문에 쉽게 테스트할 수 있습니다.
- 위젯 테스트
  - ```container``` 위젯은 정확히 ```ViewModel```을 생성해냈는지 확인하기 위해 테스트 될 수 있습니다.
  - ```presentation``` 위젯은 가짜 데이터를 넘겨주고 aseertion을 만듦으로써 테스트할 수 있습니다.
- 통합 테스트
  - 앱을 실행하고 ```flutter_driver```를 통해 drive 하십시오.
  - '페이지 객체 모형' 패턴을 사용하여 테스트를 좀 더 읽고 유지하기 쉽게 만드십시오.