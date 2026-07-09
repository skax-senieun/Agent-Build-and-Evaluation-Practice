# SAP BTP 비용 산정 보고서

- 작성일: 2026-07-09
- 대상 시스템: 직원 법인카드 비용정산 시스템
- CSP: SAP BTP
- 예상 사용자 수: 4,000명 (입력 CSV: /input/BTP Spec.txt)

## 1. 요약

CSV에 명시된 4개 애플리케이션(portal, service, file uploader, logger) + HANA Cloud + Object Store 구성 기준으로 산정했습니다. 사용자 대면 핵심 앱(portal, service)은 이중화(HA)를, 보조 기능(file uploader, logger)은 필요 최소 스펙으로 구성한 추천 조합 기준 월 비용 약 **$1,324**, 연 비용 약 **$15,892**입니다.

> ⚠️ CSV의 `unit_spec`이 범위값(예: HANA `vCPU:2~4, memory_GB:2~4, storage_GB:120~200`)으로 기재되어 있어, 로컬 캐시(고정 스펙)와 정확히 일치하지 않습니다. 가장 근접한 캐시 티어로 매핑했으며, 실제 최소/최대 요구사항 확인 후 재산정이 필요합니다.

## 2. 리소스별 예상 사용량

| 리소스 ID | 리소스 유형 | 매핑된 스펙(Tier) | 단위 스펙 | 예상 월간 사용량 |
|---|---|---|---|---|
| portal | application_runtime | medium | vCPU 2, Mem 4GB, Storage 40GB | 인스턴스 2개(HA) x 730시간 |
| service | application_runtime | medium | vCPU 2, Mem 4GB, Storage 40GB | 인스턴스 2개(HA) x 730시간 |
| file uploader | application_runtime | small | vCPU 1, Mem 2GB, Storage 20GB | 인스턴스 2개(HA) x 730시간 |
| logger | application_runtime | small | vCPU 1, Mem 2GB, Storage 20GB | 인스턴스 1개(단일) x 730시간 |
| hana.cloud | hana_db | small(근접 매핑) | Mem 32GB, Storage 128GB | 상시 가동 |
| objectstore.standard | object_store | standard | 1GB 단위 | 100GB |

## 3. 리소스별/스펙별 예상 비용

| 리소스 ID | 스펙(Tier) | 과금 단위 | 단가 | 월 비용 | 연 비용 | 비용 비율(%) |
|---|---|---|---|---|---|---|
| portal x2 | medium | hour | $0.09/hr | $131.40 | $1,576.80 | 9.9% |
| service x2 | medium | hour | $0.09/hr | $131.40 | $1,576.80 | 9.9% |
| file uploader x2 | small | hour | $0.05/hr | $73.00 | $876.00 | 5.5% |
| logger x1 | small | hour | $0.05/hr | $36.50 | $438.00 | 2.8% |
| hana.cloud | small | month | $950.00/월 | $950.00 | $11,400.00 | 71.7% |
| objectstore.standard | standard | GB-month | $0.02/GB | $2.00 | $24.00 | 0.2% |

**전체 비용 합계**
- 월 비용 합계: **$1,324.30**
- 연 비용 합계: **$15,891.60**

## 4. 최적 스펙 조합 추천

| 순위 | 조합 설명 | 월 비용 | 연 비용 | 비용 대비 성능 | 비고 |
|---|---|---|---|---|---|
| 1 | 전체 앱 Small x1(무이중화) + HANA Small + ObjectStore | $1,098.00 | $13,176.00 | 중 | 최저비용, 단일장애점 존재(4개 앱 모두 이중화 없음) |
| 2 | portal/service Medium x2(HA) + file uploader Small x2(HA) + logger Small x1 + HANA Small + ObjectStore | $1,324.30 | $15,891.60 | 상 | 핵심 서비스만 이중화, 비용/안정성 균형. **추천** |
| 3 | 전체 앱 Medium x2(HA) + HANA Medium + ObjectStore | $2,327.60 | $27,931.20 | 중 | 전체 이중화 + HANA 상향, 향후 확장 대비용 |

**추천안**: 조합 2 — 사용자 대면 핵심 기능(portal, service)만 이중화하여 가용성을 확보하고, 보조 기능(file uploader, logger)은 필요 최소 구성으로 비용을 절감합니다. 재무 데이터를 다루는 시스템이므로 핵심 컴포넌트의 이중화는 유지를 권장합니다.

## 5. 민감도 분석

| 파라미터 | 변화 범위 | 비용 영향(상/중/하) | 설명 |
|---|---|---|---|
| HANA Cloud 티어(Small→Medium) | $950 → $1,800(월) | 상 | 전체 비용의 70~87%를 차지, 티어 선택이 총 비용을 좌우 |
| 앱 인스턴스 이중화 범위(4개 전체 vs 핵심 2개만) | +$109.5~+$226 (월) | 중 | 이중화 대상 범위에 따라 비용 차이 발생 |
| 앱 런타임 스펙(Small→Medium) | $36.5 → $65.7(월, 인스턴스당) | 중 | CSV의 memory_GB:1~4 범위 중 실제 요구치에 따라 변동 |
| Object Store 용량 증가 | 100GB→1TB | 하 | GB당 단가가 낮아 비용 영향 미미 |

## 6. 가정 목록

- 입력 방식: CSV (/input/BTP Spec.txt)
- 전체 사용자 수: 4,000명
- CSV에 평일 DAU(weekday_dau) 컬럼 없음 → 별도 확인 필요(이전 대화 기준 DAU 400명 가정 시에도 유사한 결과)
- HANA `unit_spec`이 범위값(vCPU:2~4, memory_GB:2~4, storage_GB:120~200)으로 기재되어 로컬 캐시와 불일치 → 스토리지 요구량(120~200GB) 기준 storage_GB 128GB인 hana.cloud.small에 근접 매핑(메모리 요구치는 캐시보다 낮으나 최소 티어 적용). 실제 SAP Discovery Center에서 해당 스펙 정확한 단가 확인 필요.
- Cloud Foundry 앱 4개(portal, service, file uploader, logger)를 개별 리소스로 분리 산정. CSV의 "instance 1~4개, memory_GB:1~4" 범위 중 portal/service는 medium(2vCPU/4GB), file uploader/logger는 small(1vCPU/2GB)로 가정.
- portal, service(사용자 대면 핵심 기능)는 이중화(x2), file uploader는 이중화(x2), logger는 단일(x1)로 가정
- Object Store 사용량: CSV 기재값(100GB) 그대로 사용
- 월 730시간(24시간 x 30.4일) 기준으로 시간당 과금 자원의 월 비용 환산

## 7. 참고

- 비용 기준 출처: SAP Discovery Center
- 참고 링크:
  - https://discovery-center.cloud.sap/example/cf-standard-small
  - https://discovery-center.cloud.sap/example/cf-standard-medium
  - https://discovery-center.cloud.sap/example/hana-cloud-small
  - https://discovery-center.cloud.sap/example/object-store-standard
- 로컬 캐시 파일: /data/sap_discovery_costs.json
- 캐시 마지막 업데이트: 2026-07-08
- 보고서 생성 기준 데이터 타임스탬프: 2026-07-09

> ⚠️ 캐시 데이터는 예시(placeholder) 성격이 강합니다. 실사용 전 SAP Discovery Center에서 최신 단가를 재검증하시기 바랍니다.
