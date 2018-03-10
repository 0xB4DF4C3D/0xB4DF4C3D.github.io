---
layout: "post"
title: "A BEM syntax with UX in mind"
date: "2018-03-10 16:43"
category: Web
---

> 원문은 [여기](http://simurai.com/blog/2013/10/24/BEM-syntax-with-ux-in-mind) 입니다. 저는 그저 번역만 했어요.

> Original is [here](http://simurai.com/blog/2013/10/24/BEM-syntax-with-ux-in-mind). I just only translated it.

언젠가부터, MontageJS 프레임워크를 개선하기 위해 애를 쓰는 동안, 우리가 사용하기 시작해야 할 CSS 명명법은 무엇일까 하는 의문이 생겼다. [긴 논의](https://github.com/montagejs/montage/issues/795) 끝에 우리는 [BEM 방법론](http://bem.info/method/definitions/)을 사용하는 것으로 합의를 보았으나, 문법을 조금 변경했다. 이 글을 짧게 하기 위해, BEM을 사용하는게 왜 좋은 생각인지는 자세하게 다루지 않을테지만, 대신 왜 우리가 다른 문법을 택했는지에 대해 설명할 것이다.

여기 몇가지 예제들이다:
```css
.digit-Progress         /* org-Component */
.digit-Progress-bar     /* org-Component-childElement*/
.digit-Progress--small  /* org-Component--variation */
```

> 유의 : `org-`(digit-) 접두사는 네임스페이스로 쓰였으며 따라서 다른 패키지, 라이브러리, 프레임워크와 충돌하지 않을 것이다.

이제 이런 문법을 택한 이유를 보자.

# 하이픈 Hyphens (-)

우리가 언더스코어(`_`) 대신 하이픈(`-`)을 사용한 주된 이유는 더블 클릭으로 텍스트를 선택할때 그것들의 작동방식이 서로 다르다는 사실과 관계가 있다. 한번 직접 해보아라:
```css
component__element /* underscores */
component-element  /* hyphen */
```

![](http://simurai.com/img/posts/BEM-1.gif)

언더스코어를 사용할땐 선택한 부분의 앞뒤 모두를 선택하는 것을 보아라. 하지만 하이픈을 사용하면 더블 클릭한 부분만 선택할 수 있다. 덕분에 원하는 부분만 더 빠르게 편집할수 있게 된다.

![](http://simurai.com/img/posts/BEM-2.gif)

# 낙타표기법 camelCase
이제, 구성 요소나 자식 요소가 여러 단어로 구성되어 있다면 어떨까? `component_name-element_name`같은 언더스코어를 사용할 수도 있겠다. 이 방법은 여전히 더블 클릭이 가능하지만 무엇이 서로 속해있는지 보기 어렵기 때문에 가독성면에서 불리하다. 각 부분을 시각적으로 그룹화하기 위해 camelCase를 사용하는게 더 낫다: `componentName-elementName`

# 메인 요소

좋아, 이제 거의 다 온것 같다. 마지막 규칙으로, "메인" 요소를 위해 우리는 `PascalCase`을 사용한다. 이렇게 하는 이유는 **강조** 성을 더하고 메인 요소 자식 요소와 구별하기 더 쉽게 만들기 때문이다. 또한 네임스페이스를 사용할 때 메인 요소는 두번째 위치로 이동하는데, 그럼에도 불구하고 이를 눈에띄게 만들어서 더 중요하게 만들어준다: `org-Component-childElement`

# -변형 variation

우리는 변형을 위해 일반적으로 사용되는 더블 하이픈(`--`)을 그대로 쓰기로 했다. `digit-Progress--small`. 이 방법은 변형을 시각적으로 더 떨어져 있게 하며 기본 요소들 보다 뭔가 "다른" 느낌으로 보이게 만들어주기 때문에 합리적이다.

---

대충 이렇게 끝이다. 명명법에 대한 더 자세한 내용을 알려면, 같은 문법을 사용하기 시작했고 문서로 정말 잘 정리가 된 [SUIT Framework](https://github.com/suitcss/suit/blob/master/doc/naming-conventions.md)를 보아라.

끝으로, 어떤 BEM을 선택하든 개인적인 취향에 달려있지만, 유용성과 가독성을 발전시켜 훌륭한 UX에 대해 생각해보는것도 나쁘지 않다.