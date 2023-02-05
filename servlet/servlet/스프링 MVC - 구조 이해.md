# 스프링 MVC - 구조 이해

<br>

# __1. 스프링 MVC 전체 구조__

<br>

## __직접 만든 프레임워크 스프링 MVC 비교__
- FrontController -> DispatcherServlet
- handlerMappingMap -> HandlerMapping
- MyHandlerAdapter -> HandlerAdapter
- ModelView -> ModelAndView
- viewResolver -> ViewResolver
- MyView -> View 

<br>

## __DispatcherServlet 구조__
- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현
- 프론트 컬르롤러가 디스패처 서블릿 
- 디스패처 서블릿도 부모 클래스에서 `HttpServlet`을 상속받아 서블릿으로 동작
- 스프링 부트는 디스패처 서블릿을 자동으로 등록하면서 `모든경로(urlPatterns="/")`에 대해 매핑
	- *참고* : 더 자세한 경로가 우선순위가 높음. 
	- 따라서 기존에 등록한 서블릿도 동작
- 요청흐름
	- 서블릿이 호출되면 `HttpServlet`이 제공하는 `service()` 호출
	- 스프링 MVC는 디스패처 서블릿의 부모인 `FrameworkServlet`에서 `service()`를 오버라이드
	- `service()`를 시작으로 여러 메서드가 호출되면서 `DispacherServlet.doDispatch()`가 호출

<br>

### `DispacherServlet.doDispatch()`

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse  response) throws Exception {

	HttpServletRequest processedRequest = request;
	HandlerExecutionChain mappedHandler = null;
	ModelAndView mv = null;

	// 1. 핸들러 조회
	mappedHandler = getHandler(processedRequest);
	if (mappedHandler == null) {
		noHandlerFound(processedRequest, response);
		return;
	}

	// 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

	// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());


	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}


private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
	// 뷰 렌더링 호출
	render(mv, request, response);
}

protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
	
	View view;
	String viewName = mv.getViewName();

	// 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
	view = resolveViewName(viewName, mv.getModelInternal(), locale, request);

	// 8. 뷰 렌더링
	view.render(mv.getModelInternal(), request, response);
}
```

<br>

## __SpringMVC 구조__

### __동작 순서__
    1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
    2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
    3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
    4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
    5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
    6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다. JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.
    7. View 반환: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다. JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
    8. 뷰 렌더링: 뷰를 통해서 뷰를 렌더링 한다.

<br>

- 스프링 MVC의 큰 장점은 디스패처 서블릿의 코드 변경없이, 원하는 기능을 변경하거나 확장 할 수 있다는 점이다. 
- 스프링은 또한, 뷰 리졸버와 뷰 도 인터페이스로 제공한다.
- 인터페이스들을 구현하여 디스패처 서블릿에 등록하기만 하면 나만의 컨트롤러를 만드는 것도 가능

<br><br>

# __2. 핸들러 매핑과 핸들러 어댑터__

<br>

## __Controller 인터페이스__

__과거 버전 스프링 컨트롤러__

```java
public interface Controller {

	ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;

}
```

> __참고__   <br> `Controller` 인터페이스는 `@Controller` 어노테이션과는 전혀 다름

<br>

### __OldController__

```java
package hello.servlet.web.springmvc.old;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
 
	@Override
 	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
 		
		System.out.println("OldController.handleRequest");
 		return null;
 	}
}
```

- `@Component` : 이 컨트롤러는 `/springmvc/old-controller`이라는 이름의 스프링 빈으로 등록됨
- 빈의 이름으로 URL 매핑

<br>

### __실행__
- http://localhost:8080/springmvc/old-controller
- 콘솔에 OldController.handleRequest 이 출력되면 성공이다.

<br>

이 컨트롤러가 호출되려면 다음 2가지가 필요하다.

- HandlerMapping(핸들러 매핑)
	- 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 함
	- 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸드러 매핑이 필요

- HandlerAdapter(핸들러 어댑터)
	- 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터 필요
	- 예) `Controller`인터페이스를 실행 할 수 있는 핸들러 어댑터를 찾고 실행

<br>

## __스프링 부트가 자동으로 등록하는 핸들러 매핑과 핸들러 어댑터__

### __HandlerMapping__
```python
0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
....
```

<br>

### HandlerAdapter
```python
0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용)  처리
....
```

<br>

핸들러 매핑도, 핸들러 어댑터도 모두 순서대로 찾음. 만약 없으면 다음 순서로 넘어감.

<br>

## OldController 실행 매커니즘

### __1. 핸들러 매핑으로 핸들러 조회__
1. `HandlerMapping을 순서대로 실행하여 핸들러를 찾음
2. `RequestMappingHandlerMapping`은 실행 실패
3. 빈 이름으로 핸들러를 찾아야 하기 때문에 `BeanNameUrlHandlerMapping`이 실행 성공하고 핸들러인 `OldController`를 반환.

<br>

### __2. 해들러 어댑터 조회__
1. `HandlerAdapter`의 `supports()`를 순서대로 호출
2. `SimpleControllerHandlerAdapter` 가 `Controller` 인터페이스를 지원하므로 대상이 된다.

<br>

### __3. 핸들러 어댑터 실행__
1. 디스패처 서블릿이 조회한 `SimpleControllerHandlerAdapter`을 실행하며 핸들러 정보도 함께 넘김
2. 어댑터는 핸들러인 `OldController`를 내부에서 실행, 결과를 반환.

<br>

### __정리 - OldController 핸들러매핑, 어댑터__
- `OldController` 를 실행하면서 사용된 객체는 다음과 같다.

- `HandlerMapping = BeanNameUrlHandlerMapping`
- `HandlerAdapter = SimpleControllerHandlerAdapter`

<br><br>

# __3. 뷰 리졸버__

<br>

스프링 부트는 `InternalResourceViewResolver`라는 뷰 리졸버를 자동으로 등록하는데 이때, `application.properties`에 등록한 `spring.mvc.view.priefix`, `spring.mvc.view.suffix`설정정보를 사용하여 등록한다.

<br>

form과 save, list JSP 모두 `/WEB-INF/views/`안에 있으므로 다음 코드를 `application.properties`에 추가하자

```java
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

<br>

### __실행__
- http://localhost:8080/springmvc/old-controller
- 등록 폼이 정상 출력되는 것을 확인할 수 있다. 물론 저장 기능을 개발하지 않았으므로 폼만 출력되고, 더 진행하면 오류가 발생한다.

<br>

## __뷰 리졸버 동작 방식__

__스프링 부트가 자동 등록하는 뷰 리졸버__

```python
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성
기능에 사용)
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
....
```

<br>

### __1. 핸들러 어댑터 호출__
- 핸들러 어댑터를 통해 `new-form` 이라는 논리 뷰 이름을 획득한다.

<br>

2. ViewResolver 호출
- `new-form` 이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
- `BeanNameViewResolver` 는 `new-form` 이라는 이름의 스프링 빈으로 등록된 뷰를 찾아야 하는데 없다.
- `InternalResourceViewResolver` 가 호출된다.

<br>

3. InternalResourceViewResolver
- 이 뷰 리졸버는 `InternalResourceView` 를 반환한다.

<br>

4. 뷰 - InternalResourceView
- `InternalResourceView` 는 JSP처럼 포워드 `forward()` 를 호출해서 처리할 수 있는 경우에 사용한다.

<br>

5. view.render()
- `view.render()` 가 호출되고 `InternalResourceView` 는 `forward()` 를 사용해서 JSP를 실행한다.


<br><br>

# __4. 스프링 MVC - 시작하기__

<br>

## [__SpringMemberFormControllerV1 - 회원 등록 폼__](https://github.com/jerry3269/springMVC1/blob/master/servlet/servlet/src/main/java/hello/servlet/web/springmvc/v1/SpringMemberFormControllerV1.java)

- @Controller : 
	- 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component 애노테이션이 있어서 컴포넌트 스캔의 대상이 됨)
	- 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다. 
	- 즉 `RequestMappingHandlerMapping` 핸들러와 `RequestMappingHandlerAdapter` 어댑터가 매핑된다.
- @RequestMapping : 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다. 애노테이션을 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지으면 된다.
- ModelAndView : 모델과 뷰 정보를 담아서 반환하면 된다.

<br>

## [__SpringMemberSaveControllerV1 - 회원 저장__](https://github.com/jerry3269/springMVC1/blob/master/servlet/servlet/src/main/java/hello/servlet/web/springmvc/v1/SpringMemberSaveControllerV1.java)

- mv.addObject("member", member)
	- 스프링이 제공하는 ModelAndView 를 통해 Model 데이터를 추가할 때는 addObject() 를 사용하면 된다. 이 데이터는 이후 뷰를 렌더링 할 때 사용된다.

<br>

## [__SpringMemberListControllerV1 - 회원 목록__](https://github.com/jerry3269/springMVC1/blob/master/servlet/servlet/src/main/java/hello/servlet/web/springmvc/v1/SpringMemberListControllerV1.java)

### __실행__
- 등록: http://localhost:8080/springmvc/v1/members/new-form
- 목록: http://localhost:8080/springmvc/v1/members

<br><br>

# [__5. 스프링 MVC - 컨트롤러 통합__](https://github.com/jerry3269/springMVC1/blob/master/servlet/servlet/src/main/java/hello/servlet/web/springmvc/v2/SpringMemberControllerV2.java)

<br>

`@RequestMapping` 을 잘 보면 클래스 단위가 아니라 메서드 단위에 적용된 것을 확인할 수 있다. 따라서 컨트롤러 클래스를 유연하게 하나로 통합할 수 있다.

<br>

- 클래스 레벨에 다음과 같이 @RequestMapping 을 두면 메서드 레벨과 조합이 된다

```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {}
```

## __조합 결과__

- 클래스 레벨 `@RequestMapping("/springmvc/v2/members")`
	- 메서드 레벨 `@RequestMapping("/new-form")` -> `/springmvc/v2/members/new-form`
	- 메서드 레벨 `@RequestMapping("/save")` -> `/springmvc/v2/members/save`
	- 메서드 레벨 `@RequestMapping` -> `/springmvc/v2/members`

<br><br>

# __6. 스프링 MVC - 실용적인 방식__

<br>

MVC 프레임워크 만들기에서 v3은 ModelView를 개발자가 직접 생성해서 반환했기 때문에, 불편했던 기억이 날 것이다. 물론 v4를 만들면서 프론트 컨트롤러에서 ModelView를 생성하여 컨트롤러에 넘겨 줌으로써 실용적으로 개선한 기억도 날 것이다.

<br>

```java
package hello.servlet.web.springmvc.v3;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model
    ) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();


        model.addAttribute("members", members);
        return "members";
    }
}
```

<br>

### __Model 파라미터__
- `save()`, `members()` 를 보면 Model을 파라미터로 받는것을 확인 할 수 있다.
- 즉 스프링도 프론트 컨트롤러 쪽에서 모델이 미리 생성하여 컨트롤러에 전달하는 것을 알 수 있다.
- 이렇게 하면 컨트롤러는 `ModelAndView`객체를 반환하는게 아니라 `ViewName` String을 반환 할 수 있게 된다.
- 여기서 어노테이션 기반 컨트롤러는 `ModelAndView`객체와 `String`을 반환 할 수 있는데 스프링은 어노테이셔 기반 컨트롤러를 굉장히 유연하게 설계하여서 `String`이 들어온 경우에도 그것을 `viewName`으로 인지하여 사용한다.

<br>

### __ViewName 직접 반환__
뷰의 논리 이름을 반환할 수 있다.

<br>

### __@RequestParam 사용__
- 스프링은 HTTP 요청 파라미터를 `@RequestParam` 으로 받을 수 있다.
- `@RequestParam("username")` 은 `request.getParameter("username")` 와 거의 같은 코드라 생각하면 된다.
- 물론 `GET` 쿼리 파라미터, `POST` Form 방식을 모두 지원한다.

<br>

### __@RequestMapping @GetMapping, @PostMapping__
- `@RequestMapping` 은 URL만 매칭하는 것이 아니라, HTTP `Method`도 함께 구분할 수 있다.
- `@RequestMapping(value = "/new-form", method = RequestMethod.GET)`

- 이것을 `@GetMapping` , `@PostMapping` 으로 더 편리하게 사용할 수 있다.
- 참고로 Get, Post, Put, Delete, Patch 모두 애노테이션이 준비되어 있다.















