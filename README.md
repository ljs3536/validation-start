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

# /24-10-24

## FieldError, ObjectError

### FieldError생성자 - FieldError는 두 가지 생성자를 제공한다.
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure, @Nullable String[] codes, 
@Nullable Object[] arguments, @Nullable String defaultMessage)

#### 파라미터 목록
- objectName : 오류가 발생한 객체이름
- field : 오류 필드
- rejectedValue : 사용자가 입력한 값(거절된 값)
- bindingFailure : 타입 오류같은 바인딩 실패인지, 검증 실패인지 구분 값
- codes : 메시지 코드
- arguments : 메시지에서 사용하는 인자
- defaultMessage : 기본 오류 메시지

#### 오류 발생 시 사용자 입력값 유지
사용자의 입력 데이터가 컨트롤러의 @ModelAttribute에 바인딩되는 시점에 오류가 발생하면 모델 객체에 사용자 입력값을 유지하기 어렵다.
예를 들어서 가격에 숫자가 아닌 문자가 입력된다면 가격은 Integer 타입이므로 문자를 보관할 수 있는 방법이 없다.
그래서 오류가 발생한 경우 사용자 입력 값을 보관하는 별도의 방법이 필요한다.
그리고 이렇게 보관한 사용자 입력 값을 검증 오류 발생 시 화면에 다시 출력하면 된다.
FieldError는 오류 발생시 사용자 입력값을 저장하는 기능을 제공한다.

#### 타임리프의 사용자 입력값 유지
th:field="*{price}"
타임리프의 th:field는 매우 똑똑하게 동작하는데, 정상 상황에는 모델 객체의 값을 사용하지만, 오류가 발생하면 FieldError에서 보관한 값을 사용해서 값을 출력한다.

#### 스프링의 바인딩 오류 처리
타입 오류로 바인딩에 실패하면 스프링은 FieldError를 생성하면서 사용자가 입력한 값을 넣어둔다.
그리고 해당 오류를 BindingResult에 담아서 컨트롤러를 호출한다.
따라서 타입 오류같은 바인딩 실패 시에도 사용자의 오류 메시지를 정상 출력할 수 있다.

# /24-10-26

## 오류코드와 메시지 처리 1

### errors 메시지 파일 생성

# /24-10-29

## 오류코드와 메시지 처리 2

### 컨트롤러에서 BindingResult는 검증해야 할 객체인 target 바로 다음에 온다. 따라서 BindingResult는 이미 본인이 검증해야 할 객체인 target을 알고 있다.

### rejectValue(), reject()
BindingResult가 제공하는 rejectValue(), reject()를 사용하면 FieldError, ObjectError를 직접 생성하지 않고 깔끔하게 검증 오류를 다룰 수 있다.

### 축약된 오류 코드
FieldError() 를 직접 다룰 때는 오류 코드를 range.item.price 와 같이 모두 입력했다. 그런데 rejectValue() 를 사용하고 부터는 오류 코드를 range 로 간단하게 입력했다. 그래도 오류 메시지를 잘 찾아서 출
력한다. 무언가 규칙이 있는 것 처럼 보인다. 이 부분을 이해하려면 MessageCodesResolver 를 이해해야 한다

# /24-10-30

## 오류코드와 메시지 처리 3
단순하게 만들면 범용성이 좋아서 여러곳에서 사용할 수 있지만, 메시지를 세밀하게 작성하기 어렵다. 반대로 너무 자세하게 만들면 범용성이 떨어진다.
가장 좋은 방법은 범용성으로 사용하다가, 세밀하게 작성해야 하는 경우에는 세밀한 내용이 적용되도록 메시지에 단계를 두는 방법이다.

required라고 오류 코드를 사용한다고 가정한다면
오류 메시지에 required.item.itemName과 같이 객체명과 필드명을 조합한 세밀한 메시지 코드가 있으면 이 메시지를 높은 우선순위로 사용한다.

물론 이렇게 객체명과 필드명을 조합한 메시지가 있는지 우선 확인하고, 없으면 좀 더 범용적인 메시지를 선택하도록 추가 개발을 해야겠지만,
범용성 있게 잘 개발해두면, 메시지의 추가 만으로 매우 편리하게 오류 메시지를 관리할 수 있을 것이다.
스프링은 MessageCodesResolver라는 것으로 이러한 기능을 지원한다.

# /24-10-31

## MessageCodesResolver

# /24-11-04

## 오류코드와 메시지 처리 5
### 핵심은 구체적인 것에서 덜 구체적인것으로
MessageCodesResolver는 required.item.itemName처럼 구체적인 것을 먼저 만들어주고, required처럼 덜 구체적인 것을 가장 나중에 만든다.
이렇게 하면 앞서 말한 것처럼 메시지와 관련된 공통 전략을 편리하게 도입할 수 있다.

### 왜 이렇게 사용하는가?
모든 오류 코드에 대해서 메시지를 각각 다 정의하면 개발자 입장에서 관리하기 너무 힘들다.
크게 중요하지 않은 메시지는 범용성 있는 required 같은 메시지로 끝내고, 정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적이다.

# /24-11-05

## 오류코드와 메시지 처리 6

### 스프링이 직접 만든 오류 메시지 처리

검증오류는 2가지로 나눌 수 있다.
- 개발자가 직접 설정한 오류 코드 -> rejectValue()를 직접 호출
- 스프링이 직접 검증 오류에 추가한 경우(주로 타입 정보가 맞지 않음)

스프링은 타입 오류가 발생하면 typeMismatch라는 오류 코드를 사용한다.
이 오류코드가 MessageCodesResolver를 통하면서 4가지 메시지 코드가 생성된다.

# /24-11-06

## Validator 분리1

