# SAP BTP 비용 산정 보고서

- 작성일: {{report_date}}
- 대상 시스템: {{system_summary}}
- CSP: {{csp}}
- 예상 사용자 수: {{expected_users}}
- 적용 환율: 1 EUR = {{exchange_rate}} KRW (기준일: {{exchange_rate_date}})

## 1. 요약

{{summary_sentence}}

## 2. 리소스별 예상 사용량

| 리소스 ID | 리소스 유형 | 스펙(Tier) | 단위 스펙 | 예상 월간 사용량 |
|---|---|---|---|---|
| {{resource_id}} | {{resource_type}} | {{spec_tier}} | {{unit_spec}} | {{monthly_usage}} |

## 3. 리소스별/스펙별 예상 비용

| 리소스 ID | 스펙(Tier) | 과금 단위 | 단가(EUR) | 월 비용(EUR) | 연 비용(EUR) | 월 비용(KRW) | 연 비용(KRW) | 비용 비율(%) |
|---|---|---|---|---|---|---|---|---|
| {{resource_id}} | {{spec_tier}} | {{billing_unit}} | {{unit_price_eur}} | {{monthly_cost_eur}} | {{annual_cost_eur}} | {{monthly_cost_krw}} | {{annual_cost_krw}} | {{cost_ratio}} |

**전체 비용 합계**
- 월 비용 합계: {{total_monthly_cost_eur}} EUR ({{total_monthly_cost_krw}} KRW)
- 연 비용 합계: {{total_annual_cost_eur}} EUR ({{total_annual_cost_krw}} KRW)

## 4. 최적 스펙 조합 추천

| 순위 | 조합 설명 | 월 비용(EUR) | 연 비용(EUR) | 월 비용(KRW) | 연 비용(KRW) | 비용 대비 성능 | 비고 |
|---|---|---|---|---|---|---|---|
| 1 | {{combo_1_desc}} | {{combo_1_monthly_eur}} | {{combo_1_annual_eur}} | {{combo_1_monthly_krw}} | {{combo_1_annual_krw}} | {{combo_1_score}} | {{combo_1_note}} |
| 2 | {{combo_2_desc}} | {{combo_2_monthly_eur}} | {{combo_2_annual_eur}} | {{combo_2_monthly_krw}} | {{combo_2_annual_krw}} | {{combo_2_score}} | {{combo_2_note}} |
| 3 | {{combo_3_desc}} | {{combo_3_monthly_eur}} | {{combo_3_annual_eur}} | {{combo_3_monthly_krw}} | {{combo_3_annual_krw}} | {{combo_3_score}} | {{combo_3_note}} |

**추천안**: {{recommended_combo}}

## 5. 민감도 분석

| 파라미터 | 변화 범위 | 비용 영향(상/중/하) | 설명 |
|---|---|---|---|
| {{param_1}} | {{param_1_range}} | {{param_1_sensitivity}} | {{param_1_note}} |

## 6. 가정 목록

- 입력 방식: {{input_method}} (CSV / 대화형)
- 전체 사용자 수: {{total_users}}
- 평일 일간 활성 사용자 수(Weekday DAU): {{weekday_dau}}
- 동시접속률: {{concurrency_rate}}
- 평균 트랜잭션/세션: {{avg_transaction}}
- 기타 가정: {{other_assumptions}}

## 7. 참고

- 비용 기준 출처: SAP Discovery Center (기준 통화: EUR)
- 적용 환율: 1 EUR = {{exchange_rate}} KRW (기준일: {{exchange_rate_date}}, 출처: {{exchange_rate_source}})
- 참고 링크: {{source_link}}
- 로컬 캐시 파일: /data/sap_discovery_costs.json
- 캐시 마지막 업데이트: {{cache_last_updated}}
- 보고서 생성 기준 데이터 타임스탬프: {{data_timestamp}}
