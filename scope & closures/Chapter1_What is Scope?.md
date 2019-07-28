# Chapter 1: 스코프란 무엇인가
프로그래밍 언어의 기본 패러다임 중 하나는 변수에 값을 저장하고 저장된 값을 가져다 쓰고 수정하는 것이다. 변수가 프로그램에 있으므로 다음과 같은 질문이 생길 수 있다.

- 변수는 어디에 살아있는가? (변수는 어디에 저장되는가?)
- 필요할 때 프로그램은 어떻게 변수를 찾는가?

이 질문을 통해 알 수 있는 것은 변수를 저장하고 나중에 찾는데 정의된 규칙이 필요하다는 점이고 이를 *스코프<sup>scope</sup>*라 한다.

## 1.1 컴파일러 이론
자바스크립트는 일반적으로 '동적' 혹은 '인터프리터[https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%ED%94%84%EB%A6%AC%ED%84%B0]' 언어로 분류하나 사실은 '컴파일러 언어'이다.

전통적인 컴파일러 언어의 처리 과정 - 컴파일레이션(Compilation)
1. 토크나이징<sup>Tokenizing</sup>/렉싱<sup>Lexing</sup>
문자열을 '토큰<sup>token</sup>'이라 불리는 의미 있는 조각으로 만드는 과정
`var a = 2;`는 다음의 토큰으로 나눌 수 있다.
- `var`
- `a`
- `=`
- `2`
- `;`

*Note:* 토크나이징과 렉싱의 차이는 미묘하고 학술적인 차이가 있는데, 토큰이 상태값을 유지하냐<sup>stateful</sup> 않냐<sup>stateless</sup>로 구분한다. '토크나이저<sup>Tokenizer</sup>'가 상태값 유지하기 위한 파싱룰을 적용한다면 `a`가 별개의 토큰인지 다른 토큰의 일부인지 밝혀낼 것이고 이는 곧 렉싱이다. [[참고](https://stackoverflow.com/a/380487)]

2. 파싱<sup>Parsing</sup>
토큰 배열로 프로그램 문법 구조를 반영하여 중첩 원소<sup>nested elements</sup>들의 트리<sup>tree</sup>로 변환하는 과정이다. 파싱의 결과로 만들어진 트리를 "AST<sup>*A*bstract *S*yntax *T*ree<sup>" (추상 구문 트리)라 부른다.

3. 코드 생성<sup>Code-Generation</sup>
