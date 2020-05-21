---
title: [So.SSul] flutter에서 Redux를 사용하여 비동기 처리하기
last_modified_at: 2020-05-21T19:21:37-05:00
categories:
    - Project
tags:
    - Redux
    - flutter
---

Redux에서 실제로 ```state```를 직접적으로 변경하는 부분은 ```reducer```이다. *'변경'이라는 말이 오해를 불러일으킬 수 있으니 첨언하자면 reducer는 state를 변경하는 게 아니라 기존의 state를 가져와서 받은 action에 따라 부분 혹은 전체를 바꾼 뒤 새로운 state를 만들어서 반환하는 것이다. 이것을 앞으로 그냥 편하게 state를 바꾸는 것이라고 하겠다.* 어쨌든 다시 ```reducer``` 이야기로 돌아오자면 ```reducer```는 순수한 함수이다. 같은 패러미터를 받으면 반드시 같은 값을 반환해야 한다. 그래서 로컬이든 서버든 다른 api를 사용하든 외부에서 데이터를 받아오는 작업은 ```reducer```가 처리할 수 없는 작업이다. 가령 예를 들어 ```reducer```가 로컬에서 데이터를 받아 온다면 패러미터로 데이터의 위치를 받을 텐데 로컬에 같은 위치에 데이터가 변경되어 다른 결과를 받아올 수도 있으므로 순수한 함수라는 정의에 어긋나게 된다.

보통 이런 작업들은 시간이 걸리기 때문에 비동기 작업인데 이러한 ```async``` 작업을 Redux에서 가능케 하는 것이 바로 Middleware이다. Middleware는 아래의 그림과 같이 ```action```이 ```reducer```로 넘어가는 중간에 ```action```을 가로채어 자신이 처리할 ```action```이 있는지 먼저 살펴본 뒤 비동기 작업을 수행하여 그에 따른 결과물을 다시 ```reducer```로 넘겨준다.

![redux_image](https://github.com/tksuns12/tksuns12.github.io/blob/master/assets/images/redux_with_middleware.png)

Middleware를 구현하는 법은 간단하다. 다음과 같이 dart 파일을 하나 만들어 다음 코드를 작성한다.

```dart
void yourCustomMiddleware(Store<YourAppState> store, action, NextDispatcher next) {
    if (action is YourAsyncAction) {
        // 여기서 async로 불러올 데이터를 불러 오십시오. 간단한 코드를 저는 작성해보겠습니다.
        api.loadData(apiCode).then(
            (data) {
                store.dispatch(YourDataAction(data));
                }
            ).catchError((error) {
                store.dispatch(YourErrorCatchAction(error));
            });
    }

    next(action);

}
```

코드를 찬찬히 뜯어보자. 이 함수는 ```store```와 ```action```, 그리고 ```NextDispatcher``` 타입의 ```next```라는 매개변수를 받는다. 말 그대로 ```store```를 받아 ```action```을 ```reducer```로 넘어가기 전에 가로챈 뒤에 이 함수 내에서 모든 작업을 거친 후 ```action```을 ```reducer```로 넘겨준다. 본 코드에서는 ```middleware```를 호출하는 ```action```과 실제 데이터를 store에 업데이트 하는 action을 분리하여 구현했는데 공식문서에서 권장하는 방식이다.

이런 다음 main.dart의 ```Store```의 ```middleware```인자로 ```[yourCustomMiddleware]``` 이렇게 리스트 형식으로 넘겨주면 된다. ```Middleware```가 여러 개라면 리스트 안에 여러 개를 넣어주면 된다.

```dart
void main() {
    final store = Store<YourAppState>(
        yourAppReducer,
        initialState: AppState.initial(),
        middleware: [yourCustomMiddleware]
        );

        runApp(StoreProvider(store: store, child MyApp()));
}
```

```Middleware```를 쓰기 좋게 만들어 놓은 패키지들도 있다. 기본적인 ```async``` 작업에는 [redux_thunk](https://pub.dartlang.org/packages/redux_thunk)를 쓸 수 있고 ```Stream```을 처리해야 할 때에는 [redux_epics](https://pub.dartlang.org/packages/redux_epics)를 쓰면 된다. 그런데 ```async``` 작업은 이미 기본적인 ```Middleware```로 구현이 쉽게 되기 때문에 redux_thunk는 당분간 쓸 일이 없을 것 같다. 