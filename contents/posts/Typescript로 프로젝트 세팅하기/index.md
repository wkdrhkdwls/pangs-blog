---
title: "TypeScript로 프로젝트 세팅하기"
description: "TypeScript 프로젝트 세팅"
date: 2023-06-23
update: 2023-06-23
tags:
  - ReactJS
  - TypeScript
  - Setting
  - Blog
series: "TypeScript"
---

CRA없이 TypeScript와 React를 세팅하는 방법이다.

## 1. package.json 생성

- `npm init`으로 package.json을 생성한다
- `npm i react react-dom`
- `npm i typescript @types/react @types/react-dom`

## 2. Eslint

### eslint란

- 코드 점검 도구를 말한다. 직접 설정하면 팀원간 의견 충돌이 있으니 prettier에 위임한다.
- `npm i -D eslint`
- `.eslintrc`를 생성한다.

```json
/.eslintrc
{
  "extends": ["plugin:prettier/recommended"]
}
```

## 3. Prettier

- `npm i -D prettier eslint-plugin-prettier eslint-config-prettier`
- `.prettierrc`를 생성한후

```json
/.prettierrc
{
  "printWidth": 120,
  "tabWidth": 2,
  "singleQuote": true,
  "trailingComma": "all",
  "semi": true
}
```

를 넣어 기본세팅을 완료한다.

- `tabWidth`는 개행시 띄워지는 정도를 의미하고 `printWidth`는 한줄에 120이하까지만 적용시키겠다는 의미이다.

## 4. TypeScript 설정

- 언어 문법과 JavaScript 결과물이 어떻게 나와야하는지 설정하는 파일
- TypeScript 설정파일인 tsconfig.json파일을 생성한

```json
./tsconfig.json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "sourceMap": true,
    "lib": ["ES2020", "DOM"],
    "jsx": "react",
    "module": "esnext",
    "moduleResolution": "Node",
    "target": "es5",
    "strict": true, // true로 켜놓아야 타입을 체킹한다.
    "resolveJsonModule": true,
    "baseUrl": ".",
    "paths": {
      "@hooks/*": ["hooks/*"],
      "@components/*": ["components/*"],
      "@layouts/*": ["layouts/*"],
      "@pages/*": ["pages/*"],
      "@utils/*": ["utils/*"],
      "@typings/*": ["typings/*"]
    }
  },
  "ts-node": {
    "compilerOptions": {
      "module": "commonjs",
      "moduleResolution": "Node",
      "target": "es5",
      "esModuleInterop": true
    }
  }
}
```

## 5. Webpack 설정

- ts, css, json, 최신 문법의 js파일들을 하나로 합쳐준다.
- entry에서 파일을 선택하면 module에 정해진 rules대로 js로 변환하여 하나의 파일로 합쳐준다(output). plugins는 합치는 중 부가적인 효과를 준다.
- ts는 babel-loader로, css는 style-loader와 css-loader를 통해 js로 변환한다.
- babel에서는 @babel/preset-env(최신문법 전환) @babel/preset-react(리엑트 jsx 변환), @babel/preset-typescript(타입스크립트 변환)
- publicPath가 /dist고 [name].js에서 [name]이 entry에 적힌대로 app으로 바뀌어 /dist/app.js가 결과물이 된다.
- `npm i -D webpack @types/webpack @types/node`
- `npm i -D css-loader style-loader @babel/core babel-loader @babel/preset-env @babel/preset-react @babel/preset-typescript`
- `npm i style-loader css-loader`

```Typescript
./webpack.config.ts

import path from 'path';
import ReactRefreshWebpackPlugin from '@pmmmwh/react-refresh-webpack-plugin';
import webpack, { Configuration as WebpackConfiguration } from 'webpack';
import { Configuration as WebpackDevServerConfiguration } from 'webpack-dev-server';

interface Configuration extends WebpackConfiguration {
  devServer?: WebpackDevServerConfiguration;
}

import ForkTsCheckerWebpackPlugin from 'fork-ts-checker-webpack-plugin';

const isDevelopment = process.env.NODE_ENV !== 'production';

const config: Configuration = {
  name: 'sleact',
  mode: isDevelopment ? 'development' : 'production',
  devtool: !isDevelopment ? 'hidden-source-map' : 'eval',
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json'],
    alias: {
      '@hooks': path.resolve(__dirname, 'hooks'),
      '@components': path.resolve(__dirname, 'components'),
      '@layouts': path.resolve(__dirname, 'layouts'),
      '@pages': path.resolve(__dirname, 'pages'),
      '@utils': path.resolve(__dirname, 'utils'),
      '@typings': path.resolve(__dirname, 'typings'),
    },
  },
  entry: {
    app: './client',
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                targets: { browsers: ['IE 10'] },
                debug: isDevelopment,
              },
            ],
            '@babel/preset-react',
            '@babel/preset-typescript',
          ],
          env: {
            development: {
              plugins: [require.resolve('react-refresh/babel')],
            },
          },
        },
        exclude: path.join(__dirname, 'node_modules'),
      },
      {
        test: /\.css?$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ForkTsCheckerWebpackPlugin({
      async: false,
      // eslint: {
      //   files: "./src/**/*",
      // },
    }),
    new webpack.EnvironmentPlugin({ NODE_ENV: isDevelopment ? 'development' : 'production' }),
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js',
    publicPath: '/dist/',
  },
  devServer: {
    historyApiFallback: true, // react router
    port: 3090,
    devMiddleware: { publicPath: '/dist/' },
    static: { directory: path.resolve(__dirname) },
    proxy: {
      '/api/': {
        target: 'http://localhost:8080', //post보낼 api주소를 작성한다.
        changeOrigin: true,
      },
    }, //proxy를 api주소로 설정해준다.
  },
};

if (isDevelopment && config.plugins) {
  config.plugins.push(new webpack.HotModuleReplacementPlugin());
  config.plugins.push(new ReactRefreshWebpackPlugin());
}
if (!isDevelopment && config.plugins) {
}

export default config;
```

## 6. index.html 작성

```Html
./index.html

<html>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>타이틀</title>
    <style>
      html, body {
          margin: 0;
          padding: 0;
          overflow: initial !important;
      }
      body {
          font-size: 15px;
          line-height: 1.46668;
          font-weight: 400;
          font-variant-ligatures: common-ligatures;
          -moz-osx-font-smoothing: grayscale;
          -webkit-font-smoothing: antialiased;
      }
      * {
          box-sizing: border-box;
      }
    </style>
  </head>
  <body>
    <div id="app"></div>
    <script src="/dist/app.js"></script>
  </body>
</html>
```

```Typescript
./Client.tsx

import 'core-js/stable';
import 'regenerator-runtime/runtime';
import React from 'react';
import { render } from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import App from '@layouts/App';

render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.querySelector('#app'),
);
```

## 7. tsconfig-for-webpack-config.json

- `npm i cross-env`
- webpack할 때 webpack.config.ts를 인식 못하는 문제
- package.json의 scripts의 build를 cross-env TS_NODE_PROJECT=\"tsconfig-for-webpack-config.json\" webpack

```Json
./package.json

"scripts": {
    "dev": "webpack serve --env development",
    "build": "cross-env NODE_ENV=production webpack"
  },
```

- `npm run build`
- index.html 실행해보기

```Json
./tsconfig-for-webpack-config.json

{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "Node",
    "target": "es5",
    "esModuleInterop": true
  }
}
```

## 8. Webpack Dev 서버 설치

- 개발용 서버인 devServer 옵션 추가(port는 3090, publicPath는 /dist/로)
- webpack serve할 때 webpack.config.ts를 인식 못하는 문제
- `npm i -D ts-node webpack-dev-server @types/webpack-dev-server webpack-cli`
- package.json의 scripts의 dev를 cross-env TS_NODE_PROJECT=\"tsconfig-for-webpack-config.json\" webpack serve --env development

```Json
./package.json

"scripts": {
    "dev": "webpack serve --env development",
    "build": "cross-env NODE_ENV=production webpack"
  },
```

- npm run dev하면 localhost:3090에서 서버 실행됨.

## 9. Hot reloading 설정

- 새로고침 하면 자동으로 업데이트 되는 기능
- CRA에서는 자동으로 내장되어있지만, 원래는 일일이 설정해줘야하는 기능이다.
- `npm i -D @pmmmwh/react-refresh-webpack-plugin react-refresh`
- webpack의 babel-loader 안에 설정(env) 및 plugin으로 추가

```Typescript
./webpack.config.ts

...
...
...

module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                targets: { browsers: ['IE 10'] },
                debug: isDevelopment,
              },
            ],
            '@babel/preset-react',
            '@babel/preset-typescript',
          ],
          env: {
            development: {
              plugins: [require.resolve('react-refresh/babel')],
            },
          },
        },
        exclude: path.join(__dirname, 'node_modules'),
      },
      {
        test: /\.css?$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
...
...
...
```

## 10. fork-ts-checker-webpack-plugin

- webpack은 ts체크 후 eslint체크 후 빌드 시작
- ts랑 eslint는 동시에 체크하면 더 효율적이다.
- 이 플러그인이 동시에 진행하게 해준다.

```Typescript
./webpack.config.ts

import ForkTsCheckerWebpackPlugin from 'fork-ts-checker-webpack-plugin';
...
...
...
plugins: [
    new ForkTsCheckerWebpackPlugin({
      async: false,
      // eslint: {
      //   files: "./src/**/*",
      // },
    }),
    new webpack.EnvironmentPlugin({ NODE_ENV: isDevelopment ? 'development' : 'production' }),
  ],
...
...
...
```

## 11. 폴더 구조는 취향대로 세팅한다.

- pages
- components
- hooks
- utils

크게 위와같이 구분할 수 있다. 폴더 이름과 구조는 개개인마다 다르니 그때마다 새로운 시도를 해보자.

## 12. ts와 webpack에서 alias 지정

- `npm i -D tsconfig-paths`
- tsconfig에서 baseUrl와 paths 설정
- webpack에서는 resolve안에 alias 설정

```json
./tsconfig.json

...
...
...

"paths": {
      "@hooks/*": ["hooks/*"],
      "@components/*": ["components/*"],
      "@layouts/*": ["layouts/*"],
      "@pages/*": ["pages/*"],
      "@utils/*": ["utils/*"],
      "@typings/*": ["typings/*"]
    }
...
...
...
```

와 같이 설정할 수 있고, src폴더안에 페이지 구조를 담는다면 src를 앞에 추가해준다.

## 13. 부가적인 기능들 세팅

RouterDom

- `npm i react-router react-router-dom`
- `npm i -D @types/react-router @types/react-router-dom`
- `npm i axios`

- CSS는 취향대로 사용한다.
  - 개인적으로 나는 TailwindCSS, Styled-Component를 주로 사용한다.
  - TailwindCSS를 적용시키는 방법은 추후에 다시 설명하도록 하겠다.
