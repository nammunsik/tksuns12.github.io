---
title: "[So.SSul] Stream과 Firestore"
last_modified_at: 2020-05-08T21:39:25-05:00
category: 
    - Project
tags:
    - Project
    - flutter
---

오늘은 사실 코딩을 하지 않았다. 이제 로그인 기능도 구현했겠다 본격적으로 앱의 메인 기능인 소설 작성 기능을 구현하려고 하니 데이터베이스를 사용해야 했다. 사실 대부분의 페이지는 정적으로 구성되는 게 자연스럽겠지만 본격적으로 소설 이어쓰기가 진행되는 소설 방 내에서는 실시간으로 내 앞 순서인 사람이 작성 중인지 여부나 실시간 추천(혹은 좋아요) 수가 올라간다면 좀 더 역동적인 사용자 경험을 제공할 수 있을 것 같아서 일단 알아나 보았다.

## flutter에서 StreamBuilder를 사용하여 위젯 그리기

기본적인 코드 구조는 다음과 같다.

```dart
StreamBuilder<T>(
    stream: _stream, // 여기서 스트림을 설정
    builder: (context, snapshot) { //여기서 snapshot을 받아와서 처리하게 됨
        List<DocumentSnapshot> documents = snapshot.data.document;
        return ListView(
            children: documents.map((eachDocument) => DocumentView(eachDocument)).toList(),
        );
    }
);
```

주로 저렇게 스냅샷을 받아와서 각 원소를 리스트화 시켜 ListView의 자식 위젯으로 그린다. 그리고 서버 데이터에 변화가 있을 때마다 스트림빌더가 실행되어 새로운 데이터를 새로 그려준다. 그래서 굳이 setState() 함수를 호출할 필요가 없다.

## FireStore

FireStore 자체에 대해서는 아직 많이 모르지만 오늘은 가장 중요한 데이터베이스 규칙에 대해서 알아보았다. 내가 방장이 아닌데 어떤 방을 지울 수 있고 비공개로 설정한 방이 아무나 들어올 수 있게 되고 내 개인정보에 다른 유저가 접근할 수 있게 된다면 서비스가 엉망이 될 것이다. 이때 필요한 것이 데이터베이스 규칙이다.

모든 Firestore 규칙은 이런 구문으로 시작한다.

```cel
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

첫 번째 줄은 이 규칙 언어의 버전을 뜻한다. 언제나 규칙은 
match <문서 경로> {
    allow <동작(create, read, ...)>: if <조건>
}
이런 식으로 동작한다. 조건이 맞으면 문서 경로와 match 되는 문서에 한하여 동작을 허용하겠다는 것이다.

문서 경로를 작성하는 방식은 /로 구분되는 경로인데 몇 가지 문법이 존재한다. 만약 경로를 {} 중괄호로 감싸면 이 경로에 해당하는 모든 문서를 변수처럼 match 블럭 안에서 참조할 수 있다. 예를 들면 이렇게

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userID} {
      allow read, write: if userID == request.auth.uid;
    }
  }
}
}
```

이 규칙의 뜻은 요청의 uid가 users 아래의 문서 제목과 같다면 쓰기와 읽기를 허용한다는 뜻이다. 만약 어떤 컬렉션 아래의 모든 문서에 대해서 규칙을 적용하고 싶다면 {document=**} 이런 식으로 하면 재귀 와일드카드가 된다.

또, match 블럭 내에서 문서의 field를 참조하고 싶다면 resource 키워드를 쓰면 된다. 이 resource 키워드는 match 뒤에 나오는 문서 경로의 그 문서를 뜻하고 resource.data.[fieldname] 이런 식으로 문서 내에 필드를 참조할 수 있다.

내일은 데이터베이스 설계를 하고 본격적으로 코딩을 시작해봐야겠다.