---
layout: post
title:  "라즈베리파이2(Raspberry Pi2)에서 OSMC를 이용해 영화를 보자"
date:   2016-01-16
categories: linux
---

보통 집에서 다운받은 영화나 미드를 볼 때, 거실에서 TV등과 연결해서 보고 싶을 때가 있다.
그럴려면 PC로 다운받은 영화를 USB나 외장하드에 옮기고 다시 그것을 TV에 연결해서 보거나 아님, TV에서 노트북등으로 연결해서 봐야 한다.
여간 귀찮다. 그렇다고 관련된 기기를 사자니 비용이 만만치가 않다.

그러나 라즈베리파이라는 초소형 컴퓨터를 이용하면 아주 저렴하게 집에서 미디어센터를 구축할 수 있다.
라즈베리파이는 명함크기정도의 아주 작은 컴퓨터이다. 가격은 보통 4~5만원 정도밖에 하지 않는다.
라즈베리파이를 이용하면 PC로 다운받은 영화를 번거로운 이동이나 복사없이 바로 볼수있다. 물론 토렌토등을 이용해서 직접 라즈베리파이에서 받을 수도 있다.

### 1. 준비물 (하드웨어, 소프트웨어)

필요한 하드웨어들은 다음과 같다.

- 라즈베리파이2 (반드시 2이어야 한다)
- 라즈베리파이2 공식케이스 : 다른 케이스는 잘 부서진다고 함
- 5PIN 전원 연결장치 : 보통 안드로이드 스마트폰 충전기 (5v, 2000mA)
- 무선랜카드 (물론 랜선으로도 가능하다)
- Micro SD Card : OS를 설치용 (최소 8G이상이어야 함: 권장은 16 or 32G)
- Micro SD Card Reader : PC에서 Micro SD 에 연결하기 위함.
- HDMI 케이블 : TV 연결
- 키보드(USB 연결)

![ras1](/assets/images/ras1.jpg)

보통 옥션이나 11번가에서 관련 장비를 한꺼번에 파는 경우가 있으니 일괄구매하는 것이 나을 수 있다.


**전원 연결장치는 5v에 2000mA (2A)를 구매하는 것이 좋다. 일반 스마트폰은 1000mA(1A)인데 이것을 사용하면 전류가 낮아서
OSMC에서 조그만 무지개박스가 보이게 된다. 물론 영화보는데는 지장은 없지만, 추후에 외장하드연결을 할 수 있으니 2000mA로 하는 것이 좋다
(보통 USB로 연결하는 것이 아닌 전원플러그와 5pin 있는것)**

라즈베리파이2에는 4개의 USB 포트, 1개의 HDMI 포트, 그리고 전원입력포트와 Micro SD카드 입력부분을 가지고 있다.

필요한 소프트웨어는 다음과 같다.

- [SD formatter](https://www.sdcard.org/downloads/formatter_4/) : SD카드를 포멧한다.
- [OSMC](https://osmc.tv/) : 라즈베리파이2에 설치될 OS
- [PUTTY](http://www.putty.org/) : 윈도우에서 라즈베리파이에 접속터미널

`SD formatter`가 필요한 이유는 고용량의 SD카드를 윈도우에서 제대로 감지 못한다. 그래서 정식포맷프로그램을 이용해서 포멧한다.

### 2. Micro SD 설치
모든 준비가 완료되었다면 이제 SD카드에 설치를 해보자 그전에 먼저 SD formatter를 이용해서 포멧부터 한다.

![ras2](/assets/images/ras2.jpg)

처음에는 용량이 작게 보이나(예를 들어 32G인데 236M로 나옴) 당황하지 말고 포멧을 누르면 정상적인 용량으로 포멧이 되어진다.

`OSMC`에서 인스톨 프로그램을 다운받는다. `OSMC`는 미디어센터 OS이다(물론 공짜다)
작업할 PC에 맞는 OSMC 인스톨 프로그램을 받는다.

![ras3](/assets/images/ras3.jpg)


여기서는 윈도우 기반으로 설명한다.
OSMC 관련 파일을 받으면 osmc installer.exe를 실행하자.

![ras4](/assets/images/ras4.jpg)
*Raspberry Pi2 를 선택한다*

![ras5](/assets/images/ras5.jpg)
*가장 최신의 날짜를 선택한다*

![ras6](/assets/images/ras6.jpg)
*SD카드를 선택*

![ras7](/assets/images/ras7.jpg)
*무선선택*

![ras8](/assets/images/ras8.jpg)
*본인의 와이파이 설정을 해주면 된다*

그 다음은 Next만 계속 해주면 된다. 그러면 downloading과정과 install 과정을 마치게 되면 congratulations 화면이 나오고 quit를 눌러 종료하면
SD카드에 OSMC 관련 파일이 모두 설치된 것이다.  

이제 SD카드를 라즈베리파이2에 삽입하자. 무선랜카드도달고 전원도 연결하면 아래화면처럼 진행화면이 나온다.
모두 설치가 완료되면 자동으로 리부팅된다.

![ras9](/assets/images/ras9.jpg)
*설치중...*


### 3. 세팅

![ras10](/assets/images/ras10.jpg)
*설치완료 - 환경설정*

완료된 화면이다. 처음에는 언어설정을 반드시 English(us)로 하자. Korean으로 하면 지원하는 한글폰트가 없어서 모두 깨져서 나온다.
시간 설정은 마찬가지로 seoul로 설정한다. 키보드의 화살표로 메뉴로 이동할 수 있다.

이제 한글폰트를 받아서 설치하자. 한글폰트를 설치해야 제목이 한글인 파일도 읽을 수 있고, 자막도 원할하게 읽을 수 있다.

윈도우에서  putty로 연결한다 putty는 ssh로 라즈베리파이에 접속할 수 있다.
putty에 해당 아아피를 입력하고 ssh 방식으로 접속한다.

![ras10](/assets/images/ras11.jpg)
*putty 접속*

아이디랑 비번은 초기값은 osmc/osmc 이다.
접속이 잘되면 이제 한글 폰트를 설치해야 한다. 나눔폰트를 설치할 것이다. 이 한글폰트로 시스템언어를 한글로 할 것이고, 자막폰트역시 나눔폰트로 변경할 것이다.
터미널에서 아래 명령어를 차례로 실행한다.

```bash
sudo apt-get install fonts-nanum
sudo cp /usr/share/fonts/truetype/nanum/NanumGothic.ttf /usr/share/kodi/addons/skin.osmc/fonts/
```

자막폰트를 복사한다.

```bash
sudo cp /usr/share/fonts/truetype/nanum/NanumGothic.ttf /usr/share/kodi/media/Fonts/
```

OSMC의 한글 설정을 먼저 처리하도록 하자.
vi편집기를 이용하여 수정한다. (백업을 먼저해두면 좋다)

```bash
sudo vi /usr/share/kodi/addons/skin.osmc/16x9/Font.xml
```

vi에서 xxx.ttf로 설정된 것을 전부 NanumGothic.ttf 로 변경한다.
아래는 vi 명령어를 처리하면 된다.

`:` 명령행으로 가서 한줄씩 복사하고 엔터

```bash
%s/NotoSans-Regular.ttf/NanumGothic.ttf/g
%s/OpenSans-Regular.ttf/NanumGothic.ttf/g
%s/Roboto-Light.ttf/NanumGothic.ttf/g
%s/NotoSans-Regular.ttf/NanumGothic.ttf/g
%s/Arial.ttf/NanumGothic.ttf/g
```

그 다음에 우리가 사용할 스킨에선 XBMC UI인 Confluence 를 위한 한글폰트 교체작업을 해주어야 한다.

```bash
vi /usr/share/kodi/addons/skin.confluence/720p/Font.xml
```

```bash
%s/Roboto-Regular.ttf/NanumGothic.ttf/g
%s/OpenSans-Regular.ttf/NanumGothic.ttf/g
%s/Roboto-Light.ttf/NanumGothic.ttf/g
%s/NotoSans-Regular.ttf/NanumGothic.ttf/g
%s/Roboto-Bold.ttf/NanumGothic.ttf/g
%s/Arial.ttf/NanumGothic.ttf/g
```

모든 작업이 끝났으면 이제 라즈베리파이2로 가서 한글설정으로 해주자.
먼저 OSMC 한글설정을 변경한다. `International -> Language` 에서 Korean으로 변경한다.

![ras12](/assets/images/ras12.jpg)
*OSMC 한글설정변경*

![ras13](/assets/images/ras13.jpg)
*OSMC 스킨변경*

Confluence의 스킨을 선택한다. Confluence는 XBMC UI이다. 이 스킨으로 변경하는 이유는 좀더 직관적이고
무엇보다 스마트폰으로 XBMC앱을 깔면 스마트폰을 리모콘으로 사용할 수 있기 때문이다.

![ras14](/assets/images/ras14.jpg)
*Confluence의 스킨사용*

스킨을 변경하면 적용하겠는냐는 메시지고 나오고 정상적으로 처리되면 한글로 표시될 것이다.
(한글이 안보인다면 시스템으로 가서 언어설정을 Korean으로 해주면 된다)

만약에 화면이 사이즈가 아래와 같이 안맞는다면 조정이 필요하다.

![ras15](/assets/images/ras15.jpg)
*화면짤림(시간표시가 짤린다)*

`시스템->비디오출력->비디오조정`으로 이동한다.

![ras16](/assets/images/ras16.jpg)
*화면짤림(시간표시가 짤린다)*

왼쪽상단의 모서리를 키보드 화살표로 화면에 맡게 조정한다.

![ras17](/assets/images/ras17.jpg)
*화면크기조정*

처음에는 저 꺽쇠표시가 잘 안보인다. 키보드의 화살표로 보이게 한다음 모서리에 맞추면 된다.
그런 다음 엔터를 치면 오른쪽하단에도 역시 조정하는 부분이 나온다.
이제 모든 조정이 완료가 되었다. 다음은 PC와 연결하여 드디어 영화를 보도록 하자.

### 4. 영화보기
OSMC에는 이미 SAMBA가 설치되어 있다. SAMBA를 이용해서 라즈베리파이에서 윈도우에 공유된 폴더로 직접 연결이 가능하다.
윈도우에서 영화폴더를 공유한다.

`비디오->파일`로 이동한다.

![ras18](/assets/images/ras18.jpg)
*비디오 - 파일*

`비디오소스추가`에서 `탐색` 을 선택하고 윈도우 네트워크(SMB)를 선택한다.

![ras19](/assets/images/ras19.jpg)
*윈도우 네트워크(SMB)*

선택하면 같은 네트워크에 있다면 연결할 PC이름이 보일 것이다. 계정과 비번을 입력하고, `이 경로를 기억` 을 체크하면 공유된 폴더가 나온다.
해당 폴더를 선택하고 확인하여 아래의 같은 `비디오소스추가`를 이용하여 저장한다. 이렇게 하면 다음부터는 바로 접근이 가능하다.

![ras20](/assets/images/ras20.jpg)
*비디오소스추가*

아래 화면처럼 메인화면에서 `비디오 -> 파일`로 선택하면 우리가 추가한 윈도우와 연결된 폴도명이 보인다.

![ras21](/assets/images/ras21.jpg)
*추가된 폴더*

모든 작업이 완료가 되었다. 자막도 잘 나오니 이제부터 영화를 즐기면 된다.

![ras22](/assets/images/ras22.jpg)
*영화보기*


### 5. 리모콘
XBMC 에서는 리모콘으로 조정이 가능하다. 그러나 잘 안되는 부분도 있으니, XMBC 앱을 이용하여 리모콘으로 사용하자.
집에서 놀구 있는 구형 안드로이드 폰을 사용해도 되고, 아이폰을 사용해도 되지만 안드로이드 앱은
XBMC 에서 공식으로 만든 어플이 있어 더 안정정적이다.
아이폰을 사용하려면 앱스토어에서 kobi나 xbmc등으로 검색하여 적절한 리모콘앱을 설치하면 된다.

![ras23](/assets/images/ras23.jpg)
*XBMC에서 지원하는 공식 안드로이드용 앱*


- [안드로이드 공식 XBMC 어플](https://play.google.com/store/apps/details?id=org.xbmc.kore)
