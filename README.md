# 실행 기록 (offline_fixture)

**Cyberdyne Systems**

**작성자 : Cyberdyne Systems Model 101**

- 선택 단계 / 실습: Level 01 basic / 08_langchain_pipeline (08_enhanced_rag_api)
- 실행 날짜와 Python Interpreter: 2026-11-24 17:40, .backendenv (c:\WorkSpace\.backendenv\Scripts\python.exe)
- 실행 모드: **offline_fixture**
- 요청 질문과 Category: 
 - Q01: "회의실 예약을 취소하는 방법은?" (Category: 시설)
 - Q06: "정확한 과제 마감 날짜는?" (Category: 과제/안내 - 근거 부족 테스트)
 - 예외 테스트: `INVALID_K` (422 검증 오류), `FIXED_RERANK_ERROR` (502 고정 오류)
- Query Rewrite의 원 질문과 검색용 질문:
  - Q01: `회의실 예약을 취소하는 방법은?` → `회의실 예약을 취소하는 방법은?` (변환 없음)
  - Q06: `정확한 과제 마감 날짜는?` → `정확한 과제 마감 날짜는?` (변환 없음)
- Candidate ID → Rerank ID → Context included ID:
  - Q01: `[FAC-102-C1, FAC-103-C1, FAC-104-C1, FAC-105-C1, FAC-109-C1]` → `[FAC-102-C1, FAC-103-C1, FAC-104-C1, FAC-105-C1, FAC-109-C1]` → **`[FAC-102-C1, FAC-103-C1]`**
  - Q06: `[ASS-102-C1, ASS-103-C1, ASS-101-C1, IT-102-C1, ASS-104-C1]` → `[ASS-102-C1, ASS-103-C1, ASS-101-C1, IT-102-C1, ASS-104-C1]` → **`[ASS-102-C1, ASS-103-C1]`**
- 답변과 출처 문서의 실제 내용 대조:
  - Q01(200 `answered`): "[교육용 고정 응답] 회의실 예약을 취소하려면 예약 시스템의 나의 예약 메뉴에서 취소를 선택합니다. 시작 2시간 전까지 취소할 수 있습니다." — 출처 문서 `FAC-102-C1` (`guides/fac-102.md#section-1`, `citation_id: 1002`)의 예약 취소 경로 및 2시간 전 제한 규정과 일치.
  - Q06(200 `insufficient_context`): "제공된 지식으로는 답할 수 없습니다." — 포함된 Context(`ASS-102-C1`, `ASS-103-C1`)에 정확한 마감 날짜 근거가 없어 출처(`[]`) 없이 정상 거절.
- 실패 또는 예상과 다른 결과:
  - 초기 실행 시 `create_app`을 `coursekit.runtime`에서 잘못 임포트하여 `ImportError`가 발생했고, `.backendenv`에 `pytest` 모듈이 누락되어 설치 후 재실행함. 수정 후 상태 코드(`200, 200, 422, 502`) 출력 및 `python -m pytest -q tests` 실행 결과 **`1 passed`** **`17 warnings`**로 경고가 있지만 정상 통과까지 확인하였습니다.
- 고친 TODO와 이유:
  - **D27-08-B01 (`make_service_chain`)**: `coursekit.runtime`의 `create_demo_chain(offline=offline)`을 반환하도록 연결하여 Rewrite → Retrieve → Rerank → Context → Generation·Validation 파이프라인을 구성.
  - **D27-08-B02 (`wire_fastapi`)**: API 요청의 `role` 문자열을 신뢰하지 않고 서버에서 `learner` 역할을 강제하기 위해 `coursekit.api`의 `create_app(chain, trusted_role="learner")`를 연결.
- 다음 개선 가설 (실측 전에 성공으로 단정하지 말 것):
  - `offline_fixture` 모드에서는 고정된 더미 데이터와 순서를 사용하므로, 실제 임베딩·리랭킹 모델과 LLM이 연결된 `live` 환경에서 검색 순위 재정렬 및 응답 지연 시간(Latency)을 실측해 검증할 필요가 있어보입니다.
  실제로 실험을 담당하는 장비에서 live의 실행을 완수하지 못해 live의 결과값을 확인하지 못 했습니다. live의 값과 비교해 작성한다면 더 분석하기 수월해 추가 보고서 작성이 완료될 것으로 보입니다.
