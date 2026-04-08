# AirCloud Docs 프로젝트 규칙

## 프로젝트 구조

```
docs/
├── CLAUDE.md                           # 이 파일
├── references/
│   └── api-reference.md                # API 스펙 참고 자료 (마크다운 원본)
└── aircloud-sp/                        # Mintlify 문서 사이트 루트
    ├── docs.json                       # 네비게이션, 테마, 로고 등 전체 설정
    ├── api-reference/
    │   ├── openapi.json                # OpenAPI 3.1.0 스펙 (영어)
    │   ├── openapi-ko.json             # OpenAPI 3.1.0 스펙 (한국어)
    │   ├── introduction.mdx            # API 개요 페이지 (영어)
    │   ├── endpoint/                   # 영어 엔드포인트 MDX 페이지
    │   │   ├── list-endpoints.mdx
    │   │   ├── get-endpoint.mdx
    │   │   └── ...
    │   └── ko/
    │       ├── introduction.mdx        # API 개요 페이지 (한국어)
    │       └── endpoint/              # 한국어 엔드포인트 MDX 페이지
    │           └── ...
    ├── docs/                           # 일반 문서 (영어)
    │   ├── index.mdx
    │   ├── quickstart.mdx
    │   └── ko/                         # 일반 문서 (한국어)
    ├── models/
    │   ├── index.mdx                   # 모델 페이지 (영어)
    │   └── ko/
    │       └── index.mdx               # 모델 페이지 (한국어)
    ├── release-notes/
    │   ├── index.mdx                   # 릴리즈 노트 (영어)
    │   └── ko/
    │       └── index.mdx               # 릴리즈 노트 (한국어)
    ├── logo/
    └── images/
```

## API 문서 핵심 규칙

### openapi 스펙 파일은 영어/한국어 분리 (파일명으로 구분)

- **영어**: `api-reference/openapi.json`
- **한국어**: `api-reference/openapi-ko.json`
- 두 파일은 **같은 폴더**(`api-reference/`)에 위치하되 **파일명이 다름**
- 두 파일의 `paths`, `components/schemas` 구조(필드명, 타입, example 등)는 **반드시 동일**하게 유지
- 차이나는 것은 `description`, `summary`, `tags.description` 등 **텍스트 설명만**
- API 추가/변경 시 **양쪽 파일 모두 수정** 필수

> **주의: `openapi.json`이라는 이름의 파일이 하위 폴더에 여러 개 있으면 Mintlify가 충돌을 일으킴.**
> 같은 이름(`openapi.json`)을 `api-reference/`와 `api-reference/ko/`에 각각 두면
> Mintlify가 둘 다 스캔하여 한국어 스펙이 영어 페이지에도 적용되는 버그가 발생함.
> 반드시 **파일명을 다르게** (`openapi.json` vs `openapi-ko.json`) 해야 함.

### MDX 파일은 영어/한국어 각각 생성 — 양쪽 모두 openapi 파일 경로 명시 필수

영어 MDX → `openapi.json` 명시:

```mdx
---
title: "List Endpoints"
openapi: "openapi.json GET /endpoints"
---
```

한국어 MDX → `openapi-ko.json` 명시:

```mdx
---
title: "엔드포인트 목록 조회"
openapi: "openapi-ko.json GET /endpoints"
---
```

> **중요**: 영어/한국어 **양쪽 모두** `openapi:` 값에 파일명을 반드시 명시해야 함.
> 파일명을 생략하면 Mintlify가 임의로 openapi 파일을 선택하여 잘못된 언어가 표시될 수 있음.

## 신규 API 추가 절차

### 1. 영어 openapi.json에 엔드포인트 추가

`api-reference/openapi.json`의 `paths`에 새 경로 추가:
- `tags`: 사이드바 그룹명 (Endpoints, Replicas, Logs, Authentication)
- `summary`: 영어 제목
- `description`: 영어 설명
- `operationId`: 고유 ID
- `parameters`: path/query 파라미터 (description 영어, example 포함)
- `requestBody`: POST/PATCH의 경우 요청 바디 스키마
- `responses`: 응답 스키마 (`$ref`로 `components/schemas` 참조)

스키마는 `components/schemas`에 정의하고 `$ref`로 참조.

### 2. 한국어 openapi-ko.json에 동일 엔드포인트 추가

`api-reference/openapi-ko.json`에 **동일한 구조**로 추가하되, description/summary만 한국어로 작성.

### 3. 영어 MDX 파일 생성

```
파일: api-reference/endpoint/{name}.mdx
```
```mdx
---
title: "English Title"
openapi: "openapi.json METHOD /path"
---
```

### 4. 한국어 MDX 파일 생성

```
파일: api-reference/ko/endpoint/{name}.mdx
```
```mdx
---
title: "한국어 제목"
openapi: "openapi-ko.json METHOD /path"
---
```

### 5. docs.json 네비게이션에 페이지 등록

영어 탭과 한국어 탭 **양쪽 모두**에 추가:

```json
// 영어 (language: "en")
{
  "tab": "API Reference",
  "groups": [
    {
      "group": "Endpoints",
      "pages": [
        "api-reference/endpoint/list-endpoints",
        "api-reference/endpoint/new-endpoint"        // 추가
      ]
    }
  ]
}

// 한국어 (language: "ko")
{
  "tab": "API 레퍼런스",
  "groups": [
    {
      "group": "엔드포인트",
      "pages": [
        "api-reference/ko/endpoint/list-endpoints",
        "api-reference/ko/endpoint/new-endpoint"     // 추가
      ]
    }
  ]
}
```

### 6. 새 그룹 추가 시

`openapi.json`과 `openapi-ko.json` 양쪽의 `tags`에 새 태그를 추가하고, docs.json의 영어/한국어 양쪽에 새 group 추가.

## docs.json 네비게이션 구조

```json
{
  "navigation": {
    "languages": [
      {
        "language": "en",
        "tabs": [
          { "tab": "Documentation", "groups": [...] },
          { "tab": "Models", "groups": [...] },
          { "tab": "API Reference", "groups": [...] },
          { "tab": "Release Notes", "groups": [...] }
        ]
      },
      {
        "language": "ko",
        "tabs": [
          { "tab": "문서", "groups": [...] },
          { "tab": "모델", "groups": [...] },
          { "tab": "API 레퍼런스", "groups": [...] },
          { "tab": "릴리즈 노트", "groups": [...] }
        ]
      }
    ]
  }
}
```

- `tabs`: 상단 탭 네비게이션 (Documentation | Models | API Reference | Release Notes)
- `groups`: 사이드바 그룹 (Overview, Endpoints, Replicas 등)
- `pages`: 그룹 내 페이지 경로 (확장자 없이)

> **주의: `dropdown` 타입을 사용하지 말 것.**
> `"dropdown"`은 하위에 `"tabs"`를 포함하는 구조인데, `"groups"`를 직접 넣으면
> Mintlify가 네비게이션 전체를 파싱하지 못해 탭이 모두 사라지는 버그가 발생함.
> 반드시 `"tab"`을 사용할 것.

## 로컬 개발

```bash
cd docs/aircloud-sp
npx mintlify@latest dev --port 3000
```

- hot reload 지원되지만, openapi.json 변경 시 서버 재시작 필요할 수 있음
- openapi.json 변경 후 반영 안되면: 서버 재시작 + 브라우저 하드 리프레시 (Ctrl+Shift+R)

## 체크리스트

새 API 추가 시 확인:
- [ ] `api-reference/openapi.json`에 path, schema 추가 (영어)
- [ ] `api-reference/openapi-ko.json`에 동일 path, schema 추가 (한국어)
- [ ] 영어 MDX 파일 생성 (`api-reference/endpoint/`) — openapi 값에 `openapi.json` 명시
- [ ] 한국어 MDX 파일 생성 (`api-reference/ko/endpoint/`) — openapi 값에 `openapi-ko.json` 명시
- [ ] docs.json 영어 탭에 페이지 등록
- [ ] docs.json 한국어 탭에 페이지 등록
- [ ] 로컬에서 영어/한국어 양쪽 확인

## API 소스 코드 위치

API 스펙의 원본 소스 코드:
- 백엔드 repo: `nadongjun/aircloud-backend`
- External API 라우트: `app/domains/endpoints/presentation/api/external_route.py`
- Base URL: `https://external.aieev.cloud:5007/external/api/v1`
- 인증: API Key (Istio External Auth → x-org-id, x-project-id, x-key-id 헤더 주입)
