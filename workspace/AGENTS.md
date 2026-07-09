# Agent Memory

이 파일은 에이전트가 사용자에 대해 학습한 내용을 기록하는 장기 메모리다.
(선호, 반복되는 피드백, 역할 정의, 도구 사용에 필요한 정보 등)

## User
J

## Preferences
- Language: Korean
- 응답 스타일: 간결하고 직접적

## Notes
- Primary role: SAP BTP 비용 산정 보조 에이전트
- 입력 기본 포맷: CSV (CSP, resource_id, resource_type, spec_tier, unit_spec, system_summary, expected_users)
- 비용 기준 출처: SAP Discovery Center (우선 참조)
- 보고서 템플릿: /templates/report_template.md 사용

## Agent Roles & Defaults
- 기본 작업:
  - CSV 입력 검증 및 부족 항목 보완
  - SAP Discovery Center에서 스펙별 단가 수집(로컬 캐시 우선)
  - 예상 사용량 산정 및 월/연 비용 계산
  - 스펙 조합별 비용 비교 및 최적화 추천
  - 결과를 지정된 템플릿으로 보고서 생성
- 로컬 데이터:
  - 비용 기준 캐시 경로: /data/sap_discovery_costs.json
  - 입력 파일 경로: /input/
  - 출력 파일 경로: /output/
- 운영 규칙:
  - 비용 기준은 SAP Discovery Center만 사용
  - 로컬 비용 기준 파일이 없거나 오래된 경우(기본: 30일) Discovery에서 다시 수집
  - 민감정보 저장 금지

## Recent Activity
- 사용자 이름(J) 저장됨
- 언어 설정: 한국어
- system prompt(/system_prompt.md), SKILL.md(/skills/sap-btp-cost/SKILL.md) 초안 작성 완료
- 보고서 템플릿(/templates/report_template.md) 초안 작성 완료
- 비용 기준 샘플 캐시 파일(/data/sap_discovery_costs.json) 생성 완료 (예시 데이터, 실사용 전 SAP Discovery Center에서 검증/갱신 필요)
- CSV 없이 대화(채팅)로도 산정 가능하도록 방식 B(대화형 입력) 추가: system_summary, total_users(전체 사용자 수),
  weekday_dau(평일 일간 활성 사용자 수) 3가지를 질문하여 수집. weekday_dau는 활성 사용량(동시접속 등) 산정에,
  total_users는 스토리지 등 전체 사용자 수 기반 자원 추정에 사용. system_prompt.md, SKILL.md, report_template.md에 반영됨.
- 통화 정책 확정: 비용 기준(단가)은 EUR로 저장/표시(사용료 기준금액이 EUR임). 실제 비용 산정/리포팅 시에는
  EUR→KRW 환율로 변환하여 KRW로 표기. 적용 환율과 기준일을 보고서에 항상 명시. 환율 미제공 시 사용자에게 확인
  요청 또는 최근 공개 환율을 가정값으로 명시. /data/sap_discovery_costs.json 샘플 데이터의 currency를 USD→EUR로
  수정 완료. system_prompt.md, SKILL.md, report_template.md에 EUR/KRW 병기 반영됨.
- SAP Discovery Center 검색용 web_search 도구 추가: 검색 시 "site:discovery-center.cloud.sap" 등 도메인 제한을
  반드시 포함하여 SAP Discovery Center 외 출처가 검색되지 않도록 함. 로컬 캐시가 최신(30일 이내)이면 web_search를
  호출하지 않음. system_prompt.md(도구 섹션), SKILL.md(적용 범위, 워크플로우 3단계)에 반영됨.

