---
title: redux, 그리고 flutter에서 redux 사용해보기
last_modified_at: 2020-05-19T10:04:16-05:00
category: 
    - Project
tags:
    - Project
    - flutter
    - Redux
---

이때까지 부끄럽게도 flutter에서 state에 관한 것은 stateless widget과 stateful widget, 그리고 stateful widget 내에서 setState 함수를 호출하면 그때마다 stateful widget 내에 오버라이드된 build 함수가 다시 실행되면서 위젯을 다시 그려준다는 것밖에 몰랐다. 

그러나 state 관리 기법이 엄연히 존재한다. setState로 위젯을 다시 그리는 것은 local state를 관리하는 것이다. 그 위젯 내에서 어떤 데이터의 변경이 있을 때 그 위젯의 state에만 다시 반영해주는 것이다. 하지만 앱에는 global한 state도 분명히 존재할 것이며 이것이 동시에 여러 위젯에 영향을 줄 수도 있다. 우리 So.SSul 앱 같은 경우에는 딱히 global state가 없는 것 같지만 다른 위젯에서 변경된 값이 이 위젯의 state를 변경하는 부분은 있다. 관련된 데이터를 위젯의 매개변수로 넘겨주는 방식으로 했는데 이게 아주 초보적인 방식이었다.

State management에는 여러가지 기법이 있다. Google에서 추천하는 방식은 Provider와 BLoC이다. Provider는 쓰기 간단하지만 큰 프로젝트에는 어울리지 않는다고 한다. 큰 프로젝트에는 BLoC을 쓰는 것을 추천한다고 하던데 솔직히 나에겐 어려웠다. Redux도 마찬가지로 어려웠는데 State를 하나의 Source에서 관리한다는 점이 조금 더 나에게 직관적으로 다가왔다.

Redux 관련 글을 3일을 하루 종일 읽어봤는데 읽어보기만 해서 그런지 개념은 알겠는데 이걸 어떻게 구현할 때 적용해야 할지 아직도 감이 안 잡힌다. 그래서 인터넷에 많이 돌아다니는 간단한 예제를 통해서 개념을 좀 다져보려고 한다.

## Redux

Redux를 간단히 표현한 도표는 다음과 같다.

![Redux](/assets/images/redux.png)

- **View**는 프로그램에서 렌더링된 부분을 말한다. 간단히 말해 유저가 볼 수 있고 상호작용할 수 있는 UI를 말한다. Flutter로 치면 Widget들이라고 할 수 있겠다.
- **Action**은 **View**에서 어떤 상호작용이 일어났을 때 변화된 state를 가지고 있는 intent이다. Android에서의 intent를 생각해보면 일종의 데이터 전송장치라고 보면 될 것 같다.
- **Reducer**는 현재의 state와 특정한 Action을 파라미터로 받아 새로운 state를 반환하는 함수이다. Reducer는 순수한 함수라고 하는데 이것을 쉽게 말하면 같은 파라미터를 받았을 때 언제나 같은 결과물을 반환하는 함수이다. 예를 들어 외부 API와 통신하여 결과물을 받아오는 함수는 Reducer가 될 수 없다.
- **Store**는 이러한 state가 저장되어 있는 곳이다. **Reducer**에 의해 생성된 state가 이곳에 저장된다.

그러면 state가 뭔가? 굳이 번역하면 상태인데 정말 애매한 단어이다. state는 앱에서 변화하는 모든 데이터를 말한다. 그것은 테마색이 될 수도 있고 리스트의 아이템이 될 수도 있고 로그인 정보가 될 수도 있고 현재 pushed된 route일 수도 있다.

그렇다면 flutter에서는 redux를 어떻게 구현해야 할까? 간단한 쇼핑 리스트 앱을 만들어봄으로써 redux을 조금이나마 이해하려고 노력해보았다.

먼저 Android Studio에서 flutter project를 하나 만들고 lib 폴더에 다음과 같은 코드를 가지고 있는 model.dart를 만들었다.

```dart
class CartItem{
    String name;
    bool checked;

    CartItem({this.name, this.checked});
}
```

모든 장바구니에 들어있는 아이템은 이 클래스의 형식을 따르게 될 것이다. 아직은 redux에 관한 코드는 아니다.

먼저 **action**들에 대해서 정의해보겠다. lib 폴더 아래에 actions.dart 파일을 만들고 다음과 같은 코드를 작성하였다.

```dart
class AddItemAction {
  final CartItem item;

  AddItemAction(this.item);
}

class ToggleItemStateAction {
  final CartItem item;

  ToggleItemStateAction(this.item);
}
```

여기 보면 각 **Action**은 변화되어야 할 **state** 혹은 데이터, 즉 CartItem을 담고 있다. 그리고 class로 선언되어 **action** 하나가 별도의 타입으로 인식되도록 구현하고 있다.

그런 다음 **Reducer**들을 구현해야 한다. lib 폴더 아래에 reducers.dart 파일을 만들고 다음과 같은 코드를 작성하였다.

```dart
List<CartItem> appReducers(List<CartItem> items, dynamic action) {
  if (action is AddItemAction) {
    return addItem(items, action);
  } else if (action is ToggleItemStateAction) {
    return toggleItemState(items, action);
  } 
  return items;
}

List<CartItem> addItem(List<CartItem> items, AddItemAction action) {
  return List.from(items)..add(action.item);
}

List<CartItem> toggleItemState(List<CartItem> items, ToggleItemStateAction action) {
  return items.map((item) => item.name == action.item.name ?
    action.item : item).toList();
}
```

위에서 말했다시피 reducer는 현재 state와 action을 받아서 새로운 state를 반환하는 함수이다. appReducers는 각 reducer를 통합하여 관리하는 함수이다. 같은 state에 대해 action이 다르게 들어오면 각기 다른 함수를 실행한다.

addItem 함수를 먼저 보면 이 앱의 state, 즉 CartItem 타입의 리스트와 AddItemAciton타입의 action을 받는다. 그리고 기존 List에 추가하는 것이 아닌 새로운 List를 만들고 거기에다가 Action에 들어 있는 변화된 state(추가될 데이터)를 더해서 반환한다. 이것은 state는 불변이기 때문에 언제나 state의 변화는 새로운 state를 만들어서 반환하기 때문이다.

두 번째 toggleItemState를 보면 기존의 state에서 각 원소에 대해 item.name이 action에 들어 있는 item.name과 같으면 action.item으로 대체하고 그게 아니면 그대로 둔 다음 그걸 반영한 새로운 리스트를 반환한다.

사실 이 부분이 좀 헷갈렸는데 이때까지 action이라는 단어의 의미 때문에 action이 함수 같다는 생각을 자꾸 했다. 하지만 이 예제에서 분명히 알게 되었다 action은 어떤 action의 타입을 선언함과 동시에 데이터를 담는 상자(payload) 역할을 한다.

이제 action과 reducer가 구현되었으니 store와 view를 만들어야 한다.

main.dart에서 다음과 같이 작성해준다.

```dart
void main() {
  final store = Store<List<CartItem>>(
      appReducers,
      initialState: List());

  runApp(MyApp(store));
}
```

가장 먼저 호출되는 콜백함수인 main에서 앱을 구동하기 전에 store를 정의해준다. store는 generic 타입으로 List<CartItem>을 받고 있다. 이것은 이 앱의 state가 오로지 List<CartItem>밖에 없기 때문이다. 앱이 복잡해지면 AppState 클래스를 따로 만들어 여러가지 복잡한 state를 동시에 관리하게 된다. 그리고 초기 state를 설정해주어야 하는데 초기에는 빈 리스트를 띄운다.

이제 store를 만들었으니 이 store에 담긴 state를 반영하여 화면에 보여줄 view를 구현해야 한다.

MyApp 클래스를 다음과 같이 작성한다.

```dart
class MyApp extends StatelessWidget {
  Store<List<CartItem>> store;
  MyApp(Store<List<CartItem>> store);


  @override
  Widget build(BuildContext context) {
    return StoreProvider<List<CartItem>>(store: store,
      child: MyHomePage(),
    );
  }
}
```

final 변수 store를 선언하고 생성자를 만든다. 그리고 MyHomePage를 바로 리턴하는 대신 StoreProvider라는 위젯을 사용해 store를 하위 위젯에서 쓸 수 있도록 해준다. 이제 실제로 MyHomePage 클래스를 수정하여 여기서 일어나는 상호작용이 action을 reducer로 보내오 reducer가 새로운 state를 반환하여 다시 store로 오고 그 store에 저장된 새로운 state를 위젯에서 반영하도록 해야 한다. MyHomePage 위젯을 다음과 같이 작성하였다.

```dart
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StoreConnector<List<CartItem>, List<CartItem>> (
      converter: (store) => store.state,
      builder: (context, list) {
        return ListView.builder(itemBuilder: (context, position) => ShoppingListItem(list[position]));
      },
    );
  }
}
```

드디어 여기서 중요한 개념이 나왔다. 정말 헷갈렸던 부분인데 한 번 찬찬히 살펴보자. StoreConnector는 [flutter_redux 공식 문서 설명](https://pub.dev/packages/flutter_redux)에 따르면 가장 가까이에 있는 StoreProvider 부모에게서 Store를 얻어와 주어진 convert 함수에 따라 Store를 ViewModel로 전환한다(convert). 그리고 그렇게 전환된 ViewModel을 builder의 두 번째 매개변수로 넘겨준다.

여기서 ViewModel을 쉽게 말하자면(내가 정확히 이해했는지는 잘 모르겠다.) 위젯에서 쓰일 state 정보이다. 우리 앱에서야 state가 List 그 자체이기 때문에 저렇게 했지만 만약 위젯에서 쓸 state가 여러가지라면 따로 여러 정보를 담을 수 있는 클래스를 만들어 반환하도록 해야 한다. 이것은 비단 state 그 자체 뿐만이 아니라 action을 reducer로 전달하는 콜백이 될 수도 있다. 이럴 경우 store.dispatch(myAction)처럼 쓰면 된다. 어쨌든 여기서는 단순히 현재 store에서 state를 그대로 뿌려주면 되기 때문에 그런 작업은 하지 않았다.

쇼핑 아이템을 추가하는 대화창도 정의한다. 먼저 AlertDialog 위젯을 다음과 같이 구현한다.

```dart
class AddItemDialogWidget extends StatefulWidget {
  final OnAddCallback callback;

  AddItemDialogWidget(this.callback);

  @override
  State<StatefulWidget> createState() => new AddItemDialogWidgetState(callback);
}

class AddItemDialogWidgetState extends State<AddItemDialogWidget> {
  String itemName;

  final OnAddCallback callback;

  AddItemDialogWidgetState(this.callback);

  @override
  Widget build(BuildContext context) {
    return new AlertDialog(
      contentPadding: const EdgeInsets.all(16.0),
      content: new Row(
        children: <Widget>[
          new Expanded(
            child: new TextField(
              autofocus: true,
              decoration: new InputDecoration(
                  labelText: 'Item name', hintText: 'eg. Red Apples'),
              onChanged: _handleTextChanged,
            ),
          )
        ],
      ),
      actions: <Widget>[
        new FlatButton(
            child: const Text('CANCEL'),
            onPressed: () {
              Navigator.pop(context);
            }),
        new FlatButton(
            child: const Text('ADD'),
            onPressed: () {
              Navigator.pop(context);
              callback(itemName);
            })
      ],
    );
  }

  _handleTextChanged(String newItemName) {
    setState(() {
      itemName = newItemName;
    });
  }
}
```

ADD버튼의 onPressed 함수를 보자. 대화창을 pop 하면서 매개변수로 받은 callback 함수를 실행하게 되어있다. 여기서 callback 함수는 또 itemName을 매개변수로 받는다. itemName은 TextField에 입력된 텍스트이다. 그리고 이 위젯을 그려주는 위젯을 다음과 같이 구현한다.

```dart
class AddItemDialog extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new StoreConnector<List<CartItem>, OnAddCallback>(
        converter: (store) {
      return (itemName) => store
          .dispatch(AddItemAction(CartItem(name: itemName, checked: false)));
    }, builder: (context, callback) {
      return new AddItemDialogWidget(callback);
    });
  }
}
```

여기서는 위와 다르게 StoreConnector의 제네릭 타입의 두 번째 매개변수가 Function이다. 즉 이 StoreConnector에서 converter는 store를 매개변수로 받아 위의 AddItemDialogWidget의 매개변수로 넘겨줄 콜백함수를 반환한다. 그래서 ADD버튼을 누를 때마다 '(itemName) => store.dispatch(AddItemAction(CartItem(name: itemName, checked: false)));'가 실행될 것이다. store.dispatch(action) 하게 되면 해당하는 action을 reducer에 넘겨주고 reducer는 action을 보고 적절한 처리를 한 후 새로운 state를 store에 넘겨주고 store를 지켜보고 있던 위젯들은 바뀐 state에 따라 다시 UI를 그린다.

또 쇼핑 리스트를 체크하는 위젯도 만들어야 한다. 코드는 다음과 같다.

```dart
class ShoppingListItem extends StatelessWidget {
  final CartItem item;

  ShoppingListItem(this.item);

  @override
  Widget build(BuildContext context) {
    return new StoreConnector<List<CartItem>, OnStateChanged>(
        converter: (store) {
      return (item) => store.dispatch(ToggleItemStateAction(item));
    }, builder: (context, callback) {
      return new ListTile(
        title: new Text(item.name),
        leading: new Checkbox(
            value: item.checked,
            onChanged: (bool newValue) {
              callback(CartItem(item.name, newValue));
            }),
      );
    });
  }
}
```

여기서도 보면 StoreConnector는 state를 받아서 콜백 함수로 전환한다. 그리고 이 콜백함수는 builder가 빌드하는 위젯의 onChanged에 들어가 CheckBox의 상태가 변화할 때마다(체크되거나 풀릴 때마다) store.dispatch(ToggleItemStateAction(item)); 이 호출될 것이다. 다시 말해 store에서 ToggleItemStateAction을 reducer로 넘겨주어 새로운 state가 store에 반환되도록 할 것이다.

Redux 관련 글을 3일을 하루종일 읽고서야 겨우 이 정도를 이해하게 되었다. 그래도 나름 복잡한 디자인 패턴을 이해하고 나니 좀 더 코드를 보는 눈이 확장된 느낌이다. 이제는 꼭 Redux가 아니라 다른 모델-뷰 패턴을 봐도 얼추 이해가 된다. flutter에는 아직도 내가 완전히 이해하지 못하는 개념들이 있다. 앞으로 차근차근 배워나가면서 미래가 밝은 프레임워크 flutter를 더 확실히 익혀야겠다.

이 글은 이 [포스트](https://hackernoon.com/flutter-redux-how-to-make-shopping-list-app-1cd315e79b65)를 참고하여 작성되었으며 원본 코드는 [여기](https://github.com/pszklarska/flutter_shopping_cart/tree/4756839d5749dfa36073e830b208bb45cb5f8874)에 있다.
