## 9.1 Next.js로 리액트 개발 환경 구축하기
## 9.1.1 `create-next-app` 없이 하나씩 구축하기
  - `package.json` 설정 (본 설정은 next 12 버전 기준)
    - npm i react@18.2.0 react-dom@18.2.0 next@12.3.1
    - npm i -D @types/node@18.8.5 @types/react@18.0.21 @types/react-dom@18.0.6
    - npm i -D eslint@8.38.0 prettier@2.8.7 eslint-config-next@12.3.1
  ```
  {
    "name": "no-create-app-next",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
        "next": "^12.3.1",
        "react": "^18.2.0",
        "react-dom": "^18.2.0"
    },
    "devDependencies": {
        "@types/node": "^18.8.5",
        "@types/react": "^18.0.21",
        "@types/react-dom": "^18.0.6",
        "eslint": "^8.38.0",
        "eslint-config-next": "^12.3.1",
        "prettier": "^2.8.7"
    }
  }
  ```
## 9.1.2 `tsconfig.json` 작성하기
  - `compilerOptions`: 타입스크립트를 자바스크팁트로 컴파일할 때 사용하는 옵션
    - `target`: 변환 목표 언어의 버전 es6로 작성된 코드가 있다면 es5로 변환 되지만 폴리필까지 지원하진 않음
    - `lib`: 폴리필 환경이 준비되어도 타입스크립트는 `Promise`나 `Map`의 존재를 모르기에 가장 최근 버전인 esnext를 추가하여 API 정보를 확인할 수 있게 하여 에러 발생을 막는다. dom의 경우는 타입스크립트 환경에서 브라우저 위주 API 명세를 사용 할 수 있게 하기 위해서 쓰인다.(window.document 등)
    - `allowJs`: 자바스크립트를 컴파일 할지 결정하는데 이는 `ts`와 `js`를 혼재한 경우 사용하는 옵션이다.
    - `skipLibCheck`: `d.ts`의 라이브러리 검사 여부 결정
    - `strict`: 엄격 검사 여부
    - `forceConsistentCasingInFileNames`: 파일 이름 대소문자 구분
    - `noEmit`: 컴파일 하지 않고 타입만 체크 한다. next와 같이 swc는 굳이 자바스크립트로 컴파일 할 필요가없다.
    - `esModuleInterop` : CommonJS 방식으로 보낸 모듈을 ES 모듈 방식으로 import 하게 도와줌
    - `moduleResolution`: 모듈 해석 node는 node_module 기준 / classic은 tsconfig.json 기준 해석
    - `resolveJsonModule`: JSON 파일 import 하게 해줌 allowJs 옵션도 자동 켜짐
    - `isolatedModules`: 타입스크립트 컴파일러는 import나 export가 없다면 단순 스크립트 파일로 인식한다. 그렇기 때문에 파일 생성을 막기 위한 모듈로 강제화 하는 옵션이다.
    - `jsx`: .tsx 파일 내부에 있는 jsx 파일 컴파일 설정이다.
    - `incremental`: 타입스크립 컴파일러의 컴파일 속도 개선
  ```
  {
    // schemaStore에서 제공해 주는 정보로 JSON 파일이 무엇을 의미하고, 또 어떤 키와 어떤 값이 들어갈 수 있는지 알려주는 도구
    "$schema": "https://json.schemastore.org/tsconfig.json",
    "compilerOptions": { 
      "target": "es5",
      "lib": ["dom", "dom.iterable", "esnext"],
      "allowJs": false,
      "skipLibCheck": true,
      "strict": true,
      "forceConsistentCasingInFileNames": true,
      "noEmit": true,
      "esModuleInterop": true,
      "module": "esnext",
      "moduleResolution": "node",
      "resolveJsonModule": true,
      "isolatedModules": true,
      "jsx": "preserve",
      "incremental": true,
      "baseUrl": "src",
      "paths": {
        "#pages/*": ["pages/*"],
        "#hooks/*": ["hooks/*"],
        "#types/*": ["types/*"],
        "#components/*": ["components/*"],
        "#utils/*": ["utils/*"]
      }
    },
    "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
    "exclude": ["node_modules"]
  }
  ```
## 9.1.3 `next.config.js` 작성하기
  - `reactStrictMode`: 리액트 엄격모드
  - `poweredByHeader`: `X-Powered-By` 헤더 제거
  - `ignoreDuringBuilds`: 빌드 시 eslint 수행 제거 
  ```
  /** @type {import('next').NextConfig} */
  const nextConfig = {
      reactStrictMode: true,
      poweredByHeader: false,
      eslint: {
        ignoreDuringBuilds: true,
      },
  }

  module.exports = nextConfig
  ```