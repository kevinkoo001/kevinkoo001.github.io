---
author: kevinkoo001@gmail.com
comments: true
date: 2013-01-29 04:02:10+00:00
layout: post
link: http://dandylife.net/blog/archives/67
slug: technical-document-translation-helper
title: Technical Document Translation Helper
wordpress_id: 67
categories:
- Miscellaneous Stuff
tags:
- Tip
- Translation
- 번역
- 팁
---

 요즘 국내에서도 좋은 기술문서에 대한 번역이 활발해 보인다. 보이지 않는 곳에서 그들의 노고에 우선 감사한다. 기술 서적을 번역하는 분마다 개별 노하우가 있겠지만, 몇 권의 기술서를 번역하면서 느낀 점과 필자도 초반에 많이 실수했던 내용을 아래와 같이 요약했으니 향후 미래의 기술 서적 번역하는 이가 참고할 수 있도록 간략하게나마 Tip을 정리해 봤다.








 아래 내용이 항상 적용되는 법칙은 아니지만, 대체적으로 기술 서적에서 많이 발견할 수 있는, 소위 번역투로 조금만 신경쓰면 훨씬 읽기 좋은 글로 작성할 수 있다. 번역에 정답이 존재하지 않지만 읽기 편한 번역과 그렇지 않은 경우는 알 수 있다. 모쪼록 번역을 하는 사람과 번역서를 보는 사람 모두에게 도움이 되었으면 한다.







**0. 일반 원칙**




- 간결함이 최고다.




<blockquote>이러한/그러한 → 이런/그런
나올 수 있다 → 나온다
성능이 얼마나 좋은가의 문제 → 성능 문제
전송 중에 일어나는 것이 아니라 → 전송 과정이 아니라</blockquote>




- 때로는 상세히 풀어 설명하는 게 이해하기 쉽다.




<blockquote>works less well → 제대로 작동하는 경우가 드물다
the weakest-link property → 가장 약한 고리만큼 취약하다는 특성</blockquote>




 - 때로는 부정형보다 긍정형이 낫다.




<blockquote>얘기할 수 있는 것은 아니다 → 이야기하기 어렵다
실패한다(fail) → 성공하기 어렵다</blockquote>




**1. 수동형보다 능동형**




- 국어는 사람이 주어로 사용하면 읽기 편하고 자연스러우며 문장도 매끄럽다.


- '~에 의해'라고 명시적으로 나타낼 필요가 있을 때는 수동형을 써야 하지만 의미상 필요한 경우로 한정한다.
- ~된다. → ~한다,  ~어진다. → ~한다.


<blockquote>악성코드는 특별한 목적을 염두에 두고 만들어진다
→ 악성코드는 특별한 목적을 염두에 두고 제작한다.
디스어셈블러로 역분석된 것이다
→ 디스어셈블러로 역분석한 코드(2.~것)다.
코드 조각 안에서 보여지는
→ 코드에 보이는</blockquote>


**2. ~것/~것이다. **


- 동명사, 분사 구문 번역에 자주 나온다.


- '~것' 대신에 '~하는 점', '~하는 [명사]', '~음/함', '사실', '형태' 등 문맥에 맞게 다양한 형태로 변환할 수 있다.


<blockquote>프로그램을 개발하는 것에 대해
→ 프로그램 개발에 대해
결함을 찾을 수 있는 기회가 될 수 있다는 것을 의미한다.
→ 결함을 찾을 수 있는 기회가 될 수 있음을 의미한다.
신용카드 번호만을 사용하는 것보다
→ 신용카드 번호를 사용하는 방식보다
있다고 말하는 것이다
→ 있음을 뜻한다.
That is what authorizes the transaction
→ 서명을 해야 특정 거래가 승인된다.</blockquote>


- 때로는 문장 마지막에서 청유형으로 맺는 형태가 깔끔하다.


<blockquote>어플리케이션을 변경하는 방법을 알아볼 것이다.
→ 어플리케이션을 변경하는 방법을 알아보자.</blockquote>


**3. ~에 대해/ ~에 관해**


- about을 번역할 때 자주 나오는 표현인데 그냥 목적어로 두면 간결하다.




<blockquote>모바일 어플리케이션을 정확히 분석하는 방법에 대해 알아보자.
→ 모바일 어플리케이션을 정확히 분석하는 법을 알아보자.
주로 이러이러한 분야에 대해 다룬다
→ 주로 이 분야를 다룬다
위협 모델에 대해서
→ 위협모델은
시스템이 안전한지에 대한 알려진 테스트 방법 (known way of testing whether a system is secure)
→ 시스템 안전 여부를 판단할 알려진 테스트 방법</blockquote>


**4. ~하기 위해/ ~를 위한**


- for 또는 in order to 또는 목적을 의미하는 to 부정사에 주로 나타나는 번역투이다.




 - 위 번역 형태가 틀리지 않지만 ~에, ~할 목적으로, ~하려, ~해서 등으로 변환해서
여러 어구를 활용하거나 문장 순서를 바꾸는 편이 좋다.




<blockquote>공격자는 그림을 복제하기 위해 컴퓨터 화면을 촬영할 수 있고, 음악을 복제하기 위해 마이크를 통해 녹음할 수 있다.  → 공격자는 컴퓨터 화면을 촬영해 그림을 복제할 수 있고, 마이크로 녹음해서 음악을 복제할 수도 있다.</blockquote>




**5. 복수형**




- 국어는 복수형을 명시해야 할 명백한 이유가 없는 한 단수형으로 표현하는 게 낫다.


<blockquote>어플리케이션에 의해 다른 곳에서 참조하는 정적 문자열 정의들이다
→ 어플리케이션이 (1. 능동형) 다른 곳에서 참조하는 정적 문자열 정의다.
파일에서 정보를 추출하기 위해 사용되는 도구들
→ 파일에서 정보 추출에(4. 목적어투변화) 사용하는(1. 능동형) [여러] 도구
대다수 시스템들은(most systems)
→ 대다수 시스템은</blockquote>


- 많은 뒤에 복수형이 오지 않는다.






<blockquote>많은 사람들이 → 많은 사람이</blockquote>




**6. 용어통일**




- 전문 기술서를 번역하다 보면 아직 표준화되어 있지 않은 용어가 많다.




 - 이럴 경우 기본적으로 단어를 통일하고 TTA의 용어집을 활용한다.




<blockquote>Shell: 셸(O) VS 쉘(X)</blockquote>




 - 전문용어를 영문 그대로 일반적으로 사용할 경우 그냥 한글 발음대로 표기하고 도구와 같은 고유명사는 영문으로 둔다.




<blockquote>Buffer Overflow: 버퍼 오버플로우(0) VS 버퍼 초과(X)IDA Pro → IDA Pro</blockquote>




 - 여러 명이 함께 작업한다면 Google Docs 등을 이용해 용어집을 필히 공유해




    한 용어를 여러 개로 번역하지 않도록 주의한다.







**7. 주어**




- '우리는', '당신은', '여러분은' 과 같은 주어는 꼭 필요한 경우를 제외하고 제거하는 편이 좋다.




 - 필요한 경우 '독자는'이라고 쓰는 것도 한 방법이다. (매번 쓰면 별로다)







**8. 대명사**




- 이것, 저것, 그것은 직역체의 느낌으로 굳이 넣지 않아도 의미가 통하는 간결한 문장이 될 수 있다.




 - 문장의 흐름상 앞 문장을 받아 넣어야 할 경우 앞문장을 요약하는 형태로 작성하면 더욱 매끄럽다.







**9. '~를 가지고'**




- 영어식 표현으로 국어 형태로 자연스러운 느낌으로 변환한다.




<blockquote>여기서 배운 것을 가지고   → 여기서 배운 내용(2. 것)을 활용해</blockquote>




**10. '~의'**




- 불필요한 '~의'는 생략하도록 한다.




<blockquote>사용자의 관점   → 사용자 관점그림, 영화, 음악 등의 디지털 매체의   → 그림, 영화, 음악 등 디지털 매체의</blockquote>




 - 외부에서의 일반적인 공격들에 → 외부에서 가하는(1. 능동) 일반적인 공격(5. 복수형)에







**11. '및'**




- 일본식 표현으로 와/과로 사용한다.




<blockquote>실용적인 암호 시스템의 설계 및 구현 → 실용적인 암호 시스템의 설계와 구현</blockquote>




**12. 제목은 가능하면 명사형으로**




<blockquote>암호학은 매우 어렵다 → 어려운 암호학</blockquote>



