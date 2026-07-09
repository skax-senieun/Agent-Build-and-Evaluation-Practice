# SAP BTP 비용 산정 보고서

- 작성일: 2026-07-08
- 대상 시스템: 비용정산(Billing Settlement) BTP Application
- CSP: AWS
- 예상 사용자 수: 전체 4,000명 / 평일 DAU 500명
- 적용 환율: 1 EUR = 1,720 KRW (기준일: 2026-07-08, 가정값 — 실제 정산 시 최신 환율로 재확인 필요)

## 1. 요약

AWS 리전 기반 SAP BTP 비용정산 애플리케이션(전체 사용자 4,000명, 평일 DAU 500명)에 대해 Cloud Foundry Runtime + SAP HANA Cloud + Object Store 표준 구성으로 산정한 결과, **Pay-As-You-Go(PAYG) 기준 월 예상 비용은 약 2,650.56 EUR(약 456만 원), 연 예상 비용은 약 31,806.72 EUR(약 5,471만 원)**입니다. 장기 계약(BTPEA) 전환 시 월 약 2,047.68 EUR(약 352만 원), 연 약 24,572.16 EUR(약 4,226만 원)로 약 23% 절감이 가능합니다.

## 2. 리소스별 예상 사용량

| 리소스 ID | 리소스 유형 | 스펙(Tier) | 단위 스펙 | 예상 월간 사용량 |
|---|---|---|---|---|
| cf.runtime.standard | application_runtime | standard | GB 메모리 | 16 GB |
| hana.cloud.memory | hana_db | standard | GB 메모리 | 32 GB |
| objectstore.standard | object_store | standard | 100GB 블록 | 200 GB (2블록) |

**사이징 가정**
- Cloud Foundry Runtime: 기본 할당 4GB + DAU 500명 × 0.024GB/DAU(동시접속률 20% 가정, 세션당 0.24GB) ≈ 14GB → 여유분 포함 16GB로 산정
- HANA Cloud: 비용정산 트랜잭션 데이터 처리를 위한 최소 권장 메모리 32GB 적용(전체 사용자 4,000명 기준 마스터데이터/집계 처리 고려)
- Object Store: 전체 사용자 4,000명 × 평균 50MB(정산 내역/첨부 파일) ≈ 200GB

## 3. 리소스별/스펙별 예상 비용 (Pay-As-You-Go 기준)

| 리소스 ID | 스펙(Tier) | 과금 단위 | 단가(EUR) | 월 비용(EUR) | 연 비용(EUR) | 월 비용(KRW) | 연 비용(KRW) | 비용 비율(%) |
|---|---|---|---|---|---|---|---|---|
| cf.runtime.standard | standard | GB-month | 110.50 | 1,768.00 | 21,216.00 | 3,041,000 | 36,492,000 | 66.7% |
| hana.cloud.memory | standard | GB-month | 26.83 | 858.56 | 10,302.72 | 1,476,723 | 17,720,678 | 32.4% |
| objectstore.standard | standard | 100GB-month | 12.00 | 24.00 | 288.00 | 41,280 | 495,360 | 0.9% |

**전체 비용 합계 (PAYG)**
- 월 비용 합계: 2,650.56 EUR (4,558,963 KRW)
- 연 비용 합계: 31,806.72 EUR (54,707,558 KRW)

**참고: BTPEA(엔터프라이즈 계약) 기준 단가 적용 시**

| 리소스 ID | 단가(EUR, BTPEA) | 월 비용(EUR) | 연 비용(EUR) |
|---|---|---|---|
| cf.runtime.standard | 85.00 | 1,360.00 | 16,320.00 |
| hana.cloud.memory | 20.74 | 663.68 | 7,964.16 |
| objectstore.standard | 12.00(동일) | 24.00 | 288.00 |
| **합계** | | **2,047.68 (3,522,010 KRW)** | **24,572.16 (42,264,115 KRW)** |

## 4. 최적 스펙 조합 추천

| 순위 | 조합 설명 | 월 비용(EUR) | 연 비용(EUR) | 월 비용(KRW) | 연 비용(KRW) | 비용 대비 성능 | 비고 |
|---|---|---|---|---|---|---|---|
| 1 | BTPEA 계약 + 표준 구성(App 16GB / HANA 32GB / Object 200GB) | 2,047.68 | 24,572.16 | 3,522,010 | 42,264,115 | 상 | 1년 이상 운영 예정 시 PAYG 대비 약 23% 절감 |
| 2 | PAYG + 표준 구성 | 2,650.56 | 31,806.72 | 4,558,963 | 54,707,558 | 중 | 초기 검증/단기 운영, 계약 없이 즉시 시작 가능 |
| 3 | PAYG + App Runtime 축소(8GB, 동시접속률 낮게 재가정) | 1,766.56 | 21,198.72 | 3,038,483 | 36,461,798 | 중상 | 동시접속률이 20%보다 낮을 경우 적용 가능, 성능 여유 감소 리스크 |

**추천안**: 1년 이상 안정적으로 운영할 계획이라면 **조합 1(BTPEA)**을 권장합니다. 단기 PoC나 사용량 변동이 큰 초기 단계라면 조합 2(PAYG)로 시작 후 전환을 검토하세요.

## 5. 민감도 분석

| 파라미터 | 변화 범위 | 비용 영향(상/중/하) | 설명 |
|---|---|---|---|
| DAU(동시접속률) | 20% → 30% | 중 | App Runtime 메모리 증가 → 월 비용 약 5~10% 증가 예상 |
| 전체 사용자 수 증가 | 4,000명 → 8,000명 | 중 | Object Store 용량 2배 증가하나 전체 비용 비율 낮아(0.9%) 영향은 미미 |
| HANA 메모리 티어 | 32GB → 64GB | 상 | HANA가 전체 비용의 약 32% 차지, 메모리 2배 시 해당 비용도 약 2배 증가 |
| 계약 모델(PAYG↔BTPEA) | - | 상 | 약 23% 비용 차이, 장기 운영 여부에 따라 결정 |

## 6. 가정 목록

- 입력 방식: 대화형(방식 B)
- 전체 사용자 수: 4,000명
- 평일 일간 활성 사용자 수(Weekday DAU): 500명
- 동시접속률: 20% (DAU 기준 동시 세션 100명 가정)
- 평균 트랜잭션/세션 메모리: 0.24GB/세션
- Object Store 사용량 가정: 사용자당 평균 50MB
- 기타 가정: HANA Cloud 최소 권장 메모리 32GB, 계약 모델은 AWS 리전 기준 PAYG/BTPEA 요율 모두 병기

## 7. 참고

- 비용 기준 출처: SAP Discovery Center (기준 통화: EUR)
  - Cloud Foundry Runtime: https://discovery-center.cloud.sap/serviceCatalog/cloud-foundry-runtime?region=all&tab=service_plan
  - SAP HANA Cloud: https://discovery-center.cloud.sap/serviceCatalog/sap-hana-cloud?region=all&tab=service_plan
  - Object Store: https://discovery-center.cloud.sap/serviceCatalog/object-store?service_plan=standard&region=all&commercialModel=btpea&tab=service_plan
- 적용 환율: 1 EUR = 1,720 KRW (기준일: 2026-07-08, 출처: 시장 환율 참고 평균값(Wise/Investing.com 등), 가정값 — 정확한 산정 시 최신 환율 확인 권장)
- 로컬 캐시 파일: /data/sap_discovery_costs.json
- 캐시 마지막 업데이트: 2026-07-08 (SAP Discovery Center 검증 완료)
- 보고서 생성 기준 데이터 타임스탬프: 2026-07-08
