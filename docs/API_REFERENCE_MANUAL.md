# Unified E-Commerce Operations OS — API Reference Manual

**Version**: 1.0
**Last Updated**: 2026-02-05
**대상**: 개발팀 전원
**목적**: 프로젝트 구축에 필요한 모든 외부 API 및 기술 문서를 체계적으로 정리하여 개발 효율을 극대화한다.

---

## 목차

1. [이커머스 플랫폼 API](#1-이커머스-플랫폼-api)
2. [택배/배송 추적 API](#2-택배배송-추적-api)
3. [결제/PG API](#3-결제pg-api)
4. [금융/은행/세무 API](#4-금융은행세무-api)
5. [AI/ML API](#5-aiml-api)
6. [실시간 통신 API](#6-실시간-통신-api)
7. [Backend Framework & DB](#7-backend-framework--db)
8. [Frontend Framework](#8-frontend-framework)
9. [인프라/DevOps](#9-인프라devops)
10. [파일 저장소 & 유틸리티](#10-파일-저장소--유틸리티)
11. [연동 우선순위 로드맵](#11-연동-우선순위-로드맵)
12. [API 인증 방식 요약](#12-api-인증-방식-요약)
13. [Rate Limit 정리](#13-rate-limit-정리)
14. [개발 환경 세팅 체크리스트](#14-개발-환경-세팅-체크리스트)

---

## 1. 이커머스 플랫폼 API

프로젝트의 핵심 데이터 소스. 주문, 상품, 재고, 정산 데이터를 수집하는 플랫폼별 API.

### 1.1 네이버 스마트스토어

| 항목 | 내용 |
|------|------|
| **API 이름** | 네이버 커머스 API (Commerce API) |
| **문서 URL** | https://apicenter.commerce.naver.com |
| **인증 방식** | OAuth 2.0 (Client Credentials) |
| **주요 엔드포인트** | 주문 조회/변경, 상품 등록/수정, 정산 내역 조회 |
| **데이터 포맷** | JSON |
| **Rate Limit** | 초당 30건 (일반), 초당 10건 (주문 변경) |

**사용 Phase**: Phase 2 (Platform Integration)

**필수 스코프**:
- `commerce.order.read` — 주문 조회
- `commerce.order.write` — 주문 상태 변경
- `commerce.product.read` — 상품 조회
- `commerce.product.write` — 상품 등록/수정
- `commerce.settlement.read` — 정산 조회

**연동 시 주의사항**:
- 토큰 만료 시간 확인 후 자동 갱신 로직 구현 필요
- 주문 조회 시 `lastChangedFrom` 파라미터로 증분 동기화
- 대량 상품 조회 시 페이지네이션 처리 (최대 100건/페이지)

---

### 1.2 카페24

| 항목 | 내용 |
|------|------|
| **API 이름** | Cafe24 Admin API |
| **문서 URL** | https://developers.cafe24.com |
| **API 레퍼런스** | https://developers.cafe24.com/docs/ko/api/admin |
| **인증 방식** | OAuth 2.0 (Authorization Code Flow) |
| **주요 엔드포인트** | 주문(Orders), 상품(Products), 재고(Inventory), 회원(Customers) |
| **데이터 포맷** | JSON |
| **Rate Limit** | 분당 100건 (앱 기준) |

**사용 Phase**: Phase 2 (Platform Integration)

**필수 스코프**:
- `mall.read_order` — 주문 조회
- `mall.write_order` — 주문 처리
- `mall.read_product` — 상품 조회
- `mall.write_product` — 상품 수정
- `mall.read_supply` — 재고 조회
- `mall.write_supply` — 재고 수정

**연동 시 주의사항**:
- Webhook 지원: 주문 상태 변경 시 실시간 알림 수신 가능
- 멀티몰 환경에서는 `mall_id` 구분 필요
- API 버전 관리: 최신 버전(v2) 사용 권장

---

### 1.3 쿠팡

| 항목 | 내용 |
|------|------|
| **API 이름** | 쿠팡 Wing API (COUPANG WING) |
| **문서 URL** | https://wing.coupang.com |
| **개발자 문서** | https://developers.coupangcorp.com |
| **인증 방식** | HMAC-SHA256 서명 (Access Key + Secret Key) |
| **주요 엔드포인트** | 주문 조회/확인, 배송 등록, 상품 등록, 정산 조회 |
| **데이터 포맷** | JSON |
| **Rate Limit** | 분당 60건 |

**사용 Phase**: Phase 2 (Platform Integration)

**HMAC 인증 구현 가이드**:
```
1. HTTP Method + Path + Timestamp + Body를 결합하여 서명 문자열 생성
2. Secret Key로 HMAC-SHA256 서명
3. Authorization 헤더에 Access Key + 서명값 포함
```

**연동 시 주의사항**:
- 주문 확인(승인) API 호출 필수 — 미확인 시 자동 취소
- 배송 등록 시 송장번호 필수
- 정산은 주 단위로 조회 가능
- 로켓배송/판매자 직배송 구분 처리

---

### 1.4 11번가

| 항목 | 내용 |
|------|------|
| **API 이름** | 11번가 셀러 Open API |
| **문서 URL** | https://openapi.11st.co.kr |
| **인증 방식** | API Key (Header) |
| **주요 엔드포인트** | 주문 조회, 상품 등록/수정, 배송 처리 |
| **데이터 포맷** | XML (일부 JSON) |
| **Rate Limit** | 초당 5건 |

**사용 Phase**: Phase 2 (확장)

**연동 시 주의사항**:
- 응답 포맷이 XML인 경우가 많으므로 XML 파서 필요
- API Key 발급은 셀러 오피스에서 진행
- 상품 등록 시 카테고리 코드 매핑 필요

---

### 1.5 G마켓/옥션 (ESM Plus)

| 항목 | 내용 |
|------|------|
| **API 이름** | ESM Plus Open API (eBay Korea) |
| **문서 URL** | https://developer.gmarket.co.kr |
| **인증 방식** | OAuth 2.0 |
| **주요 엔드포인트** | 주문/클레임 조회, 상품 등록, 배송 처리 |
| **데이터 포맷** | JSON |
| **Rate Limit** | 분당 60건 |

**사용 Phase**: Phase 2 (확장)

**연동 시 주의사항**:
- G마켓과 옥션 통합 API (ESM 2.0)
- 클레임(교환/반품) 처리 API 별도 존재
- 셀러 등급에 따라 API 호출 제한 차등

---

### 1.6 티몬

| 항목 | 내용 |
|------|------|
| **API 이름** | 티몬 파트너 API |
| **문서 URL** | https://partners.tmon.co.kr |
| **인증 방식** | API Key |
| **주요 엔드포인트** | 딜 관리, 주문 조회, 배송 처리 |
| **데이터 포맷** | JSON |

**사용 Phase**: Phase 2 (확장)

---

### 1.7 위메프

| 항목 | 내용 |
|------|------|
| **API 이름** | 위메프 파트너 API |
| **문서 URL** | https://partner.wemakeprice.com |
| **인증 방식** | API Key |
| **주요 엔드포인트** | 상품 관리, 주문 조회, 배송 처리 |
| **데이터 포맷** | JSON |

**사용 Phase**: Phase 2 (확장)

---

## 2. 택배/배송 추적 API

주문 발송 후 배송 상태를 실시간으로 추적하기 위한 API.

### 2.1 통합 택배 추적 서비스 (권장)

> 개별 택배사 API를 각각 연동하는 것보다 통합 API 하나로 처리하는 것이 효율적.

| 항목 | 내용 |
|------|------|
| **API 이름** | 스마트택배 API (Sweet Tracker) |
| **문서 URL** | https://tracking.sweettracker.co.kr |
| **인증 방식** | API Key |
| **지원 택배사** | CJ대한통운, 한진, 롯데, 우체국, 로젠 등 국내 전 택배사 |
| **데이터 포맷** | JSON |
| **요금** | 무료 (일 100건) / 유료 플랜 |

**사용 Phase**: Phase 3 (Order & Shipping)

---

### 2.2 해외 배송 추적

| 항목 | 내용 |
|------|------|
| **API 이름** | Aftership Tracking API |
| **문서 URL** | https://www.aftership.com/docs/tracking |
| **인증 방식** | API Key (Header) |
| **지원 운송사** | 1,100+ 글로벌 운송사 (DHL, FedEx, UPS, EMS 등) |
| **데이터 포맷** | JSON |
| **Webhook** | 배송 상태 변경 시 자동 알림 지원 |

**사용 Phase**: Phase 3 + Phase 6 (해외 공장 발주 운송 추적)

**연동 시 주의사항**:
- 해외 공장→국내 입고 과정 추적에 활용
- 운송장 번호로 자동 택배사 감지 기능 제공
- Webhook 설정으로 실시간 상태 업데이트

---

### 2.3 개별 택배사 API (참고용)

| 택배사 | API/추적 URL | 비고 |
|--------|-------------|------|
| CJ대한통운 | https://www.cjlogistics.com/ko/tool/parcel/tracking | 스크래핑 or 계약 API |
| 한진택배 | https://www.hanjin.com/kor/CMS/DeliveryMgr/WaybillResult.do | 스크래핑 or 계약 API |
| 롯데택배 | https://www.lotteglogis.com/open/tracking | 스크래핑 or 계약 API |
| 우체국택배 | https://www.epost.go.kr/main.retrieveMainPage.comm | EPost Open API |
| 로젠택배 | https://www.ilogen.com/web/personal/trace | 계약 API |

> 대부분 택배사는 공개 API가 제한적이므로, 통합 추적 서비스(2.1) 사용을 권장한다.

---

## 3. 결제/PG API

B2B 인보이스 결제, 향후 자체 결제 처리를 위한 PG사 API.

### 3.1 포트원 (구 아임포트) — 권장

| 항목 | 내용 |
|------|------|
| **API 이름** | PortOne (아임포트) 통합 결제 API |
| **문서 URL** | https://developers.portone.io/docs/ko/readme |
| **API 레퍼런스** | https://developers.portone.io/api/rest-v2 |
| **인증 방식** | API Key + Secret (Bearer Token) |
| **지원 PG** | KG이니시스, NHN KCP, 토스페이먼츠, 나이스페이 등 통합 |
| **지원 결제수단** | 카드, 계좌이체, 가상계좌, 간편결제(카카오페이, 네이버페이 등) |
| **데이터 포맷** | JSON |
| **Webhook** | 결제 완료/취소/환불 시 알림 |

**사용 Phase**: Phase 5 (Finance) + Phase 6 (B2B)

**연동 시 주의사항**:
- V2 API 사용 권장
- 가상계좌(B2B 결제) 입금 확인 Webhook 처리 필수
- 정기결제(구독형 B2B) 시 빌링키 발급 플로우

---

### 3.2 토스페이먼츠

| 항목 | 내용 |
|------|------|
| **API 이름** | Toss Payments API |
| **문서 URL** | https://docs.tosspayments.com/reference |
| **인증 방식** | Basic Auth (Secret Key base64 인코딩) |
| **주요 기능** | 결제 승인, 취소, 정산 조회, 가상계좌 |
| **데이터 포맷** | JSON |
| **Sandbox** | 테스트 환경 제공 |

**사용 Phase**: Phase 5 (대안 PG)

---

### 3.3 NHN KCP

| 항목 | 내용 |
|------|------|
| **API 이름** | KCP Payment API |
| **문서 URL** | https://developer.kcp.co.kr |
| **인증 방식** | Site Code + Group ID |
| **주요 기능** | 결제, 취소, 에스크로 |

**사용 Phase**: Phase 5 (대안 PG)

---

### 3.4 이니시스

| 항목 | 내용 |
|------|------|
| **API 이름** | KG이니시스 API |
| **문서 URL** | https://manual.inicis.com |
| **인증 방식** | MID + Sign Key |
| **주요 기능** | 결제, 취소, 부분취소, 에스크로 |

**사용 Phase**: Phase 5 (대안 PG)

---

## 4. 금융/은행/세무 API

재무 관리 핵심. 은행 거래 내역, 카드 매입, 세금계산서 처리.

### 4.1 오픈뱅킹 API

| 항목 | 내용 |
|------|------|
| **API 이름** | 금융결제원 오픈뱅킹 공동 API |
| **문서 URL** | https://developers.openbanking.or.kr |
| **인증 방식** | OAuth 2.0 (Authorization Code + Client Credentials) |
| **주요 기능** | 계좌 잔액 조회, 거래 내역 조회, 이체 |
| **데이터 포맷** | JSON |
| **이용 대상** | 핀테크 서비스 등록 필요 (금융위원회 등록) |

**사용 Phase**: Phase 5 (Finance)

**연동 시 주의사항**:
- 핀테크 서비스 등록 절차 필요 (심사 소요)
- 사용자별 계좌 연결 동의 필요 (본인인증)
- 테스트베드 환경에서 충분히 검증 후 운영 전환
- 거래 내역 조회 기간 제한 확인

**핵심 API 목록**:
```
GET /account/balance    — 계좌 잔액 조회
GET /account/transaction_list — 거래 내역 조회
POST /transfer/withdraw — 출금 이체
POST /transfer/deposit  — 입금 이체
```

---

### 4.2 쿠콘 (COOCON) — 금융 데이터 수집

| 항목 | 내용 |
|------|------|
| **API 이름** | COOCON Financial Data API |
| **문서 URL** | https://developer.coocon.net |
| **인증 방식** | API Key + 암호화 |
| **주요 기능** | 은행 거래 내역 스크래핑, 카드 사용 내역, 증권 데이터 |
| **데이터 포맷** | JSON |

**사용 Phase**: Phase 5 (Finance)

**연동 시 주의사항**:
- 오픈뱅킹 대비 더 넓은 범위의 금융 데이터 수집 가능
- 카드 매입 데이터 수집에 유용
- 계약 필요 (유료 서비스)

---

### 4.3 여신금융협회 (카드 매입 데이터)

| 항목 | 내용 |
|------|------|
| **API 이름** | 여신금융협회 가맹점 매출 조회 |
| **문서 URL** | https://www.crefia.or.kr |
| **주요 기능** | 카드 매출 집계, 카드사별 매입 내역, 수수료 확인 |

**사용 Phase**: Phase 5 (Finance)

**연동 시 주의사항**:
- 사업자번호 기반 조회
- 매출 데이터 → 이커머스 정산과 대사(매칭)에 활용

---

### 4.4 팝빌 (세금계산서/인보이스)

| 항목 | 내용 |
|------|------|
| **API 이름** | Popbill API |
| **문서 URL** | https://developers.popbill.com |
| **인증 방식** | 링크아이디 + 비밀키 |
| **주요 기능** | 전자세금계산서 발행/조회, 현금영수증, 팩스, 문자 |
| **데이터 포맷** | JSON |
| **SDK** | Python, Java, PHP, Node.js 등 공식 SDK 제공 |

**사용 Phase**: Phase 5 (Invoice) + Phase 6 (B2B)

**핵심 기능**:
```
전자세금계산서:
- 정발행 (매출자 → 매입자)
- 역발행 (매입자 요청 → 매출자 승인)
- 위수탁 발행
- 발행 내역 조회/상태 확인
- 국세청 전송 상태 확인

현금영수증:
- 발급/취소
- 내역 조회
```

**연동 시 주의사항**:
- B2B 인보이스 발행 시 전자세금계산서 연동 필수
- 국세청 전송까지 자동 처리
- 테스트 환경 별도 제공

---

### 4.5 홈택스 (국세청)

| 항목 | 내용 |
|------|------|
| **API 이름** | 홈택스 Open API |
| **문서 URL** | https://www.hometax.go.kr |
| **주요 기능** | 세금계산서 조회, 사업자 상태 확인 |

**사용 Phase**: Phase 5 (참고)

> 직접 연동보다는 팝빌(4.4)을 통한 간접 연동이 효율적.

---

## 5. AI/ML API

AI 에이전트, 지식 저장소, 머신러닝 분석을 위한 API.

### 5.1 OpenAI API

| 항목 | 내용 |
|------|------|
| **API 이름** | OpenAI Platform API |
| **문서 URL** | https://platform.openai.com/docs/api-reference |
| **인증 방식** | Bearer Token (API Key) |
| **주요 모델** | GPT-4o, GPT-4o-mini, text-embedding-3-small/large |
| **데이터 포맷** | JSON |
| **Rate Limit** | 티어별 상이 (TPM/RPM) |

**사용 Phase**: Phase 8 (AI/ML Engine)

**사용 용도**:
- **Chat Completions**: CS 자동 응답, 지식 QA
- **Embeddings**: 문서/지식 벡터화 (text-embedding-3-small 권장)
- **Function Calling**: LangGraph 도구 호출

**비용 최적화 팁**:
- 간단한 분류/추출 작업: `gpt-4o-mini` (저비용)
- 복잡한 추론/분석: `gpt-4o` (고품질)
- 임베딩: `text-embedding-3-small` (가성비 최적)

---

### 5.2 Anthropic Claude API

| 항목 | 내용 |
|------|------|
| **API 이름** | Anthropic Messages API |
| **문서 URL** | https://docs.anthropic.com/en/docs |
| **API 레퍼런스** | https://docs.anthropic.com/en/api |
| **인증 방식** | API Key (x-api-key Header) |
| **주요 모델** | Claude Opus 4.5, Claude Sonnet 4, Claude Haiku 3.5 |
| **데이터 포맷** | JSON |

**사용 Phase**: Phase 8 (AI/ML Engine)

**사용 용도**:
- **Messages API**: CS 자동 응답 (대안 LLM)
- **Tool Use**: LangGraph 에이전트 도구 호출
- **긴 문맥 처리**: 200K 토큰 컨텍스트 윈도우 활용

---

### 5.3 LangGraph

| 항목 | 내용 |
|------|------|
| **라이브러리 이름** | LangGraph |
| **문서 URL** | https://langchain-ai.github.io/langgraph/ |
| **튜토리얼** | https://langchain-ai.github.io/langgraph/tutorials/ |
| **How-to Guides** | https://langchain-ai.github.io/langgraph/how-tos/ |
| **API 레퍼런스** | https://langchain-ai.github.io/langgraph/reference/ |
| **설치** | `pip install langgraph` |
| **의존성** | langchain-core |

**사용 Phase**: Phase 8 (AI/ML Engine) — 핵심 프레임워크

**핵심 개념 학습 순서**:
```
1. StateGraph 기본 — 상태 정의, 노드 추가, 엣지 연결
2. Conditional Edges — 조건부 라우팅 (문의 유형별 분기)
3. Tool Calling — 외부 도구 호출 (DB 조회, API 호출)
4. Human-in-the-Loop — 사람 개입 포인트 설정
5. Checkpointing — 상태 저장/복원 (대화 기록)
6. Subgraphs — 그래프 내 서브그래프 (복잡한 워크플로우)
7. Streaming — 실시간 응답 스트리밍
```

**프로젝트 내 Agent Graph 설계**:
```
CS Agent:       classify → route → [order/shipping/return/general] → respond → review
Analytics Agent: collect → analyze → [inventory/sales/finance] → report → alert
RAG Pipeline:    query → embed → search → rerank → generate → respond
```

---

### 5.4 LangChain

| 항목 | 내용 |
|------|------|
| **라이브러리 이름** | LangChain (Python) |
| **문서 URL** | https://python.langchain.com/docs/introduction/ |
| **API 레퍼런스** | https://python.langchain.com/api_reference/ |
| **설치** | `pip install langchain langchain-openai langchain-anthropic` |

**사용 Phase**: Phase 8

**사용하는 주요 모듈**:
- `langchain-openai`: OpenAI 모델 연동
- `langchain-anthropic`: Claude 모델 연동
- `langchain-community`: 커뮤니티 도구/벡터스토어
- `langchain.text_splitter`: 문서 청킹
- `langchain.embeddings`: 임베딩 처리

---

### 5.5 pgvector (벡터 검색)

| 항목 | 내용 |
|------|------|
| **확장 이름** | pgvector |
| **문서 URL** | https://github.com/pgvector/pgvector |
| **설치** | PostgreSQL extension (`CREATE EXTENSION vector;`) |
| **지원 인덱스** | IVFFlat, HNSW |
| **지원 거리 함수** | L2 (유클리드), Inner Product, Cosine |

**사용 Phase**: Phase 8 (벡터 지식 저장소)

**사용 가이드**:
```sql
-- 벡터 컬럼 생성
ALTER TABLE documents ADD COLUMN embedding vector(1536);

-- HNSW 인덱스 생성 (권장)
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);

-- 유사도 검색
SELECT id, content, 1 - (embedding <=> query_vector) AS similarity
FROM documents
ORDER BY embedding <=> query_vector
LIMIT 10;
```

**연동 시 주의사항**:
- OpenAI text-embedding-3-small 사용 시 차원: 1536
- HNSW 인덱스가 IVFFlat보다 검색 정확도 높음
- 10만 건 이상 시 인덱스 빌드 시간 고려

---

### 5.6 OpenAI Embeddings API

| 항목 | 내용 |
|------|------|
| **API 이름** | OpenAI Embeddings |
| **문서 URL** | https://platform.openai.com/docs/guides/embeddings |
| **모델** | text-embedding-3-small (1536d), text-embedding-3-large (3072d) |
| **인증** | Bearer Token |
| **최대 입력** | 8,191 토큰 |

**사용 Phase**: Phase 8

---

### 5.7 Cohere Embed API (대안)

| 항목 | 내용 |
|------|------|
| **API 이름** | Cohere Embed API |
| **문서 URL** | https://docs.cohere.com/reference/embed |
| **모델** | embed-multilingual-v3.0 (다국어 지원) |
| **인증** | Bearer Token |

**사용 Phase**: Phase 8 (대안 임베딩)

**연동 시 주의사항**:
- 한국어 문서가 많은 경우 다국어 모델이 유리할 수 있음
- OpenAI 임베딩과 성능 비교 테스트 권장

---

### 5.8 OCR API (인보이스 처리)

| 서비스 | 문서 URL | 특징 |
|--------|----------|------|
| **네이버 CLOVA OCR** | https://www.ncloud.com/product/aiService/ocr | 한국어 최적화, 영수증/세금계산서 특화 |
| **Google Cloud Vision** | https://cloud.google.com/vision/docs | 다국어, 문서 AI |
| **AWS Textract** | https://docs.aws.amazon.com/textract/ | 테이블/양식 추출 강점 |

**사용 Phase**: Phase 5 (인보이스 수취 OCR) + Phase 8 (AI Engine)

**권장**: 한국어 인보이스 처리에는 **CLOVA OCR** 우선 검토, 해외 인보이스에는 **Google Vision/Textract** 활용

---

## 6. 실시간 통신 API

채팅 시스템 및 알림을 위한 실시간 통신 기술.

### 6.1 Socket.IO

| 항목 | 내용 |
|------|------|
| **라이브러리 이름** | Socket.IO |
| **클라이언트 문서** | https://socket.io/docs/v4/client-api/ |
| **서버 문서** | https://socket.io/docs/v4/server-api/ |
| **Python 서버** | https://python-socketio.readthedocs.io/en/stable/ |
| **프로토콜** | WebSocket + HTTP Long Polling (자동 폴백) |

**사용 Phase**: Phase 7 (Chat & Messaging)

**핵심 이벤트 설계**:
```
Client → Server:
- chat:send_message    — 메시지 전송
- chat:typing          — 타이핑 표시
- chat:read            — 읽음 확인
- chat:join_room       — 채팅방 입장
- chat:leave_room      — 채팅방 퇴장

Server → Client:
- chat:new_message     — 새 메시지 수신
- chat:user_typing     — 타이핑 알림
- chat:message_read    — 읽음 확인 알림
- chat:user_joined     — 사용자 입장 알림
- notification:system  — 시스템 알림
```

**연동 시 주의사항**:
- Redis Adapter 사용으로 다중 서버 인스턴스 간 동기화
- 네임스페이스 분리: `/chat`, `/notifications`
- 인증: 연결 시 JWT 토큰 검증 미들웨어

---

### 6.2 Firebase Cloud Messaging (FCM)

| 항목 | 내용 |
|------|------|
| **API 이름** | Firebase Cloud Messaging |
| **문서 URL** | https://firebase.google.com/docs/cloud-messaging |
| **Admin SDK** | https://firebase.google.com/docs/cloud-messaging/server |
| **인증 방식** | 서비스 계정 키 (JSON) |
| **지원 플랫폼** | Web, Android, iOS |

**사용 Phase**: Phase 7 (푸시 알림)

**사용 용도**:
- 새 메시지 알림 (채팅)
- 주문 상태 변경 알림
- 재고 부족 알림
- 발주 승인 요청 알림

---

### 6.3 Web Push API

| 항목 | 내용 |
|------|------|
| **API 이름** | Web Push API |
| **문서 URL** | https://developer.mozilla.org/en-US/docs/Web/API/Push_API |
| **Python 라이브러리** | `pywebpush` |

**사용 Phase**: Phase 7 (웹 브라우저 푸시)

---

## 7. Backend Framework & DB

### 7.1 FastAPI

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://fastapi.tiangolo.com |
| **API 레퍼런스** | https://fastapi.tiangolo.com/reference/ |
| **설치** | `pip install fastapi[standard]` |
| **핵심 기능** | Async/Await, 자동 OpenAPI 문서, Pydantic 통합, 의존성 주입 |

**학습 우선순위**:
```
1. Path Operations (라우팅)
2. Dependency Injection (의존성 주입)
3. Pydantic Models (요청/응답 스키마)
4. Middleware (인증, 로깅, CORS)
5. Background Tasks (비동기 작업)
6. WebSocket (실시간 통신)
7. Testing (TestClient)
```

---

### 7.2 SQLAlchemy 2.0

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://docs.sqlalchemy.org/en/20/ |
| **설치** | `pip install sqlalchemy[asyncio] asyncpg` |
| **핵심 기능** | Async 지원, ORM 매핑, 마이그레이션 (Alembic) |

**연관 도구**:
- **Alembic** (DB 마이그레이션): https://alembic.sqlalchemy.org/en/latest/

---

### 7.3 Pydantic

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://docs.pydantic.dev/latest/ |
| **설치** | `pip install pydantic` |
| **핵심 기능** | 데이터 검증, 직렬화, Settings 관리 |

---

### 7.4 Celery (비동기 작업 큐)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://docs.celeryq.dev/en/stable/ |
| **설치** | `pip install celery[redis]` |
| **브로커** | Redis 또는 RabbitMQ |
| **핵심 기능** | 비동기 작업, 주기적 작업 (Celery Beat), 워크플로우 |

**사용 용도**:
- 이커머스 플랫폼 주기적 데이터 동기화
- 대량 주문 일괄 처리
- AI 분석 리포트 백그라운드 생성
- 이메일/알림 발송

---

### 7.5 PostgreSQL 16

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://www.postgresql.org/docs/16/ |
| **드라이버** | `asyncpg` (비동기) |
| **확장** | pgvector (벡터 검색), pg_trgm (퍼지 검색) |

---

### 7.6 MongoDB 7

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://www.mongodb.com/docs/manual/ |
| **Python 드라이버** | `motor` (비동기): https://motor.readthedocs.io/en/stable/ |
| **ODM** | `beanie`: https://beanie-odm.dev/ |
| **사용 용도** | 채팅 메시지, 비정형 로그, AI 대화 기록 |

---

### 7.7 Redis 7

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://redis.io/docs/latest/ |
| **Python 클라이언트** | `redis-py` (비동기 지원): https://redis-py.readthedocs.io/en/stable/ |
| **사용 용도** | 캐싱, 세션, Socket.IO Pub/Sub, Celery 브로커, Rate Limiting |

---

## 8. Frontend Framework

### 8.1 Next.js 15

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://nextjs.org/docs |
| **API 레퍼런스** | https://nextjs.org/docs/app/api-reference |
| **설치** | `npx create-next-app@latest` |
| **핵심 기능** | App Router, Server Components, SSR/SSG, API Routes |

**학습 우선순위**:
```
1. App Router (라우팅, 레이아웃)
2. Server Components vs Client Components
3. Data Fetching (fetch, Server Actions)
4. Middleware (인증 체크)
5. API Routes (BFF 패턴)
6. Image/Font 최적화
```

---

### 8.2 React

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://react.dev/reference/react |
| **학습 URL** | https://react.dev/learn |

---

### 8.3 TypeScript

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://www.typescriptlang.org/docs/ |
| **핸드북** | https://www.typescriptlang.org/docs/handbook/intro.html |

---

### 8.4 Zustand (상태 관리)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://zustand-demo.pmnd.rs/ |
| **GitHub** | https://github.com/pmndrs/zustand |
| **설치** | `npm install zustand` |
| **사용 용도** | 클라이언트 상태 관리 (장바구니, UI 상태, 필터 등) |

---

### 8.5 shadcn/ui (UI 컴포넌트)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://ui.shadcn.com/docs |
| **컴포넌트 목록** | https://ui.shadcn.com/docs/components |
| **설치** | `npx shadcn@latest init` |
| **기반** | Radix UI + Tailwind CSS |

**사용 용도**: 대시보드 UI 컴포넌트 (테이블, 폼, 다이얼로그, 차트 등)

---

### 8.6 Recharts (차트)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://recharts.org/en-US/api |
| **설치** | `npm install recharts` |
| **사용 용도** | 매출 차트, 재고 그래프, 재무 대시보드 시각화 |

---

### 8.7 TanStack Table (데이터 테이블)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://tanstack.com/table/latest/docs/introduction |
| **설치** | `npm install @tanstack/react-table` |
| **사용 용도** | 주문 목록, 재고 목록, 거래 내역 등 대량 데이터 테이블 |

---

### 8.8 TanStack Query (서버 상태 관리)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://tanstack.com/query/latest/docs/framework/react/overview |
| **설치** | `npm install @tanstack/react-query` |
| **사용 용도** | API 데이터 패칭, 캐싱, 동기화, 무한 스크롤 |

---

### 8.9 Playwright (E2E 테스트)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://playwright.dev/docs/intro |
| **설치** | `npm install -D @playwright/test` |
| **사용 용도** | 전체 유저 플로우 E2E 테스트 |

---

## 9. 인프라/DevOps

### 9.1 Docker

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://docs.docker.com/reference/ |
| **Dockerfile 레퍼런스** | https://docs.docker.com/reference/dockerfile/ |
| **Compose 레퍼런스** | https://docs.docker.com/compose/compose-file/ |

---

### 9.2 Kubernetes

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://kubernetes.io/docs/home/ |
| **API 레퍼런스** | https://kubernetes.io/docs/reference/kubernetes-api/ |
| **kubectl 치트시트** | https://kubernetes.io/docs/reference/kubectl/cheatsheet/ |

---

### 9.3 Helm (K8s 패키지 매니저)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://helm.sh/docs/ |
| **차트 개발** | https://helm.sh/docs/chart_template_guide/ |

---

### 9.4 Apache Kafka

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://kafka.apache.org/documentation/ |
| **Python 클라이언트** | `aiokafka`: https://aiokafka.readthedocs.io/en/stable/ |
| **대안 (클라우드)** | Confluent Cloud: https://docs.confluent.io/cloud/current/overview.html |

**핵심 토픽 설계**:
```
order.created        — 주문 생성
order.status_changed — 주문 상태 변경
inventory.updated    — 재고 변동
inventory.low_stock  — 재고 부족 알림
finance.transaction  — 재무 거래 발생
finance.settlement   — 정산 데이터
procurement.po_created — 발주 생성
chat.message         — 채팅 메시지 (알림용)
ai.task_completed    — AI 작업 완료
```

---

### 9.5 Keycloak (인증/IAM)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://www.keycloak.org/documentation |
| **Admin REST API** | https://www.keycloak.org/docs-api/latest/rest-api/index.html |
| **Python 클라이언트** | `python-keycloak`: https://python-keycloak.readthedocs.io/en/latest/ |
| **핵심 기능** | SSO, RBAC, 사용자 관리, 소셜 로그인, 외부 사용자(게스트) 관리 |

---

### 9.6 모니터링 스택

| 도구 | 문서 URL | 용도 |
|------|----------|------|
| **Prometheus** | https://prometheus.io/docs/ | 메트릭 수집 |
| **Grafana** | https://grafana.com/docs/grafana/latest/ | 대시보드/시각화 |
| **Loki** | https://grafana.com/docs/loki/latest/ | 로그 집계 |
| **Jaeger** | https://www.jaegertracing.io/docs/ | 분산 트레이싱 |
| **AlertManager** | https://prometheus.io/docs/alerting/latest/alertmanager/ | 알림 관리 |

---

### 9.7 GitHub Actions (CI/CD)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://docs.github.com/en/actions |
| **워크플로우 문법** | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions |
| **마켓플레이스** | https://github.com/marketplace?type=actions |

---

## 10. 파일 저장소 & 유틸리티

### 10.1 MinIO (S3 호환 Object Storage)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://min.io/docs/minio/linux/developers/minio-drivers.html |
| **Python SDK** | https://min.io/docs/minio/linux/developers/python/API.html |
| **설치** | `pip install minio` |
| **사용 용도** | 채팅 파일 업로드, 인보이스 PDF 저장, 상품 이미지 |

---

### 10.2 AWS S3 (대안)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://docs.aws.amazon.com/AmazonS3/latest/API/ |
| **Python SDK** | `boto3`: https://boto3.amazonaws.com/v1/documentation/api/latest/index.html |

---

### 10.3 WeasyPrint (PDF 생성)

| 항목 | 내용 |
|------|------|
| **문서 URL** | https://doc.courtbouillon.org/weasyprint/stable/ |
| **설치** | `pip install weasyprint` |
| **사용 용도** | 인보이스 PDF, 리포트 PDF 생성 |

---

### 10.4 이메일 발송

| 서비스 | 문서 URL | 특징 |
|--------|----------|------|
| **SendGrid** | https://docs.sendgrid.com/api-reference | 글로벌 서비스, 템플릿 지원 |
| **AWS SES** | https://docs.aws.amazon.com/ses/latest/APIReference/ | 저비용 대량 발송 |
| **Mailgun** | https://documentation.mailgun.com/docs/mailgun/api-reference/ | 개발자 친화적 |

**사용 용도**: 인보이스 이메일 발송, 주문 알림, 회원가입 인증

---

## 11. 연동 우선순위 로드맵

개발 Phase별로 어떤 API를 어떤 순서로 연동해야 하는지 정리.

### Tier 1: 즉시 필요 (Phase 1-2)

```
[필수 세팅]
├── FastAPI + SQLAlchemy + Pydantic     — 백엔드 기반
├── PostgreSQL + pgvector               — 메인 DB
├── Redis                               — 캐시/세션
├── Docker + Docker Compose             — 개발 환경
├── Keycloak                            — 인증/IAM
├── GitHub Actions                      — CI/CD
│
[플랫폼 연동 — 핵심 3개]
├── 네이버 스마트스토어 Commerce API     — 주문/상품/정산
├── 카페24 Admin API                    — 주문/상품/재고
└── 쿠팡 Wing API                       — 주문/배송/정산
```

### Tier 2: 핵심 기능 (Phase 3-5)

```
[주문/배송]
├── 스마트택배 API (통합 배송 추적)
├── Aftership API (해외 배송 추적)
│
[재무/인보이스]
├── 팝빌 API (전자세금계산서)
├── 오픈뱅킹 API (은행 거래 연동)
├── 쿠콘 API (카드/금융 데이터)
├── 포트원 API (결제 처리)
├── WeasyPrint (인보이스 PDF)
├── 여신금융협회 (카드 매입)
│
[비동기 처리]
├── Apache Kafka
└── Celery + Redis
```

### Tier 3: 커뮤니케이션 (Phase 7)

```
[채팅/알림]
├── Socket.IO + python-socketio
├── MongoDB + Motor (채팅 저장)
├── Firebase Cloud Messaging (푸시)
├── MinIO (파일 저장)
└── SendGrid/SES (이메일)
```

### Tier 4: AI/ML (Phase 8)

```
[AI 엔진]
├── LangGraph + LangChain
├── OpenAI API (GPT-4o + Embeddings)
├── Anthropic Claude API (대안 LLM)
├── pgvector (벡터 검색 — Phase 1에서 설치 완료)
├── CLOVA OCR (인보이스 OCR)
└── scikit-learn / Prophet (ML 분석)
```

### Tier 5: 프론트엔드 + DevOps (Phase 9-10)

```
[Frontend]
├── Next.js 15 + React + TypeScript
├── shadcn/ui + Tailwind CSS
├── Zustand + TanStack Query
├── Recharts + TanStack Table
├── Playwright (E2E)
│
[DevOps]
├── Kubernetes + Helm
├── Prometheus + Grafana + Loki + Jaeger
└── Trivy (보안 스캔)
```

### Tier 6: 확장 플랫폼 (Phase 2 이후 점진 추가)

```
[추가 이커머스]
├── 11번가 셀러 API
├── G마켓/옥션 ESM Plus API
├── 티몬 파트너 API
└── 위메프 파트너 API
```

---

## 12. API 인증 방식 요약

| API | 인증 방식 | 키/토큰 관리 |
|-----|----------|-------------|
| 스마트스토어 | OAuth 2.0 (Client Credentials) | Access Token 자동 갱신 |
| 카페24 | OAuth 2.0 (Authorization Code) | Refresh Token으로 갱신 |
| 쿠팡 | HMAC-SHA256 서명 | Access Key + Secret Key 보관 |
| 11번가 | API Key (Header) | 키 보관 |
| G마켓/옥션 | OAuth 2.0 | Access Token 갱신 |
| 오픈뱅킹 | OAuth 2.0 | 사용자별 동의 + 토큰 |
| 팝빌 | 링크아이디 + 비밀키 | 키 보관 |
| 포트원 | API Key + Secret | Bearer Token |
| 토스페이먼츠 | Basic Auth (Secret Key) | Base64 인코딩 |
| OpenAI | API Key (Bearer) | 환경변수 관리 |
| Anthropic | API Key (x-api-key) | 환경변수 관리 |
| Firebase | 서비스 계정 JSON | 시크릿 파일 관리 |
| Keycloak | Client ID + Secret | 서비스별 클라이언트 |

> 모든 API 키는 **환경변수** 또는 **Vault**로 관리. 코드에 직접 하드코딩 금지.

---

## 13. Rate Limit 정리

| API | Rate Limit | 초과 시 처리 |
|-----|-----------|-------------|
| 스마트스토어 | 30 req/sec | 429 → Exponential Backoff |
| 카페24 | 100 req/min | 429 → Queue + Retry |
| 쿠팡 | 60 req/min | 429 → Queue + Retry |
| 11번가 | 5 req/sec | 429 → Throttling |
| OpenAI | 티어별 TPM/RPM | 429 → Retry-After 헤더 확인 |
| Anthropic | 티어별 RPM | 429 → Retry-After |
| 오픈뱅킹 | 계약별 상이 | 429 → Backoff |
| 스마트택배 | 100 req/day (무료) | 일일 한도 모니터링 |

**공통 처리 패턴**:
```python
# Retry with Exponential Backoff + Jitter
retry_delays = [1, 2, 4, 8, 16]  # seconds
for delay in retry_delays:
    response = await call_api()
    if response.status != 429:
        break
    await asyncio.sleep(delay + random.uniform(0, 1))
```

---

## 14. 개발 환경 세팅 체크리스트

### 공통 환경

- [ ] Python 3.11+ 설치
- [ ] Node.js 20+ 설치
- [ ] Docker Desktop 설치
- [ ] Git 설정 완료
- [ ] IDE 설정 (VSCode / PyCharm)

### API 키 발급

- [ ] 네이버 커머스 API 앱 등록 + Client ID/Secret 발급
- [ ] 카페24 앱 등록 + Client ID/Secret 발급
- [ ] 쿠팡 Wing API Access Key + Secret Key 발급
- [ ] OpenAI API Key 발급 + 결제 수단 등록
- [ ] Anthropic API Key 발급 (선택)
- [ ] 팝빌 계정 생성 + 링크아이디/비밀키 발급
- [ ] 스마트택배 API Key 발급
- [ ] Firebase 프로젝트 생성 + 서비스 계정 키 다운로드
- [ ] SendGrid API Key 발급 (또는 AWS SES 설정)
- [ ] MinIO Access Key 설정 (로컬 Docker)

### 로컬 인프라 기동

```bash
# Docker Compose로 로컬 인프라 시작
docker compose -f docker-compose.dev.yml up -d

# 확인할 서비스:
# - PostgreSQL (5432) + pgvector 확장 활성화
# - MongoDB (27017)
# - Redis (6379)
# - Kafka + Zookeeper (9092)
# - Keycloak (8080)
# - MinIO (9000)
```

### 환경변수 템플릿

```env
# .env.example — 복사하여 .env 생성 후 값 채우기

# === 이커머스 플랫폼 ===
NAVER_CLIENT_ID=
NAVER_CLIENT_SECRET=
CAFE24_CLIENT_ID=
CAFE24_CLIENT_SECRET=
CAFE24_MALL_ID=
COUPANG_ACCESS_KEY=
COUPANG_SECRET_KEY=
COUPANG_VENDOR_ID=

# === 금융/세무 ===
POPBILL_LINK_ID=
POPBILL_SECRET_KEY=
OPENBANKING_CLIENT_ID=
OPENBANKING_CLIENT_SECRET=

# === 결제 ===
PORTONE_API_KEY=
PORTONE_API_SECRET=

# === AI/ML ===
OPENAI_API_KEY=
ANTHROPIC_API_KEY=

# === 인프라 ===
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/ecommerce_os
MONGODB_URL=mongodb://localhost:27017/chat
REDIS_URL=redis://localhost:6379/0
KAFKA_BOOTSTRAP_SERVERS=localhost:9092

# === 인증 ===
KEYCLOAK_URL=http://localhost:8080
KEYCLOAK_REALM=ecommerce-os
KEYCLOAK_CLIENT_ID=
KEYCLOAK_CLIENT_SECRET=

# === 파일 저장소 ===
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=
MINIO_SECRET_KEY=
MINIO_BUCKET=ecommerce-os

# === 알림 ===
SENDGRID_API_KEY=
FIREBASE_CREDENTIALS_PATH=./firebase-service-account.json

# === 배송 추적 ===
SWEETTRACKER_API_KEY=
AFTERSHIP_API_KEY=

# === OCR ===
CLOVA_OCR_API_URL=
CLOVA_OCR_SECRET_KEY=
```

---

**문서 끝**

> 이 문서는 프로젝트 진행에 따라 지속적으로 업데이트합니다.
> 새로운 API 연동 시 해당 섹션에 추가하고, 변경 사항이 있으면 Last Updated 날짜를 갱신합니다.
