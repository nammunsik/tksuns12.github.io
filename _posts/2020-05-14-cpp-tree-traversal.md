---
title: "(TIL) C++ Tree Traversal ~"
last_modified_at: 2020-05-14T09:38:12+09:00
category: 
    - TIL
tags:
    - TIL
    - C++
    - Data Structure
---

## Preorder Traversal

순회(traversal)란 트리의 노드를 체계적으로 방문하는 방법이다. 그 중 전위 순회(preorder traversal)는 부모 노드를 먼저 방문한 후 자식 노드를 순서대로 재귀적으로 방문하는 방식이다.

## Postorder Traversal

후위 순회(postorder traversal)은 자식 노드를 모두 방문한 후 부모 노드를 방문하는 방식이다. 후위 순회는 부모 노드의 속성이 자식 노드의 속성에 의해 결정되는 트리에서 부모 노드의 속성을 알아내야 할 때 유용하다.

# 이진트리(Binary Trees)

이진 트리는 다음과 같은 특징을 가진 '순서가 있는' 트리이다.

- 모든 노드는 최대 두 개의 자식 노드를 가진다.
- 모든 자식 노드는 왼쪽 자식, 오른쪽 자식으로 이름 붙여진다.
- 왼쪽 자식이 오른쪽 자식에 선행한다.

이진 트리는 다음과 같이 재귀적으로 표현할 수도 있다. 부모 노드, 그리고 왼쪽 서브 이진 트리, 오른쪽 서브 이진 트리. 그리고 재귀적 표현이기 때문에 각 서브 이진 트리도 그렇게 표현된다.

## 이진 트리의 추상 데이터 타입

이진 트리는 이전의 다른 노드 기반 자료구조들과 마찬가지로 각 노드는 원소를 가지고 있고 이 노드는 **position**이라는 객체와 연결되어있다. \* 연산자를 오버로딩 함으로써 \*position으로 노드에 접근할 수 있다. position *p*는 다음과 같은 연산을 지원한다.

- p.left() 노드의 왼쪽 자식을 반환한다.
- p.right() 노드의 오른쪽 자식을 반환한다.
- p.parent() 노드의 부모 노드를 반환하다.
- p.isRoot() 노드가 루트 노드인지 여부를 반환한다.
- p.isExternal() 노드가 잎 노드인지 반환한다.
  
그리고 트리 자체도 몇 가지 연산을 지원한다.

- size() 트리 내 노드의 개수를 반환한다.
- empty() 트리가 비어있는지 아닌지를 반환한다.
- root() 루트 노드를 반환한다.
- positions() 트리 내 모든 노드의 position 객체를 리스트 형태로 반환한다.

C++에서의 인터페이스 구현은 다음과 같다.

```c++
template <typename E> // base element type

class Position<E> { // a node position

public:
E& operator*(); // get element
Position left() const; // get left child
Position right() const; // get right child
Position parent() const; // get parent
bool isRoot() const; // root of tree?
bool isExternal() const; // an external node?
};

template <typename E> // base element type

class BinaryTree<E> { // binary tree

public: // public types
class Position; // a node position
class PositionList; // a list of positions

public: // member functions
int size() const; // number of nodes
bool empty() const; // is tree empty?
Position root() const; // get the root
PositionList positions() const; // list of nodes
};
```

## 이진 트리의 속성

이진 트리는 자식 노드의 최대 개수가 정해져 있다는 점에서 노드의 깊이와 노드 개수 간의 관계에 특징이 있다. 그 특징은 다음과 같다.

1. h+1 ≤ n ≤ 2^(h+1) −1
2. 1 ≤ nE ≤ 2^h
3. h ≤ nI ≤ 2^h −1
4. log(n+1)−1 ≤ h ≤ n−1

nE는 잎 노드 nI는 일반 노드를 말한다.

오늘은 여기까지.... 내일은 이진 트리의 구현에 대해서 배운다.