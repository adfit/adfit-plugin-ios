# 비즈보드 템플릿 광고 연동 가이드

## 사전 조건
- iOS 15.0 이상
- AdFitSDK가 Swift Package Manager(SPM)를 통해 프로젝트에 설치되어 있어야 합니다.

## 개요
비즈보드 광고는 배너 형태의 네이티브 요소로 구성된 광고입니다.
웹뷰가 아닌 네이티브 UI를 사용하여 광고가 노출됩니다.

---

## UIKit 연동 (단독 뷰)

### 1. 광고 호출하기

비즈보드 광고는 네이티브 광고이기 때문에, **AdFitNativeAdLoader**의 응답이 필요합니다.

#### Swift
```swift
import UIKit
import AdFitSDK

final class MyViewController: UIViewController, AdFitNativeAdLoaderDelegate {
    private var nativeAdLoader: AdFitNativeAdLoader?

    override func viewDidLoad() {
        super.viewDidLoad()

        nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}")
        nativeAdLoader?.delegate = self
        nativeAdLoader?.loadAd()
    }

    // MARK: AdFitNativeAdLoaderDelegate
    func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
        // 광고응답 -> nativeAd: AdFitNativeAd
    }

    func nativeAdLoaderDidFailToReceiveAd(_ nativeAdLoader: AdFitNativeAdLoader, error: Error) {
        // 광고응답 실패 -> error 확인하기
    }
}
```

#### Objective-C
```objc
#import "MyViewController.h"
#import <AdFitSDK/AdFitSDK-Swift.h>

@implementation MyViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:nil contentObject:nil];
    self.nativeAdLoader.delegate = self;
    [self.nativeAdLoader loadAdWithKeyword:nil];
}

#pragma mark - AdFitNativeAdLoaderDelegate
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    // 광고응답 -> nativeAd: AdFitNativeAd
}

- (void)nativeAdLoaderDidFailToReceiveAd:(AdFitNativeAdLoader *)nativeAdLoader error:(NSError *)error {
    // 광고응답 실패 -> error 확인하기
}
@end
```

### 2. 레이아웃 구성하기

SDK 내부에서 정해진 템플릿 형식으로 뷰를 제공합니다.

#### Swift
```swift
import UIKit
import AdFitSDK

final class MyViewController: UIViewController, AdFitNativeAdLoaderDelegate {
    private var nativeAdLoader: AdFitNativeAdLoader?
    private let bizBoardTemplate = BizBoardTemplate()

    override func viewDidLoad() {
        super.viewDidLoad()

        setUpBizBoardTemplate()
        nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}")
        nativeAdLoader?.delegate = self
        nativeAdLoader?.loadAd()
    }

    // AutoLayout
    func setUpBizBoardTemplate() {
        bizBoardTemplate.translatesAutoresizingMaskIntoConstraints = false
        let adViewHeight = bizBoardTemplate.adHeight(width: view.frame.width)
        // 비즈보드 비율에 맞는 높이 계산
        view.addSubview(bizBoardTemplate)
        NSLayoutConstraint.activate([
            bizBoardTemplate.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            bizBoardTemplate.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            bizBoardTemplate.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            bizBoardTemplate.heightAnchor.constraint(equalToConstant: adViewHeight)
        ])
    }

    // MARK: AdFitNativeAdLoaderDelegate
    // ... 생략 ...
}
```

#### Objective-C
```objc
#import "MyViewController.h"
#import <AdFitSDK/AdFitSDK-Swift.h>

@implementation MyViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    [self setupBizBoardTemplate];
    self.nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:nil contentObject:nil];
    self.nativeAdLoader.delegate = self;
    [self.nativeAdLoader loadAdWithKeyword:nil];
}

// AutoLayout
-(void)setupBizBoardTemplate {
    self.bizBoardTemplate = [[BizBoardTemplate alloc] init];
    self.bizBoardTemplate.translatesAutoresizingMaskIntoConstraints = NO;
    CGFloat adViewHeight = [self.bizBoardTemplate adHeightWithWidth:self.view.frame.size.width];
    // 비즈보드 비율에 맞는 높이 계산
    [self.view addSubview:self.bizBoardTemplate];
    [NSLayoutConstraint activateConstraints:@[
        [self.bizBoardTemplate.topAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.topAnchor],
        [self.bizBoardTemplate.leadingAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.leadingAnchor],
        [self.bizBoardTemplate.trailingAnchor constraintEqualToAnchor:self.view.safeAreaLayoutGuide.trailingAnchor],
        [self.bizBoardTemplate.heightAnchor constraintEqualToConstant:adViewHeight]
    ]];
}

#pragma mark - AdFitNativeAdLoaderDelegate
// ... 생략 ...
@end
```

`adHeight(width: CGFloat)` 메서드는 입력한 width를 기준으로 비즈보드 비율에 맞는 height를 계산합니다.
width와 height의 비율이 비즈보드 이미지의 비율과 맞지 않으면 광고가 정상 노출되지 않을 수 있으므로 반드시 이 메서드를 활용하세요.

### 3. 데이터 연결하기

뷰 구성이 끝나고, 광고 응답이 정상적으로 내려왔을 경우 데이터를 뷰에 연결합니다.

#### Swift
```swift
// MARK: AdFitNativeAdLoaderDelegate
func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
    nativeAd.bind(bizBoardTemplate)
}

func nativeAdLoaderDidFailToReceiveAd(_ nativeAdLoader: AdFitNativeAdLoader, error: Error) {
    print("광고 응답 에러: \(error.localizedDescription)")
    bizBoardTemplate.isHidden = true
}
```

#### Objective-C
```objc
#pragma mark - AdFitNativeAdLoaderDelegate
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    [nativeAd bind:self.bizBoardTemplate];
}
- (void)nativeAdLoaderDidFailToReceiveAd:(AdFitNativeAdLoader *)nativeAdLoader error:(NSError *)error {
    NSLog(@"광고 응답 에러: %@", [error localizedDescription]);
    [self.bizBoardTemplate setHidden:YES];
}
```

---

## UIKit 연동 (UITableViewCell 형태)

### 1. UITableView에 BizBoardCell 등록

#### Swift
```swift
import UIKit
import AdFitSDK

final class MyViewController: UIViewController {
    private let adCellIdentifier = "BizBoardListFeedTableViewCell"
    private let tableView = UITableView()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.register(BizBoardCell.self, forCellReuseIdentifier: adCellIdentifier)
    }
}
```

#### Objective-C
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.tableView registerClass:[BizBoardCell self] forCellReuseIdentifier:adCellIdentifier];
}
```

### 2. AdFitNativeAdLoaderDelegate 구현

광고 응답으로 받은 AdFitNativeAd를 property로 보관하고, 응답 시 tableView를 reload 합니다.

#### Swift
```swift
private var nativeAd: AdFitNativeAd?
private var nativeAdLoader: AdFitNativeAdLoader?

// MARK: - AdFitNativeAdLoaderDelegate
func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
    self.nativeAd = nativeAd
    tableView.reloadData()
}

func nativeAdLoaderDidFailToReceiveAd(_ nativeAdLoader: AdFitNativeAdLoader, error: Error) {
    tableView.reloadData()
    print("광고 응답 에러: \(error) (\(error.localizedDescription))")
}
```

#### Objective-C
```objc
#pragma mark - AdFitNativeAdLoaderDelegate
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    self.nativeAd = nativeAd;
    [self.tableView reloadData];
}

- (void)nativeAdLoaderDidFailToReceiveAd:(AdFitNativeAdLoader *)nativeAdLoader error:(NSError *)error {
    [self.tableView reloadData];
    NSLog(@"광고 응답 에러: %@", [error localizedDescription]);
}
```

### 3. UITableViewDataSource 구현

cellForRowAt에서 `adHeight(width:)` 결과를 저장합니다.

#### Swift
```swift
private var adHeight: CGFloat = 0

// MARK: UITableViewDataSource
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    if let cell = tableView.dequeueReusableCell(withIdentifier: adCellIdentifier, for: indexPath) as? BizBoardCell {
        adHeight = cell.adHeight(width: view.bounds.width)
        return cell
    }
    // ... service 구현 ...
}
```

#### Objective-C
```objc
#pragma mark - UITableViewDataSource
- (nonnull UITableViewCell *)tableView:(nonnull UITableView *)tableView cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath {
    BizBoardCell *cell = [tableView dequeueReusableCellWithIdentifier:adCellIdentifier forIndexPath:indexPath];
    if (cell) {
        self.adHeight = [cell adHeightWithWidth:self.view.bounds.size.width];
        return cell;
    }
    // ... service 구현 ...
}
```

### 4. UITableViewDelegate 구현

저장한 adHeight를 활용하여 cell 높이를 설정하고, willDisplay 시점에 데이터를 맵핑합니다.

#### Swift
```swift
// MARK: UITableViewDelegate
func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
    if let cell = cell as? BizBoardCell {
        if let nativeAd = nativeAd {
            nativeAd.bind(cell)
        }
    }
}

func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
    return adHeight
}

func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
    return adHeight
}
```

#### Objective-C
```objc
#pragma mark - UITableViewDelegate
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
    BizBoardCell *bizCell = (BizBoardCell *)cell;
    [self.nativeAd bind:bizCell];
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return self.adHeight;
}

- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath {
    return self.adHeight;
}
```

---

## 에러 코드

광고 수신에 실패한 경우, nativeAdLoaderDidFailToReceiveAd delegate 메서드를 통해 에러 객체를 전달받을 수 있습니다.

| 코드 | 메시지 | 설명 |
|:----:|--------|------|
| 1 | clientId property is nil | 배너광고만 해당. 비즈보드 광고는 해당 없음 |
| 2 | no ad to show | 노출 가능한 광고가 없는 경우. 잠시 후 재요청 |
| 3 | invalid ad received | 유효하지 않은 광고. AdFit에 문의 |
| 5 | attempted to load ad too frequently | 과도하게 짧은 간격으로 재요청한 경우 |
| 6 | HTTP failed | HTTP 에러. 반복 시 AdFit에 문의 |

---

## Advance Setting

### 광고의 갱신 시점
- AdFitNativeAdLoader와 AdFitNativeAd 인스턴스는 뷰의 라이프사이클 동안 유지됩니다.
- 보통 뷰 전환 시마다 갱신하는 것이 매출 측면에서 유리합니다.
- viewWillAppear에서 AdFitNativeAdLoader를 초기화하여 다시 loadAd()를 호출합니다.

#### Swift
```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    nativeAdLoader = AdFitNativeAdLoader(clientId: "{clientId}")
    nativeAdLoader?.delegate = self
    nativeAdLoader?.loadAd()
}
```

#### Objective-C
```objc
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];

    self.nativeAdLoader = [[AdFitNativeAdLoader alloc] initWithClientId:@"{clientId}" count:1 userObject:nil contentObject:nil];
    self.nativeAdLoader.delegate = self;
    [self.nativeAdLoader loadAdWithKeyword:nil];
}
```

### 비즈보드 Margin 설정
- 기본값: `UIEdgeInsets(top: 0, left: 16, bottom: 8, right: 16)`
- 높이 계산 로직에서 margin 수치가 제외되므로, **반드시 `adHeight(width:)` 호출 전에 margin을 설정**해주세요.

#### Swift
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // ...
    bizBoardTemplate.bgViewleftMargin = 56
    bizBoardTemplate.bgViewRightMargin = 16
    bizBoardTemplate.bgViewTopMargin = 10
    bizBoardTemplate.bgViewBottomMargin = 10
    // ...
}
```

#### Objective-C
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    bizBoardTemplate.bgViewleftMargin = 56;
    bizBoardTemplate.bgViewRightMargin = 16;
    bizBoardTemplate.bgViewTopMargin = 10;
    bizBoardTemplate.bgViewBottomMargin = 10;
}
```

### 광고 클릭 이벤트 감지
AdFitNativeAdDelegate 구현을 통해 광고 클릭 이벤트를 감지할 수 있습니다.

#### Swift
```swift
final class MyViewController: UIViewController, AdFitNativeAdLoaderDelegate, AdFitNativeAdDelegate {

    // MARK: AdFitNativeAdLoaderDelegate
    func nativeAdLoaderDidReceiveAd(_ nativeAd: AdFitNativeAd) {
        nativeAd.delegate = self
        nativeAd.bind(bizBoardTemplate)
    }

    // MARK: - AdFitNativeAdDelegate
    func nativeAdDidClickAd(_ nativeAd: AdFitNativeAd) {
        // 광고 클릭 시, 호출됨.
    }
}
```

#### Objective-C
```objc
#pragma mark - AdFitNativeAdLoaderDelegate
- (void)nativeAdLoaderDidReceiveAd:(AdFitNativeAd *)nativeAd {
    nativeAd.delegate = self;
    [nativeAd bind:self.bizBoardTemplate];
}

#pragma mark - AdFitNativeAdDelegate
- (void)nativeAdDidClickAd:(AdFitNativeAd *)nativeAd {
    // 광고 클릭 시, 호출됨.
}
```

---

## 코드 생성 규칙

사용자에게 코드를 생성할 때 아래 규칙을 따릅니다:

1. `{clientId}`는 사용자가 제공한 광고 단위 ID로 치환합니다.
2. 레이아웃 형태(view/cell)에 따라 적절한 코드를 생성합니다. 기본값은 단독 뷰(view)입니다.
3. Auto Layout을 우선적으로 사용하여 레이아웃을 구성합니다.
4. `view.safeAreaLayoutGuide`를 기준으로 제약조건을 설정하여 Safe Area를 준수합니다.
5. `adHeight(width:)` 메서드를 반드시 사용하여 비즈보드 비율에 맞는 높이를 계산합니다.
6. Margin 설정이 있는 경우 `adHeight(width:)` 호출 전에 설정합니다.
7. delegate 메서드 구현을 포함합니다.
8. 사용자의 클래스명이나 뷰 이름이 주어진 경우 해당 이름을 사용합니다.
