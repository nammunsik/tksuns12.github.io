---
title: "[So.SSul] 비동기 프로그래밍"
last_modified_at: 2020-05-07T19:33:25-05:00
category: 
    - Project
tags:
    - Project
    - flutter
---

플러터로 앱을 개발하다 보면 비동기 프로그래밍을 해야 할 때가 많다. Future와 Stream을 쓰는 경우는 아직 없지만 - 분명히 나중에 생길 거다.- 아직까지는 async 함수 내에서 await를 쓰는 정도밖에 안 된다.

사실 사용법이라는 것이 async modifier를 함수 끝에 붙여주고 함수 내에서 await가 필요한 작업에 대해서 await를 써주면 되기는 한데 이것이 은근히 묘하다.

오늘 구글 로그인 기능을 구현하기 위해서 만든 함수이다.

```dart
  Future signInGoogle({@required BuildContext context}) async {
    prefs = await SharedPreferences.getInstance();
    final GoogleSignInAccount googleUser = await _googleSignIn.signIn();
    final GoogleSignInAuthentication googleAuth = await googleUser.authentication;
    final AuthCredential credential = GoogleAuthProvider.getCredential(accessToken: googleAuth.accessToken, idToken: googleAuth.idToken);
    final FirebaseUser user = (await _auth.signInWithCredential(credential)).user;
    prefs.setString(kSignInType, 'Google');
  }
```

맨 마지막 줄에 보면 SharedPreference 패키지를 이용하여 로컬에 로그인 타입을 Google로 설정하는 코드가 보일 것이다. 맨 처음에 이렇게 작성한 이유는 뭐든지 위에서 일어난 작업이 다 완료돼야 로그인 타입을 저장하는 것이 의미가 있기 때문이다. 그런데 자꾸 저 코드가 동작을 하지 않았다. 결국 저 코드를 둘째 줄로 옮기고 나니 작동이 되긴 됐는데 그러면 의미가 없는 것이다. 로그인이 성공했든 말았든 무조건 로그인 타입을 저장해 다시 앱을 실행시킬 때 같은 방식으로 자동 로그인을 시도할 것이기 때문이다.

나중에 알고 봤더니 _auth.signInWithCredential(credential) 이 부분에서 에러가 났던 것이다. 왜냐하면 그 위에 credential을 할당할 때 getCredentiail 함수의 패러미터를 거꾸로 설정했기 때문이다. 이 부분을 고치고 나니 위의 코드도 잘 작동되었다.

여기서 내가 배운 점은

1. 예외 처리를 꼼꼼하게 하자.
2. await 해놓은 비동기 함수가 에러를 뿜으면 별도의 처리가 없으면 그 밑의 코드는 씹히는 것 같다.
3. 대신에 signInGoogle 함수 자체는 완결이 나는 것으로 처리가 되는 모양이다. 왜냐하면 signInGoogle 함수도 본 페이지에서는 await가 붙은 채로 실행되었는데 함수 내부에서 오류가 났음에도 불구하고 그 밑에 있던 Navigator 코드가 잘 작동되었기 때문이다.

이런 간단한 기능 하나를 구현하는 데 오타 하나 때문에 하루 종일 헤맸지만 결국 예외처리에 익숙지 못한 나의 서투름과 비동기 함수에 대해 잘 이해하지 못한 나의 잘못이다.

한 가지 더. 만약 어떤 함수가 반드시 특정 비동기 함수가 끝난 뒤에 실행되어야 한다면 then이나 whencomplete를 쓰면 된다. then은 비동기 함수가 에러 없이 완료되었을 때 비동기 함수의 반환값을 인자로 받아 콜백함수를 호출하고 만약에 에러가 뜨면 onError 콜백함수를 호출한다. whencomplete는 에러 유무와 상관 없이, 반환값을 받지 않고 콜백함수를 호출한다. finally와 비슷한 느낌이다.