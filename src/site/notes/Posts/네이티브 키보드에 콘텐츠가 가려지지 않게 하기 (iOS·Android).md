---
{"dg-publish":true,"permalink":"/posts/i-os-android/","tags":["CSS"],"created":"2026-05-24","updated":"2026-05-24"}
---

모바일 웹에서 화면 하단에 입력창을 고정해 두는 UI는 흔하다. 채팅, 검색, 댓글, AI 챗봇 모두 화면 맨 아래에 입력창을 붙여 둔다. 사용자가 그 입력창을 탭하면 소프트웨어 키보드가 올라오는데, 이때 입력창이나 하단에 고정해 둔 다른 요소가 키보드에 가려지는 일이 생긴다. 이 글은 그 원인과, iOS와 Android에서 실제로 통하는 대응 방법을 각 방법의 표준 지원 현황과 함께 정리한다.

### 원인: 키보드가 뷰포트를 줄이는 방식
이해에 필요한 개념은 두 가지다. 레이아웃 뷰포트(Layout Viewport)는 position: fixed 요소가 위치 기준으로 삼는 영역이고, 비주얼 뷰포트(Visual Viewport)는 사용자에게 실제로 보이는 영역이다. 평소에는 두 크기가 같지만, 핀치 줌을 하거나 키보드가 올라오면 어긋난다.

키보드가 올라올 때 브라우저가 취할 수 있는 동작은 세 가지다. 비주얼 뷰포트만 줄이는 방식, 레이아웃 뷰포트까지 함께 줄이는 방식, 어느 쪽도 줄이지 않고 키보드를 콘텐츠 위에 겹쳐 올리는 방식이다.

과거에는 이 동작이 플랫폼마다 달랐다. iOS Safari는 비주얼 뷰포트만 줄였고, Android의 Chrome과 Firefox는 레이아웃 뷰포트까지 줄였다. 그런데 Chrome 108(2022년 11월)부터 Android Chrome이 레이아웃 뷰포트를 더 이상 줄이지 않고 비주얼 뷰포트만 줄이도록 동작을 바꿔 iOS와 같아졌고, Firefox도 132 버전에서 같은 변경을 적용했다. 그래서 2026년 현재는 iOS와 Android(최신 Chrome·Firefox) 모두 기본적으로 비주얼 뷰포트만 줄인다.

![IMG_3870.gif](/img/user/IMG_3870.gif)

레이아웃 뷰포트가 그대로이므로 position: fixed로 하단에 고정한 요소는 화면 맨 아래 원래 위치에 남고, 그 위로 올라온 키보드에 가려진다. 다만 포커스를 받은 입력 필드 자체는 브라우저가 콘텐츠를 위로 밀어 보이게 해 주는 경우가 많다. 따라서 문제가 두드러지는 쪽은 포커스 대상이 아닌 하단 고정 UI, 즉 전송 버튼이나 툴바, 탭 바 같은 요소다.

![IMG_3871.gif](/img/user/IMG_3871.gif)

기본 동작인 resizes-visual에서는 비주얼 뷰포트만 줄어 position: fixed 입력창이 키보드에 가려진다. interactive-widget을 resizes-content로 지정하면 레이아웃 뷰포트까지 줄어 같은 입력창이 키보드 위에 유지된다. 단, resizes-content는 Chromium과 Firefox(Android)만 지원한다.

### 방법 1. 동적 뷰포트 단위 dvh
전체 화면을 채우는 컨테이너의 높이를 100vh 대신 100dvh로 잡는 방법이다.


```css
.app {
  height: 100vh;   /* 구형 폴백 */
  height: 100dvh;  /* 지원 브라우저가 적용 */
}
```

주의할 점이 있다. 동적 뷰포트 단위는 주로 주소창 같은 브라우저 UI의 노출 여부에 따라 값이 변하도록 정의된 단위다. 가상 키보드에 반응하는지는 브라우저와 버전에 따라 일관적이지 않고, 레이아웃 뷰포트가 변하지 않는 현재 기본 동작에서는 svh와 lvh가 키보드의 영향을 받지 않는다. 따라서 dvh는 전체 레이아웃의 큰 골격을 잡는 데는 쓸 만하지만, 키보드가 떴을 때 하단 입력창을 정확히 키보드 위로 올리는 문제의 신뢰할 만한 단독 해법으로 보기는 어렵다. 아래의 방법 2, 3과 함께 쓰는 토대로 두는 편이 낫다.

### 방법 2. interactive-widget 메타 태그 (표준, Android 한정 구현)

interactive-widget은 CSS Viewport Module Level 1 명세에 정의된 표준 트랙 속성이다. 뷰포트 메타 태그의 키로 지정하며, 키보드 같은 인터랙티브 위젯이 뷰포트를 어떻게 변형할지 선언한다.

```html
<meta name="viewport"
      content="width=device-width, initial-scale=1, interactive-widget=resizes-content">
```

값은 세 가지다.
resizes-visual은 기본값이며 비주얼 뷰포트만 줄인다. 앞서 설명한 현재 기본 동작이 이것이다.
resizes-content는 레이아웃 뷰포트, 즉 초기 컨테이닝 블록(ICB)을 줄인다. 그 결과 하단 고정 요소가 키보드 위에 유지되고 vh 같은 뷰포트 단위 값도 그에 맞춰 작아진다. 채팅 입력 바를 키보드 위에 붙여 두려는 경우에 노리는 동작이다.

overlays-content는 어느 뷰포트도 줄이지 않고 키보드를 콘텐츠 위에 겹쳐 올린다. 뒤에서 설명할 VirtualKeyboard.overlaysContent = true와 같은 동작이다.

구현 현황이 중요하다. 이 속성은 Chromium 108 이상과 Firefox 132 이상에서 지원되지만, Safari/WebKit은 아직 구현하지 않았다(WebKit Bugzilla 259770 이슈가 열려 있다). iOS의 Chrome도 내부가 WebKit이라 적용되지 않는다. 결국 실무에서는 Android의 Chromium 계열과 Firefox에서만 동작하는 선언적 수단이다. 명세에 들어간 표준이지만 엔진 셋 중 둘만 구현한 상태로 이해하면 된다.

### 방법 3. visualViewport API (표준, Safari 포함 전 브라우저)
iOS Safari에서는 interactive-widget이 통하지 않으므로, 하단 입력창을 키보드 위로 올리려면 스크립트로 직접 보정해야 한다. 이때 쓰는 것이 visualViewport API다.

표준 여부를 보면, VisualViewport 인터페이스는 CSSOM View Module에 정의된 표준이고 iOS Safari를 포함한 모든 주요 브라우저에서 지원된다. 이 글에서 다루는 방법 중 지원 범위가 가장 넓은 표준이며, Safari에서 키보드 가림을 다루는 사실상의 표준 수단이다.

window.visualViewport 객체는 보이는 영역의 상태를 노출한다. 주요 속성은 width와 height, 레이아웃 뷰포트 기준 오프셋인 offsetLeft와 offsetTop, 문서 기준 좌표인 pageLeft와 pageTop, 확대 배율 scale이다. 이벤트로는 resize와 scroll을 제공한다. height는 키보드가 뜨면 그만큼 줄어든 보이는 영역의 높이이고, offsetTop은 브라우저가 포커스된 입력을 보이게 하려고 콘텐츠를 위로 밀 때 커진다.

키보드가 가린 높이는 직접 주어지지 않으므로 역산한다. 줄지 않는 레이아웃 뷰포트의 높이인 window.innerHeight에서 보이는 영역의 높이와 오프셋을 빼면 키보드가 차지한 높이를 추정할 수 있다.

```js
const vv = window.visualViewport;
const composer = document.querySelector(".composer");

let raf = 0;
function sync() {
  cancelAnimationFrame(raf);
  raf = requestAnimationFrame(() => {
    // 레이아웃 뷰포트 높이 - 보이는 영역 높이 - 오프셋 = 키보드가 가린 높이
    const overlap = window.innerHeight - vv.height - vv.offsetTop;
    composer.style.transform = `translateY(${-Math.max(0, overlap)}px)`;
  });
}

vv.addEventListener("resize", sync);
vv.addEventListener("scroll", sync);
```

resize뿐 아니라 scroll도 함께 듣는 이유가 있다. position: fixed 요소는 레이아웃 뷰포트를 기준으로 배치되므로 비주얼 뷰포트를 따라 움직이지 않는데, 브라우저가 포커스 입력을 보이게 하려고 콘텐츠를 위로 밀면 그 변화가 scroll 이벤트로 통지되기 때문이다. 두 이벤트는 짧은 시간에 여러 번 발생하므로 requestAnimationFrame으로 묶어 레이아웃 갱신을 한 번으로 줄인다.

한계도 분명하다. 이 방식은 키보드 높이를 직접 알려 주는 API가 아니라 뷰포트 크기 변화에서 역산하는 휴리스틱이다. window.innerHeight가 기기에 따라 키보드 영역을 포함하는지 여부가 달라 미세하게 어긋날 수 있고, 키보드 앱이나 예측 입력 바가 비동기로 로드되면 값이 한 박자 늦게 안정된다. 그래도 Safari에서 표준으로 동작하는 방법은 현재 이것뿐이다.
### 방법 4. VirtualKeyboard API와 2026년 현황
VirtualKeyboard API는 키보드 영역을 표준 환경 변수로 노출해 정밀 제어를 가능하게 하려는 목적의 표준이다. 

navigator.virtualKeyboard.overlaysContent를 true로 두어 브라우저의 자동 리사이즈를 끄고, geometrychange 이벤트나 env(keyboard-inset-height) 같은 CSS 환경 변수로 키보드 영역을 직접 다룬다. 이 overlaysContent = true는 앞서 본 interactive-widget의 overlays-content 값과 같은 동작이다.

```js
if ("virtualKeyboard" in navigator) {
  navigator.virtualKeyboard.overlaysContent = true;
}
```

```css
.composer {
  position: fixed;
  bottom: env(keyboard-inset-height, 0px);
}
```

문제는 구현 현황이다. Safari는 이 API를 전혀 지원하지 않는다. navigator.virtualKeyboard도, geometrychange도, 관련 CSS 환경 변수도 없다. Firefox도 2021년부터 구현 이슈가 열려 있는 채 미지원이며 2026년 상호운용 우선순위에도 들어 있지 않다. 결국 이 API는 Android의 Chromium 계열에서만 동작한다.

그 Chromium 구현마저도 신뢰하기 어렵다. 보고되는 키보드 높이가 최종 정착 높이를 잠깐 넘겼다가 다시 내려오는 오버슈트 현상이 있고, boundingRect가 명세가 요구하는 좌표계와 어긋나며, CSS의 inset 변수들이 실제 inset이 아니라 원시 좌표값을 담고 있다. 실무에서 의도대로 동작하는 값은 keyboard-inset-height 정도라서, bottom: env(keyboard-inset-height) 패턴만 사실상 우연히 맞아떨어진다.

### 정리
2026년 기준 기본 동작은 iOS와 Android(최신 Chrome·Firefox) 모두 비주얼 뷰포트만 줄이는 것이다. 그래서 별도 처리를 하지 않으면 하단 고정 입력창은 양쪽 모두에서 키보드에 가려진다.
Android의 Chromium 계열과 Firefox에서는 interactive-widget=resizes-content로 선언적으로 해결할 수 있다. 표준 명세에 정의된 속성이고 코드도 메타 태그 한 줄이다.

iOS Safari에서는 interactive-widget과 VirtualKeyboard API가 모두 미지원이므로 visualViewport API로 직접 보정해야 한다. 지원 범위가 가장 넓은 표준이라는 점에서 크로스브라우저 폴백으로도 이 방법을 기준으로 잡는 편이 안전하다.

VirtualKeyboard API는 Android Chromium에서 env(keyboard-inset-height)를 보조적으로 쓰는 선에서 활용하고 전면적으로 의존하지 않는다.

요약하면 선언적 해결은 Android에 한해 interactive-widget=resizes-content로, iOS를 포함한 전 범위 대응은 visualViewport API로 처리하는 조합이 가장 현실적인 방법이다.