## 7.2 요소 탭
### 7.2.2 요소 정보
  - 스타일(Styles): 요소와 관련된 스타일 정보를 나타낸다.
  - 계산됨(Computed): 해당 요소의 크기, 패딩, 보더, 마진과 각종 CSS 적용 결괏 값을 알 수 있는 탭이다.
  - 레이아웃(Layout): CSS 그리드나 레이아웃과 관련된 정보 확인할 수 있다.
  - 이벤트 리스너(Event Listeners): 현재 요소에 부착된 각종 이벤트 리스너를 확인할 수 있다. 그러나 이벤트 버블링 등으로 이벤트를 발생 시키는 경우 리액트 17의 변경 사항 중 이벤트 위임의 변경에 의해 확인할 수 없다.
  - DOM 중단점: 중단점이 있는지 알려주는 탭
  - 속성(Properties): 해당 요소가 가지고 있는 모든 속성값을 나타낸다. 해당 요소가 가지고 있는 모든 값이 나옴(.attributes는 직접 할당된 값만 나옴)
  - 접근성(Accessibility): 웹 이용에 어려움을 겪는 장애인, 노약자를 위한 스크린리더기 등이 활용하는 값을 말

### 리액트에서 이벤트 리스너 처리 과정
  - 일단 결론만 말하자면 `button` 이벤트 발생 시 리액트는 상위 루트 요소를 한 번만 감지해서 내부적으로 SyntheticEvent(합성 이벤트)로 처리한다.
  - 리액트 16 이하: 모든 이벤트를 document에 위임하여 `document.addEventListener('click', ...)` 과 같은 형태로 전역에서 감지
  - 리액트 17 이후: 리액트 앱의 루트 컨테이너로 변경되면서 `#root` 같은 루트 요소에 이벤트를 등록하게 됨

    - 이벤트 버블링
      - 한 요소에서 이벤트 발생 시 핸들러 동작으로 부모 요소의 핸들러가 동작하고 가장 최상단 조상 요소를 만날 때까지 반복 되면서 각 할당된 핸들러가 동작하는게 이벤트 버블링이다.

      ```
        // p에 할당된 onclick 동작 시 div -> form -> document 객체 만날 때까지 각 요소 onClick 핸들러 동작
        <form onclick="alert('form')">
          FORM
          <div onclick="alert('div')">
            DIV
            <p onclick="alert('p')">
              P
            </p>
          </div>
        </form>
      ```
    - 이벤트 캡처링
      - 자식 요소에서 발생한 이벤트가 부모 요소부터 시작하여 이벤트를 발생시킨 자식 요소까지 도달되는 것을 말함.

      ```
      <html lang="ko">
        <body>
          <div id="grandparent" style="padding:20px; background:#e6e6ff;">
            grandparent
            <div id="parent" style="padding:20px; background:#d1ffd1;">
              parent
              <button id="child" style="padding:10px;">child (click me)</button>
            </div>
          </div>

          <script>
            const grandparent = document.getElementById('grandparent');
            const parent = document.getElementById('parent');
            const child = document.getElementById('child');

            function log(phase, id) {
              console.log(`${phase} → ${id}`);
            }

            // ---- 캡처링 단계 리스너들 (true) ----
            grandparent.addEventListener('click', () => log('capture', 'grandparent'), true);
            parent.addEventListener('click', () => log('capture', 'parent'), true);
            child.addEventListener('click', () => log('capture', 'child'), true);

            // ---- 버블링 단계 리스너들 (false / 기본값) ----
            grandparent.addEventListener('click', () => log('bubble', 'grandparent'));
            parent.addEventListener('click', () => log('bubble', 'parent'));
            child.addEventListener('click', () => log('bubble', 'child'));
          </script>
        </body>
      </html>
      ```

