# ThreeUI 사용 기록

## 확인한 배포본

- 저장소: `https://github.com/MengTo/threeui`
- 검토 리비전: `0579b809e4425f8f458c2b496a8d3e1a82dac3f7`
- npm 패키지: `@designcodeio/threeui@1.1.0`
- 패키지 라이선스: MIT

구현은 npm 패키지가 공개한 component subpath만 사용한다.

```ts
import("@designcodeio/threeui/components/OrbitalSphereBackground");
```

`SylvaHero`, `SylvaLivingWorldScene`, Pro/Beta 소스, 원격 썸네일과 미리보기 자산은 사용하지 않았다.

## 사용 파일과 변경 범위

| 구분 | 파일 | 사용 방식 | 변경 여부 |
| --- | --- | --- | --- |
| npm export | `components/OrbitalSphereBackground` | 지연 import | 변경 없음 |
| 공개 소스 | `src/shaders/orbital-sphere/OrbitalSphereBackground.tsx` | props와 lifecycle 확인 | 변경 없음 |
| 공개 소스 | `src/shaders/orbital-sphere/orbitalSphereRenderer.ts` | WebGL renderer와 dispose 경로 확인 | 변경 없음 |
| 로컬 adapter | `src/three/ThreeUiOrbitalScene.tsx` | 선택 단계에 맞는 공개 props 전달 | 새 코드 |
| 로컬 UI | `src/components/evidence-loop/` | DOM 버튼, 정적 SVG, lifecycle gate | 새 코드 |

ThreeUI 원본 코드와 asset은 복사하거나 수정하지 않았다. `OrbitalSphereBackground`에는 외부 이미지나 글꼴이 필요하지 않다. 로컬 adapter는 `speed`, `particleSize`, `particleOpacity`, `orbitOpacity`, `scale`, `haloOpacity`, `hue`, `className`만 전달한다.

공개 컴포넌트는 `ResizeObserver`로 canvas 크기를 맞추고 `IntersectionObserver`로 화면 밖의 animation frame을 멈춘다. unmount에서는 animation frame을 취소하고 observer를 끊은 뒤 geometry, material, renderer를 dispose한다. `document.hidden`에서 멈춘 frame은 visibility 이벤트만으로 다시 시작하지 않으므로, 로컬 host가 `visibilitychange`에서 컴포넌트를 unmount하고 탭이 보일 때 다시 mount한다.

## 정적 fallback

다음 조건에서는 ThreeUI 모듈을 실행하지 않고 SVG 시스템 지도를 보여준다.

- 컨테이너 폭이 719px 이하일 때
- `prefers-reduced-motion: reduce`가 설정됐을 때
- 문서가 숨겨졌거나 컴포넌트가 화면 밖에 있을 때
- WebGL을 만들 수 없을 때
- 지연 import, renderer 초기화, WebGL context에서 오류가 났을 때

네 단계의 버튼과 증거 링크는 WebGL과 별도 DOM에 있어 fallback에서도 그대로 읽고 조작할 수 있다.

## 통합 세션 인터페이스

통합 세션은 앱 엔트리의 다른 전역 스타일보다 먼저 ThreeUI 스타일을 불러와야 한다.

```ts
import "@designcodeio/threeui/style.css";
```

Evidence Loop는 다음처럼 연결한다. `items`에는 real, sim, policy, replay 네 항목이 모두 필요하다. 선택 상태를 상위 화면에서 관리하려면 `selectedStep`과 `onSelectedStepChange`를 함께 전달한다.

```tsx
import { EvidenceLoop, type EvidenceLoopItems } from "./components/evidence-loop";

const items: EvidenceLoopItems = {
  real: { label, description, href, linkLabel },
  sim: { label, description, href, linkLabel },
  policy: { label, description, href, linkLabel },
  replay: { label, description, href, linkLabel },
};

<EvidenceLoop
  items={items}
  selectedStep={selectedStep}
  onSelectedStepChange={(step) => setSelectedStep(step)}
/>;
```

`EvidenceLoop`가 로컬 CSS를 직접 불러오므로 통합 세션에서 별도 CSS import는 필요하지 않다. 앱 셸은 이 브랜치의 소유 범위가 아니어서 수정하지 않았다.

## 보존한 고지

ThreeUI 저장소의 아래 네 파일을 원문 그대로 `public/threeui/licenses/`에 복사했다.

- `LICENSE`
- `ASSET-LICENSES.md`
- `FONT-LICENSES.md`
- `THIRD_PARTY_NOTICES.md`

서비스 경로는 `/threeui/licenses/파일명`이다.
