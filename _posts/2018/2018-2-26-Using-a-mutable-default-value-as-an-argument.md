---
comments: true
layout: post
title: Using a mutable default value as an argument
category: [python, anti-pattern]
---

이 글은 [Python anti patterns : mutable default value as argument](https://docs.quantifiedcode.com/python-anti-patterns/correctness/mutable_default_value_as_argument.html)  
의 내용을 정리한 것입니다.

# 파라미터의 기본값으로 mutable한 값 사용하기

쉽게 예상하지 못하는 버그 중 하나로써 함수 파라미터의 기본값으로 mutable한 list 또는 dictionary를 사용할 경우 예상치 못한 결과를 초래할 수 있다. 보통 함수가 매번 호출될 때 마다 새로운 list 또는 directory가 생성되기를 기대할 것 이다. 하지만 파이썬은 그렇지 않다. 처음 함수가 호출될 때 파이썬은 해당 list 또는 dictionary에 대한 영속 개체를 생성한다. 이후 매번 함수가 호출될 때 마다 파이썬은 처음 함수가 호출될 때 만들어졌던 같은 개체를 사용한다.

# Anti pattern

어떤 프로그래머가 아래 코드에서 `append` 함수를 두번째 인자 없이 호출하면 새로운 list를 반환된다고 가정하고 다음과 같이 작성했다고 하자. 실제로 그런일은 일어나지 않는다. 처음에 한번 함수가 호출되면 파이썬은 영속적인 list를 생성한 뒤 그 이후에 모든 `append`의 호출은 기존에 있는 리스트에 값을 추가할 뿐이다.

```python
def append(number, number_list=[]):
    number_list.append(number)
    print(number_list)
    return number_list

append(5) # expecting: [5], actual: [5]
append(7) # expecting: [7], actual: [5, 7]
append(2) # expecting: [2], actual: [5, 7, 2]
```

# Best practice

## Sentinel 값을 사용하여 빈 list 또는 dictionary 나타내기

만약 위의 `append` 함수를 구현한 프로그래머처럼 함수가 매번 호출 될 때마다 새로운 빈 리스트를 반환하게끔 하고 싶다면 [Sentinel 값]({% link _posts/2018/2018-02-26-Sentinel_value.md %})
를 사용하여 이러한 유스케이스를 표현하고 함수 정의부를 변경하여 이러한 시나리오를 만족할 수 있다. 함수는 sentinel 값을 받으면 새로운 리스트를 반환해야 된다는 것을 알고 있다.

```python
# the keyword None is the sentinel value representing empty list
def append(number, number_list=None):
    if number_list is None:
        number_list = []
    number_list.append(number)
    print(number_list)
    return number_list

append(5) # expecting: [5], actual: [5]
append(7) # expecting: [7], actual: [7]
append(2) # expecting: [2], actual: [2]
```

# 참고
- PyLint - W0102, dangerous-default-value
- [Stack Overflow - Hidden Features of Python](http://stackoverflow.com/questions/101268/hidden-features-of-python#113198)
