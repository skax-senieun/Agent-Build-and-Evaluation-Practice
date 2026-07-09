Skill: SAP BTP 비용 산정 에이전트

목적
- SAP BTP(Cloud Foundry, Neo, Kyma 등) 및 CSP 환경에서의 리소스별 비용 산정을 자동화하고, 스펙 변경에 따른 월별/연별 비용 추정과 비용 최적화 조합을 추천한다.

적용 범위
- 비용 기준 출처: SAP Discovery Center(반드시 우선 사용)
- 입력: CSV (CSP, 자원명, 단계별 스펙, 대상 시스템 요약, 예상 사용자 수)
- 출력: 요약 문장, 리소스별 예상 사용량, 리소스별 스펙별 월별/년별 예상 비용 표, 전체 비용 최적 조합 추천 보고서(템플릿 준수)

워크플로우
1. 입력 검증
   - CSV 스키마 확인: 필수 컬럼(CSP, resource_id, resource_type, spec_tier, unit_spec, system_stage, expected_users)
   - 누락/불일치 시 사용자에게 최소한의 질문으로 보완 요청

2. 비용 기준 로컬 캐시 확인
   - 경로: /data/sap_discovery_costs.json (또는 다른 지정된 위치)
   - 파일이 존재하고 최근(예: 30일 이내)이라면 우선 사용

3. SAP Discovery Center 검색(필요 시)
   - 검색은 반드시 SAP Discovery Center 내에서만 수행
   - 검색 결과에서 스펙별 단가(unit price)와 과금 단위(시간/월/GB 등)를 추출
   - 추출 결과를 /data/sap_discovery_costs.json에 저장(버전/타임스탬프 포함)

4. 사용량(Consumption) 산정
   - 입력된 예상 사용자 수와 시스템 요약을 통해 자원별 사용 프로파일 생성(동시접속, 트랜잭션 수, 저장용량 증가율 등 기본 가정 사용)
   - 사용량 모델 템플릿을 적용하여 월별 사용량 추정
   - 사용자에게 커스텀 가정값(예: 동시접속률, 평균 트랜잭션 크기)을 입력받을 수 있음

5. 비용 산정 및 비교
   - 각 스펙별 단가와 추정 사용량을 곱해 월별/연별 비용 계산
   - 스펙 조합별로 전체 비용 및 리소스별 비용 비율 계산
   - 비용 감도분석: 스펙 변화(예: CPU vCPU 수, 메모리, 스토리지 종류)별 비용 변화 추이 생성

6. 최적화 추천
   - 제약조건(성능, SLA, 예산)을 고려해 후보 스펙 조합 생성
   - 비용과 예상 성능(간단한 규칙 기반)으로 점수화하여 상위 N개 추천

7. 보고서 작성
   - 지정된 템플릿으로 결과를 포맷팅
   - 포함: 요약 문장, 표(리소스별 사용량, 월/연 비용), 최적 스펙 조합, 가정 목록, 참고(사용한 SAP Discovery 링크 및 로컬 파일 경로)

파일/폴더 구조(권장)
- /data/sap_discovery_costs.json  # 캐시된 비용 기준
- /input/                         # 업로드된 CSV 파일 보관
- /output/                        # 생성된 보고서(HTML/Markdown/PDF)
- /templates/report_template.md   # 보고서 템플릿

입출력 예시
- 입력 CSV 컬럼 예시: CSP,resource_id,resource_type,spec_tier,unit_spec,system_stage,expected_users
- 출력: /output/report_<timestamp>.md (또는 .pdf) 및 요약(콘솔/대화)

검증 및 제한사항
- 비용 기준이 SAP Discovery Center에 없거나 불명확한 경우 사용자에게 수동 확인 요청
- 외부(Discovery 이외) 비용 기준 사용 불허
- 민감 정보(계정/비밀키 등) 저장 금지

확장 포인트
- CSP별(예: AWS, Azure, GCP) 가격 API 연동 (추후)
- 시뮬레이션 모드: 다양한 사용자 성장 시나리오 자동 생성

운영 규칙(필수)
- "자원 스펙별 비용 기준은 SAP Discovery 사이트만 서치한다. (스펙별 비용 기준 파일을 먼저 참고하고 없으면 서치)"
- 모든 보고서는 /templates/report_template.md를 따른다.

개발/배포 메모
- 우선 PoC는 파이썬 스크립트(BeautifulSoup 또는 requests + HTML 파싱)로 SAP Discovery에서 정보를 가져오고 JSON으로 저장
- 추후 UI/웹 서비스로 확장 가능
