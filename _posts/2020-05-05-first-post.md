---
title: "(TIL) C++ 자료구조 알고리즘 2일차"
last_modified_at: 2020-05-05T10:32:44-05:00
category: 
    - TIL
tags:
    - TIL
    - C++
---

## friend 키워드

클래스 내에서 함수를 private하게 만들어주는 키워드이다. 문법상 멤버함수를 정의할 수 없을 때 쓸 수 있다. 또 한 가지 쓸 수 있는 경우는 두 클래스가 밀접하게 관련 있을 때 한 클래스 내에 다른 클래스를 friend로 선언해두면 friend 클래스에서 해당 클래스의 private 멤버들에 접근할 수 있다. 하지만 한쪽에서 다른 쪽에 friend 선언이 되었다고 해서 그 반대도 저절로 되는 것은 아니다.

유용한 기능이지만 남발하면 클래스 구조가 조잡해지게 된다.

## STL(Standard Template Library)

기본적인 자료구조들을 구현해놓은 클래스의 집합이다. 구현되어있는 자료구조들의 목록으로는 스택, 큐, Double-ended 큐, 벡터, 이중 연결 리스트, 우선순위큐, 세트, 맵(딕셔너리)가 있다.

## 몇 가지 전처리문들

이것은 C++이 아니라 C문법이긴 하지만 수업시간에 이 부분은 열심히 듣지 않았던 관계로 따로 정리를 해두어야 할 것 같다. 전처리 키워드를 잘 모르니까 C나 C++ 코드가 조금만 복잡해져도 전혀 알아볼 수가 없겠더라.

```c++
#if condition
    source code
#endif
```

이런 식으로 되어 있으면 condition이 충족되지 않으면 #if ~ #endif 사이에 있는 소스 코드는 컴파일 시에 무시된다.

```c++
#ifdef DEF1
```

이 구문의 뜻은 'DEF1이 #define 되어 있다면~'이다.

그 다음으로 가장 헷갈렸던 것은

```c++
#ifndef HEADERNAME_H ~ #endif
```

이것은 HEAERNAME을 #include할 때 include 되어 있지 않다면~ 하면서 진행하는 조건 전처리문이다. 헤더를 중복해서 선언하지 않기 위해서 쓴다.

## Templates

말 그대로 함수나 클래스의 템플릿, 즉 틀을 만들어주는 것이다. 만약에 두 정수를 인자로 받아 합을 출력해주는 다음과 같은 함수가 있다고 치자.

```c++
int intSum(int a, int b) {
    return a + b;
}
```

그런데 코딩을 하다 보니 int형 뿐만이 아니라 double, float, long, short 타입 모두 합을 구해주는 함수가 필요해진 것이다. 이럴 때 generic한 함수의 템플릿을 다음과 같이 만들 수 있다.

```c++
template <typename T>
T genericSum(T a, T b) {
    return a + b;
}
```

타입 T에는 꼭 원시형 타입만 들어가지 않아도 되고 클래스가 들어가도 된다. 다만 이럴 때에는 클래스 내에 + 연산자에 대한 오버라이드가 이루어져 있어야 한다.

같은 방식으로 클래스 템플릿을 만들 수도 있다.

```c++
template <typename T>
class BasicVector{
    public:
        BasicVector(int capac = 10);
        T& operator[](int i) {return a[i];}
    private:
        T* a;
        int capacity;
};
```

클래스 내 생성자에 대해서도 클래스 본체 바깥에서 템플릿을 정의할 수 있다.

```c++
template <typename T>
BasicVector<T>::BasicVector(int capac) {
    capacity = capac;
    a = new T[capacity];
}
```

이렇게 템플릿을 만들어 두면 어떤 타입이라도 받을 수 있는 클래스가 된다.

또한 클래스 생성자의 인자로 템플릿을 받을 수도 있는데 위와 같이 일종의 배열을 구현한 클래스에서 다음과 같이 한다면

```c++
BasicVector<BasicVector<int> > xy(5);
```

이 클래스의 인덱싱은 마치 배열과 같이 xy[2][5] 이런 식으로 접근할 수 있게 된다. 위에서 \<int> 오른쪽에 공백 문자를 하나 더 넣은 것은 >>이렇게 되면 컴파일러가 비트 연산자로 취급하기 때문이다.

## 예외처리 특정하기

어떤 함수들은 실행하기 전부터 어떤 예외를 처리해야 할지 감이 오는 경우가 있다. 이럴 때 다음과 같이 예상되는 예외처리를 특정해주면 추후에 함수를 쓰는 사람이 예상되는 예외처리에 대비할 수 있고 함수 내에서 굳이 try catch문을 쓰지 않아도 미리 정의해둔 Exception 클래스에서 예외처리를 해준다.

```c++
void calculator() throw(ZeroDivide, NegativeRoot) {

}
```

오늘은 여기까지..... 120페이지까지 보았다. 내일부터는 본격적으로 자료구조에 대해 배운다. 배열, 단일 연결 리스트, 이중 연결 리스트, 순환 연결 리스트, 그리고 재귀에 대해서 배운다.
