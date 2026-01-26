# 데이터 구조 문서

부산퀴어행동 웹사이트의 데이터 파일 구조 및 스키마 정의

## 📁 파일 구조

```
public/data/
├── README.md           # 이 파일
├── manifesto.json      # 강령 및 기본원칙
├── project_2025.json   # 2025년 사업결과
├── project_2026.json   # 2026년 사업계획
├── member.txt          # 회원 가입 안내
└── supporter.txt       # 후원자 안내
```

## 📋 JSON 스키마

### 1. manifesto.json - 강령 및 기본원칙

```json
{
  "type": "Manifesto",
  "version": "1.0",
  "lastUpdated": "YYYY-MM-DD",
  "전문": {
    "id": "manifesto-full",
    "summary": "요약문",
    "full_text": "전문 전체 내용"
  },
  "기본원칙": [
    {
      "id": "principle-N",
      "order": 1,
      "summary": "원칙 요약",
      "full_text": "원칙 전체 내용"
    }
  ]
}
```

**필드 설명:**
- `type`: 문서 타입 (고정값: "Manifesto")
- `version`: 문서 버전
- `lastUpdated`: 마지막 수정일 (ISO 8601 형식)
- `전문.id`: 고유 식별자
- `전문.summary`: 전문 요약문 (목록에 표시)
- `전문.full_text`: 전문 전체 내용 (모달에 표시)
- `기본원칙[].id`: 원칙 고유 식별자
- `기본원칙[].order`: 표시 순서
- `기본원칙[].summary`: 원칙 요약
- `기본원칙[].full_text`: 원칙 전체 내용

---

### 2. project_YYYY.json - 사업 결과 (DoneProject)

```json
{
  "title": "YYYY년 사업결과",
  "type": "DoneProject",
  "year": 2025,
  "version": "1.0",
  "lastUpdated": "YYYY-MM-DD",
  "items": [
    {
      "id": "project-YYYY-NN",
      "order": 1,
      "subtitle": "프로젝트 제목",
      "description": "프로젝트 설명",
      "details": [
        "세부 내용 1",
        "세부 내용 2"
      ],
      "photos": [
        "이미지 URL 1",
        "이미지 URL 2"
      ]
    }
  ]
}
```

**필드 설명:**
- `type`: 문서 타입 (고정값: "DoneProject")
- `year`: 사업 연도
- `items[].id`: 프로젝트 고유 식별자 (형식: `project-YYYY-NN`)
- `items[].order`: 표시 순서
- `items[].subtitle`: 프로젝트 제목
- `items[].description`: 프로젝트 간단 설명
- `items[].details`: 세부 내용 배열 (선택사항)
- `items[].photos`: 사진 URL 배열 (여러 사진 지원, 슬라이더로 표시)

---

### 3. project_YYYY.json - 사업 계획 (PlanProject)

```json
{
  "title": "YYYY년 사업계획",
  "type": "PlanProject",
  "year": 2026,
  "version": "1.0",
  "lastUpdated": "YYYY-MM-DD",
  "goal": [
    {
      "id": "goal-YYYY-NN",
      "order": 1,
      "is_modal": true,
      "subtitle": "기조 제목",
      "description": "기조 설명",
      "details": [
        "상세 내용 1",
        "상세 내용 2"
      ]
    }
  ],
  "items": [
    {
      "id": "project-YYYY-NN",
      "order": 1,
      "subtitle": "사업 제목",
      "description": "사업 설명",
      "details": ["세부사항"],
      "photos": []
    }
  ]
}
```

**필드 설명:**
- `type`: 문서 타입 (고정값: "PlanProject")
- `year`: 계획 연도
- `goal[].id`: 기조 고유 식별자
- `goal[].order`: 표시 순서
- `goal[].is_modal`: 모달로 표시 여부
  - `true`: 클릭 시 모달 창으로 상세 내용 표시
  - `false`: 카드 내에 직접 표시
- `goal[].subtitle`: 기조 제목
- `goal[].description`: 기조 간단 설명
- `goal[].details`: 상세 내용 배열
- `items[]`: DoneProject의 items와 동일한 구조

---

## 🎨 데이터 업데이트 가이드

### 새 프로젝트 추가

1. 적절한 연도의 JSON 파일 선택
2. `items` 배열에 새 객체 추가
3. `id`는 `project-YYYY-NN` 형식으로 작성 (NN은 01부터 시작)
4. `order` 필드로 표시 순서 지정
5. `lastUpdated` 필드를 현재 날짜로 업데이트

### 기본원칙 수정

1. `manifesto.json` 파일 수정
2. `기본원칙` 배열의 해당 항목 수정
3. `lastUpdated` 필드 업데이트

### 사진 추가

1. 사진을 `/public/images/` 폴더에 업로드 (또는 외부 URL 사용)
2. 해당 프로젝트의 `photos` 배열에 URL 추가
3. 사진은 자동으로 슬라이더 형태로 표시됨

---

## ⚠️ 주의사항

1. **JSON 형식**: 모든 JSON 파일은 유효한 형식이어야 합니다
2. **고유 ID**: 각 항목의 `id`는 고유해야 합니다
3. **order 필드**: 항목의 표시 순서를 명확히 지정해야 합니다
4. **lastUpdated**: 수정 시 반드시 업데이트해야 합니다
5. **빈 배열**: `details`나 `photos`가 없을 경우 빈 배열(`[]`)로 유지

---

## 📝 버전 관리

- **v1.0** (2026-01-13): 초기 스키마 정의 및 구조화
  - `id`, `order`, `version`, `lastUpdated` 필드 추가
  - 명확한 타입 구분 (DoneProject, PlanProject, Manifesto)
  - 일관된 데이터 구조 적용

---

## 🔍 예시

### 프로젝트 추가 예시

```json
{
  "id": "project-2026-04",
  "order": 4,
  "subtitle": "정기 세미나 개최",
  "description": "매월 성소수자 권리에 관한 세미나를 개최합니다.",
  "details": [
    "매월 둘째 주 토요일 오후 3시",
    "온라인/오프라인 병행 진행",
    "전문가 초청 강연"
  ],
  "photos": []
}
```

---

마지막 업데이트: 2026-01-13
