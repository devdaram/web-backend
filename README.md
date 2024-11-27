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

      1. java 17
      2. Springboot 3.1.2


<h4>1. RestAPI 개발 - 사용자 관리 controller 일부 발췌</h4>

```java

    /**
    * 공통응답 class 생성 - 일부 수정발췌
    */
    @JsonInclude(JsonInclude.Include.NON_NULL)
    public class ApiFormat<T> {

	@Setter
	private ApiResult result;
	@Setter
	private T data;
    }

    public class ApiResult {

	private int code;
	private String message;

	public ApiResult(int code, String message) {
		this.code = code;
		this.message = message;
	}
    }


    /**
    * result 공통 규격 생성 - 일부 수정 발췌
    */
    @Getter
    @AllArgsConstructor
    public enum ApiResultCode {

	    PROCESS_SUCCESS(HttpStatus.OK, "ip.PROCESS_SUCCESS"),
	    ..... 생략 ......
	
	    private final HttpStatus code;
	    private final String message;
	
	    public int getStatusCode() {
	        return code.value();
	    }
	
	    public ApiResult apiResult() {
	        return new ApiResult(this.code.value(), this.message);
	    }
    }


    /**
    * 일괄 에러핸들링을 위한 ExceptionAdvice 클래스 - 일부 수정 발췌
    */
    @ControllerAdvice
    @RequiredArgsConstructor
    @Slf4j
    public class ExceptionAdvice {

	    @ExceptionHandler(RuntimeException.class)
	    public ProblemDetail runtimeException(RuntimeException exception) {
	        log.error("error : {}", exception.getMessage());
	
	        return ProblemDetail.forStatusAndDetail(HttpStatus.INTERNAL_SERVER_ERROR, exception.getMessage());
	    }
	
	    @ExceptionHandler
	    public ProblemDetail illegalArgument(IllegalArgumentException exception) {
	        return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, exception.getMessage());
	    }
	
	    @ExceptionHandler
	    public ProblemDetail MethodArgumentNotValidException(MethodArgumentNotValidException exception) {
	        return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, 	 
                exception.getFieldErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage).collect(Collectors.joining()));
    	    }
}
```
<br>

<h4>2. 다국어처리 - 일부 수정발췌</h4>

```java

/**
* ENUM 클래스에서 로깅메세지 매칭
*/
PROCESS_SUCCESS(HttpStatus.OK , "code.PROCESS_SUCCESS"),
USER_LOGIN_SUCCESS(HttpStatus.OK, "code.USER_LOGIN_SUCCESS"),
USER_LOGOUT_SUCCESS(HttpStatus.OK, "code.USER_LOGOUT_SUCCESS"),



/**
* 비즈니스로직에서 return 시 아래와 같이 return 할 수 있도록 규격 통일화
*/
return new ApiFormat(ApiResultCode.PROCESS_SUCCESS.apiResult(), info);


/**
* 사용자가 로그인한 locale을 받아오는 부분
*/
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

//message_ko.xml 와 같이 국가별로 xml 파일을 따로 생성
menu:
  INVALID: 유효하지 않은 형식 입니다.
  MENU_SEARCH_SUCCESS: 메뉴 조회 성공
  PROCESS_SUCCESS: 처리 성공
```
<br>
<h4>3. 권한/ 세션관리 - 로직 구조 일부발췌</h4>

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


//login fail reset
authenticationRepository.updateUserLoginFailCount(userIdPwdRequest.getUser_id(), 0);
       
```
<br>
<h4>4. 파일 업로드/ 다운로드 처리 - 로직 구조 일부발췌</h4>

```java
/**
* 이미지를 올릴 서버에 디렉터리를 만들기 위함
*/
private String makeImageDir(/*UserRoleType userInfo, String imageId*/) {

	chmod777Command();
	String filePath = imageUploadPath + getNowDate() + "/";

	String folderPath = filePath.replace("/", File.separator);
	log.info("**** folderPath : {}", folderPath);
	File uploadPathFolder = new File(folderPath);

	if (!uploadPathFolder.exists()) {
		uploadPathFolder.mkdirs();
		log.info(" Success to make Directory : {}", uploadPathFolder);
	}

	log.info(" folderPath : {}", folderPath);
	return folderPath;
}


/**
* 이미지를 서버에 올릴때 권한때문에 접근제한 될 수 있으므로 아래와 같은 메소드생성
*/
private void chmod777Command() {

	String folderPath = pkgUploadPath.replace("/", File.separator);
	log.info("**** folderPath : {}", folderPath);
	File uploadPath = new File(folderPath);

	if (uploadPath.exists())
		return;

	try {
		Runtime.getRuntime().exec("chmod -R 777 " + pkgUploadPath);

	}catch (IOException e){
		log.error("chmod command to Server error : {}", e);
	}
}


/**
* 이미지를 서버에 올리는 로직
*/

String imageId = KeyUtil.getDbKey();
String folderPath = makeImageDir();
String imageUri = "/api/v1/common/image?imageId="+ imageId;

String imageType = FilenameUtils.getExtension(image.getOriginalFilename());
log.error("FileName : {}", image.getName());

//확장자 체크
if(!isPermissionImageMimeType(imageType)){
	log.error("Fail to make imageType : {}", imageType);
	throw new ProcessErrorException(ApiResultCode.PROCESS_ERROR_OR_FAIL.getMessage());
}

if(!StringUtils.hasLength(folderPath)){
	log.error("Fail to make Directory : {}", folderPath);
	throw new ProcessErrorException(ApiResultCode.PROCESS_ERROR_OR_FAIL.getMessage());
}

try {
	File imageFile = new File(folderPath + imageId);
	image.transferTo(imageFile);

} catch (IOException e){
	log.error("Fail to upload to server : {}",e);
}

//DB에 image 정보 저장
imageRepository.insertImageInfo(entity);
log.debug("Success to upload file : {}", entity);

//response
ImageResponse response = ImageResponse.builder()
		.image_id(imageId)
		.image_file_name(image.getOriginalFilename())
		.image_uri(imageUri)
		.image_type(imageType)
		.build();


```
<br>

<h4>5. 허용 IP 대역체크 - 로직 구조 일부 수정 발췌</h4>

```java
private static final InetAddressValidator validator = InetAddressValidator.getInstance();

/**
* IP 형식이 맞는지 체크
*/
if(ipRequest.getIp_type().equals(IPV4)) { //ipv4
	if (!validator.isValidInet4Address(ipRequest.getFrom_ip()) || !validator.isValidInet4Address(ipRequest.getTo_ip())) 
		throw new InvalidException(ApiResultCode.MISSING_REQUIRED_VALUE.getCode(), ApiResultCode.MISSING_REQUIRED_VALUE.getMessage());
	
	
} else {//ipv6
	if (!validator.isValidInet6Address(ipRequest.getFrom_ip()) || !validator.isValidInet6Address(ipRequest.getTo_ip())) 
		throw new InvalidException(ApiResultCode.MISSING_REQUIRED_VALUE.getCode(), ApiResultCode.MISSING_REQUIRED_VALUE.getMessage());
	
}



/**
* ip 대역의 범위가 맞는지 유효성검사를 하기 위해 BigInteger로 변환하여 값비교를 진행
*/
IPAddressString addrString = new IPAddressString(ipRequest.getFrom_ip());
IPAddress addr = addrString.getAddress();
BigInteger fromValue = addr.getValue(); //toValue 도 동일하게 진행
String fromIp = addr.toFullString(); //toIP도 동일하게 진행
log.info("** fromIp convert to String : {}",fromIp);

if(ipRequest.getIp_type().equals(IPV4)){
	fromIp = addr.toCanonicalString();
	log.info("IPv4 fromIp convert to canonicalString : {}",fromIp);
}


if(ipRequest.getIp_type().equals(IPV4)) {
	if(toValue.compareTo(fromValue) <= 0) {
		throw new InvalidException(ApiResultCode.INVALID_RANGE.getCode(), ApiResultCode.INVALID_RANGE.getMessage());
	}
}

if(ipRequest.getIp_type().equals(IPV6)) {
	if(fromIp.equals(toIp)) {
		throw new InvalidException(ApiResultCode.INVALID_RANGE.getCode(), ApiResultCode.INVALID_RANGE.getMessage());
	}
	//IPv6는 비교하기 위해 IP6Address를 이용
	BigInteger from = IPv6Address.fromString(fromIp).toBigInteger();
	BigInteger to = IPv6Address.fromString(toIp).toBigInteger();

	if(to.compareTo(from) <= 0) {
		throw new InvalidException(ApiResultCode.INVALID_RANGE.getCode(), ApiResultCode.INVALID_RANGE.getMessage());
	}
}

```
