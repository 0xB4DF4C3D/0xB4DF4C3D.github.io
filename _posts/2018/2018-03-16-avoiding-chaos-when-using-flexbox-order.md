---
layout: "post"
title: "Avoiding chaos when using Flexbox 'order'"
date: "2018-03-16 12:58"
category: Web
---

> 원문은 [여기](https://codepen.io/AmeliaBR/post/avoiding-chaos-when-using-flexbox-order) 입니다. 저는 그저 번역만 했어요.

> Original is [here](https://codepen.io/AmeliaBR/post/avoiding-chaos-when-using-flexbox-order). I just only translated it.

내가 처음 CSS flex 속성을 가지고 놀던 때부터 계속 만들려고 했었던 Flexbox 데모를 최든 한 트위터 때문에 마침내 만들게 되었다.

Roma Komarov는 CSS Flexbox와 CSS만으로 테이블 정렬하는 효과를 만들기 위해 `order` 속성을 [사용하는 방법](http://kizu.ru/en/blog/variable-order/)에 대해 글을 올렸었다.

몇몇 다른 CSS 전문가들은 곧 이것은 실전에서 지양되며 심지어는 명세와 직접적으로 반대된다는 것이라 [경고했다](https://twitter.com/jensimmons/status/964360059923742720).

Roma는 비판을 인정하고 블로그 글에 경고를 덧붙였다: 이건 단지 실험입니다, 실전에서는 쓰지 마세요! 하지만 트위터에서 몇몇 사람들이 계속 질문을 올리고 또 올렸다:

> 그럼, `order`는 뭘 위한 것인가요? 만약 그것의 사용법이 "원본과 다른 시각적 순서" 라면, 그것이 다른 매체에서도 적용되지 못할 이유가 있나요?
> 
> \- Seth A. Roby(@TALIama) February 16, 2018

시작으로, 먼저 어떻게 `order`를 사용하지 않기로 되어있는지와 그 이유에 대해 검토해보자.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9d/Color-coded_bookcase_%283254322054%29.jpg/526px-Color-coded_bookcase_%283254322054%29.jpg)
> _시각적 순서를 강요하는 것이 항상 그 본질적인 의미를 반영하진 않는다._

Flexbox 명세는 경고한다:
> 사용자는 반드시 `order`를 논리적이 아닌 시각적으로 컨텐츠를 재배열해야만 합니다. 논리적 재배열을 수행하기 위해 `order`를 사용한 스타일시트는 규칙에 맞지 않습니다.

웹 프로그래머들에게 지시하는 CSS 명세는 그리 흔하지 않다. 그 "반드시"라는 것들의 대부분은 브라우저 개발자들 것이다. 그러므로 이제 이 `order`에 대한 지시가 심각하는 걸 알았다. 키보드 사용자나 스크린 리더 사용자는 DOM의 요소의 일관성 있는 순서에 의존하고 화면을 볼 수 있는 키보드 또는 보조 장치 사용자는 그들이 보고 있는 요소와 논리적 순서가 일치하도록 지시하는게 필요하다.

그렇다. floated layout 열 또한 원본 순서를 사용하기 위해 시각적 순서와 잘 맞지 않음에도 허용되지만(가끔은 어쩔 수 없이 써야하는) 그게 바로 우리가 layout에 float를 사용하는것으로부터 도망치려고 노력하는 이유이다! Flexbox & CSS 격자는 합리적인 마크업 구조를 더 쉽게 사용하려고 만들어진 것이지.&mdash;마크업 구조를 더 쉽게 무시하려는게 아니다.

만약 DOM을 재배열 하고 싶다면, layout에만 적용되는 CSS 메서드를 사용하지 말고 DOM 메서드를 사용해서 요소를 이동시켜라.

그럼 `order`는 왜 존재할까? 만약 컨텐츠를 재배열하기 위해 CSS를 사용하지 않도록 되어 있다면, 왜 CSS에 `order` 속성이 있는걸까?

왜냐하면 가끔은 layout의 논리적 순서가 Flexbox의 좌->우, 우->좌, 상->하, 하->상 과 같은 기본 flow option으로는 만들어지지 않기 때문이다. `order` 속성은 이런 경우들을 위한 것이다: 현재 사용하고 있는 디자인을 위해 시각적 순서를 논리적 순서와 일치하도록 만드는 것.

또는 Flexbox 명세 편집자 Elika Etemad(AKA fantasai)의 좀 더 우아한 말로 나타내면 이렇다:
> 시각적 인식은 좌표계 순서를 따르지 않습니다. 그것은 게슈탈트 원칙같은 것들에 적용되고 동시성을 가지고 있습니다. 따라서 우리는 원본을 읽는 순서에 맞춰 배열하고 시각적 layout이 더 자유롭게 요소들을 이동시키도록 허용합니다.
>
> fantasi (@fantasai) February 16, 2018

