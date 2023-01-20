# MVC 프레임워크 만들기

<br>

# __1. 프론트 컨트롤러 패턴 소개__

<br>


## __FrontController 패턴 특징__
- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받는다.
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출해준다.

<br>

### __스프링 웹 MVC와 프론트 컨트롤러__
- 스프링 웹 MVC의 DIspatcherServlet이 FrontController 패턴으로 구성

<br><br>

# __2. 프론트 컨트롤러 도입 - v1__

<br>

## __V1 구조__
- HTTP요청이 오면 프론트 컨트롤러에 URL매핑 정보에서 컨트롤러를 조회한다.
- 조회한 컨트롤러를 호출한다.
- 해당 컨트롤러에서 직접 JSP forward를 수행한다.
- view 렌더링이 수행된다.

<br>

## __프론트 컨트롤러 분석__

### __urlPatterns__
- `urlPatterns = "/front-controller/v1/*"` : `/front-controller/v1` 를 포함한 하위 모든 요청은 이 서블릿에서 받아들인다.
- 예) `/front-controller/v1` , `/front-controller/v1/a` , `/front-controller/v1/a/b`

<br>

### __controllerMap__
- key : 매핑 URL
- value : 호출될 컨트롤러

<br>

### __service()__
- `requestURI`를 조회하여 호출될 컨트롤러를 `controllerMap`에서 찾는다. 만약 없다면 `404 에러`를 반환한다.
- 컨트롤러를 찾고 `controller.process(request, response)`를 호출해서 해당 컨트롤러에서 `JSP forward`를 수행한다.

<br>

### __실행__
- 등록: `http://localhost:8080/front-controller/v1/members/new-form`
- 목록: `http://localhost:8080/front-controller/v1/members`

<br><br>

#  __3. View 분리 - v2__
모든 컨트롤러에서 뷰로 이동하는 부분에 중복이 있다.

```java
String viewPath = "/WEB-INF/views/new-form.jsp";
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response);
```

<br>

## __V2 구조__
- HTTP요청이 오면 FrontController에서 URL매핑 정보에서 컨트롤러를 조회한다.
- 조회된 컨트롤러를 호출하고 컨트롤러는 MyView 객체를 반환한다.
- 반환된 MyView 객체의 함수인 render()를 호출한다.
- MyView 의 render에서 JSP forward로직이 수행된다.

<br><br>

## __MyView__

```java
package hello.servlet.web.frontcontroller;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class MyView {

    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

- ControllerV2의 반환 타입이 `MyView`이므로 프론트 컨트롤러의 호출 결과로 `MyView`를 반환 받는다. 
- 그리고 `view.render()`를 호출하면 `forward` 로직을 수행해서 JSP가 실행된다.
- 프론트 컨트롤러의 도입으로 MyView 객체의 render() 를 호출하는 부분을 모두 일관되게 처리할 수 있다. 각각의 컨트롤러는 MyView 객체를 생성만 해서 반환하면 된다.

<br>

### __실행__
- 등록: `http://localhost:8080/front-controller/v2/members/new-form`
- 목록: `http://localhost:8080/front-controller/v2/members`

<br><br>

# __4. Model 추가 - v3__

<br>

## __서블릿 종속성 제거__
- 컨트롤러 입장에서 HttpServletRequest, HttpServletResponse은 요청 파라미터를 받기위해 쓰인다.
- 요청 파라미터 정보는 자바의 Map으로 대신 넘기면 컨트롤러가 서블릿 기술을 몰라도 동작할 수 있다.
- 그리고 request객체를 통해서 JSP에서 사용가능한 데이터를 저장하여 넘겨주었는데, 별도의 Model객체를 만들어서 반환하여 주도록 설계하면 된다.
- 결과적으로 구현하는 컨트롤러는 서블릿 기술을 사용하지 않아도 된다.

<br>

## __뷰 이름 중복 제거__
- 컨트롤러에서 지정하는 뷰 이름에 중복이 있는것을 알 수 있다.
- 컨트롤러는 __뷰의 논리 이름__을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화
- 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 수정하면 된다.

<br>

- `/WEB-INF/views/new-form.jsp` -> __new-form__
- `/WEB-INF/views/save-result.jsp` -> __save-result__
- `/WEB-INF/views/members.jsp` -> __members__

<br>

## __V3 구조__
- HTTP요청이 오면 프론트 컨트롤러에서 URL매핑 정보를 조회하여 컨트롤러를 조회한다.
- 조회된 컨트롤러를 호출한다.
- 컨트롤러에서 ModelView객체를 반환한다.
- ModelView객체에는 viewName(논리이름),  Map<String, Object> model(JSP에 전달할 데이터)가 있다.
- 반환받은 ModelView에서 viewName을 바탕으로 viewResolver를 호출한다.
- viewResolver는 Myview객체를 생성하고 논리이름에서 제외된 실제 주소를 viewPath로 설정하여 반환한다.
- MyView객체에는 model을 받아서 request에 정보를 저장하는 render() 하나 추가한다.
- 반환된 MyView객체의 render()를 호출하여 JSP로 이동한다.

<br>

### __ModelView__

```java
package hello.servlet.web.frontcontroller;

import java.util.HashMap;
import java.util.Map;


public class ModelView {

    private String viewName;

    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```

- 뷰의 이름과 뷰를 렌더링 하기위해 필요한 `model` 객체를 가지고 있다.
- `HttpServletRequest`가 제공하는 파라미터는 프론트 컨트롤러가 `paramMap`에 담아서 호출해주면 된다.
- 컨트롤러의 응답 결과로 뷰 이름과 뷰에 전달할 `Model` 데이터를 포함하는 `ModelView` 객체를 반환하면 된다.

<br>

### __createParamMap()__
- HttpServletRequest에서 파라미터 정보를 꺼내서 Map으로 변환한다. 그리고 해당 `Map( paramMap )`을 컨트롤러에 전달하면서 호출한다.

<br>

### __뷰 리졸버__

`MyView view = viewResolver(viewName)`
- 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경한다. 그리고 실제 물리 경로가 있는 MyView 객체를 반환한다.
- 논리 뷰 이름: members
- 물리 뷰 경로: /WEB-INF/views/members.jsp

<br>

`view.render(mv.getModel(), request, response)`
- 뷰 객체를 통해서 HTML 화면을 렌더링 한다.
- 뷰 객체의 render() 는 모델 정보도 함께 받는다.
- JSP는 request.getAttribute() 로 데이터를 조회하기 때문에, 모델의 데이터를 꺼내서 request.setAttribute() 로 담아둔다.
- JSP로 포워드 해서 JSP를 렌더링 한다.

<br>

### __실행__
- 등록: `http://localhost:8080/front-controller/v3/members/new-form`
- 목록: `http://localhost:8080/front-controller/v3/members`

<br><br>

# __5. 단순하고 실용적인 컨트롤러 - v4__

<br>

- 앞선 v3컨트롤러는 실제 컨트롤러 인터페이스를 구현하는 개발자 입장에서 보면, 항상 ModelView객체를 생성하여 반환해야 하는 부분이 번거로울 수있다. 

<br>

## __V4 구조__
- 기본적인 구조는 v3와 같다.
- 대신 컨트롤러가 ModelView를 반환하지 않고, viewName만 반환한다.

<br>

### __모델 객체 전달__

`Map<String, Object> model = new HashMap<>(); //추가`

모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다. 컨트롤러에서 모델 객체에 값을 담으면 여기에 그대로 담겨있게 된다.

<br>

### __뷰의 논리 이름을 직접 반환__

```java
String viewName = controller.process(paramMap, model);
MyView view = viewResolver(viewName);
```

컨트롤러가 직접 뷰의 논리 이름을 반환하므로 이 값을 사용하여 실제 물리 뷰를 찾을 수 있다.

<br>

### __실행__
- 등록: `http://localhost:8080/front-controller/v4/members/new-form`
- 목록: `http://localhost:8080/front-controller/v4/members`

<br>

### __정리__
- 이번 버전의 컨트롤러는 매우 단순하고 실용적이다. 기존 구조에서 모델을 파라미터로 넘기고, 뷰의 논리 이름을 반환한다는 작은 아이디어를 적용했을 뿐인데, 컨트롤러를 구현하는 개발자 입장에서 보면 이제 군더더기 없는 코드를 작성할 수 있다.
- 또한 중요한 사실은 여기까지 한번에 온 것이 아니라는 점이다. 프레임워크가 점진적으로 발전하는 과정 속에서 이런 방법도 찾을 수 있었다.

<br><br>

# __6. 유연한 컨트롤러1 - v5__
만약 어떤 개발자는 ControllerV3 방식으로 개발하고 싶고, 어떤 개발자는 ControllerV4 방식으로 개발하고 싶다면 어떻게 해야할까?

<br>

## __어댑터 패턴__
> 지금까지 우리가 개발한 프론트 컨트롤러는 한가지 방식의 컨트롤러 인터페이스만 사용할 수 있다. ControllerV3 , ControllerV4 는 완전히 다른 인터페이스이다. 따라서 호환이 불가능하다. 마치 v3는 110v이고, v4는 220v 전기 콘센트 같은 것이다. 이럴 때 사용하는 것이 바로 어댑터이다. 어댑터 패턴을 사용해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 변경해보자

<br>

## __V5 구조__
- 프론트 컨트롤러에서 핸들러 매핑 정보를 바탕으로 핸들러를 조회한다.
- 핸들러 어댑터 목록에서 핸들러를 처리할 수 있는 핸들러 어댑터를 조회한다.
- 핸들러 어댑터의 handle(handler)를 호출한다. 
- 핸들러 어댑터에서 handler(컨트롤러)를 호출한다.
- 핸들러 어댑터에서 ModelView 객체를 반환한다.
- ModelView객체를 바탕으로 프론트 컨트롤러에서 viewResolver를 호출한다.
- viewResolver에서 실제 물리정보가 들어간 MyView객체를 생성하여 반환받는다.
- 프론트 컨트롤러에서 view.render(model)을 호출하여 JSP로 이동한다.

<br>

- 핸들러 어댑터: 중간에 어댑터 역할을 하는 어댑터가 추가되었는데 이름이 핸들러 어댑터이다. 여기서 어댑터 역할을 해주는 덕분에 다양한 종류의 컨트롤러를 호출할 수 있다.
- 핸들러: 컨트롤러의 이름을 더 넓은 범위인 핸들러로 변경했다. 그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떠한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다.

<br>

## __핸들러 어댑터가 필요한 이유__
- 예를 들어, ControllerV3는 ModelView객체를 반환하고, ControllerV4는 viewName(String)을 반환한다.
- 어댑터가 없으면 프론트 컨트롤러에서 이를 다 ModelView객체로 변환하여 viewResolver를 호출해야 한다.
- 핸들러 어댑터가 ModelView 객체에 정보를 담아서 반환하도록 하여 프론트 컨트롤러가 다양한 컨트롤러에 호환되도록 설계하였다.

<br>

### __MyHandlerAdapter__

```java
package hello.servlet.web.frontcontroller.v5;

import hello.servlet.web.frontcontroller.ModelView;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public interface MyHandlerAdapter {

	boolean supports(Object handler);

 	ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}
```

- boolean supports(Object handler)
	- handler는 컨트롤러를 말한다.
	- 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드다.

<br>

- ModelView handle(HttpServletRequest request, HttpServletResponse response, Object  handler)
	- 어댑터는 실제 컨트롤러를 호출하고, 그 결과로 ModelView를 반환해야 한다.
	- 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 ModelView를 직접 생성해서라도 반환해야 한다.
	- 이전에는 프론트 컨트롤러가 실제 컨트롤러를 호출했지만 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다.


<br>

## __컨트롤러(Controller) -> 핸들러(Handler)__
- 이전에는 컨트롤러를 직접 매핑해서 사용했다. 그런데 이제는 어댑터를 사용하기 때문에, 컨트롤러 뿐만 아니라 어댑터가 지원하기만 하면, 어떤 것이라도 URL에 매핑해서 사용할 수 있다. 그래서 이름을 컨트롤러에서 더 넒은 범위의 핸들러로 변경했다.

<br><br>

# __7. 정리__

- v1: 프론트 컨트롤러를 도입
	- 기존 구조를 최대한 유지하면서 프론트 컨트롤러를 도입
- v2: View 분류
	- 단순 반복 되는 뷰 로직 분리
		- `컨트롤러가 MyView객체 반환, 프론트 컨트롤러가 view.render() 호출`
- v3: Model 추가
	- 서블릿 종속성 제거
	- 뷰 이름 중복 제거
		- `파라미터 paramMap에 담아 컨트롤러 호출, 컨트롤러가 ModelView객체 반환, 뷰 리졸버로 물리주소가 담긴 MyView반환, 프론트 컨트롤러가 view.render(model) 호출`
- v4: 단순하고 실용적인 컨트롤러
	- v3와 거의 비슷
	- 구현 입장에서 ModelView를 직접 생성해서 반환하지 않도록 편리한 인터페이스 제공
		- `프론트 컨트롤러에서 model객체 생성 및 controller.process(paramMap, model) 호출, 컨트롤러에서 model에 값 세팅, 컨트롤러가 viewName(논리이름)만 반환`
- v5: 유연한 컨트롤러
	- 어댑터 도입
	- 어댑터를 추가해서 프레임워크를 유연하고 확장성 있게 설계
		- `어댑터 인터페이스 생성, 어댑터 생성, 프론트 컨트롤러에 List<어댑터인터페이스> 변수 생성 및 생성자에서 어댑터 주입, 조회된 컨트롤러에 맞는 어댑터 조회(instanseof 이용), adapter.handle(request, response, handler)호출, 어댑터에서 handler(컨트롤러)를 조회된 컨트롤러의 인터페이스로 다운캐스팅, 어댑터에서 handler(컨트롤러)으 process() 호출, 컨트롤러는 각 컨트롤러에 맞게 반환타입 반환, 어댑터는 컨트롤러의 반환타입을 바탕으로 ModelView객체 반환`

<br>

여기에 애노테이션을 사용해서 컨트롤러를 더 편리하게 발전시킬 수도 있다. 

만약 애노테이션을 사용해서 컨트롤러를 편리하게 사용할 수 있게 하려면 어떻게 해야할까? 

바로 애노테이션을 지원하는 어댑터를 추가하면 된다!

다형성과 어댑터 덕분에 기존 구조를 유지하면서, 프레임워크의 기능을 확장할 수 있다.















































