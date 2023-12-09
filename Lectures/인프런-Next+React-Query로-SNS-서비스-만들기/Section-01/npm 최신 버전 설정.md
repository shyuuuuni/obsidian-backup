
## 문제 상황

`create-next-app` 으로 프로젝트 생성 이후, 테스트로 개발 환경 실행 시 node 버전이 낮아서 실행할 수 없는 상황 발생

```zsh
 npm run dev 

> x-clone@0.1.0 dev
> next dev

You are using Node.js 18.15.0. For Next.js, Node.js version >= v18.17.0 is required.

```

## 해결 방법

`nvm` 이 설치되어 있기에 `nvm` 상에서 `node`의 최신 버전을 받아오고, 해당 버전을 기본 버전으로 사용하도록 변경해주었음.

### 1. node 최신 버전 받아오기

```zsh
 nvm install node
Downloading and installing node v21.4.0...
Downloading https://nodejs.org/dist/v21.4.0/node-v21.4.0-darwin-arm64.tar.xz...
##################################################################################################################################################################### 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v21.4.0 (npm v10.2.4)

```

### 2. 받아온 버전 확인하기

```zsh
 nvm ls          
       v16.18.1
        v17.6.0
        v18.9.1
       v18.15.0
->      v21.4.0
         system
default -> 18 (-> v18.15.0)
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v21.4.0) (default)
stable -> 21.4 (-> v21.4.0) (default)
lts/* -> lts/iron (-> N/A)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.24.1 (-> N/A)
lts/erbium -> v12.22.12 (-> N/A)
lts/fermium -> v14.21.3 (-> N/A)
lts/gallium -> v16.20.2 (-> N/A)
lts/hydrogen -> v18.19.0 (-> N/A)
lts/iron -> v20.10.0 (-> N/A)

```

### 3. 기본 버전 변경하기

2번까지만 진행하면, 다음에 터미널을 새로 생성하면 기존 낮은 버전의 `node`를 사용하므로 기본 버전을 설정해주어야 함.

```zsh
 nvm alias default 21
default -> 21 (-> v21.4.0)

```