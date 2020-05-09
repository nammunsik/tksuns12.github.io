---
title: "[So.SSul] flutter에서 정규식을 사용해 TextField의 텍스트 검증하기"
last_modified_at: 2020-05-09T19:46:34-05:00
category: 
    - Project
tags:
    - Project
    - flutter
---

데이터베이스 설계는 했지만 나는 작업을 할 때 무조건 UX의 흐름대로 따라가는 편이라 로그인 할 때 닉네임을 설정하지 않고 글쓰기나 데이터 불러오기 등을 구현하기는 싫었다. 그래서 닉네임을 설정하는 로직을 구현하기로 했다.

여기서 맞닥뜨린 문제 두 가지는 첫 번째, 어떻게 TextField 내에서 입력된 텍스트를 검증할 것인가? 그리고 검증됐을 때가 되지 않았을 때 일어나는 Event를 어떻게 처리할 것인가? 처음에는 TextField와 if문 등으로 직접 구현하려고 했으나 코드가 너무 복잡해지는 문제가 있었다. 그래서 구글링을 해보니 Form이라는 것과 TextFormField라는 것이 있었는데 이 두 가지를 가지고 TextFormField 내에 입력된 문자열을 검증할 수 있었다. 예제 코드는 다음과 같다.

```dart
Form(
    final _formKey = GlobalKey<FormState>();
    key: _formKey,
        child: TextFormField(
                onChanged: (text) {
                nickName = text;
            },
        validator: (value) {
            Pattern nickNamePattern = r'^[ㄱ-ㅎ|ㅏ-ㅣ|가-힣0-9]{2,6}$';
            RegExp nickNameRegex = RegExp(nickNamePattern);
            if (dbManager
                .loadUserInfo(
                currentUser: _currentUser)
                .then((value) =>
                value.data["nickname"]) ==
                null) {
                    return "이미 있는 별명입니다.";
                    } else if (!nickNameRegex.hasMatch(value)) {
                    return "별명은 숫자 포함 한글 2~6자 사이입니다.";
                    } else {
                    return null;
                    }
                },
            ),
        ),
```

이해를 위해 몇 가지를 생략한 코드이다. 먼저 기존의 TextField를 TextFormField로 대체하였다. 그리고 TextFormField를 Form이라는 위젯으로 감싸주었는데 이 Form 위젯이 안에 있는 텍스트들을 일괄적으로 검증해주게 된다. 그리고 GlobalKey를 생성해주었다.

TextFormField를 보면 TextField와 다르게 validator라는 인자가 있는데 이것은 Field 내에 입력된 텍스트를 인자로 받는 콜백함수를 인자로 받는다. 텍스트가 입력되면 검증을 시작하는데 if문과 return String 을 통해 구현하면 된다. 만약 null 값을 리턴하지 않는다면 반환된 텍스트를 빨간색 글씨로 TextField 바로 아래에 경고문처럼 띄워주게 된다. (사실 TextField만 썼다면 TextField 바로 밑에 또 Text를 하나 더 만들어서 조건을 체크해서 보였다가 안 보였다가 하게 만들 작정이었다 ㅜㅜ)

그리고 두 번째 조건문을 보면 정규식을 쓰고 있는 것을 볼 수 있는데 정규식을 쓰려면 먼저 Pattern 타입의 정규식을 작성하고 그것을 RegExp 클래스의 인자로 넘겨 객체를 생성해야 한다. 그리고 RegExp 객체의 hasMatch 함수의 인자로 텍스트를 넘기면 작성해둔 Pattern 타입의 정규식으로 텍스트를 검증하게 된다.