---
layout: "post"
title: "PEP 487 -- Simpler customisation of class creation"
date: "2018-02-27 07:05"
category: [python, pep]
---

이 글은 [PEP 487 -- Simpler customisation of class creation](https://www.python.org/dev/peps/pep-0487/) 의 내용을 번역/정리한 글이다.

<table>
<colgroup><col>
<col>
</colgroup><tbody valign="top">
<tr><th>PEP:</th><td>487</td>
</tr>
<tr><th>Title:</th><td>Simpler customisation of class creation</td>
</tr>
<tr><th>Author:</th><td>Martin Teichmann &lt;lkb.teichmann at gmail.com&gt;,</td>
</tr>
<tr><th>Status:</th><td>Final</td>
</tr>
<tr><th>Type:</th><td>Standards Track</td>
</tr>
<tr><th>Created:</th><td>27-Feb-2015</td>
</tr>
<tr><th>Python-Version:</th><td>3.6</td>
</tr>
<tr><th>Post-History:</th><td>27-Feb-2015, 5-Feb-2016, 24-Jun-2016, 2-Jul-2016, 13-Jul-2016</td>
</tr>
<tr><th>Replaces:</th><td><a href="https://www.python.org/dev/peps/pep-0422">422</a></td>
</tr>
<tr><th>Resolution:</th><td><a href="https://mail.python.org/pipermail/python-dev/2016-July/145629.html">https://mail.python.org/pipermail/python-dev/2016-July/145629.html</a></td>
</tr>
</tbody>
</table>

# 개요

현재는 클래스 생성을 커스터마이징 하기 위해 커스텀 메타클레스의 사용이 필요하다. 이 커스텀 메타클래스는 커스터마이징된 클래스의 전체 생명주기동안 지속되며 잠재적으로 비논리적 메타클래스 충돌을 일으킨다.

대신에 이 PEP는 클래스 정의부의 새로운 \__init_subclass__ 훅과 속성 초기화에 훅을 걸어 더 넓은 범위의 커스터마이징 시나리오들을 지원하게끔 제안하는 바이다.

새로운 메커니즘은 커스텀 메타클래스를 구현하는것보다는 이해하고 사용하기가 더 쉬워야 한다. 따라서 적절한 설명과 함께 파이썬 메타클래스 시스템의 진면목을 살펴보자.

# 배경
메타클래스는 클래스 생성을 커스터마이징하는 강력한 도구이긴 하지만 자동으로 메타클래스들을 결합하는 방법이 없다는 문제가 있다. 만약 누군가가 어떤 클래스에 두 메타클래스를 사용하고 싶을 경우, 일반적으로 일일이 두 메타클래스를 결합하는 새로운 메타클래스를 만들어야 한다.

이러한 요구는 종종 사용자에게 충격으로 다가온다: 각각 다른 라이브러리로부터 두개의 기반 클래스들을 상속하는 것이 갑자기 라이브러리에 대한 세부 정보에 일반적으로 관심없는 결합된 메타클래스들을 필요로 한다. 한 라이브러리가 이전에 없던 메타클래스를 사용하기 시작할 경우 이것은 더욱 심해진다. 라이브러리가 아무 문제없이 잘 작동하다가 다른 라이브러리들의 클래스들을 결합하는 ㅅㅂ 뭐라는겨.. 
~~이 문서는 보류됨~~
