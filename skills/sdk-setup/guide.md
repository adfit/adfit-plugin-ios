# AdFit SDK 기본 설정 가이드

## 1. ATT 추적 권한 요청 (필수)

ATT(App Tracking Transparency) 프레임워크를 사용하여 사용자에게 앱의 추적 권한을 요청합니다.
이 설정은 **반드시** 수행해야 합니다.

### 1-1. ATT 팝업 실행 코드

#### SwiftUI

사용자의 App 진입점 파일(예: `@main struct ...App`)에 아래 코드를 추가합니다.
기존 App 구조체가 있다면 해당 파일을 수정하고, 없다면 새로 생성합니다.

```swift
import SwiftUI
import AppTrackingTransparency

@main
struct {AppName}App: App {
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
                .onChange(of: scenePhase, perform: { value in
                    if value == .active {
                        requestTrackingAuthorization()
                    }
                })
        }
    }

    private func requestTrackingAuthorization() {
        Task {
            _ = await ATTrackingManager.requestTrackingAuthorization()
            print("ATT 요청이 완료되었습니다.")
        }
    }
}
```

**치환 규칙:**
- `{AppName}`: 사용자의 앱 이름 (프로젝트에서 추론 또는 사용자에게 확인)

#### UIKit (SceneDelegate 사용 — 기본)

iOS 13 이후 Scene 기반 라이프사이클을 채택한 앱에서는 `AppDelegate.applicationDidBecomeActive`가 호출되지 않습니다.
대신 `SceneDelegate.sceneDidBecomeActive(_:)`에서 ATT를 요청해야 합니다.

사용자의 SceneDelegate 파일에 아래 코드를 추가합니다.

```swift
import UIKit
import AppTrackingTransparency

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let _ = (scene as? UIWindowScene) else { return }
    }

    func sceneDidBecomeActive(_ scene: UIScene) {
        requestTrackingAuthorization()
    }

    private func requestTrackingAuthorization() {
        Task {
            _ = await ATTrackingManager.requestTrackingAuthorization()
            print("ATT 요청이 완료되었습니다.")
        }
    }
}
```

#### UIKit (SceneDelegate 미사용 — 레거시)

SceneDelegate를 사용하지 않는 레거시 앱에서는 `AppDelegate.applicationDidBecomeActive(_:)`에서 ATT를 요청합니다.

```swift
import UIKit
import AppTrackingTransparency

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        return true
    }

    func applicationDidBecomeActive(_ application: UIApplication) {
        requestTrackingAuthorization()
    }

    private func requestTrackingAuthorization() {
        Task {
            _ = await ATTrackingManager.requestTrackingAuthorization()
            print("ATT 요청이 완료되었습니다.")
        }
    }
}
```

**판단 기준:** 사용자의 프로젝트에 `SceneDelegate.swift` 파일이 있거나 `Info.plist`에 `UIApplicationSceneManifest` 키가 있으면 SceneDelegate 패턴을 사용합니다.

### 1-2. Info.plist 설정

Info.plist에 다음 항목을 추가합니다.

```xml
<key>NSUserTrackingUsageDescription</key>
<string>맞춤형 광고 추천을 위해 iOS 기기의 광고식별자를 수집합니다.</string>
```

**구현 규칙:**
- Info.plist 파일을 직접 편집하여 `NSUserTrackingUsageDescription` 키를 추가합니다.
- 이미 해당 키가 존재하면 값만 확인하고, 없을 경우에만 추가합니다.
- 문구는 위 기본값을 사용하되, 사용자가 커스텀 문구를 원할 경우 해당 문구로 대체합니다.

---

## 2. SKAdNetwork ID 추가 (선택)

SKAdNetwork를 지원하면 서비스에 더 많은 광고를 내보낼 수 있고, 더 많은 수익을 창출할 수 있습니다.

### Info.plist 설정

Info.plist 파일에 다음 항목을 추가합니다.

```xml
<key>SKAdNetworkItems</key>
<array>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>9t245vhmpl.skadnetwork</string>
    </dict>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>v72qych5uu.skadnetwork</string>
    </dict>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>x8uqf25wch.skadnetwork</string>
    </dict>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>8s468mfl3y.skadnetwork</string>
    </dict>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>54NZKQM89Y.skadnetwork</string>
    </dict>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>t6d3zquu66.skadnetwork</string>
    </dict>
</array>
```

**구현 규칙:**
- Info.plist 파일을 직접 편집하여 `SKAdNetworkItems` 키를 추가합니다.
- 이미 `SKAdNetworkItems`가 존재하면, 기존 항목을 유지하면서 누락된 ID만 추가합니다.
- 6개의 SKAdNetwork ID를 모두 포함해야 합니다.

---

## 3. ATS(App Transport Security) 처리 (선택)

iOS 9부터 ATS(App Transport Security) 기능이 기본적으로 활성화되어 있으며, 암호화된 HTTPS 방식의 통신만 허용됩니다.
AdFit SDK는 ATS 활성화 상태에서도 정상적으로 동작하지만, 광고를 통해 노출되는 광고주 페이지는 HTTPS 방식을 지원하지 않을 수도 있습니다.

### Info.plist 설정

Info.plist 파일에 다음 항목을 추가합니다.

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```

**구현 규칙:**
- Info.plist 파일을 직접 편집하여 `NSAppTransportSecurity` 키를 추가합니다.
- 이미 `NSAppTransportSecurity`가 존재하면, `NSAllowsArbitraryLoads` 값만 확인/수정합니다.
- 사용자에게 이 설정이 앱 전체의 HTTP 통신을 허용한다는 점을 안내합니다.

---

## 실행 흐름 요약

1. **사용자 입력 확인**
   - UI 프레임워크 (UIKit / SwiftUI) — 인자로 전달되지 않은 경우 확인
   - SKAdNetwork ID 추가 여부 — "SKAdNetwork ID를 Info.plist에 추가할까요? (더 많은 광고 수익을 위해 권장됩니다)"
   - ATS 처리 여부 — "ATS 예외 처리를 추가할까요? (HTTP 방식의 광고 랜딩 페이지를 지원합니다)"

2. **ATT 설정 수행 (필수)**
   - UI 프레임워크에 맞는 ATT 팝업 코드를 기존 App/AppDelegate 파일에 추가
   - Info.plist에 `NSUserTrackingUsageDescription` 추가

3. **SKAdNetwork ID 추가 (사용자가 동의한 경우)**
   - Info.plist에 `SKAdNetworkItems` 추가

4. **ATS 처리 (사용자가 동의한 경우)**
   - Info.plist에 `NSAppTransportSecurity` 추가

5. **결과 안내**
   - 수행한 설정 항목을 요약하여 사용자에게 안내
