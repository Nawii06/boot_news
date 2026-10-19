# 아카이브 데이터 규격

schema_version: 1.0

아래 규격은 예약 실행 프롬프트와 동일한 원문을 기준으로 합니다.

## 4. 저장소 사전 조회와 아카이브 구조

먼저 저장소 메타데이터·공개범위·기본 브랜치·파일 목록·최근 성공 실행기록·기존 이슈 색인을 확인한다. 기존 스키마가 있으면 호환성을 유지한다. 빈 저장소와 접근 실패를 구분한다. 빈 저장소는 권한과 도구가 허용할 때만 아래 기본 구조로 초기화한다. 기존 README와 사용자 파일을 일괄 덮어쓰지 않는다.

| 경로 | 용도 |
|---|---|
| README.md | 목적·탐색방법·분류·자료 한계·색인 링크 |
| docs/schema.md | 필드·상태값·ID·갱신 규칙, schema_version=1.0 |
| reports/YYYY/MM/YYYY-MM-DD.md | 최초 정기 리포트, 본문 6개 절 유지 |
| reports/YYYY/MM/YYYY-MM-DD/runs/RUN_ID.md | 같은 날 추가 수동실행·수정 리포트 |
| records/YYYY/ISSUE_ID.json | 이슈별 구조화 근거·출처·분석·변경 이력 |
| indexes/YYYY-MM.md | 월별 리포트와 이슈 탐색표 |
| manifests/YYYY/MM/RUN_ID.json | 회차별 검색·처리·누락·산출물 기록 |

RUN_ID는 실제 시작시각 KST의 YYYYMMDDTHHMMSS+0900 형태를 사용하고 충돌 시 짧은 임의 접미사를 붙인다. 동일 실행의 재시도에는 기존 RUN_ID를 유지한다. 최초 정기 리포트가 이미 있으면 덮어쓰지 말고 추가 회차 경로를 쓴다. 명백한 오류 정정은 정정일·전후 내용·근거를 남긴다.

### 4-1. 이슈 단위·중복·갱신

이슈는 기사 한 건이 아니라 하나의 정책 발표·공고·프로그램·협력사례 등 사건 단위다. 같은 사건의 언론/공식 출처는 하나의 레코드에 묶는다. 같은 기관의 다른 연도 사업·다른 모집회차는 별도 이슈로 구분한다.

우선 정규화 URL을 대조하고, 이어 기관·사업명·연도·발표일·공고번호·모집회차를 비교한다. URL에서는 명백한 추적용 utm_* 등의 인자와 단순 문서 앵커만 제거한다. 게시물 식별번호 등 의미 있는 쿼리는 보존한다. PDF 페이지 앵커 등 근거 위치는 별도 보존한다. 실제 확인하지 않은 canonical URL을 만들지 않는다.

ISSUE_ID는 최초 등록일의 NEWS-YYYYMMDD-001 형식으로 당일 미사용 번호를 배정하고 이후 유지한다. 확정 저장 직전 충돌을 다시 확인한다. 기존 이슈 판별이 불확실하면 임의 병합하지 말고 duplicate_review와 비교 대상 ID를 남긴다. 과거 레코드를 열지 못하면 완전한 중복제거를 했다고 보고하지 않는다.

- new: 기존 아카이브에 같은 이슈가 확인되지 않음. 최신 뉴스 여부와는 별개다.
- updated: 마감·대상·예산·자격·시행 등 실질 변경 또는 새 공식 근거가 확인됨.
- unchanged: 본문 핵심 사실에 변화 없음. 새 이슈로 중복 생성하지 않는다.
- correction: 기존 기록의 오류를 원문에 따라 정정함.
- duplicate_review: 동일 사건 여부 추가 확인 필요.

내용 변경 시 기존 ID와 first_seen_at을 유지하고 last_verified_at·변경 전후·근거 URL·회차를 갱신한다. 변동 없는 출처의 조회만으로 ‘정책 갱신’이라고 표시하지 않는다. 같은 내용의 재시도는 중복 파일·행·커밋을 만들지 않는다. 마지막 성공 이후 검색 공백이 있으면 기본 24시간과 별도로 최대 7일의 누락 보완기간을 조사하고, 그보다 긴 공백은 미수집 기간으로 명시한다. 실패 회차를 성공으로 바꾸어 기록하지 않는다.

### 4-2. 이슈 JSON 필수 필드

UTF-8 유효 JSON으로 저장하고, 다음 필드를 모두 둔다. 미확인 값은 null 또는 빈 배열로 둔다. 날짜만 확인되면 YYYY-MM-DD로 유지하고 임의의 00:00을 넣지 않는다.

- schema_version, issue_id, title, category, priority(A/B), priority_reason
- first_seen_at, last_verified_at, published_date, modified_date, event_date, effective_date
- freshness(new_24h/supplement_72h/supplement_7d/ongoing_notice/reference), change_status
- organizations[], regions[], sectors[], keywords[], program_year, notice_number
- verification_level(official/media_only/company_claim/mixed), verified_facts[]
- sources[]: title, publisher, source_type, url, normalized_url, published_date, accessed_at, verification_scope, evidence_locator
- verified_facts[]의 각 원소: fact_id, text, source_urls[], evidence_locator, claim_type(confirmed_fact/announced_plan/company_claim)
- notice: eligibility, support, deadline, application_url, current_status, university_eligibility_verified
- analysis: relevance, interpretations[], proposed_action, prerequisites[], limitations[]
- development: use_cases[], transferable_mechanism, evidence_fact_ids[], candidate_partners[], resource_requirements[], eligibility_checks[], evidence_gaps[], maturity
- related_issue_ids[], report_paths[], changes[]

category는 bootcamp/policy_region/industry/technology_curriculum 중 하나. 복수 맥락은 keywords와 use_cases로 표현한다. use_cases는 신규사업기획/교육과정/비교과/재직자교육/산학프로젝트/현장실습인턴십/취업연계/기업수요조사/학생지원/성과관리/사업비행정에서 해당값만 선택한다.

development는 재사용 근거가 있을 때만 채우고 없으면 빈 배열·null을 사용한다. ‘어떤 대상의 문제를 어떤 교육·기업협력 구조로 해결했는가’와 사업단에 적용하기 전 필요한 조건을 적는다. 기관명 등장만으로 협력 가능 기업으로 단정하지 않는다. candidate_partners에 쓰면 검토후보임을 명시하고 연결 근거를 남긴다. maturity는 evidence_only 또는 concept_review만 사용한다. 사업단 승인·예산 확보·협약 체결 상태를 만들어내지 않는다.

changes[]에는 changed_at, run_id, field, before, after, reason, source_urls를 기록한다. 레코드 안의 숫자에는 대상·단위·기간·계획/실적 구분을 함께 적는다. 사업 제안의 근거를 fact_id와 원문까지 추적할 수 있게 한다.

월별 색인은 날짜/ISSUE_ID/제목/A·B/분류/기관·지역/활용영역/신규·변경/마감일/레코드·리포트 링크로 구성한다. 기존 행은 ID로 갱신하고 다른 월의 기록을 삭제하지 않는다. 최초 등장 월 색인을 기준으로 유지하며, 후속 회차 월에는 같은 ID를 ‘재참조’로 연결할 수 있다. 기관 내 담당자 연락처·개인정보를 추출해 축적하지 않는다.


## 7. 실행기록·검증·GitHub 반영

manifest에는 schema_version, run_id, scheduled_at, cutoff_at, started_at, collection_completed_at, search_window, supplement_windows, search_status, queries[], checked_official_sites[], verification_failures[], excluded_candidates[](URL·짧은 제외사유), previous_success_run_id, coverage_gaps[], issue_ids별 change_status, counts, report_path, changed_paths, archive_intent를 둔다. 확인하지 못한 검색어·기관을 수행 완료로 기록하지 않는다. 지나치게 긴 검색 로그·기사 전문은 남기지 않는다.

원문을 보존할 수 있다는 허용 근거 없이 기사 전문·유료자료·이미지·첨부파일을 복제하지 않는다. 공개자료의 자체 요약·정확한 링크·짧은 근거 위치로 축적한다. 공개 저장소이므로 내부 문서·비공개 계획·개인정보·토큰·키·인증 헤더를 저장하지 않는다.

반영 순서:
1. 기준 브랜치의 최신 커밋과 기존 파일을 읽고 변경 집합을 만든다. 비어 있는 저장소면 지원되는 초기 커밋 생성 경로를 사용한다. 미지원이면 준비본만 만들고 한계를 알린다.
2. JSON 파싱, 필수 필드·허용 상태값·ID 유일성, 날짜·마감 근거, facts→sources 연결, 내부 상대링크·색인 일치, 중복 제거를 점검한다. 모든 숫자·자격·마감에 근거가 있는지 확인한다.
3. 가급적 이슈·리포트·색인·manifest를 하나의 원자적 커밋으로 반영한다. 기존 트리를 기반으로 변경하며 다른 파일은 보존한다. 커밋 메시지는 archive: YYYY-MM-DD daily bootcamp news 형식으로 한다.
4. 직접 쓰기가 허용되면 기본 브랜치에 기록한다. 보호 규칙으로 직접 쓰기가 막히고 브랜치/PR 생성이 허용되면 archive/YYYY-MM-DD-RUN_ID 브랜치에 기록하고 PR을 만든다. 자동 병합은 하지 않는다. PR은 ‘병합 대기’이며 기본 브랜치 적재 완료로 세지 않는다. 권한이 없으면 우회하지 않는다.
5. 기준 SHA 변경·충돌 시 최신 상태를 다시 읽고 중복과 병합을 재계산한다. 강제 push·이력 삭제·사용자 파일 삭제를 금지한다. 충돌 재시도는 최대 2회 후 실패 범위를 알린다.
6. 반영한 커밋/파일을 실제로 다시 읽어 주요 파일과 수량·링크를 대조한다. 도구 호출 성공만으로 완료를 선언하지 않는다. 다중 파일을 순차 저장하다 일부 실패하면 ‘일부 저장’으로 표시하고 미처리 경로를 알린다.
7. 실패 후 재실행할 때 manifest와 실제 파일을 대조하여 누락만 복구한다. 다른 최신 기록을 되돌리지 않는다. 입력·분석이 같으면 불필요한 신규 커밋을 만들지 않는다.

커밋 이전 작성한 리포트·manifest에 아직 존재하지 않는 자기 커밋 SHA나 ‘검증 완료’를 넣지 않는다. 저장 리포트의 처리 결과는 ‘이 문서는 아카이브 입력본이며 최종 반영 결과는 해당 실행의 채팅 응답 및 GitHub 커밋 이력에서 확인’으로 표시하고, 커밋 전 준비 상태·예정 경로·건수를 기록한다. manifest의 archive_intent는 ‘prepared’로 둔다. 저장 후 확인한 SHA·실제 완료상태는 최종 채팅 응답에만 기재하여 자기 참조 커밋과 상태 선기록을 피한다. 다음 회차는 manifest만 믿지 말고 실제 파일·커밋을 대조해 이전 성공 여부를 판정한다.


