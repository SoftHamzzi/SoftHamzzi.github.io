---
title:  "[UE5] 추출 슈터 1-4. 스폰 시스템과 DataAsset 설계"
excerpt: "Enhanced Input Step 6 to 7"

categories:
  - Portfolio
tags:
  - [UE5, C++]

toc: true
toc_sticky: true

mermaid: true
 
date: 2026-02-08
last_modified_at: 2025-02-08
---

📌 EmploymentProj의 GameplayFramework에 대해 알아보는 포스트  
🚨 완성된 포스트가 아니므로, 지속적으로 수정됩니다!  
[👾 깃허브](https://github.com/SoftHamzzi/UE5-EmploymentProj)  
[📋 기획](https://github.com/SoftHamzzi/UE5-EmploymentProj/blob/main/DOCS/GAME.md)
{: .notice--warning}

## 개요
> 데이터 주도 설계의 이점
- 코드 수정없이 데이터만 추가하면 되므로, 확장성에 좋다.
- 데이터만 수정하면 되므로, 유지보수하기 좋다.
- 동일한 로직을 다양한 데이터에 적용 가능하므로, 재사용 가능하다.
- 코드는 로직만, 데이터는 값/설정 담당이므로 역할 분담이 쉬워진다.
- Asset Manager를 통해 필요할 때만 언로드/로드 가능하다.
  - 즉, 수백개의 아이템을 효율적으로 관리 가능하다.

## 스폰 시스템
- 스폰 포인트 구조
  - 월드에 PlayerStart 액터를 여러개 배치하였다.

- 서버 권한 스폰 로직
  - ChoosePlayerStart 함수를 오버라이드하여,
  ChoosePlayerStart_Implementation을 구현한다.
  - 사용되지 않은 PlayerStart 액터 하나를 랜덤으로 가져와,  
  플레이어를 소환하고, 사용 처리한다.

## DataAsset 활용
- EPWeaponData
  - 무기 이름, 대미지, 연사 속도, 반동, 탄퍼짐, 탄약 수를 가진다.
- EPItemData
  - 아이템 이름, 설명, 등급, 판매 가격, 퀘스트 아이템 여부, 인벤토리 슬롯 차지 수를 가진다.
- UPrimaryDataAsset 선택 이유
  - 타르코프류 게임의 특성 상, 수백가지의 아이템이 존재한다.
  - UDataAsset은 한번 참조되면, 일반적으로 메모리에 계속 올라가있다.
  - UPrimaryDataAsset은 AssetManager를 통해, 필요할 때만 사용할 수 있다.

- [DataAsset과 AssetManager](https://redchiken.tistory.com/358)

## 확장성
- 새 무기/아이템 추가 방법
  - 에디터의 콘텐츠 드로어에서 데이터 에셋을 추가한다.
  - 클래스로 EPItemData/EPWeaponData를 선택한다.
  - 값을 채워넣는다.

## 다음 편 예고
→ Replication 심화