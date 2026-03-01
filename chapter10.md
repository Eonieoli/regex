# Chapter 10: 플래그와 컴파일 옵션

> **난이도**: ⭐⭐ | **예상 학습 시간**: 2.5시간 | **선수 지식**: Chapter 5 (re 모듈 기본 함수)

---

# Part A: 도입부

## 학습 목표 (Learning Objectives)

이 Chapter를 완료하면, 당신은 다음을 할 수 있습니다:

- [ ] `re.IGNORECASE`, `re.MULTILINE`, `re.DOTALL` 등 주요 플래그의 효과를 설명하고 적용할 수 있다
- [ ] `re.MULTILINE` 모드에서 `^`/`$`와 `\A`/`\Z`의 동작 차이를 정확히 구분할 수 있다
- [ ] `re.VERBOSE`를 사용하여 복잡한 패턴에 주석과 공백을 추가하여 가독성을 높일 수 있다
- [ ] 여러 플래그를 `|` 연산자로 조합하여 사용할 수 있다
- [ ] 인라인 플래그 `(?flags)`를 패턴 내부에서 사용할 수 있다

---

## 핵심 질문 (Essential Questions)

1. **정규표현식의 동작 방식을 패턴 수정 없이 바꿀 수 있다면?** — 같은 패턴이라도 "설정"에 따라 완전히 다른 결과를 내는 것이 가능할까요?
2. **여러 줄로 이루어진 텍스트를 줄 단위로 매칭하려면?** — `^`와 `$`가 문자열 전체의 시작과 끝만 가리킨다면, 각 줄의 시작과 끝은 어떻게 잡을 수 있을까요?
3. **복잡한 정규표현식을 읽기 쉽게 만들 수 있을까?** — 길고 난해한 패턴에 주석을 달고 줄바꿈을 넣을 수 있다면 어떨까요?

---

## 개념 지도 (Concept Map)

```
[선수: re.compile(), re.search(), re.findall() (Ch5)]
[선수: ^ $ \A \Z 앵커 (Ch2)]
[선수: . 와일드카드 (Ch2)]
[선수: \d \w \s 문자 클래스 (Ch3)]
    │
    ▼
[신규: 플래그(flags) — 정규표현식의 "동작 모드 스위치"]
    │
    ├── re.IGNORECASE (re.I)  — 대소문자 무시
    ├── re.MULTILINE  (re.M)  — 줄 단위 앵커
    ├── re.DOTALL     (re.S)  — . 이 줄바꿈도 매칭
    ├── re.VERBOSE    (re.X)  — 주석·공백 허용
    ├── re.ASCII      (re.A)  — ASCII 전용 모드
    └── 인라인 플래그 (?i) (?m) (?s) (?x)
    │
    ▼
[결과: 같은 패턴으로도 다양한 매칭 동작을 유연하게 제어]
    │
    ▼
[이후: Ch11 유니코드 처리에서 re.ASCII 활용]
```

---

# Part B: 본문 (Main Content)

---

## 10.1 플래그 개요와 사용법

> 💡 **한 줄 요약**: 플래그는 정규표현식 엔진의 "동작 모드 스위치"로, 패턴 자체를 바꾸지 않고도 매칭 방식을 변경할 수 있게 해줍니다.

### 직관적 이해

카메라로 비유해 봅시다. 같은 장면을 찍더라도 카메라 설정(ISO, 화이트밸런스, 필터 등)에 따라 결과물이 달라지듯이, 같은 정규표현식 패턴이라도 **플래그 설정**에 따라 매칭 결과가 완전히 달라질 수 있습니다.

예를 들어, `hello`라는 패턴은 기본적으로 소문자 `hello`만 찾습니다. 그런데 "대소문자를 무시해"라는 스위치를 켜면, 갑자기 `Hello`, `HELLO`, `hElLo`까지 모두 찾게 됩니다. 패턴은 그대로인데, 동작 방식이 바뀐 것이죠.

왜 플래그가 필요할까요? 패턴 자체에 모든 경우의 수를 넣으면 패턴이 지나치게 복잡해지기 때문입니다. `[Hh][Ee][Ll][Ll][Oo]`처럼 쓰는 것보다 `hello`에 대소문자 무시 플래그를 켜는 것이 훨씬 간결하고 읽기 좋습니다.

### 핵심 개념 설명

> 📝 **플래그(Flag)**: 정규표현식 엔진의 동작 방식을 변경하는 옵션 설정값. 패턴의 문법을 바꾸는 것이 아니라, 엔진이 패턴을 *해석하고 적용하는 방식*을 바꿉니다.

파이썬 `re` 모듈에서 플래그를 사용하는 방법은 크게 **두 가지**입니다.

**방법 1: 함수 호출 시 `flags` 매개변수로 전달**

> 🔗 **연결**: Chapter 5에서 배운 `re.search()`, `re.findall()` 등의 함수에는 `flags` 매개변수가 있습니다.

```python
import re

# flags 매개변수에 플래그 전달
result = re.search(r'hello', 'Hello World', flags=re.IGNORECASE)
print(result.group())  # 'Hello'
```

**방법 2: `re.compile()`로 패턴 객체를 만들 때 플래그 전달**

> 🔗 **연결**: Chapter 5에서 배운 `re.compile()`을 사용하면 패턴을 사전 컴파일하여 재사용할 수 있습니다.

```python
import re

# compile 시 플래그 전달
pattern = re.compile(r'hello', re.IGNORECASE)
result = pattern.search('Hello World')
print(result.group())  # 'Hello'
```

두 방법은 기능적으로 동일합니다. 같은 패턴을 여러 번 사용한다면 `re.compile()`이 효율적이고, 한 번만 쓴다면 함수에 직접 전달하는 것이 간편합니다.

**파이썬 `re` 모듈의 주요 플래그**

| 플래그 | 약어 | 효과 |
|--------|------|------|
| `re.IGNORECASE` | `re.I` | 대소문자를 구분하지 않고 매칭 |
| `re.MULTILINE` | `re.M` | `^`와 `$`가 각 줄의 시작/끝에도 매칭 |
| `re.DOTALL` | `re.S` | `.`이 줄바꿈 문자(`\n`)에도 매칭 |
| `re.VERBOSE` | `re.X` | 패턴 내 공백과 주석(`#`) 허용 |
| `re.ASCII` | `re.A` | `\w`, `\d`, `\s` 등을 ASCII 범위로 제한 |

각 플래그에는 긴 이름과 **한 글자 약어**가 있습니다. `re.IGNORECASE`와 `re.I`는 완전히 동일합니다. 코드의 가독성을 위해 긴 이름을 사용하는 것이 권장되지만, 짧은 코드에서는 약어도 자주 쓰입니다.

**여러 플래그를 동시에 사용하기: `|` 연산자**

플래그는 비트 OR 연산자 `|`로 조합할 수 있습니다.

```python
import re

# 대소문자 무시 + 여러 줄 모드를 동시에 적용
text = "Hello\nhello\nHELLO"
results = re.findall(r'^hello$', text, flags=re.IGNORECASE | re.MULTILINE)
print(results)  # ['Hello', 'hello', 'HELLO']
```

플래그를 3개 이상 조합하는 것도 가능합니다:

```python
flags = re.IGNORECASE | re.MULTILINE | re.DOTALL
```

> 🔍 **예시 10.1.1: 플래그 유무에 따른 결과 차이**
>
> **상황**: 문자열 `"Python is Great"`에서 `python`을 찾고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "Python is Great"
>
> # 플래그 없이 — 대소문자 정확히 일치해야 함
> result1 = re.search(r'python', text)
> print(result1)  # None (매칭 실패)
>
> # re.IGNORECASE 플래그 사용
> result2 = re.search(r'python', text, re.IGNORECASE)
> print(result2.group())  # 'Python' (매칭 성공)
> ```
>
> **핵심 포인트**: 패턴 `python`은 동일하지만, 플래그 하나로 매칭 결과가 완전히 달라집니다.

> 🔍 **예시 10.1.2: 플래그 조합 사용**
>
> **상황**: 여러 줄 텍스트에서 각 줄의 시작에 있는 `#`으로 시작하는 주석 줄을 모두 찾되, `#`이 대문자인지 소문자인지는 상관없이 `#todo`도 찾고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = """# 일반 주석
> print('hello')
> #TODO: 나중에 수정
> # 또 다른 주석
> #todo: 이것도 찾아야 함"""
>
> # MULTILINE으로 각 줄의 시작 매칭 + IGNORECASE로 대소문자 무시
> results = re.findall(r'^#.*todo.*', text, re.MULTILINE | re.IGNORECASE)
> print(results)  # ['#TODO: 나중에 수정', '#todo: 이것도 찾아야 함']
> ```
>
> **핵심 포인트**: `re.MULTILINE`은 `^`가 각 줄 시작에 매칭되게 하고, `re.IGNORECASE`는 `todo`의 대소문자를 무시합니다. 두 플래그가 독립적으로 작용합니다.

> ⚠️ **주의**: 플래그는 패턴 **전체**에 적용됩니다. 패턴의 일부분에만 플래그를 적용하고 싶다면 인라인 플래그를 사용해야 하는데, 이는 Section 10.6에서 다룹니다.

### Section 요약

- 플래그는 정규표현식 엔진의 동작 모드를 변경하는 옵션 스위치이다
- `re.search(pattern, string, flags=...)` 또는 `re.compile(pattern, flags)`로 사용한다
- 각 플래그에는 긴 이름(`re.IGNORECASE`)과 약어(`re.I`)가 있다
- 여러 플래그를 `|` 연산자로 조합할 수 있다
- 플래그는 패턴 전체에 적용된다

---

## 10.2 `re.IGNORECASE` — 대소문자 무시 매칭

> 💡 **한 줄 요약**: `re.IGNORECASE`(약어 `re.I`)를 사용하면 영문자의 대소문자를 구분하지 않고 매칭합니다.

### 직관적 이해

웹사이트의 검색창을 생각해 봅시다. 사용자가 "apple"을 검색하든 "Apple"을 검색하든 "APPLE"을 검색하든 같은 결과가 나와야 합니다. 이것이 바로 대소문자 무시(case-insensitive) 매칭입니다.

플래그 없이 이것을 구현하려면 `[Aa][Pp][Pp][Ll][Ee]`처럼 모든 문자의 대소문자를 일일이 나열해야 합니다. 단어가 길어질수록 패턴은 기하급수적으로 복잡해지죠. `re.IGNORECASE` 하나면 이 모든 수고를 덜 수 있습니다.

### 핵심 개념 설명

> 📝 **`re.IGNORECASE` (re.I)**: 패턴의 영문 알파벳 매칭 시 대문자와 소문자를 동일하게 취급하는 플래그.

기본 동작을 살펴봅시다:

```python
import re

text = "Python, python, PYTHON, PyThOn"

# 기본 모드: 정확한 대소문자만 매칭
print(re.findall(r'python', text))
# ['python']

# IGNORECASE 모드: 모든 대소문자 조합 매칭
print(re.findall(r'python', text, re.IGNORECASE))
# ['Python', 'python', 'PYTHON', 'PyThOn']
```

`re.IGNORECASE`는 패턴 내의 **모든 영문 알파벳**에 적용됩니다. 패턴에 숫자, 특수문자, 한글 등이 포함되어 있어도 영문 알파벳 부분만 대소문자 무시가 적용됩니다.

> 🔗 **연결**: Chapter 3에서 배운 문자 클래스에도 IGNORECASE가 적용됩니다.

```python
import re

# 문자 클래스 [a-z]에 IGNORECASE 적용 → 대문자도 매칭
print(re.findall(r'[a-z]+', "Hello WORLD", re.IGNORECASE))
# ['Hello', 'WORLD']

# 플래그 없으면 소문자만 매칭
print(re.findall(r'[a-z]+', "Hello WORLD"))
# ['ello']
```

> 🔍 **예시 10.2.1: HTML 태그 매칭**
>
> **상황**: HTML에서 태그 이름은 대소문자를 구분하지 않는다. `<div>`, `<DIV>`, `<Div>` 모두 같은 태그이므로, 이들을 모두 찾고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> html = "<DIV>내용1</div> <Span>내용2</SPAN> <p>내용3</P>"
>
> # div 태그를 대소문자 구분 없이 찾기
> results = re.findall(r'<div>', html, re.IGNORECASE)
> print(results)  # ['<DIV>']
>
> # 모든 여는 태그를 대소문자 구분 없이 찾기
> results = re.findall(r'<[a-z]+>', html, re.IGNORECASE)
> print(results)  # ['<DIV>', '<Span>', '<p>']
> ```
>
> **핵심 포인트**: 패턴을 소문자로만 작성해도, 플래그 덕분에 대문자 조합까지 모두 매칭됩니다.

> 🔍 **예시 10.2.2: 키워드 검색**
>
> **상황**: 사용자 입력에서 "error", "warning" 키워드를 대소문자 무관하게 찾아야 한다.
>
> **풀이/설명**:
> ```python
> import re
>
> log = """ERROR: 파일을 찾을 수 없습니다
> Warning: 메모리 사용량이 높습니다
> info: 정상 동작 중
> error: 연결 시간 초과"""
>
> pattern = re.compile(r'error|warning', re.IGNORECASE)
> results = pattern.findall(log)
> print(results)  # ['ERROR', 'Warning', 'error']
> ```
>
> **핵심 포인트**: `|`(선택 연산자)와 함께 사용하면 여러 키워드를 대소문자 무시로 한번에 검색할 수 있습니다.

> ⚠️ **주의**: `re.IGNORECASE`는 **영문 알파벳**에 주로 적용됩니다. 독일어 `ß`와 `SS`의 관계처럼 유니코드의 복잡한 대소문자 변환은 `re` 모듈의 기본 IGNORECASE로 완벽하게 처리되지 않을 수 있습니다. 대부분의 영어 기반 작업에서는 문제없이 동작합니다.

> ⚠️ **주의**: `re.IGNORECASE`를 사용할 때 문자 클래스 내부의 범위 지정에 주의하세요. `[a-z]`에 IGNORECASE를 적용하면 `[a-zA-Z]`와 같은 효과이지만, `[A-z]`처럼 범위를 지정하면 예상치 못한 문자가 포함될 수 있습니다. 항상 `[a-z]` + IGNORECASE 또는 `[a-zA-Z]`처럼 명확하게 작성하세요.

### Section 요약

- `re.IGNORECASE`(또는 `re.I`)는 영문 대소문자를 구분하지 않고 매칭한다
- 패턴 전체의 영문 알파벳에 적용된다
- 문자 클래스 `[a-z]`에도 적용되어 대문자도 매칭된다
- HTML 태그, 키워드 검색 등 대소문자가 혼용되는 상황에서 유용하다

---

## 10.3 `re.MULTILINE`과 앵커의 동작 변화

> 💡 **한 줄 요약**: `re.MULTILINE`(약어 `re.M`)을 사용하면 `^`와 `$`가 문자열 전체의 시작/끝뿐 아니라, **각 줄(line)의 시작/끝**에도 매칭됩니다.

### 직관적 이해

메모장에서 "찾기" 기능을 사용할 때를 떠올려 봅시다. 문서 전체에서 "각 줄의 첫 단어"를 찾고 싶다면, 줄바꿈 문자 바로 다음에 오는 단어를 찾아야 합니다. 정규표현식에서 `^`는 "시작 위치"를 의미하는데, 기본적으로는 **전체 문자열의 시작**만을 가리킵니다. 여러 줄 텍스트에서 **각 줄의 시작**을 가리키게 하려면, `re.MULTILINE` 플래그로 동작 방식을 전환해야 합니다.

이것은 마치 책 전체를 하나의 문장으로 보느냐, 페이지별로 나누어 보느냐의 차이와 같습니다.

### 핵심 개념 설명

> 📝 **`re.MULTILINE` (re.M)**: `^`와 `$` 앵커가 문자열 전체의 시작/끝뿐만 아니라, 줄바꿈 문자(`\n`) 바로 다음/직전 위치에도 매칭되도록 변경하는 플래그.

> 🔗 **연결**: Chapter 2에서 배운 앵커들을 복습합시다. `^`는 시작 위치, `$`는 끝 위치에 매칭되고, `\A`는 문자열의 절대적 시작, `\Z`는 절대적 끝에 매칭됩니다. 당시 "`^`/`$`와 `\A`/`\Z`의 차이가 MULTILINE에서 드러난다"고 예고했는데, 바로 이 Chapter에서 그 차이를 배웁니다.

**기본 모드에서의 `^`와 `$`:**

```python
import re

text = "첫 번째 줄\n두 번째 줄\n세 번째 줄"

# 기본 모드: ^는 전체 문자열의 시작만 매칭
print(re.findall(r'^.+', text))
# ['첫 번째 줄'] — 첫 번째 줄만 매칭됨

# 기본 모드: $는 전체 문자열의 끝만 매칭
print(re.findall(r'.+$', text))
# ['세 번째 줄'] — 마지막 줄만 매칭됨
```

**MULTILINE 모드에서의 `^`와 `$`:**

```python
import re

text = "첫 번째 줄\n두 번째 줄\n세 번째 줄"

# MULTILINE 모드: ^가 각 줄의 시작에 매칭
print(re.findall(r'^.+', text, re.MULTILINE))
# ['첫 번째 줄', '두 번째 줄', '세 번째 줄'] — 모든 줄 매칭!

# MULTILINE 모드: $가 각 줄의 끝에 매칭
print(re.findall(r'.+$', text, re.MULTILINE))
# ['첫 번째 줄', '두 번째 줄', '세 번째 줄'] — 모든 줄 매칭!
```

**핵심 차이: `^`/`$` vs `\A`/`\Z`**

`re.MULTILINE`의 영향을 받는 것은 **`^`와 `$`뿐**입니다. `\A`와 `\Z`는 항상 문자열 전체의 시작과 끝만을 가리키며, MULTILINE 모드에서도 **동작이 변하지 않습니다**.

```python
import re

text = "첫 번째 줄\n두 번째 줄\n세 번째 줄"

# MULTILINE 모드에서도 \A는 전체 문자열 시작만 매칭
print(re.findall(r'\A.+', text, re.MULTILINE))
# ['첫 번째 줄'] — 여전히 첫 줄만!

# MULTILINE 모드에서도 \Z는 전체 문자열 끝만 매칭
print(re.findall(r'.+\Z', text, re.MULTILINE))
# ['세 번째 줄'] — 여전히 마지막 줄만!
```

이 차이를 표로 정리하면:

| 앵커 | 기본 모드 | MULTILINE 모드 |
|------|-----------|----------------|
| `^` | 전체 문자열 시작 | 전체 시작 + **각 줄 시작** |
| `$` | 전체 문자열 끝 | 전체 끝 + **각 줄 끝** |
| `\A` | 전체 문자열 시작 | 전체 문자열 시작 (변함없음) |
| `\Z` | 전체 문자열 끝 | 전체 문자열 끝 (변함없음) |

> 🔍 **예시 10.3.1: 여러 줄 텍스트에서 특정 패턴으로 시작하는 줄 찾기**
>
> **상황**: 설정 파일에서 `#`으로 시작하는 주석 줄만 모두 찾고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> config = """# 데이터베이스 설정
> host = localhost
> port = 5432
> # 보안 설정
> ssl = true
> # 로깅 설정
> log_level = info"""
>
> # 기본 모드: 첫 줄만 찾음
> print(re.findall(r'^#.*', config))
> # ['# 데이터베이스 설정']
>
> # MULTILINE 모드: 모든 주석 줄을 찾음
> print(re.findall(r'^#.*', config, re.MULTILINE))
> # ['# 데이터베이스 설정', '# 보안 설정', '# 로깅 설정']
> ```
>
> **핵심 포인트**: 설정 파일, 소스 코드, 로그 파일 등 여러 줄 텍스트를 줄 단위로 처리할 때 `re.MULTILINE`은 필수적입니다.

> 🔍 **예시 10.3.2: `^`/`$`와 `\A`/`\Z`의 차이 체감하기**
>
> **상황**: 여러 줄 텍스트에서 "전체 문자열이 숫자로 시작하는지" vs "숫자로 시작하는 줄이 있는지"를 구분해야 한다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "이름: 홍길동\n나이: 30\n직업: 개발자"
>
> # "전체 문자열이 숫자로 시작하는가?" → \A 사용
> print(bool(re.search(r'\A\d', text, re.MULTILINE)))
> # False — 전체 문자열은 '이'로 시작
>
> # "숫자로 시작하는 줄이 있는가?" → ^ + MULTILINE 사용
> print(bool(re.search(r'^\d', text, re.MULTILINE)))
> # False — 어떤 줄도 숫자로 시작하지 않음
>
> text2 = "이름: 홍길동\n30세\n직업: 개발자"
>
> print(bool(re.search(r'\A\d', text2, re.MULTILINE)))
> # False — 전체 문자열은 여전히 '이'로 시작
>
> print(bool(re.search(r'^\d', text2, re.MULTILINE)))
> # True — 두 번째 줄 '30세'가 숫자로 시작함!
> ```
>
> **핵심 포인트**: "전체 문자열의 시작/끝"을 확인하고 싶으면 `\A`/`\Z`, "줄 단위 시작/끝"을 확인하고 싶으면 `^`/`$` + MULTILINE을 사용합니다.

> 🔬 **Deep Dive**: `$`의 미묘한 동작
>
> *이 부분은 심화 내용입니다. 처음 학습 시 건너뛰어도 됩니다.*
>
> `$`에는 한 가지 미묘한 점이 있습니다. 기본 모드에서도 `$`는 문자열 끝의 줄바꿈 문자 **직전**에 매칭될 수 있습니다. 즉, 문자열이 `"hello\n"`으로 끝나면, `$`는 `\n` 바로 앞에도 매칭됩니다:
>
> ```python
> import re
>
> # 문자열 끝에 \n이 있는 경우
> print(re.search(r'hello$', "hello\n"))
> # <re.Match object; span=(0, 5), match='hello'> — 매칭됨!
>
> # \Z는 진짜 문자열 끝에만 매칭
> print(re.search(r'hello\Z', "hello\n"))
> # None — \n 뒤가 진짜 끝이므로 매칭 안 됨
> ```
>
> 이 동작은 파일을 줄 단위로 읽을 때 각 줄 끝에 `\n`이 붙는 경우를 편리하게 처리하기 위한 것입니다. 정밀한 문자열 끝 매칭이 필요하면 `\Z`를 사용하세요.

> ⚠️ **주의**: `re.MULTILINE`은 `^`와 `$`의 동작**만** 변경합니다. `.`(마침표)의 동작에는 영향을 주지 않습니다. `.`이 줄바꿈 문자와도 매칭되게 하려면 `re.DOTALL`(다음 Section)을 사용해야 합니다. 이 두 플래그를 혼동하는 것은 매우 흔한 실수입니다.

### Section 요약

- `re.MULTILINE`(또는 `re.M`)은 `^`와 `$`의 동작을 변경하여 줄 단위 매칭을 가능하게 한다
- `^`/`$`는 MULTILINE에서 각 줄의 시작/끝에 매칭되지만, `\A`/`\Z`는 항상 전체 문자열의 시작/끝만 매칭한다
- 여러 줄 텍스트(설정 파일, 로그, 코드)를 줄 단위로 처리할 때 핵심적인 플래그이다
- `.`의 동작과는 무관하다 (`.`의 변경은 `re.DOTALL`)

---

## 10.4 `re.DOTALL` — `.`의 줄바꿈 문자 매칭 활성화

> 💡 **한 줄 요약**: `re.DOTALL`(약어 `re.S`)을 사용하면 `.`(마침표)가 줄바꿈 문자(`\n`)를 포함한 **모든 문자**에 매칭됩니다.

### 직관적 이해

> 🔗 **연결**: Chapter 2에서 `.`(마침표)가 "줄바꿈 문자를 제외한 모든 문자"에 매칭된다고 배웠습니다.

이 제한은 보통 편리하지만, 때로는 걸림돌이 됩니다. 예를 들어, HTML 소스에서 `<div>`와 `</div>` 사이의 모든 내용을 추출하고 싶은데, 그 안에 줄바꿈이 포함되어 있다면? 기본 모드의 `.`은 줄바꿈에서 멈춰버립니다.

`re.DOTALL`은 이 제한을 풀어주는 스위치입니다. 플래그 이름에서 "DOT ALL"이 의미하는 바 그대로, "마침표(dot)가 모든(all) 것에 매칭되게 하라"는 뜻입니다.

> 💬 **참고**: `re.DOTALL`의 약어가 `re.S`인 이유는 역사적으로 이 모드를 "single-line 모드"라고 부르기도 했기 때문입니다. 전체 문자열을 하나의 긴 줄처럼 취급한다는 의미에서 붙은 이름입니다. 이름이 혼동을 줄 수 있으니 주의하세요 — `re.MULTILINE`과 `re.DOTALL`은 서로 반대 개념이 아니라, 완전히 다른 것을 제어합니다.

### 핵심 개념 설명

> 📝 **`re.DOTALL` (re.S)**: `.`(마침표) 메타문자가 줄바꿈 문자(`\n`)를 포함한 모든 문자에 매칭되도록 변경하는 플래그.

**기본 모드 vs DOTALL 모드:**

```python
import re

text = "첫 번째 줄\n두 번째 줄"

# 기본 모드: .은 \n에서 멈춤
print(re.search(r'.+', text).group())
# '첫 번째 줄' — 줄바꿈 전까지만

# DOTALL 모드: .이 \n도 포함
print(re.search(r'.+', text, re.DOTALL).group())
# '첫 번째 줄\n두 번째 줄' — 전체 텍스트를 하나로 매칭
```

> 🔍 **예시 10.4.1: 여러 줄에 걸친 블록 추출**
>
> **상황**: 아래 텍스트에서 "시작"과 "끝" 사이의 모든 내용(줄바꿈 포함)을 추출하고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = """여기는 무시
> === 시작 ===
> 중요한 내용 1
> 중요한 내용 2
> 중요한 내용 3
> === 끝 ===
> 여기도 무시"""
>
> # 기본 모드: .이 줄바꿈을 넘지 못함 → 매칭 실패
> result = re.search(r'=== 시작 ===(.+)=== 끝 ===', text)
> print(result)  # None
>
> # DOTALL 모드: .이 줄바꿈도 포함 → 매칭 성공
> result = re.search(r'=== 시작 ===(.+)=== 끝 ===', text, re.DOTALL)
> print(result.group())
> # === 시작 ===\n중요한 내용 1\n중요한 내용 2\n중요한 내용 3\n=== 끝 ===
> ```
>
> **핵심 포인트**: 여러 줄에 걸친 블록을 통째로 매칭하려면 `re.DOTALL`이 필요합니다.

> 💬 **참고**: 위 예시에서 `(.+)` 부분의 소괄호 `()`는 매칭된 내용의 일부를 따로 추출하기 위한 **캡처 그룹**입니다. 이 기능은 Chapter 6에서 자세히 다루며, 아직 배우지 않았다면 지금은 "소괄호로 감싼 부분만 별도로 꺼낼 수 있다" 정도로 이해해 주세요.

> 🔍 **예시 10.4.2: DOTALL 없이 줄바꿈을 넘는 대안 — `[\s\S]`**
>
> **상황**: `re.DOTALL` 플래그를 사용하지 않고도 줄바꿈을 포함한 모든 문자를 매칭하고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "시작\n중간\n끝"
>
> # 방법 1: re.DOTALL 사용
> print(re.search(r'시작.+끝', text, re.DOTALL).group())
> # '시작\n중간\n끝'
>
> # 방법 2: [\s\S] 사용 (공백 문자 + 비공백 문자 = 모든 문자)
> print(re.search(r'시작[\s\S]+끝', text).group())
> # '시작\n중간\n끝'
> ```
>
> **핵심 포인트**: `[\s\S]`는 "공백 문자 또는 비공백 문자" = 모든 문자를 의미하며, 플래그 없이도 줄바꿈을 넘을 수 있는 트릭입니다. 하지만 가독성 면에서는 `re.DOTALL` + `.`이 더 명확합니다.

> ⚠️ **주의**: `re.DOTALL`과 `re.MULTILINE`을 혼동하지 마세요!
>
> | 플래그 | 변경하는 것 | 기억법 |
> |--------|------------|--------|
> | `re.MULTILINE` | `^`와 `$`의 동작 | "줄 단위로 앵커" |
> | `re.DOTALL` | `.`의 동작 | "점(dot)이 모든(all) 것에 매칭" |
>
> 이 둘은 서로 독립적이므로 동시에 사용할 수도 있습니다. `re.MULTILINE | re.DOTALL`로 조합하면 `^`/`$`가 줄 단위로 동작하면서 `.`도 줄바꿈에 매칭됩니다.

### Section 요약

- `re.DOTALL`(또는 `re.S`)은 `.`이 줄바꿈(`\n`)을 포함한 모든 문자에 매칭되게 한다
- 여러 줄에 걸친 블록을 통째로 매칭할 때 필수적이다
- `[\s\S]`는 플래그 없이 같은 효과를 내는 대안 패턴이다
- `re.DOTALL`(`.`의 동작 변경)과 `re.MULTILINE`(`^`/`$`의 동작 변경)은 완전히 다른 플래그이다

---

## 10.5 `re.VERBOSE` — 공백과 주석을 이용한 패턴 가독성 향상

> 💡 **한 줄 요약**: `re.VERBOSE`(약어 `re.X`)를 사용하면 패턴 안에 공백과 `#` 주석을 자유롭게 넣어 복잡한 정규표현식의 가독성을 크게 높일 수 있습니다.

### 직관적 이해

다음 정규표현식을 보세요:

```python
r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
```

이것은 이메일 주소를 매칭하는 패턴인데, 한눈에 이해하기 어렵습니다. 프로그래밍에서 긴 코드에 주석을 달고 줄바꿈으로 구조를 나누듯이, 정규표현식에도 같은 것을 할 수 있다면 얼마나 좋을까요?

`re.VERBOSE`가 바로 그 기능을 제공합니다. 같은 패턴을 이렇게 쓸 수 있습니다:

```python
pattern = re.compile(r'''
    ^                    # 문자열 시작
    [a-zA-Z0-9._%+-]+   # 사용자 이름 부분
    @                    # @ 기호
    [a-zA-Z0-9.-]+       # 도메인 이름
    \.                   # 점 (리터럴)
    [a-zA-Z]{2,}         # 최상위 도메인 (2자 이상)
    $                    # 문자열 끝
''', re.VERBOSE)
```

훨씬 읽기 쉽죠? 기능은 완전히 동일하지만, 가독성은 비교할 수 없을 만큼 좋아졌습니다.

### 핵심 개념 설명

> 📝 **`re.VERBOSE` (re.X)**: 패턴 문자열 내의 공백(whitespace)을 무시하고, `#`부터 줄 끝까지를 주석으로 처리하는 플래그. 패턴의 논리적 구조를 시각적으로 표현할 수 있게 해줍니다.

`re.VERBOSE` 모드에서의 규칙:

1. **공백(스페이스, 탭, 줄바꿈)이 무시됩니다** — 패턴의 가독성을 위해 자유롭게 공백을 넣을 수 있습니다.
2. **`#`부터 줄 끝까지 주석으로 처리됩니다** — 패턴의 각 부분에 설명을 달 수 있습니다.
3. **문자 클래스 `[...]` 내부는 예외** — 대괄호 안의 공백은 여전히 리터럴 공백으로 취급됩니다.

```python
import re

# VERBOSE 모드: 공백과 주석이 무시됨
pattern = re.compile(r'''
    \d{3}    # 지역번호 (3자리)
    -        # 하이픈 구분자
    \d{4}    # 국번 (4자리)
    -        # 하이픈 구분자
    \d{4}    # 개별번호 (4자리)
''', re.VERBOSE)

print(pattern.search("전화번호: 010-1234-5678").group())
# '010-1234-5678'
```

**리터럴 공백이 필요한 경우**

VERBOSE 모드에서 실제 공백 문자를 패턴에 포함하고 싶다면, 두 가지 방법이 있습니다:

```python
import re

text = "hello world"

# 방법 1: 이스케이프된 공백 \  (백슬래시 + 스페이스)
pattern1 = re.compile(r'hello\ world', re.VERBOSE)
print(pattern1.search(text).group())  # 'hello world'

# 방법 2: 문자 클래스 안의 공백 [ ]
pattern2 = re.compile(r'hello[ ]world', re.VERBOSE)
print(pattern2.search(text).group())  # 'hello world'

# 방법 3: \s 사용 (공백 외에 탭 등도 포함하므로 넓은 범위)
pattern3 = re.compile(r'hello \s world', re.VERBOSE)
print(pattern3.search(text).group())  # 'hello world'
```

> 🔍 **예시 10.5.1: 복잡한 패턴을 VERBOSE로 읽기 쉽게 만들기**
>
> **상황**: 한국 전화번호 패턴 `010-1234-5678` 또는 `02-123-4567` 형식을 매칭하는 패턴을 작성하되, 나중에 다른 사람이 읽기 쉽게 만들고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> # VERBOSE 없이: 한 줄에 모든 것이 압축됨
> pattern_compact = r'(0\d{1,2})-(\d{3,4})-(\d{4})'
>
> # VERBOSE 사용: 구조가 명확하게 드러남
> pattern_verbose = re.compile(r'''
>     (0\d{1,2})    # 그룹1: 지역/통신사 번호 (0으로 시작, 2~3자리)
>     -             # 하이픈 구분자
>     (\d{3,4})     # 그룹2: 국번 (3~4자리)
>     -             # 하이픈 구분자
>     (\d{4})       # 그룹3: 개별번호 (4자리)
> ''', re.VERBOSE)
>
> test_numbers = ["010-1234-5678", "02-123-4567", "031-456-7890"]
> for num in test_numbers:
>     match = pattern_verbose.search(num)
>     if match:
>         print(f"{num} → 매칭됨: {match.group()}")
> # 010-1234-5678 → 매칭됨: 010-1234-5678
> # 02-123-4567 → 매칭됨: 02-123-4567
> # 031-456-7890 → 매칭됨: 031-456-7890
> ```
>
> **핵심 포인트**: VERBOSE 모드는 패턴의 기능을 변경하지 않으면서, 사람이 읽고 유지보수하기 쉽게 만들어 줍니다. 복잡한 패턴일수록 효과가 큽니다.

> 🔍 **예시 10.5.2: VERBOSE에서 리터럴 `#`과 공백 사용하기**
>
> **상황**: Python 소스 코드에서 `# 주석`으로 시작하는 줄을 찾는 패턴을 VERBOSE 모드로 작성하되, 패턴 안에서 리터럴 `#`을 사용해야 한다.
>
> **풀이/설명**:
> ```python
> import re
>
> code = """x = 10
> # 이것은 주석
> y = 20
> # 또 다른 주석"""
>
> # VERBOSE에서 리터럴 #을 쓰려면 이스케이프(\#) 또는 문자 클래스([#])
> pattern = re.compile(r'''
>     ^          # 줄의 시작
>     [#]        # 리터럴 # 문자 (문자 클래스 안에서 사용)
>     .+         # 나머지 주석 내용
> ''', re.VERBOSE | re.MULTILINE)
>
> results = pattern.findall(code)
> print(results)  # ['# 이것은 주석', '# 또 다른 주석']
> ```
>
> **핵심 포인트**: VERBOSE 모드에서 리터럴 `#`이 필요하면 `[#]` 또는 `\#`으로 이스케이프해야 합니다. 그냥 `#`을 쓰면 주석 시작으로 해석됩니다.

> ⚠️ **주의**: VERBOSE 모드에서 가장 흔한 실수는 **의도하지 않은 공백이 패턴에 포함되는 것**입니다. 기본 모드에서는 `hello world`의 공백이 리터럴 공백이지만, VERBOSE 모드에서는 무시됩니다. 기존 패턴에 VERBOSE 플래그를 추가할 때는 공백의 처리를 반드시 확인하세요.
>
> ```python
> # 의도: "hello world"를 매칭하고 싶음
>
> # 잘못된 예: VERBOSE에서 공백이 무시됨 → "helloworld"를 매칭하게 됨
> re.search(r'hello world', "hello world", re.VERBOSE)  # None!
>
> # 올바른 예: 공백을 명시적으로 표현
> re.search(r'hello\ world', "hello world", re.VERBOSE)  # 매칭됨
> ```

### Section 요약

- `re.VERBOSE`(또는 `re.X`)는 패턴 내 공백과 `#` 주석을 허용하여 가독성을 높인다
- 공백은 무시되고, `#`부터 줄 끝까지는 주석으로 처리된다
- 문자 클래스 `[...]` 내부의 공백은 여전히 리터럴로 취급된다
- 리터럴 공백이 필요하면 `\ `, `[ ]`, 또는 `\s`를 사용한다
- 리터럴 `#`이 필요하면 `\#` 또는 `[#]`을 사용한다
- 복잡한 패턴의 유지보수성을 크게 향상시키는 실무 필수 플래그이다

---

## 10.6 `re.ASCII`와 기타 플래그

> 💡 **한 줄 요약**: `re.ASCII`는 `\w`, `\d`, `\s`, `\b` 등의 매칭 범위를 ASCII 문자로 제한하며, 인라인 플래그를 사용하면 패턴 문자열 안에서 직접 플래그를 지정할 수 있습니다.

### 직관적 이해

파이썬 3에서 정규표현식은 기본적으로 **유니코드 모드**로 동작합니다. 즉, `\w`는 영문자뿐 아니라 한글, 중국어, 일본어 등 모든 언어의 "단어 문자"에 매칭됩니다. 이것은 다국어 환경에서 강력한 장점이지만, 때로는 "영문과 숫자만 매칭하고 싶다"는 요구가 있을 수 있습니다. `re.ASCII`는 바로 이때 사용합니다.

또한 지금까지 플래그를 함수의 매개변수로 전달해 왔는데, 패턴 문자열 자체에 플래그를 내장하는 방법도 있습니다. 이를 **인라인 플래그**라고 합니다.

### 핵심 개념 설명

**`re.ASCII` (re.A)**

> 📝 **`re.ASCII` (re.A)**: `\w`, `\W`, `\b`, `\B`, `\d`, `\D`, `\s`, `\S`의 매칭 범위를 ASCII 문자(영문, 숫자, 기본 공백)로 제한하는 플래그.

> 🔗 **연결**: Chapter 3에서 배운 `\w`, `\d`, `\s`를 떠올려 봅시다. 기본적으로 `\w`는 `[a-zA-Z0-9_]` + 유니코드 문자입니다.

```python
import re

text = "hello_세계_123"

# 기본(유니코드) 모드: \w가 한글도 포함
print(re.findall(r'\w+', text))
# ['hello_세계_123'] — 한글을 포함하여 한 덩어리로 매칭

# ASCII 모드: \w가 영문, 숫자, 밑줄만
print(re.findall(r'\w+', text, re.ASCII))
# ['hello_', '_123'] — '세계'가 \w에서 제외됨
```

주요 차이를 표로 정리하면:

| 축약 | 유니코드 모드 (기본) | ASCII 모드 (`re.ASCII`) |
|------|---------------------|------------------------|
| `\d` | 모든 유니코드 숫자 (아라비아, 페르시아 숫자 등 포함) | `[0-9]`만 |
| `\w` | 모든 유니코드 문자 + 숫자 + `_` | `[a-zA-Z0-9_]`만 |
| `\s` | 모든 유니코드 공백 | `[ \t\n\r\f\v]`만 |
| `\b` | 유니코드 단어 경계 | ASCII 단어 경계 |

> 💬 **참고**: `re.ASCII`의 세부적인 유니코드 동작 차이는 Chapter 11 "유니코드와 한국어 처리"에서 더 자세히 다룹니다. 지금은 "ASCII 모드는 영문·숫자 위주의 처리에 적합하다" 정도로 이해하면 충분합니다.

> 🔍 **예시 10.6.1: `re.ASCII`로 영문만 추출하기**
>
> **상황**: 영어와 한국어가 혼합된 텍스트에서 영단어만 추출하고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "Python은 재미있는 language입니다. 정규표현식은 powerful합니다."
>
> # 기본 모드: \w+가 한글 단어도 포함
> print(re.findall(r'\b\w+\b', text))
> # ['Python은', '재미있는', 'language입니다', '정규표현식은', 'powerful합니다']
>
> # ASCII 모드: \w+가 영문만 매칭, \b도 ASCII 경계로 동작
> print(re.findall(r'[a-zA-Z]+', text, re.ASCII))
> # ['Python', 'language', 'powerful']
> ```
>
> **핵심 포인트**: 다국어 텍스트에서 영문만 선택적으로 추출할 때 `re.ASCII`가 유용합니다.

---

**인라인 플래그 (Inline Flags)**

> 📝 **인라인 플래그**: 패턴 문자열 내부에 `(?flags)` 형태로 삽입하여, `flags` 매개변수 없이도 플래그를 적용하는 방법.

지금까지는 플래그를 함수 호출 시 매개변수로 전달했습니다. 인라인 플래그를 사용하면 패턴 자체에 플래그 정보를 내장할 수 있습니다.

| 인라인 플래그 | 대응하는 상수 |
|--------------|-------------|
| `(?i)` | `re.IGNORECASE` |
| `(?m)` | `re.MULTILINE` |
| `(?s)` | `re.DOTALL` |
| `(?x)` | `re.VERBOSE` |
| `(?a)` | `re.ASCII` |

**사용 방법 1: 패턴 시작 부분에 배치 (전체 적용)**

```python
import re

# flags 매개변수 대신 인라인 플래그 사용
result = re.search(r'(?i)hello', 'Hello World')
print(result.group())  # 'Hello'

# 여러 인라인 플래그 조합
result = re.search(r'(?im)^hello', 'Hi\nHello\nWorld')
print(result.group())  # 'Hello'
```

**사용 방법 2: 인라인 플래그의 조합**

인라인 플래그 문자는 하나의 `(?...)` 안에 여러 개를 넣을 수 있습니다:

```python
import re

# (?ims) = IGNORECASE + MULTILINE + DOTALL
text = "Start\nHello World\nEnd"
result = re.search(r'(?ims)^hello.+end', text)
print(result.group())
# 'Hello World\nEnd'
```

> 🔍 **예시 10.6.2: 인라인 플래그 활용**
>
> **상황**: 패턴 문자열을 변수로 저장해서 여러 곳에서 재사용할 때, 플래그 정보도 패턴에 포함시키고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> # 패턴 안에 플래그 정보가 내장됨
> EMAIL_PATTERN = r'(?i)[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}'
>
> texts = [
>     "연락처: User@Example.COM",
>     "이메일: admin@COMPANY.co.kr",
>     "주소: test@test.org"
> ]
>
> for text in texts:
>     match = re.search(EMAIL_PATTERN, text)
>     if match:
>         print(f"발견: {match.group()}")
> # 발견: User@Example.COM
> # 발견: admin@COMPANY.co.kr
> # 발견: test@test.org
> ```
>
> **핵심 포인트**: 인라인 플래그를 사용하면 패턴 문자열 하나에 동작 모드가 내장되어, 패턴을 전달할 때 플래그를 별도로 관리할 필요가 없습니다.

> 🔬 **Deep Dive**: 인라인 플래그의 범위 제한
>
> *이 부분은 심화 내용입니다. 처음 학습 시 건너뛰어도 됩니다.*
>
> 파이썬 3.6 이전에는 인라인 플래그를 패턴 중간에 배치하여 해당 위치부터만 적용할 수 있었지만, 파이썬 3.6부터 이 동작에 대한 경고가 도입되었고, 이후 버전에서는 인라인 플래그를 **패턴의 시작 부분에만** 배치하거나, 그룹 단위로 적용하도록 권장합니다.
>
> ```python
> # 권장: 패턴 시작에 배치
> r'(?i)hello world'
>
> # 비권장 (파이썬 버전에 따라 경고 또는 오류):
> # r'hello (?i)world'
> ```
>
> 그룹 단위 인라인 플래그는 `(?i:pattern)` 형태로, 해당 그룹 내부에서만 플래그가 적용됩니다:
>
> ```python
> import re
>
> # 'hello'는 대소문자 무시, 'world'는 정확히 매칭
> result = re.search(r'(?i:hello) world', 'Hello world')
> print(result.group())  # 'Hello world'
>
> result = re.search(r'(?i:hello) world', 'Hello World')
> print(result)  # None — 'World'의 W가 대문자이므로 매칭 안 됨
> ```
>
> 이 문법은 패턴의 일부에만 선택적으로 플래그를 적용할 때 유용합니다.

> ⚠️ **주의**: 인라인 플래그와 `flags` 매개변수를 동시에 사용하면, 두 가지가 OR 조합됩니다. 하지만 코드의 명확성을 위해 한 가지 방식만 일관되게 사용하는 것이 좋습니다.

**기타 알아두면 좋은 플래그**

| 플래그 | 약어 | 설명 |
|--------|------|------|
| `re.LOCALE` | `re.L` | `\w`, `\b` 등을 현재 로케일 설정에 맞게 변경 (바이트 패턴에서만 사용, 거의 사용하지 않음) |
| `re.DEBUG` | 없음 | 컴파일된 패턴의 내부 구조를 출력 (디버깅용) |

> 💬 **참고**: `re.DEBUG`는 Chapter 16 "성능 최적화와 디버깅"에서 자세히 다룹니다.

### Section 요약

- `re.ASCII`(또는 `re.A`)는 `\w`, `\d`, `\s`, `\b`의 매칭 범위를 ASCII 문자로 제한한다
- 인라인 플래그 `(?i)`, `(?m)`, `(?s)`, `(?x)`, `(?a)`를 사용하면 패턴 안에 플래그를 내장할 수 있다
- 인라인 플래그는 패턴 시작 부분에 배치하는 것이 권장된다
- `(?i:pattern)` 형태로 특정 그룹에만 플래그를 적용할 수 있다
- 플래그를 일관된 방식(매개변수 또는 인라인 중 하나)으로 사용하는 것이 좋다

---

# Part C: 통합 및 정리 (Integration Block)

---

## Chapter 핵심 요약 (Chapter Summary)

이 Chapter에서는 정규표현식의 동작 모드를 변경하는 **플래그(flags)**를 학습했습니다. 패턴 자체를 수정하지 않고도 매칭 방식을 유연하게 제어할 수 있는 강력한 도구입니다.

- **10.1 플래그 개요와 사용법**: 플래그는 `flags` 매개변수 또는 `re.compile()`로 전달하며, `|` 연산자로 여러 플래그를 조합할 수 있습니다.
- **10.2 `re.IGNORECASE`**: 영문 대소문자를 구분하지 않고 매칭합니다. HTML 태그, 키워드 검색 등에 유용합니다.
- **10.3 `re.MULTILINE`**: `^`와 `$`가 각 줄의 시작/끝에도 매칭됩니다. `\A`/`\Z`는 영향받지 않습니다.
- **10.4 `re.DOTALL`**: `.`이 줄바꿈 문자를 포함한 모든 문자에 매칭됩니다. 여러 줄에 걸친 블록 매칭에 필수적입니다.
- **10.5 `re.VERBOSE`**: 패턴에 공백과 주석을 넣어 가독성을 높입니다. 복잡한 패턴의 유지보수에 핵심적입니다.
- **10.6 `re.ASCII`와 인라인 플래그**: `re.ASCII`는 매칭 범위를 ASCII로 제한하고, 인라인 플래그 `(?i)` 등은 패턴 안에 플래그를 내장합니다.

---

## 핵심 용어 정리 (Glossary)

| 용어 | 정의 | 관련 Section |
|------|------|-------------|
| 플래그 (Flag) | 정규표현식 엔진의 동작 모드를 변경하는 옵션 설정값 | 10.1 |
| `re.IGNORECASE` (`re.I`) | 영문 대소문자를 구분하지 않고 매칭하는 플래그 | 10.2 |
| `re.MULTILINE` (`re.M`) | `^`/`$`가 각 줄의 시작/끝에도 매칭되도록 변경하는 플래그 | 10.3 |
| `re.DOTALL` (`re.S`) | `.`이 줄바꿈을 포함한 모든 문자에 매칭되도록 변경하는 플래그 | 10.4 |
| `re.VERBOSE` (`re.X`) | 패턴 내 공백 무시와 `#` 주석을 허용하는 플래그 | 10.5 |
| `re.ASCII` (`re.A`) | `\w`, `\d`, `\s` 등의 매칭 범위를 ASCII로 제한하는 플래그 | 10.6 |
| 인라인 플래그 (Inline Flag) | `(?i)`, `(?m)` 등 패턴 문자열 내에 삽입하는 플래그 표기법 | 10.6 |
| 그룹 단위 인라인 플래그 | `(?i:pattern)` 형태로 특정 그룹에만 플래그를 적용하는 문법 | 10.6 |

---

## 개념 연결 맵 (Connection Map)

**이전 Chapter와의 연결:**
- Chapter 2에서 배운 `^`, `$`, `\A`, `\Z` 앵커와 `.` 와일드카드의 동작이 `re.MULTILINE`과 `re.DOTALL`에 의해 변경됩니다. Chapter 2에서 "MULTILINE에서의 동작 차이를 나중에 배운다"고 예고했던 내용이 이 Chapter에서 완성되었습니다.
- Chapter 3에서 배운 `\w`, `\d`, `\s`, `\b`의 매칭 범위가 `re.ASCII`에 의해 제한될 수 있음을 배웠습니다.
- Chapter 5에서 배운 `re.compile()`과 함수형 호출에 `flags` 매개변수를 전달하는 방법을 적용했습니다.

**이후 Chapter로의 연결:**
- **Chapter 11** (유니코드와 한국어 처리)에서 `re.ASCII`의 유니코드 모드와의 차이를 더 깊이 다룹니다.
- **Chapter 12~14** (실전 응용)에서 여러 플래그를 조합하여 복잡한 실무 문제를 해결합니다.
- **Chapter 16** (성능 최적화와 디버깅)에서 `re.DEBUG` 플래그를 활용한 패턴 분석을 배웁니다.

---

## 자기 점검 질문 (Self-Check Questions)

**Q1.** `re.search(r'hello', 'HELLO', re.IGNORECASE)`의 결과는 매칭 성공인가, 실패인가?

<details>
<summary>정답</summary>

**매칭 성공.** `re.IGNORECASE`가 대소문자를 무시하므로 `hello` 패턴이 `HELLO`에 매칭됩니다.
</details>

**Q2.** `re.MULTILINE` 모드에서 `\A`는 각 줄의 시작에 매칭된다. (O/X)

<details>
<summary>정답</summary>

**X.** `\A`는 MULTILINE 모드에서도 **전체 문자열의 시작**에만 매칭됩니다. 각 줄의 시작에 매칭되는 것은 `^`입니다.
</details>

**Q3.** `re.DOTALL` 모드에서 `.`은 어떤 문자에 매칭되는가?

<details>
<summary>정답</summary>

줄바꿈 문자(`\n`)를 **포함한 모든 문자**에 매칭됩니다. 기본 모드에서는 `.`이 줄바꿈을 제외한 모든 문자에 매칭됩니다.
</details>

**Q4.** `re.VERBOSE` 모드에서 패턴 내의 공백은 어떻게 처리되는가?

<details>
<summary>정답</summary>

**무시됩니다.** 리터럴 공백을 매칭하려면 `\ `(이스케이프), `[ ]`(문자 클래스), 또는 `\s`를 사용해야 합니다. 단, 문자 클래스 `[...]` 내부의 공백은 그대로 리터럴로 취급됩니다.
</details>

**Q5.** 여러 플래그를 동시에 사용하려면 어떤 연산자를 사용하는가?

<details>
<summary>정답</summary>

비트 OR 연산자 **`|`**를 사용합니다. 예: `re.IGNORECASE | re.MULTILINE | re.DOTALL`
</details>

**Q6.** 인라인 플래그 `(?im)`은 어떤 플래그 조합에 해당하는가?

<details>
<summary>정답</summary>

`re.IGNORECASE | re.MULTILINE`에 해당합니다. `i`는 IGNORECASE, `m`은 MULTILINE입니다.
</details>

**Q7.** `re.MULTILINE`과 `re.DOTALL`은 서로 반대 개념이다. (O/X)

<details>
<summary>정답</summary>

**X.** 이 둘은 서로 독립적인 플래그입니다. `re.MULTILINE`은 `^`/`$`의 동작을 변경하고, `re.DOTALL`은 `.`의 동작을 변경합니다. 동시에 사용할 수도 있습니다.
</details>

---

# Part D: 문제 풀이 (Problem Set)

---

## D1. Practice (연습 문제) — 기본 개념 확인

> **Practice 10.1**: 다음 코드의 출력 결과를 예측하세요.
> ```python
> import re
> print(re.findall(r'python', 'Python PYTHON python', re.IGNORECASE))
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `['Python', 'PYTHON', 'python']`
>
> **해설**: `re.IGNORECASE` 플래그로 인해 `python` 패턴이 대소문자를 구분하지 않고 모든 변형을 매칭합니다. `findall()`은 매칭된 모든 문자열을 리스트로 반환합니다.
> </details>

> **Practice 10.2**: 다음 코드의 출력 결과를 예측하세요.
> ```python
> import re
> text = "apple\nbanana\ncherry"
> print(re.findall(r'^[a-z]+', text))
> print(re.findall(r'^[a-z]+', text, re.MULTILINE))
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> - 첫 번째: `['apple']`
> - 두 번째: `['apple', 'banana', 'cherry']`
>
> **해설**: 기본 모드에서 `^`는 전체 문자열의 시작만 매칭하므로 `'apple'`만 찾습니다. `re.MULTILINE`을 사용하면 `^`가 각 줄의 시작에도 매칭되어 세 단어 모두 찾습니다.
> </details>

> **Practice 10.3**: 다음 중 `re.DOTALL` 플래그가 필요한 경우는?
>
> (a) `re.search(r'hello.world', 'hello world')`  
> (b) `re.search(r'hello.world', 'hello\nworld')`  
> (c) `re.search(r'hello.world', 'helloXworld')`  
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: (b)
>
> **해설**: (a)와 (c)에서는 `.`이 매칭할 문자가 공백과 `X`로, 줄바꿈이 아니므로 기본 모드에서도 동작합니다. (b)에서는 `.`이 `\n`(줄바꿈)에 매칭해야 하므로 `re.DOTALL`이 필요합니다.
> </details>

> **Practice 10.4**: 다음 두 패턴은 동일한 결과를 내는가? (text = `"Hello\nWorld"`, MULTILINE 모드)
>
> 패턴 A: `r'^World'` (with `re.MULTILINE`)  
> 패턴 B: `r'\AWorld'` (with `re.MULTILINE`)  
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: 동일하지 않습니다.
>
> **해설**: 패턴 A에서 `^`는 MULTILINE 모드이므로 두 번째 줄의 시작에 매칭되어 `'World'`를 찾습니다. 패턴 B에서 `\A`는 항상 전체 문자열의 시작만 매칭하므로, `'H'`로 시작하는 문자열에서 `World`를 찾지 못합니다. 결과: A는 매칭 성공, B는 매칭 실패.
> </details>

> **Practice 10.5**: `re.VERBOSE` 모드에서 다음 패턴이 매칭하는 문자열은 무엇인가요?
> ```python
> pattern = re.compile(r'''
>     \d{4}   # 연도
>     -       # 구분자
>     \d{2}   # 월
>     -       # 구분자
>     \d{2}   # 일
> ''', re.VERBOSE)
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `2024-01-15`, `1999-12-31` 등 `YYYY-MM-DD` 형식의 날짜 문자열.
>
> **해설**: VERBOSE 모드에서 공백과 `#` 뒤의 주석이 무시되므로, 이 패턴은 `\d{4}-\d{2}-\d{2}`와 동일하게 동작합니다.
> </details>

> **Practice 10.6**: 인라인 플래그 `(?i)`를 사용하여 다음 코드를 재작성하세요.
> ```python
> result = re.search(r'error', log_text, re.IGNORECASE)
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> result = re.search(r'(?i)error', log_text)
> ```
>
> **해설**: `(?i)`를 패턴 시작에 배치하면 `flags=re.IGNORECASE`와 동일한 효과입니다. `flags` 매개변수를 생략할 수 있습니다.
> </details>

> **Practice 10.7**: `re.VERBOSE` 모드에서 리터럴 공백을 매칭하는 올바른 방법 **두 가지**를 쓰세요.
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `\ `(백슬래시 + 스페이스), `[ ]`(문자 클래스 안의 스페이스). 추가로 `\s`도 공백을 매칭하지만 탭 등 다른 공백 문자도 포함합니다.
>
> **해설**: VERBOSE 모드에서 일반 공백은 무시되므로, 실제 공백 문자를 매칭하려면 이스케이프하거나 문자 클래스로 감싸야 합니다.
> </details>

---

## D2. Exercise (응용 문제) — 개념 적용 및 통합

> **Exercise 10.1**: 다음과 같은 여러 줄 서버 로그에서, 각 줄의 시작에 있는 타임스탬프(`YYYY-MM-DD` 형식)를 모두 추출하세요.
>
> ```python
> log = """2024-01-15 서버 시작
> 메모리 사용량: 75%
> 2024-01-15 요청 처리 완료
> 응답 시간: 120ms
> 2024-01-16 에러 발생"""
> ```
>
> **힌트**: `^`와 `re.MULTILINE`을 조합하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 타임스탬프가 줄 시작에 있으므로 `^` 앵커를 사용하되, 여러 줄에서 모두 찾아야 하므로 `re.MULTILINE`을 적용합니다.
>
> **풀이 과정**:
> ```python
> import re
>
> log = """2024-01-15 서버 시작
> 메모리 사용량: 75%
> 2024-01-15 요청 처리 완료
> 응답 시간: 120ms
> 2024-01-16 에러 발생"""
>
> results = re.findall(r'^\d{4}-\d{2}-\d{2}', log, re.MULTILINE)
> print(results)
> ```
>
> **정답**: `['2024-01-15', '2024-01-15', '2024-01-16']`
>
> **보충 설명**: `re.MULTILINE` 없이는 첫 줄의 `2024-01-15`만 반환됩니다. "메모리 사용량"이나 "응답 시간" 줄은 숫자로 시작하지 않으므로 `^\d`에 매칭되지 않습니다.
> </details>

> **Exercise 10.2**: 다음 HTML 코드에서 모든 `<style>` ~ `</style>` 블록의 내용(태그 포함)을 추출하세요. 스타일 블록은 여러 줄에 걸쳐 있을 수 있으며, 태그의 대소문자는 무관합니다.
>
> ```python
> html = """<html>
> <head>
> <STYLE>
> body { color: red; }
> p { font-size: 14px; }
> </Style>
> </head>
> <body>내용</body>
> </html>"""
> ```
>
> **힌트**: 대소문자 무시, 줄바꿈을 넘는 매칭 — 두 개의 플래그가 필요합니다.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: (1) `<style>`과 `</style>`의 대소문자가 다를 수 있으므로 `re.IGNORECASE`, (2) 내용이 여러 줄에 걸쳐 있으므로 `re.DOTALL`이 필요합니다.
>
> **풀이 과정**:
> ```python
> import re
>
> html = """<html>
> <head>
> <STYLE>
> body { color: red; }
> p { font-size: 14px; }
> </Style>
> </head>
> <body>내용</body>
> </html>"""
>
> result = re.search(r'<style>.+?</style>', html, re.IGNORECASE | re.DOTALL)
> print(result.group())
> ```
>
> **정답**:
> ```
> <STYLE>
> body { color: red; }
> p { font-size: 14px; }
> </Style>
> ```
>
> **보충 설명**: `.+?`의 `?`는 비탐욕적 매칭으로, 가장 가까운 `</style>`에서 매칭을 멈춥니다(Chapter 4에서 배운 내용). `re.IGNORECASE`가 없으면 `<STYLE>`과 `</Style>`을 매칭하지 못하고, `re.DOTALL`이 없으면 줄바꿈을 넘지 못합니다.
> </details>

> **Exercise 10.3**: 다음 설정 파일에서 주석 줄(`#`으로 시작)을 모두 제거하고, 빈 줄도 제거한 결과를 출력하세요. `re.VERBOSE`를 활용하여 패턴에 주석을 달아 가독성을 높이세요.
>
> ```python
> config = """# 서버 설정
> host = localhost
> port = 8080
> # 데이터베이스
> db_name = myapp
>
> # 캐시 설정
> cache = true"""
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: `re.MULTILINE`로 줄 단위 매칭, `re.VERBOSE`로 패턴 가독성 확보. 주석 줄과 빈 줄을 매칭하여 제거합니다.
>
> **풀이 과정**:
> ```python
> import re
>
> config = """# 서버 설정
> host = localhost
> port = 8080
> # 데이터베이스
> db_name = myapp
>
> # 캐시 설정
> cache = true"""
>
> # 주석 줄 또는 빈 줄을 매칭하는 패턴 (VERBOSE로 주석 추가)
> pattern = re.compile(r'''
>     ^           # 줄의 시작
>     (           # 두 가지 경우:
>         [#].*   #   1) # 으로 시작하는 주석 줄
>         |       #   또는
>         \s*     #   2) 공백만 있는 빈 줄
>     )           #
>     $           # 줄의 끝
> ''', re.MULTILINE | re.VERBOSE)
>
> # 매칭되는 줄을 빈 문자열로 치환
> # (re.sub은 Chapter 7의 내용이지만, 여기서는 간단히 사용)
> cleaned = re.sub(pattern, '', config)
>
> # 연속된 줄바꿈을 하나로 정리
> cleaned = re.sub(r'\n{2,}', '\n', cleaned).strip()
> print(cleaned)
> ```
>
> **정답**:
> ```
> host = localhost
> port = 8080
> db_name = myapp
> cache = true
> ```
>
> **보충 설명**: `re.VERBOSE` 덕분에 패턴의 각 부분이 무엇을 하는지 명확하게 드러납니다. `[#]`은 VERBOSE 모드에서 리터럴 `#`을 표현하기 위해 문자 클래스를 사용한 것입니다.
> </details>

> **Exercise 10.4**: 다음 코드의 출력 결과를 예측하고, 그 이유를 설명하세요.
>
> ```python
> import re
> text = "Hello\nworld\n"
> 
> print("A:", re.findall(r'.+', text))
> print("B:", re.findall(r'.+', text, re.DOTALL))
> print("C:", re.findall(r'^.+$', text, re.MULTILINE))
> print("D:", re.findall(r'^.+$', text, re.MULTILINE | re.DOTALL))
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **정답**:
> ```
> A: ['Hello', 'world']
> B: ['Hello\nworld\n']
> C: ['Hello', 'world']
> D: ['Hello\nworld']
> ```
>
> **풀이 과정**:
> - **A**: 기본 모드에서 `.`은 `\n`을 제외한 모든 문자에 매칭. `.+`가 각 줄의 내용만 탐욕적으로 매칭하여 `['Hello', 'world']`.
> - **B**: DOTALL 모드에서 `.`이 `\n`도 포함. `.+`가 전체 문자열을 한 덩어리로 탐욕적 매칭하여 `['Hello\nworld\n']`.
> - **C**: MULTILINE 모드에서 `^`/`$`가 각 줄 시작/끝에 매칭. 기본 모드의 `.`이 `\n` 제외이므로 각 줄 단위로 매칭하여 `['Hello', 'world']`. 마지막 빈 줄은 `.+`의 `+`(1회 이상)에 매칭되지 않음.
> - **D**: MULTILINE + DOTALL. `^`/`$`가 줄 단위이지만, `.`이 `\n`도 포함하여 `.+`가 탐욕적으로 가능한 한 많이 매칭. `^`는 첫 줄 시작에 매칭되고, `.+`가 `Hello\nworld`까지 매칭한 후 `$`가 `world` 뒤 줄 끝에 매칭. 결과: `['Hello\nworld']`.
>
> **보충 설명**: D가 가장 까다로운 경우입니다. DOTALL로 `.`이 `\n`을 넘지만, `$`가 MULTILINE에서 각 줄 끝에 매칭되므로 탐욕적 `.+`와 `$`의 상호작용으로 전체가 아닌 마지막 `\n` 직전까지 매칭됩니다.
> </details>

---

## D3. Problem (심화 문제) — 깊이 있는 사고와 창의적 문제 해결

> **Problem 10.1**: 아래와 같은 Python 소스 코드 문자열이 주어졌을 때, 여러 줄 문자열(triple-quoted string, `"""..."""`)의 내용을 추출하는 패턴을 작성하세요. 패턴은 `re.VERBOSE`를 사용하여 주석을 포함해야 합니다.
>
> ```python
> source = '''
> x = 10
> description = """이것은
> 여러 줄에 걸친
> 문자열입니다"""
> y = 20
> note = """또 다른
> 멀티라인 문자열"""
> '''
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: `"""`로 시작하고 끝나는 블록을 매칭해야 하며, 내부에 줄바꿈이 있으므로 `re.DOTALL`이 필요합니다. 탐욕적 매칭이면 첫 `"""`부터 마지막 `"""`까지 하나로 묶이므로, 비탐욕적 매칭을 사용해야 합니다.
>
> **풀이 전략**: `re.VERBOSE | re.DOTALL`을 조합하고, `""".+?"""`로 비탐욕적 매칭.
>
> **상세 풀이**:
> ```python
> import re
>
> source = '''
> x = 10
> description = """이것은
> 여러 줄에 걸친
> 문자열입니다"""
> y = 20
> note = """또 다른
> 멀티라인 문자열"""
> '''
>
> pattern = re.compile(r'''
>     \"{3}     # 여는 삼중 따옴표 ("""를 리터럴로 매칭)
>     (.+?)     # 내용 (비탐욕적 — 가장 짧은 매칭)
>     \"{3}     # 닫는 삼중 따옴표
> ''', re.VERBOSE | re.DOTALL)
>
> results = pattern.findall(source)
> for i, content in enumerate(results, 1):
>     print(f"--- 문자열 {i} ---")
>     print(content)
> ```
>
> **결과**:
> ```
> --- 문자열 1 ---
> 이것은
> 여러 줄에 걸친
> 문자열입니다
> --- 문자열 2 ---
> 또 다른
> 멀티라인 문자열
> ```
>
> **확장 생각**: 실제 Python 파서는 삼중 따옴표 안에 이스케이프된 `\"""`가 있을 수도 있고, 작은따옴표 버전(`'''...'''`)도 있습니다. 완전한 파서를 만들려면 정규표현식만으로는 한계가 있으며, 이는 Chapter 17에서 다루는 "정규표현식의 대안" 주제와 연결됩니다.
> </details>

> **Problem 10.2**: 다음과 같은 Markdown 형식의 텍스트가 있습니다. `re.VERBOSE`와 적절한 플래그 조합을 사용하여, 각 섹션의 **제목**(#으로 시작하는 줄)과 **내용**(다음 # 줄 직전까지)을 분리 추출하세요.
>
> ```python
> markdown = """# 소개
> 이 문서는 파이썬 정규표현식에 대한
> 학습 자료입니다.
>
> # 기본 문법
> 정규표현식의 기본 문법을 알아봅시다.
> 메타문자와 리터럴이 있습니다.
>
> # 마무리
> 지금까지 학습한 내용을 정리합니다."""
> ```
>
> 기대하는 출력: 각 섹션의 제목과 내용이 구분되어 추출
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: `#`으로 시작하는 줄이 제목이고, 다음 `#` 줄 또는 문자열 끝까지가 내용입니다. `re.MULTILINE`으로 줄 단위 `^`를 사용하고, `re.DOTALL`로 줄바꿈을 넘는 매칭을 합니다.
>
> **풀이 전략**: 비탐욕적 매칭으로 각 섹션을 분리합니다.
>
> **상세 풀이**:
> ```python
> import re
>
> markdown = """# 소개
> 이 문서는 파이썬 정규표현식에 대한
> 학습 자료입니다.
>
> # 기본 문법
> 정규표현식의 기본 문법을 알아봅시다.
> 메타문자와 리터럴이 있습니다.
>
> # 마무리
> 지금까지 학습한 내용을 정리합니다."""
>
> pattern = re.compile(r'''
>     ^[#][ ]     # 줄 시작의 "# " (VERBOSE에서 # 리터럴은 [#]로)
>     (.+)        # 제목 텍스트 (줄바꿈 전까지)
>     \n          # 줄바꿈
>     (.*?)       # 내용 (비탐욕적, 줄바꿈 포함)
>     (?=^[#]|\Z) # 다음 섹션 시작 또는 문자열 끝 직전에서 멈춤
> ''', re.VERBOSE | re.MULTILINE | re.DOTALL)
>
> for match in pattern.finditer(markdown):
>     title = match.group(1)
>     content = match.group(2).strip()
>     print(f"제목: {title}")
>     print(f"내용: {content}")
>     print()
> ```
>
> **결과**:
> ```
> 제목: 소개
> 내용: 이 문서는 파이썬 정규표현식에 대한
> 학습 자료입니다.
>
> 제목: 기본 문법
> 내용: 정규표현식의 기본 문법을 알아봅시다.
> 메타문자와 리터럴이 있습니다.
>
> 제목: 마무리
> 내용: 지금까지 학습한 내용을 정리합니다.
> ```
>
> **보충 설명**: 이 풀이에서 `(?=^[#]|\Z)` 부분은 **전방 탐색(lookahead)**이라는 기법으로, Chapter 9에서 다루는 고급 기능입니다. 현재는 "다음 `#` 줄이 시작되거나 문자열이 끝나는 위치를 찾되, 그 위치의 텍스트를 소비하지 않는다"는 의미로 이해하면 됩니다. 전방 탐색을 사용하지 않는 대안으로 `re.split()`을 활용할 수도 있습니다.
>
> **확장 생각**: 실제 Markdown 파서는 `##`, `###` 등 다단계 헤딩, 코드 블록 안의 `#`, 그리고 다양한 Markdown 확장 문법을 처리해야 합니다. 정규표현식으로 처리할 수 있는 범위와 전용 파서가 필요한 범위를 구분하는 감각이 중요합니다.
> </details>

---

# Part E: 마무리 (Closing Block)

---

## 다음 단계 안내 (What's Next)

이번 Chapter에서 배운 플래그 중 `re.ASCII`는 파이썬의 **유니코드 처리**와 밀접하게 연결되어 있습니다. 다음 **Chapter 11: 유니코드와 한국어 처리**에서는 파이썬 3의 유니코드 기본 모드에서 `\w`, `\d`, `\s`가 어떤 범위의 문자를 매칭하는지, 한글(`[가-힣]`)과 한글 자모를 어떻게 매칭하는지, 그리고 `re.ASCII`와 기본 유니코드 모드의 차이가 실무에서 어떤 영향을 미치는지를 깊이 있게 다룹니다. 한국어 데이터를 다루는 개발자에게 특히 중요한 내용입니다.

---

## 추가 학습 자원 (Further Resources)

- **Python 공식 문서 — re 모듈 플래그**: [https://docs.python.org/3/library/re.html#flags](https://docs.python.org/3/library/re.html#flags) — 모든 플래그의 공식 정의
- **regex101.com**: 오른쪽 패널에서 플래그(flags)를 체크박스로 설정하며 실시간으로 결과 변화를 확인할 수 있습니다. Python 모드를 선택하세요.
- **Jeffrey Friedl, 《Mastering Regular Expressions》** — Chapter 3 "Overview of Regular Expression Features and Flavors"에서 플래그의 역사와 엔진별 차이를 상세히 다룹니다.
