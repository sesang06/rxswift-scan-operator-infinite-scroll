# RxSwift Scan을 이용하여 무한 스크롤링을 함수형답게 구현하기.


https://sesang06.tistory.com/133


Scan은 전의 옵저블을 옵저빙하여 새로운 값을 방출하는 옵저버입니다.

http://reactivex.io/documentation/operators/scan.html



무한 스크롤링에서는 두 옵저버가 필요합니다. 

한 옵저버는 최초 로드 액션입니다. 이 옵저버는 피드를 여러 개 가져오는 액션입니다. 만약 전에 있던 피드가 있는데 로드 옵션이 또 불린다면, 이 옵저버는 전의 피드 값을 덮어씌웁니다.

```
let loadObservable: Observable<[Feed]> = PublishSubject<[Feed]>().asObserver()
```



다른 한 옵저버는 추가 로드 옵션입니다. 이 옵저버는 기존에 있던 값에서 피드를 추가적으로 덧붙입니다.

```
let loadMoreObservable: Observable<Feed> = PublishSubject<Feed>().asObserver()
```



두 옵저버를 합쳐서 전체 피드를 구하기 위해서, 보조 enum을 하나 정의합니다.

```
 enum Action {
    case load([Feed])
    case loadMore(Feed)
  }
```

두 옵저버블을 Action 타입으로 매핑합니다.

```
let loadObservable: Observable<[Feed]> = PublishSubject<[Feed]>().asObserver()

let loadMoreObservable: Observable<Feed> = PublishSubject<Feed>().asObserver()

let loadMap: Observable<Action> = loadObservable
      .map { Action.load($0) }

let loadMoreMap: Observable<Action> =
    loadMoreObservable
      .map { Action.loadMore($0) }
```



두 옵저버블을 마지하고 스캔 연산자로 아래처럼 처리하면, 우리가 원하는 전체 피드가 나옵니다.

```
    let loadObservable: Observable<[Feed]> = PublishSubject<[Feed]>().asObserver()

    let loadMoreObservable: Observable<Feed> = PublishSubject<Feed>().asObserver()

    let loadMap: Observable<Action> = loadObservable
      .map { Action.load($0) }

    let loadMoreMap: Observable<Action> =
    loadMoreObservable
      .map { Action.loadMore($0) }

    let feedResult: Observable<[Feed]> =
      Observable.merge(loadMap, loadMoreMap)
        .scan(into: [Feed]()) { feeds, action in
          switch action {
          case .load(let newFeeds):
            feeds = newFeeds
          case .loadMore(let newFeed):
            feeds.append(newFeed)
          }
    }
```

