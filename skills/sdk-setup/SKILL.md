---
name: sdk-setup
description: AdFit SDK 연동을 위한 기본 프로젝트 설정을 수행합니다. ATT 추적 권한 요청(필수), SKAdNetwork ID 추가(선택), ATS 처리(선택)를 지원합니다.
user-invocable: true
argument-hint: [UIKit|SwiftUI]
---

## Overview
AdFit SDK를 연동하려는 외부 매체 개발자를 위한 프로젝트 초기 설정 스킬입니다.
사용자의 프로젝트 환경(UIKit / SwiftUI)에 맞게 필수/선택 설정을 수행합니다.

## 설정 항목

| 항목 | 필수 여부 | 설명 |
|------|-----------|------|
| ATT 추적 권한 요청 | **필수** | App Tracking Transparency 프레임워크를 사용한 추적 권한 요청 코드 생성 및 Info.plist 설정 |
| SKAdNetwork ID 추가 | 선택 | SKAdNetwork 지원을 위한 Info.plist 설정 |
| ATS 처리 | 선택 | HTTP 방식의 광고 랜딩을 지원하기 위한 Info.plist 설정 |

## 실행 절차

1. 사용자에게 아래 정보를 확인합니다. (인자로 전달된 경우 생략)
   - **UI 프레임워크**: UIKit 또는 SwiftUI
   - UIKit인 경우, 프로젝트에 `SceneDelegate.swift` 파일 존재 여부를 자동 탐색하여 SceneDelegate/레거시 패턴을 판단합니다.
   - **SKAdNetwork ID 추가 여부**: 예/아니오 (선택, 기본값: 예)
   - **ATS 처리 여부**: 예/아니오 (선택, 기본값: 아니오)

2. 확인된 정보를 바탕으로 [guide.md](guide.md)를 참고하여 설정을 수행합니다.

3. 각 설정 항목에 대해 수행한 내용을 간략히 설명합니다.
