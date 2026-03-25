# 팝업 광고 연동 가이드

## 사전 조건
- iOS 15.0 이상
- AdFitSDK가 Swift Package Manager(SPM)를 통해 프로젝트에 설치되어 있어야 합니다.

## 제한 사항
- 아이패드 환경은 지원하지 않습니다.
- 팝업 광고를 노출 시킬 때, 가로모드 상태일 경우 광고가 노출되지 않습니다.
- 팝업 광고가 노출된 상황에서, 가로모드로 전환할 수 없습니다.

---

## UIKit 연동

### 1. 광고 노출하기

#### Swift
```swift
import UIKit
import AdFitSDK

class MyViewController: UIViewController, SuperboardPopUpDelegate {
    private let adFitPopUp = SuperboardPopUp(adUnitId: "{clientId}")

    override func viewDidLoad() {
        super.viewDidLoad()
        adFitPopUp.delegate = self
        adFitPopUp.present()
        // adFitPopUp.present(self)
        // ㄴ present를 실행할 rootViewController를 직접 지정하고 싶을 때
    }
}
```

SuperboardPopUp 객체를 생성 후, `present()` 호출을 통해 광고를 노출합니다.
- keyWindow의 rootViewController를 기준으로, 광고뷰를 가지고 있는 UIViewController가 present 됩니다.
- 직접 노출되기를 원하는 UIViewController를 인자로 넣어줄 경우, 해당 UIViewController 상위에 노출됩니다.

### 2. 팝업 속성 설정

#### delegate
- 팝업 광고로부터 발생하는 특정 이벤트를 감지하기 위해, `delegate` 프로퍼티를 활용할 수 있습니다.
- `SuperboardPopUpDelegate` 프로토콜을 구현한 클래스의 객체를 `delegate` 프로퍼티에 할당해 주면 됩니다.

### 3. delegate 메서드 구현

`SuperboardPopUpDelegate` 프로토콜을 구현함으로써 광고로부터 발생하는 특정 이벤트를 감지할 수 있습니다.

#### Swift
```swift
// 광고 응답이 성공한 경우
func adViewDidReceiveAd() {
    print("didReceiveAd")
}

// 광고 응답이 실패한 경우
func adViewDidFailToReceiveAd(error: Error) {
    print("didFailToReceiveAd - error :\(error.localizedDescription)")
}

// 광고를 탭한 경우
func adViewDidClickAd() {
    print("didClickAd")
}

// 닫기 버튼을 탭한 경우
func adViewControllerClickClose() {
    print("adViewControllerWillDismiss")
}

// 오늘만 그만보기 버튼을 탭한 경우
func adViewControllerClickHideForToday() {
    print("adViewControllerClickHideForToday")
}
```

---

## 코드 생성 규칙

사용자에게 코드를 생성할 때 아래 규칙을 따릅니다:

1. `{clientId}`는 사용자가 제공한 광고 단위 ID로 치환합니다.
2. delegate 메서드 구현을 반드시 포함합니다.
3. 사용자의 클래스명이나 뷰 이름이 주어진 경우 해당 이름을 사용합니다.
4. rootViewController를 직접 지정하는 방법도 주석으로 안내합니다.
