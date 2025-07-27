## **YouTube 요약 확장 프로그램 정보 구조(IA) 설계 문서**

본 문서는 YouTube 영상 요약 크롬 확장 프로그램의 정보 구조(IA)를 정의합니다. 제품 관리자(PM)와 최고 기술 책임자(CTO)가 제품의 구조와 흐름을 명확히 이해하고, 이를 바탕으로 성공적인 개발을 진행하는 것을 목표로 합니다.

-----

### **1. 사이트맵 (Sitemap)**

확장 프로그램이 사용자에게 노출되는 모든 화면(Surface)과 기능의 구조를 나타냅니다.

  - **콘텐츠 스크립트 (YouTube 페이지에 주입)**
      - **YouTube 플레이어 UI**
          - 영상 요약 버튼 (`Summarize Video`)
      - **우측 사이드 패널 (Transcript Panel)**
          - 패널 헤더 (제목, 접기/펴기 버튼)
          - 트랜스크립트 뷰 (추출된 스크립트 텍스트)
          - 패널 푸터 (복사 버튼, 상태 메시지)
  - **확장 프로그램 옵션 페이지**
      - **상단 탭 바**
          - 일반 설정 (현재는 이 탭만 존재)
      - **설정 콘텐츠**
          - AI 서비스 선택 섹션 (드롭다운 메뉴: ChatGPT, Gemini, Claude 등)
  - **확장 프로그램 팝업 (툴바 아이콘 클릭 시 - *선택 사항*)**
      - 빠른 작업 버튼
          - `현재 영상 요약하기` (콘텐츠 스크립트의 '영상 요약 버튼'과 동일 기능)
          - `옵션 페이지 열기` (바로가기)

-----

### **2. 페이지 (뷰) 계층 구조 (Page/View Hierarchy)**

사용자 접점 간의 관계와 이동 경로를 시각적으로 표현합니다.

```text
(Root: Chrome Browser)
|
├── 1. YouTube 웹사이트 (*.youtube.com)
|   └── Injected UI Layer (Content Script)
|       ├── 1a. Player Control Button ('영상 요약')
|       └── 1b. Right-hand Transcript Panel (Collapsible)
|
└── 2. 확장 프로그램 UI (chrome-extension://[ID]/...)
    |
    ├── 2a. Options Page (options.html)
    |   │   (접근 경로: 확장 프로그램 아이콘 우클릭 > 옵션)
    |   └── AI Service Settings View
    |
    └── 2b. Popup (popup.html)
        │   (접근 경로: 툴바의 확장 프로그램 아이콘 좌클릭)
        └── Quick Actions View
```

-----

### **3. 콘텐츠 조직화 및 데이터 흐름 (Content Organization & Data Flow)**

#### **콘텐츠 조직화**

모든 데이터는 백엔드 없이 사용자의 브라우저 내에서 처리 및 저장됩니다.

| 데이터 항목 | 저장 위치 | 설명 |
| :--- | :--- | :--- |
| **영상 트랜스크립트** | 메모리 (On-the-fly) | 사용자가 '영상 요약' 버튼 클릭 시, 현재 페이지에서 실시간으로 추출. 저장되지 않음. |
| **선택된 AI 서비스** | `chrome.storage.local` | 사용자가 옵션 페이지에서 선택한 AI 챗봇 서비스 (예: 'chatgpt', 'gemini'). 확장 프로그램 전반에서 이 설정을 참조함. |
| **UI 상태** | 컴포넌트 State | 패널의 열림/닫힘 상태 등 UI의 일시적인 상태는 각 컴포넌트 내부에서 관리. |

#### **핵심 데이터 흐름 (요약 기능)**

1.  **시작 (Trigger)**: 사용자가 YouTube 플레이어의 **'영상 요약' 버튼**을 클릭합니다.
2.  **추출 (Extraction)**: 콘텐츠 스크립트가 현재 영상의 트랜스크립트를 탐색합니다.
      - 우선순위: 1) 공식 자막 -\> 2) 자동 생성 자막
      - 트랜스크립트가 없으면 사용자에게 '없음' 알림(Toast)을 표시하고 중단합니다.
3.  **설정 조회 (Read Storage)**: `chrome.storage.local`에서 사용자가 설정한 **AI 서비스** (예: 'gemini')와 해당 URL(예: `https://gemini.google.com/app`)을 가져옵니다.
4.  **새 탭 열기 (Open Tab)**: 조회한 AI 서비스 URL로 새 탭을 엽니다 (`window.open`).
5.  **데이터 전달 및 붙여넣기 (Paste)**:
      - 백그라운드 스크립트(Service Worker)가 새 탭이 열렸다는 것을 감지합니다.
      - 해당 탭에 **콘텐츠 스CRIPT를 주입**하여, 미리 준비된 트랜스크립트 텍스트를 챗봇의 입력창(textarea)에 **자동으로 붙여넣습니다**.
      - 보안을 위해 `activeTab` 권한과 스크립팅 API를 활용합니다.
6.  **완료**: 사용자는 트랜스크립트가 입력된 AI 챗봇 페이지에서 직접 '전송' 버튼을 눌러 요약을 요청합니다.

-----

### **4. 인터랙션 패턴 (Interaction Patterns)**

주요 UI 요소의 상태 변화와 사용자 피드백 방식을 정의합니다.

#### **버튼 상태 (Button States)**

| 컴포넌트 | 상태 | 시각적/기능적 표현 |
| :--- | :--- | :--- |
| **영상 요약 버튼** | **Default** | 기본 아이콘과 텍스트 |
| | **Hover** | 배경색 변경 또는 미세한 확대 효과 |
| | **Loading** | 아이콘이 스피너(spinner)로 변경. 트랜스크립트 추출 및 새 탭 여는 동안 표시. |
| | **Disabled** | 비활성화(회색 처리). 트랜스크립트를 사용할 수 없는 영상에서 표시. |
| **패널 내 복사 버튼** | **Default** | 기본 복사 아이콘 |
| | **Success** | 클릭 후 1-2초간 체크(✓) 아이콘으로 변경되며 "복사 완료\!" 툴팁 표시. |

#### **알림 (Toast/Snackbar Alerts)**

  - **성공**: "트랜스크립트가 클립보드에 복사되었습니다." (패널 복사 버튼 클릭 시)
  - **오류/실패**: "이 영상은 트랜스크립트를 제공하지 않습니다." (요약 버튼 클릭 시 스크립트가 없는 경우)
  - **정보**: "AI 챗봇 사이트를 새 탭으로 엽니다." (요약 버튼 클릭 직후)

-----

### **5. URL 구조 및 라우팅 (URL Structure & Routing)**

#### **확장 프로그램 내부 페이지**

  - **옵션 페이지**: `chrome-extension://[EXTENSION_ID]/options.html`
      - **라우팅 전략**: 현재는 단일 뷰이므로 별도의 내부 라우팅이 필요 없습니다. 향후 '고급 설정', '디자인' 등 탭이 추가될 경우, 해시(\#) 기반 라우팅(예: `.../options.html#general`, `.../options.html#appearance`) 도입을 고려할 수 있습니다.
  - **팝업 페이지**: `chrome-extension://[EXTENSION_ID]/popup.html`
      - 라우팅이 필요 없는 단일 뷰입니다.

#### **연결 전략 (Linking Strategy)**

  - **내부 연결**:
      - 팝업 -\> 옵션 페이지: `chrome.runtime.openOptionsPage()` API를 사용하여 옵션 페이지를 엽니다.
  - **외부 연결**:
      - 요약 기능 -\> AI 챗봇: `chrome.tabs.create({ url: '...' })` API를 사용해 사용자가 선택한 AI 챗봇 사이트(예: `https://chat.openai.com`)를 새 탭으로 엽니다. 모든 외부 링크는 `target="_blank"`와 동일하게 동작합니다.

-----

### **6. 컴포넌트 계층 구조 (Component Hierarchy)**

UI를 구성하는 재사용 가능한 컴포넌트들의 관계를 React 스타일의 트리 구조로 나타냅니다.

#### **1. 주입된 UI (Injected UI on YouTube)**

```
<YouTubePage>
  └── <InjectedUIContainer>
      ├── <SummaryButton />
      └── <TranscriptPanel visible={isPanelOpen}>
          ├── <PanelHeader title="Transcript" onClose={() => setPanelOpen(false)} />
          ├── <TranscriptContent text={transcriptText} />
          └── <PanelFooter>
              <CopyButton textToCopy={transcriptText} />
          </PanelFooter>
      </TranscriptPanel>
</InjectedUIContainer>
```

#### **2. 옵션 페이지 (Options Page)**

```
<OptionsPage>
  <Tabs>
    <Tab title="일반 설정" active={true} />
  </Tabs>
  <SettingsContent>
    <AIServiceSelector
      label="요약에 사용할 AI 서비스를 선택하세요:"
      options={['ChatGPT', 'Gemini', 'Claude']}
      selectedValue={storage.aiService}
      onChange={(newValue) => setStorage('aiService', newValue)}
    />
  </SettingsContent>
</OptionsPage>
```

#### **3. 팝업 (Popup)**

```
<PopupContainer>
  <Title text="YouTube 요약기" />
  <ActionButton
    icon="summarize"
    text="현재 영상 요약하기"
    onClick={handleSummarize}
  />
  <ActionButton
    icon="settings"
    text="옵션 페이지 열기"
    onClick={openOptionsPage}
  />
</PopupContainer>
```