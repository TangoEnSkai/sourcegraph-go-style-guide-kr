Go style guide

---

[원문/Source Document](https://about.sourcegraph.com/handbook/engineering/go_style_guide)

이 글은 Sourcegraph에서 사용하고 있는 스타일 가이드를 요약한 글이다. 코드 이외 (non-code)의 스타일 가이드에 대해서는 다음을 참고하기 바란다: [CodeReviewComments](https://about.sourcegraph.com/handbook/communication/style_guide)
본 문서는 스타일 가이드에 대한 모든 것을 망라하고 있지는 않는다. 따라서 여기서 다루지 않는 부분은 [Go Code Review Comments](https://code.google.com/p/go-wiki/wiki/CodeReviewComments) 혹은 [Effective Go](http://golang.org/doc/effective_go.html)을 후에 참고하기 바란다.

# Panics
`panic`은 일반적으로 code paths에서 절대로 도달해서는 안되는 곳에 사용하도록 한다.

# Options
일반적으로、`Get(build BuildSpec, opt *BuildGetOptions) (*Build, Response, error)`처럼 
함수의 인수에 option형 구조체 포인터에 nil을 전달할 때 사용한다.
포인터가 `nil`인 경우 함수는 기본 동작(default behaviour)을 한다.
만약, 위의 option이 `nil`이 아니여야 하는 경우, 포인터(pointer) 대신 값(value)을 사용하거나 문서화 해야 한다. 

# Pull requests
만약 여러분들이 하나의 새로운 상수(one new constant) 혹은 다른 류의 정의(other kind of declaration)을 리스트에(a list) 추가한 경우,
`gofmt`가 자동적으로 다시 개행(re-indent)하기 때문에, 만약 이것이 개행에 영향을 준다면,
도입하려고 하는 새로운 상수나 변수 정의를 다른 블록으로 나누어 추가하여 리뷰를 요청한다 (newline으로 기존의 코드와 구별해 준다).
이러한 방법은 리뷰어로부터 하여금 불필요한 변경에 의식을 하지 않고, 추가하고자 하는 것에 집중할 수 있을 것이다. 
또한, 혹시 다른 사람들이 같은 블록에서 함께 작업하고 있다고 할 지라도, merge conflict를 걱정하지 않아도 된다. 
그리고 merge 전/후로 추가한 불필요한 newlines을 제거하고, 리뷰된 상수나 변수는 적절한 장소에 배치(place)한다.

# Group code blocks logically
여러분들의 변수(variables)는 실제로 사용되는 곳에 선언(declare)한다.

```go
a, b, c := Vector{1, 2, 3}, Vector{4, 5, 6}, Vector{7, 8, 9}
a, b := swap(a, b)

total := add([]Vector(a, b, c))...)
```

보다는

```go
a, b := Vector{1, 2, 3}, Vector{4, 5, 6}
a, b := swap(a, b)

c := Vector{7, 8, 9}
total := add([]Vector(a, b, c))...)
```

를 더 선호한다.

`Vector`형 변수`c`는 `Add()`직전까지 사용할 수 없다. 그러므로, 사용하기 직전에 바로 선언하는게 좋지 않을까?
구성요소들을 논리적으로 그룹화 하면, 여러분들은 구성 요소(components)의 맥락(context)를 잃지 않도록 할 수 있다. 
더욱 구체적으로는: 

* 여러분들의 멘탈 버퍼(mental buffer)를 줄일 수 있다 - 여러분들이 만약 screenreader를 사용한다면 어떤 방법이 좋겠는가?
* 코드 베이스(code base)로부터 정의(definitions)와 선언(declarations)을 찾기 위한 탐색(navigation)을 최소한으로 유지할 수 있다 - 그리고 여러분들이 복잡한 제어(fine motor control)를 다룰 때 어려움이 있을 때도 유용하다. 
* 코드 블록을 이해(comprehend a block of code)하기 위한 탐색 정도(the amount of navigation)를 최소화 할 수 있다.
* 정의와 변수를 쉽게 찾을 수 있도록 코드베이스를 작게 유지합시다. 복잡한 제어가 필요한 경우에도 효과적입니다.

이와 같은 룰은 인터페이스나 구조체 등에도 적용됩니다.

# Keep names short

```go
var vectorA, vectorB Vector
```

보다

```go
var a, b Vector
```

를 더 선호한다.

Go는 이미 간략한 [변수명을 장려](https://github.com/golang/go/wiki/CodeReviewComments#variable-names)하고 있다.

추가적으로, 짧은 이름(short names)이란:
* 빠르게 듣거나 읽을 수 있다
* 쉽게 탐색(navigate around) 할 수 있다
* 타이핑 할 때 비교적 적은 노력이 필요하다

위의 예와 같이, 여러분들은 아마도 `vectorA` 혹은 `vectorB`가 이름 내에 context를 함유하고 있기 때문에 좋은 이름이라고 생각할 지도 모른다.
이런 경우, 위와 같은 변수들이 다른 곳에서 언급(referred)된다면 혼란과 모호성(confusion/ambiguity)를 없엘 수 있다.
그러나, 앞서 언급한 것과 같이, 이러한 것들은 [블록을 논리적으로 그룹화 한다면](https://github.com/TangoEnSkai/sourcegraph-go-style-guide-kr/blob/master/guide.md#group-code-blocks-logically), 불필요하고 쓸모 없을(redundant / not necessary) 것이다.

# Make names meaningful

```go
var tVec, sVec Vector
```

보다

```go
var total, scaled Vector
```

를 선호한다.

의미있는 변수 이름을 사용하는 것은, 코드를 이해하기 위한 비용을 줄일 수 있다 (reduces the amount of work). 
구체적으로는:

* 그 변수가 무엇을 할 것인가에 대한 문맥(context)를 모두 기억할 필요가 없게 된다.
* 정의(definitions)나 사용법(usage)과 같은 것을 찾기 위해서 이곳저곳 살펴볼 필요가 없게 된다. 
* 중요한 변수인지 단순한 임시 변수(temporary placeholders) 판별하는데 도움이 된다.

가능한 한, 추가적으로 설명하는 주석(explanatory comments)보다는 의미있는 이름을 선호하도록 하자.
이해하기 복잡한 변수명과 함께 주석을 다는 경우, 주석은 코드를 탐색(navigate around)할 때 추가적인 요소가 될 수 있다.
결국 이후에 여러분들이 구현 중 애매한 변수명을 가진 변수를 마주하면, 변수 선언부분의 주석/설명 을 보기 위해서 코드베이스를 왔다 갔다(jumping around)해야 하는 데,
결국 주석의 자세한 설명이 있어도, 코드를 여러번 왔다 갔다 하는 횟수를 줄여주는데는 도움이 되지 않는다. 


# Use pronounceable names

```go
var tVec Vector
var addAllVecs(...)
```

보다

```go
var total Vector
func add(...)
```

를 선호한다. 

발음 하기 쉬운 이름은:  
 
를 선호한다. 
* screen reader로 실제 읽을 수 있다.
* 문자 각각 읽는 것 보다 더 빠르게 읽을 수 있다. 

[@juliaferraioli가 스크린리더를 통해서 Go코드를 탐색하는 간략한 유튜브 비디오를 보는 것을 추천한다.](https://www.youtube.com/watch?v=xwjvufcJK-Q)

이 데모에서 여러분들은 스크린리더가 어떻게 변수명을 다루는지(handle) 확인할 수 있었을 것이다. 결코 쉽지 않다! (It's rough, right?)


[@juliaferraioli](https://twitter.com/juliaferraioli)는 코드 리뷰를 할 때, 스크린리더가“gee-thub”라고 읽는 것을 사실은 “GitHub”이라고 이해하는데 15분 동안 머리를 쥐어짜야만(scratching her head) 했던 일화(anecdote)도 소개했다.
그러므로, 발음 하기 쉬운 이름을 사용할 것을 권고한다.
단어를 구성하지 마라 (Don't make up words). 
그 단어가 어떻게 발음 되는 지를 생각 해 보라. 
**가능한 단어의 조합이 되는 변수명은 피하도록 해라.** 스크린리더가 이를 한 단어로 취급 해 버릴지도 모르기 때문이다.

# Use new lines intentionally

```go
a, b := Vector{1, 2, 3}, Vector{4, 5, 6}
a, b := swap(a, b)

c := Vector{7, 8, 9}
total := add([]Vector(a, b, c))...)
```

만약 우리가 추천되는 조직화(recommended organisation)을 재고(revisit)한다면, new line의 사용법도 생각 해 볼 수 있다. 
줄 바꿈은 특별한 생각이나 의도 없이 코드에서 사용하는 일종의 향신료/후추(pepper)라고 할 수 있다. 
그러나, 개행은 강력한 메시지(signal)를 가질 수 있다. 
여러분들에게 줄 바꾸는 것을 일종의 단락 나누기(paragraph breaks)처럼 사용할 것을 추천한다. 
이를 전혀 사용하지 않는다면, 여러분들의 독자는 없을 것이다. 
단, 이를 또한 너무 많이 사용하게 되면 메시지가 조각화(fragmented)된다. 
new line을 잘 사용하면 사용자에게 논리적으로 코드 블록을 보고 읽을 수 있도록 안내(guide)할 수 있다.

그러므로, new line을 의도록으로 잘 사용해 보도록 하자!

