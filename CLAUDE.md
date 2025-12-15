# Gold-River-MCP

## 프로젝트 개요

**Gold-River-MCP**는 서울 공인중개사무소의 업무를 혁신하기 위한 Claude Code + MCP(Model Context Protocol) 기반의 AI 에이전트 시스템입니다.

### 프로젝트 목표
부모님이 운영하시는 공인중개사무소의 **매출 증대**와 **업무 효율화**를 위한 '한국형 부동산 AI 에이전트' 구축

### 핵심 가치 (우선순위 순)
1. **매물-고객 매칭 시스템** - 집주인 조건 ↔ 세입자 조건 자동 매칭 (★ 최우선)
2. **매물/고객 통합 관리** - 현재 엑셀/수기로 관리하는 데이터를 DB화
3. **마케팅 자동화** - 네이버 SEO 최적화 블로그 콘텐츠 자동 생성
4. **리스크 분석** - 전세 사기 예방을 위한 권리분석 자동화
5. **데이터 통합** - 파편화된 부동산 데이터를 하나의 인터페이스로 통합

---

## 현업 Pain Point (부모님 피드백 기반)

### 핵심 문제: A-B-C 소통 실패
```
A (공인중개사) ─── B (손님/세입자) ─── C (집주인)

문제: 집주인 조건과 세입자 조건이 맞지 않아 중개 실패
예시: 집주인 "반려동물 X" vs 세입자 "고양이 1마리"
     → 현재는 전화 통화 후에야 알게 됨 (시간 낭비)
```

### 집주인(C) 조건 유형
- 성별 제한: 여성만, 남성만
- 반려동물: 불가, 소형만 가능
- 거주 기간: 오래 살 사람 (월세의 경우)
- 나이대 제한
- 직업 제한: 직장인만 등

### 손님(B) 정보 유형
- 예산: 보증금/월세, 대출 여부, 보증보험 가입 여부
- 직장 위치: 출퇴근 고려
- 가족 구성: 1인, 신혼, 자녀 유무
- 반려동물 유무
- 라이프스타일: 주차 필요, 학군 중요 등

### 현재 데이터 관리 상태
- **엑셀**: 일부 매물/고객 정보
- **수기 장부**: 상당수 정보
- **부동산 프로그램**: 광고용만, 전체 매물 보관 안함
- **문제점**: 내 매물을 통합 보관하는 시스템이 없음!

### 사용 중인 외부 프로그램
- 네이버 부동산 (유료) - 광고/매물 등록
- 한방 (중개사 협회) - 계약서 작성
- 날개 (유료) - 은평구 인기, 매물 관리

---

## 시스템 아키텍처 (수정됨)

### 4대 핵심 모듈

```
┌─────────────────────────────────────────────────────────────────┐
│                    Claude Desktop App (UI)                       │
│                    사용자 인터페이스 (부모님)                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │ MCP Protocol
┌───────────────────────────┴─────────────────────────────────────┐
│                     MCP Host (Claude)                            │
│              자연어 이해, 추론, 매칭 로직                          │
└─────┬──────────────┬──────────────┬──────────────┬──────────────┘
      │              │              │              │
┌─────▼─────┐ ┌──────▼──────┐ ┌─────▼─────┐ ┌─────▼─────┐
│  Matching │ │   CRM DB    │ │   Data    │ │    Mkt    │
│   Engine  │ │  매물/고객   │ │ 공공데이터 │ │  마케팅   │
│ 조건 매칭  │ │  통합 관리   │ │ API 연동  │ │ 블로그    │
└─────┬─────┘ └──────┬──────┘ └─────┬─────┘ └─────┬─────┘
      │              │              │              │
      └──────────────┴──────────────┴──────────────┘
                            │
                     ┌──────▼──────┐
                     │  Local DB   │
                     │   SQLite    │
                     └─────────────┘
```

### 모듈 상세

| 모듈 | 역할 | 우선순위 |
|------|------|----------|
| **Matching Engine** | 집주인 조건 ↔ 세입자 조건 자동 매칭 | ★★★ 1순위 |
| **CRM DB** | 매물/고객 정보 통합 관리 (엑셀/수기 대체) | ★★★ 1순위 |
| **Data Server** | 공공데이터(실거래가, 대장) 조회 | ★★ 2순위 |
| **Marketing Server** | 네이버 블로그 원고 자동 생성 | ★★ 2순위 |
| **Doc Server** | 등기부등본 OCR/권리분석 | ★ 3순위 |

---

## 데이터 모델 설계 (실제 현업 데이터 기반)

### 매물 (Property) 테이블
```python
class Property:
    id: str                    # 매물 고유 ID
    title: str                 # 제목 (예: "중앙시장 코너")

    # 위치 정보
    district: str              # 구 (예: "은평구")
    dong: str                  # 동 (예: "응암동")
    address: str               # 상세주소

    # 매물 유형
    property_type: str         # 아파트/빌라/오피스텔/원룸/상가/사무실
    trade_type: str            # 매매/전세/월세

    # 가격 정보
    deposit: int               # 보증금 (만원 단위)
    monthly_rent: int | None   # 월세 (만원 단위, 월세인 경우)
    maintenance_fee: int       # 관리비 (만원 단위)
    premium: int | None        # 권리금 (상가의 경우)

    # 면적/구조
    area_sqm: float            # 전용면적 (㎡)
    room_count: int            # 방 개수 ★ 손님 질문 1순위
    bathroom_count: int        # 화장실 개수
    floor: str                 # 층수 (예: "1층", "3/5층")
    total_floor: int           # 건물 총 층수

    # 매물 특성 (용어집 기반)
    direction: str | None      # 방향: 남향/남동향/남서향/동향/서향/북향
    structure: str | None      # 구조: 판상형/타워형
    expansion: str | None      # 확장여부: 확장형/비확장(기본형)
    renovation: str | None     # 수리상태: 올수리/특올수리/부분수리/기본상태

    # 편의시설 ★ 손님 질문 2~3순위
    parking: bool              # 주차 가능 여부
    parking_count: int | None  # 주차 대수
    near_station: bool         # 역세권 여부
    station_distance: int | None  # 역까지 도보 분

    # 집주인 정보
    owner_name: str            # 집주인명
    owner_phone: str           # 집주인 연락처

    # ★★★ 집주인 조건 (매칭의 핵심) ★★★
    owner_conditions: dict
    # {
    #   "gender_requirement": None | "여성만" | "남성만",
    #   "pet_allowed": True | False | "소형만",
    #   "min_stay_months": 24,        # 최소 거주 기간 (월세 시)
    #   "age_requirement": None | "20-40" | "직장인만",
    #   "occupation_requirement": None | "직장인" | "학생불가",
    #   "other": "..."                # 기타 조건 메모
    # }

    # 상태
    status: str                # 광고중/계약진행중/계약완료/보류
    notes: str                 # 주요내용/메모 (예: "코너 응암시장 입지 아주저렴 급")

    # 메타
    source: str | None         # 출처: 직접/한방/날개/네이버부동산
    created_at: datetime
    updated_at: datetime
```

### 고객 (Customer) 테이블
```python
class Customer:
    id: str                    # 고객 고유 ID
    name: str                  # 이름
    phone: str                 # 연락처

    # ★★★ 손님 조건 (업무 플로우 순서대로) ★★★

    # 1. 방 개수 (1순위 질문)
    room_count_min: int        # 최소 방 개수
    room_count_max: int | None # 최대 방 개수

    # 2. 주차 (2순위 질문)
    need_parking: bool         # 주차 필요 여부
    car_count: int             # 차량 대수

    # 3. 역세권 (3순위 질문)
    need_station: bool         # 역세권 필요 여부
    max_station_distance: int | None  # 역까지 최대 도보 분

    # 4. 대출 (4순위 질문)
    loan_needed: bool          # 대출 필요 여부
    loan_available: bool       # 대출 가능 여부
    guarantee_insurance: bool  # 보증보험 가입 가능 여부

    # 5. 층수 (5순위 - 어르신은 저층)
    floor_preference: str | None  # "저층" | "중층" | "고층" | "상관없음"
    avoid_top_floor: bool      # 탑층 기피 여부
    avoid_first_floor: bool    # 1층 기피 여부

    # 예산
    trade_type: str            # 매매/전세/월세
    budget_deposit_max: int    # 보증금 최대 (만원)
    budget_monthly_max: int | None  # 월세 최대 (만원)

    # 희망 지역
    preferred_districts: list[str]  # 희망 구 (예: ["은평구", "서대문구"])
    preferred_dongs: list[str] | None  # 희망 동 (더 구체적)
    workplace: str | None      # 직장 위치 (출퇴근 고려)

    # 개인 정보 (매칭용)
    family_type: str           # 1인/신혼/자녀있음/어르신
    gender: str | None         # 성별 (집주인 조건 매칭용)
    age_group: str | None      # 나이대 (20대/30대/40대/50대이상)
    occupation: str | None     # 직업 (직장인/자영업/학생/무직)

    # 반려동물
    has_pet: bool              # 반려동물 유무
    pet_type: str | None       # 반려동물 종류 (개/고양이/기타)
    pet_size: str | None       # 크기 (소형/중형/대형)

    # 기타
    move_in_date: date | None  # 입주 희망일
    notes: str                 # 기타 메모
    status: str                # 상담중/매물소개중/계약완료/보류
    created_at: datetime
```

### 매칭 로직 (MatchResult)
```python
class MatchResult:
    property_id: str
    customer_id: str
    match_score: int           # 0-100

    # 매칭 결과 상세
    is_compatible: bool        # 최종 매칭 가능 여부
    hard_mismatches: list[str] # 절대 불가 사유 (예: ["반려동물 불가"])
    soft_mismatches: list[str] # 협의 가능 사유 (예: ["예산 500만원 초과"])

    # 매칭 점수 상세
    score_breakdown: dict
    # {
    #   "budget": 100,      # 예산 매칭
    #   "location": 80,     # 위치 매칭
    #   "room": 100,        # 방 개수 매칭
    #   "parking": 100,     # 주차 매칭
    #   "floor": 50,        # 층수 매칭
    #   "conditions": 0,    # 집주인 조건 매칭 (반려동물 등)
    # }
```

### 매칭 규칙 (Matching Rules)
```python
# Hard Rules (하나라도 실패하면 매칭 불가)
HARD_RULES = [
    "예산 초과 20% 이상",
    "반려동물 불가인데 반려동물 있음",
    "성별 제한 불일치",
    "방 개수 부족",
]

# Soft Rules (협의 가능, 점수 감점)
SOFT_RULES = [
    "예산 초과 20% 미만",  # -20점
    "역세권 아님",          # -10점
    "주차 부족",           # -15점
    "희망 층수 불일치",     # -5점
]
```

---

## 핵심 기능 명세

### 모듈 0: 매물-고객 매칭 시스템 (★ 최우선)

#### 주요 도구 (Tools)
1. **`register_property(data: dict)`**
   - 새 매물 등록 (집주인 조건 포함)

2. **`register_customer(data: dict)`**
   - 새 고객 등록 (희망 조건 포함)

3. **`find_matching_properties(customer_id: str)`**
   - 특정 고객에게 맞는 매물 자동 검색
   - 조건 불일치 사유도 함께 반환

4. **`find_matching_customers(property_id: str)`**
   - 특정 매물에 맞는 고객 자동 검색

5. **`check_compatibility(property_id: str, customer_id: str)`**
   - 특정 매물-고객 조합의 호환성 체크
   - 반환: 매칭 점수 + 불일치 사유

#### 사용자 워크플로우
```
1. 부모님: "손님 김철수씨, 전세 3억 예산, 고양이 있음, 강남역 출근"
2. Claude → register_customer() 호출
3. Claude → find_matching_properties() 호출
4. Claude: "3건의 매물이 조건에 맞습니다.
           단, 래미안은 반려동물 불가라 제외했습니다."
5. 부모님: 헛걸음 없이 바로 매물 소개 가능
```

### 모듈 1: 지능형 매물 마케팅 자동화

#### 주요 도구 (Tools)
1. **`get_trending_keywords(location: str)`**
   - 특정 동네의 네이버 연관 검색어 추출

2. **`generate_blog_content(property_id: str)`**
   - DB에 저장된 매물 정보로 블로그 원고 생성
   - 네이버 SEO 최적화 ('해요체' + 이모지)

### 모듈 2: 실시간 권리분석 및 리스크 진단

#### 주요 도구 (Tools)
1. **`analyze_registry_pdf(file_path: str)`**
   - 등기부등본 PDF OCR 분석

2. **`calculate_safety_ratio(market_price: int, debts: list, deposit: int)`**
   - 전세 안전 비율 계산
   - 70% 이상: 주의, 80% 이상: 위험

#### 법적 주의사항
- 결과물은 **"참고용 데이터 정리 자료"**로만 표시
- "안전하다" 등 단정적 표현 금지

### 모듈 3: 로컬 마켓 인텔리전스

#### 주요 도구 (Tools)
1. **`get_apt_transaction_history(district_code: str, year_month: str)`**
   - 국토교통부 아파트 실거래가 API 조회

---

## 디렉토리 구조

```
Gold-River-MCP/
├── CLAUDE.md                    # 이 파일 (Claude Code 컨텍스트)
├── README.md                    # 프로젝트 소개
├── pyproject.toml               # Python 프로젝트 설정
├── requirements.txt             # Python 의존성
│
├── src/
│   ├── __init__.py
│   │
│   ├── models/                  # 데이터 모델 (★ 새로 추가)
│   │   ├── __init__.py
│   │   ├── property.py          # 매물 모델
│   │   ├── customer.py          # 고객 모델
│   │   └── matching.py          # 매칭 결과 모델
│   │
│   ├── servers/                 # MCP 서버들
│   │   ├── __init__.py
│   │   ├── matching_server.py   # 매칭 엔진 서버 (★ 새로 추가)
│   │   ├── crm_server.py        # CRM 서버 (★ 새로 추가)
│   │   ├── data_server.py       # 공공데이터 API 서버
│   │   └── marketing_server.py  # 마케팅 자동화 서버
│   │
│   ├── tools/                   # MCP 도구(Tool) 정의
│   │   ├── __init__.py
│   │   ├── matching.py          # 매칭 도구 (★ 새로 추가)
│   │   ├── crm.py               # CRM 도구 (★ 새로 추가)
│   │   ├── real_estate_api.py   # 실거래가 조회
│   │   └── blog_generator.py    # 블로그 생성
│   │
│   ├── resources/               # MCP 리소스
│   │   └── seoul_district_codes.json
│   │
│   └── utils/
│       ├── __init__.py
│       └── money_converter.py
│
├── tests/
├── data/
│   ├── cache/
│   └── db/
│       └── gold_river.db        # 매물/고객 통합 DB
│
└── templates/
    └── blog_post.jinja2
```

---

## 개발 규칙 및 컨벤션

### 코드 스타일
- **Python 버전**: 3.11+
- **포맷터**: Black
- **린터**: Ruff
- **타입 힌트**: 필수 사용
- **독스트링**: Google 스타일

### 네이밍 컨벤션
- **파일명**: snake_case (예: `matching_server.py`)
- **클래스명**: PascalCase (예: `MatchingEngine`)
- **함수명**: snake_case (예: `find_matching_properties`)
- **상수**: UPPER_SNAKE_CASE (예: `DEFAULT_MATCH_THRESHOLD`)

### Git 커밋 메시지
```
<type>(<scope>): <subject>

types: feat, fix, docs, style, refactor, test, chore
scope: matching, crm, data, mkt, utils 등
```

---

## 부동산 용어 사전 (코드에서 사용)

### 수리 상태 (renovation)
| 값 | 의미 |
|---|---|
| `올수리` | 도배, 장판, 화장실, 싱크대, 전등 교체 |
| `특올수리` | 올수리 + 샷시 + 문짝 + 단열 (신축급) |
| `부분수리` | 도배/장판 일부만 교체 |
| `기본상태` | 수리 안됨 |

### 구조/방향
| 값 | 의미 |
|---|---|
| `판상형` | ㅡ자 구조, 맞바람 환기 (한국인 선호) |
| `타워형` | Y자/ㅁ자 구조, 뷰 좋음 |
| `확장형` | 베란다 터서 방/거실 넓힘 |
| `남향` | 하루종일 해 잘듦 (최고 선호) |

### 입지 관련
| 값 | 의미 |
|---|---|
| `역세권` | 지하철역 도보 5~10분 |
| `초품아` | 초등학교 품은 아파트 |
| `학세권` | 학교/학원가 인접 |

### 거래 용어
| 값 | 의미 |
|---|---|
| `급매` | 시세보다 저렴 (집주인 급함) |
| `세안고` | 세입자 계약 승계 조건 매매 |
| `네고` | 가격 협상 가능 |

### 매물 상태 (status)
| 값 | 의미 |
|---|---|
| `광고중` | 현재 광고 중인 매물 |
| `계약진행중` | 계약 진행 중 |
| `계약완료` | 거래 완료 |
| `보류` | 일시 중단 |

### 고객 상태 (status)
| 값 | 의미 |
|---|---|
| `상담중` | 첫 상담 진행 중 |
| `매물소개중` | 매물 소개 단계 |
| `계약완료` | 계약 성사 |
| `보류` | 연락 두절 또는 일시 중단 |

---

## API 키 및 환경변수

### 필요한 API 키
```env
# .env 파일 (절대 커밋하지 않음!)

# 공공데이터포털 (필수)
DATA_GO_KR_API_KEY=your_decoded_service_key

# Google Cloud Vision (OCR용, 선택)
GOOGLE_CLOUD_VISION_API_KEY=your_key

# 네이버 검색광고 API (선택)
NAVER_AD_API_KEY=your_key
NAVER_AD_SECRET_KEY=your_secret
```

---

## 개발 로드맵 (수정됨)

### Phase 1: CRM + 매칭 시스템 (1~2주) ★ 최우선
- [ ] SQLite 데이터베이스 스키마 설계
- [ ] Property, Customer 모델 구현
- [ ] 매물/고객 CRUD 도구 구현
- [ ] 매칭 엔진 로직 구현
- [ ] 기본 MCP 서버 구축

### Phase 2: 공공데이터 + 마케팅 (3~4주)
- [ ] 공공데이터포털 API 연동
- [ ] 실거래가 조회 기능
- [ ] 블로그 자동 생성 기능

### Phase 3: 권리분석 + 고도화 (5~6주)
- [ ] OCR 연동
- [ ] 등기부등본 파싱
- [ ] 안전 비율 계산
- [ ] Claude Desktop 연동 및 부모님 교육

### Phase 4: 네트워크 연계 (향후)
- [ ] 법무사/세무사/이사업체 연락처 DB
- [ ] 원스탑 서비스 추천 기능

---

## 주의사항 (개인정보 및 보안)

### 개인정보보호법(PIPA) 준수
- 고객 전화번호, 주민번호는 **절대 Claude에게 전송 금지**
- 로컬 DB에만 민감 정보 저장
- 등기부등본 분석 시 주민번호 마스킹 필수

### 데이터 처리 원칙
```python
def mask_phone(phone: str) -> str:
    """010-1234-5678 → 010-****-5678"""
    return phone[:4] + "****" + phone[-4:]
```

---

## Claude Code 활용 팁

### 유용한 MCP 도구들
- **context7**: 라이브러리 문서 조회
- **playwright**: 웹 자동화 및 테스트
- **github**: 레포지토리 관리
- **exa**: 웹 검색 및 코드 컨텍스트

### 권장 개발 워크플로우
1. **계획 수립**: EnterPlanMode로 구현 계획 먼저 수립
2. **구현**: 단계별로 TodoWrite로 진행상황 추적
3. **리뷰**: code-reviewer 에이전트로 품질 검증
4. **문서화**: 변경사항 CLAUDE.md에 반영

---

## 연락처 및 참고자료

### 기술 문서
- [MCP 공식 문서](https://modelcontextprotocol.io)
- [공공데이터포털 API 가이드](https://data.go.kr)

### 프로젝트 관리
- **GitHub**: https://github.com/seosamuel02/Gold-River-MCP
- **개발자**: seosamuel02

---

## 변경 이력

### 2025-01-12 (2차): 실제 데이터 샘플 기반 모델 정교화
- **매물 데이터 구조**: 실제 "한방/날개" 시스템 데이터 형식 반영
- **업무 플로우**: 손님 상담 시 질문 순서 반영 (방개수→주차→역세권→대출→층수)
- **용어 사전 추가**: 올수리/특올수리, 판상형/타워형, 역세권 등 현업 용어
- **매칭 규칙**: Hard Rules vs Soft Rules 구분

### 2025-01-12 (1차): 부모님 피드백 반영
- **핵심 변경**: 매물-고객 매칭 시스템이 1순위로 격상
- **추가된 모듈**: Matching Engine, CRM DB
- **Pain Point 명확화**: A-B-C 소통 실패 문제가 핵심
- **현황 파악**: 현재 엑셀/수기 관리, 통합 시스템 부재
- **외부 프로그램**: 네이버부동산(유료), 한방(협회), 날개(유료) 사용 중
