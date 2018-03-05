---
layout: "post"
title: "Bad except clauses order"
date: "2018-02-28 20:05"
category: [Python, Anti-pattern]
---

>**이 글은 [Python anti patterns : Bad except clauses order](https://docs.quantifiedcode.com/python-anti-patterns/correctness/bad_except_clauses_order.html) 의 내용을 번역한 것입니다.**

예외가 발생했을 때, Python은 발생한 예외 유형과 일치하는 첫번째 예외 처리 절을 찾을 것이다. 꼭 정확히 일치할 필요는 없고 예외 처리 절이 발생한 예외의 부모 클래스라면 일치하게 된다. 예를 들어 `ZeroDivisionError` 예외가 발생하고 첫번째 예외 처리 절이 `Exception`이라면 `ZeroDivisionError`가 `Exception` 클래스의 자식 클래스므로 `Exception` 절이 실행된다. 따라서 더 세부적인 클래스의 예외 처리 절은 항상 그것들의 부모 클래스의 예외 처리 절 보다 앞에 두어 가능한 한 더 세부적이고 유용하게끔 해야 한다.

# Anti-pattern

아래 코드는 결과적으로 `ZeroDivisionError`를 발생시키는 나눗셈을 수행한다. 코드에는 문제의 원인을 정확히 짚어주는 유용한 예외 처리 절을 포함하고 있는데 이 `ZeroDivisionError` 절은 절대 도달할 수 가 없다. `Exception` 절이 그 앞에 위치하고 있기 때문이다. Python이 예외를 다룰 때 순차적으로 각 예외 처리 절을 검사하게 되며 발생한 예외와 일치하는 처음 절을 실행하게 된다. 정확히 동일하게 일치할 필요는 없다. 발생한 예외가 예외 처리 절의 자식 클래스라면 Python은 나머지 절들은 모두 생략하고 그 절을 실행한다. 이 코드는 가능한 한 더욱 정확하게 예외를 처리하고 식별하는 예외 처리 절의 목적을 무색하게 만든다.

```python
try:
    5 / 0
except Exception as e:
    print("Exception")
# unreachable code!
except ZeroDivisionError as e:
    print("ZeroDivisionError")
```

# Best practice

## 자식 클래스의 예외 처리 절은 부모 클래스의 절보다 앞에 위치할 것

아래 수정된 코드는 `ZeroDivisionError` 예외 처리 절을 `Exception` 절 앞에 두었다. 이제 에러가 발생하면 좀 더 명확하고 적절한 `ZeroDivisionError` 예외 처리 절이 실행될 것 이다.

```python
try:
    5 / 0
except ZeroDivisionError as e:
    print("ZeroDivisionError")
except Exception as e:
    print("Exception")
```