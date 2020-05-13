---
title: "(TIL) C++ Sequence & Trees"
last_modified_at: 2020-05-12T10:39:12-05:00
category: 
    - TIL
tags:
    - TIL
    - C++
    - Data Structure
---

오늘은 어제에 이어 STL container들에 대해 좀 더 자세히 배웠다.

vector, list, deque와 같은 자료구조들을 sequence container라고 부른다. 왜냐하면 원소들이 순서대로 저장되어 있기 때문이다. set, multiset, map, multimap과 같은 자료구조들을 associated container라고 부른다. 왜냐하면 원소들이 연합된 키값을 통해 접근되기 때문이다.

## Sequence

Sequence는 vector와 list를 아우르는 추상 데이터 타입이다. Sequence는 인덱스로 원소에 접근할 수 있는 데이터 타입을 말한다.

Sequence를 이중 연결리스트로 구현할 수 있다. 이 구현에서는 특정 인덱스를 찾아가려면 양쪽 끝 중 하나를 선택해서 해당하는 인덱스가 나올 때까지 이웃 노드로 뛰어넘어가야 한다. 그래서 최악의 경우에 O(N) 시간 복잡도가 나온다.

Sequence를 배열로 구현할 수 있다. A에 대한 참조와 인덱스를 멤버변수로 가지고 있는 객체 p를 설정하고 element(p) 함수를 구현하면 빠르게 배열의 i 인덱스에 있는 원소에 접근할 수 있다. 그러나 이 방법의 문제점은 각 셀이 대응하는 위치에 대한 참조가 없다는 것이다. 이 문제를 해결하기 위해서 배열에 원소를 저장하는 대신 각 인덱스의 배열 원소가 그 원소를 가리키는 포인터가 되도록 하는 것이다. 즉 실질적인 sequence의 내용물은 배열 바깥에 있고 배열은 단지 그 내용물의 주소만 저장하고 있는 것이다. 이렇게 하면 insert와 erase 연산은 O(N) 시간 복잡도를 가지고 나머지는 모두 O(1) 시간 복잡도를 가지게 된다.

# Tree

Tree란 지금까지 배웠던 선형적 자료구조와 다르게 '계층적' 자료구조이다. Tree에서 나오는 개념은 다음과 같다.

- 부모
- 자식
- 형제
- 뿌리
- 잎
- edge
- 경로

모든 노드는 부모-자식 관계를 맺고 있다. 그리고 같은 부모를 둔 노드끼리는 형제이고 가장 위에 있는 노드를 뿌리라고 한다. 그리고 자식이 없는 노드를 잎이라고 부른다. 또한 (u, v) 이런 식으로 서로 부모-자식 관계인 노드의 쌍을 edge라고 하며 이런 edge들의 집합을 경로라고 한다. C++에서 Tree 인터페이스를 작성할 때에는 외부에서 노드에 직접 접근하게 하는 것보다 
Position이라는 클래스를 새로 만들어 오버로딩된 * 연산자를 이용해 노드에 접근하게 한다. Position 타입의 원소를 가진 List를 생성해 트리를 표현하게 된다.

그러나 트리의 구조를 생각해보면 연결 구조로 구현하는 게 더 자연스러워 보인다. 각 position 객체는 노드의 원소와 parent, child를 가리키는 포인터를 멤버 변수로 가지게 된다.

## 트리 순회 알고리즘

1. 노드의 깊이
   
   노드의 깊이는 노드가 루트 노드까지 몇 개의 부모 노드를 거쳐가야 하는지를 나타낸 숫자이다. 노드의 깊이를 알아내는 코드는 다음과 같은 재귀 함수이다.

```c++
int depth(const Tree& T, const Position& p) {
if (p.isRoot())
return 0; // root has depth 0
else
return 1 + depth(T, p.parent()); // 1 + (depth of parent)
}
```

2. 트리의 높이
   
트리의 높이는 잎 노드의 최고 깊이와 같다. 트리의 높이를 측정하는 재귀 함수는 다음과 같다.

```c++
int height2(const Tree& T, const Position& p) {
if (p.isExternal()) return 0; // leaf has height 0
int h = 0;
PositionList ch = p.children(); // list of children
for (Iterator q = ch.begin(); q != ch.end(); ++q)
h = max(h, height2(T, *q));
return 1 + h; // 1 + max height of children
}
```

위 재귀함수의 시간 복잡도를 계산해보면 세번째 줄까지 일단 O(p.children.size)만큼이고 for문을 보면 또 p.children.size 만큼 돌아가고 안에 비교 연산 1, 그리고 비교 연산 내부의 재귀 함수에서 또 자식 노드들이 새로운 부모 노드가 되어 자식 노드들을 탐색한다. 그래서 시간 복잡도는 모든 부모 노드들에 대해 1 + children.size를 sum한 것이고 이것은 n-1이다. 왜냐하면 모든 부모 노드들에 대해 자식 노드의 개수를 합하면 결국 root 노드를 제외한 모든 노드가 되기 때문이다. 그래서 시간 복잡도는 O(n)이다.

오늘은 여기까지..... 내일은 preorder traverse부터 시작하면 되겠다. 278쪽부터이다.