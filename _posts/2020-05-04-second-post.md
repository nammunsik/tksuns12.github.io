---
title: "[So.SSul]Flutter에서 await를 initState 함수 내에서 사용하기"
last_modified_at: 2020-05-04T17:33:14-04:00
categories: 
    - Project

tags:
    - flutter
    - Project
---

Flutter에서 코딩을 하다보면 위젯이 build 되기 전에 로컬데이터를 불러와야 하는 경우가 있다. 그냥 위젯을 하나 더 만들어서 데이터를 불러오는 용도로 사용하고 그에 맞춰 Route를 설정해주면 되긴 하나 위젯을 또 하나 더 만들게 되면 리소스를 많이 잡아먹을 것 같아 Stateful 위젯의 initState() 함수 내에서 로컬 데이터를 불러오고 싶었다.

initState() 함수는 build() 함수보다 먼저 호출되는데 문제는 initState() 함수 내에서 await를 쓸 수 없다는 것이다. 그런데 로컬 데이터는 반드시 비동기로 불러와야 하기 때문에 await를 쓰지 않으면 데이터를 불러오기도 전에 build() 함수가 호출되어 버린다.

이렇게 어떤 비동기 함수가 Future 타입을 반환할 수 없거나 다른 곳에서 await 키워드를 실행할 수 없을 때

```dart
asyncFunction().then(onComplete);
```

이런 형식으로 비동기 함수가 작업을 완료할 때까지 기다리게 할 수 있다. onComplete에는 콜백함수가 들어가는데 asyncFuction의 작업이 완료되었을 때 실행할 동작을 넣으면 된다. 다만 단지 await 대신에 비동기 함수를 작동시키기 위해서 쓰는 거라면 빈 익명 함수 (value){}를 넣으면 잘 작동 된다.