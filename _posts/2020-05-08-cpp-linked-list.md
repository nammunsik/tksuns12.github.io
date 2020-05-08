---
title: "(TIL) C++ 연결 리스트"
last_modified_at: 2020-05-08T11:15:30-05:00
category: 
    - TIL
tags:
    - TIL
    - C++
    - Data Structure
---

# 연결 리스트(Linked List)

이틀 전에는 배열을 사용하여 원소를 추가, 제거하는 법을 배웠다. 하지만 이 방식의 큰 문제점이 무엇이냐 하면 추가/제거 할 때 원소들을 오른쪽/왼쪽으로 하나씩 다 밀어줘야 한다는 뜻이고 이 작업은 매우 비쌀 수 있다. 그리고 원소를 추가하다가 배열의 크기를 넘어서게 되면 사실 상 '우아한' 방식으로 해결할 방법은 없다고 봐야 한다. 더 큰 크기를 가진 배열을 새로 생성하여 모든 원소를 복사해야 한다. 배열이 이렇게밖에 만들어질 수 없는 이유는 언제나 배열의 원소들은 연속된 메모리 공간을 차지하기 때문이다. 그래서 함부로 배열의 크기를 늘릴 수 없다. 왜냐하면 배열의 마지막 원소 다음 메모리 공간이 이미 사용되고 있을 가능성이 매우 높기 때문이다.

이 문제를 해결하기 위한 자료구조가 바로 연결 리스트이다. 모든 원소를 연속적인 메모리 공간에 배치하는 대신 '노드'라는 개념을 사용하여 노드에 원소를 저장하고 노드 안의 다른 공간에 다음 원소의 주소, 즉 C++에서는 포인터를 저장하는 것이다. 이렇게 되면 다음 원소가 메모리 공간 어디에 있든지 상관 없이 현재 노드에 저장되어 있는 메모리 주소로 즉시 넘어갈 수 있다(link hopping).

연결 리스트는 세 가지 종류가 있는데 단순 연결리스트, 이중 연결리스트, 순환 연결리스트이다. 어차피 원리는 다 비슷하므로 단순 연결리스트의 코드만 보고 나머지는 말로만 정리해야겠다.

먼저 단순 연결리스트를 구현하려면 노드를 정의해야 한다.

```c++
class StringNode { // a node in a list of strings
private:
string elem; // element value
StringNode* next; // next item in the list
friend class StringLinkedList; // provide StringLinkedList access
};
```

노드는 원소인 elem과 다음 노드를 가리키는 포인터인 next를 멤버 변수로 가지고 StringLinkedList를 friend 클래스로 두고 있다. 이렇게 하면 연결 리스트 클래스에서 노드 내의 private 멤버들에 접근할 수 있다.

그리고 StringLinkedList의 구현을 보자.

```c++
class StringLinkedList { // a linked list of strings
public:
StringLinkedList(); // empty list constructor
˜StringLinkedList(); // destructor
bool empty() const; // is list empty?
const string& front() const; // get front element
void addFront(const string& e); // add to front of list
void removeFront(); // remove front item list
private:
StringNode* head; // pointer to the head of list
};
```

보면 bool값을 반환하는 empty 함수가 있다. 리스트가 비어있는데 remove를 호출하면 안 되기 때문에 있는 것이다. 배열과는 다른 점은 배열은 원소의 갯수가 배열의 최대 크기에 도달하였는지를 확인하는 함수가 있었는데 연결리스트는 최대 크기라는 것이 없기 때문에 그런 함수는 필요 없다. 그리고 단순 연결 리스트는 언제나 한 원소에서 다음 원소를 참조할 수밖에 없기 때문에 언제나 원소를 앞쪽에서 추가/제거하게 된다. 일종의 Stack이라 보면 될 것 같다. 나중에 추가된 원소가 먼저 제거된다(Last In First Out).

구체적인 클래스의 구현은 다음과 같다.

```c++
StringLinkedList::StringLinkedList() // constructor
: head(NULL) { }

StringLinkedList::˜StringLinkedList() // destructor
{ while (!empty()) removeFront(); }

bool StringLinkedList::empty() const // is list empty?
{ return head == NULL; }

const string& StringLinkedList::front() const // get front element
{ return head−>elem; }

void StringLinkedList::addFront(const string& e) { // add to front of list
StringNode* v = new StringNode; // create new node
v−>elem = e; // store data
v−>next = head; // head now follows v
head = v; // v is now the head
}

void StringLinkedList::removeFront() { // remove front item
StringNode* old = head; // save current head
head = old−>next; // skip over old head
delete old; // delete the old head
}
```

다른 함수들은 보기에 매우 직관적이니 넘어가도록 하고 addFront와 removeFront 이 두 함수들이 바로 연결 리스트의 연산을 잘 보여준다. 원소를 추가하거나 제거할 때 나머지 원소들을 한 칸씩 밀 필요 없이 그냥 포인터만 바꿔서 이어주면 된다.

이중 연결리스트는 단순 연결리스트와 달리 노드에 이전 노드의 메모리 주소를 가리키는 포인터 변수 previous(보통 prev라고 적는다)를 저장하여 앞뒤로 탐색할 수 있게 한다. 그리고 단순 연결리스트에서 첫 번째 원소가 head가 가리키는 원소이듯 이중 연결리스트에서는 마지막 원소가 아무것도 담겨있지 않은 trailer 노드를 가리킨다. 그래서 어떤 노드의 next의 elem이 null을 반환하면 이 연결 리스트의 마지막에 도달했다는 것을 알 수 있다. 그리고 이중 연결 리스트에서는 addFront와 addBack 함수가 있고 이를 이용하여 특정 원소 앞에 원소를 끼워넣거나 특정 원소를 지우는 것도 가능하다.

순환 연결리스트는 단순 연결 리스트와 원리는 비슷한데 head가 따로 없고 마지막 노드의 포인터가 바로 첫 노드를 가리킨다. 순환 연결 리스트에서 나오는 개념은 cursor인데 현재 참조하고 있는 노드를 뜻한다. 이 cursor가 가리키는 노드를 front라고 하고 이 바로 다음 노드를 back이라 한다. 또 advance 함수를 구현하여 cursor를 다음 노드로 이동할 수 있다.

# 재귀(Recursion)

으아... 드디어 재귀를 배우게 됐다. 역시 다른 현대 언어들에 비해 low-level인 C++의 책이어서 그런지 재귀에 대해서 이때까지 내가 본 어떤 책들보다 자세히 설명하고 있었다. 학교에서 배울 때도 재귀가 가장 이해하기 힘들었는데 이번 기회에 더 깊이 이해할 수 있도록 노력해야겠다.

## 선형 재귀(Linear Recursion)

선형 재귀란 함수가 한 번 호출될 때 한 번의 재귀호출이 일어나는 재귀를 말한다. 언제나 종료 조건을 테스트하면서 시작하고 그 다음 재귀한다.

### 문제를 재귀 문제로 바꾸는 법

사실 여기에 공식과 같은 방법은 없지만 대원칙이 하나 있다면 문제 하나를 사실은 작은 하위 문제들의 비슷하지만 큰 형태의 문제로 기술하는 것이다.

#### Tail Recursion

Tail Recursion은 어떤 재귀 함수가 선형 재귀이면서 마지막 동작이 재귀 호출인 것을 말한다. 예를 들면 다음 의사코드는 Tail Recursion이다.

```
Algorithm ReverseArray(A,i, j):
Input: An array A and nonnegative integer indices i and j
Output: The reversal of the elements in A starting at index i and ending at j
if i < j then
Swap A[i] and A[ j]
ReverseArray(A,i+1, j −1)
return
```

그러나 다음 의사코드는 그렇지 않다.

```
Algorithm LinearSum(A,n):
Input: A integer array A and an integer n ≥ 1, such that A has at least n elements
Output: The sum of the first n integers in A
if n = 1 then
return A[0]
else
return LinearSum(A,n−1) +A[n−1]
```

왜냐하면 마지막에 재귀 호출을 하고는 있지만 자세히 보면 재귀 호출을 한 다음에 A[n-1]라는 더하기 연산이 있기 때문에 이것은 Tail Recursion이 아니다. 모든 Tail Recursion은 반복문으로 치환할 수 있다.

## 이진 재귀(Binary Recursion)

이진 재귀는 어떤 알고리즘이 두 번의 재귀 호출을 사용하는 재귀이다. 예시 의사 코드는 다음과 같다.

```
Algorithm BinarySum(A,i,n):
Input: An array A and integers i and n
Output: The sum of the n integers in A starting at index i
if n = 1 then
return A[i]
return BinarySum(A,i,⌈n/2⌉) + BinarySum(A,i+⌈n/2⌉,⌊n/2⌋)
```

이진 재귀를 분석하기 위해서는 이진 트리 형태의 Recursion Trace를 사용해야 한다. 이진 트리의 특성상 그 깊이는 O(logN)밖에 안 된다. 그래서 이진 재귀는 공간-효율적이다. 

오늘은 여기까지.... 내일은 분석 도구에 대해서 배우게 된다. 역시 재귀는 아직도 이해가 어렵다. 개념 자체는 알겠는데 이걸 문제해결에 적용하는 것이 아직 어렵다. 자료구조와 알고리즘 공부를 끝내고 본격적으로 연습문제를 풀기 시작하면서 익혀야겠다.