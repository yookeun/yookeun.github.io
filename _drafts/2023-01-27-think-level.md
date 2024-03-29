---
layout: post
title:  "하수와 고수의 차이"
date:   2023-01-27
categories: think
---

프로젝트 오픈이 2주도 채 남지 않은 어느날, 갑자기 고객의 요구사항이 변경된다. 

> 설계자 : 아니, 지금 와서 그렇게 요구사항이 변경되면 테이블 구조도 변경되고, 결국 관련된 소스도 모두 변경되어야 합니다!
>
> PM : 어쩌겠어요. 고객이  이 부분이 이번 오픈전에 반드시 반영되어야 한다고 하네요.  저도 미치겠네요 ㅠㅠ 살려주세요!

고객의 요구사항을 반영하면 기존 테이블 구조가 변경되어야 정규화 및  중복 로직등 피할 수 있다.  고민끝에 개발자들에게 상황을 이야기한다. 

> 백엔드 : 아니 장난해요? 지금  그 API 로직 테스트도 다 끝났는데, 테이블 구조 변경되면 관련된 로직도 수정되어야 한다고요!
>
> 프론트: 아니 백엔드 변경되면 우리도 수정해야 하잖아요? 오픈 일정 변경가능해요?
>
> PM: 물론 불가능하죠 ㅠㅠ. 일단 기존꺼는 그대로 두고 새로 요구사항 반영하는게 어떨까요? 고객이 다시 변심할 수도 있잖어요. 
>
> 백엔드/프론트 : 차라리 그게 낫겠어요. 기존꺼 다시 수정하느니, 신규로 해당 요구사항 반영하는게 더 빠를 것 같아요. 
>
> 설계자: 음.. 그렇게 되면 테이블 정규화가 안되는데, 중복되는 컬럼도 생기고... 

고민끝에 기존 소스를 다시 변경하느니 조금 중복되는 부분이 있더라도 신규로 구현하는 것이 현 상황에서 최선이라고 여기고 결론을 내었다.

그렇게 몇 일의 야근끝에 악몽같은 오픈주기가 지났다. 1년이 지나서 해당 프로젝트는 고도화가 결정되었고, 새로운 인력들이 투입되었다.

새로운 개발자가 업무 분석을 위해서 ERD를 열어보았다. 

> 새 개발자: 와..미쳤다. 중복되는 컬럼의 테이블이 있네, 정규화도 안했네. 엉망이다. API도 다르지만, 비슷한게 있어, 아니 OCP(Open-Closed Principle)도 몰라? 

역시 새로운 PM이 묻는다

> 새 PM : 기존 소스 분석 잘되어가요?
>
> 새 개발자 : 엉망이에요. 테이블 구조도 엉망이고, 소스도 그렇고, 이거 다 뜯어고치던지, 시간없다면 그냥 뭐 이런식으로 할 수 밖에요..

어떤가? 익숙한 상황이 아닌가? 새 개발자의 말도 틀린 것은 아니다. 그런데 이 전 개발자들도 비난받을 만큼 크게 잘못한 것일까?

몇년 전에 필자가 개발팀장으로 있을 때, 좀 엉망인 프로젝트를 인수 받았다. 위와 같은 상황이었는데 술자리에서 그 프로젝트에 투입된 안면도 없는 개발자들의 실력을 안주발로 삼고 있었다. 그러자 같이 술잔을 기울이던 그 팀원이 나에게 말했다.

>  근데 팀장님,  머 그럴 수 밖에 없는 상황이 있었지 않았을까요?

머리를 한대 맞은 느낌이었다. 어떤 상황인지 모른채 내가 너무 속단했다는 생각이 들었다. 잦은 요구사항의 변경, 매일 반복되는 야근과 철야의 연속속에서 나는 그렇게 잘날 수 있었을까?  나는 몇년전에 그렇게 잘난 사람이었을까?

현재 필자는 프리랜서를 하고 있는데 얼마전에 다른 파트에서 성능에 문제가 되는 소스좀 검토해달라는 부탁을 받았다.  거기서 나한테 인사이트를 주는 개발자와 함께 소스를 보면서 중복되고 불필요한 구현 부분에 대해 이야기를 했다. 그러자 그 개발자가 이렇게 말했다.

> 네. 중복되고 불필요한 부분이 있네요. 하지만 머 이유가 있었겠죠. 그리고 이것이 성능 저하의 중요한 원인 같아보이진 않아요. 좀 더 확실한 다른 부분을 찾아보죠.

또 한대 맞았다. 비난보다 문제 해결이 먼저였다. 

운이 좋게도 내가 일하는 파트에서는 각 분야의 고수들이 모였다. 프론트, 퍼플리셔, 디자인, 설계등등.  일년정도 같이 일해본 경험은 그들은 한결같이 상황이나 누군가를 탓하지 않았다. 그들은 항상 다음과 같이 말했다.

> 조금 복잡하지만 불가능하진 않아요.
>
> 일단 한번 해보고 말씀 드릴게요.
>
> 현재 상황에서는 이렇게 하는게 최선일 것 같아요.

하지만 어떤 사람들은 이렇게 말한다

> 다 엉망이에요. 못해요
>
> 그건 안되요. 전부 다 뜯어 고쳐야 되요. 

물론 경우에 따라서 저렇게 후자의 표현이 정말 맞을 때도 있다. 그런데 지나치게 문제 회피적인 태도를 태하는 것이 하수들의 특징이다.  반면에 고수들이 문제 수용적(?) 태도를 보일수록 일을 시키는 소위 갑들도 함부로 대하지 않았다. 그들의 의견을 존중하고, 인정했다. 

그 사람의 인품이나 됨됨이는 어려운 상황을 맞을 때 나온다고 한다. 평화롭고 풍요로운 시간에는 누구든 친절하고 따뜻하다. 하지만 진면목은 어려운 상황에서 나온다. 마찬가지로 고수와 하수의 차이는 어려운 문제를 마주할때 나온다. 고수들은 문제 해결의 최선의 방법을 먼저 찾지만, 하수들은 비난할 대상을 먼저 찾는다. 하수들의 이야기를 들어보면 틀린 말은 하나도 없다. 하지만 그들은 그렇게 비난과 불평만 한다.  물론 개발자라면 더 좋은 코드, 더 나은 방식으로 개발을 목표로 하는 것이 맞다. 반드시 그러게 해야 한다. 요점은 지나온 것을 탓하기 보다 해결에 초점을 맞추어야 한다는 것이다. 

흔히 고수와 하수는 업무적/기술적 차이가 발생한다. 그 차이가 존재하는 이유는 고수는 문제해결에 초점을 맞추니, 당연히 실력이 좋아 질수 밖에 없다. 반면 하수는 문제비난에 초점을 맞추니, 실력이 나아질 수 없다.  문제를 바라보는 출발점이 다르기 때문이다. 고수들이 한 모든 것이 완벽하지는 않다. 하지만 그들은 완벽함을 찾기전에 상황에 맞는 최선의 방법을 찾고 뛰어든다.

당신은 누구랑 일하고 싶은가? 아니, 당신은 하수인가 고수인가?







