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
    - 이벤트 위임
      - 사용자의 액션에 의해 이벤트 발생 시 이벤트는 버블링에 의해 document 레벨까지 올라가게 되는데 이로 인해 자식 엘리먼트에서 발생하는 이벤트를 부모가 감지 할 수 있는데 이런 작동 방식을 이용해 자식 엘리먼트의 이벤트를 부모로 위임 시켜 처리하는 방식을 말합니다.
      - 동적 엘리먼트 생성 시에도 유용 하다.

    - `event.currentTarget` vs `event.target`
      - `event.currentTarget`: 실제 누른 주체로 실행되는 요소라 보면 된다.
      - `event.target`: 이벤트를 발생 시킨 위치라 보면 된다.
    ```
    <html>
      <body>  
        <div id="parent" style="padding: 30px; background: #cde;">
          parent
          <button id="child" style="padding: 10px;">child</button>
        </div>
      </body>
      <script>
        const parent = document.getElementById('parent');
        const child = document.getElementById('child');

        // 이벤트 위임을 사용해서 비교 해보기
        parent.addEventListener('click', (e) => {
          console.log('parent 리스너 실행!');
          console.log('event.target:', e.target); // parent 엘리먼트를 누르면 parent child를 누르면 child 
          console.log('event.currentTarget:', e.currentTarget); // parent 내에서 어딜 누르던 parent 엘리먼트
        });
      
        // 이건 이벤트 영역이 child로 되어 있기때문에 child 엘리먼트만 나온다.
        <!-- child.addEventListener('click', (e) => {
          console.log('child 리스너 실행!');
          console.log('event.target:', e.target);
          console.log('event.currentTarget:', e.currentTarget);
        }); -->
      </script>
    </html>

    <!-- 리액트에서는 어떻게 사용 할까? -->
    // ul로 handleClick를 위임 시켰지만 li 각 요소들을 정확하게 구분하기 위해서는 e.target를 통해 실제 누른 주체를 찾는다.
    function List({ items }) {
      const handleClick = (e) => {
        const target = e.target.closest('li'); // 실제 클릭된 li 찾기
        if (!target) return;
        console.log('clicked item id:', target.dataset.id);
      };

      return (
        <ul onClick={handleClick}>
          {items.map((item) => (
            <li key={item.id} data-id={item.id}>
              {item.name}
            </li>
          ))}
        </ul>
      );
    }
    ```

    - 이벤트 위임 예시 전 자식 엘리먼트에 각각 `addEventListener`를 부여 하는 경우 메모리 측면에서 문제가 있는 예시
    ```
    <html>
      <body>
        <div class="menu">
          <button class="items">첫번째 버튼</button>
          <button class="items">두번째 버튼</button>
          <button class="items">세번째 버튼</button>
        </div>
        <script>
          for (const item of document.querySelectorAll(".items")) {
            item.addEventListener("click", (e) => {
              alert(`${e.target.innerText}입니다.`);
            });
          }
        </script>
      </body>
    </html>
    ```

    - `이벤트 버블링`을 통한 부모 엘리먼트에게 이벤트를 위임하는 코드
    ```
    <html>
      <body>
        <div class="menu">
          <button class="items">첫번째 버튼</button>
          <button class="items">두번째 버튼</button>
          <button class="items">세번째 버튼</button>
        </div>
        <script>
          const menu = document.querySelector(".menu");
          menu.addEventListener("click", (e) => {
            alert(`${e.target.innerText}입니다.`);
          });
        </script>
      </body>
    </html>
    ```

    - 동적 엘리먼트 생성으로 이벤트를 위임할떄 효율적이다.
    ```
    <!DOCTYPE html>
    <html>
      <body>
        <button class="create">CREATE</button>
        <div class="menu">
          <button class="items">첫번째 버튼</button>
          <button class="items">두번째 버튼</button>
          <button class="items">세번째 버튼</button>
        </div>
        <script>
          const menu = document.querySelector(".menu");
          const create = document.querySelector(".create");

          create.addEventListener("click", () => {
            menu.innerHTML += `
              <button class="items">네번째 버튼</button>
              <button class="items">다섯번째 버튼</button>
            `;
          });

          // create 버튼을 누르기 전에 이미 이벤트 리스너가 등록되어 있기때문에 네번째, 다섯번째 버튼 클릭 시는 이벤트 발생이 안된다.
          for (const item of document.querySelectorAll(".items")) {
            item.addEventListener("click", (e) => {
              alert(`${e.target.innerText}입니다.`);
            });
          }
        </script>
      </body>
    </html>


    <html>
      <body>
        <button class="create">CREATE</button>
        <div class="menu">
          <button class="items">첫번째 버튼</button>
          <button class="items">두번째 버튼</button>
          <button class="items">세번째 버튼</button>
        </div>
        <script>
          const menu = document.querySelector(".menu");
          const create = document.querySelector(".create");

          create.addEventListener("click", () => {
            menu.innerHTML += `
              <button class="items">네번째 버튼</button>
              <button class="items">다섯번째 버튼</button>
            `;
          });

          // 부모로 위임을 함으로서 버블링 이벤트에 의해 이벤트가 동작한다.
          menu.addEventListener("click", (e) => {
            alert(`${e.target.innerText}입니다.`);
          });
        </script>
      </body>
    </html>
    ```

    - 리액트 이벤트 처리 방식
      - li 태그에 1만개의 onClick를 구현해도 react는 리스너를 1만개 만들까?
        => 결론은 아니다. 리액트 내부 위임 방식으로 인해 17 버전 이후 부터는 root Dom Container 에 SyntheticEvent를 통해 이벤트를 위임 하여 처리를 한다. 하지만 클로저 메모리에는 약간 문제가 있는게 분명하다.
        => 16 버전 이하에서는 Native Dom 이벤트로 이벤트를 위임 시켜 document 전역 위임 시킴
          - 다중 React 앱 충돌로 인해 인식이 어려웠다는 점을 찾을 수 있음
    ```
    function App() {
      const [items, setItems] = useState(['항목1', '항목2', '항목3'])
      const handleClick(e, item) => {
        alert('ㅎㅇㅎㅇ')
      }
      return (
        <div>
          <ul>
            {items.map((item, index)=> {
              return {
                <li key={index} onClick={handleClick(item)}>{item}</li>
              }
            })}
          </ul>
        </div>
      )
    }
    ```
## 7.6 Next 환경 디버깅하기
  - `npx create-next-app@12 my-next-app`
  - ApacheBench 설치를 통한 성능 측정 방법
    - https://www.apachelounge.com/download/
    - 압축 해제 > bin > exe 경로 시스템 환경변수 PATH로 등록
    - ab -k -c 50 -n 10000 http://127.0.0.1:3000/ > 테스트
      - 명령어 옵션 설명
        - `-k` : Keep-Alive 사용 (HTTP 커넥션 재사용)
        - `-c 50`: 동시 접속자 50명
        - `-n 10000` : 총 10,000번 요청
        ```
        This is ApacheBench, Version 2.3 <$Revision: 1923142 $>
        Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
        Licensed to The Apache Software Foundation, http://www.apache.org/

        Benchmarking 127.0.0.1 (be patient)
        Completed 1000 requests
        Completed 2000 requests
        Completed 3000 requests
        Completed 4000 requests
        Completed 5000 requests
        Completed 6000 requests
        Completed 7000 requests
        Completed 8000 requests
        Completed 9000 requests
        Completed 10000 requests
        Finished 10000 requests


        Server Software:
        Server Hostname:        127.0.0.1
        Server Port:            3000

        Document Path:          /
        Document Length:        899 bytes

        Concurrency Level:      50
        Time taken for tests:   3.930 seconds
        Complete requests:      10000
        Failed requests:        0
        Keep-Alive requests:    10000
        Total transferred:      11290000 bytes
        HTML transferred:       8990000 bytes
        Requests per second:    2544.22 [#/sec] (mean)
        Time per request:       19.652 [ms] (mean)
        Time per request:       0.393 [ms] (mean, across all concurrent requests)
        Transfer rate:          2805.10 [Kbytes/sec] received

        Connection Times (ms)
                      min  mean[+/-sd] median   max
        Connect:        0    0   0.0      0       1
        Processing:     7   20  10.9     15      75
        Waiting:        4   20  10.9     15      75
        Total:          7   20  10.9     15      75

        Percentage of the requests served within a certain time (ms)
          50%     15
          66%     17
          75%     20
          80%     22
          90%     32
          95%     49
          98%     57
          99%     63
        100%     75 (longest request)
        ```
  - Chrome inspect를 통한 디버깅 방법
    - `"dev": "NODE_OPTIONS='--inspect' next dev",` 혹은 `"dev": "cross-env NODE_OPTIONS=--inspect next dev",`

    - 실행 시 아래와 같은 ws 주소가 나오면 직접 접근
      - node 관련해서 debugger은 아래에서 자세하게 디버깅이 가능함
    ```
      devtools://devtools/bundled/inspector.html?v8only=true&ws=127.0.0.1:9230/95433c0e-a282-4973-9d1b-ff494cbdd273
    ```
