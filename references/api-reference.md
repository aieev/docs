# AirCloud External API 가이드

### 개요

- AirCloud External API는 **웹 콘솔을 통하지 않고도 프로그래밍 방식으로 엔드포인트를 관리**할 수 있는 API이며 엔드포인트 상태 조회, 시작/중지, 레플리카 스케일링 등의 작업에서 사용 가능합니다.
- **본 문서는 신규 API 추가에 따라 계속 업데이트될 예정입니다.**

### 사용 가능 API 경로

*Base URL: `https://external.aieev.cloud:5007/external/api/v1`

| API | Method | 경로 | 설명 | 추가일 |
| --- | --- | --- | --- | --- |
| 상태 조회 | GET | `/endpoints/{endpoint_id}` | 엔드포인트 현재 상태 확인 | 2026-01-22 |
| 시작 | POST | `/endpoints/{endpoint_id}/start` | 비활성 엔드포인트 시작 | 2026-01-22 |
| 중지 | POST | `/endpoints/{endpoint_id}/stop` | 활성 엔드포인트 중지 | 2026-01-22 |
| 레플리카 스케일링 | POST | `/endpoints/{endpoint_id}/scale` | 레플리카 수 조정 **(활성 상태에서만 가능)** | 2026-01-22 |
| 엔드포인트 수정 | PATCH | `/endpoints/{endpoint_id}` | 엔드포인트의 일부 운영 설정 변경 | 2026-04-01 |
| 엔드포인트 목록 조회 | GET | `/endpoints` | 접근 가능한 엔드포인트 목록 조회 | 2026-04-08 |
| 레플리카 목록 조회 | GET | `/endpoints/{endpoint_id}/replicas` | 특정 엔드포인트의 실행 중 레플리카 목록 조회 | 2026-04-08 |
| 로그 파일 목록 조회 | GET | `/endpoints/{endpoint_id}/logs` | 특정 엔드포인트의 로그 파일 목록 조회 | 2026-04-08 |
| 로그 파일 내용 조회 | GET | `/endpoints/{endpoint_id}/logs/file` | 특정 로그 파일의 내용 조회 | 2026-04-08 |
| 현재 인증 컨텍스트 조회 | GET | `/me` | 현재 API key가 매핑된 org/project/user 정보 조회 | 2026-04-08 |

---

### 인증

모든 요청에 `Authorization` 헤더로 API Key를 전달해야 합니다.

```
Authorization: Bearer {api_key}
```

- 프로젝트 개요의 **API 키** 메뉴에서 키 생성
- 해당 키가 액세스 가능한 엔드포인트를 지정

---

### API 상세

---

#### 현재 인증 컨텍스트 조회

현재 사용 중인 API key가 어떤 organization, project, user에 매핑되어 있는지 조회합니다.

```
GET /me
```

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| organization_id | string | API Key에 연결된 조직 ID |
| project_id | string | API Key에 연결된 프로젝트 ID |
| api_key_id | string \| null | 인증에 사용된 API Key ID |
| user_id | string \| null | API Key를 생성한 사용자 ID |
| authenticated_via | string | 인증 방식. 항상 `"api_key"` |

##### 요청 예시

```bash
curl -X GET "https://external.aieev.cloud:5007/external/api/v1/me" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
{
  "organization_id": "org-0I2GoSBhwODG",
  "project_id": "proj-sGF4ynUPbc8H",
  "api_key_id": "9e9a776f-2ce6-42c9-ad70-c75936ce662b",
  "user_id": "6377ae7c-8986-477c-ab87-7f8eb63bf785",
  "authenticated_via": "api_key"
}
```

---

#### 엔드포인트 목록 조회

현재 API key로 접근 가능한 엔드포인트 목록을 페이지네이션으로 조회합니다.

```
GET /endpoints
```

##### Query Parameters

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
| --- | --- | --- | --- | --- |
| search | string | No | - | 엔드포인트 이름으로 검색 |
| is_active | boolean | No | - | 활성 상태로 필터링. `true`면 실행 중인 엔드포인트만, `false`면 중지된 엔드포인트만 반환 |
| page | integer | No | 1 | 페이지 번호. 1부터 시작 |
| size | integer | No | 50 | 페이지당 항목 수. 최소 1, 최대 2000 |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| items | array | 엔드포인트 객체 배열 |
| total | integer | 전체 엔드포인트 수 |
| page | integer | 현재 페이지 번호 |
| size | integer | 페이지당 항목 수 |
| pages | integer | 전체 페이지 수 |

**items 각 항목:**

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| id | string | 엔드포인트 고유 ID |
| name | string | 엔드포인트 이름 |
| endpoint_type | string | 엔드포인트 유형. `"container"` 등 |
| is_active | boolean | 현재 활성(실행 중) 여부 |
| status | string | 현재 상태. `"RUNNING"`, `"STOPPED"`, `"DEPLOYING"` 등 |
| num_replicas | integer | 현재 레플리카 수 |
| enable_autoscaling | boolean | 오토스케일링 활성화 여부 |
| instance_type_name | string \| null | 할당된 인스턴스 타입 이름. 예: `"RTX 4070 Super"` |
| serving_endpoint_url | string \| null | 엔드포인트에 요청을 보낼 수 있는 공개 URL |
| created_at | datetime | 생성 시간 (ISO 8601) |
| updated_at | datetime | 마지막 수정 시간 (ISO 8601) |

##### 요청 예시

```bash
curl -X GET "https://external.aieev.cloud:5007/external/api/v1/endpoints" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
{
  "items": [
    {
      "id": "cad59665-b77b-4cb6-9308-d276294bce79",
      "name": "cpu-bench",
      "endpoint_type": "container",
      "is_active": false,
      "status": "RUNNING",
      "num_replicas": 2,
      "enable_autoscaling": false,
      "instance_type_name": "RTX 4070 Super",
      "serving_endpoint_url": "https://ap-1.aieev.cloud/ac/7/cad59665-b77b-4cb6-9308-d276294bce79",
      "created_at": "2026-01-21T04:30:18.998248Z",
      "updated_at": "2026-04-08T03:18:17.974691Z"
    }
  ],
  "total": 1,
  "page": 1,
  "size": 50,
  "pages": 1
}
```

---

#### 엔드포인트 상태 조회

특정 엔드포인트의 현재 상태를 조회합니다. 활성 상태일 경우 레플리카 상태 요약도 포함됩니다.

```
GET /endpoints/{endpoint_id}
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| endpoint_id | string | 엔드포인트 고유 ID |
| name | string | 엔드포인트 이름 |
| endpoint_type | string | 엔드포인트 유형. `"container"` 등 |
| is_active | boolean | 현재 활성(실행 중) 여부 |
| status | string | 현재 상태. 활성 시 Ray Serve에서 실시간 상태를 가져옴 |
| num_replicas | integer | 현재 레플리카 수 |
| enable_autoscaling | boolean | 오토스케일링 활성화 여부 |
| instance_type_name | string \| null | 인스턴스 타입 이름 |
| serving_endpoint_url | string \| null | 엔드포인트 공개 URL |
| replica_status_summary | object \| null | 상태별 레플리카 수. 활성 상태에서만 제공 |
| created_at | datetime | 생성 시간 (ISO 8601) |
| updated_at | datetime | 수정 시간 (ISO 8601) |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |

##### 요청 예시

```bash
curl -X GET "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "name": "aieev-endpoint",
  "endpoint_type": "container",
  "is_active": true,
  "status": "RUNNING",
  "num_replicas": 2,
  "enable_autoscaling": false,
  "instance_type_name": "RTX 5090",
  "serving_endpoint_url": "https://ap-1.aieev.cloud/ac/7/51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "replica_status_summary": {
    "RUNNING": 2
  },
  "created_at": "2026-01-20T10:00:00Z",
  "updated_at": "2026-01-22T15:30:00Z"
}
```

---

#### 엔드포인트 시작

비활성 상태의 엔드포인트를 시작합니다. **레플리카 스케일링 전에 반드시 엔드포인트가 활성화되어 있어야 합니다.**

```
POST /endpoints/{endpoint_id}/start
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| endpoint_id | string | 엔드포인트 고유 ID |
| is_active | boolean | 작업 후 활성 상태. 시작 성공 시 `true` |
| message | string | 결과 메시지 |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |
| 500 | 시작 실패 (잔액 부족, 리소스 할당 실패 등) |

##### 요청 예시

```bash
curl -X POST "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}/start" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "is_active": true,
  "message": "Endpoint started successfully"
}
```

이미 활성 상태인 경우:

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "is_active": true,
  "message": "Endpoint is already active"
}
```

---

#### 엔드포인트 중지

활성 상태의 엔드포인트를 중지합니다.

```
POST /endpoints/{endpoint_id}/stop
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| endpoint_id | string | 엔드포인트 고유 ID |
| is_active | boolean | 작업 후 활성 상태. 중지 성공 시 `false` |
| message | string | 결과 메시지 |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |
| 500 | 중지 실패 |

##### 요청 예시

```bash
curl -X POST "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}/stop" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "is_active": false,
  "message": "Endpoint stopped successfully"
}
```

이미 중지 상태인 경우:

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "is_active": false,
  "message": "Endpoint is already stopped"
}
```

---

#### 레플리카 스케일링

활성화된 엔드포인트의 레플리카 수를 조정합니다. **오토스케일링이 비활성화된 상태에서만 사용 가능합니다.**

```
POST /endpoints/{endpoint_id}/scale
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### Request Body

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| num_replicas | integer | Yes | 변경할 레플리카 수. 최소 1, 최대 100 |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| endpoint_id | string | 엔드포인트 고유 ID |
| previous_replicas | integer | 변경 전 레플리카 수 |
| current_replicas | integer | 변경 후 레플리카 수 |
| message | string | 결과 메시지 |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 400 | 엔드포인트가 비활성 상태. 먼저 시작해야 함 |
| 400 | 오토스케일링이 활성화된 상태. 먼저 비활성화해야 함 |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |
| 500 | 스케일링 실패 |

##### 요청 예시

```bash
curl -X POST "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}/scale" \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{"num_replicas": 3}'
```

##### 응답 예시

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "previous_replicas": 2,
  "current_replicas": 3,
  "message": "Successfully scaled from 2 to 3 replicas"
}
```

이미 동일한 레플리카 수인 경우:

```json
{
  "endpoint_id": "51c910f7-ecde-4c6c-88f8-d8e37ca9d598",
  "previous_replicas": 3,
  "current_replicas": 3,
  "message": "No change - endpoint already has the requested number of replicas"
}
```

---

#### 엔드포인트 수정

엔드포인트의 일부 운영 설정을 수정합니다. **엔드포인트가 비활성 상태일 때만 수정 가능합니다.** 최소 하나 이상의 필드를 제공해야 합니다.

```
PATCH /endpoints/{endpoint_id}
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### Request Body

모든 필드가 선택사항이지만, 최소 하나는 제공해야 합니다.

| 필드 | 타입 | 설명 | 예시 |
| --- | --- | --- | --- |
| container_start_command | string | 컨테이너 시작 명령어 | `"python app.py"` |
| container_env_vars | array | 컨테이너 환경 변수 목록. 각 항목은 `{"key": "키", "value": "값"}` 형식 | `[{"key":"ENV","value":"prod"}]` |
| container_port | integer | 컨테이너 서비스 포트. 1~65535 범위 | `8080` |
| container_health_check_path | string | 헬스체크 경로 | `"/healthz"` |
| container_health_check_timeout | integer | 컨테이너 헬스체크 타임아웃(초). 1 이상 | `30` |
| health_check_timeout | integer | 전체 헬스체크 타임아웃(초). 1 이상. 이 시간 내에 헬스체크가 통과하지 않으면 배포 실패 | `300` |
| container_metric_path | string | Prometheus 메트릭 수집 경로 | `"/metrics"` |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| endpoint_id | string | 엔드포인트 고유 ID |
| name | string | 엔드포인트 이름 |
| is_active | boolean | 활성 여부 (수정 성공 시 항상 `false`) |
| container_start_command | string \| null | 현재 설정된 시작 명령어 |
| container_env_vars | array \| null | 현재 설정된 환경변수 |
| container_port | integer \| null | 현재 설정된 포트 |
| container_health_check_path | string \| null | 현재 설정된 헬스체크 경로 |
| container_health_check_timeout | integer \| null | 현재 설정된 헬스체크 타임아웃 |
| health_check_timeout | integer \| null | 현재 설정된 전체 헬스체크 타임아웃 |
| container_metric_path | string \| null | 현재 설정된 메트릭 경로 |
| updated_at | datetime | 수정 시간 (ISO 8601) |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 400 | 수정할 필드가 하나도 제공되지 않음 |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |
| 409 | 엔드포인트가 활성 상태. 먼저 중지해야 함 |
| 500 | 수정 실패 |

##### 요청 예시

```bash
curl -X PATCH "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}" \
  -H "Authorization: Bearer {api_key}" \
  -H "Content-Type: application/json" \
  -d '{
    "container_port": 8088
  }'
```

##### 응답 예시

```json
{
  "endpoint_id": "cad59665-b77b-4cb6-9308-d276294bce79",
  "name": "cpu-bench",
  "is_active": false,
  "container_start_command": "server -http-port 8080",
  "container_env_vars": [{"key": "", "value": ""}],
  "container_port": 8088,
  "container_health_check_path": "/health",
  "container_health_check_timeout": 600,
  "health_check_timeout": 300,
  "container_metric_path": null,
  "updated_at": "2026-04-08T03:18:17.974691Z"
}
```

---

#### 레플리카 목록 조회

특정 엔드포인트에서 현재 **실행 중인 레플리카** 목록을 조회합니다.

```
GET /endpoints/{endpoint_id}/replicas
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### 응답 필드 (배열)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| replica_id | string | 레플리카 고유 ID |
| state | string | 레플리카 상태. `"RUNNING"`, `"STARTING"`, `"STOPPING"` 등 |
| node_id | string \| null | 레플리카가 실행 중인 노드 ID |
| node_ip | string \| null | 노드 IP 주소 |
| actor_id | string \| null | Ray actor ID |
| actor_name | string \| null | Ray actor 이름 |
| worker_id | string \| null | Ray worker ID |
| pid | integer \| null | 프로세스 ID |
| start_time_s | number \| null | 레플리카 시작 시간 (Unix timestamp, 초 단위) |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |

##### 요청 예시

```bash
curl -X GET "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}/replicas" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
[
  {
    "replica_id": "2xoeq1pe",
    "state": "RUNNING",
    "node_id": "f5efc481ed6aa2bdbe054beaa7b96c00c8827259931faaa8cdd77be9",
    "node_ip": "100.64.0.11",
    "actor_id": "2da9a83b9d0a551f2ea04df150000000",
    "actor_name": "SERVE_REPLICA::org-0I2GoSBhwODG.proj-sGF4ynUPbc8H.584ef3d4#ContainerRelayDeployment#2xoeq1pe",
    "worker_id": "b8dabf7fd2e70b036339041a5896de447517222edc8f8990f86cfe75",
    "pid": 398977,
    "start_time_s": 1775452417.236
  }
]
```

---

#### 로그 파일 목록 조회

특정 엔드포인트의 로그 파일 목록을 조회합니다. 레플리카별, 이력 로그 포함 조회가 가능합니다.

```
GET /endpoints/{endpoint_id}/logs
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### Query Parameters

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
| --- | --- | --- | --- | --- |
| replica_id | string | No | - | 특정 레플리카의 로그만 필터링 |
| include_history | boolean | No | true | `false`로 설정하면 현재 활성 로그만 반환하고 이전 로테이션된 로그는 제외 |

##### 응답 필드 (배열)

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| node_id | string | 로그가 저장된 노드 ID |
| replica_id | string | 연결된 레플리카 ID |
| filename | string | 로그 파일 경로. `/logs/file` API의 `filename` 파라미터에 사용 |
| is_historical | boolean | 이전 로테이션된(과거) 로그 파일 여부. `true`이면 종료된 레플리카의 로그 |
| start_time | datetime \| null | 로그 시작 시간. 이력 로그에서만 제공 |
| end_time | datetime \| null | 로그 종료 시간. 이력 로그에서만 제공 |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |

##### 요청 예시

```bash
curl -X GET "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}/logs" \
  -H "Authorization: Bearer {api_key}"
```

##### 응답 예시

```json
[
  {
    "node_id": "f5efc481ed6aa2bdbe054beaa7b96c00c8827259931faaa8cdd77be9",
    "replica_id": "m8gce67b",
    "filename": "org-0I2GoSBhwODG/proj-sGF4ynUPbc8H/584ef3d4.m8gce67b/container/ssh-test-m8gce67b-20260326.log",
    "is_historical": true,
    "start_time": "2026-03-26T06:19:06.911000Z",
    "end_time": "2026-03-26T06:44:41.437000Z"
  },
  {
    "node_id": "f5efc481ed6aa2bdbe054beaa7b96c00c8827259931faaa8cdd77be9",
    "replica_id": "2xoeq1pe",
    "filename": "org-0I2GoSBhwODG/proj-sGF4ynUPbc8H/584ef3d4.2xoeq1pe/container/ssh-2xoeq1pe-20260408.log",
    "is_historical": false,
    "start_time": null,
    "end_time": null
  }
]
```

---

#### 로그 파일 내용 조회

특정 로그 파일의 실제 내용을 라인 범위로 조회합니다.

```
GET /endpoints/{endpoint_id}/logs/file
```

##### Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| endpoint_id | string | Yes | 엔드포인트 고유 ID |

##### Query Parameters

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
| --- | --- | --- | --- | --- |
| node_id | string | Yes | - | 로그 파일이 위치한 노드 ID. `/logs` 응답의 `node_id` 값 사용 |
| filename | string | Yes | - | 로그 파일 경로. `/logs` 응답의 `filename` 값 사용 |
| start_line | integer | No | 1 | 읽기 시작할 라인 번호. 1부터 시작 |
| end_line | integer | No | 1000 | 읽기 종료할 라인 번호. 최소 1, 최대 50000 |

##### 응답 필드

| 필드 | 타입 | 설명 |
| --- | --- | --- |
| content | string | 로그 파일 내용 (지정된 라인 범위). 각 줄은 JSON 형식의 로그 엔트리 |
| start_line | integer | 실제 반환된 시작 라인 번호 |
| end_line | integer | 실제 반환된 종료 라인 번호 |

##### 에러

| 코드 | 설명 |
| --- | --- |
| 401 | API Key 누락 또는 유효하지 않음 |
| 403 | 엔드포인트가 해당 조직 또는 프로젝트에 속하지 않음 |
| 404 | 엔드포인트를 찾을 수 없음 |

##### 요청 예시

```bash
curl -G "https://external.aieev.cloud:5007/external/api/v1/endpoints/{endpoint_id}/logs/file" \
  -H "Authorization: Bearer {api_key}" \
  --data-urlencode "node_id={node_id}" \
  --data-urlencode "filename={filename}" \
  --data-urlencode "start_line=1" \
  --data-urlencode "end_line=200"
```

##### 응답 예시

```json
{
  "content": "{\"source\":\"stderr\",\"log\":\"[I 2026-04-08 01:01:37.213 ServerApp] Client disconnected.\",\"timestamp\":\"2026-04-08T10:01:37.000Z\"}\n{\"source\":\"stderr\",\"log\":\"[I 2026-04-08 01:34:00.303 ServerApp] Client connected.\",\"timestamp\":\"2026-04-08T10:34:00.000Z\"}\n",
  "start_line": 1,
  "end_line": 200
}
```

---

### 참고 사항

- API Key는 생성 시 지정한 **프로젝트와 엔드포인트에만 접근 가능**.
- 레플리카 스케일링은 엔드포인트가 **활성 상태**일 때만 가능.
- 오토스케일링이 활성화된 엔드포인트는 수동 스케일링이 불가능하므로, External API를 통한 스케일링이 필요할 경우 웹 콘솔에서 **오토스케일링 비활성화** 적용 필요.
- 엔드포인트 설정 수정(PATCH)은 **비활성 상태**에서만 가능. 활성 상태에서 수정 시도 시 `409 Conflict` 반환.
