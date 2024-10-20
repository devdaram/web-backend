<h1>✨ 맡은 업무 설명</h1>
<h2> 1. SKT 프로젝트 </h2>
      
      [장비 토폴로지 등록]
      - Web Backend : 장비 토폴로지(Ring)등록 작업 RestAPI 개발
      (링 중복조회, 링 등록 일괄처리, 링 삭제 등)
    
      
<h3> 💻 개발 환경 </h3>

      1. java 1.8
      2. Springboot 2.3.12

<h4>아래는 실제 개발했던 RestAPI를 연동한 화면 일부 발췌</h4>

<img width="1592" alt="image" src="https://github.com/user-attachments/assets/2ba5c196-4a84-4665-8585-84b01feeb90f">


--------------
<h2> 2. 삼성전자 프로젝트</h2>

      [전체적인 NMS Web 개발]
      - Web Backend : 사용자관리, 로그인, 메뉴관리, 권한관리, Tenant관리, IP 권한관리, 세션관리, 세션 히스토리관리, 3rd party 관리, 장비 Group 관리 등 RestAPI 개발하였음.
      - DB : 맡은 기능 DB ERD 설계
      
      [다국어 처리]
      - Exception 처리 및 log 등 다국어처리가 될 수 있도록 locale을 받아 해당 언어로 응답 될 수있도록 개발
      
      [권한 및 세션관리]
      - 사용자의 로그인기능(password 일치여부, lock처리, IP대역체크 등)과 사용자 권한(Tenant, 운용자 권한 등)에 따른 로직처리, 만료된 세션을 체크 경험이 있음.
      
      
<h3> 💻 개발 환경 </h3>

      1. java 
      2. Springboot


<h4>1. RestAPI 개발 - DB ERD 일부발췌 </h4>


<h4>1. 다국어처리</h4>

```java
//일부발췌
//ENUM 클래스에서 로깅메세지 매칭
PROCESS_SUCCESS(HttpStatus.OK , "code.PROCESS_SUCCESS"),
USER_LOGIN_SUCCESS(HttpStatus.OK, "code.USER_LOGIN_SUCCESS"),
USER_LOGOUT_SUCCESS(HttpStatus.OK, "code.USER_LOGOUT_SUCCESS"),

//비즈니스로직에서 return 시
return new ApiFormat(ApiResultCode.PROCESS_SUCCESS.apiResult(), info);

//사용자가 로그인한 locale을 받아오는 부분
@Bean
	public LocaleResolver localeResolver() {
		CookieLocaleResolver localeResolver = new CookieLocaleResolver();
		localeResolver.setCookieName("locale");
		localeResolver.setDefaultLocale(new Locale("en"));
		localeResolver.setCookieHttpOnly(true);

		return localeResolver;
	}
```

```xml
//일부발췌

//message_ko.xml 와 같이 국가별로 xml 파일을 따로 생성
menu:
  INVALID: 유효하지 않은 형식 입니다.
  MENU_SEARCH_SUCCESS: 메뉴 조회 성공
  PROCESS_SUCCESS: 처리 성공
```

<h4>1. 권한/ 세션관리 - 로직 구조 일부발췌</h4>

```java

 	//없는 계정인 경우
        userNotFoundException(userIdPwdRequest, loginUserInfo);

        //허용IP 대역 체크
        if(ipCheck(getClientIp()) < 1) {
            throw new NoPermitUserException(ApiResultCode.NO_PERMIT.getCode(), ApiResultCode.NO_PERMIT.getMessage());
        }

        //비밀번호 틀린경우
        badCredentialsException(userIdPwdRequest, loginUserInfo, failHistory);

        //lockout
        lockedException(userIdPwdRequest, loginUserInfo, failHistory);
        //비밀번호 만료
        credentialsExpiredException(loginUserInfo, failHistory);
        
        //만료된 세션 정리, 로그인 성공 후/동시 접속자수 제어 전에 수행 필요
        sessionCleanUpByUserId(userIdPwdRequest.getUser_id());
        
	//동시 접속자수 제어
        if(!loginUserInfo.getUser_id().equals(SUPER_USER)) {
            sessionExceedException();
            alarmProcess.getAlarmCode(maxSessionCount); //알람발생
        }

        //로그인 성공시, cookie와 token 발급
        final String cookie = cookieUtil.createAccessCookie();
        final String token = jwtUtil.createTokenUserInfo(loginUserInfo, cookie);
	
 	//로그인 record (로직생략)

        //login fail reset
        authenticationRepository.updateUserLoginFailCount(userIdPwdRequest.getUser_id(), 0);
       
```


