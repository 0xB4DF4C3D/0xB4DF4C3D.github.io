---
layout: "post"
title: "Assigning to built-in function"
date: "2018-02-28 19:47"
category: [Python, Anti-pattern]
---

>**이 글은 [Python anti patterns : Assigning to built-in function](https://docs.quantifiedcode.com/python-anti-patterns/correctness/assigning_to_builtin.html) 의 내용을 번역한 것입니다.**

Python은 인터프리터에서 항상 접근 가능한 여러 기본 내장 함수들을 가지고 있다. 만약 특별한 사유가 없는 한, 이 함수들을 재정의하거나 같은 이름의 변수에 값을 할당하지 말아야 한다. 내장 함수를 재정의 하는 것은 의도치 않는 부작용을 만들어내거나 런타임 에러를 발생 시킬 수 있다. Python 개발자들은 보통 내장 함수들을 **있는 그대로** 사용한다. 만약 그것들의 작동방식이 변경될 경우 실제 에러를 역추적하기가 더욱 까다로워질 수 있게 된다.

# Anti-pattern

아래 코드에서, `list` 내장 함수가 재정의 되었다. 이것은 `list`를 사용해 변수를 list로 정의하는것을 더 이상 불가능 하게 만든다. 이것이 바로 문제가 도대체 무엇인지 알기 쉬운 정확한 예이다. 그러나 `list`에 할당하고 `cars`에 할당하는 사이에 수백여줄이 있을 경우엔 문제를 식별하기가 더 어려워질 수 있다.

```python
# Overwriting built-in 'list' by assigning values to a variable called 'list'
list = [1, 2, 3]
# Defining a list 'cars', will now raise an error
cars = list()
# Error: TypeError: 'list' object is not callable
```

# Best practice

내장 함수랑 같은 이름을 가진 변수를 사용할 진짜 특별한 이유가 없는 한, 내장 함수 이름과 변수 이름은 서로 간섭하지 않도록 하는게 추천된다.

```python
# Numbers used as variable name instead of 'list'
numbers = [1, 2, 3]
# Defining 'cars' as list, will work just fine
cars = list()
```