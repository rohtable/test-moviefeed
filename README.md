# 🎞 영화 뉴스피드 프로그램
프로그램 역할 : 영화 리뷰를 보는 프로그램
* 프로젝트 기간 : 2024.06.04 ~ 2024.06.10 (6일)
* 참여자 : 노상윤(👑), 유균한, 홍준빈, 최영주
<br>

## 프로젝트 소개
<details>
<summary> ⁉ 필수 구현 기능 </summary>

<br>
✅ 공통 조건  

* 예외처리는 아래와 같은 형태로 처리하여 `Response` 합니다.

  | Http Status Code | Message |  
  | :--------------: | :-----: |  
  | 400              | 잘못된 요청입니다. |

* Status Code 분류는 [Link](https://hongong.hanbit.co.kr/http-%EC%83%81%ED%83%9C-%EC%BD%94%EB%93%9C-%ED%91%9C-1xx-5xx-%EC%A0%84%EC%B2%B4-%EC%9A%94%EC%95%BD-%EC%A0%95%EB%A6%AC/)를 참고합니다.
* 모든 엔티티에는 `생성일자`와 `수정일자`가 존재합니다.
* 클라이언트는 Postman이고 프론트엔드는 별도 구현하지 않습니다.
<br>

<details>
<summary> ✅ 사용자 인증 기능   </summary>

* 사용자 entity & status
  * 회원
    * bigint : ID
    * varchar : 사용자ID, 비밀번호, 이름, 이메일, 한 줄 소개, 회원상태코드, refresh token
    * timestamp : 상태변경시간, 생성일자, 수정일자
  * 회원상태코드
    * 정상
    * 탈퇴
    * +이메일 인증
<br>

  * 사용자 인증 기능 공통 조건
    * Spring Security와 JWT를 사용하여 설계 및 구현합니다.
    * JWT는 Access Token, Refresh Token을 구현합니다.
    * Access Token 만료 시 : 유효한 Refresh Token을 통해 새로운 Access Token과 Refresh Token을 발급
    * Refresh Token 만료 시 : 재로그인을 통해 새로운 Access Token과 Refresh Token을 발급
    * API를 요청할 때는 Access Token을 사용합니다.
<br>
 
* 회원가입  
  신규 가입자는 `사용자ID`, `비밀번호`를 입력하여 서비스에 가입할 수 있습니다.
  * 사용자ID
    * 중복된 ID, 탈퇴한 ID로는 회원가입 할 수 없습니다.
    * 대소문자 포함 영문 + 숫자만을 허용합니다.
    * 사용자 ID는 최소 10글자 이상, 최대 20글자 이하여야 합니다.
  * 비밀번호
    * `Bcrypt`로 단방향-인코딩합니다.
    * 대소문자 포함 영문 + 숫자 + 특수문자를 최소 1글자씩 포함합니다.
    * 비밀번호는 최소 10글자 이상이어야 합니다.
  * ⚠️ 필수 예외처리
    * 중복된 `사용자 ID`로 가입하는 경우
    * `사용자 ID` 비밀번호 형식이 올바르지 않은 경우
* 회원탈퇴  
  회원탈퇴는 가입된 사용자의 **회원 상태**를 변경하여 탈퇴처리 합니다.  
  탈퇴 처리 시 `비밀번호`를 확인한 후 일치할 때 탈퇴처리 합니다.
  * 조건
    * 탈퇴한 사용자 ID는 재사용할 수 없고, 복구할 수 없습니다.
    * 탈퇴처리된 사용자는 **재탈퇴** 처리가 불가합니다.
  * ⚠️ 필수 예외처리
    * `사용자 ID`와 `비밀번호`가 일치하지 않는 경우
    * 이미 탈퇴한 `사용자 ID`인 경우
* 로그인  
  사용자는 자신의 계정으로 서비스에 **로그인**할 수 있습니다.
  * 조건
    * 로그인 시 클라이언트에게 토큰을 발행합니다.
        
        | 토큰 종류 | 만료기간 |
        | --- | --- |
        | Access Token | 30분 |
        | Refresh Token | 2주 |
    
    * 회원가입된 사용자 ID와 비밀번호가 일치하는 사용자만 로그인할 수 있습니다.
    * 로그인 성공 시, **header**에 토큰을 추가하고 성공 상태코드와 메세지를 반환합니다.
    * 탈퇴했거나 로그아웃을 한 경우, `Refresh Token`이 유효하지 않은 상태가 되어야합니다.
  * ⚠️ 필수 예외처리
    * 유효하지 않은 사용자 정보로 로그인을 시도한 경우
        ex. 회원가입을 하지 않거나 회원 탈퇴한 경우
    * `사용자 ID`와 `비밀번호`가 일치하지 않는 사용자 정보로 로그인을 시도한 경우
* 로그아웃  
  사용자는 로그인 되어 있는 본인의 계정을 **로그아웃** 할 수 있습니다.
  * 조건
    * 로그아웃 시, 발행한 토큰은 **초기화** 합니다.
    * 로그아웃 후 초기화 된 `Refresh Token`은 재사용할 수 없고, 재로그인해야 합니다.
<br>

</details>

<details>
<summary> ✅ 프로필 관리 기능 </summary>

* 프로필 조회
  * **사용자 ID, 이름, 한 줄 소개, 이메일**을 볼 수 있습니다.
  * **ID(사용자 ID X), 비밀번호, 생성일자, 수정일자**와 같은 데이터는 노출하지 않습니다.
* 프로필 수정
  로그인한 사용자는 본인의 사용자 정보를 수정할 수 있습니다.
  * 수정 가능한 사용자 정보 : 이름, 이메일(이메일 인증 기능 구현으로 제외), 한 줄 소개, 비밀번호
  * 비밀번호 수정 조건
    * 비밀번호 수정 시, 본인 확인을 위해 현재 비밀번호를 입력하여 올바른 경우에만 수정할 수 있습니다.
    * 현재 비밀번호와 동일한 비밀번호로는 변경할 수 없습니다.
  * ⚠️ 필수 예외처리
    * 비밀번호 수정 시, 본인 확인을 위해 입력한 현재 비밀번호가 일치하지 않은 경우
    * 비밀번호 형식이 올바르지 않은 경우
    * 현재 비밀번호와 동일한 비밀번호로 수정하는 경우
<br>

</details>

<details>
<summary> ✅ 뉴스피드 게시물 CRUD 기능 </summary>

* 뉴스피드 entity
  * bigint : ID, 작성자 ID
  * longText : 내용
  * timestamp : 생성일자, 수정일자
<br>

* 게시물 작성, 조회, 수정, 삭제  
  게시물 조회는 모든 사용자가 조회할 수 있습니다.
  * 조건
    * 게시물 작성, 수정, 삭제는 **인가(Authorization)**가 필요합니다.
    * 유효한 JWT 토큰을 가진 작성자 본인만 처리할 수 있습니다.
  * ⚠️ 필수 예외처리
    * 작성자가 아닌 다른 사용자가 게시물 작성, 수정, 삭제를 시도하는 경우
* **뉴스피드 조회 기능**  
  모든 사용자가 전체 뉴스피드 데이터를 조회할 수 있습니다.
  * 조건
    * 모든 사용자는 전체 뉴스피드를 조회할 수 있습니다.
    * 기본 정렬은 **생성일자 기준으로 최신순**으로 정렬합니다.
    * 뉴스피드가 없는 경우, 아래와 같이 반환합니다.

</details>
<br>

</details>

<details>
<summary> ⭐ 추가 구현 기능(일부) </summary>

* ❌ 뉴스피드 추가 구현
  * 페이지 네이션
    * 10개씩 페이지네이션하여, 각 페이지 당 뉴스피드 데이터가 10개씩 나오게 합니다.
  * 정렬 기능
    * 생성일자 기준 최신순
    * 좋아요 많은 순
  * 기간별 검색 기능
    * 예) 2024.05.01 ~ 2024.05.27 동안 작성된 뉴스피드 게시물 검색
* ✅ 댓글 CRUD 기능
  * 댓글 entity
    * bigint : ID, 뉴스피드ID, 작성자ID, 좋아요 수
    * varchar : 내용
    * timestamp : 생성일자, 수정일자
  * **댓글 작성, 조회, 수정, 삭제 기능**
    * 사용자는 게시물에 댓글을 작성할 수 있고, 본인의 댓글은 **수정 및 삭제**를 할 수 있습니다.
    * **내용**만 수정이 가능합니다.
    * 댓글 작성, 수정, 삭제는 **인가(Authorization)**가 필요합니다.
    * 유효한 JWT 토큰을 가진 작성자 본인만 처리할 수 있습니다.
      * 예) 본인이 작성한 댓글 외엔 수정 및 삭제 불가
* ✅ 이메일 가입 및 인증 기능
  * 이메일 가입 시, **이메일 인증 기능**을 추가
    * Step 1 : 사용자가 가입한 이메일 주소로 인증번호 발송
    * Step 2 : 발송한 인증번호와 입력란의 인증번호가 일치하는 지 확인
    * Step 3 : 이메일 인증이 완료되지 않은 회원들의 `회원상태코드`를 ‘인증 전’ 으로 설정
* ✅ 좋아요 기능
  * 좋아요 entity
    * bigint : ID, 사용자ID, 콘텐츠ID
    * varchar : 콘텐츠 유형(댓글, 게시물)
    * timestamp : 생성일자, 수정일자
  * 게시물 및 댓글 좋아요/ 좋아요 취소 기능
    * 사용자가 게시물이나 댓글에 좋아요를 남기거나 취소할 수 있습니다.
    * 본인이 작성한 게시물과 댓글에 좋아요를 남길 수 없습니다.
    * 같은 게시물에는 사용자당 한 번만 좋아요가 가능합니다.
* ❌ Swagger 적용
  * 라이브러리 적용 후 Swagger에서 제공되는 기능들은 사용하지 않습니다.
  * localhost:8080/swagger-ui/index.html  주소로 접근시 접속이 가능해야 합니다.
<br>

</details>

<details>
<summary> 🏆 명예의 전당 기능(일부) </summary>

* ✅ 이메일 가입 및 인증 추가 구현
  * 이메일 가입 시 이메일 인증 기능을 포함하는 것이 좋습니다.
    * 인증번호 입력을 180초 안에 하지 않으면 유효하지 않음.
    * 회원 테이블에 인증 메일이 발송된 시간 컬럼을 추가하여 제한시간을 넘는 지 확인.
* ❌소셜 로그인 기능 구현
  * 소셜 로그인에 필요한 **테이블(entity)**을 설계해서 **ERD**에 추가합니다.
  * [네이버 로그인 개발가이드](https://developers.naver.com/docs/login/devguide/devguide.md)를 참고하여 네이버 로그인을 구현해보세요.
  * [카카오 로그인 개발가이드](https://developers.kakao.com/docs/latest/ko/kakaologin/common)를 참고하여 카카오 로그인을 구현해보세요.
* ❌ 프로필에 사진 업로드 기능 구현
  * 프로필 사진을 저장할 때는 반드시 **AWS S3**를 이용합니다.
* ❌ 게시물에 멀티미디어 지원 기능 구현
  * 사진 업로드 기능과 동일하게 **AWS S3**를 이용합니다.
  * 게시물 본문에 사진이나 영상 등의 미디어를 포함하는 기능을 추가합니다.
    * 적절한 용량과 특정 파일 형식만을 업로드할 수 있도록 구현합니다.
      * 여러 장 가능합니다.
      * 한 게시물에 대해 최대 5개 제한
      * `JPG`, `PNG`, `JPEG` → 최대 10MB
      * `MP4`, `AVI`, `GIF` → 최대 200MB
  * 게시물 수정시에 첨부된 미디어를 수정할 수 있습니다.
  * 게시물 삭제시에 첨부된 미디어도 함께 삭제합니다.
  * 댓글에는 추가하지 않습니다.
* ❌ 팔로우 기능 구현
  * 특정 사용자를 팔로우 / 언팔로우를 할 수 있습니다.
  * 팔로우 기능이 구현되었다면, 뉴스피드에 팔로우하는 사용자의 게시물을 볼 수 있습니다.
  * 팔로우를 하고 있는 사람들이 작성한 게시물을 볼 때 정렬 기준은 최신순입니다.
* ❌ HTTP를 HTTPS로 업그레이드 하기
  * HTTPS를 적용하여 보안이 강화된 웹 페이지를 제공해봅시다.
  
</details>

<br>

## 👩‍💻👨‍💻 팀원 구성

<br>


## 🤝 역할 분담

<br>

## ☁ ERD 다이어그램

<br>

## 📑 API 명세서

<br>

## 🏗 프로젝트 구조
```
```
<br>

## 🙄 신경쓴 부분

<br>

## 😫 트러블 슈팅
* 문제 상황  
  * 의도했던 기능 : 내용 작성  
  * 발생한 현상(트러블) : 내용 작성  

* 트러블 원인 추론  
// 내용 작성  

* 해결방법  
// 내용 작성  

<br>

#### 🧡 노상윤

#### 💙 유균한

#### 🖤 홍준빈

#### 💚 최영주
