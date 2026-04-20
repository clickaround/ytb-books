---
name: books-math-formula
description: LaTeX 수식 → PNG 렌더링 + 유니코드 변환 (MathJax v4)
user-invocable: false
paths: short-video-maker/src/YTB-books-project/src/services/MathFormulaService.ts
---

## 용도
LaTeX 수식 문자열을 PNG 이미지로 렌더링하거나, 유니코드 텍스트로 변환. 수학 교육 영상의 수식 오버레이에 사용.

## 핵심 파일
- `src/YTB-books-project/src/services/MathFormulaService.ts`

## 의존성
- `mathjax-full` (npm) — TeX → SVG 변환
- `sharp` (npm) — SVG → PNG 래스터화

## API / 인터페이스
```typescript
class MathFormulaService {
  // LaTeX → PNG 버퍼 (영상 오버레이용)
  renderLatexToPng(latex: string, options?: { fontSize?: number; color?: string }): Promise<Buffer>

  // LaTeX → 유니코드 문자열 (자막/TTS용)
  convertLatexToDisplayText(latex: string): string
}
```

## MathJax 설정
```typescript
// 반드시 TeX({}) 단독 사용
import { TeX } from 'mathjax-full/js/input/tex.js';
// AllPackages 절대 사용 금지 — Node.js 크래시 발생
```

## 렌더링 파이프라인
1. TeX → MathJax SVG 출력
2. `<mjx-container>` 래퍼 제거
3. `fill="currentColor"` → `fill="white"` 치환 (어두운 배경용)
4. sharp로 SVG → PNG 변환

## 다른 프로젝트에 이식하기
1. `MathFormulaService.ts` 복사
2. `npm install mathjax-full sharp`
3. 색상/크기 옵션을 프로젝트에 맞게 조정

## 주의사항
- **AllPackages 사용 금지**: `import { AllPackages }` 하면 Node.js 프로세스 크래시
- `TeX({})` 빈 설정으로만 초기화해야 안정
- `fill="currentColor"` 미치환 시 흰 배경에서만 보임 — 반드시 `fill="white"` 변환
- `<mjx-container>` 래퍼가 SVG 감싸므로 제거 후 sharp에 전달
- sharp는 native addon이므로 Docker 빌드 시 `npm rebuild sharp` 필요
