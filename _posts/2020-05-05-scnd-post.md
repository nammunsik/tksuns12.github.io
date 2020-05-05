---
title: "[So.SSul] 자식 위젯에서 부모 위젯 setState 하기"
last_modified_at: 2020-05-05T19:57:25-05:00
category: 
    - Project
tags:
    - Project
    - flutter
---

오늘의 문제점은 바로 이것이었다. 우리 앱의 루트 레이아웃은 **MaterialApp**이다. 즉 **appBar**, **body**, **bottomNavigationBar** 3단 구성이다. 그리고 BottomNavigationBar의 버튼을 누르면 **body** 부분만 교환된다. 

그런데 두 번째 네비게이션 버튼을 누르면 모든 글 목록이 보이게 되는데 홈 화면의 장르 선택 버튼을 눌러도 사실 상 장르로 필터링을 해서 보여주는 거긴 하지만 어쨌든 글 목록을 보여주게 되는 것이다. UI의 일관성을 유지하기 위해서 어디서 어떻게 누르든 **body**를 교체해서 글 목록을 보여줘야 하는 것이다. 이렇게 하려면 **HomeBody**에서 부모 위젯인 **MaterialApp**의 **setState**를 호출하여 **MaterialApp**의 **build** 함수가 다시 호출되도록 해야 한다. 어떻게 할까 고민하며 구글링 해본 결과 Callback을 이용하면 된다. 

예제는 다음과 같다. **Parent** 위젯의 **body**에 대입되어 있는 **Child** 위젯에서 **Parent** 위젯의 **isChanged** 값을 **true**로 바꾸면서 state를 갱신하는 코드이다.

parent.dart

```dart
class Parent extends StatefulWidget {

    bool isChanged = false;

    @override
    _ParentState createState() => _ParentState();
}

class _ParentState extends State<Parent>{
    Child child;

    @override
    void initState() {
        super.initState();
        child = Child(changeParent:change));
    }

    @override
    Widget build(BuildContext context) {
        return MaterialApp(
            appBar: Appbar(),
            body: Child(),
        )
    }

    void change() {
        setState(() {
            isChanged = true;
        });
    }
}
```

child.dart

```dart
class Child extends StatefulWidget{
    final Function changeParent;

    Child({this.changeParent});

    @override
    _ChildState createState() => _ChildState();
}

class _ChildState extends State<Child> {
    @override
    Widget build(BuildContext context) {
        return Container(
            child: Flatbutton(
                child: Text('부모의 상태를 변경'),
                onPressed: () {widget.changeParent();}
            )
        )
    }
}
```
이렇게 부모 위젯에서 자식 위젯을 미리 생성하면서 함수를 인자로 전달해주면 자식 위젯에서 부모 위젯에 정의되어 있는 함수를 실행할 수 있고 이렇게 하면 자식 위젯에서 부모 위젯의 상태를 바꿀 수 있다. 부모 위젯에서 자식 위젯의 상태를 바꾸는 것도 추후에 알아보아야겠다.