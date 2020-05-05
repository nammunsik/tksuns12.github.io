---
title: "(TIL) C++"
date: 2020-05-04 12:54:24 -0400
categories: [DataStructure, TIL]
---

김포프님께서 말씀하셨다, 모름지기 프로그래머라면 Managed Language 하나와 Unmanaged Language 하나 정도는 다룰 줄 알아야 한다고. 무슨 말인지 알 것 같은 게 나는 머신러닝 공부 때문에 Python을 가장 먼저 배웠는데 사실 Python이 익숙해지고 나니 Java, Javascript, C#(이 놈은 특히 Java와 너무 비슷해서 Android로 수업 프로젝트를 하다가 배우기가 참 수월했다.), Dart 같은 언어들을 차례로 배우기 굉장히 쉬웠다. 그러나 Unmanaged Language라고는 기초 프로그래밍 수업 시간에 배운 C가 전부여서 사실 잘 다룬다고 말하기도 어렵다. (그래도 그 수업 내에서는 내가 가장 잘 다뤘다.)

아마 앞으로도 내가 더 많이 쓰게 될 언어는 Managed Language일 것 같다. 그렇기 때문에 자료구조와 알고리즘을 공부할 때만이라도 Unamanged Language로 개발 기초 체력을 쌓고자 하는 마음에 C++로 자료구조와 알고리즘에 대해 배울 수 있는 [Data Structures & Algorithms in C++](http://www.sso.sy/sites/default/files/data-structures-and-algorithms-in-c.pdf), 이 책을 통해 간단히 C++를 살펴보고 자료구조 공부를 해보려 한다. C++은 이제 너무 많이 발전해서 C와는 다른 언어가 되었다고 볼 수 있지만 그래도 C의 문법을 계승한 부분이 많아 C에서 배우지 않았던 것들을 위주로 먼저 정리해보고자 한다.

1. C++에는 가비지 컬렉터가 없으므로 new 연산자로 새로운 동적 객체를 생성했을 때에는 적당한 시기에 delete 연산자를 통해 힙 공간에서 삭제해주어야 한다. 그렇지 않으면 메모리 누수(Memory Leak)가 발생한다.

2. 객체와 다르게 new 연산자를 통해 생성된 배열은 delete가 아니라 delete [] 연산자를 통해 제거해야 한다.
3. References, 이제 다음 코드의 2행이 실행되면 penName이 author의 alias가 된다고 한다. 여전히 잘 이해가 되지는 않지만 유용한 기능일 것 같다.

```c++
string author = "Samuel Clemens";
string& penName = author;
penName = "Mark Twain";
cout << author;
```

4. typedef 이것은 C 수업 시간에 배운 것 같기는 하지만 잊어버린 것 같으니 정리해둬야겠다. 기존에 제공되는 타입에 새로운 이름을 붙여 쓸 수 있다.

```c++
typedef char* BufferPtr;

BufferPtr p;
```

5. 이름의 중복을 막기 위해 여러 가지 namespace를 만들 수 있으며 namespace에는 타입, 클래스, 함수, 객체의 정의들이 모두 들어갈 수 있다.

6. 비교 연산자는 STL strings끼리는 가능하나 C-style strings끼리는 안 된다. 그리고 포인터끼리도 비교될 수 있지만 어차피 얘들은 메모리 주소이기 때문에 같은지 다른지를 비교할 때는 의미없다.

7. 정적 캐스팅이라는 것을 처음 봤다. 보통 숫자를 정수에서 부동소수점으로 부동소수점에서 정수로 변환할 때 쓴다고 한다. 부동소수점에서 정수로 변환할 때에는 반올림하는 것이 아니라 그냥 소수점을 버린 값을 반환한다. 그런데 왜 숫자를 타입캐스팅할 때 정적캐스팅이 더 적절한 건지에 대해서는 논리적 이유가 안 적혀있다. 추후에 또 찾아봐야겠다.
8. In-line 함수는 매우 짧은 함수이다. 일반 함수도 짧을 수 있지만 일반 함수와 다른 점은 In-line 함수는 Body에 조건문과 반복문이 들어갈 수 없고 시스템의 호출-반환 메커니즘을 사용하지 않는다.
9. 클래스를 초기화할 때 대입 연산자를 지원하지 않는 타입의 멤버 변수를 초기화하려면 초기화 리스트를 사용해야 한다. 사용 방법은 다음과 같다.

```c++
ClassName::ClassName(member1, member2, member3)
: member1(initial_value1), memeber2(initial_value2), memeber3(initial_value3)
```

10. C++은 역시 Unamanaged Language라 그런지 생성자가 있으면 철거자?(Destructor)가 있다. Destructor는 생성자가 new 연산자로 객체가 생성될 때 실행되는 반면에 철거자는 delete 연산자로 객체가 제거될 때 실행된다.

11. 자기 자신을 할당하기 위해 new 연산자를 쓰는 모든 클래스는 다음 세가지를 정의해야 한다.

- 객체의 할당을 제거할 철거자
- 새로운 멤버변수를 할당하고 복사할 복사 생성자
- 옛 저장공간을 할당 해제하고 새로운 공간을 할당하며 모든 멤버 변수들을 카피할 대입 연산자

오늘은 여기까지.... 67페이지까지 보았음. 앞으로 매일 오전에는 C++ 공부와 자료구조 공부를 병행할 예정..