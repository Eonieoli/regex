# Chapter 5: re 모듈 기본 함수

> **난이도**: ⭐⭐ | **예상 학습 시간**: 3시간 | **선수 지식**: Chapter 1~4 (정규표현식 기본 문법, 메타문자, 문자 클래스, 수량자)

---

# Part A: 도입부

## 학습 목표 (Learning Objectives)

이 Chapter를 완료하면, 당신은 다음을 할 수 있습니다:

- [ ] `re.match()`, `re.search()`, `re.fullmatch()`의 차이를 이해하고 적절히 선택하여 사용할 수 있다
- [ ] `re.findall()`과 `re.finditer()`를 사용하여 모든 매칭 결과를 추출할 수 있다
- [ ] `re.compile()`로 패턴을 사전 컴파일하고 재사용할 수 있다
- [ ] `re.escape()`를 사용하여 사용자 입력 문자열의 메타문자를 안전하게 이스케이프할 수 있다
- [ ] 각 함수의 반환 타입과 매칭 실패 시 동작을 정확히 예측할 수 있다

---

## 핵심 질문 (Essential Questions)

1. **지금까지 배운 정규표현식 패턴을 파이썬 코드에서 실제로 어떻게 실행하는가?**
2. **"첫 번째 매칭만 찾기"와 "모든 매칭 찾기"는 어떻게 다르며, 각각 어떤 상황에서 사용하는가?**
3. **같은 패턴을 여러 번 사용할 때, 매번 패턴 문자열을 전달하는 것과 미리 컴파일하는 것은 어떤 차이가 있는가?**

---

## 개념 지도 (Concept Map)

```
[선수: 정규표현식 패턴 문법 (Ch1~4)]
    │
    │  "패턴은 만들 줄 안다. 이제 파이썬에서 실행하자!"
    │
    ▼
[신규: re 모듈] ─────────────────────────────────────────
    │
    ├── re.match()      ─ 문자열 시작에서 매칭
    ├── re.search()     ─ 문자열 전체에서 첫 매칭 탐색
    ├── re.fullmatch()  ─ 문자열 전체가 패턴과 일치하는지
    ├── re.findall()    ─ 모든 매칭을 리스트로 반환
    ├── re.finditer()   ─ 모든 매칭을 이터레이터로 반환
    ├── re.compile()    ─ 패턴 사전 컴파일 (패턴 객체)
    ├── re.escape()     ─ 메타문자 안전 이스케이프
    └── re.error        ─ 잘못된 패턴 예외 처리
    │
    ▼
[결과: 매치 객체 (Match object)] → Ch6에서 상세 학습
```

---

# Part B: 본문 (Main Content)

---

## 5.1 re 모듈 개요와 import

> 💡 **한 줄 요약**: `re` 모듈은 파이썬에서 정규표현식을 사용하기 위한 표준 라이브러리이며, **함수형 호출**과 **객체형 호출** 두 가지 방식을 제공합니다.

### 직관적 이해

지금까지 Part 1(Chapter 1~4)에서 우리는 정규표현식의 "언어"를 배웠습니다. `.`이 임의의 문자를 의미하고, `\d+`가 하나 이상의 숫자를 매칭하며, `[a-z]{3,5}`가 소문자 3~5개를 매칭한다는 것을 알고 있습니다.

하지만 이 패턴들을 배웠다고 해서, 파이썬 코드에서 바로 사용할 수 있는 것은 아닙니다. 마치 악보를 읽을 줄 아는 것과 실제로 피아노를 연주하는 것이 다르듯, **패턴 문법을 아는 것**과 **파이썬에서 패턴을 실행하는 것**은 별개의 기술입니다.

파이썬에서 정규표현식을 실행하려면 `re`라는 표준 라이브러리 모듈이 필요합니다. 이 모듈이 바로 우리가 작성한 패턴 문자열을 받아서, 실제 문자열에 대해 매칭·검색·추출 작업을 수행하는 **엔진 역할**을 합니다.

### 핵심 개념 설명

#### re 모듈 가져오기

`re` 모듈은 파이썬에 기본 내장되어 있으므로, 별도 설치 없이 `import`만 하면 됩니다.

```python
import re
```

> 📝 **re 모듈**: 파이썬 표준 라이브러리에 포함된 정규표현식 모듈. 패턴 컴파일, 매칭, 검색, 치환, 분할 등 정규표현식과 관련된 모든 기능을 제공합니다.

#### 두 가지 호출 방식

`re` 모듈을 사용하는 방식은 크게 두 가지입니다.

**방식 1: 함수형 호출** — `re` 모듈의 함수에 패턴 문자열과 대상 문자열을 직접 전달합니다.

```python
import re

result = re.search(r'\d+', '주문번호: 12345')
```

**방식 2: 객체형 호출** — `re.compile()`로 패턴을 미리 컴파일하여 **패턴 객체**를 만들고, 이 객체의 메서드를 호출합니다.

```python
import re

pattern = re.compile(r'\d+')       # 패턴 객체 생성
result = pattern.search('주문번호: 12345')  # 패턴 객체의 메서드 호출
```

> 📝 **패턴 객체 (compiled pattern)**: `re.compile()`이 반환하는 객체로, 정규표현식 패턴이 내부적으로 컴파일된 형태입니다. `search()`, `match()`, `findall()` 등 `re` 모듈 함수와 동일한 이름의 메서드를 가지고 있어, 패턴을 재사용할 때 편리합니다.

두 방식은 기능적으로 동일한 결과를 반환합니다. 차이점은 `compile()`에 대해 다루는 Section 5.4에서 자세히 살펴보겠습니다. 이 Chapter에서는 주로 **함수형 호출** 방식을 사용하면서 각 함수의 동작을 익히되, 필요한 곳에서 객체형 호출도 함께 보여드리겠습니다.

> 🔗 **연결**: 패턴 문자열을 작성할 때 `r''`(raw string)을 사용하는 이유는 Chapter 2의 Section 2.5에서 배웠습니다. 이 Chapter의 모든 예시에서도 패턴 문자열에 `r''`을 사용합니다.

### 상세 예시

> 🔍 **예시 5.1.1: 첫 번째 re 모듈 사용**
>
> **상황**: 문자열에서 연속된 숫자가 있는지 확인하고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "오늘 기온은 23도입니다"
> result = re.search(r'\d+', text)
>
> if result:
>     print("숫자를 찾았습니다:", result.group())  # "23"
> else:
>     print("숫자가 없습니다")
> ```
>
> `re.search()`는 매칭에 성공하면 **매치 객체**를 반환하고, 실패하면 `None`을 반환합니다. 매치 객체의 `.group()` 메서드로 매칭된 문자열을 꺼낼 수 있습니다.
>
> **핵심 포인트**: `re.search(패턴, 문자열)` → 매치 객체 또는 `None` 반환. 매치 객체에서 `.group()`으로 결과를 추출합니다.

> 🔍 **예시 5.1.2: 함수형 vs 객체형 비교**
>
> **상황**: 같은 패턴을 두 방식으로 실행하여 결과가 동일함을 확인한다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "Error 404: Page not found"
>
> # 함수형 호출
> result1 = re.search(r'[A-Z][a-z]+', text)
> print(result1.group())  # "Error"
>
> # 객체형 호출
> pattern = re.compile(r'[A-Z][a-z]+')
> result2 = pattern.search(text)
> print(result2.group())  # "Error"
> ```
>
> **핵심 포인트**: 두 방식의 결과는 완전히 동일합니다. 차이는 패턴을 재사용할 때 드러나며, 이는 Section 5.4에서 다룹니다.

> 💬 **참고**: 매치 객체의 `.group()` 메서드와 그 외 다양한 속성은 Chapter 6에서 자세히 다룹니다. 이번 Chapter에서는 `.group()`으로 매칭 결과를 꺼내는 것만 알면 충분합니다.

### Section 요약

- `re` 모듈은 파이썬에서 정규표현식을 실행하기 위한 표준 라이브러리이다
- `import re`로 가져오며, 별도 설치가 필요 없다
- **함수형 호출**(`re.search(패턴, 문자열)`)과 **객체형 호출**(`pattern.search(문자열)`) 두 방식이 있다
- 두 방식은 기능적으로 동일하며, 상황에 따라 선택하면 된다

---

## 5.2 `match()`와 `search()`

> 💡 **한 줄 요약**: `match()`는 문자열의 **시작 부분**에서만 패턴을 찾고, `search()`는 문자열 **전체를 훑어서** 첫 번째 매칭을 찾으며, `fullmatch()`는 문자열 **전체가** 패턴과 일치하는지 확인합니다.

### 직관적 이해

도서관에서 책을 찾는 상황을 상상해 봅시다.

- **`match()`**: 책장의 **맨 첫 번째 칸**만 확인합니다. 원하는 책이 첫 칸에 있으면 성공, 아니면 바로 포기합니다.
- **`search()`**: 책장을 **처음부터 끝까지** 훑으며, 원하는 책을 **처음 발견한 순간** 멈춥니다.
- **`fullmatch()`**: 책장에 **딱 그 책 한 권만** 있는지 확인합니다. 다른 책이 하나라도 섞여 있으면 실패입니다.

이 세 함수의 차이는 "문자열의 **어디에서** 패턴을 찾느냐"입니다. 같은 패턴이라도 어떤 함수를 사용하느냐에 따라 결과가 달라집니다.

### 핵심 개념 설명

#### re.match(pattern, string)

> 📝 **re.match()**: 문자열의 **시작(beginning)**에서 패턴과의 매칭을 시도합니다. 시작 부분이 패턴과 일치하면 매치 객체를 반환하고, 일치하지 않으면 `None`을 반환합니다.

```python
import re

# 문자열이 숫자로 시작하는가?
re.match(r'\d+', '123abc')    # 매치 객체 반환 (매칭: "123")
re.match(r'\d+', 'abc123')    # None (문자열 시작이 숫자가 아님)
```

`match()`의 핵심은 **항상 문자열의 위치 0(맨 처음)부터 매칭을 시도**한다는 점입니다. 문자열 중간이나 끝에 패턴이 있더라도, 시작 부분이 일치하지 않으면 `None`을 반환합니다.

단, `match()`는 문자열의 시작에서 매칭을 시작하지만, **패턴이 문자열 전체를 소비할 필요는 없습니다**. 시작 부분만 패턴과 일치하면 성공입니다.

```python
re.match(r'\d+', '123abc')  # 성공! "123"만 매칭하고 "abc"는 무시
```

#### re.search(pattern, string)

> 📝 **re.search()**: 문자열 **전체를 스캔**하여 패턴과 일치하는 **첫 번째 위치**를 찾습니다. 매칭을 발견하면 매치 객체를 반환하고, 끝까지 찾지 못하면 `None`을 반환합니다.

```python
import re

# 문자열 어디에서든 숫자를 찾는다
re.search(r'\d+', '123abc')    # 매치 객체 (매칭: "123", 위치 0)
re.search(r'\d+', 'abc123')    # 매치 객체 (매칭: "123", 위치 3)
re.search(r'\d+', 'abcdef')    # None (숫자가 없음)
```

`search()`는 문자열의 시작에서 매칭을 시도하고, 실패하면 한 칸 옮겨서 다시 시도하고, 또 실패하면 또 한 칸 옮기는 방식으로 문자열 끝까지 탐색합니다.

#### re.fullmatch(pattern, string)

> 📝 **re.fullmatch()**: 문자열 **전체가** 패턴과 정확히 일치하는지 확인합니다. 전체가 일치하면 매치 객체를 반환하고, 일부만 일치하거나 불일치하면 `None`을 반환합니다.

```python
import re

re.fullmatch(r'\d+', '123')      # 매치 객체 (전체가 숫자)
re.fullmatch(r'\d+', '123abc')   # None (전체가 숫자가 아님)
re.fullmatch(r'\d+', 'abc')      # None
```

`fullmatch()`는 입력값 검증에 특히 유용합니다. 예를 들어 사용자가 입력한 값이 "순수하게 숫자로만 이루어져 있는지" 확인할 때 적합합니다.

> 💬 **참고**: `fullmatch()`는 `^패턴$`처럼 앵커를 사용한 것과 비슷한 효과를 냅니다. 하지만 의도를 더 명확하게 전달하므로 입력 전체 검증 시에는 `fullmatch()`를 쓰는 것이 가독성 면에서 좋습니다.

#### 세 함수의 비교 정리

| 함수 | 매칭 위치 | 반환값 (성공) | 반환값 (실패) | 대표 용도 |
|------|-----------|--------------|--------------|-----------|
| `re.match()` | 문자열 시작만 | 매치 객체 | `None` | 시작 부분 확인 |
| `re.search()` | 문자열 전체 탐색 | 매치 객체 | `None` | 패턴이 어딘가에 있는지 탐색 |
| `re.fullmatch()` | 문자열 전체 일치 | 매치 객체 | `None` | 입력값 형식 검증 |

### 상세 예시

> 🔍 **예시 5.2.1: match()와 search()의 동작 차이**
>
> **상황**: 로그 메시지에서 "ERROR"라는 단어를 찾고 싶다.
>
> **풀이/설명**:
> ```python
> import re
>
> log = "2024-01-15 ERROR: 서버 연결 실패"
>
> # match(): 문자열이 "ERROR"로 시작하는가?
> result1 = re.match(r'ERROR', log)
> print(result1)  # None — "2024-01-15 ..."로 시작하므로 실패
>
> # search(): 문자열 어딘가에 "ERROR"가 있는가?
> result2 = re.search(r'ERROR', log)
> print(result2.group())  # "ERROR" — 위치 11에서 발견
> ```
>
> **핵심 포인트**: `match()`는 시작 부분만 보기 때문에, 문자열 중간에 있는 패턴을 찾지 못합니다. 특정 패턴이 문자열 *어딘가에* 있는지 확인하려면 `search()`를 사용해야 합니다.

> 🔍 **예시 5.2.2: fullmatch()를 이용한 입력 검증**
>
> **상황**: 사용자가 입력한 우편번호가 "숫자 5자리" 형식인지 검증한다.
>
> **풀이/설명**:
> ```python
> import re
>
> def is_valid_zipcode(code):
>     return re.fullmatch(r'\d{5}', code) is not None
>
> print(is_valid_zipcode("12345"))      # True
> print(is_valid_zipcode("1234"))       # False (4자리)
> print(is_valid_zipcode("123456"))     # False (6자리)
> print(is_valid_zipcode("1234a"))      # False (숫자가 아닌 문자 포함)
> print(is_valid_zipcode("abc12345"))   # False (앞에 문자 포함)
> ```
>
> **핵심 포인트**: `fullmatch()`는 문자열 전체가 패턴과 정확히 일치하는지 확인하므로, 입력값 형식 검증에 가장 적합합니다. `search()`를 사용하면 `"abc12345"` 같은 입력에서도 `"12345"` 부분이 매칭되어 잘못된 결과를 줄 수 있습니다.

> 🔍 **예시 5.2.3: match()가 유용한 경우**
>
> **상황**: 여러 줄의 로그에서, 각 줄이 타임스탬프(`[YYYY-MM-DD]`)로 시작하는지 확인한다.
>
> **풀이/설명**:
> ```python
> import re
>
> lines = [
>     "[2024-01-15] 서버 시작",
>     "ERROR: 연결 실패",
>     "[2024-01-15] 요청 처리 완료",
> ]
>
> for line in lines:
>     if re.match(r'\[\d{4}-\d{2}-\d{2}\]', line):
>         print(f"정상 형식: {line}")
>     else:
>         print(f"비정상 형식: {line}")
> ```
> 출력:
> ```
> 정상 형식: [2024-01-15] 서버 시작
> 비정상 형식: ERROR: 연결 실패
> 정상 형식: [2024-01-15] 요청 처리 완료
> ```
>
> **핵심 포인트**: 각 줄이 특정 형식으로 **시작하는지** 확인할 때는 `match()`가 자연스럽습니다.

### 흔한 실수와 주의사항

> ⚠️ **주의: match()를 search()처럼 사용하는 실수**
>
> 가장 흔한 실수는 `re.match()`가 문자열 전체를 탐색한다고 착각하는 것입니다.
>
> ```python
> # ❌ 흔한 실수: match()로 문자열 중간의 패턴을 찾으려 함
> text = "주문번호는 A-12345입니다"
> result = re.match(r'A-\d+', text)
> print(result)  # None — "주문번호는..."으로 시작하므로 실패!
>
> # ✅ 올바른 방법: search() 사용
> result = re.search(r'A-\d+', text)
> print(result.group())  # "A-12345"
> ```
>
> **기억하세요**: "어딘가에 있는지 찾기" → `search()`, "시작 부분 확인" → `match()`, "전체 일치 확인" → `fullmatch()`

> ⚠️ **주의: match()와 fullmatch()의 차이**
>
> `match()`는 시작 부분만 일치하면 성공하지만, `fullmatch()`는 전체가 일치해야 합니다.
>
> ```python
> re.match(r'\d+', '123abc')      # 성공 (시작의 "123" 매칭)
> re.fullmatch(r'\d+', '123abc')  # 실패 (전체가 숫자가 아님)
> ```

### Section 요약

- `re.match()`: 문자열의 **시작**에서만 매칭 시도. 시작이 맞으면 성공, 아니면 `None`
- `re.search()`: 문자열 **전체를 스캔**하여 **첫 번째** 매칭 발견. 가장 범용적인 탐색 함수
- `re.fullmatch()`: 문자열 **전체가** 패턴과 일치하는지 확인. 입력 검증에 이상적
- 세 함수 모두 성공 시 **매치 객체**, 실패 시 **`None`** 반환

---

## 5.3 `findall()`과 `finditer()`

> 💡 **한 줄 요약**: `findall()`은 패턴에 매칭되는 **모든 결과를 리스트**로, `finditer()`는 **모든 결과를 이터레이터**로 반환합니다.

### 직관적 이해

`match()`와 `search()`는 모두 **단 하나의 매칭**만 찾습니다. `search()`조차 첫 번째 매칭을 발견하면 거기서 멈춥니다.

하지만 실전에서는 "문자열에 있는 **모든** 이메일 주소를 찾아라", "텍스트에서 **모든** 숫자를 추출하라"처럼 **전부** 찾아야 하는 경우가 훨씬 많습니다. 이럴 때 사용하는 것이 `findall()`과 `finditer()`입니다.

비유하자면, `search()`가 "금속탐지기로 해변을 걸으며 첫 번째 동전을 찾으면 멈추는 것"이라면, `findall()`은 "해변 전체를 훑어서 **모든 동전을 수거해 바구니에 담는 것**"입니다.

### 핵심 개념 설명

#### re.findall(pattern, string)

> 📝 **re.findall()**: 문자열 전체를 스캔하여, 패턴에 매칭되는 **모든 부분 문자열**을 찾아 **리스트(list)**로 반환합니다. 매칭이 없으면 빈 리스트 `[]`를 반환합니다.

```python
import re

text = "전화: 010-1234-5678, 팩스: 02-987-6543"
numbers = re.findall(r'\d+', text)
print(numbers)  # ['010', '1234', '5678', '02', '987', '6543']
```

`findall()`의 핵심 특징은 매치 객체가 아닌 **문자열의 리스트**를 반환한다는 점입니다. 덕분에 결과를 바로 반복문이나 리스트 연산에 활용할 수 있어 매우 편리합니다.

> ⚠️ **주의**: `findall()`에 **캡처 그룹 `()`**이 포함된 패턴을 사용하면, 전체 매칭이 아닌 **캡처 그룹 내용만** 반환됩니다. 이 동작은 Chapter 6에서 자세히 다루므로, 지금은 캡처 그룹 없는 패턴에 집중합니다.

#### re.finditer(pattern, string)

> 📝 **re.finditer()**: `findall()`과 동일하게 모든 매칭을 찾지만, 리스트 대신 **매치 객체의 이터레이터(iterator)**를 반환합니다. 각 매칭의 위치 정보와 상세 정보에 접근할 수 있습니다.

```python
import re

text = "전화: 010-1234-5678, 팩스: 02-987-6543"
for match in re.finditer(r'\d+', text):
    print(f"'{match.group()}' (위치: {match.start()}~{match.end()})")
```

출력:
```
'010' (위치: 4~7)
'1234' (위치: 8~12)
'5678' (위치: 13~17)
'02' (위치: 23~25)
'987' (위치: 26~29)
'6543' (위치: 30~34)
```

> 📝 **이터레이터 (iterator)**: 값을 하나씩 순서대로 꺼내올 수 있는 객체입니다. `for` 루프로 순회하며 각 항목을 처리할 수 있습니다. 리스트와 달리 모든 결과를 한꺼번에 메모리에 올리지 않으므로, 대용량 데이터 처리 시 효율적입니다.

#### findall() vs finditer() 비교

| 특성 | `findall()` | `finditer()` |
|------|------------|-------------|
| **반환 타입** | `list` (문자열 리스트) | `iterator` (매치 객체) |
| **위치 정보** | ❌ 없음 | ✅ `start()`, `end()`, `span()` |
| **매치 객체 접근** | ❌ 불가 | ✅ 각 매칭의 매치 객체 |
| **메모리 효율** | 모든 결과를 한 번에 메모리에 저장 | 필요할 때 하나씩 생성 |
| **편의성** | 결과를 바로 리스트로 활용 가능 | `for` 루프 필요 |
| **매칭 없을 때** | `[]` (빈 리스트) | 빈 이터레이터 (반복 안 됨) |

### 상세 예시

> 🔍 **예시 5.3.1: 텍스트에서 모든 단어 추출**
>
> **상황**: 영문 텍스트에서 모든 단어를 추출하여 리스트로 만든다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "Hello, World! Python is great."
> words = re.findall(r'[A-Za-z]+', text)
> print(words)  # ['Hello', 'World', 'Python', 'is', 'great']
> ```
>
> `\w+`를 사용하면 한글, 숫자, 언더스코어까지 포함되므로, 순수 영문 단어만 추출하려면 `[A-Za-z]+`가 적합합니다.
>
> **핵심 포인트**: `findall()`은 결과를 리스트로 바로 반환하므로, 후속 처리(정렬, 카운팅, 필터링)가 간편합니다.

> 🔍 **예시 5.3.2: finditer()로 매칭 위치 추적**
>
> **상황**: 문서에서 "TODO"라는 키워드가 등장하는 모든 위치를 추적한다.
>
> **풀이/설명**:
> ```python
> import re
>
> document = "TODO: 로그인 구현. 테스트 완료. TODO: 에러 처리. TODO: 문서화."
>
> print("TODO 항목 발견 위치:")
> for i, match in enumerate(re.finditer(r'TODO', document), 1):
>     print(f"  #{i}: 위치 {match.start()} - '{document[match.start():match.start()+20]}...'")
> ```
> 출력:
> ```
> TODO 항목 발견 위치:
>   #1: 위치 0 - 'TODO: 로그인 구현. 테스트...'
>   #2: 위치 20 - 'TODO: 에러 처리. TODO...'
>   #3: 위치 31 - 'TODO: 문서화....'
> ```
>
> **핵심 포인트**: `finditer()`는 각 매칭의 **위치 정보**까지 알 수 있어, 단순한 추출을 넘어 텍스트 분석에 활용할 수 있습니다.

> 🔍 **예시 5.3.3: findall()로 간단한 통계 분석**
>
> **상황**: 로그에서 HTTP 상태 코드를 모두 추출하여 발생 빈도를 분석한다.
>
> **풀이/설명**:
> ```python
> import re
>
> log_data = """
> GET /index.html 200
> GET /about.html 200
> POST /login 401
> GET /dashboard 200
> GET /missing 404
> POST /submit 500
> GET /index.html 200
> """
>
> # 줄 끝의 3자리 숫자를 상태 코드로 추출
> status_codes = re.findall(r'\b\d{3}$', log_data, re.MULTILINE)
> print("상태 코드:", status_codes)
> # ['200', '200', '401', '200', '404', '500', '200']
>
> # 빈도 계산
> from collections import Counter
> counts = Counter(status_codes)
> for code, count in counts.most_common():
>     print(f"  {code}: {count}회")
> ```
> 출력:
> ```
> 상태 코드: ['200', '200', '401', '200', '404', '500', '200']
>   200: 4회
>   401: 1회
>   404: 1회
>   500: 1회
> ```
>
> **핵심 포인트**: `findall()`의 결과 리스트를 `Counter` 같은 파이썬 도구와 결합하면, 강력한 텍스트 분석이 가능합니다.

> 💬 **참고**: 예시 5.3.3에서 사용한 `re.MULTILINE`은 여러 줄 모드를 활성화하는 **플래그**로, Chapter 10에서 자세히 다룹니다. 여기서는 "여러 줄 텍스트에서 각 줄의 끝(`$`)을 인식하게 해주는 옵션"이라고만 이해하면 됩니다.

### 흔한 실수와 주의사항

> ⚠️ **주의: findall() 결과가 빈 리스트일 때**
>
> `findall()`은 매칭이 없을 때 `None`이 아닌 **빈 리스트 `[]`**를 반환합니다. 이 점은 `match()`/`search()`와 다릅니다.
>
> ```python
> result = re.findall(r'\d+', 'no numbers here')
> print(result)       # []  (None이 아님!)
> print(len(result))  # 0
>
> # 매칭 유무 확인
> if result:           # 빈 리스트는 False로 평가됨
>     print("숫자 발견")
> else:
>     print("숫자 없음")  # 이 줄이 실행됨
> ```

> ⚠️ **주의: finditer()는 한 번만 순회 가능**
>
> 이터레이터는 한 번 순회하면 소진됩니다. 결과를 여러 번 사용해야 한다면, `list()`로 변환하거나 `findall()`을 사용하세요.
>
> ```python
> matches = re.finditer(r'\d+', '1 and 2 and 3')
>
> # 첫 번째 순회 — 정상 동작
> for m in matches:
>     print(m.group(), end=' ')  # 1 2 3
>
> print()
>
> # 두 번째 순회 — 아무것도 출력되지 않음!
> for m in matches:
>     print(m.group(), end=' ')  # (아무 출력 없음)
> ```

### Section 요약

- `re.findall()`은 모든 매칭 결과를 **문자열 리스트**로 반환한다. 매칭 없으면 `[]` 반환
- `re.finditer()`는 모든 매칭 결과를 **매치 객체 이터레이터**로 반환한다
- 단순 추출에는 `findall()`, 위치 정보가 필요하면 `finditer()` 사용
- 이터레이터는 한 번만 순회 가능하다는 점에 유의

---

## 5.4 `compile()`과 패턴 객체

> 💡 **한 줄 요약**: `re.compile()`은 정규표현식 패턴 문자열을 미리 컴파일하여 **패턴 객체**로 만들어두는 함수로, 같은 패턴을 반복 사용할 때 코드를 깔끔하게 정리할 수 있습니다.

### 직관적 이해

요리 비유로 생각해 봅시다. 매번 요리할 때마다 레시피 책을 펼쳐서 재료 목록을 읽고, 해석하고, 준비하는 것은 번거롭습니다. 대신, 자주 만드는 요리의 레시피를 **카드에 정리해두면** — 재료를 미리 파악하고 순서를 외워두면 — 다음번에는 카드만 보고 바로 요리에 들어갈 수 있습니다.

`re.compile()`이 바로 이 "레시피 카드 정리" 과정입니다. 패턴 문자열을 한 번 **컴파일**(해석·변환)하여 패턴 객체로 만들어두면, 이후에는 이 객체를 바로 사용할 수 있습니다.

### 핵심 개념 설명

#### 기본 사용법

```python
import re

# 패턴을 컴파일하여 패턴 객체 생성
email_pattern = re.compile(r'[\w.+-]+@[\w-]+\.[\w.]+')

# 패턴 객체의 메서드를 사용
result1 = email_pattern.search("연락처: user@example.com")
result2 = email_pattern.findall("a@b.com 그리고 c@d.org")
result3 = email_pattern.match("test@mail.com은 유효합니다")
```

`re.compile()`이 반환하는 패턴 객체는 `re` 모듈의 함수들과 동일한 이름의 메서드를 갖고 있습니다.

| `re` 모듈 함수 | 패턴 객체 메서드 |
|----------------|-----------------|
| `re.search(pattern, string)` | `pattern_obj.search(string)` |
| `re.match(pattern, string)` | `pattern_obj.match(string)` |
| `re.fullmatch(pattern, string)` | `pattern_obj.fullmatch(string)` |
| `re.findall(pattern, string)` | `pattern_obj.findall(string)` |
| `re.finditer(pattern, string)` | `pattern_obj.finditer(string)` |

패턴 객체의 메서드를 호출할 때는 패턴 문자열을 다시 전달할 필요가 없습니다 — 이미 객체 안에 컴파일되어 있기 때문입니다.

#### compile()의 장점

**1. 코드 가독성 향상**

패턴에 의미 있는 변수명을 부여할 수 있습니다.

```python
# 함수형: 패턴의 의도를 파악하려면 패턴 자체를 읽어야 함
result = re.search(r'\b\d{3}-\d{4}-\d{4}\b', user_input)

# 객체형: 변수명으로 패턴의 의도를 즉시 알 수 있음
phone_pattern = re.compile(r'\b\d{3}-\d{4}-\d{4}\b')
result = phone_pattern.search(user_input)
```

**2. 패턴 재사용의 편리함**

같은 패턴을 여러 곳에서 사용할 때, 패턴 문자열을 반복 작성할 필요가 없습니다.

```python
phone_pattern = re.compile(r'\b\d{3}-\d{4}-\d{4}\b')

# 여러 텍스트에 같은 패턴 적용
for document in documents:
    phones = phone_pattern.findall(document)
    if phones:
        print(f"전화번호 발견: {phones}")
```

**3. 패턴 객체의 유용한 속성**

패턴 객체는 컴파일에 사용된 원래 패턴 문자열과 플래그 정보를 속성으로 가지고 있습니다.

```python
pattern = re.compile(r'\d{3}-\d{4}')

print(pattern.pattern)   # '\\d{3}-\\d{4}' (원본 패턴 문자열)
print(pattern.flags)     # 32 (플래그 값 — 기본값)
```

### 상세 예시

> 🔍 **예시 5.4.1: 반복 검색에 compile() 활용**
>
> **상황**: 고객 피드백 목록에서 이메일 주소가 포함된 피드백을 모두 찾는다.
>
> **풀이/설명**:
> ```python
> import re
>
> feedbacks = [
>     "좋은 서비스입니다. 문의: kim@company.co.kr",
>     "배송이 느립니다.",
>     "추가 문의는 lee@service.com으로 주세요.",
>     "만족합니다!",
>     "환불 요청합니다. 연락처: park@mail.net",
> ]
>
> # 패턴을 한 번만 컴파일
> email_pattern = re.compile(r'[\w.+-]+@[\w-]+\.[\w.]+')
>
> for fb in feedbacks:
>     emails = email_pattern.findall(fb)
>     if emails:
>         print(f"이메일 포함 피드백: '{fb[:20]}...' → {emails}")
> ```
> 출력:
> ```
> 이메일 포함 피드백: '좋은 서비스입니다. 문의: kim...' → ['kim@company.co.kr']
> 이메일 포함 피드백: '추가 문의는 lee@service...' → ['lee@service.com']
> 이메일 포함 피드백: '환불 요청합니다. 연락처: pa...' → ['park@mail.net']
> ```
>
> **핵심 포인트**: `email_pattern`은 한 번 컴파일되어 5개의 피드백에 반복 사용됩니다. 패턴 문자열을 매번 전달하는 것보다 코드가 깔끔합니다.

> 🔍 **예시 5.4.2: 여러 패턴을 한꺼번에 관리**
>
> **상황**: 텍스트에서 여러 종류의 데이터(날짜, 금액, 전화번호)를 동시에 추출한다.
>
> **풀이/설명**:
> ```python
> import re
>
> # 여러 패턴을 사전에 정의
> patterns = {
>     '날짜': re.compile(r'\d{4}-\d{2}-\d{2}'),
>     '금액': re.compile(r'\d{1,3}(?:,\d{3})*원'),
>     '전화번호': re.compile(r'\d{2,3}-\d{3,4}-\d{4}'),
> }
>
> text = "주문일: 2024-03-15, 금액: 52,000원, 연락처: 010-1234-5678"
>
> for name, pattern in patterns.items():
>     found = pattern.findall(text)
>     if found:
>         print(f"{name}: {found}")
> ```
> 출력:
> ```
> 날짜: ['2024-03-15']
> 금액: ['52,000원']
> 전화번호: ['010-1234-5678']
> ```
>
> **핵심 포인트**: `compile()`을 사용하면 패턴에 이름을 붙여 사전(dict)으로 관리할 수 있어, 코드의 구조화에 유리합니다.

> 💬 **참고**: 예시 5.4.2의 금액 패턴에서 `(?:,\d{3})*`라는 표현이 보입니다. `(?:...)`는 "비캡처 그룹"이라 하여, 그룹핑은 하되 캡처는 하지 않는 문법입니다. 이 개념은 Chapter 8에서 자세히 다룹니다. 지금은 "쉼표+숫자3자리"를 0회 이상 반복한다는 의미로만 이해하면 됩니다.

### 흔한 실수와 주의사항

> ⚠️ **주의: compile() 없이도 내부적으로 컴파일은 일어난다**
>
> `re.search(r'\d+', text)`처럼 함수형으로 호출해도, 파이썬은 내부적으로 패턴을 컴파일합니다. 또한 `re` 모듈은 최근에 사용된 패턴을 **캐시(cache)**에 저장하여 재컴파일을 피합니다(기본 캐시 크기: 512개).
>
> 따라서 `compile()`의 주된 이점은 성능보다는 **코드 가독성과 구조화**에 있습니다. 다만, 다음의 경우에는 `compile()`을 사용하는 것이 확실히 좋습니다:
> - 같은 패턴을 루프 안에서 수천~수만 번 반복 사용할 때
> - 패턴에 의미 있는 이름을 부여하여 코드 가독성을 높이고 싶을 때
> - 여러 패턴을 변수나 자료구조로 관리하고 싶을 때

### Section 요약

- `re.compile(패턴)`은 패턴 문자열을 컴파일하여 **패턴 객체**를 반환한다
- 패턴 객체는 `search()`, `match()`, `findall()` 등과 동일한 메서드를 가진다
- `compile()`의 주요 이점은 코드 **가독성**, 패턴 **재사용**, 패턴의 **구조적 관리**이다
- 함수형 호출도 내부적으로 컴파일과 캐싱이 일어나므로, 성능 차이만을 위해 `compile()`을 쓸 필요는 적다

---

## 5.5 `re.escape()`와 `re.error`

> 💡 **한 줄 요약**: `re.escape()`는 문자열의 모든 메타문자를 자동으로 이스케이프하여 리터럴 검색에 안전하게 만들어주고, `re.error`는 잘못된 패턴에 대한 예외를 제공합니다.

### 직관적 이해

정규표현식에서 `.`, `*`, `+`, `?`, `(`, `)` 같은 문자들은 특별한 의미를 가진 **메타문자**입니다.

> 🔗 **연결**: 메타문자의 정의와 종류는 Chapter 2에서 배웠습니다.

그런데 사용자가 검색어를 직접 입력하는 상황을 생각해 봅시다. 사용자가 "C++"을 검색하고 싶어서 입력했는데, `+`가 수량자로 해석되면 엉뚱한 결과가 나오거나 에러가 발생합니다. 또, 사용자가 "(주)"라는 텍스트를 검색하면 `(`, `)`가 그룹 문법으로 해석됩니다.

`re.escape()`는 이런 문제를 해결합니다. 문자열 안의 모든 메타문자 앞에 `\`를 자동으로 붙여서, 모든 문자가 **리터럴(있는 그대로의 문자)**로 해석되게 만들어줍니다.

한편, 사용자가 잘못된 패턴(예: 괄호 짝이 맞지 않는 `(abc`)을 입력하면 `re.error` 예외가 발생합니다. 이를 적절히 처리하면 프로그램이 비정상 종료되는 것을 막을 수 있습니다.

### 핵심 개념 설명

#### re.escape(pattern)

> 📝 **re.escape()**: 문자열 내의 모든 정규표현식 메타문자 앞에 백슬래시(`\`)를 추가하여, 해당 문자열이 리터럴 텍스트로 매칭되도록 변환합니다.

```python
import re

# 메타문자가 포함된 문자열
text = "가격은 $100.00 (할인가)"

escaped = re.escape(text)
print(escaped)  # '가격은\\ \\$100\\.00\\ \\(할인가\\)'
```

`re.escape()`가 변환하는 주요 문자들: `. ^ $ * + ? { } [ ] \ | ( )`

**활용 시나리오**: 사용자 입력을 정규표현식 패턴에 포함시킬 때

```python
import re

def find_exact_text(search_term, document):
    """사용자가 입력한 텍스트를 정확히 찾는 함수"""
    safe_pattern = re.escape(search_term)
    return re.findall(safe_pattern, document)

# "C++" 검색 — re.escape() 없이는 + 가 수량자로 해석되어 문제 발생
results = find_exact_text("C++", "C++ 개발자, C++ 표준, C# 개발자")
print(results)  # ['C++', 'C++']
```

#### re.error

> 📝 **re.error**: 잘못된 정규표현식 패턴을 컴파일하거나 사용하려 할 때 발생하는 예외(exception)입니다. 에러 메시지에는 어떤 문제가 발생했는지, 패턴의 어느 위치에서 문제가 발생했는지에 대한 정보가 포함됩니다.

```python
import re

try:
    re.search(r'(abc', 'test')  # 괄호가 닫히지 않음
except re.error as e:
    print(f"패턴 오류: {e}")
    # 패턴 오류: missing ), unterminated subpattern at position 0
```

`re.error` 예외 객체의 주요 속성:

| 속성 | 설명 |
|------|------|
| `e.msg` | 에러 메시지 |
| `e.pattern` | 문제가 된 패턴 문자열 |
| `e.pos` | 에러가 발생한 위치 (인덱스) |

```python
try:
    re.compile(r'[a-z')
except re.error as e:
    print(f"메시지: {e.msg}")       # unterminated character set
    print(f"패턴: {e.pattern}")     # [a-z
    print(f"위치: {e.pos}")         # 0
```

### 상세 예시

> 🔍 **예시 5.5.1: 사용자 입력을 안전하게 검색**
>
> **상황**: 문서에서 사용자가 입력한 검색어를 찾는 검색 기능을 구현한다. 검색어에 메타문자가 포함될 수 있다.
>
> **풀이/설명**:
> ```python
> import re
>
> document = """
> FAQ:
> Q: 환불이 가능한가요? (주문 후 7일 이내)
> A: 네, 가능합니다. 자세한 내용은 help@shop.com으로 문의하세요.
> Q: 가격이 $50.00인가요?
> A: 네, 맞습니다.
> """
>
> def safe_search(query, text):
>     """메타문자를 안전하게 처리하는 검색 함수"""
>     safe_query = re.escape(query)
>     matches = re.findall(safe_query, text)
>     return matches
>
> # 메타문자가 포함된 검색어들
> print(safe_search("$50.00", document))         # ['$50.00']
> print(safe_search("(주문 후 7일 이내)", document))  # ['(주문 후 7일 이내)']
> print(safe_search("help@shop.com", document))  # ['help@shop.com']
> ```
>
> **핵심 포인트**: `re.escape()`를 쓰지 않으면, `$50.00`에서 `$`는 앵커로, `.`는 와일드카드로 해석되어 원하지 않는 결과가 나옵니다.

> 🔍 **예시 5.5.2: 패턴 오류의 안전한 처리**
>
> **상황**: 사용자가 정규표현식 패턴을 직접 입력할 수 있는 프로그램에서, 잘못된 패턴 입력 시 안전하게 에러를 처리한다.
>
> **풀이/설명**:
> ```python
> import re
>
> def safe_regex_search(pattern_str, text):
>     """사용자 입력 패턴을 안전하게 처리하는 검색 함수"""
>     try:
>         results = re.findall(pattern_str, text)
>         return results
>     except re.error as e:
>         print(f"⚠️ 잘못된 정규표현식입니다: {e.msg}")
>         print(f"   패턴: '{e.pattern}', 위치: {e.pos}")
>         return None
>
> text = "Hello World 123"
>
> # 정상적인 패턴
> print(safe_regex_search(r'\d+', text))       # ['123']
>
> # 잘못된 패턴들
> print(safe_regex_search(r'[abc', text))       # 에러 메시지 출력 후 None
> print(safe_regex_search(r'*abc', text))       # 에러 메시지 출력 후 None
> print(safe_regex_search(r'(?P<name', text))   # 에러 메시지 출력 후 None
> ```
>
> **핵심 포인트**: `try-except re.error`로 감싸면, 잘못된 패턴이 들어와도 프로그램이 비정상 종료되지 않고 에러를 우아하게 처리할 수 있습니다.

### 흔한 실수와 주의사항

> ⚠️ **주의: re.escape()와 re.error의 사용 구분**
>
> - 사용자 입력을 **리터럴 텍스트로 검색**하고 싶다면 → `re.escape()` 사용
> - 사용자 입력을 **정규표현식 패턴으로 실행**하고 싶다면 → `try-except re.error` 사용
>
> 이 둘의 용도를 혼동하지 마세요.
>
> ```python
> # 시나리오 1: "C++"이라는 텍스트를 정확히 찾고 싶다
> safe = re.escape("C++")   # 'C\\+\\+'
> re.findall(safe, "C++ 개발자")  # ['C++'] ✅
>
> # 시나리오 2: 사용자가 정규표현식을 직접 쓰고 싶다
> user_pattern = r'\d{3}-\d{4}'  # 사용자가 입력한 패턴
> try:
>     re.findall(user_pattern, "전화: 010-1234")  # ✅
> except re.error:
>     print("잘못된 패턴입니다")
> ```

### Section 요약

- `re.escape(문자열)`은 모든 메타문자를 이스케이프하여 **리터럴 검색**을 안전하게 만든다
- 사용자 입력을 패턴에 포함시킬 때 반드시 `re.escape()`를 사용해야 한다
- `re.error`는 잘못된 패턴에 대한 예외이며, `try-except`로 처리할 수 있다
- `re.error` 객체의 `msg`, `pattern`, `pos` 속성으로 오류 원인을 진단할 수 있다

---

## 5.6 함수 선택 가이드

> 💡 **한 줄 요약**: 상황에 맞는 `re` 함수를 고르는 것은 올바른 패턴을 작성하는 것만큼 중요합니다.

### 직관적 이해

지금까지 `match()`, `search()`, `fullmatch()`, `findall()`, `finditer()`, `compile()`, `escape()` 등 여러 함수를 배웠습니다. 각각의 동작을 이해하는 것도 중요하지만, 실전에서는 **"지금 이 상황에서 어떤 함수를 써야 하지?"**라는 판단이 더 중요합니다.

이 Section에서는 자주 마주치는 상황별로 어떤 함수를 선택해야 하는지를 정리합니다.

### 핵심 개념 설명

#### 상황별 함수 선택 플로우차트

```
질문: 무엇을 하고 싶은가?
│
├─ "패턴이 있는지 확인만 하고 싶다" (True/False)
│   ├─ 문자열 시작 부분만? → re.match() 후 is not None 체크
│   ├─ 문자열 전체 일치?  → re.fullmatch() 후 is not None 체크
│   └─ 어딘가에 있는지?   → re.search() 후 is not None 체크
│
├─ "매칭된 내용을 추출하고 싶다"
│   ├─ 첫 번째 매칭만?   → re.search() → .group()
│   ├─ 모든 매칭, 리스트로? → re.findall()
│   └─ 모든 매칭, 위치 정보 포함? → re.finditer()
│
├─ "같은 패턴을 반복 사용한다"
│   └─ re.compile()로 패턴 객체 생성 후 메서드 호출
│
├─ "사용자 입력을 리터럴로 검색하고 싶다"
│   └─ re.escape()로 메타문자 이스케이프 후 검색
│
└─ "사용자 입력을 패턴으로 실행하고 싶다"
    └─ try-except re.error로 감싸서 안전하게 실행
```

#### 함수 전체 비교표

| 함수 | 탐색 범위 | 반환 (성공) | 반환 (실패) | 대표 용도 |
|------|-----------|------------|------------|-----------|
| `match()` | 시작만 | 매치 객체 | `None` | 시작 형식 확인 |
| `search()` | 전체 (첫 매칭) | 매치 객체 | `None` | 패턴 존재 여부, 첫 추출 |
| `fullmatch()` | 전체 일치 | 매치 객체 | `None` | 입력값 형식 검증 |
| `findall()` | 전체 (모든 매칭) | 문자열 리스트 | `[]` | 모든 매칭 추출 |
| `finditer()` | 전체 (모든 매칭) | 매치 객체 이터레이터 | 빈 이터레이터 | 모든 매칭 + 위치 정보 |
| `compile()` | — | 패턴 객체 | — | 패턴 재사용, 코드 정리 |
| `escape()` | — | 이스케이프된 문자열 | — | 리터럴 검색 안전 처리 |

### 상세 예시

> 🔍 **예시 5.6.1: 실전 상황별 함수 선택**
>
> **상황**: 다양한 실전 요구사항에 대해 적합한 함수를 선택한다.
>
> **풀이/설명**:
> ```python
> import re
>
> # --- 상황 1: 사용자 입력이 유효한 이메일 형식인지 확인 ---
> email = "user@example.com"
> if re.fullmatch(r'[\w.+-]+@[\w-]+\.[\w.]+', email):
>     print("유효한 이메일")  # ✅ fullmatch: 전체 일치 검증
>
> # --- 상황 2: 로그 한 줄에서 첫 번째 IP 주소 추출 ---
> log_line = "접속: 192.168.1.100 → 서버 10.0.0.1"
> ip = re.search(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}', log_line)
> if ip:
>     print(f"첫 번째 IP: {ip.group()}")  # ✅ search: 첫 매칭 추출
>
> # --- 상황 3: 문서에서 모든 해시태그 추출 ---
> post = "오늘의 #일상 #맛집 방문기 #서울"
> tags = re.findall(r'#\w+', post)
> print(f"해시태그: {tags}")  # ✅ findall: 모든 매칭 리스트
>
> # --- 상황 4: CSV 줄이 올바른 형식으로 시작하는지 확인 ---
> csv_line = "2024-01-15,John,100"
> if re.match(r'\d{4}-\d{2}-\d{2}', csv_line):
>     print("날짜로 시작하는 행")  # ✅ match: 시작 형식 확인
>
> # --- 상황 5: 사용자가 입력한 텍스트를 정확히 찾기 ---
> query = "price: $9.99 (sale!)"
> text = "Current price: $9.99 (sale!) — limited offer"
> found = re.findall(re.escape(query), text)
> print(f"발견: {found}")  # ✅ escape: 메타문자 안전 처리
> ```
>
> **핵심 포인트**: 각 상황에서 가장 적합한 함수를 선택하는 습관을 들이세요. 잘못된 함수를 선택하면 기대와 다른 결과를 얻거나, 불필요하게 복잡한 코드가 됩니다.

> 🔍 **예시 5.6.2: 같은 작업을 다른 함수로 수행했을 때의 차이**
>
> **상황**: "문자열에 숫자가 포함되어 있는가?"라는 동일한 질문에 대해 여러 함수를 비교한다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "가격: 15,000원"
>
> # 방법 1: search() — 가장 자연스러운 선택 ✅
> if re.search(r'\d', text):
>     print("숫자 포함")
>
> # 방법 2: findall() — 작동하지만, 존재 확인에 비해 과함
> if re.findall(r'\d', text):  # ['1', '5', '0', '0', '0'] — 불필요한 리스트 생성
>     print("숫자 포함")
>
> # 방법 3: match() — ❌ 잘못된 선택
> if re.match(r'\d', text):  # None — "가격:"으로 시작하므로 실패
>     print("숫자 포함")  # 이 줄은 실행되지 않음!
> ```
>
> **핵심 포인트**: "어딘가에 있는지" 확인할 때는 `search()`가 정답입니다. `findall()`도 되지만, 단순 존재 확인에 모든 결과를 리스트로 만드는 것은 낭비입니다. `match()`는 시작 부분만 보기 때문에 이 용도에는 부적합합니다.

### Section 요약

- 입력값 **전체 형식 검증** → `fullmatch()`
- 패턴이 문자열 **어딘가에 있는지** 확인/추출 → `search()`
- 문자열 **시작 부분** 확인 → `match()`
- **모든 매칭**을 리스트로 → `findall()`
- **모든 매칭** + 위치 정보 → `finditer()`
- 패턴 **반복 사용** → `compile()`
- 사용자 입력을 **리터럴로 검색** → `escape()`

---

# Part C: 통합 및 정리 (Integration Block)

---

## Chapter 핵심 요약 (Chapter Summary)

이 Chapter에서는 Part 1(Chapter 1~4)에서 배운 정규표현식 패턴 문법을 **파이썬에서 실행**하기 위한 `re` 모듈의 핵심 함수들을 학습했습니다.

- **Section 5.1**: `re` 모듈을 `import re`로 가져오며, 함수형 호출(`re.search(패턴, 문자열)`)과 객체형 호출(`패턴객체.search(문자열)`) 두 방식을 제공합니다.
- **Section 5.2**: `match()`는 문자열 시작에서만, `search()`는 전체에서 첫 매칭을, `fullmatch()`는 전체 일치를 확인합니다. 세 함수 모두 매치 객체 또는 `None`을 반환합니다.
- **Section 5.3**: `findall()`은 모든 매칭을 문자열 리스트로, `finditer()`는 매치 객체 이터레이터로 반환합니다. 단일 매칭이 아닌 **전체 탐색**이 필요할 때 사용합니다.
- **Section 5.4**: `compile()`로 패턴을 사전 컴파일하면 코드 가독성이 높아지고, 패턴 재사용에 편리합니다.
- **Section 5.5**: `escape()`는 메타문자를 이스케이프하여 리터럴 검색을 안전하게 만들고, `re.error`로 잘못된 패턴의 예외를 처리합니다.
- **Section 5.6**: 상황별 적합한 함수를 선택하는 기준을 정리했습니다.

---

## 핵심 용어 정리 (Glossary)

| 용어 | 정의 | 관련 Section |
|------|------|-------------|
| `re` 모듈 | 파이썬 표준 라이브러리의 정규표현식 모듈 | 5.1 |
| 패턴 객체 (compiled pattern) | `re.compile()`이 반환하는, 패턴이 사전 컴파일된 객체 | 5.1, 5.4 |
| 매치 객체 (Match object) | 매칭 성공 시 반환되는 객체로, 매칭 결과와 위치 정보 포함 | 5.1, 5.2 |
| `re.match()` | 문자열 시작에서 패턴 매칭을 시도하는 함수 | 5.2 |
| `re.search()` | 문자열 전체에서 첫 번째 매칭을 찾는 함수 | 5.2 |
| `re.fullmatch()` | 문자열 전체가 패턴과 일치하는지 확인하는 함수 | 5.2 |
| `re.findall()` | 모든 매칭을 문자열 리스트로 반환하는 함수 | 5.3 |
| `re.finditer()` | 모든 매칭을 매치 객체 이터레이터로 반환하는 함수 | 5.3 |
| 이터레이터 (iterator) | 값을 하나씩 순차적으로 꺼낼 수 있는 객체 | 5.3 |
| `re.compile()` | 패턴 문자열을 사전 컴파일하여 패턴 객체를 생성하는 함수 | 5.4 |
| `re.escape()` | 메타문자를 자동 이스케이프하여 리터럴 검색을 안전하게 만드는 함수 | 5.5 |
| `re.error` | 잘못된 정규표현식 패턴 사용 시 발생하는 예외 클래스 | 5.5 |

---

## 개념 연결 맵 (Connection Map)

```
[Chapter 1~4에서 배운 것]                [이번 Chapter에서 배운 것]
━━━━━━━━━━━━━━━━━━━━━                 ━━━━━━━━━━━━━━━━━━━━━━
정규표현식 패턴 문법                      re 모듈 함수들
 • 메타문자 (. ^ $ | \)                  → match(), search(), fullmatch()
 • 문자 클래스 ([...], \d, \w)            → findall(), finditer()
 • 수량자 (*, +, ?, {n,m})               → compile(), escape()
                                         → re.error 예외 처리

        "패턴을 만들 수 있다"        →        "파이썬에서 실행할 수 있다"

[다음 Chapter에서 배울 것]
━━━━━━━━━━━━━━━━━━━━━━
Chapter 6: 매치 객체의 상세 활용
 • group(), groups(), start(), end(), span()
 • 캡처 그룹 () 기초
Chapter 7: 치환과 분할
 • re.sub(), re.subn(), re.split()
```

이전 Chapter들에서 패턴을 **작성하는 법**을 배웠고, 이번 Chapter에서 패턴을 **실행하는 법**을 배웠습니다. 다음 Chapter 6에서는 실행 결과인 **매치 객체**를 더 깊이 활용하는 방법을 배웁니다.

---

## 자기 점검 질문 (Self-Check Questions)

**Q1.** `re.match(r'\d+', 'abc123')`의 반환값은 무엇인가?

<details>
<summary>정답 확인</summary>

**`None`**. `match()`는 문자열의 시작에서만 매칭을 시도합니다. `'abc123'`은 `'a'`로 시작하므로 `\d+` 패턴과 일치하지 않습니다.
</details>

**Q2.** `re.search(r'\d+', 'abc123def456')`이 매칭하는 문자열은 무엇인가?

<details>
<summary>정답 확인</summary>

**`'123'`**. `search()`는 전체를 탐색하여 **첫 번째** 매칭을 반환합니다. `'456'`은 두 번째 매칭이므로 반환되지 않습니다.
</details>

**Q3.** `re.findall(r'[A-Z]', 'Hello World')`의 결과는?

<details>
<summary>정답 확인</summary>

**`['H', 'W']`**. `findall()`은 대문자 하나에 매칭되는 모든 결과를 리스트로 반환합니다.
</details>

**Q4.** `re.fullmatch(r'\d+', '123abc')`의 결과는 매치 객체인가, `None`인가?

<details>
<summary>정답 확인</summary>

**`None`**. `fullmatch()`는 문자열 **전체**가 패턴과 일치해야 성공합니다. `'123abc'`는 전체가 숫자가 아니므로 실패합니다.
</details>

**Q5.** `re.escape('a.b+c')`의 결과는?

<details>
<summary>정답 확인</summary>

**`'a\\.b\\+c'`**. `.`과 `+`는 메타문자이므로 앞에 `\`가 추가됩니다. `a`, `b`, `c`는 일반 문자이므로 그대로 유지됩니다.
</details>

**Q6.** `re.findall(r'\d+', '숫자 없는 문장')`의 반환값은 `None`인가, `[]`인가?

<details>
<summary>정답 확인</summary>

**`[]` (빈 리스트)**. `findall()`은 매칭이 없을 때 `None`이 아닌 빈 리스트를 반환합니다. `match()`와 `search()`만 실패 시 `None`을 반환합니다.
</details>

**Q7.** `re.compile()`을 사용하지 않고 `re.search()`를 여러 번 호출하면, 매번 패턴이 새로 컴파일되는가?

<details>
<summary>정답 확인</summary>

**아니요**. 파이썬의 `re` 모듈은 내부적으로 최근 사용된 패턴을 **캐시**에 저장합니다 (기본 512개). 같은 패턴 문자열로 반복 호출하면 캐시된 컴파일 결과를 재사용합니다. 다만, `compile()`을 사용하면 코드 가독성과 구조화 면에서 이점이 있습니다.
</details>

---

# Part D: 문제 풀이 (Problem Set)

---

## D1. Practice (연습 문제) — 기본 개념 확인

> **Practice 5.1**: 다음 코드의 출력 결과를 예측하세요.
> ```python
> import re
> print(re.match(r'hello', 'hello world'))
> print(re.match(r'world', 'hello world'))
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> - 첫 번째: `<re.Match object; ...>` (매치 객체 — 문자열이 `'hello'`로 시작하므로 성공)
> - 두 번째: `None` (`'world'`는 문자열 시작이 아닌 중간에 있으므로 `match()` 실패)
>
> **해설**: `match()`는 항상 문자열의 시작(위치 0)에서만 매칭을 시도합니다.
> </details>

> **Practice 5.2**: 문자열 `"Today is 2024-03-15 and tomorrow is 2024-03-16"`에서 모든 날짜를 추출하는 코드를 작성하세요. 날짜 형식은 `YYYY-MM-DD`입니다.
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> text = "Today is 2024-03-15 and tomorrow is 2024-03-16"
> dates = re.findall(r'\d{4}-\d{2}-\d{2}', text)
> print(dates)  # ['2024-03-15', '2024-03-16']
> ```
>
> **해설**: 모든 매칭을 리스트로 추출하므로 `findall()`이 적합합니다. 패턴 `\d{4}-\d{2}-\d{2}`는 "숫자4자리-숫자2자리-숫자2자리" 형식을 매칭합니다.
> </details>

> **Practice 5.3**: 다음 각 입력에 대해 `re.fullmatch(r'[a-z]+', 입력)`의 결과가 매치 객체인지, `None`인지 답하세요.
> - (a) `"hello"`
> - (b) `"Hello"`
> - (c) `"hello123"`
> - (d) `""`
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> - (a) **매치 객체** — 전체가 소문자 알파벳으로만 구성
> - (b) **`None`** — `'H'`가 대문자이므로 `[a-z]`에 포함되지 않음
> - (c) **`None`** — `'123'`이 소문자가 아님
> - (d) **`None`** — `+`는 1회 이상을 요구하므로 빈 문자열은 실패
>
> **해설**: `fullmatch()`는 문자열 전체가 패턴과 정확히 일치해야 하며, `[a-z]+`는 소문자 1개 이상을 요구합니다.
> </details>

> **Practice 5.4**: `re.escape('(주)한국상사')`의 결과를 예측하세요.
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `'\\(주\\)한국상사'`
>
> **해설**: `(`와 `)`는 정규표현식 메타문자(그룹핑)이므로 앞에 `\`가 추가됩니다. `주`, `한`, `국`, `상`, `사`는 일반 문자이므로 그대로 유지됩니다.
> </details>

> **Practice 5.5**: 다음 코드 중 에러가 발생하는 것을 모두 고르고, 에러 종류를 답하세요.
> ```python
> # (a)
> re.search(r'\d+', 'hello')
>
> # (b)
> re.search(r'[a-z', 'hello')
>
> # (c)
> re.findall(r'(abc', 'abcabc')
>
> # (d)
> re.findall(r'\d+', 'hello')
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: (b)와 (c)에서 `re.error` 예외 발생
>
> - (a): 에러 없음. 매칭이 없을 뿐이며, `None`을 반환합니다.
> - (b): **`re.error`** — `[a-z`에서 대괄호가 닫히지 않았습니다 (unterminated character set).
> - (c): **`re.error`** — `(abc`에서 소괄호가 닫히지 않았습니다 (missing ), unterminated subpattern).
> - (d): 에러 없음. 매칭이 없으므로 빈 리스트 `[]`를 반환합니다.
>
> **해설**: 매칭 실패는 에러가 아니라 정상 동작(None 또는 [] 반환)입니다. `re.error`는 패턴 자체의 문법이 잘못되었을 때만 발생합니다.
> </details>

> **Practice 5.6**: 문자열 `"apple banana cherry"`에서 `search()`와 `findall()`로 각각 `\b\w+\b` 패턴을 적용했을 때 결과의 차이를 설명하세요.
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> text = "apple banana cherry"
>
> # search(): 첫 번째 매칭만
> result = re.search(r'\b\w+\b', text)
> print(result.group())  # 'apple'
>
> # findall(): 모든 매칭
> result = re.findall(r'\b\w+\b', text)
> print(result)  # ['apple', 'banana', 'cherry']
> ```
>
> **해설**: `search()`는 첫 번째 매칭(`'apple'`)을 찾으면 즉시 멈추고 매치 객체를 반환합니다. `findall()`은 문자열 끝까지 스캔하여 모든 매칭을 리스트로 반환합니다.
> </details>

---

## D2. Exercise (응용 문제) — 개념 적용 및 통합

> **Exercise 5.1**: 아래 `data` 리스트에서, 각 문자열이 "숫자로 시작하는지" 확인하고, 숫자로 시작하는 문자열에서는 **시작 부분의 숫자**를 추출하여 출력하는 프로그램을 작성하세요.
>
> ```python
> data = [
>     "100개 주문",
>     "주문번호 200",
>     "300명 참가",
>     "총 400건",
>     "50%할인",
> ]
> ```
>
> 예상 출력:
> ```
> '100개 주문' → 시작 숫자: 100
> '300명 참가' → 시작 숫자: 300
> '50%할인' → 시작 숫자: 50
> ```
>
> **힌트**: 문자열 시작에서 숫자를 찾는 함수는 무엇이었나요?
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: "숫자로 시작"하는지 확인하려면 `re.match()`를 사용합니다. `match()`는 문자열 시작에서만 패턴을 찾으므로 이 용도에 정확히 맞습니다.
>
> **풀이 과정**:
> ```python
> import re
>
> data = [
>     "100개 주문",
>     "주문번호 200",
>     "300명 참가",
>     "총 400건",
>     "50%할인",
> ]
>
> for item in data:
>     result = re.match(r'\d+', item)
>     if result:
>         print(f"'{item}' → 시작 숫자: {result.group()}")
> ```
>
> **보충 설명**: `re.search(r'^\d+', item)`으로도 같은 결과를 얻을 수 있지만, 문자열 시작에서의 매칭이 목적이라면 `match()`가 의도를 더 명확히 전달합니다.
> </details>

> **Exercise 5.2**: 아래의 텍스트에서 **해시태그**(# 뒤에 한글 또는 영문 단어가 오는 형태)를 모두 추출하되, 각 해시태그의 **텍스트 내 위치(시작 인덱스)**도 함께 출력하세요.
>
> ```python
> post = "오늘 #맛집 탐방! #서울 #강남역 근처 #카페 추천합니다 #daily"
> ```
>
> 예상 출력:
> ```
> #맛집 (위치: 3)
> #서울 (위치: 11)
> #강남역 (위치: 15)
> #카페 (위치: 22)
> #daily (위치: 29)
> ```
>
> **힌트**: 매칭 결과와 위치 정보를 함께 얻으려면 어떤 함수를 사용해야 할까요?
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 모든 매칭의 **위치 정보**가 필요하므로 `re.finditer()`를 사용합니다. `findall()`은 매칭 문자열만 반환하고 위치 정보는 제공하지 않습니다.
>
> **풀이 과정**:
> ```python
> import re
>
> post = "오늘 #맛집 탐방! #서울 #강남역 근처 #카페 추천합니다 #daily"
>
> for match in re.finditer(r'#\w+', post):
>     print(f"{match.group()} (위치: {match.start()})")
> ```
>
> **보충 설명**: `\w`는 파이썬 3의 기본 유니코드 모드에서 한글도 매칭하므로, 별도의 한글 처리 없이 `#\w+`로 한글·영문 해시태그를 모두 잡을 수 있습니다.
> </details>

> **Exercise 5.3**: 사용자로부터 검색어를 입력받아, 주어진 텍스트에서 해당 검색어의 등장 횟수를 세는 함수 `count_occurrences(query, text)`를 작성하세요. 검색어에 정규표현식 메타문자가 포함되어 있어도 안전하게 동작해야 합니다.
>
> 테스트:
> ```python
> text = "C++ is great. C++ is fast. C# is also good. price: $10.00"
>
> print(count_occurrences("C++", text))      # 2
> print(count_occurrences("$10.00", text))   # 1
> print(count_occurrences("Java", text))     # 0
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 사용자 입력에 메타문자가 포함될 수 있으므로, `re.escape()`로 안전하게 변환한 후 `findall()`로 모든 매칭을 찾고 그 개수를 셉니다.
>
> **풀이 과정**:
> ```python
> import re
>
> def count_occurrences(query, text):
>     safe_query = re.escape(query)
>     matches = re.findall(safe_query, text)
>     return len(matches)
>
> text = "C++ is great. C++ is fast. C# is also good. price: $10.00"
>
> print(count_occurrences("C++", text))      # 2
> print(count_occurrences("$10.00", text))   # 1
> print(count_occurrences("Java", text))     # 0
> ```
>
> **보충 설명**: `re.escape("C++")`는 `"C\\+\\+"`를 반환하여, `+`가 수량자가 아닌 리터럴 `+`로 해석됩니다. `re.escape()` 없이 `re.findall("C++", text)`를 실행하면, `+`가 수량자로 해석되어 예상치 못한 결과가 나옵니다.
> </details>

> **Exercise 5.4**: 다음 함수를 완성하세요. 이 함수는 문자열 리스트를 받아서, `pattern`에 맞는 항목만 필터링하여 반환합니다. 패턴이 잘못되었을 경우 빈 리스트를 반환하고 경고 메시지를 출력합니다.
>
> ```python
> import re
>
> def filter_by_pattern(pattern_str, items):
>     """
>     items 리스트에서 pattern_str 패턴에 매칭되는 항목만 반환.
>     패턴이 잘못된 경우 빈 리스트 반환 + 경고 출력.
>     매칭 기준: 항목 문자열 내 어딘가에 패턴이 존재하면 포함.
>     """
>     # 여기에 코드를 작성하세요
>     pass
> ```
>
> 테스트:
> ```python
> files = ["report_2024.csv", "data.json", "log_2024.txt", "image.png", "backup_2023.csv"]
>
> print(filter_by_pattern(r'\.csv$', files))
> # ['report_2024.csv', 'backup_2023.csv']
>
> print(filter_by_pattern(r'2024', files))
> # ['report_2024.csv', 'log_2024.txt']
>
> print(filter_by_pattern(r'[invalid', files))
> # 경고 메시지 출력 후 []
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: `re.search()`로 각 항목에 패턴이 존재하는지 확인합니다. 패턴 문법 오류에 대비하여 `try-except re.error`로 감쌉니다.
>
> **풀이 과정**:
> ```python
> import re
>
> def filter_by_pattern(pattern_str, items):
>     try:
>         pattern = re.compile(pattern_str)
>     except re.error as e:
>         print(f"⚠️ 잘못된 패턴입니다: {e.msg}")
>         return []
>
>     return [item for item in items if pattern.search(item)]
>
> files = ["report_2024.csv", "data.json", "log_2024.txt", "image.png", "backup_2023.csv"]
>
> print(filter_by_pattern(r'\.csv$', files))
> # ['report_2024.csv', 'backup_2023.csv']
>
> print(filter_by_pattern(r'2024', files))
> # ['report_2024.csv', 'log_2024.txt']
>
> print(filter_by_pattern(r'[invalid', files))
> # ⚠️ 잘못된 패턴입니다: unterminated character set
> # []
> ```
>
> **보충 설명**: `compile()`을 사용해 패턴을 한 번만 컴파일하고, 리스트 컴프리헨션에서 패턴 객체의 `search()` 메서드를 반복 호출합니다. `compile()` 시점에서 패턴 오류를 잡을 수 있으므로, 에러 처리가 깔끔합니다.
> </details>

---

## D3. Problem (심화 문제) — 깊이 있는 사고와 창의적 문제 해결

> **Problem 5.1**: 간단한 **텍스트 통계 분석기**를 만드세요. 주어진 영문 텍스트에 대해 다음 정보를 모두 추출하여 출력하는 함수 `analyze_text(text)`를 작성합니다.
>
> 1. 총 단어 수 (영문 단어 기준, `[A-Za-z]+` 패턴)
> 2. 총 숫자 수 (연속된 숫자 묶음 기준, `\d+`)
> 3. 가장 긴 단어와 그 길이
> 4. 모든 대문자로 시작하는 단어 목록
>
> 테스트 텍스트:
> ```python
> text = """Python 3.12 was released in October 2023.
> It includes MANY improvements and 15 new features.
> The GIL is being redesigned for BETTER performance."""
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: `findall()`로 다양한 패턴을 적용하여 데이터를 추출하고, 파이썬의 내장 함수(`len()`, `max()`, 리스트 컴프리헨션)와 결합합니다.
>
> **풀이 전략**:
> 1. 영문 단어 추출: `re.findall(r'[A-Za-z]+', text)`
> 2. 숫자 추출: `re.findall(r'\d+', text)`
> 3. 가장 긴 단어: `max(words, key=len)`
> 4. 대문자 시작 단어: `re.findall(r'\b[A-Z][a-z]*\b', text)` — 하지만 이 패턴은 전체 대문자 단어(MANY)를 놓칩니다. `re.findall(r'\b[A-Z]\w*\b', text)` 대신 `re.findall(r'\b[A-Z][a-zA-Z]*\b', text)`를 사용하면 대문자로 시작하는 모든 영문 단어를 캡처합니다.
>
> **상세 풀이**:
> ```python
> import re
>
> def analyze_text(text):
>     # 1. 모든 영문 단어
>     words = re.findall(r'[A-Za-z]+', text)
>     print(f"총 단어 수: {len(words)}")
>
>     # 2. 모든 숫자
>     numbers = re.findall(r'\d+', text)
>     print(f"총 숫자 수: {len(numbers)} → {numbers}")
>
>     # 3. 가장 긴 단어
>     if words:
>         longest = max(words, key=len)
>         print(f"가장 긴 단어: '{longest}' ({len(longest)}자)")
>
>     # 4. 대문자로 시작하는 단어
>     capitalized = re.findall(r'\b[A-Z][a-zA-Z]*\b', text)
>     print(f"대문자 시작 단어: {capitalized}")
>
> text = """Python 3.12 was released in October 2023.
> It includes MANY improvements and 15 new features.
> The GIL is being redesigned for BETTER performance."""
>
> analyze_text(text)
> ```
>
> 출력:
> ```
> 총 단어 수: 21
> 총 숫자 수: 4 → ['3', '12', '2023', '15']
> 가장 긴 단어: 'improvements' (12자)
> 대문자 시작 단어: ['Python', 'October', 'It', 'MANY', 'The', 'GIL', 'BETTER']
> ```
>
> **확장 생각**: 이 분석기에 `finditer()`를 사용하면 각 단어/숫자의 텍스트 내 위치까지 추적할 수 있습니다. 또한 `compile()`로 여러 패턴을 미리 정의해두면, 여러 문서를 반복 분석할 때 더 구조적인 코드가 됩니다.
> </details>

> **Problem 5.2**: **정규표현식 패턴 테스터**를 만드세요. 사용자가 패턴과 테스트 문자열을 입력하면, 다음 정보를 모두 출력하는 함수 `test_pattern(pattern_str, test_string)`을 작성합니다.
>
> 1. `match()` 결과 (성공/실패, 성공 시 매칭 내용)
> 2. `search()` 결과 (성공/실패, 성공 시 매칭 내용과 위치)
> 3. `fullmatch()` 결과 (성공/실패)
> 4. `findall()` 결과 (전체 매칭 리스트와 매칭 수)
> 5. 패턴이 잘못된 경우 에러 메시지
>
> 다음 테스트 케이스를 모두 통과해야 합니다:
> ```python
> test_pattern(r'\d+', '가격 1500원, 배송비 3000원')
> test_pattern(r'^[A-Z]', 'Hello World')
> test_pattern(r'[invalid', 'test')  # 잘못된 패턴
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 이번 Chapter에서 배운 모든 주요 함수를 종합하여 하나의 진단 도구를 만듭니다. `try-except re.error`로 패턴 유효성을 먼저 확인한 후, 각 함수를 순서대로 실행합니다.
>
> **상세 풀이**:
> ```python
> import re
>
> def test_pattern(pattern_str, test_string):
>     print(f"\n{'='*50}")
>     print(f"패턴: r'{pattern_str}'")
>     print(f"테스트 문자열: '{test_string}'")
>     print(f"{'='*50}")
>
>     # 패턴 유효성 확인
>     try:
>         pattern = re.compile(pattern_str)
>     except re.error as e:
>         print(f"❌ 패턴 오류: {e.msg} (위치: {e.pos})")
>         return
>
>     # 1. match()
>     m = pattern.match(test_string)
>     if m:
>         print(f"match():     ✅ '{m.group()}' (위치 0~{m.end()})")
>     else:
>         print(f"match():     ❌ 매칭 실패")
>
>     # 2. search()
>     s = pattern.search(test_string)
>     if s:
>         print(f"search():    ✅ '{s.group()}' (위치 {s.start()}~{s.end()})")
>     else:
>         print(f"search():    ❌ 매칭 실패")
>
>     # 3. fullmatch()
>     f = pattern.fullmatch(test_string)
>     if f:
>         print(f"fullmatch(): ✅ 전체 일치")
>     else:
>         print(f"fullmatch(): ❌ 전체 일치하지 않음")
>
>     # 4. findall()
>     all_matches = pattern.findall(test_string)
>     print(f"findall():   {all_matches} ({len(all_matches)}개 매칭)")
>
> # 테스트
> test_pattern(r'\d+', '가격 1500원, 배송비 3000원')
> test_pattern(r'^[A-Z]', 'Hello World')
> test_pattern(r'[invalid', 'test')
> ```
>
> **확장 생각**: 이 도구에 `finditer()`를 추가하여 모든 매칭의 위치를 시각적으로 표시하는 기능(예: 원본 텍스트 아래에 `^` 마커)을 추가하면 더욱 유용한 디버깅 도구가 됩니다. 또한 이 구조는 Chapter 10에서 배울 **플래그** 옵션을 추가하여 확장할 수 있습니다.
> </details>

---

# Part E: 마무리 (Closing Block)

---

## 다음 단계 안내 (What's Next)

이번 Chapter에서 `re` 모듈의 핵심 함수를 배우면서, 우리는 **매치 객체(Match object)**라는 것을 여러 번 만났습니다. `.group()`으로 매칭된 문자열을 꺼내는 것까지는 해봤지만, 매치 객체가 품고 있는 풍부한 정보의 극히 일부만 사용한 것입니다.

**Chapter 6: 매치 객체와 캡처 그룹 기초**에서는 매치 객체를 본격적으로 파헤칩니다. 특히 소괄호 `()`를 사용한 **캡처 그룹**을 배우면, 패턴의 일부분만 골라서 추출하는 강력한 기법을 익히게 됩니다. 예를 들어 `"2024-03-15"` 전체를 매칭하면서도 년도, 월, 일을 각각 별도로 뽑아낼 수 있습니다.

---

## 추가 학습 자원 (Further Resources)

- **파이썬 공식 문서**: [re 모듈 — Regular expression operations](https://docs.python.org/3/library/re.html) — 이 Chapter에서 다룬 모든 함수의 공식 레퍼런스
- **온라인 테스트 도구**: [regex101.com](https://regex101.com) — Python 플레이버를 선택하고 패턴을 실시간으로 테스트할 수 있습니다. 각 함수를 배우면서 여기서 직접 실험해보는 것을 강력히 권장합니다.
- **파이썬 정규표현식 HOWTO**: [Regular Expression HOWTO](https://docs.python.org/3/howto/regex.html) — 파이썬 공식 문서에서 제공하는 정규표현식 튜토리얼
