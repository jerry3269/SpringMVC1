# 서블릿, JSP, MVC 패턴

<br><br>

# 1. 회원 관리 웹 애플리케이션 요구사항

<br>

## __회원 정보__

- `이름`: username
- `나이`: age

<br>

## __기능 요구사항__
- `회원 저장`
- `회원 목록 조회`

<br>

## [__회원 도메인 모델__](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/domain/member/Member.java)

- `id` 는 `Member` 를 회원 저장소에 저장하면 회원 저장소가 할당한다

<br>

## [__회원 저장소__](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/domain/member/MemberRepository.java)
- 회원 저장소는 `싱글톤 패턴`을 적용했다. 스프링을 사용하면 스프링 빈으로 등록하면 되지만, 지금은 최대한 스프링 없이 순수 서블릿 만으로 구현하는 것이 목적이다. 싱글톤 패턴은 객체를 단 하나만 생생해서 공유해야 하므로 생성자를 `private` 접근자로 막아둔다

<br>

### [__회원 저장소 테스트 코드__](https://github.com/jerry3269/springMVC1/blob/master/src/test/java/hello/servlet/domain/member/MemberRepositoryTest.java)
- 회원을 저장하고, 목록을 조회하는 테스트를 작성했다. 각 테스트가 끝날 때, 다음 테스트에 영향을 주지 않도록 각 테스트의 저장소를 `clearStore()` 를 호출해서 초기화했다.

<br><br>

# 2. 서블릿으로 회원 관리 웹 애플리케이션 만들기

<br>

## [__`MemberFormServlet` - 회원 등록 폼__](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/web/servlet/MemberFormServlet.java)
- MemberFormServlet 은 단순하게 회원 정보를 입력할 수 있는 HTML Form을 만들어서 응답한다. 자바 코드로 HTML을 제공해야 하므로 쉽지 않은 작업이다.

<br>

__`실행`__
- `http://localhost:8080/servlet/members/new-form`
- `HTML Form` 데이터를 `POST`로 전송해도, 전달 받는 서블릿을 아직 만들지 않았다. 그래서 오류가 발생하는 것이 정상이다.

<br><br>

이번에는 `HTML Form`에서 데이터를 입력하고 전송을 누르면 실제 회원 데이터가 저장되도록 해보자. 전송 방식은 `POST HTML Form`에서 학습한 내용과 같다.

<br>

## [__`MemberSaveServlet` - 회원 저장__](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/web/servlet/MemberSaveServlet.java)
`MemberSaveServlet` 은 다음 순서로 동작한다.
1. 파라미터를 조회해서 `Member` 객체를 만든다.
2. `Member` 객체를 `MemberRepository`를 통해서 저장한다.
3. `Member` 객체를 사용해서 결과 화면용 `HTML`을 동적으로 만들어서 응답한다.

<br>

__`실행`__
- `http://localhost:8080/servlet/members/new-form `
- 데이터가 전송되고, 저장 결과를 확인할 수 있다.

<br>

이번에는 저장된 모든 회원 목록을 조회하는 기능을 만들어보자

<br>

## [__`MemberListServlet` - 회원 목록__](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/web/servlet/MemberListServlet.java)
`MemberListServlet` 은 다음 순서로 동작한다.
1. `memberRepository.findAll()` 을 통해 모든 회원을 조회한다.
2. 회원 목록 `HTML`을 `for 루프`를 통해서 회원 수 만큼 동적으로 생성하고 응답한다.

<br>

__`실행`__
- `http://localhost:8080/servlet/members`
- 저장된 회원 목록을 확인할 수 있다.

<br>

## __템플릿 엔진으로__
- 지금까지 서블릿과 자바 코드만으로 `HTML`을 만들어보았다. 서블릿 덕분에 동적으로 원하는 HTML을 마음껏 만들 수 있다. 정적인 `HTML` 문서라면 화면이 계속 달라지는 회원의 저장 결과라던가, 회원 목록 같은 동적인 `HTML`을 만드는 일은 불가능 할 것이다.
- 그런데, 코드에서 보듯이 이것은 매우 복잡하고 비효율 적이다. 자바 코드로 `HTML`을 만들어 내는 것 보다 차라리 `HTML` 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣을 수 있다면 더 편리할 것이다. 이것이 바로 템플릿 엔진이 나온 이유이다. 템플릿 엔진을 사용하면 `HTML` 문서에서 필요한 곳만 코드를 적용해서 동적으로 변경할 수 있다.
- 템플릿 엔진에는 `JSP, Thymeleaf, Freemarker, Velocity`등이 있다.

<br><br>

다음 시간에는 `JSP`로 동일한 작업을 진행해보자

<br><br>

# 3. `JSP`로 회원 관리 웹 애플리케이션 만들기

<br>

__`JSP` 라이브러리 추가__

JSP를 사용하려면 먼저 다음 라이브러리를 추가해야 한다.

<br>

__스프링 부트 3.0 미만__

`build.gradle` 에 추가

```java
//JSP 추가 시작
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
implementation 'javax.servlet:jstl'
//JSP 추가 끝
```

<br>

__스프링 부트 3.0 이상__

`build.gradle` 에 추가

```java
//JSP 추가 시작
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
implementation 'jakarta.servlet:jakarta.servlet-api' //스프링부트 3.0 이상
implementation 'jakarta.servlet.jsp.jstl:jakarta.servlet.jsp.jstl-api' //
스프링부트 3.0 이상
implementation 'org.glassfish.web:jakarta.servlet.jsp.jstl' //스프링부트 3.0 이상
//JSP 추가 끝
```

스프링 부트 3.0 이상이면 `javax.servlet:jstl` 을 제거하고 위 코드를 추가해야 한다.


<br><br>

## [__회원 등록 폼 `JSP`__](https://github.com/jerry3269/springMVC1/blob/master/src/main/webapp/jsp/members/new-form.jsp)
- `<%@ page contentType="text/html;charset=UTF-8" language="java" %>`
	- 첫 줄은 JSP문서라는 뜻이다. JSP 문서는 이렇게 시작해야 한다.

<br>

__`실행`__
- `http://localhost:8080/jsp/members/new-form.jsp`
	- 실행시 .jsp 까지 함께 적어주어야 한다.

<br>

## [__회원 저장 `JSP`__](https://github.com/jerry3269/springMVC1/blob/master/src/main/webapp/jsp/members/save.jsp)
- `JSP`는 서버 내부에서 서블릿으로 변환

- `JSP`는 자바 코드를 그대로 다 사용할 수 있다.
	- `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`
		- 자바의 `import` 문과 같다.
	- `<% ~~ %>`
		- 이 부분에는 자바 코드를 입력할 수 있다.
	- `<%= ~~ %>`
		- 이 부분에는 자바 코드를 출력할 수 있다.

- JSP에서 `<% %>` 내부에는 자바 코드가 들어가는데 이때 서블릿에서 사용되는 `request`와 `response`모두 사용가능하다. `<% %>`내부가 아닌 외부는 `response`에 `write` 된다.

<br>

## [__회원 목록 `JSP`__](https://github.com/jerry3269/springMVC1/blob/master/src/main/webapp/jsp/members.jsp)
- 회원 리포지토리를 먼저 조회하고, 결과 `List`를 사용해서 중간에 `<tr><td> HTML 태그`를 반복해서 출력하고 있다.

<br>

## __`서블릿`과 `JSP`의 한계__
- 서블릿으로 개발할 때는 뷰(View)화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서 지저분하고 복잡했다.
- JSP를 사용한 덕분에 뷰를 생성하는 HTML 작업을 깔끔하게 가져가고, 중간중간 동적으로 변경이 필요한 부분에만 자바 코드를 적용했다. 그런데 이렇게 해도 해결되지 않는 몇가지 고민이 남는다.

- 회원 저장 `JSP`를 보자. 코드의 상위 절반은 회원을 저장하기 위한 비즈니스 로직이고, 나머지 하위 절반만 결과를 `HTML`로 보여주기 위한 뷰 영역이다. 회원 목록의 경우에도 마찬가지다.
- 코드를 잘 보면, `JAVA` 코드, 데이터를 조회하는 리포지토리 등등 다양한 코드가 모두 `JSP`에 노출되어 있다. 
- `JSP`가 너무 많은 역할을 한다.

<br>

## __`MVC` 패턴의 등장__
- 비즈니스 로직은 서블릿 처럼 다른곳에서 처리하고, `JSP`는 목적에 맞게 `HTML`로 화면(View)을 그리는 일에 집중하도록 하자. 과거 개발자들도 모두 비슷한 고민이 있었고, 그래서 MVC 패턴이 등장했다.

<br><br>

# 4. `MVC` 패턴 - 개요

<br>

## __너무 많은 역할__
- 하나의 서블릿이나 `JSP` 만으로 비지니스 로직과 뷰 렌더링까지 모두 처리하면 너무 많은 역할을 하게 되고 , 유지보수가 어려워진다.

<br>

## __변경의 라이프 사이클__
- `UI`의 일부를 수정하는 일과 비지니스 로직을 수정하는 일은 각각 다르게 발생할 확률이 높고 서로에게 영향을 주지 않는다. 따라서 변경 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수에 좋지 않다.

<br>

## __기능 특화__
- `JSP`같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어있기 때문에 해당 업무만 하는 것이 효과적이다.

<br>

## __`Model View Controller`__
- `MVC패턴`은 하나의 서블릿이나 `JSP`로 처리하던 컨트롤러와 뷰의 영역을 나눈것이다.
- 웹 애플리케이션은 보통 `MVC패턴`을 사용한다.

<br>

- __`컨트롤러`__ : `HTTP`요청을 받아서 파라미터를 검증, 비지니르 로직을 수행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델이 담는다.
- __`모델`__ : 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담는다. 덕분에 뷰는 화면을 렌더링하는 일에 집중할 수 있다.
- __`뷰`__ : 모델에 담겨있는 데이터를 사용해서 화면을 그린다. 

<br>

### 참고
- 컨트롤러에 비지니스 로직을 두면 컨트롤러가 너무 많은 역할을 수행한다. 그래서 별도의 서비스 계층을 만들어 비지니스 로직을 수행하고 컨트롤러는 해당 비지니스 로직을 호출하는 역할을 한다.

<br><br>

# 5. `MVC` 패턴 - 적용

<br>

- 서블릿을 컨트롤러로 사용하고, JSP를 뷰로 사용해서 MVC 패턴을 적용해보자.
* `Model`은 `HttpServletRequest` 객체를 사용한다. `request`는 내부에 데이터 저장소를 가지고 있는데, `request.setAttribute()` , `request.getAttribute()` 를 사용하면 데이터를 보관하고, 조회할 수 있다.

<br>

## __회원 등록__

### [**회원 등록 폼 - 컨트롤러**](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/web/servletmvc/MvcMemberFormServlet.java)
- `dispatcher.forward()` : 다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.

<br>
  
### __`/WEB-INF`__   
> 이 경로안에 `JSP`가 있으면 외부에서 직접 `JSP`를 호출할 수 없다. 우리가 기대하는 것은 항상 컨트롤러를 통해서 `JSP`를 호출하는 것이다.

<br>
  
### __`redirect vs forward`__
> 리다이렉트는 실제 클라이언트(웹 브라우저)에 응답이 나갔다가, 클라이언트가 `redirect` 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고, `URL` 경로도 실제로 변경된다. 반면에 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.

<br>
  
### [**회원 등록 폼 - 뷰**](https://github.com/jerry3269/springMVC1/blob/master/src/main/webapp/WEB-INF/views/new-form.jsp)
- 여기서 `form`의 `action`을 보면 절대 경로`( / 로 시작)`가 아니라 상대경로`( / 로 시작X)`인 것을 확인할 수 있다. 이렇게 상대경로를 사용하면 폼 전송시 현재 `URL이 속한 계층 경로` + `save`가 호출된다.
- 현재 계층 경로: `/servlet-mvc/members/`
- 결과: `/servlet-mvc/members/save`

<br>
  
__`실행`__
* `http://localhost:8080/servlet-mvc/members/new-form`
* `HTML Form`이 잘 나오는 것을 확인할 수 있다.

<br>
  
## **회원 저장**

<br>
  
### [**회원 저장 - 컨트롤러**](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/web/servletmvc/MvcMemberSaveServlet.java)
- `HttpServletRequest`를 `Model`로 사용한다.
- `request`가 제공하는 `setAttribute()` 를 사용하면 `request` 객체에 데이터를 보관해서 뷰에 전달할 수 있다.
- 뷰는 `request.getAttribute()` 를 사용해서 데이터를 꺼내면 된다.

<br>
  
### [**회원 저장 - 뷰**](https://github.com/jerry3269/springMVC1/blob/master/src/main/webapp/WEB-INF/views/save-result.jsp)
- `<%= request.getAttribute("member")%>` 로 모델에 저장한 `member` 객체를 꺼낼 수 있지만, 너무 복잡해진다.
- `JSP`는 `${} 문법`을 제공하는데, 이 문법을 사용하면 `request`의 `attribute`에 담긴 데이터를 편리하게 조회할 수 있다.

<br>
  
__`실행`__
- `http://localhost:8080/servlet-mvc/members/new-form`
- `HTML Form`에 데이터를 입력하고 전송을 누르면 저장 결과를 확인할 수 있다.

<br>
  
## **회원 목록 조회**
  
<br>
  
### [**회원 목록 조회 - 컨트롤러**](https://github.com/jerry3269/springMVC1/blob/master/src/main/java/hello/servlet/web/servletmvc/MvcMemberListServlet.java)
- `request` 객체를 사용해서 `List<Member> members` 를 모델에 보관했다.

<br>

### [**회원 목록 조회 - 뷰**](https://github.com/jerry3269/springMVC1/blob/master/src/main/webapp/WEB-INF/views/members.jsp)
- 모델에 담아둔 `members`를 `JSP`가 제공하는 `taglib기능`을 사용해서 반복하면서 출력했다.
- `members` 리스트에서 `member` 를 순서대로 꺼내서 `item` 변수에 담고, 출력하는 과정을 반복한다.

- `<c:forEach>` 이 기능을 사용하려면 다음과 같이 선언해야 한다.
	- `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`

<br>
  
__`실행`__
- `http://localhost:8080/servlet-mvc/members`
- 저장된 결과 목록을 확인할 수 있다.

<br><br>

# 6. `MVC` 패턴 - 한계

<br>

## **`MVC` 컨트롤러의 단점**

### **포워드 중복**
  
```java
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```
  
- `View`로 이동하는 코드가 항상 중복 호출되어야 한다. 물론 이 부분을 메서드로 공통화해도 되지만, 해당 메서드도 항상 직접 호출해야 한다.

<br>

### **`ViewPath`에 중복**
  
```java
String viewPath = "/WEB-INF/views/new-form.jsp";
```
  
- prefix: `/WEB-INF/views/`
- suffix: `.jsp`
- 그리고 만약 `jsp`가 아닌 `thymeleaf` 같은 다른 뷰로 변경한다면 전체 코드를 다 변경해야 한다.

<br>
  
### **사용하지 않는 코드**
  
```java
HttpServletRequest request, HttpServletResponse response
```
  
- `response`는 현재 코드에서 사용되지 않는다.
- 그리고 이런 `HttpServletRequest` , `HttpServletResponse` 를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.

<br>
  
__`공통 처리가 어렵다.`__
>기능이 복잡해질 수 록 컨트롤러에서 공통으로 처리해야 하는 부분이 점점 더 많이 증가할 것이다. 단순히 공통 기능을 메서드로 뽑으면 될 것 같지만, 결과적으로 해당 메서드를 항상 호출해야 하고, 실수로 호출하지 않으면 문제가 될 것이다. 그리고 호출하는 것 자체도 중복이다.

<br>

__`정리하면 공통 처리가 어렵다는 문제가 있다.`__
>이 문제를 해결하려면 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다. 소위 수문장 역할을 하는 기능이 필요하다. `프론트 컨트롤러(Front Controller)` 패턴을 도입하면 이런 문제를 깔끔하게 해결할 수 있다. `스프링 MVC`의 핵심도 바로 이 프론트 컨트롤러에 있다.
