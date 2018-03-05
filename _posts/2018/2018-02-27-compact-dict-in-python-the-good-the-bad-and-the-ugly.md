---
layout: "post"
title: "Compact Dict in Python"
date: "2018-02-27 06:10"
category: Python
---

> **이 글은 <http://techblog.en.klab-blogs.com/archives/19880026.html> 의 글을 번역한 것입니다.**


# 배경

옛날 2015년 1월 PyPy 개발자 블로그에 ordered dictionaries에 대한 기사가 있었다: [Faster, more memory efficient and more ordered dictionaries on PyPy](https://morepypy.blogspot.jp/2015/01/faster-more-memory-efficient-and-more.html)

dict의 삽입 순서 지원과 동시에 메모리 사용량 감소에 대한 지속적인 요구 후에 PyPy의 새로운 버전 2.5.1이 릴리즈 되었다.

다른 소식으로는 누군가가 "[PEP 468](https://www.python.org/dev/peps/pep-0468/): 우리는 placeholder 인자들로써 **kwargs 문법으로 키워드 인자들을 받음으로써 인자들의 순서를 유지할 수 있다" 라고 CPython에 획기적인 아이디어를 제안하였다.

예를 들어 SQLAlchemy 쿼리에서 만약 우리가 `.filter_by(name="methane", age=32)`라고 썼을때 그것이  `WHERE name = "methane" AND age = 32` 또는 `WHERE age = 32 AND name="methane"` 라고 읽을지 어떤 쿼리를 만들지 확신할 방법이 없었다. 이 새로운 변화는 인자의 순서를 보존할 수 있는 추가적인 이점을 제공한다. 

(`filter_by` 그리고 다른 제 2의 해결책들은 특별한 단축 함수들이다. `filter` 메서드를 사용하면 키워드 인자들을 사용하지 않고서 인자들의 순서를 보존할 수 있다.)

이 아이디어를 제안한 사람은 순수 파이썬의 일부였던 `OrderedDict` 클래스를 C에서 재구현하였다. Python 3.5 이후로 `OrderedDict`는 눈에띄게 빨라졌고 더 효율적으로 메모리를 사용하게 되었다. (그가 dict를 개선하는 걸 피한 이유는 기존에 이미 Python Interpreter를 위해 다양하고 복잡한 방법들로 최적화가 되어왔기 때문이었다.)

하지만 C로 재구현 되면서 양방향 링크를 관리하던 `OrderedDict`는 여전히 상당한 오버헤드르 지니고 있었다. 그것은 거의 두배 되는 메모리를 사용했었다. 

```python
Python 3.6.3 |Anaconda custom (64-bit)| (default, Oct 15 2017, 03:27:45) [MSC v.1900 64 bit (AMD64)] on win32
Type "copyright", "credits" or "license()" for more information.
>>> from collections import OrderedDict
>>> import sys
>>> d = {i:i for i in range(100)}
>>> od = OrderedDict((i,i) for i in range(100))
>>> sys.getsizeof(d), sys.getsizeof(od)
(4704, 10016)
```
이것과 다른 요인들로 인해 PEP 468은 보류되었었다. 

즉 모두 정리하면 몇몇 경우에 그것이 유용할지라도 모든 키워드 인자들의 퍼포먼스가 저하될 우려가 있다면 스펙을 변경하는 아이디어는 나를 포함하여 저항이 심했었다. 나는 또한 PyPy의 dict에도 관심을 가지고 있어서 충분한 시간만 주어진다면 Python 3.6 때에는 그 아이디어가 적용될 수도 있다는 생각을 했었다.

(베타 버전은 9월 상반기에 출시될 예정이며 이후에는 더 이상 기능을 추가 할 수 없다. 그것이 새로운 버전의 파이썬에 적용되기 위해선 난 변경 사항을 구현하고 평가 검증 셋을 돌리고 데드라인까지 메일로 토론을 해야 될것 같았다.)

# 자료 구조

내가 변경한 유일한 자료구조는 `PyDictKeysObject`뿐이다. 하지만, 이전보다 메모리 레이아웃이 훨씬 더 동적으로 변하였다.

```c
struct _dictkeysobject {
    Py_ssize_t dk_refcnt;
    Py_ssize_t dk_size;
    dict_lookup_func dk_lookup;
    Py_ssize_t dk_usable;
    Py_ssize_t dk_nentries;  /* How many entries are used. */
    char dk_indices[8];    /* Dynamically sized. 8 is the minimum. */
};

#define DK_SIZE(dk) ((dk)->dk_size)
#define DK_IXSIZE(dk) (DK_SIZE(dk) <= 0xff ? 1 : DK_SIZE(dk) <= 0xffff ? 2 : \
                       DK_SIZE(dk) <= 0xffffffff ? 4 : sizeof(Py_ssize_t))
#define DK_ENTRIES(dk) ((PyDictKeyEntry*)(&(dk)->dk_indices[DK_SIZE(dk) * \
                        DK_IXSIZE(dk)]))

dk_get_index(PyDictKeysObject *keys, Py_ssize_t i)
{
    Py_ssize_t s = DK_SIZE(keys);
    if (s <= 0xff) {
        return ((char*) &keys->dk_indices[0])[i];
    }
    else if (s <= 0xffff) {
        return ((PY_INT16_T*)&keys->dk_indices[0])[i];
    }
    else if (s <= 0xffffffff) {
        return ((PY_INT32_T*)&keys->dk_indices[0])[i];
    }
    else {
        return ((Py_ssize_t*)&keys->dk_indices[0])[i];
    }
}

dk_set_index(PyDictKeysObject *keys, Py_ssize_t i)
{
…
```

이 이전 해시테이블은 세 단어(해시, 키, 값) `PyDictKeyEntry`의 배열들이었다. 하지만 내가 시도했던 새로운 방법에서는 이 해시테이블 원소들을 정수값으로 변경했다. `char dk_index[8]`는 데이터 구조를 정의하는데 사용되지만 여기서는 최소 `dk_size`를 8로 설정할 뿐이다. 그것이 할당될때 그보다 더 큰 공간을 얻게될 것이다. 이 정수형은 실제로 128, 1글자 짜리까지의 `dk_size`를 허용 한다. 하지만 256부터는 `int16_t`가 되는데 이것은 내가 가능한 한 해시 테이블 크기를 줄이고자 하기 위해서였다.

더욱이 우리는 직접적으로 데이터 구조를 정의할 수 없었는데 왜냐하면 `dk_indices`의 크기가 동적이었기 때문이었다. 그러나, 나는 조심스럽게 `PyDictKeyEntry` 데이터 구조 배열 끝에 이 데이터 구조를 배치하였다. 이 배열의 크기는 `dk_size`가 아니지만 대신에 그것의 2/3를 가진다. (나는 이전 기사에서 이 아이디어를 소개했다. 이것은 이 해시테이블에 삽입될 수 있는 최대 원소 개수이다.) 새 원소들을 삽입할 때, 우리는 단순히 이 배열에 그것들을 작성하고 `dk_indices`에 인덱스를 저장한다. "`dk_nentries`"는 배열에서 찾은 원소의 개수를 나타낸다.

동일한 키가 기존에 없다고 가정하는 다음의 pseudo-code는 새로운 원소들이 삽입될 때의 코드의 모습을 보여준다. 

```c
// Search for the place of insertion inside dk_indices
pos = lookup(keys, key, hash);


// Write the new entry into the entries array

DK_ENTRIES(mp)[keys->dk_nentries].me_hash = hash;
DK_ENTRIES(mp)[keys->dk_nentries].me_key = key;
DK_ENTRIES(mp)[keys->dk_nentries].me_value = value;


// Save the index of that entry into dk_indices 
dk_set_index(keys, pos, keys->dk_nentries);


// Lastly, we increment the entry number we just finished using
mp->dk_nentries++;
```

# 원소 삭제

이 버전의 dict에서 원소들을 삭제하기 위해 우리는 `dk_indices` 안에 같은 위치에 "더미 원소" 라는 placeholder를 삽입해야만 한다. (1 바이트 엔트리 넘버를 통해 처리되는각 인덱스가 최대 256이 아닌 128까지 확장되는 이유는 더미 값으로 삽입되고 빈 공간으로 표현되는 음수 값들을 처리해주어야 하기 때문이다.)

우리의 dict값 들에서 원소들을 삭제하는 방법에는 두가지가 있다.

"compact dict" 아이디어가 Python 개발자 메일링 리스트에 처음 올라왔을 때는, 마지막 원소를 삭제된 원소가 이전에 위치했던 곳으로 옮김으로써 꽤 dict의 값들을 보존하는 것을 제안했었다. 이 방법을 사용하면 마지막 원소가 앞으로 이동하며, 이는 우리가 원소를 삭제할 때 우리가 "원소가 삽입된 순서를 보존하는" 속성을 잃어버림을 의미한다.

반면에 PyPy 커뮤니티와 내가 생각한 적절한 방법은 비워진 인덱스 위치에 단순히 NULL을 삽입하는 것이다.

```c
// Search for the position of the index of the element that was deleted inside dk_indices 
pos = lookup(keys, key, hash);

// Get the position of the element to be deleted inside the entry array 
index = dk_get_index(keys, pos);


// Delete the element
DK_ENTRIES(mp)[index].me_key = NULL;
DK_ENTRIES(mp)[index].me_value = NULL;


// Add a “dummy” inside dk_indices
dk_set_index(keys, pos, DUMMY);
```

이런식으로 처리하는 방법에선 많은 삽입 및 삭제가 수행될때 dict의 항목들이 dummy 항목들로 채워지게 된다. 이 때문에 압축을 해줘야 하는 다소 불편한 일이 생긴다. 그러나 내가 처음 제안한 방법에서 해시테이블은 반복적으로 삽입 및 삭제가 수행될 때 dummy 항목들로 채워지게 되고 나중엔 더 이상 탐색을 수행할 수 없게되는 상황을 초래한다. 두 방법 다 압축을 해야만 한다. 결국 가장 중요한 것은 삽입 순서를 보존하는 것이었다.

여하튼, dict 항목에서 마지막 원소를 삭제하고 `dk_nentries`를 감소시킬 때 `.popitem()` 는 보통 O(1)의 비용을 필요로 한다. 이 경우 또한, `dk_usable`로 알려져 있는 "삽입 가능한 원소의 개수"를 증가시키지 않는다. 이 말은 반복적으로 원소를 삽입 및 삭제할 때 나중에 해시테이블을 압축하고 재구성하는 것이 필요하게 될 것임을 의미한다.

# 공유 키 dict

이제 우리는 `공유 키 dict`라 알려진 문제를 다룰 수 있다. 

이것은 내가 처음에 프로젝트를 시작했을 때 생각한 것이다: 만약 해시 테이블에 dummy 원소를 삽입하지 않고 처음부터 dict 항목쪽이 NULL로 설정되어 있다면 `compact dict`를 구현하기 전에 처럼 이것을 단순히 또 하나의 dummy 원소로 간주할 수 있다.

맙소사, 내가 더 공부했어야 했나? 이 방법으론 공유 키에 처음 새 원소가 삽입될 때 `dict`에서 삽입 순서를 보존할 수가 없었다.

```c
>>> class A:
...     pass
...
>>> a = A()
>>> b = A()
>>> a.a = 1
>>> a.b = 2
>>> b.b = 1
>>> b.a = 2
>>> a.__dict__.items()
dict_items([('a', 1), ('b', 2)])
>>> b.__dict__.items()  # Even though the actual insertion order is b, a…
dict_items([('a', 2), ('b', 1)])
```
> 역주: Python 3.6 부터는 삽입 순서가 유지 된다!

이 문제를 해결하기 위해, 우리는 다음 세가지 방법을 고려해야 했다. 메일링 리스트에서 이 문제에 대해 토론을 하고 난 뒤, 우리는 Guido 또는 Guido가 지정한 핵심 개발자가 최종 결정을 내릴 때까지 어떤식으로 언어가 진행될 지 알길이 없었다.

0. 그냥 받아들이기
  현재 파이썬이 동작하는 언어 스펙에서 `dict`의 순서는 명확하지 않았다. 결과적으로 파이썬 작동방식이 현재는 삽입 순서(인스턴스 특성을 관리하는 `dict`는 제외)를 보존하더라도 언어의 실제 스펙 관점에서 보면 문제될 게 없었다.

  `compact dict`를 사용할 때, `공유 키 dict`와 `ma_values` 배열 크기는 `dk_keys` 크기의 2/3이 되어 훨씬 더 압축 된다.
  
  반면, 이 모든것의 안좋은점이라면 거의 모든 경우에 대해 삽입 순서를 보존해주는 추가적인 작업을 해야 했다. 언어 스펙을 확인해 볼 시간이 없는 프로그래머들에겐 이것이 실수로 이어질 수 있었다. 언어 스펙 때문에 실수 하는 이를 탓하는건 공정치 못하다. 예를 들어 Go의 단점을 피하기 위해 `map` 이 반복될때의 순서는 의도적으로 비결정적으로 남겨 둔다.(빠르게 생성된 의사 난수를 사용하여)

0. 삽입 순서가 잘못되었을 경우 `공유 키` 사용 중지

  만약 새 원소가 `공유 키`에 의해 유지된 순서와 다른 순서로 삽입이 시도 될 경우 언제든지 `공유 키` 사용을 중지 할 수 있다.
  
  이게 얼핏보면 가장 안전한 방법처럼 보이지만, `공유 키`를 얼마나 유지해야 하는지 알기란 사용될 리소스량을 예측하는것 처럼 쉽지 않은 일이다. 또한 거의 사용되지 않은 경로가 삽입된 순서와 다른 경우 `공유 키`가 해제 된다. 같은 크기의 `dict`를 계속 사용함에도 불구하고 메모리 사용량의 크기는 느리지만 계속 증가할 것이다. 이 방법을 적용하기엔 이런 문제들이 있었다.
  
  오랫동안 돌아가는 웹 어플리케이션과 같은 프로그램에서 메모리 사용량을 예측하기 어려운데다가 점점 증가까지 한다는 것은 결코 아무도 바라지 않을 일이다. 까다로운 프로그래머들은 이 방법을 꺼릴 것이다.
  
0. `공유 키 dict` 사용 중지
  
  `공유 키 dict`는 흥미로운 물건이다. 잘만 쓰면 정말 효율적이지만 `compact ordered dict`는 더 안정적이고 전반적인 경우에 더 효율적이다. 게다가 `공유 키 dict`를 지원하기 위해 `dict`의 구현은 훨씬 더 복잡해졌다. 나는 `공유 키 dict` 구현 부를 어느 정도까지 제거할 수 있는지 확인해 보았다. 총 4100여줄 가운데 500줄 남짓을 제거 할 수 있다. 나는 그것들을 지워버렸고 만약 좀만 더 리팩토링 한다면 더 지울 수도 있을 것이다.
  
  이것이 효율적임에도 내가 Sphinx를 사용한 파이썬 문서를 구축하고 /usr/bin/time을 통해 maxrss를 측정한 결과는 다음과 같다:
  
  - 공유 키: 176312k
  - compact + 공유 키: 158104k
  - compact: 166888k

  보다시피, 공유 키 사용을 중지하더라도 `compact dict`의 영향으로 메모리 사용량이 줄어든다. 
  
  또한 `공유 키`를 삭제하고 나머지 프로그램을 구동할 때 개별적이지만 좀 더 효율적인 특수 `dict`를 구현함으로써 `compact + 공유 키` 보다 더 높은 효율을 목표로 할 수 있다.
  
# OrderedDict

마지막으로 OrderedDict를 사용해 `compact dict`의 속도를 높이는 좀 더 자세한 방법을 추가하며 이 글을 마치겠다.

Python3은 Python 2.7에선 적용되지 않았던 `move_to_end(key, last=True)` 메서드를 포함한다. 이 키워드 인자들은 사용하기 꽤 까다롭다. 하지만 `move_to_end(key, last=False)`를 사용하면 원소를 줄의 처음으로 이동할 수 있다. (실제 기능은 제쳐두고, 나는 이 메서드 이름이 너무 형편없이 지어졌다고 생각한다. 차라리 `move_to_front(key)` 였어야 했다)

이 기능을 구현하기 위해 `dK_entries` 배열을 정적 크기 동적 배열이 아닌 정적 크기 deque로써 처리할 필요가 있을 것이다. 다시 말하면, 지금은 모든 dK_entries[0]부터 dK_entries[dk_nentries-1]까지를 사용 한다는 것이다. 그러나 이거 이외에도 맨 앞에 원소를 추가할 때 그것들은 dk_entries 뒤에 추가하고 삽입시에 맨 앞으로 옮기게 된다.
  
이 아이디어를 구체화하기 위해, 해시테이블의 스캐닝 및 크기 조절 기능을 사용할 수 있게끔 수정한 `dk_nentries`의 특이한 버전을 만들어야 했다.각 OrderedDict에 1 워드(8 바이트)를 추가함으로써 메모리 소모량을 절반으로 줄일 수 있었다.

정리하면, 아직도 `공유 키` 문제가 산더미처럼 쌓여 있다. 게다가 우리가 한번 `dict`의 삽입 순서를 보존하는게 가능해진 후로 OrderedDict를 사용할 기회는 결과적으로 더 줄어들며 이 아이디어를 구현할 동기가 약해졌다. 적어도 최소한 나는 이것이 Python 3.6에 들어갈 준비가 되어 있다고 보지 않는다. (누군가 하드캐리 해주지 않는 이상..)

---

으아 Python compact dict에 대한 내용을 찾다가 나온 첫번째 글이었는데 번역이 너무 힘들었다. 그냥 막판에 귀찮아서 의역 때렸는데 이게 더 나은것 같다. 아직 두번째 글이니까 내 자신에게 너무 엄격할 필요는 없어보인다. 
~~근데 원작자 말로는 도저히 못하겠다고 했는데 지금보니까 적용되었다. 어떻게든 잘 하셧나보네~~
