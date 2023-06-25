---
title: "TypeScript + TailwindCSS로 프로젝트 시작하기"
description: "TypeScript 프로젝트 세팅"
date: 2023-06-24
update: 2023-06-24
tags:
  - ReactJS
  - TypeScript
  - CRA
  - TailwindCSS
  - Blog
series: "Project"
---

CRA로 TypeScript와 React, CSS로 TailwindCSS를 세팅해보자.

## 1. Create React App(CRA)

> `npx create-react-app 프로젝트명 --template typescript`

## 2. TailwindCSS 설치하기

> `yarn add -D tailwindcss postcss autoprefixer`
> or
> `npm i -D tailwindcss postcss autoprefixer`

- `D 옵션`으로 패키지를 개발 종속성으로 추가한다.

## 3. TailwindCSS config 파일 초기화

`yarn tailwind init -p`

```Javascript
./tailwind.config.js

module.exports = {
  content: [
  	"./src/**/*.{js,jsx,ts,tsx}",
    // src 하위 파일 중 확장자가 .js,.jsx,.ts,.tsx인 파일을 대상으로 한다는 의미
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

## 4. TailwindCSS 적용

src/index.css 상단에

```Css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

를 추가해준다.
