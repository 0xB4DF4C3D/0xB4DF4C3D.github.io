---
comments: true
layout: post
title: Sentinel value
category: [programming]
---

https://en.wikipedia.org/wiki/Sentinel_value 를 번역한 글이다. 오역, 의역 있을 수 있으니 주의.

> *[Sentinel node](https://en.wikipedia.org/wiki/Sentinel_node)  또는 [Canary value](https://en.wikipedia.org/wiki/Canary_value) 와 혼동하지 말것._*

컴퓨터 프로그래밍에서 Sentinel 값(flag value, trip value, rogue value, signal value 또는 dummy data등으로도 불림)은 보통 루프나 재귀 알고리즘같은 곳에서 종료 조건으로 쓰이는 알고리즘의 특수한 값이다.
_(역주: Sentinel이란 뜻을 한국어로 마땅히 쓸말이 없어 그대로 표기함. 일관성을 위해 다른 명칭도 동일하게 적용.)_

Sentinel 값은 out-of-band 데이터(명시적으로 크기를 나타낸)가 주어지지 않았을 때 데이터의 끝을 식별하게끔 해주는 in-band 데이터의 한 형태이다. 그 값은 다른 유효한 모든 데이터 값들로부터 구분되어짐이 보장되어야 한다. 그렇지 않으면 그 값의 존재로 인해 데이터가 중간에 끝날 수 있기 때문이다. Sentinel 값은 가끔 "Elephand in Cairo"라고도 알려지는데 왜냐면 이 농담이 물리적 Sentinel로 사용되기 때문이다.(굳이 농담이 뭔지 알고 싶다면 [여기](https://en.wikipedia.org/wiki/Elephant_in_Cairo)를 참고할 것) 조금 더 보충하면 Sentinel 값을 사용하는 대부분의 경우는  [option types](https://en.wikipedia.org/wiki/Option_type)로 치환함으로써 예외적인 경우를 좀 더 명시적으로 처리할 수 있다.

# 예제

일반적인 Sentinel 값들과 그것들의 사용처들 중 일부:
- _널 종료 문자열_의 끝을 나타내는데 쓰이는 _널 문자_.
- _링크드 리스트_ 또는 _트리_ 의 끝을 나타내는 _널 포인터_
- 양수 시퀀스 값들의 끝을 나타내기 위한 음수.

# 응용

약간 다른 상황에서 사용되는 비슷한 방법으로는 데이터의 끝에 특정 값을 추가함으로써 몇몇 처리 루프의 끝을 명시적으로 검사하지 않아도 되게끔 하는 것이다. 왜냐하면 그 값은 이미 다른 이유로 존재하던 테스트들에 의해 루프를 종료시키기 때문이다. 위의 용도와 달리, 이 방법은 데이터가 자연스럽게 저장되거나 처리되는 방식이 아니라 종료를 확인하는 간단한 알고리즘과 비교하여 좀 더 최적화된 방식이다._(역주:나도 이게 뭔말인지 모르겠다.)_

예를 들면, 정렬되지 않은 리스트에서 특정 값을 찾고자 할 때 모든 원소들은 루프를 돌며 같은 값이 나올때까지 그 값과 비교된다. 하지만 그 값이 없을 경우에도 처리해주기 위해 반드시 각 스텝이 끝난 뒤 탐색이 끝났는지 검사도 해주어야 한다. 리스트의 끝에 그 값을 추가해줌으로써 탐색 실패는 더이상 일어나지 않는다. 또한 더이상 내부 루프에서 명시적인 종료 검사가 필요하지 않다. 이후에 이게 진짜 원소가 찾아진건지 아니면 그냥 마지막에 추가된 원소와 매치된건지 확인해야 하지만 매 스텝마다 수행하는 대신 마지막에 한번만 수행하면 된다는 이점이 있다. Knuth(_The Art of Computer Programming의 저자_)는 데이터의 마지막에 배치된 값을 sentinel이라 부르는 대신 dummy 값이라 부른다.

## 예제
### 배열
예를 들어 C에서 배열의 원소를 탐색할 때 간단한 구현법은 다음과 같다; 아무 결과도 반환하지 않는 semipredicate problem을 해결하기 위해 음수(유효하지 않은 인덱스)를 사용한 것에 유의하라.

```c
// Returns index of value, -1 for no result
int find(int* a, int l, int v)
{
  int i;
  for (i = 0; i < l; i++)
    if (a[i] == v)
      return i;
  return -1; // -1 means "no result"
}
```

그런데 이 방법은 각 스텝마다 검사를 두번한다: 값을 찾았는지, 배열의 끝인지. 후자의 검사는 sentinel값을 통해 생략할 수 있다. 배열이 원소 1개만큼은 확장될 수 있다는 가정하에 (메모리 할당이나 해제 없이; 이것은 아래처럼 링크드리스트에서 더 현실적이다.) 다음처럼 다시 쓰여질 수 있다:

```c
int find(int* a, int l, int v)
{
  int i;
  // add sentinel item:
  a[l] = v; // prepare it with sentinel value
  for (i = 0; ; i++)
    if (a[i] == v) {
      if (i != l) // real result
        return i;
      // was sentinel value, not real result:
      return -1;
    }
}
```

이 경우 각 스텝은 한번의 검사면 된다. (찾으려는 값과 일치하는지), 그리고 sentinel 값을 통해 종료를 보장한다. 매 스텝마다 데이터의 끝을 확인하는 대신 루프 종료시, sentinel 값이 매치되었는지만 확인하는 검사 한번이면 된다.

이 경우엔 루프가 while을 사용하여 좀 더 간단하게 표현될 수 있다:

```c
int find(int* a, int l, int v)
{
  int i;
  // add sentinel item:
  a[l] = v; // prepare it with sentinel value
  i = 0;
  while (a[i] != v)
    i++;
  if (i != l) // real result
    return i;
  // was sentinel value, not real result:
  return -1;
}
```

### 링크드리스트
링크드리스트에서 탐색할 때, 다음은 주어진 헤드 노드부터 시작하는 간단한 알고리즘이다; semipredicate 문제를 해결하기 위해 NULL을 사용한것에 유의하라:

```c
typedef struct Node{
  Node* next;
  int value;
} Node;

// Returns pointer to node with value, NULL for no result
Node* find(Node* n, int v)
{
  if (n->value == v)
    return n;

  while(n->next!=NULL) {
    n = n->next;
    if (n->value == v)
      return n;
  }
  return NULL;
}
```

하지만 만약 마지막 노드가 알려져 있다면, 먼저 마지막 노드 다음에 sentinel 노드를 추가함으로써  최적화 될 수 있다:

```c
typedef struct List {
  Node* firstElement;
  Node* lastElement;
} List;

Node* find(List* l, int v)
{
  Node *n, sentinelNode;
  // Add sentinel node:
  l->lastElement->next = &sentinelNode;
  sentinelNode.value = v; // prepare sentinel node with sentinel value

  // main loop
  n = l->firstElement;
  while (n->value != v)
    n = n->next;

  // termination
  l->lastElement->next = NULL; // clean up
  if (n != &sentinelNode) // real result
    return n;
  // was sentinel node, not real result:
  return NULL;
}
```
이 방법은 sentinel node를 식별하기 위한 고유값을 제공하는 메모리 주소에 의존한다는 것을 유의하라; 보통 이것은 구현하는데 있어 지양되는 방법이다.

# 같이보면 좋은 것들
- [Canary value](https://en.wikipedia.org/wiki/Canary_value)
- [Sentinel node](https://en.wikipedia.org/wiki/Sentinel_node)
- [semipredicate problem](https://en.wikipedia.org/wiki/Semipredicate_problem)
- [Elephant in Cairo](https://en.wikipedia.org/wiki/Elephant_in_Cairo)
- [Magic number (programming)](https://en.wikipedia.org/wiki/Magic_number_%28programming%29)
- [Magic string](https://en.wikipedia.org/wiki/Magic_string)
- [Null Object pattern](https://en.wikipedia.org/wiki/Null_Object_pattern)
- [Time formatting and storage bugs](https://en.wikipedia.org/wiki/Time_formatting_and_storage_bugs)

참고 자료 :
0. https://en.wikipedia.org/wiki/Sentinel_value
1. Knuth, Donald (1973). The Art of Computer Programming, Volume 1: Fundamental Algorithms (second edition). Addison-Wesley. pp. 213–214, also p. 631. ISBN 0-201-03809-9.
2. Mehlhorn, Kurt; Sanders, Peter (2008), Algorithms and Data Structures: The Basic Toolbox 3 Representing Sequences by Arrays and Linked Lists (PDF), Springer, ISBN 978-3-540-77977-3 p. 63
3. McConnell, Steve. "Code Complete" Edition 2 Pg. 621 ISBN 0-7356-1967-0
4. Knuth, Donald (1973). The Art of Computer Programming, Volume 3: Sorting and searching. Addison-Wesley. p. 395. ISBN 0-201-03803-X.


---

이 글은 Using a mutable default value as an argument 라는 글을 번역하면서 Sentinel value라는 용어가 나왔는데 내가 모르기도 하고 한국어 자료가 얼마 없는것 같아서 얕은 지식이지만 나름 번역해본 것이다. 몇몇 문장은 매끄럽지 못한 부분이 있는데 개선할만한 부분을 찾았다면 지적해주길 바란다.
