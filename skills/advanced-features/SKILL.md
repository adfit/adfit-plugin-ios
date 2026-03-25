---
name: advanced-features
description: AdFit SDK의 고급 기능을 연동하는 코드를 생성합니다. 타겟팅 향상을 위한 지면 데이터 입력(ContentObject, UserObject)과 Opt out 버튼 설정을 지원합니다.
user-invocable: true
argument-hint: [surface-data|opt-out]
---

## Overview
AdFit SDK의 고급 기능을 연동하려는 외부 개발자를 위한 스킬입니다.
네이티브 광고의 타겟팅 향상을 위한 지면 데이터 입력과 Opt out 버튼 설정 코드를 생성합니다.

## 실행 절차

1. 사용자에게 아래 정보를 확인합니다. (인자로 전달된 경우 생략)
   - **기능 종류**: 지면 데이터 입력(surface-data) 또는 Opt out 버튼 설정(opt-out)
   - **clientId**: 발급받은 광고 단위 ID

2. 확인된 정보를 바탕으로 [guide.md](guide.md)를 참고하여 연동 코드를 생성합니다.

3. 생성된 코드에 대해 간략한 설명을 제공합니다.
