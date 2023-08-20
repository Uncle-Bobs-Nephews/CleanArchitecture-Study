## 아키텍처가 중요한 이유

![](https://velog.velcdn.com/images/qnm83/post/f8695674-b87b-4c00-9a6d-abb0b4849c33/image.png)

왼쪽 사진은 아키텍처를 생각하지 않고 코드를 작성해 `자주 변하는 잉크 병`에 상대 적으로 `자주 변하지 않는 것들`이 의존하고 있는 상황입니다.

잉크 병은 잉크를 다 쓰거나 다른 색이 필요할 때마다 교체해 줘야 합니다. 잉크 병에 의존하고 있는 것들은 잉크 병의 변화에 영향을 받아 잉크 병이 바뀔 때마다 수정해줘야합니다.

오른쪽 사진처럼 `자주 변하지 않는 것들`을 `자주 변하는 잉크 병`에 의존하지 않게 아키텍처를 개선한다면 잉크 병을 교체할 때마다 다른 것들에 영향을 미치지 않고 잉크병만 교체할 수 있습니다.

### 최근 다양한 아키텍처들의 특정

> 디테일은 다르지만 다음 규칙들을 지키려고 노력하고 있습니다.
- 소프트웨어의 layer를 나누어 `관심사의 분리`를 하려고 노력합니다.
- 나누어진 layer는 `Dependency Rule(의존성 규칙)`에 따라 의존합니다.

위의 규칙들을 지켰을 때 다음과 같은 이점이 있습니다.

#### 1. Independent of Framework

- 아키텍처는 라이브러리들에 의존하지 않게 해줍니다. 이는 framework에 의존하기보다 도구로써 사용하게 해줍니다.

#### 2. Testable

- business rule을 UI, 데이터베이스, 서버 등 다른 `외부 요소들 없이 테스트`할 수 있습니다.
- Interface(Protocol)에만 의존해 Interface의 구현체만 갈아끼우는 방식으로 쉽게 테스트할 수 있습니다.

#### 3. Independent of UI

- `UI는 시스템의 나머지 부분을 변경하지 않고도 쉽게 변경`할 수 있습니다.
-  ViewModel에서 Foundation만 import해서 View를 UIKit이든 SwiftUI이든 쉽게 변경할 수 있게 해줍니다.

#### 4. Independent of Database

- DB도 `business rule에 얽메이지 않고` 독립적으로 변경할 수 있습니다.


<br>



## The Dependency Rule

![](https://velog.velcdn.com/images/qnm83/post/5c0a9924-d660-4cb3-933e-315945f2a04e/image.png)

> source code 의존성은 `안쪽으로만` 향할 수 있습니다.
- source code 의존성은 low level에서 high level로만 향할 수 있습니다.
- high level은 low level에 의존할 수 없습니다.
- 외부 원에서 선언된 것은 안쪽 원에서 언급될 수 없고 영향을 줘도 안됩니다.
- 안쪽에 있는 원은 바깥쪽에 있는 원에 대해 아무것도 알 수 없습니다.

|High Level  | Low Level |
|:----------:|:---------:|
|자주 변하지 않음 <br> Polycy|자주 변함 <br> Mechanism  |
|안쪽			|바깥쪽       |
|추상화된 개념 <br> Abstract, General |세부적인 개념 <br> Detail |


<br>

## Layers

### Entities

- 프로젝트의 Enterprise Business Rules을 캡슐화 합니다.
- Actor가 `필요로 하는 데이터 모델`을 의미합니다.

```swift
struct Movie: Equatable, Identifiable {
    typealias Identifier = String
    enum Genre {
        case adventure
        case scienceFiction
    }
    let id: Identifier
    let title: String?
    let genre: Genre?
    let posterPath: String?
    let overview: String?
    let releaseDate: Date?
}

struct MoviesPage: Equatable {
    let page: Int
    let totalPages: Int
    let movies: [Movie]
}

```

### Use cases

- Application business rules를 포함합니다.
- Actor가 `Entity를 필요로할 때 사용되는 로직`을 의미합니다.
- 시스템의 모든 use case를 캡슐화하고 구현합니다.

```swift
protocol SearchMoviesUseCase {
    func execute(
        requestValue: SearchMoviesUseCaseRequestValue,
        cached: @escaping (MoviesPage) -> Void,
        completion: @escaping (Result<MoviesPage, Error>) -> Void
    ) -> Cancellable?
}

final class DefaultSearchMoviesUseCase: SearchMoviesUseCase {

    private let moviesRepository: MoviesRepository
    private let moviesQueriesRepository: MoviesQueriesRepository

    init(
        moviesRepository: MoviesRepository,
        moviesQueriesRepository: MoviesQueriesRepository
    ) {

        self.moviesRepository = moviesRepository
        self.moviesQueriesRepository = moviesQueriesRepository
    }

    func execute(
        requestValue: SearchMoviesUseCaseRequestValue,
        cached: @escaping (MoviesPage) -> Void,
        completion: @escaping (Result<MoviesPage, Error>) -> Void
    ) -> Cancellable? {

        return moviesRepository.fetchMoviesList(
            query: requestValue.query,
            page: requestValue.page,
            cached: cached,
            completion: { result in

            if case .success = result {
                self.moviesQueriesRepository.saveRecentQuery(query: requestValue.query) { _ in }
            }

            completion(result)
        })
    }
}

struct SearchMoviesUseCaseRequestValue {
    let query: MovieQuery
    let page: Int
}
```

### Interface Adapters

- Use Case와 Entity에 용이한 형식 <-> DB나 Web 등 외부의 기능에 용이한 형식으로 상호 데이터를 변환해주는 어뎁터의 집합입니다.
- GUI의 MVC 아키텍처를 완전히 포함한다.
  - Contoller -> Use Case -> DB or Web -> Use Cae -> Presenter -> View
  - model이 전달되면서 상황에 맞게 변환됩니다.
  
  
### Frameworks and Drivers

- 가장 바깥쪽의 layer는 Database, WebFramework 등 같은 framework, tool로 구성되어 있다. 주로 안쪽 layer 들과 소통하는 glue code 들로 구성돼있습니다.

<br>

## Crossing boundaries

![](https://velog.velcdn.com/images/qnm83/post/7838d0c1-9450-4bbc-b45b-819fcef53b8e/image.png)

Controller, Presenter가 High Level Layer인 Use Case와 소통하고있습니다. Flow of control은 Controller에서 시작해서 Use Case를 통해 이동한 다음 Presenter에서 실행됩니다.

![](https://velog.velcdn.com/images/qnm83/post/6ebab138-3f26-4855-a806-acad84083513/image.jpeg)

use case가 presenter를 직접 호출하게 되면 상대적으로 high level layer인 Use Case가 low level layer인 Presenter를 의존해 Dependency Rule을 지키지 못해 모순인 상황입니다.

![](https://velog.velcdn.com/images/qnm83/post/b3c41170-3787-4988-8c81-cb63a8fcb818/image.jpeg)

`Dependency Inversion Principle(의존성 역전 원칙)`을 사용해 이런 상황을 해결합니다. Use case는 인터페이스(Use Case Output Port)를 호출하고 바깥 원의 Presenter는 인터페이스의 요구사항을 구현합니다. dynamic polymorphism(동적 다형성)을 사용해 `flow of control을 원하는 방향으로 유지하고 소스코드 의존성도 Dependency Rule을 위반하지 않게`해줍니다.

<br>

## What data crosses the boundaries.

> 경게를 넘어 데이터를 전달할 떄는 항상 `안쪽 원의 다루기 쉬운 형식`으로 전달해야합니다. 

안쪽 원의 다루기 쉬운 형식으로 변환하지 않으면 안쪽 원이 바깥쪽 원에대해 알아야 하기때문에 dependependency rule을 지키지 못하게 됩니다.

```swift
// MARK: - Mappings to Domain

extension MoviesResponseDTO {
    func toDomain() -> MoviesPage {
        return .init(page: page,
                     totalPages: totalPages,
                     movies: movies.map { $0.toDomain() })
    }
}

extension MoviesResponseDTO.MovieDTO {
    func toDomain() -> Movie {
        return .init(id: Movie.Identifier(id),
                     title: title,
                     genre: genre?.toDomain(),
                     posterPath: posterPath,
                     overview: overview,
                     releaseDate: dateFormatter.date(from: releaseDate ?? ""))
    }
}

extension MoviesResponseDTO.MovieDTO.GenreDTO {
    func toDomain() -> Movie.Genre {
        switch self {
        case .adventure: return .adventure
        case .scienceFiction: return .scienceFiction
        }
    }
}
```

<br>

## Conclusion

### 장점

소프트웨어를 계층으로 나누고 의존성 규칙을 따름으로써 테스트하기 쉽고 Framework에 의존하지 않고 쉽게 교체할 수 있습니다.

### 단점 

클린 아키텍처의 장점도 많지만 항상 좋다고만은 할 수 없습니다. 
- 클린 아키텍처를 사용하면 코드 량과 파일 량이 늘어나 오히려 복잡성이 증가해 구조파악하기 힘들어질 수 있습니다.
- 러닝 커브가 높은 편이라 적용하는데 시간이 오래걸립니다.
- 소규모 프로젝트나 짧은 시간안에 진행해야 않은 경우 적절하지 않을 수 있습니다.
- 협업시 layer가 너무 독립적으로 분리되면 팀내 협업이 어려워질 수 있습니다.


<br>


## 참고

- [The Clean Architecture](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Clean Architecture and MVVM on iOS](https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3)
- [[iOS] 1. Clean Architecture + MVVM 개념 확실하게 이해하기 (Entity, Use Case, Presenter)](https://ios-development.tistory.com/665)
- [[NHN FORWARD 22] 클린 아키텍처 애매한 부분 정해 드립니다.](https://www.youtube.com/watch?v=g6Tg6_qpIVc&t=1813s)
- [Clean Architecture 에 대해서 알아봅시다.](https://www.youtube.com/watch?v=jVyA5DV6r8w&t=354s)
