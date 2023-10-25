일상을 공유하는 작은 커뮤니티를 직접 개발해 운영하고 있는 김배달(1세).
사이드프로젝트로 만든 커뮤니티에 많은 사람들이 들어와 응원의 글을 남기자 배달이는 행복해해요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5mRGT%2Fbtsy451Hjjj%2Fel4IhMlKIV2jWqEra46UK1%2Fimg.png)

그런데 어느날!
카카오톡에 '되게신난 배달이' 신상 이모티콘이 출시되고,
이 이모티콘이 선풍적인 인기를 끌며
배달이의 카페에 유래없이 많은 사람들이 유입되기 시작했어요.

하지만 쪼꼬미 배달이처럼 쪼그만 서버와 데이터베이스로는
엄청나게 많은 사람들로 인한 트래픽을 감당하기에 너무나 버거웠고,

결국 배달이의 데이터베이스에 과부하가 걸려
고장나기에 이르렀어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsnZ7b%2Fbtsy5ec0rCp%2FfsJK9tae3hkf0uMPcP3Pk0%2Fimg.png)


데이터베이스를 뚝딱뚝딱 고쳐서 다시 가동하기까지 걸린 시간은 길어야 3시간 남짓.
하지만 그 **3시간동안 카페를 전혀 이용하지 못한** 사용자들은 불만이 가득했어요.

짜증 한가득인 사람들에게 사과문을 쓰고 마음에 깊은 상처를 받은 배달이는
**다음에 또 데이터베이스 과부하가 생기더라도
카페를 내리지 않고 계속 서비스할 수 있는 방법**을 공부해 적용하기로 합니다.

---

# 데이터베이스를 하나 더 두자 - 스케일 아웃
배달이는 생각했어요.
하나뿐인 데이터베이스가 고장나면
이것을 완전히 고칠 때까지 카페를 정지할 수밖에 없구나.

만약 **똑같은 데이터를 가지고 있는** 예비용 데이터베이스 서버를 하나 더 둘 수 있다면,
원래 쓰던 하나가 고장나더라도
예비용 서버로 카페를 운영하고 그동안 고장난 쪽을 수리하면 되겠구나!

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbO2ym1%2Fbtsy9h7BgB7%2Fkz5eNAytPfGML24TZK5Jl0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMDN78%2Fbtsy3Q5b21g%2FANCCQeSG6eZkTcWa6vrfiK%2Fimg.png)

배달이는 카페의 **단일 장애점**, 즉 **SPOF**에 해당하는 데이터베이스의 문제점을 해결하기 위해
데이터베이스에 대한 **스케일 아웃**을 도입해보기로 결정했어요.

오! 어떻게 이런 생각을 해냈담.
배달이는 자신이 천재가 아닌가 생각했어요.


실행력 강한 배달이는 우선 데이터베이스 서버 2대를 추가로 구입해,
**총 3대의 데이터베이스 서버**를 가진 구조를 갖추었어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKDHAz%2Fbtsy9gOqNid%2FsLhX400FA5GombC3RzSDc1%2Fimg.png)

헉! 하지만 서버만 뚝딱뚝딱 늘리면 데이터 관리는 조상님이 해주나요,
서버의 물리적인 숫자가 늘어나자
배달이는 이전까지 해본 적 없던 새로운 고민을 하게 되었습니다.

## 여러 개의 데이터베이스가 어떻게 데이터를 똑같이 가져? - 리플리케이션 개요
배달이 서버에 데이터베이스를 더 설치해서
데이터베이스가 총 세 개가 되긴 했는데,

실시간으로 데이터가 빠르게 생겼다 수정됐다 사라지는 상황에서
어떻게 다수의 데이터베이스 **데이터를 똑같이 <span style="color:#FF6C6C">동기화</span>, 즉 <span style="color:#FF6C6C">리플리케이션(Replication)</span>**해주느냐는 문제가 생겼습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FF1l8Q%2Fbtsy03jfOVm%2FdfndX7h4uK6lSqc7njOaZK%2Fimg.png)


데이터베이스의 리플리케이션 방법을 사흘 밤낮으로 조사한 배달이는
자신의 서버에서 사용하고 있는 MySQL에서 리플리케이션 기능을 제공해주고 있다는 것을 알게 되었고
이 기능을 공부해보기로 하였어요!

# MySQL의 리플리케이션
"**MySQL 서버에서 일어나는 모든 변경 기록**은 <span style="color:#FF6C6C">**바이너리 로그(Binary Log)**</span> 라는 파일에 순서대로 기록된다.
그리고 이 바이너리 로그 안에 기록된 변경 기록 하나하나를 **바이너리 로그 이벤트**, 또는 간략하게 <span style="color:#FF6C6C">**이벤트(Event)**</span>라고 한다."

이 문장을 읽자마자 눈을 반짝이는 배달이.
그렇다면 **운영 중인 데이터베이스에서 기록한 모든 바이너리 로그를
예비용 데이터베이스로 가져와 똑같이 적용**하면
간단하게 데이터 동기화를 해낼 수 있겠구나!

배달이는 MySQL의 데이터 복제 과정을 조금 더 자세히 들여다 보았어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcJwos5%2Fbtsy4uf4blO%2Fw8JHcwTwalYd2KIuYbnCIk%2Fimg.png)

<span style="color:#FF6C6C">**바이너리 로그**</span> -> **바이너리 로그 덤프 스레드** -> **리플리케이션 I/O 스레드** -> <span style="color:#FF6C6C">**릴레이 로그**</span> -> **리플리케이션 SQL 스레드**로 이어지는 긴 여정을 통해
두 데이터베이스가 빠르게 동기화할 수 있도록 MySQL에서 도와주고 있었어요.
(빨간색 단어들은 이 포스팅 내내 계속 반복해서 나오니 개념을 미리 꼼꼼히 읽어두면 좋습니다.)

신기방기!

---

MySQL의 리플리케이션 기능을 이용해 이 3대의 서버 데이터를 동기화하기 위해서는
먼저 리플리케이션에서 아주 중요한 전략 하나를 정해야 했는데,
바로 **소스 서버에 있는 바이너리 로그의 이벤트 하나하나를 리플리카 서버에서 어떤 방식으로 읽어올 것인가**였습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMAyA3%2Fbtsy79Wxy7z%2FG8Coht4yVPYBijLmTvdiT1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FTkbSm%2Fbtsy4uf4btW%2FwCvm6hKoKwaJgRoE5VXOZK%2Fimg.png)


# 배달이의 고민 1 - 바이너리 로그를 어떻게 식별할까

## 1. 바이너리 로그 이벤트의 이름과 위치를 통해 식별하자

바이너리 로그 파일 안에는 여러 개의 이벤트들이 순서대로 저장되어 있습니다.
'그럼 **복제하려는 파일의 이름**이랑, **복제해 올 이벤트의 위치만 정확하게 알 수 있다면**
이걸 사용해 로그를 식별하면 되겠다!' 라고 배달이는 생각했어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbl7osl%2Fbtsy7OkN4TH%2Fck42kbFbrvDEFyYT4ynO0K%2Fimg.png)

그 생각을 그대로 재현한
<span style="color:#FF6C6C">**바이너리 로그 파일 위치 기반 리플리케이션**</span>을 적용해 두 데이터베이스의 동기화에 성공한 배달이.
소스 서버에서 리플리카 서버로 데이터를 쇽쇽 복사해주는 것을 지켜보면서 뿌듯해하고 있습니다.

### 위 방법의 문제점

그런데 어느날, 또다른 배달이 이모티콘 신상이 출시되면서
소스 서버가 또 터지고 말았어요!

배달이는 하얗게 불타버린 기존의 소스 서버 대신
읽기 전용으로 사용하던 리플리카 서버를 새로운 소스 서버로 대체하고,
백업용으로 사용하던 리플리카 서버를 읽기 전용 서버로 대체하려고 했습니다.

![](https://velog.velcdn.com/images/backfox/post/5b73f2bd-b4c3-4c96-8cb8-dd30f555b1da/image.gif)


하지만 백업용 리플리카 서버의 상태를 보고 크게 당황한 배달이.
통계용 리플리카 서버에는 소스 서버로부터의 동기화가 50분 정도 늦게 진행되어,
소스 서버가 가지고 있던 원본 데이터와 50분만큼의 차이가 있는 상태였어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdYghPv%2Fbtsy8usAfTr%2FKssESvbwFFWSPEobvQqixK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FtBHch%2Fbtsy03Q7kmJ%2FAM80gjUjjdCOPGH2pBbvUK%2Fimg.png)


'으잉, 그래도 A 서버는 소스 서버와 같은 데이터를 가지고 있으니까,
A 서버의 바이너리 로그를 읽어서 데이터 동기화를 이어가면 되겠다!' 라는 똑똑한 생각을 하는 배달이.
하지만 **바이너리 로그 위치 기반 리플리케이션의 치명적인 단점**으로 인해

배달이의 계획은 무용지물이 되었고, 결과적으로
**과부하로 마비된 데이터베이스 서버는 1대인데**
**실제로 쓸 수 없게 된 서버는 2대가 되는** 참혹한 상황을 맞게 되었어요.

결국 2대가 분담해야 할 대량의 부하를 1대가 감당하게 되었고,
또다시 배달이의 카페는 새하얀 오류 화면만을 보여주게 되었습니다.

어떤 문제가 있었던 걸까요?

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb1QwEU%2Fbtsy9PQz6HN%2FndYeYQMqKBwoOGGcHaIK50%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcn3us0%2Fbtsy6jk5JN6%2FSc5aEkgh76XjRbEQwwcMv1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdaLbHa%2Fbtsy8b7TQ4o%2FqQD7Zd1tZ6huR08Zn8m3kk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdFnrVr%2Fbtsy4jeLRt1%2FwxQI2iG7CoVl8ma9sYC7mk%2Fimg.png)

결국 또다시 반성문을 쓰게 된 불쌍한 배달이.
바이너리 로그 파일 위치 기반 복제의 문제점을 몸소 깨닫게 되었네요.

## 2. 바이너리 로그 이름을 통일해서 사용하자

'**특정 서버에서만 식별할 수 있는 이름과 위치 정보는 다른 서버에서 이용할 수 없다.**'

뼈아픈 깨달음을 얻은 배달이는
리플리카 서버에서 소스 서버의 이벤트를 읽어올 더 나은 방법을 생각해내야 했어요.

그러다 문득 떠오른 아이디어.
'소스 서버와 리플리카 서버에서 실행한 작업의 내용은 동일한데도
**파일을 표현하는 방법이 달라서** 인식하지 못했던 거잖아.
**같은 작업을 수행한 이벤트는 어떤 서버에서든 같은 이름을 갖게 한다면**
문제를 해결할 수 있지 않을까?'

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FA3Nla%2Fbtsy9UEkiJS%2FKTi5AfSx4mDnkGyauqYih1%2Fimg.png)

놀랍게도, 5.5 버전 이후의 MySQL은 이 아이디어를 실현하여,
<span style="color:#FF6C6C">**글로벌 트랜잭션 아이디(Global Transaction Identifier, GTID)**</span>를 이용한 복제 방식을 제공하고 있었습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWt4Jn%2Fbtsy9OjPC9n%2FdTLMut3gl7bPGiXcNNWUmK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWVuon%2Fbtsy9dqD0h3%2F5OyjNXCchlMpUalADS6Ir1%2Fimg.png)


### GTID 기반 복제의 효과

어느날 배달 어쩌구저쩌구 컬렉션이 또 출시되어
소스 데이터베이스 서버가 또 고장났어요!
GTID 기반 복제를 적용한 지금은 과연 장애 대처를 잘 해낼 수 있을까요?

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcGF69F%2Fbtsy03Q7IMW%2FJ5dVzRKPTakD8rwv1KzcOk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fqb6ll%2Fbtsy8oTtROj%2FTLxKFEQKGhc9VvKjIfT7uk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc2Zfbm%2Fbtsy3hhdtFK%2FpkApAdFwkjlgjjrB0GKyKk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlVblq%2Fbtsy8dSceOE%2F7tViD5DjjfrDr7RR21EbcK%2Fimg.png)

와우!
바이너리 로그 파일 위치 기반 복제와 다르게
소스 서버의 GTID를 리플리카 서버에서도 이용할 수 있게 됨으로써
리플리카 서버 A로부터 리플리카 서버 B로의 데이터 동기화가 가능하게 되었어요.

만세를 부르는 배달이!

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbgFXQU%2Fbtsy8csdCzs%2FkiW8G98CWMD3SN1SWES1Z1%2Fimg.png)

---

### 요약
> - MySQL 리플리케이션의 복제 타입
1. **바이너리 로그 파일 위치 기반 복제**
2. **글로벌 트랜잭션 아이디(GTID) 기반 복제**
- 자료
  Real MySQL 8.0 2권 434p ~ 445p

---

# 배달이의 고민 2 - 이벤트에  어떤 내용을 기록해야 해?

GTID 기반의 안정적인 복제 방식을 도입한 후
잘 운영되는 카페를 보며 행복한 나날을 보내는 배달이.

그런데 어느 날, 데이터베이스에 저장된 회원 데이터를 전체적으로 업데이트하는 작업을 하던 도중

데이터베이스의 소스 서버와 리플리카 서버 A, 리플리카 서버 B의
데이터베이스의 여유 공간이 부족하다는 경고를 차례대로 보내왔어요.

데이터를 수정만 했지, 한꺼번에 대량으로 추가하는 작업은 하지 않았기 때문에 매우 의아해하던 배달이는
그 원인이 바이너리 로그에 있음을 알아차리고 깊은 고민에 빠졌어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FFaP75%2Fbtsy8jkxQJm%2FTsUTkmkkPdkFKojtorHpCK%2Fimg.png)

## 변경된 데이터를 기록한다면 - Row 포맷

배달이네 서버의 데이터베이스는 바이너리 로그의 각 이벤트들을 <span style="color:#FF6C6C">**Row 포맷**</span>으로 기록하고 있었는데,
변경된 데이터를 그대로 바이너리 로그에 등록하는 특성상
**굉장히 많은 데이터를 변경한 경우**
**그 대량의 데이터들이 전부 로그에 기록되면서 데이터베이스 공간에 과부하를 줄 수 있다**는 문제가 있음을 알게 되었어요.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbB6vr0%2Fbtsy3gJpNDf%2FtLHfkWm2IVUQoaTW5CdTj1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcePco0%2Fbtsy7iffK2m%2FlKEiyK3r3Sd1CWFqyFW4I1%2Fimg.png)


그렇다면 변경된 데이터를 그대로 바이너리 로그에 기록하는 대신
다른 방법을 쓸 수는 없을까.
**차라리 실행한 SQL을 기록하는 게 낫지 않을까?**

## 실행한 SQL을 기록한다면 - Statement 포맷

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FOJQpp%2Fbtsy7QXesE7%2FeBKNxiEy1Z7b20tK9Rq1D0%2Fimg.png)

각 이벤트에서 실행한 SQL문을 바이너리 로그에 기록하는
<span style="color:#FF6C6C">**Statement 기반 바이너리 로그 포맷**</span>을 도입한다면
바이너리 로그의 용량이 크게 줄어들기 때문에 더 이상 용량 걱정을 하지 않아도 될 거에요.

하지만 Statement 포맷을 선뜻 적용하기에는 찝찝함이 느껴집니다.

SQL문이 동일하다고 해도,
**실행할 때마다 결과가 다르게 나타나는 몇몇 비확정적(Non-Deterministic) 쿼리**들이 있다면
소스 서버와 리플리카 서버의 데이터가 다르게 기록되는 정합성 문제가 일어날 수 있기 때문이었는데요!

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb3aMeY%2Fbtsy9kDgDwu%2FjohtqZrl1YIdkPDwanjI5K%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FllnwZ%2Fbtsy43vZcB4%2FD6KgValri1jOcDqkC6BDTk%2Fimg.png)

Statement 포맷을 사용하게 된다면
**데이터베이스의 저장 공간을 취하는 대신 데이터 정합성이 깨진다**는 딜레마에 빠지고 만 배달이.

데이터를 동일하고 안전하게 복제하면서도 바이너리 로그의 크기를 적당하게 유지할 수 있는 좋은 방법은 없을 걸까요 🥹

## 두 방법의 장점만 취할 수 있다면 - Mixed 포맷

이런 고민을 MySQL 개발자들도 당연히 했을 것!

배달이가 겪고있는 딜레마를 해소해주기 위해 MySQL에서는
**평소에는 Statement 포맷으로 기록**하다가,
만약 **Statement 포맷으로 복제했을 때 문제가 될 가능성이 있는 쿼리인 경우에는
Row 포맷으로 변환하여 기록**해주는 <span style="color:#FF6C6C">**Mixed 포맷**</span>을 제공하고 있습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fq1MDi%2Fbtsy9lIWCB4%2Fh2eGVv0dIOTdZLscYZiOwk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fx8kAS%2Fbtsy3ONblgI%2FBogEETbz6C3MofUUbgte11%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNsRi8%2Fbtsy4hVAg6C%2FroKO9t9hhX2VPKzBIxp6gK%2Fimg.png)


Mixed 포맷을 적용함으로써
바이너리 로그의 용량과 데이터 정합성을 성공적으로 지켜낸 배달이!

시행착오를 겪으면서 많은 공부와 성장을 해내고 있습니다.

### 요약
> - MySQL 리플리케이션의 복제 데이터 포맷
1. **Statement 기반 바이너리 로그 포맷**
2. **Row 기반 바이너리 로그 포맷**
3. **Mixed 포맷**
- 자료
  Real MySQL 8.0 2권 469p ~ 485p

# 무뇽의 고민 3 - 복제하는 동안 데이터에 차이가 있지 않을까

데이터의 리플리케이션을 처음 경험하는 배달이는 이런 고민을 하기도 했습니다.
'소스 서버에서 리플리카 서버로 데이터를 복제하는 데에도 시간이 걸리잖아.
**복제하는 동안은 두 서버 사이의 데이터가 다를텐데, 문제가 되지는 않을까**?'

## 비동기

이런 고민이 들 만도 한 것이,
배달이네 카페의 데이터베이스 리플리케이션은 <span style="color:#FF6C6C">**비동기 복제(Asynchronous replication)**</span>방식으로 동작하고 있었기 때문인데요!

이름에서 유추할 수 있듯, 소스 서버의 이벤트가 리플리카 서버들에게 잘 전송되었는지 확인하지 **않기** 때문에
서버들 간 데이터에 차이가 생길 가능성이 있습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F2rbdg%2Fbtsy8kKxzsZ%2FJT5shf76yIKpA97qgZ4nv0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F0Du6z%2Fbtsy5cGis6G%2FEXwV86b3EPxGf8J7ZlDThk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbN9JZc%2Fbtsy7QQtBc4%2FIUtvoktYkQkK9wRT5SO8gk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbRp3YD%2Fbtsy9PXmTXl%2FsyIWEKlYwZ1f2XxRtGUJCK%2Fimg.png)

비동기 방식의 리플리케이션을 계속 하기에는 찜찜한 배달이.
'동기 방식' 같은 더 안전한 동기화 방법은 없을까 찾아보던 중
웃긴 이름의 동기화 방식을 알게 되었습니다.

## '반'동기

<span style="color:#FF6C6C">**반동기(Semi-synchronous replication)**</span>.
동기면 그냥 동기 하지 왜 **반**만 동기일까요?

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc4fqjC%2Fbtsy4t9lq5h%2FpEeN3ZKsAZ2AtL1x0D9eE0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwCfLV%2Fbtsy9UqN2Cz%2FSxeOlwMXTyEKdCX6Cslf51%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fd1xHdZ%2Fbtsy05OSvIJ%2FenBkRuGXjKksupVQyGzSuK%2Fimg.png)

비동기 복제 대신 반동기 복제를 선택하면
소스 서버와 리플리카 서버 사이의 동기화를 '어느 정도'는 보장할 수 있겠네요!

하지만 반동기 복제 또한
**정확히 어느 시점에 이벤트를 리플리카 서버로 전송하고 응답을 받느냐**에 따라
<span style="color:#FF6C6C">**AFTER_SYNC**</span>, <span style="color:#FF6C6C">**AFTER_COMMIT**</span>이라는 두 종류로 나눌 수 있었습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwwJhm%2Fbtsy43W6n6A%2FoNFUCJyfCv6pdM9JCyIFU0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdO3Lu2%2Fbtsy7QXe7Qm%2F9dNl2PGK3yauAvEYXjpno0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F39s7m%2Fbtsy46GhNGT%2FgLWqun7Tgk0KKA77o5J5Kk%2Fimg.png)

흔치 않은 장애 상황에서만 일어날 수 있는 문제이지만,
**팬텀 리드**와 같은 데이터 정합성 문제가 일어날 여지가 있는 방식이 AFTER_COMMIT이군요 🤨

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FAym4y%2Fbtsy4mWGDoy%2FUqdAzxVeglbZWAWzuW4Jv0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbx2i4q%2Fbtsy3NHtWBI%2Fi7fyZMvFKFaxVdI4xscXOk%2Fimg.png)

데이터의 정합성을 지키려면
AFTER_SYNC 방식의 반동기 복제를 적용하면 되겠다는 결론을 내린 배달이.
반동기 복제 적용 방법이 MySQL 문서에 자세히 나와있으니 그대로 따라하기만 하면 되지만,
적용하기 전 마지막으로 진지하게 고민해보기로 합니다.
**데이터베이스의 속도를 내리면서까지 꼭 정합성을 지켜야만 할까**?

## 굳이?
'사용자가 소스 서버에 쓰기 작업을 하고, 리플리카 서버에서 그 데이터를 읽으려 하면 데이터가 없을 수도 있잖아!' 라고 생각한다면
데이터 정합성을 위해 반동기 방식을 쓰는 게 안전해 보이지만,
사실 데이터의 리플리케이션 작업은
우리의 생각보다 훨씬 짧은 시간안에 이루어진다고 해요.

200~300밀리초라는 어마어마하게 짧은 시간 안에
쓰기 -> 읽기 작업을 샥샥💨 해서
데이터 불일치로 생기는 불편을 느낄 일이 과연 얼마나 있을지 먼저 생각해보아야 합니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbwTMrs%2Fbtsy6fJJZlA%2FiVjnC7Vwo55L9g129Ec3Lk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F5Uezk%2Fbtsy7fbEuLS%2Fv48PZoRd2RmPDoQ1WVEke1%2Fimg.png)


배달이는 자신이 운영하는 카페의 성격을 되짚어보았을 때,
비동기 복제 방식을 계속 쓴다고 하더라도 큰 불편이 일어날 일이 없으며
오히려 반동제 복제를 사용했을 때 데이터베이스의 작업 처리 속도가 느려져서 불편해할 사람들이 더 많겠구나! 라는 결론을 내렸어요.

그렇게 비동기 복제 방식을 계속 사용하기로 마음먹은 배달이는
**더 새롭고 좋은 기술이 있다고 해도
무조건 적용하기 전에 그것이 본인에게 꼭 필요한 기술인지 고민하는 것**이 중요하다는 깨달음을 얻었습니다. 😊

---

### 요약
> - MySQL 복제 동기화 방식
1. **비동기**
2. **반동기**
- 자료
  Real MySQL 8.0 2권 484p ~ 493p

---

# 번외 - 리플리케이션 구조(토폴로지)

지금까지 리플리케이션을 설명하면서 예시로 들었던 배달이네 카페 데이터베이스 설정은
소스 서버(쓰기용) + 리플리카 서버 1(읽기용) + 리플리카 서버 2(예비용) 이렇게 3대로 구성된 **멀티 리플리카 복제 구성**을 하고 있었습니다.

이런 멀티 리플리카 구성 외에도, 리플리케이션을 수행하는 목적에 따라 다양한 형태로 데이터베이스들을 두고 운영할 수 있어요.

일반적으로 사용되는 데이터베이스 구성 방식들을 간단히 알아본 후 포스트를 마무리하겠습니다. 🐋

## 싱글 리플리카 복제 구성

**소스 서버 1 + 리플리카 서버 1**로 구성된
가장 단순한 구성방식을 <span style="color:#FF6C6C">**싱글 리플리카 복제 구성**</span>이라고 불러요!

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbmwfcU%2Fbtsy8cy0wLK%2F6h1fabosAhsNVYGMrDfti1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fex93RP%2Fbtsy8t1wGjS%2FlKqmzC6GXKWz0KvtwkK7Y0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdCbx0z%2Fbtsy8nUzue9%2F7undmkCSLamr0sjnPKUaMk%2Fimg.png)

## 멀티 리플리카 복제 구성

싱글 리플리카 복제 구성에서,
다른 용도의 데이터베이스가 더 필요해서 여분의 리플리카 서버를 더 두기 시작한다면
<span style="color:#FF6C6C">**멀티 리플리카 복제 구성**</span>이 돼요.

배달이의 데이터베이스도 백업 용도 외에 읽기 작업을 수행하는 데이터베이스가 필요하게 되면서
멀티 리플리카 복제를 사용하게 됐었죠!

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNX9VA%2Fbtsy4j0bg2B%2FwWEKzALTuYl4KiotqpSZwK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fmwbfy%2Fbtsy36furUc%2FWITfeNAk91ZyKS5okgARFK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fycp7M%2Fbtsy4nafuiA%2FVxu5MOWwftN7qm1RJe5ba1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbNPsTq%2Fbtsy47yq3GG%2Fwa38Skk2RDw4lCCZ7O7Yv1%2Fimg.png)

## 체인 복제 구성
만약 소스 서버는 하나인데, 복제를 해주어야 하는 리플리카 서버가 엄청 많아진다면 어떻게 해야 할까요?
예를 들어, [대규모 서비스를 지탱하는 기술](https://product.kyobobook.co.kr/detail/S000001550638)이라는 서적에서 예시로 든 하테나라는 기업은 데이터베이스 서버만 25대였다고 해요.

만약 이 기업의 리플리케이션 구성이 소스 서버 1대와 리플리카 서버 24대로 이루어졌다면,
소스 서버에서 리플리카 서버들로 이벤트를 전송하는 작업 자체가 부하로 작용할 수도 있을 거에요.

이때는 **리플리카 서버가 또다시 다른 리플리카 서버에 대한 소스 서버 역할**을 하는, 사슬같은 구조의 <span style="color:#FF6C6C">**체인 복제 구성**</span>을 생각해볼 수 있습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb4lYj6%2Fbtsy4svPJeC%2FJkRJCwgbY9of9Ny6sAp7PK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F0Gvee%2Fbtsy43W6y6I%2FLlPYJ7k1wp0kUvI7qjXBp0%2Fimg.png)


![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcJxQ5z%2Fbtsy5ejNrx9%2FbRZ0rJOusUw6wa133dz5g1%2Fimg.png)

MySQL 서버를 전체적으로 업그레이드하거나, 장비를 일괄적으로 바꿀 때 복제 그룹 단위의 교체를 할 수 있는데요!
대략 아래와 같은 과정을 통해 교체가 이루어집니다.

![](https://velog.velcdn.com/images/backfox/post/519747cb-9af2-4cc0-865f-0f7573498647/image.gif)

## 듀얼 소스 복제 구성

지금까지 알아본 복제 구성은
데이터 쓰기용 서버는 하나만 사용하고,
읽기용 서버만을 확장한다는 전제를 바탕으로 한 구성이었어요.

하지만 쓰기 작업을 아주 많이 하는 서비스인 경우에는
**쓰기용 서버를 여러 개** 두어야 할 수도 있습니다!

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbh0aFp%2Fbtsy8iMHSF4%2FGNnFlQksdcXkkcP1GmKVN1%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMzZR8%2Fbtsy7RIA9m2%2FcISzskk6Pd6JhbcD1wdIH1%2Fimg.png)

### ACTIVE-PASSIVE와 ACTIVE-ACTIVE

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdmyREY%2Fbtsy3fRjwMs%2FcnwWqEsKeR0BlEbyVl3iKk%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnC6mn%2Fbtsy9TMc0ZR%2F4iVZHFnekxHnJh45rRWuQ0%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbI4B2k%2Fbtsy8aHWrU9%2FJ0IhdpCI90DaN7ZOfClOiK%2Fimg.png)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbjV39B%2Fbtsy39XPLvk%2FUkWZACooCJdB9GgVcRXT50%2Fimg.png)

---

### 요약
> - MySQL 리플리케이션의 복제 토폴로지
1. **싱글 리플리카 복제 구성**
2. **멀티 리플리카 복제 구성**
3. **체인 복제 구성**
4. **듀얼 소스 복제 구성**
- 자료
  Real MySQL 8.0 2권 494p ~ 503p

---

이렇게 배달이네 카페의 시행착오를 따라가보면서
데이터베이스, 그 중에서도 MySQL의 리플리케이션 작업에 대해 알아보았습니다.

참고자료로 활용한 [Real MySQL 8.0 2권](https://m.yes24.com/Goods/Detail/105536168)에 훨씬 자세한 내용이 기록되어 있으니 참고하시길 바라요!

읽어주셔서 고마워요.
배달이도 이제 행복하게 살아야 한다! 🙌🏻🙌🏻🙌🏻

---

도움을 주신 분
* [우아한테크코스의 멋쟁이 천재 박스터](https://drunkenhw.github.io)
* [우아한테크코스의 멋쟁이 주노](https://github.com/choi-jjunho)
