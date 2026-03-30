# adfit-plugin-ios

AdFit SDK iOS 연동을 위한 Claude Code 플러그인입니다.

## 설치

### 1. 마켓플레이스 연동

플러그인을 설치하려면 먼저 마켓플레이스 연동이 필요합니다. 연동 방법은 [sdk-plugins 마켓플레이스](https://github.com/adfit/sdk-plugins)를 참고하세요.

### 2. 플러그인 설치

마켓플레이스 연동 후 아래 명령어를 실행하세요.

```bash
/plugin install adfit-plugin-ios@sdk-plugins
```

## 사용 가능한 스킬

| 스킬 | 설명 |
|---|---|
| `/sdk-setup` | AdFit SDK 기본 프로젝트 설정 (ATT, SKAdNetwork, ATS 등) |
| `/banner-ad` | 배너 광고 연동 코드 생성 (UIKit, SwiftUI) |
| `/native-ad` | 네이티브 광고 연동 코드 생성 (UIKit) |
| `/popup-ad` | 팝업 광고 연동 코드 생성 (UIKit) |
| `/hybrid-ad` | 하이브리드 광고 연동 코드 생성 (WKWebView) |
| `/bizboard-template` | 비즈보드 템플릿 광고 연동 코드 생성 (UIKit) |
| `/advanced-features` | 고급 기능 연동 (타겟팅 데이터 입력, Opt out 설정) |
