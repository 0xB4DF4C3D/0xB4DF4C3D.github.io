---
layout: "post"
title: "Accessing a protected member from outside the class"
date: "2018-02-28 07:23"
category: [Python, Anti-pattern]
---

>** 원문 : https://docs.quantifiedcode.com/python-anti-patterns/correctness/accessing_a_protected_member_from_outside_the_class.html **

바깥에서 클래스의 protect 멤버(_로 시작하는 멤버)에 접근하는 것은 일반적으로 에러가 나길 기대할 것이다. 왜냐하면 이 멤버가 바깥에 노출되는 것은 그 클래스를 만든 사람의 의도와 맞지 않기 때문이다.

# Anti-pattern

```python
class Rectangle(object):
    def __init__(self, width, height):
        self._width = width
        self._height = height

r = Rectangle(5, 6)
# direct access of protected member
print("Width: {:d}".format(r._width))
```

# Best practice

만약 바깥에서 꼭 protect 멤버에 접근해야 되는게 확실하다면 다음을 따라라:
  - 바깥에서 멤버에 접근하는 것이 의도치않은 어떠한 부작용도 일으키지 않게 확실히 할 것.
  - 그것이 클래스의 public 인터페이스의 일부가 되도록 리팩토링 할 것.