# OAuth + jwt 

## 이슈
- 전통방식 로긴 프로세스는 세션-쿠키 방식이고 세션은 서버에 저장된다. 하지만 jwt 인증시에 인증서버를 조회하지 않는 방식이고 그렇기 때문에, client 가 인증주체를 보관하고 있어야 한다. client 가 어떤식으로 토큰을 저장하고 있는 것이 안전한가?
- refresh token 을 사용해도될까?


## 가이드

### 세션 활용(서버 메모리)
- 전통방식의 세션 쿠키 방식과 유사하게 브라우져에서 최초 로긴시 인증서버를 통해 토큰을 세션에 저장하고 사용하는 방식
- access token 은 만료시간을 약 30분으로 두고 cookie 저장
- refresh token 은 세션에 저장 후 현재 세션 시간 정도로 설정
- jwt 의 장점은 서버의 io 가 없다는 것이니까, access token 은 쿠키에 두어서 이 장점을 살린다. 대신 탈취될 때를 가정에 시간을 짧게 설정, refresh token 은 user 에 잦은 로긴을 막기 위해 사용하되, 세션을 사용하여 안전하게 저장
- refresh token 은 서버에 있는 것이기 때문에 만료시키는 것도 가능.




### secure 쿠키 와 httpOnly 쿠키 사용
- 쿠키에 저장하되, css 공격을 막을 수 있고 해당 도메인에서만 사용 가능한 secure 쿠키 와 httpOnly 쿠키를 사용하는 방안 
- [설명 link](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies#Secure%EA%B3%BC_HttpOnly_%EC%BF%A0%ED%82%A4)



### 회의 후
- refresh token 만료 시간이 30분
- access token 은 5분 정도 


### 중복 로그인 이슈
- 확인해 봤을 때, 한 유저가 여러 브라우져를 통해 각각의 access token 을 받을 경우 refresh token 도 각각 받으므로, refresh token 이 계속 연장되는 경우가 생기지는 않는다. 그리고, refresh token 을 세션에 저장해서 사용하므로, 세션이 날라갈 때, refresh token 이 같이 제거 되므로, 기존 세션과 유사하게 사용가능하다.


## csrf / cors
- httpOnly 토큰을 사용하면 csrf 방어가된다.


## OAuth approval 에 사용?
- 일단 approval 에 대해서 이해하려면 oAuth 개념에 대해서 다시 살펴볼 필요가 있다.
OAuth 인증 방식은 외부에 클라이언트에게 권한의 범위(scope) 를 지정한 후 유저 로긴 시에, 해당 scope 에 대해서 access 를 허락할 지 묻고 해당 상태를 저장한다. 이 때 scope 를 유저가 재선택 할 수도 있다.
- 위와 같은 상황에서, spring security 에 revoke 를 요청할 수도 있다. 
- spring security 에서 인증시, approval 상태를 체크할 수 있다.
- 하지만 jwt 상황에서는 인증시 DB를 바라보지 않기 때문에, auth_approval 을 시키고, DB를 바라보지 않기 때문에, approval 은 의미를 잃게 될 것 같고, 토큰의 권한을 빼았고 싶으면 다른 방안을 생각해 보아야 겠다.

## jwt 에서 토큰의 권한을 빼았으려면?
1. resource server 에서 인증시, 토큰만 확인하는 것이 아니고 db를 통한 검증 프로세스가 들어가도록 하는 경우
2. 


### jwt 저장관련 참고 링크
- [https://webhack.dynu.net/?idx=20161110.002](https://webhack.dynu.net/?idx=20161110.002)
- [https://forum.vuejs.org/t/vuejs-jwt/14976](https://forum.vuejs.org/t/vuejs-jwt/14976)
- [https://medium.com/@jcbaey/authentication-in-spa-reactjs-and-vuejs-the-right-way-e4a9ac5cd9a3](https://medium.com/@jcbaey/authentication-in-spa-reactjs-and-vuejs-the-right-way-e4a9ac5cd9a3)
- [https://auth0.com/docs/security/store-tokens](https://auth0.com/docs/security/store-tokens)


## csrf 관련 글

- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html


### 아래 글이 결론인 것 같다
https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
