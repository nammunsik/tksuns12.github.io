---
title: "(TIL) C++ Implementation of Priority Queue with Heap ~"
last_modified_at: 2020-05-21T10:40:58-05:00
category: 
    - TIL
tags:
    - TIL
    - C++
    - Data Structure
---

## Heap을 사용하여 우선순위큐 구현하기

### 삽입

삽입은 Heap에서 가장 마지막 노드에 한다. 그런 다음 루트 노드를 만날 때까지 혹은 힙을 만족할 때까지 부모 노드의 key와 비교하면서 swap을 해나가게 된다. 이렇게 가장 아래의 노드에서 위로 올라가며 힙 조건을 만족해나가는 것을 up-heap bubbling이라고 한다.

### 최소key 삭제

최소 key를 가지고 있는 루트를 지워버리면 편하겠지만 루트를 지우는 순간 트리 구조가 깨지기 때문에 삭제를 할 때에는 루트와 마지막 노드를 교체한 후 마지막 노드를 삭제한다. 그런 다음 교체된 루트 노드를 마지막 노드를 만날 때까지 혹은 힙 조건이 만족될 때까지 자식 노드와 swap 해 나가게 된다. 이렇게 위에서부터 아래로 내려가면서 힙 조건을 만족시켜나가는 것을 down-heap bubbling이라고 한다.

### 시간복잡도

size, empty, min 연산은 상수시간이 걸리고 삽입과 최소 key 삭제 연산은 로그 시간이 걸린다.

### C++에서의 구현

코드는 다음과 같다.

```c++
template <typename E, typename C>
class HeapPriorityQueue {

public:
int size() const; // number of elements
bool empty() const; // is the queue empty?
void insert(const E& e); // insert element
const E& min(); // minimum element
void removeMin(); // remove minimum

private:
VectorCompleteTree<E> T; // priority queue contents
C isLess; // less-than comparator
// shortcut for tree position
typedef typename VectorCompleteTree<E>::Position Position;
};

template <typename E, typename C> // number of elements
int HeapPriorityQueue<E,C>::size() const
{ return T.size(); }
template <typename E, typename C> // is the queue empty?
bool HeapPriorityQueue<E,C>::empty() const
{ return size() == 0; }
template <typename E, typename C> // minimum element
const E& HeapPriorityQueue<E,C>::min()
{ return *(T.root()); } // return reference to root element

template <typename E, typename C> // insert element
void HeapPriorityQueue<E,C>::insert(const E& e) {
T.addLast(e); // add e to heap
Position v = T.last(); // e’s position
while (!T.isRoot(v)) { // up-heap bubbling
Position u = T.parent(v);
if (!isLess(*v, *u)) break; // if v in order, we’re done
T.swap(v, u); // . . .else swap with parent
v = u;
}
}

template <typename E, typename C> // remove minimum
void HeapPriorityQueue<E,C>::removeMin() {
if (size() == 1) // only one node?
T.removeLast(); // . . .remove it
else {
Position u = T.root(); // root position
T.swap(u, T.last()); // swap last with root
T.removeLast(); // . . .and remove last
while (T.hasLeft(u)) { // down-heap bubbling
Position v = T.left(u);
if (T.hasRight(u) && isLess(*(T.right(u)), *v))
v = T.right(u); // v is u’s smaller child
if (isLess(*v, *u)) { // is u out of order?
T.swap(u, v); // . . .then swap
u = v;
}
else break; // else we’re done
}
}
}
```

## 힙 정렬

이전에 우선순위큐를 통하여 정렬을 했던 것처럼 힙을 사용해서도 정렬할 수 있다. 정렬되지 않은 리스트를 힙에 삽입하는 데 logn 시간이 걸리고 힙을 다시 리스트로 옮기는 데 n시간이 걸리므로 총 nlogn 시간이 걸리게 된다.

### In-place 힙 정렬 구현

힙은 반전된 비교자를 사용하여 최대값이 루트에 오게 하고 언제나 삽입은 왼쪽 부분에서 시작한다. 처음에는 빈 힙에다가 리스트의 원소를 하나씩 삽입한 뒤에 힙이 다 만들어지면 removeMax 연산을 사용하여 최대값부터 삭제하면서 뽑아 리스트의 마지막 원소부터 덮어씌운다. 이 작업을 위해 추가적인 메모리 용량을 사용하지 않았으므로 이 알고리즘을 In-place하다고 한다.

## 상향식 힙 구성

힙을 구성할 때 원소를 하나씩 삽입하여 구성하면 원소의 개수를 n이라고 했을 때 O(nlogn)시간이 걸리는 것을 확인했다. 그러면 원소를 하나씩 삽입하는 것이 아니라 주어진 리스트가 있을 때 이것을 한 번에 힙으로 바꿀 수 있을까? 이때 사용하는 것이 상향식 힙 구성이다. 리스트 원소의 개수 n이 2의 승수 - 1만큼 있다고 했을 때에 가장 처음에 (n+1)/2 만큼의 원소를 바닥에 깐다. 그리고 또 그것의 반만큼의 원소를 그 위에 부모 노드로 깔아주고 힙 조건을 맞춘다. 이렇게 재귀적으로 루트 노드까지 올라가게 되면 힙을 구성할 수 있다. 이 방식은 O(n)시간밖에 안 걸리므로 시간효율적이다.

오늘은 여기까지...  내일은 Adaptable Priority Queues부터 해야겠다. 이 부분은 오늘 이미 공부했지만 잘 이해되지 않아서 글 정리는 내일 하는 게 좋겠다.