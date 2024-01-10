# React + TypeScript + Vite + Yarn berry Template

```
"packageManager": "yarn@4.0.2"
```

## 시작하기

1. `yarn` or `yarn install`
2. `yarn dev`

- 에디터가 vscode가 아니라면 `.vscode` 삭제 후 별도의 에디터 셋업이 필요합니다.
  > https://yarnpkg.com/getting-started/editor-sdks#editor-setup

## 트러블 슈팅

1. 100% zero-install은 불가능했다.
   - vite는 개발 서버 최초 실행할 때 esbuild를 사용해서 dependencies를 사전 번들링한다. 개발자가 변경하지 않는 패키지들을 미리 ESM으로 번들링 해놓음으로써 성능을 개선하기 위함이다.
   - yarn pnp는 read-only dependencies만 `.yarn/cache`에 zip파일로 관리하고 아닌 것들은 `.yarn/unplugged`에서 따로 관리한다.
   - read-only하지 않은 dependencies란 예를 들어 install 과정에서 현재 로컬 환경에 대한 바이너리 정보를 수집한 후 실행하는 의존성 등이 있다. install 없이 모든 환경에서 동일한 ready-only zip으로만 관리하면 실행 실패할 것이다.
   - vite가 사용하는 esbuild는 `preferUnplugged: true`이기 때문에 `.yarn/unplugged` 하위에 설치되고 해당 폴더는 [공식문서](https://yarnpkg.com/getting-started/qa#which-files-should-be-gitignored)에 따르면 git에 올리지 않는게 권장된다.
     > https://github.com/evanw/esbuild/releases?q=pnp&expanded=true
   - 그래서 형상관리에 포함되지 않는 dependency인 vite의 esbuild로 인해 최초 1번은 yarn을 통해 패키지를 설치해야 한다.(운영 빌드에 사용되는 rollup도 해당된다)
2. `yarn install` 이후 생긴 `node_modules`
   - 개발 서버를 돌려보면 `node_modules/.vite/deps`가 생성된다.
   - 하위 파일들을 살펴보면 dependencies들이 ESM으로 트랜스파일링 되어있다. vite가 esbuild로 사전 번들링한 결과물이다.
   - [공식문서](https://v2.vitejs.dev/config/#publicdir)에 캐시 파일들의 default 경로가 `node_modules/.vite`라고 나와있다. `node_modules`의 단점을 보완하기 위해 pnp를 도입했는데 `node_modules`와 공존하는건 아이러니하다.(물론 nodeLinker: node_modules인 경우나 호환 문제로 인해 얼마든지 공존할 수 있다.)
   - 그래서 `vite.config.ts`에서 `cacheDir: "./.vite"` 경로를 변경해줬다. (`node_modules` -> `.vite`)
