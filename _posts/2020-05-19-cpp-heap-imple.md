---
title: "(TIL) C++ Implementation of Priority Queue~"
last_modified_at: 2020-05-19T09:38:58+09:00
category: 
    - TIL
tags:
    - TIL
    - C++
    - Data Structure
---
# STL List를 통한 우선 순위 큐의 구현

우선 순위 큐를 List로 구현할 때에는 이 List를 정렬 할지 하지 않을지에 따라 방법이 갈린다.

## 정렬되지 않은 리스트로 구현

정렬되지 않은 채로 둘 거라면 insert 연산 같은 경우에는 그냥 List의 마지막 원소 다음에 넣으면 되므로 O(1) 시간이 걸린다. 그래서 단순 삽입이 아닌 removeMin과 같이 key에 의존하는 연산의 경우에는 O(n) 시간이 얄짤없이 걸리게 된다. 그리고 최선의 경우에도 O(n) 시간이 언제나 걸리게 되는데 이것은 언제나 가장 작은 key를 가진 하나의 원소를 전체 List를 탐색해서 찾아야 하기 때문이다. 모든 원소를 탐색하지 않으면 최소 key를 가졌다는 것을 보장할 수 없다. 그래서 다시 말하면 key와 관련된 연산은 Θ(n) 시간이 걸린다. 정리해서 말하자면 삽입 연산은 상수 시간이 걸리고 탐색과 삭제에는 선형 시간이 걸린다.

## 정렬된 리스트로 구현

반대로 정렬된 리스트로 구현하기 위해서는 insert 연산이 이루어질 때마다 key를 모두 탐색하여 삽입코자 하는 원소의 key가 들어갈 자리를 찾아야 하므로 O(n)시간이 걸린다. 그러나 removeMin 연산은 무조건 맨 앞의 원소를 제거하면 되므로 O(1) 시간이 걸린다.

## C++에서 리스트로 우선 순위 큐 구현

코드는 다음과 같다. 샘플 코드라 예외 체크와 같이 몇 가지가 생략되어 있다.

```c++
template <typename E, typename C>
class ListPriorityQueue {
public:
int size() const; // number of elements
bool empty() const; // is the queue empty?
void insert(const E& e); // insert element
const E& min() const; // minimum element
void removeMin(); // remove minimum
private:
std::list<E> L; // priority queue contents
C isLess; // less-than comparator
};

template <typename E, typename C> // number of elements
int ListPriorityQueue<E,C>::size() const
{ return L.size(); }
template <typename E, typename C> // is the queue empty?
bool ListPriorityQueue<E,C>::empty() const
{ return L.empty(); }

template <typename E, typename C> // insert element
void ListPriorityQueue<E,C>::insert(const E& e) {
typename std::list<E>::iterator p;
p = L.begin();
while (p != L.end() && !isLess(e, *p)) ++p; // find larger element
L.insert(p, e); // insert e before p
}

template <typename E, typename C> // minimum element
const E& ListPriorityQueue<E,C>::min() const
{ return L.front(); } // minimum is at the front
template <typename E, typename C> // remove minimum
void ListPriorityQueue<E,C>::removeMin()
{ L.pop front(); }
```

insert 함수를 보면 알 수 있듯이 이것은 정렬된 리스트로 구현된 우선순위큐이다. 첫 번째 노드를 p로 정하고 삽입하고자 하는 원소 e보다 큰 p의 원소값 즉 *p가 나올 때까지 p를 한 칸씩 움직이며 탐색한다.

## Selection sort 와 Insertion sort

### Selection Sort

Selection Sort는 정렬되지 않은 우선순위 큐에서 최소값을 반복적으로 추출하여 새로운 리스트에 순서대로 집어넣어 정렬된 리스트를 얻는 것이다. n개의 원소를 가진 우선순위 큐에서 최소값을 찾는 것은 n 시간이 걸리고 그 다음에는 n-1, n-2, ... , 원소가 하나 남았을 때 1만큼의 시간이 걸리므로 등차수열의 합 공식에 의하여 O(n(n+1)/2) 즉 O(n^2) 시간이 걸린다.

### Insertion Sort

만약 우리가 정렬된 우선순위 큐를 사용한다면 총 정렬 시간이 O(n) 시간밖에 걸리지 않을 것이다 왜냐하면 removeMin은 상수 시간이 걸리기 때문이다. 그러나 정렬해야 하는 리스트를 정렬된 우선순위 큐로 만드는 시간 자체가 O(n^2)시간이 걸린다.

# Heap

우선순위큐에 대한 설명을 보면 알겠지만 어떤 방식으로 정렬을 하든 trade-off가 있다. 큐를 만들 때 지수 시간이 걸리든지 아니면 정렬을 할 때 지수 시간이 걸리든지 어떻게든 지수 시간을 피할 수가 없다. 그러나 이 둘 간의 균형을 맞출 수 있는 자료구조가 있다. Heap이다.

## Heap 자료구조

선형적 구조의 리스트의 문제를 해결해준다는 데서 눈치를 챘을 수도 있지만 Heap은 이진트리의 한 종류이다. 각 노드는 원소와 key를 가지고 있고 관계 속성과 구조적 속성을 지니고 있다. 관계 속성은 key가 어떤 방식으로 저장되는지에 관한 것이고 구조적 속성은 노드가 어떻게 정의되는지에 관한 것이다. *참고로 여기서 말하는 heap 자료구조와 동적으로 데이터가 할당되는 메모리상의 heap은 아무 상관이 없다.*

**Heap-Order 속성**: heap T에서 루트를 제외한 모든 노드 v에 대하여 v의 key는 언제나 v의 부모 key보다 크거나 같다.

**완전 이진 트리 속성**: h의 높이를 가진 heap T는 완전 이진 트리이다. 즉, T의 모든 레벨에서 노드는 최대의 자식 노드 수를 가진다. 그리고 자식 노드는 왼쪽에서 오른쪽으로 채워진다.

## Heap의 높이

heap은 완전이진트리이므로 원소의 개수가 n인 heap의 높이 h는 logn이다.

오늘은 여기까지... 내일은 완전이진트리의 표현부터 진도를 나가야겠다.