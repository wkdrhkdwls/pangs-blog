---
title: "TypeScript로 진경옥몰 초기 세팅하기"
description: "TypeScript 프로젝트 세팅"
date: 2023-06-27
update: 2023-06-27
tags:
  - ReactJS
  - TypeScript
  - Setting
  - Blog
  - 진경옥몰
series: "Project"
---

## 1. Typescript로 세팅하기

### 1-1. Create React App(CRA)

> `npx create-react-app 프로젝트명 --template typescript`

으로 원하는 폴더에 typescript로 프로젝트를 할 기반 설정 및 설치를 한다.
CRA로 설치 시 장점은 babel이나 webpack등을 기본적으로 설정해준다는 것이다.
터미널은 윈도우기준 `git bash`를 사용한다.

- 그러나 CRA는 Webpack 설정을 숨겨놓고 있기때문에, 수정하기 힘들다는 단점이 있다.

## 2. TailwindCSS 설치하기

- 이번 프로젝트는 CSS 프레임워크를 TailwindCSS로 정하여 세팅을 해주기로 했다. 왜냐면 일단 너무 간편하기도하고 평소에 주로 사용해봤기때문에...

> `npm i -D tailwindcss postcss autoprefixer`

### 2-1 tailwind.config.js 만들기

> `npx tailwindcss init`명령어를 사용하여 tailwind.config.js 파일을 생성한다.

```Javascript
module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

content안에 위와 같이 설정해준다(src폴더 사용경우)

### 2-2 TailwindCSS 적용

src/index.css 상단에

```Css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

를 추가해준다.

## 3. Redux 설치

> ` npm i redux react-redux`
>
> `npm i --dev @types/react-redux `
>
> `npm i @reduxjs/toolkit`

상태관리를 위해 Redux를 설치한다. 기본적인 구조는 [참고블로그](https://bum-developer.tistory.com/entry/React-Redux%EC%99%80-TypeScript-%ED%95%A8%EA%BB%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)를 참고하여 구성하였다. 리덕스에 대한 설명은 다음번에 추가적으로 하도록 하겠다.

## 4. Loadable 설치

> `npm i @loadable/component`
>
> `npm i --save-dev @types/loadable/component`

코드 스플리팅을 위해 설치한다. SPA의 단점인 초기 데이터 요청 시간이 길어짐을 보완할 수 있다.

## 5. React-Router-DOM 설치하기

> `npm i react-router-dom`
>
> `npm i react-router-dom @types/react-router-dom`
>
> `npm i --save @types/react @types/react-dom`

### 5-1. 라우터 구성

```Typescript
./App.tsx

import React from "react";
import loadable from "@loadable/component";
import { Route, Routes } from "react-router-dom";
const Home = loadable(() => import("./pages/Home/index"));
const LogIn = loadable(() => import("./pages/LogIn/index"));
const SignUp = loadable(() => import("./pages/SignUp/index"));
const Product = loadable(() => import("./pages/Product/index"));

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/login" element={<LogIn />} />
      <Route path="/signup" element={<SignUp />} />
      <Route path="/product/:id" element={<Product />} />
    </Routes>
  );
}

export default App;
```

loadable로 코드 스플리팅을 해주고 우선 필요할 것 같은 라우팅만 설정해준다.

## 6. tsconfig paths를 절대경로로 설정하기

원래는 tsconfig.json을

```Json
./tsconfig.json

"baseUrl": "./src",
"paths": {
    "@hooks/*": ["./hooks/*"],
    "@components/*": ["./components/*"],
    "@pages/*": ["./pages/*"],
    "@utils/*": ["./utils/*"],
    "@store/*": ["./store/*"],
    "@reducer/*": ["./reducer/*"]
}
```

와 같이 수정하면 절대경로로 설정할 수 있다. 그러나 CRA로 구축했을때는 tsconfig.json에서 path를 인식하지 못하는 문제가 발생한다. CRA 내부의 Webpack 설정 때문에 tsconfig를 변경한 내용이 적용되지 않고 초기에 생성된 tsconfig 설정으로 돌아가기 때문이라고 한다.

> `npm i @craco/craco`
>
> `npm i -D craco-alias`
>
> `npm i --save-dev react-app-alias`
> 를 설치한 후 `루트 디렉토리`에 craco.config.js를 만든 후

```JavaScript
./craco.config.js

const { CracoAliasPlugin } = require('react-app-alias');

module.exports = {
  plugins: [
    {
      plugin: CracoAliasPlugin,
      options: {
        source: 'tsconfig',
        baseUrl: '.',
        tsConfigPath: './tsconfig.paths.json',
      },
    },
  ],
};
```

와 같이 설정한다.<br/>
그 후 덮어쓸 내용만 담은 tsconfig를 `tsconfig.paths.json`으로 따로 생성한 후

```Json
./tsconfig.paths.json

  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["./components/*"],
      "@hooks/*": ["./hooks/*"],
      "@pages/*": ["./pages/*"],
      "@reducer/*": ["./reducer/*"],
      "@store/*": ["./store/*"],
    }
  }
}
```

와 같이 작성한다.<br/>
(`tsconfig.paths.json`에 주석을 달면 에러가 발생한다.)<br/>
`tsconfig`에는 extends 속성을

```Json
./tsconfig.json

{
  "extends": "./tsconfig.paths.json",
  "compilerOptions": {
    // ...
  }
}
```

와 같이 추가한 후 `package.json`에서 `react-scripts`명령어를 craco로 변경한다.

```Json
./package.json

{
  "scripts": {
    "start": "craco start",
    "build": "craco build",
    "test": "craco test",
    "eject": "react-scripts eject"
  }
}
```

---

## 다음

상품의 결제페이지를 만드는 방법을 정리하도록 하겠다.
