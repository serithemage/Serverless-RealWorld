# Chapter 1: Angular 클라이언트 애플리케이션


Serverless-RealWorld 프로젝트에 오신 것을 환영합니다! 이 튜토리얼 시리즈를 통해 실제 운영 가능한 풀스택 애플리케이션이 어떻게 구성되고 작동하는지 함께 살펴보겠습니다. 첫 번째 장에서는 사용자와 가장 먼저 만나는 부분인 Angular 클라이언트 애플리케이션에 대해 알아볼 것입니다.

여러분이 온라인 쇼핑몰에서 마음에 드는 옷을 고르고 장바구니에 담는다고 상상해 보세요. 이때 여러분이 직접 보고 클릭하는 화면이 바로 클라이언트 애플리케이션입니다. 이 애플리케이션이 없다면, 아무리 좋은 상품(데이터나 기능)이 창고(서버)에 가득해도 고객에게 보여줄 방법이 없겠죠? Angular 클라이언트 애플리케이션은 바로 이 문제를 해결합니다. 즉, 사용자에게 정보를 보여주고, 사용자의 행동(클릭, 입력 등)에 반응하여 서버와 통신하는 역할을 합니다.

우리의 Serverless-RealWorld 프로젝트에서는 블로그 게시글을 보고, 작성하고, 댓글을 다는 등의 기능을 제공합니다. 이 장에서는 **사용자가 웹사이트에 접속했을 때 게시글 목록을 보고, 특정 게시글을 클릭하여 상세 내용을 보는 과정**을 중심으로 Angular 클라이언트 애플리케이션이 어떻게 작동하는지 살펴보겠습니다.

## Angular 클라이언트 애플리케이션이란 무엇일까요?

Angular 클라이언트 애플리케이션은 **사용자가 직접 상호작용하는 웹 애플리케이션**입니다. 잘 꾸며진 상점의 **쇼룸**과 같습니다. 고객(사용자)이 진열된 상품(기능)을 보고, 선택하며, 구매(요청)하는 공간이죠.

이 쇼룸(Angular 앱)은 다음과 같은 주요 기술들로 구성됩니다:

1.  **Angular**: 사용자 인터페이스(UI)를 만드는 데 사용되는 강력한 프레임워크입니다. 쇼룸의 진열대, 상품 배치, 안내판 등을 만드는 도구라고 생각할 수 있습니다.
2.  **Apollo Client**: 쇼룸 직원이 창고(서버)에 상품 정보를 문의하거나 주문을 전달하는 통신 수단입니다. 우리 앱에서는 [컨ジット 게이트웨이 (API 게이트웨이)](03_컨ジット_게이트웨이__api_게이트웨이__.md)를 통해 [GraphQL API 스키마 및 리졸버](04_graphql_api_스키마_및_리졸버_.md)와 통신하는 데 사용됩니다.
3.  **`AppStateService`**: 쇼룸 전체의 중요한 정보(예: 현재 로그인한 고객 정보, 쇼룸 특별 할인 정보 등)를 관리하는 중앙 안내 데스크와 같습니다. 애플리케이션 전반의 상태를 관리합니다.

이러한 요소들이 모여 사용자가 편리하게 서비스를 이용할 수 있는 웹 환경을 만듭니다.

## 어떻게 작동할까요? (게시글 목록 보기 예시)

사용자가 우리 웹사이트에 접속해서 게시글 목록을 보는 과정을 간단히 따라가 보겠습니다.

1.  **쇼룸 입장 (애플리케이션 시작)**: 사용자가 웹 브라우저를 통해 우리 웹사이트 주소로 접속합니다.
2.  **쇼룸 준비 (Angular 앱 초기화)**: Angular 애플리케이션이 로드되고, 화면을 구성할 준비를 합니다.
3.  **상품 진열 요청 (데이터 요청)**: "최신 게시글 목록을 보여주세요!"라는 요청을 Apollo Client를 통해 서버([컨ジット 게이트웨이 (API 게이트웨이)](03_컨ジット_게이트웨이__api_게이트웨이__.md))로 보냅니다.
4.  **상품 도착 (데이터 응답)**: 서버는 요청받은 게시글 목록 데이터를 보내줍니다.
5.  **상품 진열 (화면 표시)**: Angular 앱은 받은 데이터를 사용해 사용자 화면에 게시글 목록을 예쁘게 보여줍니다.

이제 각 단계에서 중요한 코드 조각들을 살펴보며 좀 더 자세히 이해해 봅시다.

### 1. 애플리케이션의 시작: `main.ts`

모든 Angular 애플리케이션은 시작점이 있습니다. 우리 프로젝트에서는 `main.ts` 파일이 그 역할을 합니다.

```typescript
// 파일: apps/client/src/main.ts
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module'; // 우리 앱의 핵심 설계도

platformBrowserDynamic()
  .bootstrapModule(AppModule) // AppModule을 기준으로 앱을 실행!
  .catch((err) => console.error(err));
```

이 코드는 마치 자동차 키를 돌려 시동을 거는 것과 같습니다. `platformBrowserDynamic().bootstrapModule(AppModule)`은 `AppModule`이라는 우리 애플리케이션의 핵심 설계도를 바탕으로 웹 브라우저에서 애플리케이션을 실행하라는 명령입니다.

### 2. 애플리케이션의 설계도: `app.module.ts`

`AppModule`은 우리 애플리케이션의 전체적인 구조와 필요한 부품들을 정의하는 파일입니다. 쇼룸의 전체 레이아웃과 어떤 코너(기능 모듈)들이 있는지 알려주는 안내도와 같죠.

```typescript
// 파일: apps/client/src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { ApolloModule, APOLLO_OPTIONS } from 'apollo-angular'; // Apollo Client 설정
import { HttpLink } from 'apollo-angular/http';
import { HttpClientModule } from '@angular/common/http'; // HTTP 통신 모듈

import { AppComponent } from './app.component'; // 앱의 기본 틀 컴포넌트
import { createApollo } from './shared/apollo/createApollo'; // Apollo Client 설정 함수
// ... (HeaderComponent, FooterComponent 등 다른 컴포넌트 및 모듈 import) ...

@NgModule({
  declarations: [
    AppComponent, // 이 모듈이 관리하는 컴포넌트
    /* HeaderComponent, FooterComponent 등 */
  ],
  imports: [ // 다른 필요한 모듈들을 가져옴
    BrowserModule,        // 브라우저에서 실행하기 위한 기본 모듈
    ApolloModule,         // GraphQL 통신을 위한 Apollo 모듈
    HttpClientModule,     // HTTP 요청을 보내기 위한 모듈
    // ... (AuthModule, ArticleModule 등 기능별 모듈) ...
  ],
  providers: [ // 서비스(데이터 처리, 상태 관리 등) 설정
    {
      provide: APOLLO_OPTIONS,
      useFactory: createApollo, // Apollo Client 인스턴스 생성 방법
      deps: [HttpLink],
    },
  ],
  bootstrap: [AppComponent], // 앱 시작 시 가장 먼저 보여줄 컴포넌트
})
export class AppModule {}
```

-   `declarations`: 이 모듈에 포함된 컴포넌트(화면 조각)들을 선언합니다. `AppComponent`는 우리 앱의 가장 바깥 틀을 담당합니다.
-   `imports`: 다른 모듈들을 가져와서 사용합니다. 예를 들어, `BrowserModule`은 웹 브라우저 환경에서 앱이 잘 돌아가도록 돕고, `ApolloModule`과 `HttpClientModule`은 서버와 통신하는 데 필요합니다.
-   `providers`: 서비스들을 등록합니다. 서비스는 데이터 처리, 서버 통신, 상태 관리 등 보이지 않는 곳에서 중요한 역할들을 합니다. `APOLLO_OPTIONS`는 Apollo Client를 어떻게 설정할지 알려줍니다.
-   `bootstrap`: 애플리케이션이 처음 시작될 때 어떤 컴포넌트를 화면에 보여줄지 지정합니다. 여기서는 `AppComponent`입니다.

### 3. 화면의 구성 요소: 컴포넌트 (예: `app.component.ts`)

컴포넌트는 화면의 특정 부분을 만들고 관리하는 조립 블록입니다. `AppComponent`는 우리 앱 전체의 뼈대가 되는 가장 기본적인 컴포넌트입니다.

```typescript
// 파일: apps/client/src/app/app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'conduit-root', // HTML에서 이 컴포넌트를 <conduit-root></conduit-root> 태그로 사용
  templateUrl: './app.component.html', // 이 컴포넌트의 HTML 구조 파일
  styleUrls: ['./app.component.scss'], // 이 컴포넌트의 스타일(디자인) 파일
})
export class AppComponent { } // 특별한 로직은 현재 없음
```

`@Component` 데코레이터(장식자)는 이 클래스가 컴포넌트임을 Angular에게 알립니다.
-   `selector`: HTML에서 이 컴포넌트를 부를 때 사용할 이름입니다 (예: `<conduit-root>`).
-   `templateUrl`: 이 컴포넌트가 화면에 어떻게 보일지 정의한 HTML 파일의 경로입니다.
-   `styleUrls`: 이 컴포넌트의 모양을 꾸미는 CSS(SCSS) 파일의 경로입니다.

우리 예시인 "게시글 목록 보기"는 주로 `HomeComponent` (홈 화면 컴포넌트)에서 이루어집니다. 이 컴포넌트가 `ArticleService`를 통해 데이터를 가져와 화면에 표시하게 됩니다.

```typescript
// 파일: apps/client/src/app/modules/home/home.component.ts (일부 개념적 코드)
import { Component, OnInit } from '@angular/core';
import { ArticleService } from '../../services/article.service'; // 게시글 서비스
// ... IArticle 등 필요한 모델 import ...

@Component({
  selector: 'conduit-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.scss'],
})
export class HomeComponent implements OnInit {
  articles: any[] = []; // 게시글 목록을 저장할 변수

  constructor(private articleService: ArticleService) {} // ArticleService 주입

  ngOnInit() { // 컴포넌트가 초기화될 때 실행
    // 실제 사용 시 사용자 정보 등을 AppStateService에서 가져와 활용
    // const currentUser = ''; // 예시: 현재 사용자 이름
    // const token = '';     // 예시: 사용자 토큰

    // this.articleService.getAll(currentUser, token).subscribe(response => {
    //   // this.articles = response.data.articles; // 서버에서 받은 게시글 목록
    //   console.log('서버로부터 게시글 목록을 받았습니다!', response);
    // });
    //  위의 주석 처리된 코드가 실제 게시글을 가져오는 부분입니다.
    //  지금은 개념 설명을 위해 단순화했습니다.
  }
}
```
`HomeComponent`는 `ArticleService`를 사용하여 게시글 목록을 가져옵니다. `ngOnInit`은 컴포넌트가 만들어지고 화면에 표시되기 직전에 호출되는 특별한 함수로, 여기서 데이터 로딩 같은 초기화 작업을 수행합니다. `.subscribe(...)` 부분은 서버로부터 응답이 왔을 때 실행될 코드를 정의합니다.

### 4. 서버와의 대화 창구: Apollo Client 설정 (`createApollo.ts`)

Apollo Client는 우리 Angular 앱이 서버의 [GraphQL API 스키마 및 리졸버](04_graphql_api_스키마_및_리졸버_.md)와 쉽게 대화할 수 있도록 도와주는 라이브러리입니다. `createApollo.ts` 파일에서 이 통신 설정을 합니다.

```typescript
// 파일: apps/client/src/app/shared/apollo/createApollo.ts (주요 부분)
import { HttpLink } from 'apollo-angular/http';
import { setContext } from '@apollo/client/link/context';
import { InMemoryCache, ApolloLink } from '@apollo/client/core';
import { URLs } from '../constants/common'; // 서버 URL 등 상수 관리
import { AppStateService } from '../../services/common/appStateService'; // 앱 상태 서비스

export const createApollo = (httpLink: HttpLink) => {
  const auth = setContext((operation, context) => { // 모든 요청에 헤더 추가
    const token = AppStateService.getUserTokenStatic(); // AppStateService에서 사용자 토큰 가져오기

    return {
      headers: {
        Authorization: token ? `Bearer ${token}` : '', // 토큰이 있으면 Authorization 헤더에 추가
      },
    };
  });

  // API 서버 주소 설정 및 인증 헤더 링크 결합
  const link = ApolloLink.from([auth, httpLink.create({ uri: URLs.serverUrl })]);
  const cache = new InMemoryCache({ /* 캐시 정책 설정 ... */ }); // 데이터 임시 저장소

  return { link, cache /*, defaultOptions */ };
};
```

-   `URLs.serverUrl`: 우리 앱이 통신할 서버 API의 주소([컨ジット 게이트웨이 (API 게이트웨이)](03_컨ジット_게이트웨이__api_게이트웨이__.md)의 주소)입니다.
-   `setContext`: 모든 GraphQL 요청을 보내기 전에 실행되는 함수입니다. 여기서는 `AppStateService`를 통해 현재 사용자의 인증 토큰을 가져와, 요청 헤더의 `Authorization` 항목에 추가합니다. 이렇게 하면 서버는 이 요청이 인증된 사용자의 요청인지 알 수 있습니다. 마치 VIP 클럽에 들어갈 때 회원 카드를 보여주는 것과 같아요.
-   `InMemoryCache`: 서버에서 받아온 데이터를 잠시 저장해두는 공간입니다. 같은 데이터를 또 요청할 필요 없이 캐시에서 바로 가져다 쓸 수 있어 앱 반응 속도를 높일 수 있습니다.

### 5. 데이터 가져오기 전문가: 서비스 (예: `article.service.ts`)

서비스는 컴포넌트가 직접 처리하기에는 복잡하거나 여러 곳에서 반복적으로 사용되는 로직들을 담당합니다. `ArticleService`는 게시글과 관련된 모든 서버 통신 및 데이터 처리를 맡습니다.

```typescript
// 파일: apps/client/src/app/services/article.service.ts (일부)
import { Injectable } from "@angular/core";
import { Apollo } from "apollo-angular"; // Apollo Client 주입
import { GET_ARTICLES } from "../shared/constants/queries/article-queries"; // GraphQL 쿼리문

@Injectable() // 다른 곳에서 이 서비스를 주입받아 사용할 수 있게 함
export class ArticleService {
  constructor(private readonly apollo: Apollo) { } // Apollo Client 인스턴스 주입

  // 모든 게시글 가져오기
  getAll(currentUser: string, token: string) {
    return this.apollo.query({ // Apollo Client를 사용해 GraphQL 쿼리 실행
      query: GET_ARTICLES,       // 미리 정의된 "모든 게시글 조회" 쿼리
      variables: {               // 쿼리에 필요한 변수들
        currentUser,
        token
      },
      fetchPolicy: 'network-only' // 항상 네트워크에서 최신 데이터 가져오기 (캐시 사용 안 함)
    });
  }
  // ... (특정 게시글 가져오기, 게시글 작성, 수정, 삭제 등의 다른 함수들) ...
}
```

`ArticleService`는 `Apollo` 서비스를 주입받아 사용합니다. `getAll` 함수는 `this.apollo.query({...})`를 호출하여 서버에 게시글 목록을 요청합니다.
-   `query: GET_ARTICLES`: 어떤 데이터를 원하는지 정의한 GraphQL 쿼리문입니다. (이 쿼리문의 자세한 내용은 [GraphQL API 스키마 및 리졸버](04_graphql_api_스키마_및_리졸버_.md) 장에서 다룹니다.)
-   `variables`: 쿼리에 필요한 추가 정보(예: 현재 사용자 정보)를 전달합니다.
-   `fetchPolicy: 'network-only'`: 항상 서버로부터 새로운 데이터를 가져오도록 하는 옵션입니다. 상황에 따라 캐시된 데이터를 우선 사용하도록 변경할 수도 있습니다.

반환값은 `Observable`이라는 특별한 객체인데, `HomeComponent`에서 이것을 `.subscribe()` 하여 데이터가 도착했을 때 처리할 수 있습니다.

### 6. 애플리케이션 상태 관리: `AppStateService`

`AppStateService`는 애플리케이션 전반에서 공유되어야 하는 중요한 상태(예: 현재 로그인한 사용자 정보, 사용자 토큰 등)를 관리합니다. 쇼룸의 중앙 안내 데스크나 상황실과 비슷합니다.

```typescript
// 파일: apps/client/src/app/services/common/appStateService.ts (일부)
import { Injectable } from "@angular/core";
import { BehaviorSubject } from "rxjs"; // 데이터의 변화를 감지하고 알림
import { AppConstants } from "../../shared/constants/app-constants";
import { IUser } from "../../shared/model/IUser";
import { SLSService } from "./secureLocalStorageService"; // 안전한 로컬 저장소 서비스

@Injectable()
export class AppStateService {
  // 사용자 토큰을 외부에 공개 (읽기 전용)
  public userTokenData$ = new BehaviorSubject<string>(this.getUserToken()).asObservable();
  // 현재 사용자 정보를 외부에 공개 (읽기 전용)
  public currentUserData$ = new BehaviorSubject<IUser | undefined>(undefined).asObservable();

  // (정적 메소드) 저장된 사용자 토큰 가져오기
  static getUserTokenStatic(): string {
    return SLSService.getValueByKey(AppConstants.AUTH_TOKEN_KEY);
  }

  // 사용자 토큰 설정 및 변경 알림
  public setUserToken(token: string): void {
    SLSService.setKeyValue(AppConstants.AUTH_TOKEN_KEY, token); // 로컬 저장소에 토큰 저장
    (this.userTokenDataSource as BehaviorSubject<string>).next(this.getUserToken()); // 변경사항 알림
  }

  // ... (현재 사용자 정보 설정/가져오기, 사용자 정보 초기화 등 다른 함수들) ...

  // (private으로 변경되거나 내부에서만 사용될 수 있음)
  private getUserToken(): string { // 실제 userTokenDataSource를 참조하도록 변경해야 함
    return SLSService.getValueByKey(AppConstants.AUTH_TOKEN_KEY);
  }
}
```
`AppStateService`는 `BehaviorSubject`를 사용하여 상태 값의 변화를 다른 컴포넌트나 서비스에 알릴 수 있습니다. 예를 들어, 사용자가 로그인하면 `setUserToken` 함수가 호출되어 토큰 정보를 저장하고, 이 변경 사항을 구독하고 있는 모든 부분에 알립니다. `createApollo.ts`에서 토큰을 가져올 때 이 서비스의 `getUserTokenStatic` 함수를 사용했습니다.

## 내부 동작 흐름 요약

사용자가 게시글 목록을 보는 과정을 간단한 순서도로 표현하면 다음과 같습니다.

```mermaid
sequenceDiagram
    participant 사용자
    participant Angular 클라이언트 (예: HomeComponent)
    participant ArticleService
    participant ApolloClient (createApollo 설정 포함)
    participant API 게이트웨이

    사용자->>Angular 클라이언트: 웹사이트 방문 (홈페이지 요청)
    Angular 클라이언트->>ArticleService: 게시글 목록 보여줘! (getAll 호출)
    ArticleService->>ApolloClient: 이 GraphQL 쿼리로 데이터 요청해줘 (GET_ARTICLES)
    ApolloClient->>API 게이트웨이: HTTP 요청 (쿼리 + 인증 헤더 from AppStateService)
    API 게이트웨이-->>ApolloClient: HTTP 응답 (게시글 데이터)
    ApolloClient-->>ArticleService: 데이터 도착!
    ArticleService-->>Angular 클라이언트: 여기 게시글 데이터야
    Angular 클라이언트->>사용자: 게시글 목록을 화면에 표시
```

이처럼 Angular 클라이언트 애플리케이션은 여러 구성 요소들이 각자의 역할을 수행하며 유기적으로 협력하여 사용자에게 웹 서비스를 제공합니다.

## 정리하며

이번 장에서는 Serverless-RealWorld 애플리케이션의 "쇼룸"에 해당하는 Angular 클라이언트 애플리케이션에 대해 알아보았습니다. Angular 앱이 어떻게 시작되고, 어떤 주요 구성 요소(모듈, 컴포넌트, 서비스)들로 이루어져 있는지, 그리고 서버와 어떻게 통신하여 사용자에게 정보를 보여주는지 간략하게 살펴보았습니다. 특히 `main.ts`, `AppModule`, 컴포넌트, 서비스, 그리고 Apollo Client 설정의 기본적인 역할을 이해하는 데 중점을 두었습니다.

Angular 클라이언트 애플리케이션은 사용자와 직접 소통하는 중요한 창구입니다. 하지만 이 쇼룸이 멋지게 작동하려면, 쇼룸 뒤편에서 상품을 준비하고 관리하는 복잡한 시스템이 필요합니다.

다음 장에서는 이 쇼룸에 상품을 공급하고, 다양한 백엔드 기능을 제공하는 핵심 구조인 [마이크로서비스 아키텍처](02_마이크로서비스_아키텍처_.md)에 대해 자세히 살펴보겠습니다. 이를 통해 우리 애플리케이션이 어떻게 더 크고 복잡한 요청들을 효율적으로 처리하는지 이해할 수 있을 것입니다.

---

Generated by [AI Codebase Knowledge Builder](https://github.com/The-Pocket/Tutorial-Codebase-Knowledge)