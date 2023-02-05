# @PathVariable VS @RequestParam

<br>

## @PathVariable

- `@RequestMapping` 은 `URL` 경로를 템플릿화(동적 처리) 할 수 있는데, `@PathVariable` 을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.
- 예) items/{itemId} 일때 @PathVariable("itemId")을 사용하면 itemId 조회 가능
- 변수 이름이 같을 때 @PathVariable로 생략 가능
- 예) `@PathVariable("itemId") Long id` -> `@PathVariable Long itemId`

<br>

## @RequestParam

- `URL` 경로로 `GET 쿼리 파라미터`(`ex) xxx?username=kim`) 요청이 오거나
`Post`로 `HTML Form`으로 파라미터가 넘어올때 해당 파라미터를 편리하게 조회하기 위해 사용
- 파라미터와 변수 명이 같을 때 `@RequestParam`으로 생략 가능
- 예) `@RequestParam("username") String name` -> `@RequestParam String username`
- `String, int` 등의 단순 타입이면 `@RequestParam` 도 생략 가능
- `@RequestParam String username` -> `String username`


