## Q) 아키텍처를 사용해서 보다 가독성있고 가용성있게 코드를 작성해야할 필요성을 느끼는 건 맞지만 왜 Clean Architecture여야만 하는가?

### a. 좋은 아키텍처는 다음을 수행합니다.

1. `가독성있고`, `유지 보수 및 배포하기 쉬운 시스템` 만들기
2. `시스템 정책을 분리`
3. 시스템을 `테스트 가능`하게 만들기
4. `코드 재사용성` 장려

### b. 클린 아키텍처란?

클린 아키텍처는 개발자라면  클린 코드를 저술한 로버트 마틴(Robert C. Martin)이 제안한 `시스템 아키텍처`로, 기존의 `계층형 아키텍처`가 가지던 의존성에서 벗어나도록 하는 설계를 제공합니다.

### c. 그래서 왜 클린 아키텍처가 필요한가?

nhncloud 블로그 발췌)

여러분이 A 배달 앱의 개발자이며, 어느 날 A 배달 앱이 B 배달 앱과 통합된다고 가정하겠습니다. 이때 여러분은 다음과 같은 요구를 받게 됩니다.

**“A 배달 앱 시스템이 잘 되어 있으니 A 앱의 핵심 기능은 유지하고, `UI`와 `DB` 쪽만 바꿔 주세요.”**

또는 다음과 같은 요구를 받을 수도 있습니다.

**“A 배달 앱이 너무 잘되니 서비스를 `웹으로 확장`해 봅시다.”**

비즈니스의 로직은 비슷한데, 변경해야 하는 부분도 많고, 아예 새로 만들 수도 없는 이러한 상황이라면, 여러분은 어떻게 하실 건가요?

만약 클린 아키텍처를 도입했다면, 단순하게 `**인터페이스 어댑터 영역**`과 **`프레임워크`**와 **`드라이브 영역`**만 수정하면 됩니다. 왜냐하면 ‘고객과 업체 사이에서 배달 서비스를 중계한다’는 비즈니스 로직은 변하지 않았기 때문입니다. 이와 같이 클린 아키텍처는 비즈니스 로직은 바꾸지 않으면서, 언제든 DB와 프레임워크에 구애 받지 않고 교체할 수 있는 아키텍처인 셈입니다.

⇒ `시스템 정책을 분리` 및 `유지 보수` 용이

<br>

`Clean Architecture` 시스템 아키텍처 도식화

<img src="https://github.com/kudoleh/iOS-Clean-Architecture-MVVM/raw/master/README_FILES/CleanArchitecture+MVVM.png?raw=true" width=60%>

### Domain Layer

가장 깊은 계층으로 다른 계층을 의존하지 않음. 다른 계층의 어떤 것(프로퍼티, 함수)도 포함하지 말아야 함

`Entities(Business Models)`, `Use Cases`, and `Repository Interfaces`을 포함하는 계층

- **Use Case**: `비즈니스 로직`이 들어 있는 영역
- **Entity**: `화면에 표시할 데이터`

### Presentation Layer

하나 이상의 `Use Cases`를 실행하는 `ViewModel`은 `View`를 조정. `Presentation Layer`는 오직 `Domain Layer`만 의존.

- **View**: `화면에 데이터를 표시`
- **Presenter**: `ViewModel`과 같은 역할을 하고 사용자 `인터렉션을 어떻게 처리할지(핸들링)`하는 영역

### Data Layer

저장소의 구현부들과 하나 이상의 `Data Sources`를 포함하는 계층.

서로 다른 `Data Sources`를 조정(Encoding, Decoding).

- **Repository**: `Use Case`가 필요로 하는 데이터의 `(post, get, put, delete)`와 `CRUD` 등의 기능을 제공하는 영역으로, 데이터 소스를 인터페이스로 참조하여, `네트워크 통신`과 `로컬 DB`을 자유롭게 할 수 있습니다.
- **Data Source**: `실제 데이터의 입출력`

### QA) 상위 하위 모듈로 나누는 기준

상위: 변하기 어려운 것 

하위: 변하기 쉬운 것

※ 참고 출처

[codemagic](https://blog.codemagic.io/clean-architecture-explored/)

[nhncloud](https://meetup.nhncloud.com/posts/345)  

[kudoleh](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM)