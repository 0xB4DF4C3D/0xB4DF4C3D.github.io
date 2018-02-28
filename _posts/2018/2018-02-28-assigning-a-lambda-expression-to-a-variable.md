---
layout: "post"
title: "Assigning a <i>lambda</i> expression to a variable"
date: "2018-02-28 07:39"
category: [Python, Anti-pattern]
---

>**이 글은 [Python anti patterns : Assigning a <i>lambda</i> expression to a variable](https://docs.quantifiedcode.com/python-anti-patterns/correctness/assigning_a_lambda_to_a_variable.html) 의 내용을 정리한 것입니다.**

`def` 대신 `lambda` 표현식을 쓰면 좋은 유일한 장점은 `lambda`가 더 큰 표현식 안에 익명함수로 삽입될 수 있다는 것이다. 만약 `lambda`에다가 이름을 붙이고 싶다면 차라리 그냥 `def`로 정의하는게 더 나은 선택이다.

PEP 8 스타일 가이드에 따르면:

**Good:**
```python
def f(x): return 2*x
```

**Bad:**
```python
f = lambda x: 2*x
```

첫번째 형태는 결과 함수 객체의 이름이 `generic <lambda>`가 아닌 `f`임을 의미한다. 이것은 일반적으로 역추적을 하거나 문자열로 표현하기에 더 유용하다. 두번째처럼 대입을 사용하면 `lambda` 표현식이 명시적인 def문 대신 제공할 수 있던 유일한 이점을 없애는 짓이다.

# Anti-pattern

다음 코드는 입력의 두배를 반환하는 `lambda` 함수를 변수에 할당한다. 이것은 기능적으로 `def`와 동일하다.
```python
f = lambda x: 2*x
```

# Best practice

## 이름있는 표현식에는 `def`를 사용할 것

위의 `lambda` 표현식을 이름있는 `def` 표현식으로 리팩토링 할 것.
```python
def f(x):
  return 2*x
```