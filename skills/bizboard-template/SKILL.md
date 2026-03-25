---
name: bizboard-template
description: AdFit SDK의 비즈보드 템플릿 광고를 연동하는 코드를 생성합니다. UIKit 환경에서 BizBoardTemplate을 사용한 네이티브 형식의 배너 광고 연동 코드를 작성해줍니다.
user-invocable: true
argument-hint: [clientId] [view|cell]
---

## Overview
AdFit SDK를 사용하여 비즈보드 템플릿 광고를 연동하려는 외부 개발자를 위한 스킬입니다.
UIKit 환경에서 단독 뷰(view) 또는 UITableViewCell(cell) 형태의 비즈보드 템플릿 연동 코드를 생성합니다.

## 실행 절차

1. 사용자에게 아래 정보를 확인합니다. (인자로 전달된 경우 생략)
   - **clientId**: 발급받은 광고 단위 ID
   - **레이아웃 형태**: 단독 뷰(view, 기본값) 또는 UITableViewCell(cell)

2. 확인된 정보를 바탕으로 [guide.md](guide.md)를 참고하여 연동 코드를 생성합니다.

3. 생성된 코드에 대해 간략한 설명을 제공합니다.
