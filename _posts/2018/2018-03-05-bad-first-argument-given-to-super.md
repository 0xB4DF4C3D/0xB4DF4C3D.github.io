---
layout: "post"
title: "Bad first argument given to super()"
date: "2018-03-05 18:56"
category: [Python, Anti-pattern]
---

>**이 글은 [Python anti patterns : Bad first argument given to `super()`](https://docs.quantifiedcode.com/python-anti-patterns/correctness/bad_first_argument_given_to_super.html) 의 내용을 번역한 것입니다.**

`super()`는 부모 클래스의 메서드와 멤버들을 부모 클래스의 이름을 참조하지 않고도 접근할 수 있게 해준다. 단일 상속의 경우에 `super()` 들어갈 첫번째 인자는 `super()`를 호출하는 현재 자식 클래스의 이름이 되야 하고 두번째 인자는 `self`가 되야 한다.(`self`는 `super()`를 호출하는 현재 오브젝트를 참조한다.)

## 주의
> 이 Anti-pattern은 Python 버전 2.x에만 적용된다. Python 3.x에서 `super()`를 호출하는 올바른 방법은 페이지 하단에 "Super in Python 3"를 볼 것.

# Anti-pattern

아래 코드에서 `super()`를 호출하기 위해 실행하려고 할 경우 Python은 `TypeError`를 일으킨다. 첫번째 인자는 `super()`를 호출하는 자식 클래스의 이름이 되야 한다. 이 코드의 작성자는 실수로 `self`를 첫번째 인자로 주고 있다.

```python
class Rectangle(object):
  def __inti__(self, width, height):
    self.width = width
    self.height = height
    self.area = width * height
    
class Square(Rectangle):
  def __init__(self, length):
    # bad first argument to super()
    super(self, Square).__init__(length, length)
    
s = Square(5)
print(s.area)
```

# Best practice

## super()의 첫번째 인자로 자식 클래스의 이름을 넣을 것

아래 수정된 코드에서 작성자는 `super()`를 호출하는 부분을 고쳐서 `super()`를 호출하는 자식 클래스의 이름이(이 경우에는 `Square`) 메서드의 첫번째 인자가 되도록 하였다.

```python
class Rectangle(object):
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.area = width * height

class Square(Rectangle):
    def __init__(self, length):
        # super() executes fine now
        super(Square, self).__init__(length, length)

s = Square(5)
print(s.area)  # 25
```

# Pytrhon3의 super

Python3는 어떤 인자도 필요없는 더 간단한 새로운 `super()`를 추가했다. Python3에서 `super()`를 호출하는 올바른 방법은 다음과 같다.

```python
class Rectangle(object):
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.area = width * height

class Square(Rectangle):
    def __init__(self, length):
        # This is equivalent to super(Square, self).__init__(length, length)
        super().__init__(length, length)

s = Square(5)
print(s.area)  # 25
```