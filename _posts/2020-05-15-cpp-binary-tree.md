---
title: "(TIL) C++ Implementation of Binary Tree ~"
last_modified_at: 2020-05-15T09:54:12+09:00
category: 
    - TIL
tags:
    - TIL
    - C++
    - Data Structure
---

## 연결 구조로 이진 트리 구현

가장 자연스러운 방식으로 이진트리를 구현하는 방법은 연결 구조로 만드는 것 같다. 한 노드는 네 개의 멤버 변수를 가지고 있는데 parent, left, right, 그리고 원소를 나타내는 element이다. 구현이 굉장히 직관적이고 간단해 보인다. 코드는 다음과 같다.

```c++
struct Node { // a node of the tree
Elem elt; // element value
Node* par; // parent
Node* left; // left child
Node* right; // right child
Node() : elt(), par(NULL), left(NULL), right(NULL) { } // constructor
};

class Position { // position in the tree
private:
Node* v; // pointer to the node
public:
Position(Node* v = NULL) : v( v) { } // constructor
Elem& operator*() // get element
{ return v−>elt; }
Position left() const // get left child
{ return Position(v−>left); }
Position right() const // get right child
{ return Position(v−>right); }
Position parent() const // get parent
{ return Position(v−>par); }
bool isRoot() const // root of the tree?
{ return v−>par == NULL; }
bool isExternal() const // an external node?
{ return v−>left == NULL && v−>right == NULL; }
friend class LinkedBinaryTree; // give tree access
};
typedef std::list<Position> PositionList; // list of positions

typedef int Elem; // base element type
class LinkedBinaryTree {
protected:
// insert Node declaration here. . .
public:
// insert Position declaration here. . .
public:
LinkedBinaryTree(); // constructor
int size() const; // number of nodes
bool empty() const; // is tree empty?
Position root() const; // get the root
PositionList positions() const; // list of nodes
void addRoot(); // add root to empty tree
void expandExternal(const Position& p); // expand external node
Position removeAboveExternal(const Position& p); // remove p and parent
// housekeeping functions omitted. . .
protected: // local utilities
void preorder(Node* v, PositionList& pl) const; // preorder utility
private:
Node* root; // pointer to the root
int n; // number of nodes
};

LinkedBinaryTree::LinkedBinaryTree() // constructor
: root(NULL), n(0) { }
int LinkedBinaryTree::size() const // number of nodes
{ return n; }
bool LinkedBinaryTree::empty() const // is tree empty?
{ return size() == 0; }
LinkedBinaryTree::Position LinkedBinaryTree::root() const // get the root
{ return Position( root); }
void LinkedBinaryTree::addRoot() // add root to empty tree
{ root = new Node; n = 1; }

void LinkedBinaryTree::expandExternal(const Position& p) {
Node* v = p.v; // p’s node
v−>left = new Node; // add a new left child
v−>left−>par = v; // v is its parent
v−>right = new Node; // and a new right child
v−>right−>par = v; // v is its parent
n += 2; // two more nodes
}

LinkedBinaryTree::removeAboveExternal(const Position& p) {
Node* w = p.v; Node* v = w−>par; // get p’s node and parent
Node* sib = (w == v−>left ? v−>right : v−>left);
if (v == root) { // child of root?
root = sib; // . . .make sibling root
sib−>par = NULL;
}
else {
Node* gpar = v−>par; // w’s grandparent
if (v == gpar−>left) gpar−>left = sib; // replace parent by sib
else gpar−>right = sib;
sib−>par = gpar;
}
delete w; delete v; // delete removed nodes
n −= 2; // two fewer nodes
return Position(sib);
}

LinkedBinaryTree::PositionList LinkedBinaryTree::positions() const {
PositionList pl;
preorder( root, pl); // preorder traversal
return PositionList(pl); // return resulting list
}
// preorder traversal
void LinkedBinaryTree::preorder(Node* v, PositionList& pl) const {
pl.push back(Position(v)); // add this node
if (v−>left != NULL) // traverse left subtree
preorder(v−>left, pl);
if (v−>right != NULL) // traverse right subtree
preorder(v−>right, pl);
}
```

마지막 두 업데이트 함수를 보면 먼저 expandExternal 함수는 외부 노드에 왼쪽 자식과 오른쪽 자식을 추가하여 외부노드를 내부노드화 하는 함수이다. removeAboveExternal 함수는 어떤 외부노드를 그 부모와 같이 삭제하고 부모 자리를 외부노드가 아닌 형제 노드로 대체하는 함수이다.

모든 연산의 시간복잡도는 O(1)이나 positions 함수의 시간 복잡도는 O(n)이다. n은 노드의 개수이다.

## 벡터 기반 구조로 이진 트리 구현

일종의 리스트라 볼 수 있는 벡터로 이진 트리를 구현할 수 있다. 벡터는 노드와 달리 parent나 child를 가리킬 수 없기 때문에 다음과 같은 규칙으로 구분한다.

v가 트리의 루트라면 v.index = 1
v가 u의 왼쪽 자식이라면 v.index = 2*u.index
v가 u의 오른쪽 자식이라면 v.index = 2*u.index + 1

하지만 이 경우에 중간에 비는 인덱스가 있을 수 있어 공간 낭비가 심각할 수 있다.

## 이진 트리의 순회

이진 트리 또한 트리이기 때문에 전위 순회와 후위 순회 모두 가능하다. 그리고 이진 트리는 자식이 둘밖에 없기 때문에 중위 순회라는 것도 가능하다. 코드들은 다음과 같다.

특히 후위 순회는 상향식 문제 해결에 쓰면 좋다. 중위 순회는 트리를 바닥에 그대로 투사한 것처럼 순서대로 원소를 왼쪽부터 오른쪽으로 탐색한다.

## 이진 탐색 트리

이진 탐색 트리는 이진 트리 + 원소의 관계성이다. 왼쪽 서브트리에 있는 원소들은 자기 자신보다 언제나 작고 오른쪽 서브트리에 있는 원소들은 언제나 자기 자신보다 크다. 그렇기 때문에 원소를 탐색할 때 타겟 원소보다 작거나 크거나에 따라 어느 서브트리로 탐색해야 할지가 갈리기 때문에 탐색 시간이 O(logn) 시간으로 매우 효율적이다.

## 오일러 투어

오일러 투어는 순회 알고리즘의 하나이다. 이것은 마치 트리를 루트부터 외곽선을 따라 빙 둘러서 순회하는 것처럼 보인다. 그래서 각 노드는 노드의 왼쪽에서 한 번, 아래에서 한 번, 오른쪽에서 한 번 방문된다. 외부 노드 같은 경우에는 이 세 가지 방문이 한 번에 이루어진다. 왼쪽 방문만 기록하면 전위 순회와 같고 아래 방문만 기록하면 중위, 오른쪽 방문만 기록하면 후위 순회와 같다. 그런 점에서 오일러 투어는 전위, 중위, 후위 순회의 일반화된 형태이다. 게다가 다른 형태의 순회도 오일러 투어를 이용해서 할 수 있다.

## 모든 트리를 이진 트리로 표현하기

이 문제는 왼쪽 자식 오른쪽 형제 표기법이라고 보면 된다. 어떤 노드의 맨 첫 번째 자식을 왼쪽 자식으로 표현하고 그에 딸린 모든 형제들을 오른쪽 자식으로 쭉 나열하는 방식이다. 이렇게 하면 모든 트리를 이진 트리로 일관되게 표현할 수 있다.

오늘은 여기까지... 내일은 우선순위 큐와 힙에 대해서 배우기 시작한다.