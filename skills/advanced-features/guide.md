# 고급 기능 연동 가이드

## 사전 조건
- iOS 15.0 이상
- AdFitSDK가 Swift Package Manager(SPM)를 통해 프로젝트에 설치되어 있어야 합니다.
- 네이티브 광고(AdFitNativeAdLoader) 연동이 완료된 상태여야 합니다.

---

## 1. 타겟팅 향상을 위한 지면 데이터 입력

사용자에게 향상된 타겟팅으로 맞춤형 광고를 노출하고 싶다면, 광고를 요청할 때 추가 정보를 입력할 수 있습니다. 모든 항목은 옵셔널 값입니다.

### 1.1 ContentObject (컨텐츠 정보)

#### Swift
```swift
let contentObject = ContentObject()
    .setId("contentId")                          // 컨텐츠 ID
    .setUrl("https://abcd/efgh/")                // 컨텐츠 URL
    .setTitle("여기는 타이틀 입니다")                // 컨텐츠 제목
    .setSearchKeyword("샤넬")                     // 유입 키워드
    .setKeywords(["놀이", "기다림", "신발장"])       // 키워드 목록
    .setCategoryTaxonomies(1000)                  // 카테고리세트 번호
    .setCategories(["a", "b", "c"])              // 컨텐츠 분류
    .setSectionCategories(["aa", "bb", "cc"])     // 상위 섹션 카테고리

nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}", contentObject: contentObject)
```

#### Objective-C
```objc
ContentObject *contentObject = [[ContentObject alloc] init];
[contentObject setId:@"contentId"];
[contentObject setUrl:@"https://abcd/efgh/"];
[contentObject setTitle:@"여기는 타이틀 입니다."];
[contentObject setSearchKeyword:@"샤넬"];
[contentObject setKeywords:@[@"a", @"b", @"c"]];
[contentObject setCategoryTaxonomies:1000];
[contentObject setCategories:@[@"a", @"b", @"c"]];
[contentObject setSectionCategories:@[@"aa", @"bb", @"cc"]];

nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:nil contentObject:contentObject];
```

#### ContentObject 메서드 목록

| 메서드 | 설명 |
|--------|------|
| `setId(_ id: String)` | 컨텐츠 ID |
| `setUrl(_ url: String)` | 컨텐츠 URL |
| `setTitle(_ title: String)` | 컨텐츠 제목 |
| `setSearchKeyword(_ search: String)` | 현재 컨텐츠를 찾는데 사용된 검색어(유입 키워드) |
| `setKeywords(_ keywords: [String])` | 키워드 목록 |
| `setCategoryTaxonomies(_ cattax: Int)` | [카테고리세트 번호](https://github.com/InteractiveAdvertisingBureau/AdCOM/blob/main/AdCOM%20v1.0%20FINAL.md#list_categorytaxonomies) |
| `setCategories(_ cat: [String])` | 컨텐츠 분류 |
| `setSectionCategories(_ sectioncat: [String])` | 컨텐츠 상위 섹션의 카테고리 (예: 뉴스/스포츠) |

### 1.2 UserObject (유저 정보)

#### Swift
```swift
let userObject = UserObject()
    .setKeywords(["서울", "판교", "부산"])    // 사용자의 관심 키워드 목록

nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}", userObject: userObject)
```

#### Objective-C
```objc
UserObject *userObject = [[UserObject alloc] init];
[userObject setKeywords:@[@"서울", @"판교", @"부산"]];

nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:userObject contentObject:nil];
```

### 1.3 ContentObject + UserObject 함께 사용

#### Swift
```swift
let contentObject = ContentObject()
    .setId("contentId")
    .setTitle("컨텐츠 제목")

let userObject = UserObject()
    .setKeywords(["서울", "판교"])

nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}", userObject: userObject, contentObject: contentObject)
```

---

## 2. Opt out 버튼 설정

광고단위에 따른 서버의 설정값에 따라 SDK에서 OptOut 버튼에 대한 배치를 처리하고 있습니다.
옵트아웃 버튼의 구현 및 배치, 랜딩까지 모두 SDK에서 처리합니다.

### 2.1 infoIconPosition 변경

옵트아웃의 기본 포지션은 우상단(topRight)입니다. 동영상 광고의 볼륨 버튼과의 레이아웃 충돌을 방지하기 위해 좌상단(topLeft)으로 변경할 수 있습니다.

#### Swift
```swift
nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}")
nativeAdLoader?.delegate = self
nativeAdLoader?.infoIconPosition = .topLeft
nativeAdLoader?.loadAd()
```

#### Objective-C
```objc
self.nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:nil contentObject:nil];
self.nativeAdLoader.delegate = self;
self.nativeAdLoader.infoIconPosition = AdFitInfoIconPositionTopLeft;
[self.nativeAdLoader loadAdWithKeyword:nil];
```

---

## 코드 생성 규칙

사용자에게 코드를 생성할 때 아래 규칙을 따릅니다:

1. `{clientId}`는 사용자가 제공한 광고 단위 ID로 치환합니다.
2. ContentObject, UserObject의 각 메서드는 모두 옵셔널이므로, 사용자가 필요한 항목만 포함합니다.
3. 사용자가 지면 데이터 입력과 Opt out 설정을 동시에 요청한 경우, 하나의 코드에 통합하여 제공합니다.
4. 사용자의 클래스명이나 변수명이 주어진 경우 해당 이름을 사용합니다.
