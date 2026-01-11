# New Blog Post Skill

이 SKILL은 새로운 블로그 포스트를 자동으로 생성하는 역할을 합니다.

## 역할

- Quartz 블로그의 새로운 마크다운 포스트 생성
- 템플릿 기반의 일관된 포스트 구조 유지
- 블로그 작성 규칙 및 가이드라인 준수

## 참조 파일

### 1. 템플릿 파일
- **경로**: `content/template/new_post.md`
- **용도**: 새 포스트 생성 시 frontmatter 및 기본 구조 제공
- **내용**:
  - title, description, tags 등 메타데이터 구조
  - permalink, draft, lang 등 Quartz 설정
  - enableToc, cssclasses, socialImage 등 추가 옵션
  - created, updated 날짜 필드

### 2. 블로그 작성 규칙 파일 (권장)
- **권장 경로**: `content/private/00.template/blog-writing-rules.md`
- **용도**: AI가 양질의 블로그 컨텐츠를 생성하기 위한 작성 규칙
- **포함 권장 내용**:
  - 글쓰기 스타일 가이드 (톤앤매너, 문체)
  - 기술 블로그 작성 시 주의사항
  - 코드 예제 작성 규칙
  - 이미지 및 미디어 사용 가이드
  - SEO 최적화 팁
  - 태그 분류 기준

## 새 포스트 생성 프로세스

### 1. 사용자 요구사항 파악
```
- 포스트 주제
- 대상 독자
- 주요 키워드/태그
- 카테고리 (예: dev, private 등)
```

### 2. 템플릿 적용
```
1. content/template/new_post.md 읽기
2. 사용자 요구사항에 맞게 frontmatter 커스터마이징
3. 적절한 title, description 생성
4. 관련 tags 선정
5. created, updated 날짜를 현재 날짜로 설정
```

### 3. 블로그 작성 규칙 준수
```
1. blog-writing-rules.md 파일이 있다면 읽기
2. 규칙에 따라 컨텐츠 구조화
3. 코드 예제, 이미지 등 포맷 규칙 적용
4. SEO 최적화 고려
```

### 4. 파일 생성 위치
```
- 공개 포스트: content/dev/ 또는 content/posts/
- 비공개 포스트: content/private/
- 파일명 형식: YYYY-MM-DD-title.md 또는 title.md
```

### 5. 초안 작성
```
1. 제목과 부제목 구조 생성
2. 서론, 본론, 결론 구조화
3. 코드 예제 포함 (해당하는 경우)
4. 참고 자료 및 링크 추가
```

## 사용 예시

```
사용자: "React Hook의 useCallback과 useMemo 차이에 대한 기술 블로그 포스트를 작성해줘"

AI 작업 순서:
1. content/template/new_post.md 읽기
2. content/private/00.template/blog-writing-rules.md 읽기 (있는 경우)
3. 적절한 frontmatter 생성:
   - title: "React Hook: useCallback vs useMemo 완벽 가이드"
   - tags: ["React", "Hooks", "Performance"]
   - description: "useCallback과 useMemo의 차이점과 사용 시나리오"
4. content/dev/react-hooks-comparison.md 파일 생성
5. 구조화된 컨텐츠 작성
```

## 주의사항

1. **draft 설정**: 초안인 경우 `draft: true`로 설정
2. **날짜 형식**: ISO 8601 형식 사용 (YYYY-MM-DD)
3. **태그 일관성**: 기존 포스트의 태그와 일관성 유지
4. **한글/영어**: lang 필드 적절히 설정 (ko/en)
5. **permalink**: URL 친화적인 형식으로 생성
6. **이미지 경로**: public/ 디렉토리 기준 절대 경로 사용

## 개선 제안

블로그 작성 규칙 파일을 만들어 다음 내용을 포함하세요:
- 코드 블록에 언어 지정 필수
- 실용적인 예제 우선
- 명확하고 간결한 설명
- 적절한 heading 레벨 사용
- 내부 링크 활용
- 마크다운 best practices
