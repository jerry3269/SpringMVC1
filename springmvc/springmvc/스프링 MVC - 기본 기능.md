# 스프링 MVC - 기본 기능

<br><br>

# __1. 프로젝트 생성__

<br>

__스프링 부트 스타터 사이트로 이동해서 스프링 프로젝트 생성__

`https://start.spring.io`

- 프로젝트 선택
	- `Project`: Gradle Project
	- `Language`: Java
	- `Spring Boot`: 2.4.x

- Project Metadata
	- `Group`: hello
	- `Artifact`: springmvc
	- `Name`: springmvc
	- `Package name`: hello.springmvc
	- `Packaging`: Jar (주의!)
	- `Java`: 11

- `Dependencies`: Spring Web, Thymeleaf, Lombok

<br>

- `war`를 사용하면 `/webapp/` 경로에 index.html 을 두면 `http://localhost:8080` 호출시 `index.html` 페이지가 열린다.
- `war`는 내장 서버도 사용 가능 하지만, 주로 외부 서버 배포 목적으로 사용
- JSP를 사용하지 않기 때문에 `Jar`사용
- `Jar`를 사용하면 `webapp` 경로 사용 x, 내장 톰켓 서버 사용에 최적화
- `Jar`는 `/resources/static/` 위치에 `index.html` 파일을 Welcome 페이지로 처리

<br><br>

# __2. 로깅 간단히 알아보기__

<br>

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리(`spring-boot-starter-logging`)가 포함된다.
스프링 부트 로깅 라이브러리는 다음 로깅 라이브러리를 사용한다.

- `SLF4J` : 로그 라이브러리를 통합하여 인터페이스로 제공
- `Logback` : 로그 라이브러리 

<br>

## __로그 선언__
- `@Slf4j`

<br>

## __로그 호출__

```java
String name = "Spring";

log.trace("trace log={}", name);
log.debug("debug log={}", name);
log.info(" info log={}", name);
log.warn(" warn log={}", name);
log.error("error log={}", name);
```

<br>

### __RestController__
- `@Controller`는 반환 값이 String이면 뷰 이름으로 인식 됨
- `@RestController`는 반환 값으로 뷰를 찾지 않고, HTTP 메시지 바디에 직접 입력

<br>

### __로그 레벨__
- LEVEL: `TRACE` > `DEBUG` > `INFO` > `WARN` > `ERROR`

<br>

### __로그 레벨 설정__

__`application.properties`__

```java
#전체 로그 레벨 설정(기본 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

- `개발 서버`는 `debug` 출력
- `운영 서버`는 `info` 출력

<br>

## __올바른 로그 사용법__
- `log.debug("data =" + data)`
	- 로그 출력 레벨을 info로 설정하면 디버그 메시지는 출력되지 않짐나 해당 연산은 실행됨
- `log.debug("data ={}", data)`
	- 로그 출력 레벨을 info로 설정하면 아무 일도 발생 하지 않음. 따라서 이 방법으로 로그를 작성해야 함

<br>

## __로그 장점__
- `쓰레드 정보`, `클래스 이름` 같은 부가 정보를 볼 수 있고, 출력 모양 조절 가능
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력, 운영 서버에서는 출력하지 않는 등 로그 출력 레벨 설정 가능
- 시스템 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등 로그를 별도의 위치에 저장 가능
	- 특히 파일로 남길 때 , 일별, 특정 용량에 따라 로그 분할도 가능
- 일반 `System.out`보다 성능이 좋음. 
	- 내부 버퍼링, 멀티 쓰레드 등

<br><br>

# __3. 요청 매핑__

<br>

## __둘다 허용 - 스프링 부트 3.0 이전__
- 다음 두가지 요청은 다른 URL이지만, 스프링은 다음 URL 요청들을 같은 요청으로 매핑한다.
- 매핑: `/hello-basic`
- URL 요청: `/hello-basic` , `/hello-basic/`

<br>

## 스프링 부트 3.0 이후 <br>
> 스프링 부트 3.0 부터는 `/hello-basic` , `/hello-basic/` 는 서로 다른 URL 요청을 사용해야 한다. <br>
기존에는 마지막에 있는 `/ `(slash)를 제거했지만, 스프링 부트 3.0 부터는 마지막의 / (slash)를 유지한다. <br>
따라서 다음과 같이 다르게 매핑해서 사용해야 한다. <br>
매핑: `/hello-basic` -> URL 요청: `/hello-basic` <br>
매핑: `/hello-basic/` -> URL 요청: `/hello-basic/` <br>

<br>

## __PathVariable(경로 변수) 사용__

<br>

- @GetMapping("/mapping/{userId}")
	- @RequestMapping 은 URL 경로를 템플릿화(`동적 처리`) 할 수 있는데, `@PathVariable` 을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.

<br>

## __특정 파라미터 조건 매핑__

```java
/**
 * 파라미터로 추가 매핑
 * params="mode",
 * params="!mode"
 * params="mode=debug"
 * params="mode!=debug" (! = )
 * params = {"mode=debug","data=good"}
 */
@GetMapping(value = "/mapping-param", params = "mode=debug")
```

<br>

### __실행__
- `http://localhost:8080/mapping-param?mode=debug`

특정 파라미터가 있거나 없는 조건을 추가할 수 있다. 잘 사용하지는 않는다.

<br>

## __특정 헤더 조건 매핑__

```java
/**
 * 특정 헤더로 추가 매핑
 * headers="mode",
 * headers="!mode"
 * headers="mode=debug"
 * headers="mode!=debug" (! = )
 */
@GetMapping(value = "/mapping-header", headers = "mode=debug")
```

<br>

## __미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume__

```java
/**
     * Content-Type 헤더 기반 추가 매핑 Media Type (요청 메시지의 Content-Type)
     * consumes="application/json"
     * consumes="!application/json"
     * consumes="application/*"
     * consumes="*\/*"
     * MediaType.APPLICATION_JSON_VALUE
     */
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
```

HTTP 요청의 `Content-Type` 헤더를 기반으로 미디어 타입으로 매핑한다.
만약 맞지 않으면 `HTTP 415` 상태코드(`Unsupported Media Type`)을 반환한다.

<br>

## __미디어 타입 조건 매핑 - HTTP 요청 Accept, produce__

```java
/**
     * Accept 헤더 기반 Media Type (요청 메시지 헤더의 Accept 헤더 )
     * Accept 헤더 : client가 서버로 부터 받을 수 있는 Content-type)
     * produces = "text/html"
     * produces = "!text/html"
     * produces = "text/*"
     * produces = "*\/*"
     */
    @PostMapping(value = "/mapping-produce", produces = "text/html")
```

HTTP 요청의 `Accept` 헤더를 기반으로 미디어 타입으로 매핑한다.
만약 맞지 않으면 `HTTP 406 상태코드`(`Not Acceptable`)을 반환한다.

<br><br>

# __4. HTTP 요청 - 기본, 헤더 조회__

<br>

- HttpServletRequest
- HttpServletResponse
- HttpMethod : 요청 메서드 조회
- Locale : 가장 선호하는 언어 조회
- @RequestHeader MultiValueMap<String, String> headerMap
	- 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
- @RequestHeader("host") String host
	- host HTTP 헤더를 조회한다.
	- 속성
		- 필수 값 여부: required
		- 기본 값 속성: defaultValue
- @CookieValue(value = "myCookie", required = false) String cookie
	- myCookie 쿠키를 조회한다.
	- 속성
		- 필수 값 여부: required
		- 기본 값: defaultValue

<br>

### __MultiValueMap__
- `MAP`과 유사한데, 하나의 키에 여러 값을 받을 수 있다.

```java
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");

//[value1,value2]
List<String> values = map.get("keyA");
```

<br><br>

# __5. HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form__

<br>

`HttpServletRequest` 의 `request.getParameter()` 를 사용하면 다음 두가지 요청 파라미터를 조회할 수 있다.

`GET 쿼리 파리미터` 전송 방식이든, `POST HTML Form` 전송 방식이든 둘다 형식이 같으므로 구분없이 조회할 수 있다.

이것을 간단히 요청 파라미터(`request parameter`) 조회라 한다.

<br><br>

# __6. HTTP 요청 파라미터 - @RequestParam__

<br>

### __requestParamV2__
```java
/**
     * @RequestParam 사용
     * - 파라미터 이름으로 바인딩
     * @ResponseBody 추가
     * - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
     */

    @ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge)  {

        log.info("username = {}, age = {}", memberName, memberAge);

        return "ok";
    }
```
`@RequestParam`을 사용하면 파라미터 이름으로 바인딩 해줄 뿐 만 아니라 형도 자동으로 변환 해준다. 

- ex) `@RequestParam("age") int memberAge`
- 원래는 `Integer.parseInt`를 사용해서 형을 변환해서 값을 받아야 한다.


<br>

### __requestParamV3__

```java
    /**
     * @RequestParam 사용
     * HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
     */
    @ResponseBody
    @RequestMapping("/request-param-v3")
    public String requestParamV3(
            @RequestParam String username,
            @RequestParam int age) {
        log.info("username={}, age={}", username, age);
        return "ok";
    }
```

<br>

### __requestParamV4__
```java
/**
     * @RequestParam 사용
     * String, int 등의 단순 타입이면 @RequestParam 도 생략 가능
     */
    @ResponseBody
    @RequestMapping("/request-param-v4")
    public String requestParamV4(String username, int age) {
        log.info("username={}, age={}", username, age);
        return "ok";
    }
```

> __주의__ <br>
`@RequestParam` 애노테이션을 생략하면 스프링 MVC는 내부에서 `required=false` 를 적용한다.

<br>

### __requestParamRequired__
```java
/**
     * @RequestParam.required
     * /request-param-required -> username이 없으므로 400 예외 발생
     *
     * 주의!
     * /request-param-required?username= -> 빈문자로 통과 (오류 발생 하지 않음)
     *
     * 주의!
     * /request-param-required
     * int age = null 이 들어가야함
     * int age -> null을 int에 입력하는 것은 불가능(500 예외 발생, 따라서 Integer 변경해야 함(또는 다음에 나오는 defaultValue 사용)
     */
    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username,
            @RequestParam(required = false) Integer age) {
        log.info("username={}, age={}", username, age);
        return "ok";
    }
```

<br>

### __requestParamDefault__
```java
    /**
     * @RequestParam
     * - defaultValue 사용
     *
     * 참고: defaultValue는 빈 문자의 경우에도 적용
     * /request-param-default?username=
     * username=guest, age=-1
     * 사실 defaultValue 있으면 required 필요 없음
     */
    @ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefault(
            @RequestParam(required = true, defaultValue = "guest") String username,
            @RequestParam(required = false, defaultValue = "-1") int age) {
        log.info("username={}, age={}", username, age);
        return "ok";
    }
```

<br>

### __requestParamMap__
```java
 /**
     * @RequestParam Map, MultiValueMap
     * 파라미터 한번에 조회
     * Map(key=value)
     * MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])
     */
    @ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
        log.info("username={}, age={}", paramMap.get("username"),
                paramMap.get("age"));
        return "ok";
    }
```

<br><br>

# __7. HTTP 요청 파라미터 - @ModelAttribute__

<br>

```java
@RequestParam String username;
@RequestParam int age;

HelloData data = new HelloData();
data.setUsername(username);
data.setAge(age);
```

- 실제 개발에서 요청 파라미터를 받으면 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다. 
- 스프링은 이 과정을 완전히 자동화 해주는 `@ModelAttribute`기능을 제공한다.

<br>

요청 파라미터를 바인딩 받을 객체를 만들어보자

### __HelloData__
```java
package hello.springmvc.basic;

import lombok.Data;

@Data
public class HelloData {
 	private String username;
 	private int age;
}
```

- `@Data`
	- `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor`를 자동으로 적용해준다.

<br>

### __@ModelAttribute 적용 - modelAttributeV1__
```java
/**
     * @ModelAttribute 사용
     * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때
    자세히 설명
     */
    @ResponseBody
    @RequestMapping("/model-attribute-v1")
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        log.info("helloData = {}", helloData);
        return "ok";
    }
```

- 스프링MVC는 `@ModelAttribute` 가 있으면 다음을 실행한다.
- HelloData 객체를 생성한다.
- 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.
- 예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.

<br>

### __프로퍼티__
- 객체에 `getUsername()`, `setUsername()` 메서드가 있으면, 이 객체는 `username`이라는 프로퍼티를 가지고 있다.
- `username` 프로퍼티의 값을 변경하면 `setUsername()`이 호출되고, 조회하면 `getUsername()`이 호출된다.

<br>

### __@ModelAttribute 생략 - modelAttributeV2__
```java
@ResponseBody
    @RequestMapping("/model-attribute-v2")
    public String modelAttributeV2(HelloData helloData) {
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
        log.info("helloData = {}", helloData);
        return "ok";
    }
```
- `@ModelAttribute` 는 생략할 수 있다.
- 그런데 `@RequestParam` 도 생략할 수 있으니 혼란이 발생할 수 있다.

- `String , int , Integer` 같은 단순 타입 = `@RequestParam`
- 나머지 = `@ModelAttribute` (`argument resolver` 로 지정해둔 타입 외)

<br><br>

# __8. HTTP 요청 메시지 - 단순 텍스트__

<br>

- 요청 파라미터와 다르게, HTTP 메시지 바디로 데이터가 직접 넘어오는 경우 `@RequestParam`, `@ModelAttribute`를 사용할 수 없다. 
- `HTML Form` 형식으로 전달되는 경우는 요청 파라미터로 인식

## __requestBodyStringV1__

```java
@PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
       
	ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody = {}", messageBody);

        response.getWriter().write("ok");
    }
```

- 앞서 서블릿에서 본것과 같이 `request` 메시지 바디를 `getInputStream()`을 통해 바이트 코드로 전환하고, `StreamUtils.copyToString`로 바이트코드를 `utf-8`로 인코딩 한다.
- `response`에서 `writer`객체를 가져와 직접 `write()`를 수행한다.

<br>

### __실행(Postman)__
- POST `http://localhost:8080/request-body-string-v1`
- `Body` `row`, `Text`` 선택

<br>

## __Input, Output 스트림, Reader - requestBodyStringV2__

```java
@PostMapping("/request-body-string-v2")
    public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody = {}", messageBody);

        responseWriter.write("ok");
    }
```

- `InputStream(Reader)`: HTTP 요청 메시지 바디의 내용을 직접 조회
- `OutputStream(Writer)`: HTTP 응답 메시지의 바디에 직접 결과 출력

<br>

## __HttpEntity - requestBodyStringV3__

```java
@PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

        String messageBody = httpEntity.getBody();
        log.info("messageBody = {}", messageBody);

        return new HttpEntity<>("ok");
    }
```

- `HttpEntity` : `HTTP header`, `body` 정보를 편리하게 조회
	- `Http message body`를 스펙화 해놓은 것
	- 메시지 바디 정보를 직접 조회
	- 요청 파라미터를 조회하는 기능과 관계 없음 (`@RequestParam`, `@ModelAttribute`)
- `HttpEntity`는 응답에도 사용 가능
	- 메시지 바디 정보 직접 반환
	- 헤더 정보 포함 가능
	- `view`를 통한 조회x
- `HttpMessageConverter` 사용 -> `StringHttpMessageConverter` 적용

<br>

- `HttpEntity` 를 상속받은 다음 객체들도 같은 기능을 제공
- `RequestEntity`
	- HttpMethod, url 정보가 추가, 요청에서 사용
- `ResponseEntity`
	- HTTP 상태 코드 설정 가능, 응답에서 사용
	- `return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)`

<br>

> __참고__ <br>
스프링MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 이때 HTTP 메시지 컨버터( `HttpMessageConverter` )라는 기능을 사용한다. 이것은 조금 뒤에 HTTP 메시지 컨버터에서 자세히 설명한다.

<br>

## __@RequestBody - requestBodyStringV4__

```java
@ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {

        log.info("messageBody = {}", messageBody);

        return "ok";
    }
```

### __@RequestBody__
`@RequestBody` 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가 필요하다면 `HttpEntity` 를 사용하거나 `@RequestHeader` 를 사용하면 된다. 

이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 `@RequestParam` , `@ModelAttribute` 와는 전혀 관계가 없다.

<br>

### __요청 파라미터 vs HTTP 메시지 바디__
- 요청 파라미터를 조회하는 기능: `@RequestParam` , `@ModelAttribute`
- HTTP 메시지 바디를 직접 조회하는 기능: `@RequestBody`

<br>

### __@ResponseBody__
- `@ResponseBody` 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.
물론 이 경우에도 `view`를 사용하지 않는다.

<br><br>

# __9. HTTP 요청 메시지 - JSON__

<br>

## __requestBodyJsonV1__

```java
private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);


        log.info("messageBody={}", messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={} , age={}", helloData.getUsername(), helloData.getAge());

        response.getWriter().write("ok");
    }
```

- `HttpServletRequest`를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
- 문자로 된 `JSON` 데이터를 Jackson 라이브러리인 `objectMapper` 를 사용해서 자바 객체로 변환한다.

<br>

### __실행(Postman)__
- POST http://localhost:8080/request-body-json-v1
- raw, JSON, content-type: application/json
- {"username":"hello", "age":20}

<br>

## __requestBodyJsonV2 - @RequestBody 문자 변환__

```java
@ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

        log.info("messageBody={}", messageBody);
        HelloData data = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={} , age={}", data.getUsername(), data.getAge());

        return "ok";
    }
```

- 이전에 학습했던 `@RequestBody` 를 사용해서 HTTP 메시지에서 데이터를 꺼내고 `messageBody`에
저장한다.
- 문자로 된 `JSON` 데이터인 `messageBody` 를 `objectMapper` 를 통해서 자바 객체로 변환한다.

<br>

## __requestBodyJsonV3 - @RequestBody 객체 변환__
- @ModelAttribute처럼 한번에 객체로 변환하기

```java
@ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData data) throws IOException {

        log.info("username={} , age={}", data.getUsername(), data.getAge());
        return "ok";
    }
```

- `HttpEntity` , `@RequestBody` 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가
원하는 문자나 객체 등으로 변환해준다.
- HTTP 메시지 컨버터는 문자 뿐만 아니라 `JSON`도 객체로 변환해주는데, 우리가 방금 `V2`에서 했던 작업을
대신 처리해준다.

<br>

### __@RequestBody 생략 불가능__
- 스프링 부트에서 파라미터의 애노테이션을 생략한다면 
	- `단순타입` = @RequestParam
	- `그외` = @ModelAttribute(argument resolver로 지정해둔 타입 외)
- 따라서 `@RequestBody`를 생략할 수 없다.

<br>

> __주의__ <br>
HTTP 요청시에 content-type이 application/json인지 꼭! 확인해야 한다. 그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.

<br>

## __requestBodyJsonV4 - HttpEntity__

```java
@ResponseBody
    @PostMapping("/request-body-json-v4")
    public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) throws IOException {
        HelloData data = httpEntity.getBody();
        log.info("username={} , age={}", data.getUsername(), data.getAge());
        return "ok";
    }
```

<br>

## __requestBodyJsonV5__

```java
  @ResponseBody
    @PostMapping("/request-body-json-v5")
    public HelloData requestBodyJsonV5(@RequestBody HelloData data) throws IOException {
        log.info("username={} , age={}", data.getUsername(), data.getAge());
        return data;
    }
```

### __@ResponseBody
- 응답의 경우도 객체를 HTTP 메시지 바디에 직접 넣어 줄 수 있다.
- 반환 타입 `HttpEntity` 사용가능

<br>

- @RequestBody 요청
	- JSON 요청 -> HTTP 메시지 컨버터 -> 객체
	-  HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type: application/json)
- @ResponseBody 응답
	- 객체 -> HTTP 메시지 컨버터 -> JSON 응답
	- HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용(Accept: application/json)

<br><br>


# __10. HTTP 응답 - 정적 리소스, 뷰 템플릿__

<br>

서버에서 응답 데이터를 만드는 방법은 3가지이다.

- `정적 리소스`
	- 예) 웹 브라우저에 정적인 `HTML, css, js`를 제공할때 정적리소스 사용
- `뷰 템플릿 사용`
	- 예) 웹 브라우저에 동적인 HTML을 제공할 때 `뷰 템플릿` 사용
- `HTTP 메시지 사용`
	- HTTP API를 제공하는 경우 HTML이 아니라 `데이터`를 전달해야 하므로, HTTP 메시지 바디에 `JSON`과 같은 형식으로 데이터를 보냄

<br>

## __정적 리소스__
다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

```python
정적 리소스 경로
src/main/resources/static

다음 경로에 파일이 들어있으면
src/main/resources/static/basic/hello-form.html

웹 브라우저에서 다음과 같이 실행하면 된다.
http://localhost:8080/basic/hello-form.html

정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.
```

<br>

## __뷰 템플릿__
- 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
- 일반적으로 HTML을 동적으로 생성하는 용도
- 스프링 부트는 기본 뷰 템플릿 경로를 제공한다.

```python
뷰 템플릿 경로
src/main/resources/templates

뷰 템플릿 생성
src/main/resources/templates/response/hello.html
```
<br>

## __뷰 템플릿을 호출하는 컨트롤러__

```java
@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1(){
        ModelAndView mav = new ModelAndView("response/hello")
                .addObject("data", "hello!");
        return mav;
    }

    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model){
        model.addAttribute("data", "hello!");
        return "response/hello";
    }

    @RequestMapping("response/hello")
    public void responseViewV3(Model model){
        model.addAttribute("data", "hello!");
    }
}
```

### __String을 반환하는 경우__
- `@ResponseBody` 가 없으면 `response/hello` 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.
- `@ResponseBody` 가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 `response/hello` 라는
문자가 입력된다.

<br>

### __Void를 반환하는 경우__
`@Controller` 를 사용하고, `HttpServletResponse` , `OutputStream(Writer)` 같은 `HTTP 메시지
바디`를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용

<br>

### __HTTP 메시지__
`@ResponseBody` , `HttpEntity` 를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에
직접 응답 데이터를 출력할 수 있다.

<br><br>

# __11. HTTP 응답 - HTTP API, 메시지 바디에 직접 입력__

<br>

```java
@Slf4j
@Controller
//@RestController
public class ResponseBodyController {

    @GetMapping("/response-body-string-v1")
    public void responseBodyStringV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }


    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyStringV2() {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }

    @ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyStringV3() {
        return "ok";
    }

    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);
        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);
        return helloData;
    }
}
```

### __responseBodyV1__
- 서블릿을 직접 다룰 때 처럼
`HttpServletResponse` 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다.

<br>

### __responseBodyV2__
- `ResponseEntity` 는 HttpEntity 를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디
정보를 가지고 있다. ResponseEntity 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
- HttpStatus.CREATED 로 변경하면 201 응답이 나가는 것을 확인할 수 있다.

<br>

### __responseBodyV3__
- `@ResponseBody` 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접
입력할 수 있다. ResponseEntity 도 동일한 방식으로 동작한다.

<br>

### __responseBodyJsonV1__
- `ResponseEntity` 를 반환한다. HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.

<br>

### __responseBodyJsonV2__
- `ResponseEntity` 는 HTTP 응답 코드를 설정할 수 있는데, `@ResponseBody` 를 사용하면 이런 것을
설정하기 까다롭다.
- `@ResponseStatus(HttpStatus.OK)` 애노테이션을 사용하면 응답 코드도 설정할 수 있다.
- 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 동적으로
변경하려면 `ResponseEntity` 를 사용하면 된다.

<br>

### __@RestController__
- 참고로 `@ResponseBody` 는 클래스 레벨에 두면 전체 메서드에 적용된다.
- `@RestController`는 `@Controller와 @ResponseBody`가 합쳐진 것이다.

<br><br>

# __12. HTTP 메시지 컨버터__

<br>

뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지
바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.

<br>

- @ResponseBody 를 사용
	- HTTP의 BODY에 문자 내용을 직접 반환
	- viewResolver 대신에 HttpMessageConverter 가 동작
	- 기본 문자처리: StringHttpMessageConverter
	- 기본 객체처리: MappingJackson2HttpMessageConverter
	- byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

> __참고__ : 응답의 경우 클라이언트의 HTTP Accept 해더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서
`HttpMessageConverter` 가 선택된다.

<br>

### 스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.
- HTTP 요청: `@RequestBody` , `HttpEntity(RequestEntity)` 
- HTTP 응답: `@ResponseBody` , `HttpEntity(ResponseEntity)`

<br>

### HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.
- `canRead() , canWrite()` : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- `read() , write()` : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

<br>

### 스프링 부트 기본 메시지 컨버터

```
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter 
2 = MappingJackson2HttpMessageConverter
....
```

스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서
사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.


몇가지 주요한 메시지 컨버터를 알아보자.
- `ByteArrayHttpMessageConverter` : byte[] 데이터를 처리한다.
	- 클래스 타입: `byte[]` , 미디어타입: `*/*` ,
	- 요청 예) @RequestBody byte[] data
	- 응답 예) @ResponseBody return byte[] 쓰기 미디어타입 application/octet-stream
- `StringHttpMessageConverter` : String 문자로 데이터를 처리한다.
	- 클래스 타입: `String` , 미디어타입: `*/*`
	- 요청 예) @RequestBody String data
	- 응답 예) @ResponseBody return "ok" 쓰기 미디어타입 text/plain
- MappingJackson2HttpMessageConverter : application/json
	- 클래스 타입: `객체` 또는 `HashMap` , 미디어타입 `application/json` 관련
	- 요청 예) @RequestBody HelloData data
	- 응답 예) @ResponseBody return helloData 쓰기 미디어타입 application/json 관련

<br>

### StringHttpMessageConverter
```
content-type: application/json

@RequestMapping
void hello(@RequestBody String data) {}
```

### MappingJackson2HttpMessageConverter
```
content-type: application/json

@RequestMapping
void hello(@RequestBody HelloData data) {}
```

### 탈락
```
content-type: text/html

@RequestMapping
void hello(@RequestBody HelloData data) {}
```

<br>

## HTTP 요청 데이터 읽기
- HTTP 요청이 오고, 컨트롤러에서 `@RequestBody` , `HttpEntity` 파라미터를 사용한다.
- 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 `canRead()` 를 호출한다.
	- 대상 `클래스 타입`을 지원하는가.
		- 예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )
	- HTTP 요청의 `Content-Type` 미디어 타입을 지원하는가.
		- 예) text/plain , application/json , */*
- `canRead()` 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다.

<br>

## HTTP 응답 데이터 생성
- 컨트롤러에서 `@ResponseBody` , `HttpEntity` 로 값이 반환된다. 
- 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 `canWrite()` 를 호출한다.
	- 대상 `클래스 타입`을 지원하는가.
		- 예) return의 대상 클래스 ( byte[] , String , HelloData )
	- HTTP 요청의 `Accept` 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces )
		- 예) text/plain , application/json , */*
- `canWrite()` 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

<br><br>

# __13. 요청 매핑 헨들러 어뎁터 구조__

<br>

그렇다면 HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는 것일까?

바로 `@RequestMapping`을 처리하는 핸들러 어댑터인 `RequestMappingHandlerAdapter` (요청 매핑 헨들러 어뎁터)에 있다.

<br>

## RequestMappingHandlerAdapter 동작 방식

### ArgumentResolver
```
생각해보면, 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다. <br>
`HttpServletRequest , Model` 은 물론이고, `@RequestParam , @ModelAttribute` 같은 애노테이션 <br>
그리고 `@RequestBody , HttpEntity` 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보여주었다.

이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 `ArgumentResolver` 덕분이다.

애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter` 는 바로 이 <br>
`ArgumentResolver` 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다. 

그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
```

<br>

> __참고__ <br>
가능한 파라미터 목록은 다음 공식 메뉴얼에서 확인할 수 있다. <br>
https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments

<br>

정확히는 `HandlerMethodArgumentResolver` 인데 줄여서 `ArgumentResolver` 라고 부른다.

<br>

## 동작 방식
`ArgumentResolver` 의 `supportsParameter()` 를 호출해서 루프를 돌며 해당 파라미터를 지원하는지 체크하고, 
지원하면 resolveArgument() 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러
호출시 넘어가는 것이다.

그리고 원한다면 여러분이 직접 이 인터페이스를 확장해서 원하는 `ArgumentResolver` 를 만들 수도 있다. 

<br>

## ReturnValueHandler
`HandlerMethodReturnValueHandler` 를 줄여서 `ReturnValueHandler` 라 부른다. <br>
`ArgumentResolver` 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.

<br>

컨트롤러에서 `String`으로 뷰 이름을 반환해도, 동작하는 이유가 바로 `ReturnValueHandler` 덕분이다.

스프링은 10여개가 넘는 ReturnValueHandler 를 지원한다.
- 예) `ModelAndView , @ResponseBody , HttpEntity , String`

<br>

> __참고__ <br>
가능한 응답 값 목록은 다음 공식 메뉴얼에서 확인할 수 있다. <br>
https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annreturn-types


<br>

## HTTP 메시지 컨버터

### __요청의 경우__ 
`@RequestBody` 를 처리하는 `ArgumentResolver` 가 있고, `HttpEntity` 를 처리하는
`ArgumentResolver` 가 있다. 이 `ArgumentResolver` 들이 HTTP 메시지 컨버터를 사용해서 필요한
객체를 생성하는 것이다. 

<br>

### __응답의 경우__
`@ResponseBody` 와 `HttpEntity` 를 처리하는 `ReturnValueHandler` 가 있다. 그리고
여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

<br>

스프링 MVC는 `@RequestBody` `@ResponseBody` 가 있으면 <br>
`RequestResponseBodyMethodProcessor (ArgumentResolver)` <br>
`HttpEntity` 가 있으면 `HttpEntityMethodProcessor (ArgumentResolver)`를 사용한다.

<br>

### 확장
- 스프링은 다음을 모두 인터페이스로 제공한다. 따라서 필요하면 언제든지 기능을 확장할 수 있다.
	- `HandlerMethodArgumentResolver`
	- `HandlerMethodReturnValueHandler`
	- `HttpMessageConverter`

<br>

기능 확장은 `WebMvcConfigurer` 를 상속 받아서 스프링 빈으로 등록하면 된다. 
```java
@Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {

            @Override
            public void addArgumentResolvers(List<HandlerMethodArgumentResolver>resolvers) {
                //...
            }

            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                //...
            }
        };
    }
```






































 

































































