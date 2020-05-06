---
title: "[So.SSul] 위젯이 빌드되기 전 async 함수를 실행시켜 build를 넘어가기"
last_modified_at: 2020-05-07T19:57:25-05:00
category: 
    - Project
tags:
    - Project
    - flutter
---

오늘 개발하면서 막혔던 부분은 **Stateful Widget**에서 콜백함수 **initState**가 호출될 때 로컬에서 데이터를 불러와서 데이터에 대해 판단한 다음 이 위젯을 빌드할 필요 없다는 판단이 내려지면(그냥 다음 화면으로 바로 넘어가도 되겠다는 판단이 내려지면) **initState** 자체에서 **Navigator**를 이용하여 바로 다음 위젯으로 넘어가게 하려고 하면서 생긴 문제다.

[이 글](https://tksuns12.github.io/project/second-post/)에서의 문제와 다른 점은 이 글에서는 그래도 **build** 함수가 실행된 후에 **Navigator**가 실행되었다는 점이다.

어쨌든 새로 발생한 이 문제를 해결하기 위해서 두 가지 방법을 혼용하였다.

1. **initState** 내에서 실행될 비동기 함수를 미리 async modifier를 병기하여 구현한다. 이 함수 내에서 **SharedPreference**를 사용하여 로컬 데이터를 불러 오면 된다. 그리고 **Navigator**도 이 함수 안에서 실행하면 된다.

2. 로컬에서 불러온 데이터를 저장할 변수를 클래스의(이 경우에는 Stateful Widget) 멤버 변수로 선언하지 않고 함수의 지역 변수로 선언하였다.
3. 그리고 마지막으로 **initState** 함수의 body에 위에서 구현한 함수를 호출하면 끝이다.

우리 앱은 Native Splash 화면이 있고 그와 똑같이 생긴 가짜 Splash가 있다. 이 가짜 Splash 화면의 initState에서 로그인 정보와 앱 첫 실행 여부를 불러와 튜토리얼 화면을 보여주든지 바로 로그인 하여 메인화면으로 넘어가든지 하는 분기를 구현한다.