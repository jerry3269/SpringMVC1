# 스프링 MVC - 웹 페이지 만들기

<br>

# 1. 요구사항 분석

- 상품 도메인 모델
	- 상품 ID
	- 상품명
	- 가격
	- 수량
- 상품 관리 기능
	- 상품 목록
	- 상품 상세
	- 상품 등록
	- 상품 수정

<br>

## 프론트엔드 개발자가 만든 html 폼
- 상품목록(등록, 상세 버튼)
- 상품등록 폼(저장, 취소 버튼)
- 상품 상세(수정, 목록 버튼)
- 상품 수정 폼(저장, 취소 버튼)

```
1) 클라이언트가 상품 목록 요청
2) `상품목록 controller`에서 `상품목록 폼` 랜더링
	2-1) 상품등록 버튼 클릭 -> `상품등록 controller`에서 `상품등록 폼` 랜더링
		3-1) 상품저장 버튼 클릭 -> `상품저장 controller`에서 `상품상세 폼` 랜더링
		3-2) 취소 버튼 클릭 -> `상품등록 폼`에서 `상품목록 링크`로 이동
	2-2) 상품상세 버튼 클릭 -> `상품상세 controller`에서 `상품상세 폼` 랜더링
		3-1) 수정 버튼 클릭 -> `상품수정 controller`에서 `상품수정 폼` 랜더링
			4-1) 저장 버튼 클릭 -> `상품상세 controller` 링크로 `redirect`
			4-1) 취소 버튼 클릭 -> `상품수정 폼`에서 `상품상세 주소`로 이동
		3-2) 취소 버튼 클릭 -> `상품수정 폼`에서 `상품목록 링크`로 이동
```

<br>

> __참고__ <br>
부트 스트랩은 웹사이트를 쉽게 만들 수 있게 도와주는 HTML, CSS, JS 프레임워이다. <br>
하나의 CSS로 휴대폰, 태블릿, 데스크탑까지 다양한 기기에서 작동한다. 다양한 기능을 제공하여 사용자가 쉽게 웹사이트를 제작, 유지 ,보수 할 수 있도록 도와준다. -출처: 위키백과

<br>

## RequiredArgsConstructor

- final 이 붙은 멤버변수만 사용해서 생성자를 자동으로 만들어준다. 
- 생성자가 하나라면 @Autowired로 의존관계 주입해줌

<br>

## PostContruct

- 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출(즉 고객이 요청하기 전에 호출됨)

<br>

# 2. 타임리프 알아보기

<br>

## 타임리프 사용 선언

`<html xmlns:th="http://www.thymeleaf.org">`

<br>

## 속성 변경 - th:href
- href="value1" 을 th:href="value2" 의 값으로 변경한다.
- 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 만약 값이 없다면 새로 생성한다.
- HTML을 그대로 볼 때는 href 속성이 사용되고, 뷰 템플릿을 거치면 th:href 의 값이 href 로 대체되면서 동적으로 변경할 수 있다.

<br>

## 타임리프 핵심
- 핵심은 th:xxx 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다. `th:xxx` 이 없으면 기존 html의 xxx 속성이 그대로 사용된다.
- HTML을 파일로 직접 열었을 때, `th:xxx` 가 있어도 웹 브라우저는 `th:` 속성을 알지 못하므로 무시한다.
- 따라서 HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.

<br>

## URL 링크 표현식 - @{...},
- 타임리프는 URL 링크를 사용하는 경우 @{...} 사용
- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다.

<br>

## 리터럴 대체 - |...|
- 타임리프에서 문자와 표현식 등은 분리되어 있기 때문에 더해서 사용해야 한다.
	- `<span th:text="'Welcome to our application, ' + ${user.name} + '!'">`
- 다음과 같이 리터럴 대체 문법을 사용하면, 더하기 없이 편리하게 사용할 수 있다.
	- `<span th:text="|Welcome to our application, ${user.name}!|">`

<br>

## 반복 출력 - th:each
- `<tr th:each="item : ${items}">`
- 반복은 th:each 를 사용한다. 이렇게 하면 모델에 포함된 items 컬렉션 데이터가 item 변수에 하나씩
포함되고, 반복문 안에서 item 변수를 사용할 수 있다.
- 컬렉션의 수 만큼 `<tr>..</tr>` 이 하위 테그를 포함해서 생성된다.

<br>

## 변수 표현식 - ${...}
- `<td th:text="${item.price}">10000</td>`
- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다.
- 프로퍼티 접근법을 사용한다. ( `item.getPrice()` )

<br>

## 내용 변경 - th:text
- `<td th:text="${item.price}">10000</td>`
- 내용의 값을 `th:text` 의 값으로 변경한다.
- 여기서는 `10000`을 `${item.price}` 의 값으로 변경한다.

<br>

## URL 링크 표현식2 - @{...},
- `th:href="@{/basic/items/{itemId}(itemId=${item.id})}"`
- URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다.
- 경로 변수( `{itemId}` ) 뿐만 아니라 쿼리 파라미터도 생성한다.
- 예) `th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}"`
	- 생성 링크: `http://localhost:8080/basic/items/1?query=test`

<br>

## URL 링크 간단히
- `th:href="@{|/basic/items/${item.id}|}"`
- 상품 이름을 선택하는 링크를 확인해보자.
- 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.

<br>

## 속성 변경 - th:value
- `th:value="${item.id}"`
- 모델에 있는 item 정보를 획득하고 프로퍼티 접근법으로 출력한다. ( `item.getId()` )
- `value` 속성을 `th:value` 속성으로 변경한다.

<br>

## 속성 변경 - th:action
- `th:action`
- HTML form에서 action 에 값이 없으면 현재 URL에 데이터를 전송한다.
- 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분한다.
	- 상품 등록 폼: GET `/basic/items/add`
	- 상품 등록 처리: POST `/basic/items/add`
- 이렇게 하면 하나의 URL로 등록 폼과, 등록 처리를 깔끔하게 처리할 수 있다.

<br>

## th:if 
해당 조건이 참이면 실행

<br>

## ${param.status} 
- 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능
- 원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야 한다. 그런데 쿼리 파라미터는 자주 사용해서
타임리프에서 직접 지원한다.

<br>

### 참고
> 타임리프는 순수 HTML 파일을 웹 브라우저에서 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을
거치면 동적으로 변경된 결과를 확인할 수 있다. JSP를 생각해보면, JSP 파일은 웹 브라우저에서 그냥 열면
JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야
한다. 이렇게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 네츄럴 템플릿이라 한다.

<br><br>

# 3. 상품 등록 처리 - @ModelAttribute

<br>

## @ModelAttribute - 요청 파라미터 처리
@ModelAttribute 는 지정된 객체를 생성하고, 요청 파라미터의 값을 프로퍼티 접근법(setXxx)으로 입력해준다.

<br>

## @ModelAttribute - Model 추가
- @ModelAttribute 는 모델(Model)에 @ModelAttribute 로지정한 객체를 자동으로 넣어준다.
- @ModelAttribute 의 이름을 생략하면 모델에 저장될 때 클래스명을 사용한다. 이때 클래스의 첫글자만 소문자로 변경해서 등록한다.
- @ModelAttribute 생략 가능

<br>

- `@ModelAttribute("hello") Item item` -> 이름을 hello 로 지정 -> 모델에 hello 이름으로 item 저장
- `@ModelAttribute Item item` -> 모델에 item이름으로 item 저장
- `Item item` -> 모델에 item이름으로 item 저장

<br>

# 4. 리다이렉트
상품 수정은 마지막에 뷰 템플릿을 호출하는 대신에 상품 상세 화면으로 이동하도록 리다이렉트를 호출한다.
- 스프링은 `redirect:/...` 으로 편리하게 리다이렉트를 지원한다.
- `redirect:/basic/items/{itemId}`
	- 컨트롤러에 매핑된 `@PathVariable` 의 값은 `redirect` 에도 사용 할 수 있다.
	- `redirect:/basic/items/{itemId}` -> `{itemId}` 는 `@PathVariable Long itemId` 의 값을 그대로 사용한다.

<br>

# 5. PRG (Post/Redirect/Get)

- 상품 등록 처리 컨트롤러에는 심각한 문제가 있음.
- 상품 등록을 완료하고 웹 브라우저를 새로고침 해보면 상품이 계속해서 중복 등록되는 것을 확인 할 수 있음
- 이유는 새로고침시 가장 최근에 보낸 요청을 다시 재요청 하기 때문

<br>

## 문제
상품 등록 폼에서 데이터를 입력하고 저장을 누르면 `POST /add` + 상품데이터를 서버로 전송
새로고침시 다시 전송

<br>

## POST, Redirect GET
- 상품 저장후 뷰 템플릿으로 이동하지 않고 상품 상세 화면으로 리다이렉트 호출하면 문제 해결
- 이후 새로고침을 시도해도 리다이렉트로 상세화면에 대한 get요청이 재시도 

<br>

## RedirectAttributes
- `"redirect:/basic/items/" + item.getId()` 에서 `item.getId()` 변수 처럼 URL에 더해서 사용하는것은 URL 
인코딩이 안되기 때문에 위험.
따라서 RedirectAttributes 사용

<br>

```java
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {

 	Item savedItem = itemRepository.save(item);
 	redirectAttributes.addAttribute("itemId", savedItem.getId());
 	redirectAttributes.addAttribute("status", true);
 	return "redirect:/basic/items/{itemId}";
}
```

<br>

- `redirectAttributes.addAttribute("itemId", savedItem.getId());` : rediect 경로에 itemId 가 있으므로 savedItem.getId() 값으로 치환
- `redirectAttributes.addAttribute("status", true);` : redirect 경로에 statuc가 없으므로 쿼리 파라미터로 전송

<br>

- 실행해보면 다음과 같은 리다이렉트 결과가 나온다.
- `http://localhost:8080/basic/items/3?status=true`

<br>

- `redirect:/basic/items/{itemId}`
	- pathVariable 바인딩: `{itemId}`
	- 나머지는 쿼리 파라미터로 처리: `?status=true`


