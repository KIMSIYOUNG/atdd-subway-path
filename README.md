# 우아한 테크코스 - 지하철 경로 조회


## 미션 회고 및 학습 내용
### 미션 포인트 ⏰ 
   - HTTP 캐시 적용하기(ETag)
   - 객체지향 생활체조 준수
   - Inside Out 방식으로 구현하기
   - 외부 API 사용방법 및 학습법 찾기
   - 완성도 있는 어플리케이션 구현하기
### 학습 내용 📖
   - ETag 및 다양한 캐싱 전략을 통해 Server에 부하를 줄일 수 있다.
   - ControllerAdvice를 활용하여 어플전역에서 발생하는 예외를 일관되게 관리할 수 있다.
        - 예외 구조를 설계함으로써 일관성 있는 예외를 사용할 수 있다.
        - 현재 예외 구조(BusinessException, NotFound, Invalid 상속 구조)        
        ```java        
        @RestControllerAdvice
        public class GlobalExceptionHandler {

            @ExceptionHandler(MethodArgumentNotValidException.class)
            protected ResponseEntity<ErrorResponse> invalidInputException(MethodArgumentNotValidException e) {
                ErrorResponse errorResponse = new ErrorResponse(ErrorCode.INVALID_REQUEST, e.getBindingResult());
                return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
            }

            @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
            protected ResponseEntity<ErrorResponse> invalidRequestMethodException(HttpRequestMethodNotSupportedException e) {
                ErrorResponse errorResponse = new ErrorResponse(ErrorCode.INVALID_REQUEST_METHOD);
                return new ResponseEntity<>(errorResponse, HttpStatus.valueOf(ErrorCode.INVALID_REQUEST_METHOD.getStatus()));
            }

            @ExceptionHandler(BusinessException.class)
            protected ResponseEntity<ErrorResponse> invalidBusinessException(BusinessException e) {
                ErrorResponse errorResponse = new ErrorResponse(e.getMessage(), e.getStatus());
                return new ResponseEntity<>(errorResponse, HttpStatus.valueOf(errorResponse.getStatus()));
            }
        }
        ```
   - TDD와 ATDD에 얽매이는 것이 아니라, 아는것에서 모르는 것으로 개발하는 사이클에 익숙해지자.
        - 세부적인 구현사항을 알기 힘들다면 ATDD로 시작하여 협력할 객체를 덩어리 위주로 생각하고 이후에 해당 객체를 TDD로 개발하는 방식
        - 도메인에 대한 이해가 충분하다면 TDD사이클로 시작하는 것도 방법
   - 전략패턴 및 DI를 통해, 특정 기술에 종속적이지 않은 형태로 만들 수 있다.
        - 다엑스트라에 종속적이지 않은 Path Interface 사용
   - 외부 API는 학습 테스트를 통해 기본적인 사용법을 학습할 수 있다.
        - 사용법 학습의 목적
        - 테스트를 통해, API가 변경되더라도 바로 찾을 수 있다.
   - 외부 API의 동작원리를 파악하면 충분히 커스터마이징해서 사용할 수 있다.(Weight를 커스터마이징)
   ```java
   public class CustomEdge extends DefaultWeightedEdge {
        private final LineStation lineStation;
        private CriteriaType criteriaType;

        public CustomEdge(LineStation lineStation, CriteriaType criteriaType) {
            this.lineStation = lineStation;
            this.criteriaType = criteriaType;
        }

        public LineStation getLineStation() {
            return lineStation;
        }

        @Override
        protected double getWeight() {
            return criteriaType.get(lineStation);
        }
    }

   ```
   - 기존에 알고 있던 지식도 복습을 통해 충분히 학습하자
        - Parameterized 테스트, extracting, usingRecursive 등등
   - 특정 Type과 연관된 로직을 수행하는 경우 타입에 따른 전략을 Enum을 통해 활용할 수 있다.
   - 자신이 만든 Collection(일급 컬렉션 등)을 통해 컬렉션을 변경에 용이하게 관리할 수 있다.
   - Service는 도메인의 실행순서 및 트랜잭션 등을 보장하기 위한 역할을 담당한다.
   - flatMap을 통해 반복문을 가독성 있게 변경할 수 있다.
   ```java
   lines.stream()
            .flatMap(it -> it.getLineStationsId().stream())
            .forEach(graph::addVertex);
   ```
### 아직 어려운 개념들 😂
   - 간단한 경우 이외에도 flatMap등 Stream에 익숙해져야겠다.!
   - 특정 인터페이스를 상속하는 경우 변하지 않는 것은 Template으로 두고, 변하는 부분만 상속을 통해 해결하였다.
        - 다만 이 방법이 맞는지 알기 힘들다.
        - 변하지 않는 구조에선 합리적일 수 있으나 변하지 않는다는 보장이 없다.
        - 이런 경우 어떤 선택을 하는 것이 좋을지 어렵다..

---

<br/>

## 기능 목록
- [x] 지하철 노선도 페이지 조회
    - [x] 모든 지하철 노선과 지하철역 목록을 조회
- [x] 캐시 적용
- [x] 최단 경로
- [x] 예외 처리
- [x] 최소 시간

## 시나리오(ATDD)

````gherkin
Feature: 전체 지하철 노선도 정보 조회

  Scenario: 지하철 노선도 정보 조회를 한다.
    Given 지하철역이 여러 개 추가되어있다.
    And 지하철 노선이 여러 개 추가되어있다.
    And 지하철 노선에 지하철역이 여러 개 추가되어있다.
    
    When 지하철 노선도 전체 조회 요청을 한다.
    
    Then 지하철 노선도 전체를 응답 받는다.
````

## TO-DO

- [x] 최단 경로
    - [x] 인수테스트 ← 라이브러리 쓰기
    - [x] 컨르롤러 레이어 구현 이후 서비스 레이어 구현 시 서비스 테스트 우선 작성 후 기능 구현
    - [x] 서비스 테스트 내부에서 도메인들간의 로직의 흐름을 검증, 이 때 사용되는 도메인은 mock 객체를 활용
    - [x] 외부 라이브러리를 활용한 로직을 검증할 때는 가급적 실제 객체를 활용
    - [x] Happy 케이스에 대한 부분만 구현( Side 케이스에 대한 구현은 다음 단계에서 진행)
    
- [x] 경로 조회 화면
    - [x] 출발역과 도착역의 경로 정보 노출
    - [x] 총 소요시간, 정차역 등
    
- [x] 예외 처리
    - [x] 출발역과 도착역이 같은 경우
    - [x] 출발역과 도착역이 연결이 되어 있지 않은 경우
    - [x] 존재하지 않은 출발역이나 도착역을 조회 할 경우
    
- [x] 최소 시간
    - [x] 최소 시간 경로 기능을 추가
    - [x] 최단 경로 조회 기능과의 중복 제거를 수행(테스트 코드도 마찬가지로 중복제거)

- [x] 경로조회 로직 최적화
