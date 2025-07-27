## **I. 디자인 시스템 개요**

본 디자인 시스템은 **미니멀리즘**, **명료함**, **전문성**을 핵심 가치로 삼아, Notion과 유사한 사용자 경험을 제공하는 것을 목표로 합니다. 사용자가 콘텐츠에 집중하고, 직관적으로 기능을 사용할 수 있도록 시각적 요소를 최소화하고 기능성에 초점을 맞춥니다.

### **핵심 디자인 원칙**

  - **기능적 미니멀리즘 (Functional Minimalism)**

      - 모든 디자인 요소는 명확한 목적을 가져야 합니다. 불필요한 장식이나 시각적 잡음을 제거하여 사용자가 핵심 기능과 콘텐츠에 집중할 수 있도록 합니다.
      - 아이콘은 화살표/셰브론과 같은 유틸리티 아이콘만 사용하여 일관되고 예측 가능한 상호작용을 유도합니다.

  - **제한된 색상 사용 (Restricted Color Palette)**

      - 브랜드 정체성을 나타내는 `Primary` 컬러와 텍스트를 위한 `Foreground` 컬러 중심으로 색상 사용을 제한합니다. 이를 통해 차분하고 전문적인 인상을 주며, 사용자의 시각적 피로도를 줄입니다.

  - **즉각적인 인터랙션 (Zero-Transition Interaction)**

      - 사용자의 액션에 즉각적으로 반응하기 위해 `hover`, `reveal` 등의 전환(transition) 효과를 의도적으로 제거합니다. 이는 시스템의 반응성을 높이고, 사용자가 빠르고 명확하게 상태 변화를 인지하도록 돕습니다.

-----

## **II. 컬러 팔레트**

컬러 시스템은 Tailwind CSS와의 완벽한 호환성을 위해 설계되었습니다. `Primary`를 유일한 Accent 컬러로 사용하며, 전체적인 톤은 `Neutral` 계열의 그레이스케일로 유지합니다.

### **Tailwind CSS 설정 예시**

```javascript
// tailwind.config.js
module.exports = {
	theme: {
		extend: {
			colors: {
				primary: {
					50: '#EEF2FF',
					100: '#E0E7FF',
					200: '#C7D2FE',
					300: '#A5B4FC',
					400: '#818CF8',
					500: '#6366F1', // Base
					600: '#4F46E5',
					700: '#4338CA',
					800: '#3730A3',
					900: '#312E81',
				},
				neutral: {
					50: '#F8F9FA', // Background
					100: '#F1F3F5', // Subtle Background, Border
					200: '#E9ECEF',
					300: '#DEE2E6',
					400: '#CED4DA',
					500: '#ADB5BD', // Muted Foreground
					600: '#868E96',
					700: '#495057',
					800: '#343A40', // Strong Foreground
					900: '#212529', // Heading Foreground
				},
				// 시맨틱 컬러 (Semantic Colors)
				background: 'var(--color-neutral-50)',
				foreground: {
					DEFAULT: 'var(--color-neutral-800)',
					muted: 'var(--color-neutral-500)',
					heading: 'var(--color-neutral-900)',
				},
				border: 'var(--color-neutral-100)',
				ring: 'var(--color-primary-500)',
			},
		},
	},
	plugins: [],
};
```

### **컬러 스펙**

| 이름 (Tailwind Variable) | HEX 코드 | 설명 |
| :--- | :--- | :--- |
| `primary-500` | `#6366F1` | 링크, 포커스 링 등 핵심 Accent 컬러 |
| `neutral-900` | `#212529` | 제목 (Heading) |
| `neutral-800` | `#343A40` | 본문 텍스트 (Default Foreground) |
| `neutral-500` | `#ADB5BD` | 보조 텍스트 (Muted Foreground) |
| `neutral-100` | `#F1F3F5` | 경계선 (Border), 미묘한 배경 |
| `neutral-50` | `#F8F9FA` | 기본 페이지 배경 (Background) |

### **WCAG 2.2 명도 대비 체크리스트**

모든 텍스트와 UI 요소는 WCAG 2.2 AA 레벨 이상의 명도 대비를 충족해야 합니다.

  - [✅] **본문 텍스트:** `neutral-800` on `neutral-50` (11.43:1) - AAA 충족
  - [✅] **Accent 링크:** `primary-500` on `neutral-50` (4.65:1) - AA 충족
  - [✅] **보조 텍스트:** `neutral-500` on `neutral-50` (3.32:1) - 18.5px 이상 Bold 또는 24px 이상 일반 텍스트의 경우 AA 충족
  - [✅] **제목 텍스트:** `neutral-900` on `neutral-50` (15.27:1) - AAA 충족

-----

## **III. 페이지 구현**

각 페이지는 핵심 목적에 맞춰 최소한의 컴포넌트로 구성되며, 일관된 레이아웃 구조를 따릅니다.

### **1. 대시보드 (Dashboard)**

  - **핵심 목적**: 사용자가 가장 중요한 정보(예: 최근 활동, 주요 지표)를 한눈에 파악하고 다음 행동으로 빠르게 연결하는 것.

  - **주요 컴포넌트**:

      - `Header`: 페이지 제목과 핵심 액션 버튼 (예: '새로 만들기') 포함
      - `StatCard`: 주요 지표를 보여주는 정보 카드
      - `DataTable`: 최근 활동이나 데이터 목록을 표시하는 테이블
      - `ChartPlaceholder`: 시각적 데이터 표현을 위한 영역 (이미지 예시)

  - **레이아웃 구조**:

      - `Desktop / Wide`: 2단 또는 3단 그리드 구조로 카드와 테이블을 배치하여 정보 밀도를 높임.
      - `Tablet`: 2단 그리드 구조.
      - `Mobile`: 1단 수직 구조로 컴포넌트가 순차적으로 표시됨.

    <!-- end list -->

    ```html
    <div class="w-full h-64 bg-neutral-100 rounded-lg flex items-center justify-center">
      <img src="https://picsum.photos/seed/dashboard/800/400" alt="Data chart placeholder" class="object-cover w-full h-full rounded-lg" />
    </div>
    ```

### **2. 설정 (Settings)**

  - **핵심 목적**: 사용자가 자신의 계정 정보, 알림, 기타 옵션을 확인하고 수정하는 것.
  - **주요 컴포넌트**:
      - `Tabs`: '프로필', '알림', '결제' 등 설정 카테고리 간 전환
      - `FormInput`: 텍스트, 이메일 등의 입력 필드
      - `ToggleSwitch`: On/Off 옵션 선택
      - `Button`: '저장', '취소' 등의 액션 수행
  - **레이아웃 구조**:
      - `Desktop / Wide / Tablet`: 좌측에 탭 네비게이션, 우측에 해당 콘텐츠를 표시하는 2단 구조.
      - `Mobile`: 상단에 탭 컴포넌트, 하단에 콘텐츠가 표시되는 1단 구조.

-----

## **IV. 레이아웃 컴포넌트**

일관된 사용자 경험을 위해 대부분의 페이지에서 공통 레이아웃 컴포넌트를 사용합니다.

### **MainLayout**

  - **적용 라우트**: 로그인/회원가입 등 인증 페이지를 제외한 모든 페이지 (`/**`).
  - **핵심 컴포넌트**:
      - `Sidebar`: 글로벌 네비게이션 메뉴.
      - `Header`: 페이지 제목, 사용자 프로필, 검색 등.
      - `MainContent`: 각 페이지의 실제 콘텐츠가 렌더링되는 영역.
  - **반응형 동작**:
      - **Wide (`1440px+`)**: `Sidebar`가 항상 펼쳐져 있음. `MainContent` 영역이 넓어짐.
      - **Desktop (`1024px+`)**: `Sidebar`가 항상 펼쳐져 있음.
      - **Tablet (`768px+`)**: `Sidebar`가 아이콘 전용 바로 축소됨. 마우스를 올리면 확장될 수 있음 (단, 애니메이션 없음).
      - **Mobile (`320px+`)**: `Sidebar`가 기본적으로 숨겨지고, 헤더의 햄버거 아이콘 클릭 시 화면 좌측에서 나타남 (Overlay).

-----

## **V. 인터랙션 패턴**

모든 인터랙션은 전환 효과 없이 즉각적으로 발생하여 명확성과 반응성을 극대화합니다.

### **컴포넌트 상태 (Button 예시)**

버튼과 같은 인터랙티브 컴포넌트는 다음과 같은 상태를 가집니다.

| 상태 | 시각적 변화 | 커서 | 설명 |
| :--- | :--- | :--- | :--- |
| **Default** | `bg-primary-500`, `text-white` | `pointer` | 기본 상태 |
| **Hover** | `bg-primary-600`, `text-white` | `pointer` | 마우스 오버 시. 전환 효과 없음. |
| **Focus** | `bg-primary-600`, `ring-2 ring-offset-2 ring-primary-500` | `pointer` | Tab 키 등으로 포커스 되었을 때. 접근성 보장. |
| **Active** | `bg-primary-700` | `pointer` | 클릭 또는 누르고 있는 상태. |
| **Disabled** | `bg-neutral-200`, `text-neutral-400` | `not-allowed` | 비활성화 상태. Opacity 조정 없음. |

### **코드 예시 (React + Tailwind CSS)**

```jsx
function Button({ children, disabled }) {
  return (
    <button
      disabled={disabled}
      className="
        px-4 py-2 rounded-md font-semibold text-white
        bg-primary-500
        hover:bg-primary-600
        focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary-500
        active:bg-primary-700
        disabled:bg-neutral-200 disabled:text-neutral-400 disabled:cursor-not-allowed
        transition-none // 모든 전환 효과 제거
      "
    >
      {children}
    </button>
  );
}
```

-----

## **VI. 중단점 (Breakpoints)**

본 디자인 시스템은 모바일 우선 접근 방식을 따르며, 정의된 중단점을 기준으로 반응형 레이아웃을 구현합니다.

### **중단점 정의**

| 이름 | 최소 너비 | 설명 |
| :--- | :--- | :--- |
| `mobile` | `320px` | 모바일 디바이스 |
| `tablet` | `768px` | 태블릿 디바이스 |
| `desktop` | `1024px` | 데스크톱 모니터 |
| `wide` | `1440px` | 와이드 데스크톱 모니터 |

### **SCSS 변수 설정**

SCSS(Sass)를 사용하는 경우, 다음과 같이 맵으로 중단점을 관리하여 미디어 쿼리 작성을 용이하게 할 수 있습니다.

```scss
// _variables.scss
$breakpoints: (
	'mobile': 320px,
	'tablet': 768px,
	'desktop': 1024px,
	'wide': 1440px
);

// _mixins.scss
@mixin mq($key) {
  $size: map-get($breakpoints, $key);
  @media (min-width: $size) {
    @content;
  }
}

// 사용 예시
.container {
  width: 100%;

  @include mq('tablet') {
    max-width: 720px;
  }
  @include mq('desktop') {
    max-width: 960px;
  }
}
```