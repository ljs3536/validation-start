# /24-10-15

## 검증 요구사항
컨트롤러의 중요한 역할 중 하나는 HTTP 요청이 정상인지 검증하는 것이다.

### 참고 : 클라이언트 검증, 서버 검증
- 클라이언트 검증은 조작할 수 있으므로 보안에 취약하다.
- 서버만으로 검증하면, 즉각적인 고객 사용성이 부족해진다.
- 둘을 적절히 섞어서 사용하되, 최종적으로 서버 검증은 필수
- API 방식을 사용하면 API 스펙을 잘 정의해서 검증 오류를 API 응답 결과에 잘 남겨주어야함


## 프로젝트 설정 V1

# /24-10-18

## 검증 직접 처리 - 소개

## 검증 직접 처리 - 개발

# /24-10-19

### 참고 Safe Navigation Opearator
만약 여기에서 errors가 null이라면 어떻게 될까?
생각해보면 등록폼에 진입한 시점에는 errors가 없다.
따라서 errors.containsKey()를 호출하는 순간 NullPointerException이 발생한다.

errors?. 은 errors가 null일 때 NullPointerException이 발생하는 대신, null을 반환하는 문법이다.
th:if문에서 null은 실패로 처리되므로 오류 메시지가 출력되지 않는다.

### 정리
- 만약 검증 오류가 발생하면 입력 폼을 다시 보여준다.
- 검증 오류들을 고객에게 친절하게 안내해서 다시 입력할 수 있게 한다.
- 검증 오류가 발생해도 고객이 입력한 데이터가 유지된다.

### 남은 문제점
- 뷰 템플릿에서 중복 처리가 많다. 뭔가 비슷하다.
- 타입 오류 처리가 안된다. Item의 price, quantity같은 숫자 필드는 타입이 Integer 이므로 문자 타입으로 설정하는 것이 불가능하다.
  그런데 이ㅓㄹ한 오류는 스프링 MVC에서 컨트롤러에 진입하기도 전에 예외가 발생하기 때문에, 컨트롤러가 호출되지도 않고, 400 예외가 발생하면서 오류 페이지를 띄운다.
- Item의 price에 문자를 입력하는 것 처럼 타입 오류가 발생해도 고객이 입력한 문자를 화면에 남겨야 한다.
  만약 컨트롤러가 호출된다고 가정해도 Item의 price는 Integer이므로 문자를 보관할 수가 없다.
  결국 문자는 바인딩이 불가능하므로 고객이 입력한 문자가 사라지게 되고, 고객은 본인이 어떤 내용을 입력해서 오류가 발생했는지 이해하기 어렵다.
- 결국 고객이 입력한 값도 어딘가에 별도로 관리가 되어야 한다.

# /24-10-21

## 프로젝트 준비 V2

## BindingResult1

### 주의
BindingResult bindingResult 파라미터 위치는 @ModelAttribute Item item 다음에 와야한다.

### 필드 오류 - FieldError
bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
필드에 오류가 있으면 FieldError 객체를 생성해서 bindingResult에 담아두면 된다.
- objectName : @ModelAttribute 이름
- field : 오류가 발생한 필드 이름
- defaultMessage : 오류 기본 메시지

### 글로벌 오류 - ObjectError
bindingresult.addError(new ObjectError("item","item","가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
특정 필드를 넘어서는 오류가 있으면 ObjectError 객체를 생성해서 bindingResult에 담아두면 된다.
- objectName : @ModelAttribute의 이름
- defaultMessage : 오류 기본 메시지

### 타임리프 스프링 검증 오류 통합 기능
타임리프는 스프링의 BindingResult를 활용해서 편리하게 검증 오류를 표현하는 기능을 제공한다.
- #fields : #fields로 BindingResult가 제공하는 검증 오류에 접근할 수 있다.
- th:errors : 해당 필드에 오류가 있는 경우에 태그를 출력한다. th:if의 편의 버전이다.
- th:errorclass : th:field에서 지정한 필드에 오류가 있으면 class 정보를 추가한다.

# /24-10-22

## BindingResult2
- 스프링이 제공하는 검증 오류를 보관하는 객체이다. 검증오류가 발생하면 여기에 보관하면 된다.
- BindingResult가 있으면 @ModelAttribute에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다.

### 예) @ModelAttribute에 바인딩 시 타입오류가 발생하면?
- BindingResult가 없으면 -> 400 오류가 발생하면서 컨트롤러가 호출되지 않고, 오류 페이지로 이동한다.
- BindingResult가 있으면 -> 오류정보 (FieldError)를 BindingResult에 담아서 컨트롤러를 정상 호출한다.

### BindingResult에 검증 오류를 적용하는 3가지 방법
- @ModelAttribute의 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 FieldError 생성해서 BindingResult에 넣어준다.
- 개발자가 직접 넣어준다.
- Validator사용

### 타입 오류 확인
숫자가 입력되어야 할 곳에 문자를 입력해서 타입을 다르게 해서 BindingResult를 호출하고 bindingResult의 값을 확인해보자

### 주의
- BindingResult는 검증할 대상 바로 다음에 와야한다. 순서가 중요하다. 예를 들어서 @ModelAttribute Item item, 바로 다음에 BindingResult가 와야한다.
- Bindingresult는 Model에 자동으로 포함된다.

### BindingResult와 Errors
BindingResult는 인터페이스이고, Errors 인터페이스를 상속받고 있다.
실제 넘어오는 구현체는 BeanPropertyBindingResult라는 것인데, 둘다 구현하고 있으므로 BindingResult 대신에 Errors를 사용해도 된다.
Errors 인터페이스는 단순한 오류 저장과 조회 기능을 제공한다. 
BindingResult는 여기에 더해서 추가적인 기능들을 제공한다. 
addError()도 BindingResult가 제공하므로 여기서는 BindingResult를 사용하자

### 정리
BindingResult, FieldError, ObjectError를 사용해서 오류 메시지를 처리하는 방법을 알아보았다.
그런데 오류가 발생하는 경우 고객이 입력한 내용이 모두 사라진다. 이문제를 해결해보자.

