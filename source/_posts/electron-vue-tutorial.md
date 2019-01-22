---
title: Electron + Vue.js 애플리케이션 만들기
date: 2019-01-22 19:46:04
categories:
- electron 튜토리얼
tags:
- node.js
- electron
- Vue.js
---

ELectron을 사용한 애플리케이션을 만들기 위한 Renderer, 사용자에게 보여지는 GUI를 구현하기 위해 Vue.js  를 이용하여 구현할 수 있다. Electron에 대한 설명은 [Electron 애플리케이션 만들기](https://hsoh1990.github.io/2019/01/18/electron-tutorial/)를 참고하고 [Vue.js](https://kr.vuejs.org/v2/guide/index.html)를 참고 하면 된다.

<!--more-->  

**Table of Contents**

- [Quick start](#Quick-start)
- [Project Structure](Project-Structure)
- [NPM Scripts](#NPM-Scripts)
- [Reference](#Reference)
- [Contributors](#Contributors)



vue로 구축 된 electron 애플리케이션을 만들기 위한 보일러 플레이트는 [electron-vue](https://github.com/SimulatedGREG/electron-vue/tree/master/docs/ko)를 사용한다. electron-vue은 스캐폴딩을 위한 `vue-cli`, `vue-loader`이 있는 `webpack`, `electron-packager`, `electron-builder`, `vue-router`, `vuex` 등과 같이 가장 많이 사용되는 플러그인을 사용한다.



## Quick start

```bash
# vue-cli와 스캐폴딩 보일러 플레이트 설치
$ npm install -g vue-cli
$ vue init simulatedgreg/electron-vue my-project

# 의존성 설치 및 개발자 모드로 실행
$ cd my-project
$ npm install
$ npm run dev
```



## Project Structure

```
my-project
├─ .electron-vue
│  └─ <build/development>.js files
├─ build
│  └─ icons/
├─ dist
│  ├─ electron/
│  └─ web/
├─ node_modules/
├─ src
│  ├─ main
│  │  ├─ index.dev.js
│  │  └─ index.js
│  ├─ renderer
│  │  ├─ components/
│  │  ├─ router/
│  │  ├─ store/
│  │  ├─ App.vue
│  │  └─ main.js
│  └─ index.ejs
├─ static/
├─ test
│  ├─ e2e
│  │  ├─ specs/
│  │  ├─ index.js
│  │  └─ utils.js
│  ├─ unit
│  │  ├─ specs/
│  │  ├─ index.js
│  │  └─ karma.config.js
│  └─ .eslintrc
├─ .babelrc
├─ .eslintignore
├─ .eslintrc.js
├─ .gitignore
├─ package.json
└─ README.md
```



## NPM Scripts

- `npm run build` -> 프로덕션과 패키지 용 앱을 빌드([**Building Your App**](https://simulatedgreg.gitbooks.io/electron-vue/content/ko/building_your_app.html))
- `npm run dev` -> 현재 프로젝트를 개발용으로 실행
- `npm run lint` -> 모든 `src/`와 `test/`의 JS & Vue component 파일을 Lint 
- `npm run lint:fix` -> 모든 `src/`와 `test/`의 JS & Vue component 파일을 Lint하고 문재 해결을 시도
- `npm run pack` -> `npm run pack:main` & `npm run pack:renderer` 둘 다 실행
- `npm run pack:main`  -> `main` 프로세스 소스 코드를 번들
- `npm run pack:renderer` ->  `renderer` 프로세스 소스 코드를 번들
- `npm run unit`  -> Karma와 Jasmine로 단위 테스트를 실행 ([**Unit Testing**](https://simulatedgreg.gitbooks.io/electron-vue/content/ko/unittesting.html))
- `npm run e2e` -> Spectron + Mocha로 end-to-end 테스트를 실행([**End-to-end Testing**](https://simulatedgreg.gitbooks.io/electron-vue/content/ko/end-to-end_testing.html))
- `npm test` -> `npm run unit` & `npm run e2e` 둘 다 실행([Testing**](https://simulatedgreg.gitbooks.io/electron-vue/content/ko/testing.html))





## Reference

- https://electronjs.org/docs
- https://kr.vuejs.org/v2/guide/index.html
- https://github.com/SimulatedGREG/electron-vue/tree/master/docs/ko
- https://simulatedgreg.gitbooks.io/electron-vue/content/ko/getting_started.html

## Contributors

- 오형석[(ohs4123@gmail.com)](