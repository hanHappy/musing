# Musing 프로젝트 가이드

## 개요
**Musing**은 LLM 기반 노트 관리 시스템. 자동 분석, 카테고리 추천, 유사 노트 병합 기능 제공.

## 기술 스택
- **Backend**: Python + SQLAlchemy + PostgreSQL + OpenAI API (GPT-4o-mini)
- **Frontend**: Next.js 16 + React 19 + TypeScript + Tailwind + shadcn/ui

## 아키텍처
```
Frontend (musing-web) ←→ REST API (TODO) ←→ Backend (musing-py) ←→ PostgreSQL + OpenAI
```

## 핵심 워크플로우
```
노트 입력 → DB 저장 → 유사 노트 검색(threshold: 0.3) → [병합 or 새 노트]
→ LLM 분석(제목/정제/카테고리/태그) → 카테고리 선택 → 완료
```

## 데이터 모델
### Category
- `full_path` (Unique): "개발/백엔드/API"
- `level`: 1~3 (3단계 계층)

### Note
- `original_text` / `refined_text`: 원본/정제된 텍스트
- `version`: 병합 시 증가
- `is_merged`: 병합 여부
- `category_id` (FK), `tags` (M2M)

### Tag
- `name` (Unique), `notes` (M2M)

## LLM 서비스 (musing-py/app/llm_service.py)
1. **find_similar_notes**: 의미론적 유사도 검색 (threshold: 0.3)
2. **analyze_note**: 제목 추출(20자), 텍스트 정제, 카테고리/태그 추천
3. **merge_notes**: 노트 통합 및 버전 증가

## 주요 파일 위치
### Backend (musing-py)
- `app/main.py`: CLI 진입점
- `app/models.py`: ORM 모델
- `app/note_service.py`: 비즈니스 로직
- `app/llm_service.py`: LLM 통합
- `chat_api.py`: OpenAI API 래퍼

### Frontend (musing-web)
- `app/page.tsx`: 메인 페이지
- `components/sidebar.tsx`: 카테고리 트리
- `components/note-viewer.tsx`: 노트 상세/편집
- `lib/mock-data.ts`: Mock 데이터 (현재 사용중)
- `types/index.ts`: TypeScript 인터페이스

## 현재 상태
✅ **완료**: DB 모델, LLM 통합, CLI, 프론트엔드 UI (Mock 데이터)
🚧 **TODO**: REST API 구현, 프론트엔드 API 연동, 검색, 노트 생성 UI

## 개발 가이드
- **Backend**: PEP 8, snake_case, 타입 힌트 필수
- **Frontend**: TypeScript strict, camelCase, shadcn/ui 패턴
- **성능**: LLM 호출 최소화, useMemo로 트리 빌드 캐싱
- **환경**: `.env`에 DATABASE_URL, OPENAI_API_KEY 설정

### 김지민의 테스트