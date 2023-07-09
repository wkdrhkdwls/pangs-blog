---
title: "fullpage.js 사용하기"
description: "fullpage.js 사용하기 (제이쿼리 풀페이지)"
date: 2023-07-10
update: 2023-07-10
tags:
  - ReactJS
  - TypeScript
  - Blog
  - fullpage
series: "Project"
---

평소에 어떻게 메인페이지를 구성할까 생각하다 fullpage.js를 적용해보기로 했다.
fullpage.js란 [배달의민족](https://www.baemin.com/?gclid=Cj0KCQjwtamlBhD3ARIsAARoaEy0xD47wHqxZVeHroSIN46fCLVhOgE5aFG9YnxBTtBxCDOj0GCvZskaAuYfEALw_wcB)와 같이 이미지 또는 동영상으로 스크롤하면 페이지 단위로 움직이는 제이쿼리 플러그인이다.
[Fullpage](https://alvarotrigo.com/fullPage/ko/)

## 1. 설치하기

- 우선 fullpage.js는 v2.xx까지는 무료이지만 v3부터는 유료 라이센스키가 필요하기 때문에 우리는 v2.96으로 다운그레이드하여 사용할 것이다.

> `npm i fullpage.js@2.9.6`
>
> `npm i jquery --save`
>
> `npm i @types/fullpage.js@2.9.3`

로 fullpage와 jquery를 설치해준다.

## 2. 페이지 적용하기

### 2-1. 기촐 레이아웃 생성

```Typescript

import $ from "jquery"; // jquery를 선언해준다.
import "fullpage.js";
import "fullpage.js/dist/jquery.fullpage.min.css";

const Home = () => {
  $(() => {
    $("#fullpage").fullpage({
      menu: "#menu",
      anchors: ["1", "2", "3", "4"],
      sectionsColor: ["black", "blue", "red", "orange"],
    });
  });

  return (
    <Layout>
      <div id="fullpage">
        <div className="section" id="#section1">
          1번
        </div>
        <div className="section" id="#section2">
          2번
        </div>
        <div className="section" id="#section3">
          3번
        </div>
        <div className="section" id="#section4">
          4번
        </div>
      </div>
    </Layout>
  );
};
```

> import "fullpage.js";
>
> import "fullpage.js/dist/jquery.fullpage.min.css";

를 불러온후 jQuery를 사용하여 위와같이 설정해준다. 그 후 id로 섹션별로 구분해준다.

### 2-2. 동영상 추가하기

```Typescript
@types/global/index.d.ts

declare module "*.png";
declare module "*.jpg";

declare module "*.mp4";
```

타입스크립트에서는 모듈 확장을 사용하여 기존 모듈 또는 라이브러리의 기능을 확장할 수 있다. 우리는 동영상 파일을 사용해야하고, 컴파일러가 이러한 파일 형식을 만났을 때 오류를 발생시키는 것을 방지하기 위해 선언해준다.
<br/>
그 후에 비디오를 보여줄 컴포넌트를 생성해준다.

```Typescript
const fullpageSection = () => {
  const videos = [sea, grape, food];

  return (
    <>
      {videos.map((video, index) => (
        <div className="section" id={`#section${index}`}>
          <video
            className="h-full w-full object-cover"
            data-autoplay
            muted
            loop
          >
            <source src={video} type="video/mp4" />
          </video>
        </div>
      ))}
    </>
  );
};
```

그 후 다음과 같이 FullpageSection 컴포넌트를 불러와 수정해준다.

```Typescript

import Layout from "@components/Layout/layout";
import $ from "jquery";
import "fullpage.js";
import "fullpage.js/dist/jquery.fullpage.min.css";

import FullpageSection from "@components/Fullpage/fullpageSection";

const Home = () => {
  $(() => {
    $("#fullpage").fullpage({
      menu: "#menu",
      anchors: ["1", "2", "3"],
      sectionsColor: ["black", "black", "black"], //이를 삭제하면 "/"에서만 섹션 이동이 가능하다.
    });
  });

  return (
    <Layout>
      <div id="fullpage">
        <FullpageSection />
      </div>
    </Layout>
  );
};
```

+++

## 3. 문제점

다른 페이지는 문제가 없지만 fullpage.js를 적용한 페이지만 Header와 Footer를 묶어논 Layout에서 Footer가 보이지 않는 문제가 발생했다.

### 3-1. 해결책

1. 전역으로 설정한 Layout을 각각의 페이지마다 설정해준다.
2. fullpage의 section을 추가해 footer를 추가로 section으로 넣어준다.

```Typescript

const Home = () => {
  $(() => {
    $("#fullpage").fullpage({
      menu: "#menu",
      anchors: ["1", "2", "3","footer"],
      sectionsColor: ["black", "black", "black","white],
    });
  });

    return (
    <>
      <Header />
      <div id="fullpage">
        {videos.map((video, index) => (
          <div className="section" id={`#section${index}`}>
            <video
              className="h-full w-full object-cover"
              data-autoplay
              muted
              loop
            >
              <source data-src={video} type="video/mp4" />
            </video>
          </div>
        ))}
        <div className="section fp-auto-height" id="#section4">
          <Footer />
        </div>
      </div>
    </>
  );
};
```

FullpageSection를 제거하고 Footer를 Section에 추가해주면 footer도 잘나오는 것을 확인 할 수 있다.
