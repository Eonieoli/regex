# Chapter 17: 서드파티 라이브러리와 종합 프로젝트

> **난이도**: ⭐⭐⭐⭐⭐ | **예상 학습 시간**: 4시간 | **선수 지식**: Chapter 16 (성능 최적화와 디버깅), 그리고 이를 통해 Chapter 1~15의 모든 내용

---

# Part A: 도입부

## 학습 목표 (Learning Objectives)

이 Chapter를 완료하면, 당신은 다음을 할 수 있습니다:

- [ ] `regex` 모듈의 확장 기능(유니코드 카테고리, 소유적 수량자 구문, 원자적 그룹 구문 등)을 활용할 수 있다
- [ ] 정규표현식의 한계를 인식하고, 상황에 따라 적절한 대안 도구를 선택할 수 있다
- [ ] 복합적인 실전 프로젝트에서 정규표현식을 설계, 구현, 테스트, 최적화하는 전체 워크플로우를 수행할 수 있다
- [ ] 다른 사람이 작성한 복잡한 정규표현식을 분석하고 개선할 수 있다

---

## 핵심 질문 (Essential Questions)

1. **파이썬 표준 `re` 모듈의 한계는 무엇이며, `regex` 모듈은 이를 어떻게 극복하는가?**
2. **정규표현식이 최선의 도구가 아닌 상황은 언제이며, 그때 어떤 대안을 선택해야 하는가?**
3. **실전 프로젝트에서 정규표현식을 설계부터 최적화까지 체계적으로 수행하는 워크플로우는 어떻게 구성되는가?**

---

## 개념 지도 (Concept Map)

```
[선수: re 모듈 전체 (Ch5~Ch7)]
[선수: 고급 패턴 — 그룹핑, 전후방 탐색, 유니코드 (Ch8~Ch11)]
[선수: 실전 응용 — 검증, 정제, 로그 분석 (Ch12~Ch14)]
[선수: 엔진 원리 — NFA/백트래킹/소유적 수량자 개념 (Ch15)]
[선수: 성능 최적화/디버깅 (Ch16)]
        │
        ▼
[신규: regex 모듈] ──→ [유니코드 카테고리 \p{}, 소유적 수량자 ++, 원자적 그룹 (?>...)]
        │                              │
        ▼                              ▼
[신규: 퍼지 매칭]              [성능 최적화의 실전 적용]
        │
        ▼
[신규: 대안 도구] ──→ [parse, pyparsing, PEG 파서]
        │
        ▼
[종합 프로젝트] ──→ [설계 → 구현 → 테스트 → 최적화 워크플로우]
```

이 Chapter는 과정 전체의 **마지막 Chapter**입니다. 지금까지 배운 모든 지식을 종합하여, 표준 `re` 모듈의 한계를 넘어서는 확장 도구를 익히고, 정규표현식이 적합하지 않은 상황의 대안까지 파악합니다. 마지막으로 종합 프로젝트를 통해 실전 역량을 완성합니다.

---

# Part B: 본문 (Main Content)

---

## 17.1 `regex` 모듈 소개

> 💡 **한 줄 요약**: `regex` 모듈은 파이썬 표준 `re` 모듈의 상위 호환으로, 유니코드 카테고리, 소유적 수량자, 원자적 그룹, 퍼지 매칭 등 강력한 확장 기능을 제공합니다.

### 직관적 이해

지금까지 우리는 파이썬 표준 라이브러리의 `re` 모듈로 정규표현식의 거의 모든 것을 학습했습니다. `re` 모듈은 대부분의 텍스트 처리 작업에 충분하지만, 실무에서 다음과 같은 아쉬움을 느끼는 순간이 옵니다:

- "유니코드 문자를 카테고리(예: 모든 문자, 모든 숫자)로 매칭하고 싶은데, `\w`만으로는 세밀한 제어가 안 된다"
- "Chapter 15에서 배운 소유적 수량자와 원자적 그룹을 실제 코드에서 쓰고 싶은데, `re` 모듈에서는 문법이 지원되지 않는다"
- "오탈자가 포함된 텍스트에서도 '비슷한' 문자열을 찾고 싶다"

`regex` 모듈은 이러한 `re` 모듈의 한계를 정면으로 해결하기 위해 만들어진 서드파티 라이브러리입니다. 비유하자면, `re`가 일반 자동차라면 `regex`는 같은 도로 위를 달리되 터보 엔진과 추가 기능을 장착한 업그레이드 버전입니다. 기존 `re` 코드와 완벽하게 호환되면서도 강력한 확장 기능을 추가로 제공합니다.

### 핵심 개념 설명

#### 설치와 기본 사용법

`regex` 모듈은 PyPI에서 설치할 수 있습니다:

```bash
pip install regex
```

사용법은 `re` 모듈과 거의 동일합니다:

```python
import regex

# re 모듈과 동일한 인터페이스
m = regex.search(r'\d+', 'abc 123 def')
print(m.group())  # '123'

# re 모듈의 모든 함수를 그대로 사용 가능
results = regex.findall(r'[a-z]+', 'hello world')
print(results)  # ['hello', 'world']

# compile도 동일
pattern = regex.compile(r'\b\w+\b')
```

> 📝 **`regex` 모듈**: 파이썬 표준 `re` 모듈의 드롭인(drop-in) 대체 라이브러리. `re` 모듈의 모든 기능을 포함하면서 유니코드 카테고리, 소유적 수량자, 원자적 그룹, 퍼지 매칭 등의 확장 기능을 추가로 제공합니다.

> 📝 **드롭인 대체(drop-in replacement)**: 기존 코드에서 `import re`를 `import regex as re`로 바꾸기만 하면 기존 코드가 그대로 동작하는 호환성을 의미합니다.

#### `re` 모듈과의 호환성

`regex` 모듈은 두 가지 동작 모드를 제공합니다:

```python
import regex

# VERSION0: re 모듈과 완전히 동일한 동작 (기본값이 아님)
regex.DEFAULT_VERSION = regex.VERSION0

# VERSION1: regex 모듈의 확장 기능 활성화 (기본값)
regex.DEFAULT_VERSION = regex.VERSION1
```

`VERSION1`(기본값)에서는 일부 동작이 `re` 모듈과 미세하게 달라질 수 있습니다. 기존 `re` 코드와의 완벽한 호환이 중요하다면 `VERSION0`을 사용하거나, 개별 패턴에서 플래그로 지정할 수 있습니다:

```python
# 패턴별로 버전 지정
pattern = regex.compile(r'\d+', flags=regex.VERSION0)
```

> 🔗 **연결**: `re.compile()`과 플래그 지정 방식은 Chapter 5(re 모듈 기본 함수)와 Chapter 10(플래그와 컴파일 옵션)에서 배운 것과 동일합니다.

#### `regex` 모듈의 주요 확장 기능 개요

`regex` 모듈이 `re` 모듈 대비 제공하는 주요 확장 기능을 먼저 전체적으로 조망해 봅시다. 각 기능은 이어지는 Section들에서 자세히 다룹니다.

| 확장 기능 | 문법 예시 | 설명 |
|-----------|----------|------|
| 유니코드 카테고리 | `\p{L}`, `\p{Han}` | 유니코드 속성 기반 매칭 |
| 소유적 수량자 | `a++`, `\d*+` | 백트래킹 억제 수량자 구문 |
| 원자적 그룹 | `(?>...)` | 백트래킹 억제 그룹 구문 |
| 퍼지 매칭 | `(?:pattern){e<=2}` | 오차 허용 매칭 |
| 부분 매칭 | `partial=True` | 불완전한 입력에 대한 부분 매칭 |
| 향상된 유니코드 | 대소문자 폴딩 등 | 유니코드 기반 대소문자 처리 개선 |

### 상세 예시

> 🔍 **예시 17.1.1: `re`에서 `regex`로 전환하기**
>
> **상황**: 기존 `re` 모듈을 사용하던 코드를 `regex` 모듈로 전환하려 합니다.
>
> **풀이/설명**:
> ```python
> # 기존 코드 (re 모듈 사용)
> import re
> 
> pattern = re.compile(r'(\d{4})-(\d{2})-(\d{2})')
> text = '날짜: 2024-03-15, 2024-12-25'
> 
> for m in re.finditer(pattern, text):
>     print(f"{m.group(1)}년 {m.group(2)}월 {m.group(3)}일")
> 
> # regex 모듈로 전환 — import만 변경하면 됨
> import regex as re  # 이 한 줄만 변경!
> 
> pattern = re.compile(r'(\d{4})-(\d{2})-(\d{2})')
> text = '날짜: 2024-03-15, 2024-12-25'
> 
> for m in re.finditer(pattern, text):
>     print(f"{m.group(1)}년 {m.group(2)}월 {m.group(3)}일")
> # 출력 동일:
> # 2024년 03월 15일
> # 2024년 12월 25일
> ```
>
> **핵심 포인트**: `import regex as re` 한 줄만 변경하면 기존 코드가 그대로 동작합니다. 이것이 "드롭인 대체"의 의미입니다.

> 🔍 **예시 17.1.2: `regex` 모듈에서만 가능한 기능 맛보기**
>
> **상황**: 텍스트에서 모든 '문자'(숫자·기호 제외)만 추출하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import regex
> 
> text = "Hello 세계! 123 こんにちは Мир 🌍"
> 
> # re 모듈: \w는 문자+숫자+밑줄을 매칭 → 숫자도 포함됨
> import re
> print(re.findall(r'\w+', text))
> # ['Hello', '세계', '123', 'こんにちは', 'Мир']
> # ↑ 123도 포함됨, 이모지는 제외됨
> 
> # regex 모듈: \p{L}로 '문자 카테고리'만 정확히 매칭
> print(regex.findall(r'\p{L}+', text))
> # ['Hello', '세계', 'こんにちは', 'Мир']
> # ↑ 숫자 제외, 문자만 깔끔하게 추출!
> ```
>
> **핵심 포인트**: `\p{L}`(유니코드 문자 카테고리)은 `re` 모듈에서는 지원되지 않는 `regex` 모듈 고유의 기능입니다. `\w`보다 훨씬 세밀한 매칭이 가능합니다.

> ⚠️ **주의**: `regex` 모듈은 서드파티 라이브러리이므로 별도 설치가 필요합니다. 배포 환경에서 의존성 추가가 어려운 경우에는 표준 `re` 모듈을 사용해야 합니다. 또한 `regex`는 `re`보다 메모리를 더 사용할 수 있으므로, 단순한 패턴에는 `re`로 충분합니다.

### Section 요약

- `regex` 모듈은 PyPI에서 설치할 수 있는 서드파티 라이브러리로, `re` 모듈의 상위 호환입니다.
- `import regex as re`로 기존 코드를 그대로 사용할 수 있는 드롭인 대체가 가능합니다.
- 주요 확장 기능으로 유니코드 카테고리, 소유적 수량자 구문, 원자적 그룹 구문, 퍼지 매칭 등이 있습니다.
- `VERSION0`(re 호환)과 `VERSION1`(확장 기능) 두 가지 동작 모드를 제공합니다.

---

## 17.2 유니코드 카테고리 매칭

> 💡 **한 줄 요약**: `\p{카테고리}` 문법을 사용하면 유니코드 문자를 속성(문자, 숫자, 기호, 스크립트 등)에 따라 정밀하게 매칭할 수 있습니다.

### 직관적 이해

> 🔗 **연결**: Chapter 11(유니코드와 한국어 처리)에서 `\w`, `\d` 등이 유니코드 모드에서 어떻게 동작하는지 배웠습니다. 또한 `[가-힣]` 같은 범위 표현으로 한글을 매칭하는 방법도 학습했습니다.

Chapter 11에서 배운 방식은 효과적이지만, 한계가 있었습니다. 예를 들어:
- `\w`는 "문자+숫자+밑줄"을 한꺼번에 매칭하므로, "문자만" 또는 "숫자만" 매칭하려면 별도 처리가 필요합니다.
- `[가-힣]`은 한글 완성형 음절만 매칭하며, "모든 한글 관련 문자"(자모 포함)를 한 번에 지정하기 어렵습니다.
- "모든 CJK 한자", "모든 키릴 문자", "모든 이모지" 같은 요구사항을 범위 표현만으로 처리하기는 번거롭습니다.

유니코드 카테고리 매칭은 이런 문제를 근본적으로 해결합니다. 유니코드 표준은 모든 문자에 "이 문자는 어떤 종류인가?"라는 **속성(property)**을 부여합니다. 예를 들어 'A'는 "대문자 알파벳", '3'은 "십진 숫자", '가'는 "한글 음절"이라는 속성을 가집니다. `\p{}` 문법은 이 속성을 기준으로 문자를 매칭합니다.

### 핵심 개념 설명

#### 일반 카테고리 (General Category)

유니코드의 일반 카테고리는 대문자 한 글자(대분류)와 소문자 한 글자(소분류)로 구성됩니다.

> 📝 **유니코드 일반 카테고리(General Category)**: 유니코드 표준에서 모든 문자를 분류하는 체계. 대문자 한 글자로 대분류를, 두 글자로 소분류를 나타냅니다.

**주요 대분류:**

| 카테고리 | 의미 | 매칭 대상 예시 |
|----------|------|---------------|
| `\p{L}` | Letter (문자) | a, B, 가, あ, Б |
| `\p{N}` | Number (숫자) | 0, ², ½, ⅓ |
| `\p{P}` | Punctuation (구두점) | . , ! ? ; : |
| `\p{S}` | Symbol (기호) | $ + < = > © ® 🌍 |
| `\p{Z}` | Separator (분리자) | 공백, 비분리 공백 |
| `\p{M}` | Mark (결합 기호) | 악센트, 결합 자모 |
| `\p{C}` | Other (기타) | 제어 문자, 서로게이트 |

**주요 소분류:**

| 카테고리 | 의미 | 매칭 대상 예시 |
|----------|------|---------------|
| `\p{Lu}` | Uppercase Letter | A, B, Ω |
| `\p{Ll}` | Lowercase Letter | a, b, ω |
| `\p{Lt}` | Titlecase Letter | ǅ, ǈ |
| `\p{Lo}` | Other Letter | 가, 漢, あ |
| `\p{Nd}` | Decimal Digit Number | 0-9, ٠-٩ |
| `\p{Nl}` | Letter Number | Ⅰ, Ⅱ, Ⅲ |
| `\p{Ps}` | Open Punctuation | ( [ { |
| `\p{Pe}` | Close Punctuation | ) ] } |

```python
import regex

text = "Hello 세계 123 ½ © 🌍 あいう Мир"

# 모든 문자(Letter)만 추출
print(regex.findall(r'\p{L}+', text))
# ['Hello', '세계', 'あいう', 'Мир']

# 모든 숫자(Number)만 추출 — 분수, 로마 숫자 등도 포함
print(regex.findall(r'\p{N}+', text))
# ['123', '½']

# 모든 기호(Symbol)만 추출
print(regex.findall(r'\p{S}', text))
# ['©', '🌍']

# 대문자만 추출
print(regex.findall(r'\p{Lu}', text))
# ['H', 'М']  — 'М'은 키릴 대문자
```

#### 부정 카테고리

카테고리의 부정형은 대문자 `P`를 사용합니다:

```python
import regex

text = "Hello 123 세계!"

# \p{L} → 문자,  \P{L} → 문자가 아닌 것
print(regex.findall(r'\P{L}+', text))
# [' ', ' 123 ', '!']  — 문자가 아닌 연속된 부분

# 소문자가 아닌 문자만
print(regex.findall(r'\P{Ll}', "Hello World"))
# ['H', ' ', 'W']
```

#### 스크립트(Script) 매칭

유니코드 카테고리 매칭의 진정한 위력은 **스크립트 기반 매칭**에서 드러납니다. 스크립트란 문자가 속한 문자 체계(한글, 한자, 히라가나 등)를 의미합니다.

> 📝 **유니코드 스크립트(Script)**: 문자가 속한 문자 체계를 나타내는 속성. 예를 들어 '가'는 Hangul 스크립트, '漢'은 Han 스크립트, 'A'는 Latin 스크립트에 속합니다.

```python
import regex

text = "한글테스트 漢字テスト English тест"

# 한글(Hangul) 스크립트만 매칭
print(regex.findall(r'\p{Hangul}+', text))
# ['한글테스트']

# 한자(Han) 스크립트만 매칭 — CJK 통합 한자
print(regex.findall(r'\p{Han}+', text))
# ['漢字']

# 히라가나(Hiragana)와 가타카나(Katakana)
print(regex.findall(r'\p{Katakana}+', text))
# ['テスト']

# 라틴(Latin) 스크립트만 매칭
print(regex.findall(r'\p{Latin}+', text))
# ['English']

# 키릴(Cyrillic) 스크립트만 매칭
print(regex.findall(r'\p{Cyrillic}+', text))
# ['тест']
```

> 🔗 **연결**: Chapter 11에서 `[가-힣]`로 한글 완성형 음절을 매칭했습니다. `\p{Hangul}`은 이보다 더 포괄적으로, 한글 자모, 호환 자모, 음절까지 모두 매칭합니다.

**`[가-힣]`과 `\p{Hangul}`의 차이:**

```python
import regex

text = "가나다 ㄱㄴㄷ ㅏㅓㅗ"

# [가-힣]은 완성형 음절만 매칭
print(regex.findall(r'[가-힣]+', text))
# ['가나다']

# \p{Hangul}은 자모까지 포함
print(regex.findall(r'\p{Hangul}+', text))
# ['가나다', 'ㄱㄴㄷ', 'ㅏㅓㅗ']
```

#### 블록(Block) 매칭

유니코드 블록은 코드포인트의 연속된 범위입니다. 스크립트와 비슷하지만 더 세분화된 매칭이 가능합니다:

```python
import regex

# 이모지 관련 블록 매칭
text = "안녕 👋 Hello 🌍 세계 🎉"
print(regex.findall(r'\p{Emoticons}', text))
# 이모지 블록에 해당하는 문자들

# CJK 통합 한자 확장 블록
text = "基本漢字"
print(regex.findall(r'\p{InCJKUnifiedIdeographs}+', text))
# ['基本漢字']
```

### 상세 예시

> 🔍 **예시 17.2.1: 다국어 텍스트에서 각 언어별 단어 분리**
>
> **상황**: 여러 언어가 섞인 텍스트에서 각 언어(스크립트)별로 단어를 분류해야 합니다.
>
> **풀이/설명**:
> ```python
> import regex
> from collections import defaultdict
> 
> text = "Python은 파이썬이라고도 하며, 日本語では「パイソン」と言います"
> 
> # 스크립트별로 분류
> scripts = {
>     'Latin': r'\p{Latin}+',
>     'Hangul': r'\p{Hangul}+',
>     'Han': r'\p{Han}+',
>     'Hiragana': r'\p{Hiragana}+',
>     'Katakana': r'\p{Katakana}+',
> }
> 
> result = {}
> for script_name, pattern in scripts.items():
>     matches = regex.findall(pattern, text)
>     if matches:
>         result[script_name] = matches
> 
> for script, words in result.items():
>     print(f"{script}: {words}")
> # Latin: ['Python']
> # Hangul: ['은', '파이썬이라고도', '하며']
> # Han: ['日本語']
> # Hiragana: ['では', 'と', 'います']
> # Katakana: ['パイソン']
> ```
>
> **핵심 포인트**: `\p{Script}` 문법으로 각 문자 체계를 정확히 분리할 수 있어, 다국어 텍스트 분석에 매우 유용합니다.

> 🔍 **예시 17.2.2: 숫자가 아닌 "문자"만으로 구성된 토큰 추출**
>
> **상황**: 텍스트에서 순수한 문자 토큰만 추출하되, 숫자 포함 토큰은 제외하려 합니다.
>
> **풀이/설명**:
> ```python
> import regex
> 
> text = "주문번호 A12345 고객명 홍길동 수량 100개 상품 노트북"
> 
> # \w+로 추출하면 숫자 포함 토큰도 나옴
> print(regex.findall(r'\w+', text))
> # ['주문번호', 'A12345', '고객명', '홍길동', '수량', '100개', '상품', '노트북']
> 
> # \p{L}+로 순수 문자 토큰만 추출
> print(regex.findall(r'\p{L}+', text))
> # ['주문번호', 'A', '고객명', '홍길동', '수량', '개', '상품', '노트북']
> 
> # 2글자 이상의 순수 문자 토큰만 추출
> print(regex.findall(r'\p{L}{2,}', text))
> # ['주문번호', '고객명', '홍길동', '수량', '상품', '노트북']
> ```
>
> **핵심 포인트**: `\p{L}`은 `\w`와 달리 숫자와 밑줄을 제외한 순수 문자만 매칭합니다. 데이터 정제 시 토큰 분류에 매우 유용합니다.

> 🔬 **Deep Dive: `\p{}` vs `\w` — 언제 무엇을 써야 하나?**
>
> *이 부분은 심화 내용입니다. 처음 학습 시 건너뛰어도 됩니다.*
>
> `\w`와 `\p{L}`의 매칭 범위 차이를 정확히 이해하면 도구 선택이 명확해집니다:
>
> | 문자 | `\w` (re/regex) | `\p{L}` (regex) | `\p{N}` (regex) |
> |------|-----------------|-----------------|-----------------|
> | a-z, A-Z | ✅ | ✅ | ❌ |
> | 0-9 | ✅ | ❌ | ✅ |
> | _ (밑줄) | ✅ | ❌ | ❌ |
> | 가-힣 (한글) | ✅ | ✅ | ❌ |
> | 漢 (한자) | ✅ | ✅ | ❌ |
> | ½ (분수) | ❌ | ❌ | ✅ |
> | © (기호) | ❌ | ❌ | ❌ |
>
> **선택 기준**:
> - 일반적인 "단어" 매칭 → `\w`로 충분
> - 문자와 숫자를 분리해야 할 때 → `\p{L}`과 `\p{N}` 사용
> - 특정 언어 스크립트만 대상으로 할 때 → `\p{Hangul}`, `\p{Latin}` 등 사용
> - 표준 라이브러리만 사용해야 할 때 → `\w`와 문자 클래스 조합으로 대응

> ⚠️ **주의**: `\p{}` 문법은 `regex` 모듈에서만 지원됩니다. 표준 `re` 모듈에서 `\p{L}`을 사용하면 오류가 발생합니다. 코드의 이식성이 중요한 경우 이 점을 고려하세요.

### Section 요약

- `\p{카테고리}`로 유니코드 속성 기반 정밀 매칭이 가능합니다(대분류: `\p{L}`, `\p{N}` 등, 소분류: `\p{Lu}`, `\p{Nd}` 등).
- `\P{카테고리}`(대문자 P)로 부정 매칭을 수행합니다.
- `\p{Hangul}`, `\p{Han}`, `\p{Latin}` 등 스크립트 매칭으로 다국어 텍스트를 체계적으로 분류할 수 있습니다.
- `\p{Hangul}`은 `[가-힣]`보다 포괄적으로, 한글 자모까지 포함합니다.
- `\p{}` 문법은 `regex` 모듈 전용이며, 표준 `re` 모듈에서는 지원되지 않습니다.

---

## 17.3 소유적 수량자와 원자적 그룹의 실전 사용

> 💡 **한 줄 요약**: Chapter 15에서 개념을 배운 소유적 수량자(`++`, `*+`)와 원자적 그룹(`(?>...)`)을 `regex` 모듈에서 실제 코드로 사용하여 성능을 극적으로 향상시킬 수 있습니다.

### 직관적 이해

> 🔗 **연결**: Chapter 15(정규표현식 엔진의 원리)에서 소유적 수량자와 원자적 그룹의 **개념과 원리**를 배웠습니다. 소유적 수량자는 "한 번 매칭한 문자를 절대 내놓지 않는" 수량자이고, 원자적 그룹은 "그룹 내부에서 한 번 매칭이 완료되면 백트래킹하지 않는" 그룹이라는 것을 기억하실 겁니다.

> 🔗 **연결**: Chapter 16(성능 최적화와 디버깅)에서 재앙적 백트래킹의 원인과 위험성을 학습했습니다. 소유적 수량자와 원자적 그룹은 이 문제를 근본적으로 해결하는 도구입니다.

문제는 파이썬 표준 `re` 모듈에서는 이 두 기능의 **구문(syntax)**을 지원하지 않는다는 것이었습니다. 개념은 알지만 코드로 쓸 수 없는 상황이었죠. `regex` 모듈은 이 구문을 완벽하게 지원합니다.

### 핵심 개념 설명

#### 소유적 수량자 구문

소유적 수량자는 기존 수량자 뒤에 `+`를 붙여 표현합니다:

| 탐욕적 | 비탐욕적 | 소유적 | 의미 |
|--------|---------|--------|------|
| `*` | `*?` | `*+` | 0회 이상, 백트래킹 없음 |
| `+` | `+?` | `++` | 1회 이상, 백트래킹 없음 |
| `?` | `??` | `?+` | 0 또는 1회, 백트래킹 없음 |
| `{n,m}` | `{n,m}?` | `{n,m}+` | n~m회, 백트래킹 없음 |

```python
import regex

# 탐욕적 수량자: 최대한 매칭 후 필요시 백트래킹
print(regex.search(r'".*"', '"hello" and "world"').group())
# '"hello" and "world"'  — 가능한 한 많이 매칭

# 비탐욕적 수량자: 최소한 매칭 후 필요시 확장
print(regex.search(r'".*?"', '"hello" and "world"').group())
# '"hello"'  — 최소한만 매칭

# 소유적 수량자: 최대한 매칭, 백트래킹 없음
result = regex.search(r'".*+"', '"hello" and "world"')
print(result)
# None!  — 모든 문자를 소유적으로 가져간 후 마지막 "를 찾을 수 없어 실패
```

마지막 예시가 중요합니다. `.*+`는 `"`를 만난 후 문자열 끝까지 모든 문자를 가져갑니다. 그런데 패턴의 마지막 `"`를 매칭할 문자가 남아있지 않습니다. 탐욕적 수량자라면 백트래킹으로 문자를 하나씩 돌려주겠지만, 소유적 수량자는 **절대 돌려주지 않으므로** 매칭이 실패합니다.

#### 소유적 수량자의 올바른 사용 — 성능 최적화

소유적 수량자의 진정한 가치는 **매칭이 실패할 때** 드러납니다. 불필요한 백트래킹을 방지하여 성능을 극적으로 향상시킵니다.

```python
import regex
import time

# 재앙적 백트래킹 패턴 (Chapter 16에서 배운 위험한 패턴)
text = 'a' * 25 + 'b'

# 탐욕적 수량자 — 매칭 실패 시 재앙적 백트래킹
start = time.time()
try:
    # 주의: 이 패턴은 매우 오래 걸릴 수 있습니다!
    result = regex.search(r'^(a+)+$', text, timeout=3)  # regex는 timeout 지원
except regex.error:
    print(f"시간 초과!")
end = time.time()
print(f"탐욕적: {end - start:.3f}초")

# 소유적 수량자 — 백트래킹 없이 즉시 실패
start = time.time()
result = regex.search(r'^(a++)+$', text)
end = time.time()
print(f"소유적: {end - start:.6f}초")  # 거의 즉시 완료
print(f"결과: {result}")  # None — 즉시 실패 판정
```

> 🔗 **연결**: Chapter 16에서 `(a+)+`가 재앙적 백트래킹을 일으키는 대표적인 안티패턴임을 배웠습니다. `(a++)+`는 이 문제를 소유적 수량자로 해결합니다.

#### 원자적 그룹 구문

원자적 그룹은 `(?>...)`로 표현합니다:

```python
import regex

# 원자적 그룹: 그룹 안에서 매칭이 완료되면 백트래킹 불가
text = "aaaaab"

# 일반 그룹 — 백트래킹 가능
print(regex.search(r'(a+)ab', text).group())
# 'aaaaab' — a+가 aaaa를 매칭하고 마지막 ab를 남겨둠

# 원자적 그룹 — 백트래킹 불가
result = regex.search(r'(?>a+)ab', text)
print(result)
# None — a+가 모든 a를 가져간 후 돌려주지 않아 ab 매칭 실패
```

원자적 그룹은 소유적 수량자보다 더 유연합니다. 그룹 안에 **복잡한 패턴**을 넣을 수 있기 때문입니다:

```python
import regex

# 원자적 그룹 안에 복잡한 패턴
# "선택(|)이 포함된 패턴에서 백트래킹 방지"
text = "integer"

# 일반 패턴: 'int'를 매칭한 후 실패하면 'integer'를 시도
print(regex.search(r'(?:integer|int)', text).group())
# 'integer'

# 원자적 그룹: 첫 번째 대안이 매칭되면 다른 대안은 시도하지 않음
print(regex.search(r'(?>integer|int)', text).group())
# 'integer' — 이 경우는 동일

# 순서를 바꾸면 차이가 드러남
print(regex.search(r'(?>int|integer)', text).group())
# 'int' — 'int'가 먼저 매칭되면 'integer'는 시도하지 않음
```

#### 소유적 수량자와 원자적 그룹의 관계

소유적 수량자는 원자적 그룹의 축약형으로 볼 수 있습니다:

```python
import regex

text = "aaab"

# 다음 두 패턴은 동일한 동작
result1 = regex.search(r'a++', text)
result2 = regex.search(r'(?>a+)', text)

print(result1.group())  # 'aaa'
print(result2.group())  # 'aaa'

# 즉, X++ 는 (?>X+)와 동일
# X*+ 는 (?>X*)와 동일
# X?+ 는 (?>X?)와 동일
```

#### 실전 적용 패턴

```python
import regex

# 실전 1: CSV 필드 파싱 — 쌍따옴표 안의 내용 추출
csv_line = '"John ""Johnny"" Doe",30,"New York"'

# 소유적 수량자로 효율적 매칭
fields = regex.findall(r'"([^"]*+(?:""[^"]*+)*+)"', csv_line)
print(fields)

# 실전 2: 주석이 아닌 코드 라인만 매칭
code = """
x = 10  # 변수 설정
# 이것은 주석입니다
y = 20
"""

# 앵커 + 소유적 수량자로 효율적 매칭
non_comment_lines = regex.findall(
    r'^(?!#)\s*+\S.+$', code, regex.MULTILINE
)
print(non_comment_lines)
# ['x = 10  # 변수 설정', 'y = 20']
```

### 상세 예시

> 🔍 **예시 17.3.1: 재앙적 백트래킹의 소유적 수량자 해결**
>
> **상황**: 사용자 입력을 정규표현식으로 검증하는 웹 서비스에서, 악의적인 입력으로 인한 ReDoS 공격을 방지해야 합니다.
>
> **풀이/설명**:
> ```python
> import regex
> import time
> 
> def benchmark(pattern_str, text, label):
>     """패턴의 실행 시간을 측정합니다."""
>     try:
>         pat = regex.compile(pattern_str)
>         start = time.time()
>         result = pat.search(text)
>         elapsed = time.time() - start
>         matched = result.group() if result else "None"
>         print(f"{label}: {elapsed:.6f}초, 결과: {matched[:30]}...")
>     except Exception as e:
>         print(f"{label}: 오류 - {e}")
> 
> # 이메일 형식이 아닌 긴 입력 (ReDoS 공격 시나리오)
> malicious_input = 'a' * 50 + '@'
> 
> # 취약한 패턴 (중첩 수량자)
> # benchmark(r'^([a-zA-Z0-9]+)+@', malicious_input, "취약한 패턴")
> # ↑ 이 패턴은 수십 초 이상 걸릴 수 있어 주석 처리
> 
> # 소유적 수량자로 수정한 안전한 패턴
> benchmark(r'^([a-zA-Z0-9]++)++@', malicious_input, "소유적 패턴")
> # 소유적 패턴: 0.000010초, 결과: None...  — 즉시 실패!
> 
> # 더 나은 설계: 중첩 수량자 자체를 제거
> benchmark(r'^[a-zA-Z0-9]++@', malicious_input, "최적 패턴")
> # 최적 패턴: 0.000005초, 결과: None...
> ```
>
> **핵심 포인트**: 소유적 수량자는 ReDoS 공격에 대한 방어 수단이 됩니다. 다만 가장 좋은 방법은 중첩 수량자 자체를 제거하는 패턴 재설계입니다. 소유적 수량자는 패턴 구조를 바꾸기 어려울 때의 보완책으로 활용하세요.

> 🔍 **예시 17.3.2: 원자적 그룹으로 IP 주소 매칭 최적화**
>
> **상황**: 대량의 로그에서 IP 주소를 추출하는데, 매칭 실패 시의 성능을 최적화하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import regex
> 
> # 일반 패턴: 매칭 실패 시 불필요한 백트래킹 발생 가능
> ip_normal = r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b'
> 
> # 원자적 그룹 활용: 각 옥텟 매칭 후 백트래킹 방지
> ip_atomic = r'\b(?>\d{1,3})\.(?>\d{1,3})\.(?>\d{1,3})\.(?>\d{1,3})\b'
> 
> log_lines = [
>     '192.168.1.1 - - [10/Oct/2024:13:55:36]',
>     'Error: connection refused from 10.0.0.255',
>     'Not an IP: 999.999.999.999.999',  # 매칭 실패 케이스
>     'Random text without any IP address at all',  # 매칭 실패 케이스
> ]
> 
> pattern = regex.compile(ip_atomic)
> for line in log_lines:
>     m = pattern.search(line)
>     if m:
>         print(f"IP 발견: {m.group()} ← {line[:40]}")
>     else:
>         print(f"IP 없음: {line[:40]}")
> # IP 발견: 192.168.1.1 ← 192.168.1.1 - - [10/Oct/2024:13:55:
> # IP 발견: 10.0.0.255 ← Error: connection refused from 10.0.0
> # IP 없음: Not an IP: 999.999.999.999.999
> # IP 없음: Random text without any IP address
> ```
>
> **핵심 포인트**: 원자적 그룹은 특히 **매칭 실패가 빈번한 대량 데이터 처리**에서 성능 이점이 큽니다. 각 옥텟의 숫자를 매칭한 후 실패하면 즉시 다음으로 넘어갑니다.

> ⚠️ **주의**: 소유적 수량자와 원자적 그룹은 백트래킹을 제거하므로, 패턴의 매칭 결과가 달라질 수 있습니다. 기존 탐욕적 수량자를 무조건 소유적으로 바꾸면 안 됩니다. "이 수량자가 문자를 돌려줄 필요가 없는가?"를 반드시 확인한 후 적용하세요.

### Section 요약

- `regex` 모듈에서 소유적 수량자(`++`, `*+`, `?+`, `{n,m}+`)와 원자적 그룹(`(?>...)`)을 실제 코드로 사용할 수 있습니다.
- 소유적 수량자는 한 번 매칭한 문자를 절대 돌려주지 않아 불필요한 백트래킹을 방지합니다.
- 원자적 그룹은 그룹 내부 매칭이 완료되면 그룹 안으로 백트래킹하지 않습니다.
- `X++`는 `(?>X+)`와 동일한 동작을 합니다.
- 재앙적 백트래킹/ReDoS 방어에 효과적이지만, 매칭 결과 변화에 주의해야 합니다.

---

## 17.4 퍼지 매칭과 기타 확장

> 💡 **한 줄 요약**: `regex` 모듈의 퍼지 매칭은 오탈자나 약간의 변형이 있는 텍스트에서도 "대략적으로 비슷한" 문자열을 찾아낼 수 있는 강력한 기능입니다.

### 직관적 이해

실제 텍스트 데이터는 깔끔하지 않습니다. 사용자가 "necessary"를 "neccessary"로 입력하거나, OCR로 인식한 텍스트에 오류가 있거나, 수기 입력 데이터에 오탈자가 섞여 있는 경우가 매우 흔합니다.

기존 정규표현식은 **정확한 매칭**만 수행합니다. 패턴에 맞거나 맞지 않거나 둘 중 하나입니다. 하지만 실무에서는 "대략 비슷한 것"을 찾아야 하는 경우가 많습니다. `regex` 모듈의 퍼지 매칭은 이 요구를 충족합니다.

> 📝 **퍼지 매칭(Fuzzy Matching)**: 허용 가능한 오류 범위(삽입, 삭제, 치환의 횟수)를 지정하여, 패턴과 정확히 일치하지 않더라도 "충분히 비슷한" 문자열을 매칭하는 기능입니다.

퍼지 매칭은 마치 검색 엔진의 "혹시 이것을 찾으셨나요?" 기능과 비슷합니다. 완벽하지 않아도 가까운 결과를 찾아줍니다.

### 핵심 개념 설명

#### 퍼지 매칭 기본 문법

퍼지 매칭은 패턴 그룹 뒤에 `{오류조건}`을 붙여 사용합니다:

```python
import regex

# 기본 퍼지 매칭: 최대 1개의 오류 허용
m = regex.search(r'(?:apple){e<=1}', 'aple')  # 'p' 하나가 빠짐
print(m.group())  # 'aple' — 1개 삭제 오류로 매칭 성공

m = regex.search(r'(?:apple){e<=1}', 'appple')  # 'p' 하나가 추가됨
print(m.group())  # 'appple' — 1개 삽입 오류로 매칭 성공

m = regex.search(r'(?:apple){e<=1}', 'apble')  # 'p'이 'b'로 치환됨
print(m.group())  # 'apble' — 1개 치환 오류로 매칭 성공

m = regex.search(r'(?:apple){e<=1}', 'abcle')  # 2개 치환 (p→b, p→c)
print(m)  # None — 2개 오류로 매칭 실패
```

#### 오류 유형 세분화

퍼지 매칭에서 허용하는 오류는 세 가지 유형이 있으며, 각각을 개별적으로 제어할 수 있습니다:

> 📝 **편집 거리(Edit Distance)**: 한 문자열을 다른 문자열로 변환하기 위해 필요한 최소 편집 연산(삽입, 삭제, 치환)의 수. 레벤슈타인 거리(Levenshtein Distance)라고도 합니다.

| 기호 | 오류 유형 | 의미 | 예시 |
|------|----------|------|------|
| `i` | Insertion (삽입) | 대상에 여분의 문자가 삽입됨 | apple → appple |
| `d` | Deletion (삭제) | 대상에서 문자가 빠짐 | apple → aple |
| `s` | Substitution (치환) | 대상의 문자가 다른 문자로 바뀜 | apple → apble |
| `e` | Error (전체) | 삽입+삭제+치환의 합계 | 모든 유형의 합 |

```python
import regex

# 치환만 허용 (삽입/삭제 불가)
m = regex.search(r'(?:apple){s<=2}', 'axble')
print(m.group() if m else "None")  # 'axble' — 2개 치환으로 매칭

m = regex.search(r'(?:apple){s<=2}', 'aple')
print(m)  # None — 삭제는 허용하지 않으므로 매칭 실패

# 삭제만 허용 (삽입/치환 불가)
m = regex.search(r'(?:apple){d<=1}', 'aple')
print(m.group() if m else "None")  # 'aple' — 1개 삭제로 매칭

# 삽입만 허용
m = regex.search(r'(?:apple){i<=1}', 'appple')
print(m.group() if m else "None")  # 'appple' — 1개 삽입으로 매칭

# 복합 조건: 치환 최대 1개, 삭제 최대 1개, 전체 오류 최대 2개
m = regex.search(r'(?:apple){s<=1,d<=1,e<=2}', 'aple')
print(m.group() if m else "None")  # 'aple'
```

#### 퍼지 매칭의 매칭 정보 확인

퍼지 매칭의 결과에서 어떤 종류의 오류가 발생했는지 확인할 수 있습니다:

```python
import regex

m = regex.search(r'(?:python){e<=2}', 'pyhton')  # 'th' → 'ht' (2개 치환)
if m:
    print(f"매칭된 문자열: {m.group()}")
    print(f"퍼지 카운트: {m.fuzzy_counts}")  # (삽입, 삭제, 치환) 튜플
    print(f"퍼지 변경: {m.fuzzy_changes}")    # 각 변경의 위치 정보
# 매칭된 문자열: pyhton
# 퍼지 카운트: (0, 0, 2)  — 치환 2개
# 퍼지 변경: ([], [], [2, 3])  — 인덱스 2, 3에서 치환
```

#### `BESTMATCH` 플래그

기본적으로 퍼지 매칭은 허용 오류 범위 내에서 **첫 번째 매칭**을 반환합니다. `BESTMATCH` 플래그를 사용하면 오류가 **가장 적은** 매칭을 반환합니다:

```python
import regex

text = "He said 'hello' and then said 'hallo'"

# 기본: 첫 번째 매칭
m = regex.search(r'(?:hello){e<=1}', text)
print(m.group(), m.fuzzy_counts)
# 'hello' (0, 0, 0) — 정확한 매칭이 먼저 발견

# BESTMATCH: 오류가 가장 적은 매칭
results = regex.findall(r'(?:hello){e<=1}', text, flags=regex.BESTMATCH)
print(results)
# ['hello', 'hallo'] — 둘 다 오류 1개 이내
```

#### 기타 `regex` 모듈 확장 기능

**부분 매칭 (Partial Matching):**

```python
import regex

# 부분 매칭: 입력이 완전하지 않아도 "매칭될 가능성"을 판단
pattern = regex.compile(r'\d{3}-\d{4}-\d{4}')

# 완전한 입력
m = pattern.fullmatch('010-1234-5678')
print(f"완전 매칭: {m is not None}")  # True

# 불완전한 입력 — 부분 매칭으로 "가능성" 판단
m = pattern.fullmatch('010-12', partial=True)
print(f"부분 매칭: {m is not None}")  # True — 완성될 가능성 있음

m = pattern.fullmatch('abc', partial=True)
print(f"부분 매칭: {m is not None}")  # False — 완성 가능성 없음
```

부분 매칭은 실시간 입력 검증(사용자가 타이핑하는 동안 형식 검사)에 유용합니다.

**`\G` 앵커:**

```python
import regex

# \G: 이전 매칭이 끝난 위치에 앵커
text = "aaa bbb ccc"
# \G 뒤에서만 매칭 시작 → 연속된 매칭
for m in regex.finditer(r'\G\w', text):
    print(m.group(), end='')
print()
# aaa  — 'aaa' 이후 공백에서 \G\w가 실패하여 멈춤
```

### 상세 예시

> 🔍 **예시 17.4.1: OCR 오류가 있는 텍스트에서 회사명 찾기**
>
> **상황**: OCR로 스캔한 문서에서 회사명 "Samsung Electronics"를 찾아야 하는데, OCR 인식 오류로 철자가 약간 틀릴 수 있습니다.
>
> **풀이/설명**:
> ```python
> import regex
> 
> ocr_texts = [
>     "Sarnsung Electronics reported record profits",     # m → rn
>     "Samsumg Electronlcs announced new products",       # n → m, i → l
>     "Samsung Electoronics filed a patent",              # 'ro' 삽입
>     "Samsung Electronics leads the market",             # 정확한 텍스트
> ]
> 
> # 각 단어에 최대 2개 오류 허용
> pattern = regex.compile(
>     r'(?:Samsung){e<=2}\s+(?:Electronics){e<=2}',
>     regex.IGNORECASE
> )
> 
> for text in ocr_texts:
>     m = pattern.search(text)
>     if m:
>         print(f"발견: '{m.group()}' (오류: {m.fuzzy_counts})")
>     else:
>         print(f"미발견: {text[:50]}")
> # 발견: 'Sarnsung Electronics' (오류: (0, 0, 1))
> # 발견: 'Samsumg Electronlcs' (오류: (0, 0, 2))
> # 발견: 'Samsung Electoronics' (오류: (2, 0, 0))
> # 발견: 'Samsung Electronics' (오류: (0, 0, 0))
> ```
>
> **핵심 포인트**: 퍼지 매칭은 OCR 텍스트, 수기 입력 데이터 등 정확성이 보장되지 않는 데이터에서 "비슷한" 텍스트를 찾는 데 매우 효과적입니다. `fuzzy_counts`로 오류 정도도 확인할 수 있습니다.

> 🔍 **예시 17.4.2: 실시간 입력 검증에 부분 매칭 활용**
>
> **상황**: 사용자가 전화번호를 입력하는 동안 실시간으로 형식을 검증하고 피드백을 주고 싶습니다.
>
> **풀이/설명**:
> ```python
> import regex
> 
> phone_pattern = regex.compile(r'010-\d{4}-\d{4}')
> 
> # 사용자의 타이핑 시뮬레이션
> typing_sequence = ['0', '01', '010', '010-', '010-1', '010-12',
>                     '010-123', '010-1234', '010-1234-',
>                     '010-1234-5', '010-1234-56', '010-1234-567',
>                     '010-1234-5678']
> 
> for typed in typing_sequence:
>     full = phone_pattern.fullmatch(typed)
>     partial = phone_pattern.fullmatch(typed, partial=True)
>     
>     if full:
>         status = "✅ 완성"
>     elif partial:
>         status = "⏳ 입력 중 (올바른 형식)"
>     else:
>         status = "❌ 잘못된 형식"
>     
>     print(f"'{typed}' → {status}")
> # '0' → ⏳ 입력 중 (올바른 형식)
> # '01' → ⏳ 입력 중 (올바른 형식)
> # ...
> # '010-1234-5678' → ✅ 완성
> ```
>
> **핵심 포인트**: `partial=True`는 "이 입력이 완성되면 패턴에 맞을 수 있는가?"를 판단합니다. 실시간 입력 검증 UI에서 즉각적인 피드백을 줄 때 유용합니다.

> ⚠️ **주의**: 퍼지 매칭은 매우 강력하지만, 오류 허용 범위를 너무 크게 설정하면 의도하지 않은 문자열까지 매칭될 수 있습니다. 또한 퍼지 매칭은 일반 매칭보다 계산 비용이 높으므로, 대량 데이터 처리 시 성능을 확인하세요.

### Section 요약

- 퍼지 매칭은 `(?:패턴){e<=N}` 문법으로, 최대 N개의 오류(삽입/삭제/치환)를 허용하여 매칭합니다.
- `i`, `d`, `s`로 오류 유형별 허용 횟수를 세밀하게 제어할 수 있습니다.
- `m.fuzzy_counts`로 매칭된 결과의 오류 유형과 횟수를 확인할 수 있습니다.
- `BESTMATCH` 플래그는 오류가 가장 적은 매칭을 우선합니다.
- `partial=True`는 불완전한 입력의 매칭 가능성을 판단하는 부분 매칭 기능입니다.

---

## 17.5 정규표현식의 대안

> 💡 **한 줄 요약**: 정규표현식은 만능이 아닙니다. 상황에 따라 문자열 메서드, `parse` 라이브러리, `pyparsing`, PEG 파서 등이 더 적합한 도구가 될 수 있습니다.

### 직관적 이해

지금까지 17개 Chapter에 걸쳐 정규표현식의 거의 모든 것을 학습했습니다. 여러분은 이제 정규표현식의 전문가입니다. 하지만 진정한 전문가는 **자신의 도구가 최선이 아닌 순간**도 아는 사람입니다.

비유하자면, 정규표현식은 매우 뛰어난 "스위스 아미 나이프"입니다. 대부분의 텍스트 처리 작업을 해낼 수 있지만, 때로는 전문 도구(전동 드릴, 톱, 렌치)가 훨씬 효율적인 경우가 있습니다.

> 🔗 **연결**: Chapter 16의 Section 16.6 "정규표현식을 사용하지 말아야 할 때"에서 이 주제를 간략히 다뤘습니다. 이 Section에서는 구체적인 대안 도구들을 상세히 살펴봅니다.

### 핵심 개념 설명

#### 대안 1: 파이썬 문자열 메서드

가장 가까운 대안은 이미 알고 있는 파이썬 문자열 메서드입니다.

**문자열 메서드가 더 나은 경우:**

```python
import re

text = "hello world"

# ❌ 정규표현식 — 불필요하게 복잡
result = re.search(r'^hello', text)
has_prefix = result is not None

# ✅ 문자열 메서드 — 간단하고 빠름
has_prefix = text.startswith('hello')


# ❌ 정규표현식
parts = re.split(r',', "a,b,c,d")

# ✅ 문자열 메서드
parts = "a,b,c,d".split(',')


# ❌ 정규표현식
new_text = re.sub(r'world', 'Python', text)

# ✅ 문자열 메서드
new_text = text.replace('world', 'Python')
```

**문자열 메서드의 장점:**
- 정규표현식보다 **2~10배 빠른** 성능
- 코드가 더 읽기 쉽고 의도가 명확
- 정규표현식 문법 오류 가능성 없음
- 메타문자 이스케이프 불필요

**판단 기준:**

| 상황 | 추천 도구 |
|------|----------|
| 고정 문자열 검색/치환 | 문자열 메서드 (`in`, `find`, `replace`) |
| 접두사/접미사 확인 | `startswith()`, `endswith()` |
| 단일 구분자로 분할 | `str.split()` |
| 공백 제거 | `strip()`, `lstrip()`, `rstrip()` |
| 대소문자 변환 | `upper()`, `lower()`, `title()` |
| **패턴** 기반 검색/치환 | 정규표현식 |
| 복잡한 구분자로 분할 | 정규표현식 |
| 구조화된 데이터 추출 | 정규표현식 또는 파서 |

#### 대안 2: `parse` 라이브러리

> 📝 **`parse` 라이브러리**: 파이썬 `format()` 문자열의 역방향 도구. 서식 문자열(format string)을 사용하여 텍스트에서 값을 추출합니다. "정규표현식의 가독성 높은 대안"으로 불립니다.

```bash
pip install parse
```

`parse`는 `format()`의 반대 동작을 합니다:

```python
from parse import parse, search, findall

# format()의 반대 — 구조화된 텍스트에서 값 추출
result = parse("이름: {name}, 나이: {age:d}세", "이름: 홍길동, 나이: 30세")
print(result['name'])   # '홍길동'
print(result['age'])    # 30 (정수로 자동 변환!)

# search — 텍스트 중간에서 패턴 찾기
result = search("가격: {price:d}원", "상품A의 가격: 15000원입니다")
print(result['price'])  # 15000

# findall — 모든 매칭 찾기
results = findall("({item}, {qty:d}개)", "주문: (사과, 3개), (바나나, 5개)")
for r in results:
    print(f"{r['item']}: {r['qty']}개")
# 사과: 3개
# 바나나: 5개
```

**`parse`의 서식 지정자:**

| 지정자 | 의미 | 정규표현식 등가 |
|--------|------|----------------|
| `{}` | 임의 문자열 | `.+?` |
| `{:d}` | 정수 | `[-+]?\d+` |
| `{:f}` | 실수 | `[-+]?\d*\.?\d+` |
| `{:w}` | 단어 (알파벳+숫자) | `\w+` |
| `{:l}` | 문자만 | `[a-zA-Z]+` |

**`parse` vs 정규표현식 비교:**

```python
import re
from parse import parse

log_line = '192.168.1.1 - - [10/Oct/2024:13:55:36 +0900] "GET /index.html HTTP/1.1" 200 2326'

# 정규표현식 — 강력하지만 읽기 어려움
m = re.match(
    r'(\d+\.\d+\.\d+\.\d+) - - '
    r'\[(\d{2}/\w{3}/\d{4}:\d{2}:\d{2}:\d{2} [+\-]\d{4})\] '
    r'"(\w+) (\S+) (\S+)" (\d+) (\d+)',
    log_line
)
if m:
    print(f"IP: {m.group(1)}, 상태: {m.group(6)}")

# parse — 직관적이고 가독성 높음
result = parse(
    '{ip} - - [{timestamp}] "{method} {path} {protocol}" {status:d} {size:d}',
    log_line
)
if result:
    print(f"IP: {result['ip']}, 상태: {result['status']}")
# 두 방법 모두 동일한 결과:
# IP: 192.168.1.1, 상태: 200
```

**`parse`의 한계:**
- 정규표현식만큼 세밀한 패턴 제어가 불가능합니다.
- 전후방 탐색, 수량자 세밀 제어, 조건 패턴 등은 지원하지 않습니다.
- 복잡한 중첩 구조나 재귀적 패턴은 처리할 수 없습니다.
- 성능은 정규표현식보다 일반적으로 느립니다.

#### 대안 3: `pyparsing` 라이브러리

> 📝 **`pyparsing`**: 파이썬으로 작성된 파싱 라이브러리. 정규표현식보다 더 강력한 문법 기반 파서를 파이썬 코드로 직접 구성할 수 있습니다. BNF(배커스-나우르 형식)와 유사한 방식으로 문법을 정의합니다.

```bash
pip install pyparsing
```

`pyparsing`은 정규표현식으로는 처리하기 어려운 **구조화된 문법**을 파싱하는 데 탁월합니다:

```python
from pyparsing import Word, alphas, nums, Suppress, Group, OneOrMore

# 간단한 키-값 파서 정의
key = Word(alphas + '_')
value = Word(alphas + nums + '_.-')
pair = Group(key + Suppress('=') + value)
config_parser = OneOrMore(pair)

# 파싱 실행
config_text = "host=localhost port=8080 debug=true"
result = config_parser.parseString(config_text)

for item in result:
    print(f"키: {item[0]}, 값: {item[1]}")
# 키: host, 값: localhost
# 키: port, 값: 8080
# 키: debug, 값: true
```

**`pyparsing`이 정규표현식보다 나은 경우:**

```python
from pyparsing import (nestedExpr, Word, alphanums, 
                        infixNotation, opAssoc, Literal)

# 1. 중첩된 괄호 구조 — 정규표현식으로는 불가능!
nested = nestedExpr('(', ')')
result = nested.parseString("(a (b c (d e)) f)")
print(result.asList())
# [['a', ['b', 'c', ['d', 'e']], 'f']]

# 2. 수학 표현식 파싱 — 연산자 우선순위 처리
# 정규표현식으로는 사실상 불가능한 작업
integer = Word(alphanums)
expr = infixNotation(integer, [
    (Literal('*'), 2, opAssoc.LEFT),
    (Literal('+'), 2, opAssoc.LEFT),
])
result = expr.parseString("3+4*5+2")
print(result.asList())
# [['3', '+', ['4', '*', '5'], '+', '2']]
# ↑ 연산자 우선순위가 올바르게 반영됨
```

> 💬 **참고**: 중첩된 괄호 구조는 정규표현식의 근본적 한계 중 하나입니다. 정규표현식은 **정규 언어(regular language)**만 인식할 수 있고, 중첩 구조를 인식하려면 **문맥 자유 문법(context-free grammar)**이 필요합니다. `pyparsing`은 이 수준의 파싱이 가능합니다.

#### 대안 4: PEG 파서 개념

> 📝 **PEG 파서(Parsing Expression Grammar)**: 형식 문법을 기반으로 문자열을 파싱하는 방법. 정규표현식보다 강력한 표현력을 가지며, 프로그래밍 언어의 구문 분석 등에 사용됩니다.

파이썬에서는 `parsimonious`, `pe`, `tatsu` 등의 PEG 파서 라이브러리가 있습니다. PEG 파서는 다음과 같은 경우에 적합합니다:

- 프로그래밍 언어나 DSL(도메인 특화 언어)의 파싱
- 복잡한 재귀적 구조(JSON, XML, HTML 등)의 파싱
- 수학 표현식, 쿼리 언어 등의 해석

```python
# 개념적 예시 — PEG 문법 정의
# JSON 값을 파싱하는 PEG 문법 (pseudo-code)
"""
value    = string / number / object / array / 'true' / 'false' / 'null'
string   = '"' [^"]* '"'
number   = [0-9]+
object   = '{' (pair (',' pair)*)? '}'
pair     = string ':' value
array    = '[' (value (',' value)*)? ']'
"""
# 이런 재귀적 정의는 정규표현식으로 불가능하지만
# PEG 파서로는 자연스럽게 표현 가능
```

#### 도구 선택 의사결정 가이드

```
텍스트 처리 작업이 필요하다
    │
    ├── 고정 문자열 매칭인가? ──Yes──→ 문자열 메서드
    │
    ├── 단순한 패턴인가? ──Yes──→ 정규표현식 (re)
    │
    ├── 유니코드 카테고리/퍼지 매칭이 필요한가? ──Yes──→ regex 모듈
    │
    ├── 구조화된 텍스트에서 값 추출인가? ──Yes──→ parse 라이브러리
    │
    ├── 중첩/재귀 구조가 있는가? ──Yes──→ pyparsing 또는 PEG 파서
    │
    └── 프로그래밍 언어/복잡한 문법인가? ──Yes──→ PEG 파서 또는 전용 파서
```

### 상세 예시

> 🔍 **예시 17.5.1: 같은 작업을 다른 도구로 — 로그 파싱 비교**
>
> **상황**: 웹 서버 로그에서 IP, 요청 경로, 상태 코드를 추출해야 합니다. 세 가지 도구로 같은 작업을 수행해 봅시다.
>
> **풀이/설명**:
> ```python
> log = '192.168.1.100 - - [15/Mar/2024:09:30:15 +0900] "GET /api/users HTTP/1.1" 200 1234'
> 
> # 방법 1: 정규표현식
> import re
> m = re.search(
>     r'([\d.]+) .+? "(GET|POST|PUT|DELETE) (\S+) .+?" (\d{3})',
>     log
> )
> print(f"[regex] IP={m.group(1)}, Path={m.group(3)}, Status={m.group(4)}")
> 
> # 방법 2: parse
> from parse import parse
> r = parse(
>     '{ip} - - [{timestamp}] "{method} {path} {proto}" {status:d} {size:d}',
>     log
> )
> print(f"[parse] IP={r['ip']}, Path={r['path']}, Status={r['status']}")
> 
> # 방법 3: 문자열 분할 (구조가 고정적일 때)
> parts = log.split('"')
> ip = parts[0].split()[0]
> request_parts = parts[1].split()
> method, path = request_parts[0], request_parts[1]
> status = parts[2].strip().split()[0]
> print(f"[split] IP={ip}, Path={path}, Status={status}")
> 
> # 세 방법 모두 동일한 결과:
> # IP=192.168.1.100, Path=/api/users, Status=200
> ```
>
> **핵심 포인트**: 
> - **정규표현식**: 가장 유연하지만, 패턴이 복잡하면 읽기 어려움
> - **`parse`**: 가장 읽기 쉽지만, 세밀한 패턴 제어 어려움
> - **문자열 분할**: 가장 빠르지만, 형식이 조금만 바뀌면 깨짐

> 🔍 **예시 17.5.2: 정규표현식이 부적합한 경우 — HTML 파싱**
>
> **상황**: HTML에서 특정 태그의 내용을 추출하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import re
> 
> html = """
> <div class="content">
>     <p>첫 번째 문단</p>
>     <div class="nested">
>         <p>중첩된 문단</p>
>     </div>
>     <p>마지막 문단</p>
> </div>
> """
> 
> # ❌ 정규표현식으로 div 내용 추출 시도 — 중첩 구조에 취약
> m = re.search(r'<div class="content">(.*?)</div>', html, re.DOTALL)
> if m:
>     print("regex 결과:", m.group(1).strip()[:50])
>     # 중첩된 </div>에서 잘못 끊길 수 있음!
> 
> # ✅ 적절한 도구: HTML 파서 사용
> from html.parser import HTMLParser
> # 또는 BeautifulSoup, lxml 등 전용 파서 사용
> # from bs4 import BeautifulSoup
> # soup = BeautifulSoup(html, 'html.parser')
> # content = soup.find('div', class_='content')
> 
> print("\n⚠️ HTML/XML/JSON 등 중첩 구조는 전용 파서를 사용하세요!")
> ```
>
> **핵심 포인트**: HTML, XML, JSON 같은 중첩 구조(nested structure)는 정규표현식의 근본적 한계입니다. 이런 데이터에는 BeautifulSoup, lxml, `json` 모듈 등 전용 파서를 사용하는 것이 올바른 접근입니다.

> ⚠️ **주의**: "정규표현식으로 HTML을 파싱하면 안 된다"는 것은 프로그래밍 커뮤니티의 유명한 격언입니다. 간단한 HTML 태그 추출은 정규표현식으로 가능하지만, 중첩된 태그, 속성 내의 꺾쇠 괄호, 주석 등을 올바르게 처리하려면 반드시 전용 파서를 사용하세요.

### Section 요약

- 고정 문자열 작업에는 문자열 메서드(`str.split()`, `str.replace()` 등)가 더 빠르고 읽기 쉽습니다.
- `parse` 라이브러리는 구조화된 텍스트에서 값을 추출할 때 정규표현식보다 가독성이 높습니다.
- `pyparsing`은 중첩 구조, 연산자 우선순위 등 정규표현식으로 처리할 수 없는 문법 파싱에 적합합니다.
- HTML/XML/JSON 등 중첩 구조에는 반드시 전용 파서를 사용해야 합니다.
- 진정한 전문가는 정규표현식의 한계를 알고, 상황에 맞는 최적의 도구를 선택합니다.

---

## 17.6 종합 프로젝트

> 💡 **한 줄 요약**: 지금까지 배운 모든 정규표현식 지식을 종합하여, 실전 수준의 프로젝트를 설계부터 최적화까지 완성합니다.

### 직관적 이해

이 Section은 과정 전체의 대미를 장식하는 **종합 프로젝트**입니다. 지금까지 17개 Chapter에 걸쳐 배운 모든 것—기본 문법, `re` 모듈, 고급 패턴, 실전 응용, 엔진 원리, 성능 최적화, 서드파티 확장—을 하나의 완성된 프로젝트로 통합합니다.

개별 기술을 아는 것과, 그것들을 조합하여 완성된 솔루션을 만들어내는 것은 전혀 다른 능력입니다. 이 Section에서는 **설계 → 구현 → 테스트 → 최적화**의 전체 워크플로우를 경험합니다.

### 핵심 개념 설명

#### 정규표현식 프로젝트 워크플로우

실전 프로젝트에서 정규표현식을 다룰 때의 체계적인 접근법입니다:

```
1. 요구사항 분석
   └── 입력 데이터의 형식과 변형 파악
   └── 출력으로 원하는 결과 명확히 정의
   └── 엣지 케이스 목록 작성

2. 패턴 설계
   └── 작은 부분부터 시작하여 점진적으로 확장
   └── regex101 등으로 실시간 테스트
   └── re.VERBOSE를 활용한 가독성 확보

3. 구현
   └── 적절한 함수 선택 (search, findall, sub 등)
   └── 캡처 그룹 또는 명명 그룹으로 결과 구조화
   └── 예외 처리와 매칭 실패 대응

4. 테스트
   └── 정상 케이스, 경계 케이스, 예외 케이스
   └── 매칭 성공과 매칭 실패 모두 검증
   └── 비정상 입력에 대한 안전성 확인

5. 최적화
   └── 성능 벤치마킹 (timeit)
   └── 불필요한 백트래킹 제거
   └── 필요시 regex 모듈이나 대안 도구 검토
```

#### 프로젝트 1: 마크다운 간이 파서

이 프로젝트는 마크다운 텍스트의 핵심 요소를 HTML로 변환하는 간이 파서를 만듭니다.

**지원할 마크다운 요소:**
- 제목 (`#`, `##`, `###`)
- 굵게 (`**text**`)
- 이탤릭 (`*text*`)
- 인라인 코드 (`` `code` ``)
- 링크 (`[text](url)`)
- 순서 없는 목록 (`- item`)

```python
import re

def markdown_to_html(text):
    """마크다운 텍스트를 HTML로 변환하는 간이 파서"""
    lines = text.split('\n')
    html_lines = []
    in_list = False
    
    for line in lines:
        stripped = line.strip()
        
        # 빈 줄 처리
        if not stripped:
            if in_list:
                html_lines.append('</ul>')
                in_list = False
            html_lines.append('')
            continue
        
        # 제목 처리: # ~ ###
        heading_match = re.match(r'^(#{1,3})\s+(.+)$', stripped)
        if heading_match:
            if in_list:
                html_lines.append('</ul>')
                in_list = False
            level = len(heading_match.group(1))
            content = heading_match.group(2)
            content = _inline_format(content)
            html_lines.append(f'<h{level}>{content}</h{level}>')
            continue
        
        # 순서 없는 목록 처리
        list_match = re.match(r'^[-*]\s+(.+)$', stripped)
        if list_match:
            if not in_list:
                html_lines.append('<ul>')
                in_list = True
            content = _inline_format(list_match.group(1))
            html_lines.append(f'  <li>{content}</li>')
            continue
        
        # 일반 문단
        if in_list:
            html_lines.append('</ul>')
            in_list = False
        content = _inline_format(stripped)
        html_lines.append(f'<p>{content}</p>')
    
    if in_list:
        html_lines.append('</ul>')
    
    return '\n'.join(html_lines)


def _inline_format(text):
    """인라인 마크다운 서식을 HTML로 변환"""
    
    # 인라인 코드: `code` → <code>code</code>
    # 다른 변환보다 먼저 처리하여 코드 안의 마크업이 변환되지 않도록 함
    text = re.sub(r'`([^`]+)`', r'<code>\1</code>', text)
    
    # 굵게: **text** → <strong>text</strong>
    # 이탤릭보다 먼저 처리 (** 가 *를 포함하므로)
    text = re.sub(r'\*\*(.+?)\*\*', r'<strong>\1</strong>', text)
    
    # 이탤릭: *text* → <em>text</em>
    # 이미 처리된 <strong> 안의 *는 건드리지 않음
    text = re.sub(r'(?<!\*)\*(?!\*)(.+?)(?<!\*)\*(?!\*)', r'<em>\1</em>', text)
    
    # 링크: [text](url) → <a href="url">text</a>
    text = re.sub(r'\[([^\]]+)\]\(([^)]+)\)', r'<a href="\2">\1</a>', text)
    
    return text


# 테스트
sample_md = """# 정규표현식 가이드

정규표현식은 **매우 강력한** 텍스트 처리 도구입니다.

## 주요 기능

- *패턴 매칭*으로 텍스트 검색
- `re.sub()`으로 **텍스트 치환**
- [공식 문서](https://docs.python.org/3/library/re.html) 참고

### 참고 사항

인라인 `코드`와 **굵은 글씨**를 함께 사용할 수 있습니다.
"""

result = markdown_to_html(sample_md)
print(result)
```

> 🔗 **연결**: 이 프로젝트에서는 다음 Chapter의 지식이 종합적으로 활용됩니다:
> - Chapter 2: 앵커(`^`, `$`), 이스케이프
> - Chapter 4: 수량자(`+`, `+?`, `{1,3}`)
> - Chapter 5: `re.match()`, `re.sub()`
> - Chapter 6: 캡처 그룹과 `group(n)`
> - Chapter 7: `re.sub()`의 역참조 치환
> - Chapter 8: 비캡처 그룹
> - Chapter 9: 전후방 탐색(`(?<!\*)`, `(?!\*)`)

**이 프로젝트의 핵심 설계 판단:**

1. **처리 순서**: 인라인 코드 → 굵게 → 이탤릭 순으로 처리합니다. 코드 블록 안의 `*`가 서식으로 해석되지 않도록 먼저 처리하고, `**`가 `*`를 포함하므로 굵게를 이탤릭보다 먼저 처리합니다.

2. **비탐욕적 수량자**: `.+?`를 사용하여 가장 가까운 닫는 기호까지만 매칭합니다. `.+`(탐욕적)을 쓰면 `**a** and **b**`에서 `a** and **b`를 매칭해버립니다.

3. **전후방 탐색**: 이탤릭 처리에서 `(?<!\*)\*(?!\*)`로 단독 `*`만 매칭합니다. `**` 안의 `*`를 잘못 인식하지 않기 위한 것입니다.

#### 프로젝트 2: 로그 분석 보고서 생성기

이 프로젝트는 웹 서버 로그를 분석하여 통계 보고서를 생성하는 도구입니다.

```python
import re
from collections import Counter, defaultdict
from datetime import datetime

def analyze_log(log_text):
    """웹 서버 로그를 분석하여 통계 보고서를 생성합니다."""
    
    # Apache Combined Log Format 패턴
    # 명명 그룹을 활용하여 가독성 확보
    log_pattern = re.compile(
        r'(?P<ip>\d{1,3}(?:\.\d{1,3}){3})\s'     # IP 주소
        r'(?P<ident>\S+)\s'                          # 식별자
        r'(?P<user>\S+)\s'                           # 사용자
        r'\[(?P<timestamp>[^\]]+)\]\s'               # 타임스탬프
        r'"(?P<method>\w+)\s'                        # HTTP 메서드
        r'(?P<path>\S+)\s'                           # 요청 경로
        r'(?P<protocol>[^"]+)"\s'                    # 프로토콜
        r'(?P<status>\d{3})\s'                       # 상태 코드
        r'(?P<size>\d+|-)',                           # 응답 크기
        re.VERBOSE
    )
    
    # 통계 수집
    stats = {
        'total_requests': 0,
        'ip_counter': Counter(),
        'path_counter': Counter(),
        'status_counter': Counter(),
        'method_counter': Counter(),
        'errors': [],
        'hourly_traffic': defaultdict(int),
    }
    
    for line in log_text.strip().split('\n'):
        if not line.strip():
            continue
            
        m = log_pattern.match(line)
        if not m:
            stats['errors'].append(f"파싱 실패: {line[:80]}...")
            continue
        
        data = m.groupdict()
        stats['total_requests'] += 1
        stats['ip_counter'][data['ip']] += 1
        stats['path_counter'][data['path']] += 1
        stats['status_counter'][data['status']] += 1
        stats['method_counter'][data['method']] += 1
        
        # 시간대별 트래픽
        ts_match = re.search(r':(\d{2}):', data['timestamp'])
        if ts_match:
            hour = ts_match.group(1)
            stats['hourly_traffic'][hour] += 1
    
    return stats


def generate_report(stats):
    """분석 통계를 사람이 읽기 쉬운 보고서로 출력합니다."""
    
    report = []
    report.append("=" * 60)
    report.append("              웹 서버 로그 분석 보고서")
    report.append("=" * 60)
    report.append(f"\n총 요청 수: {stats['total_requests']}")
    
    report.append("\n--- 상태 코드 분포 ---")
    for status, count in stats['status_counter'].most_common():
        bar = '█' * (count * 20 // stats['total_requests'])
        report.append(f"  {status}: {count:>5}건 {bar}")
    
    report.append("\n--- 상위 5개 요청 IP ---")
    for ip, count in stats['ip_counter'].most_common(5):
        report.append(f"  {ip:<20} {count:>5}건")
    
    report.append("\n--- 상위 5개 요청 경로 ---")
    for path, count in stats['path_counter'].most_common(5):
        report.append(f"  {path:<30} {count:>5}건")
    
    report.append("\n--- HTTP 메서드 분포 ---")
    for method, count in stats['method_counter'].most_common():
        report.append(f"  {method:<8} {count:>5}건")
    
    if stats['errors']:
        report.append(f"\n--- 파싱 오류 ({len(stats['errors'])}건) ---")
        for err in stats['errors'][:3]:
            report.append(f"  {err}")
    
    report.append("\n" + "=" * 60)
    return '\n'.join(report)


# 테스트 데이터
sample_log = """192.168.1.1 - frank [10/Oct/2024:13:55:36 +0900] "GET /index.html HTTP/1.1" 200 2326
192.168.1.1 - frank [10/Oct/2024:13:56:10 +0900] "GET /api/users HTTP/1.1" 200 1234
10.0.0.1 - - [10/Oct/2024:14:02:15 +0900] "POST /api/login HTTP/1.1" 401 512
10.0.0.1 - - [10/Oct/2024:14:02:30 +0900] "POST /api/login HTTP/1.1" 200 256
172.16.0.50 - admin [10/Oct/2024:14:15:22 +0900] "GET /admin/dashboard HTTP/1.1" 200 8192
172.16.0.50 - admin [10/Oct/2024:14:15:45 +0900] "DELETE /api/users/5 HTTP/1.1" 403 128
192.168.1.1 - frank [10/Oct/2024:15:30:00 +0900] "GET /static/style.css HTTP/1.1" 200 4096
192.168.1.100 - - [10/Oct/2024:15:31:12 +0900] "GET /nonexistent HTTP/1.1" 404 256
192.168.1.100 - - [10/Oct/2024:16:00:00 +0900] "GET /index.html HTTP/1.1" 200 2326
10.0.0.1 - - [10/Oct/2024:16:05:30 +0900] "GET /api/users HTTP/1.1" 200 1234"""

stats = analyze_log(sample_log)
print(generate_report(stats))
```

> 🔗 **연결**: 이 프로젝트에서 활용된 핵심 기법들:
> - Chapter 8: 명명 그룹 `(?P<name>...)` — 로그 필드에 의미 있는 이름 부여
> - Chapter 8: 비캡처 그룹 `(?:...)` — IP 주소의 반복 패턴
> - Chapter 10: `re.VERBOSE` — 긴 패턴의 가독성 확보
> - Chapter 14: 로그 파싱의 실전 기법
> - Chapter 16: 성능을 고려한 패턴 설계

#### 프로젝트 3: 데이터 정제 파이프라인

이 프로젝트는 비정형 텍스트 데이터를 단계적으로 정제하여 구조화된 형태로 변환합니다.

```python
import re
import regex  # 유니코드 카테고리 활용

class DataCleaningPipeline:
    """정규표현식 기반 데이터 정제 파이프라인"""
    
    def __init__(self):
        self.steps = []
        self.log = []
    
    def add_step(self, name, func):
        """정제 단계를 추가합니다."""
        self.steps.append((name, func))
        return self  # 메서드 체이닝 지원
    
    def process(self, text):
        """모든 정제 단계를 순서대로 적용합니다."""
        result = text
        self.log = []
        
        for name, func in self.steps:
            before = result
            result = func(result)
            if before != result:
                self.log.append(f"[변경] {name}")
            else:
                self.log.append(f"[유지] {name}")
        
        return result
    
    def get_log(self):
        return '\n'.join(self.log)


# 정제 함수 정의

def normalize_whitespace(text):
    """연속 공백을 단일 공백으로, 줄바꿈 정규화"""
    text = re.sub(r'[ \t]+', ' ', text)          # 연속 공백/탭 → 단일 공백
    text = re.sub(r'\n{3,}', '\n\n', text)        # 3개 이상 줄바꿈 → 2개
    text = re.sub(r'^ +| +$', '', text, flags=re.MULTILINE)  # 줄 앞뒤 공백 제거
    return text.strip()

def normalize_punctuation(text):
    """구두점 정규화"""
    text = re.sub(r'\.{2,}', '…', text)           # .. 이상 → …
    text = re.sub(r'!{2,}', '!', text)             # !! 이상 → !
    text = re.sub(r'\?{2,}', '?', text)            # ?? 이상 → ?
    text = re.sub(r'~{2,}', '~', text)             # ~~ 이상 → ~
    return text

def normalize_phone_numbers(text):
    """전화번호 형식 통일 (XXX-XXXX-XXXX)"""
    # 다양한 형식의 전화번호를 통일
    pattern = r'(01[016789])[\s.-]?(\d{3,4})[\s.-]?(\d{4})'
    return re.sub(pattern, r'\1-\2-\3', text)

def normalize_dates(text):
    """날짜 형식 통일 (YYYY-MM-DD)"""
    # 2024.03.15, 2024/03/15 → 2024-03-15
    text = re.sub(r'(\d{4})[./](\d{1,2})[./](\d{1,2})', r'\1-\2-\3', text)
    # 제로패딩: 2024-3-5 → 2024-03-05
    def zero_pad(m):
        y, mo, d = m.group(1), m.group(2), m.group(3)
        return f"{y}-{mo.zfill(2)}-{d.zfill(2)}"
    text = re.sub(r'(\d{4})-(\d{1,2})-(\d{1,2})', zero_pad, text)
    return text

def remove_personal_info(text):
    """개인정보 마스킹"""
    # 이메일 마스킹
    text = re.sub(
        r'([a-zA-Z0-9._%+-])([a-zA-Z0-9._%+-]*)(@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})',
        lambda m: m.group(1) + '*' * len(m.group(2)) + m.group(3),
        text
    )
    # 주민등록번호 형식 마스킹
    text = re.sub(r'(\d{6})-(\d)\d{6}', r'\1-\2******', text)
    return text

def extract_structured_data(text):
    """구조화된 데이터 추출 (명명 그룹 활용)"""
    records = []
    pattern = re.compile(
        r'이름[:\s]+(?P<name>\S+)\s*'
        r'(?:,\s*)?연락처[:\s]+(?P<phone>[\d\s.-]+)\s*'
        r'(?:,\s*)?(?:이메일|email)[:\s]+(?P<email>\S+)',
        re.IGNORECASE
    )
    for m in pattern.finditer(text):
        records.append(m.groupdict())
    return records


# 파이프라인 실행 예시
pipeline = DataCleaningPipeline()
pipeline.add_step("공백 정규화", normalize_whitespace)
pipeline.add_step("구두점 정규화", normalize_punctuation)
pipeline.add_step("전화번호 통일", normalize_phone_numbers)
pipeline.add_step("날짜 형식 통일", normalize_dates)
pipeline.add_step("개인정보 마스킹", remove_personal_info)

sample_data = """
이름: 홍길동   연락처: 010.1234.5678   이메일: hong@example.com
등록일: 2024.3.5

이름: 김영희,  연락처: 010 5678 1234,  이메일: kim.yh@test.co.kr
등록일: 2024/03/15

주민번호: 900101-1234567
연락해주세요!!   빨리요~~~...

"""

cleaned = pipeline.process(sample_data)
print("=== 정제 결과 ===")
print(cleaned)
print("\n=== 처리 로그 ===")
print(pipeline.get_log())
print("\n=== 구조화된 데이터 추출 ===")
records = extract_structured_data(sample_data)
for r in records:
    print(r)
```

### 상세 예시

> 🔍 **예시 17.6.1: 복잡한 정규표현식 분석과 개선**
>
> **상황**: 동료가 작성한 이메일 검증 정규표현식이 있는데, 이해하기 어렵고 성능이 좋지 않습니다. 이를 분석하고 개선해야 합니다.
>
> **풀이/설명**:
> ```python
> import re
> 
> # 동료가 작성한 원본 패턴 (문제가 있음)
> original = r'^([a-zA-Z0-9]+[._-]?)*[a-zA-Z0-9]+@([a-zA-Z0-9]+[.-]?)*[a-zA-Z0-9]+\.[a-zA-Z]{2,}$'
> 
> # 분석:
> # 1. (…)* 중첩 수량자 → 재앙적 백트래킹 위험
> # 2. 패턴이 읽기 어려움
> # 3. 불필요하게 복잡한 구조
> 
> # 개선된 패턴
> improved = re.compile(r"""
>     ^
>     [a-zA-Z0-9]            # 시작: 영숫자
>     [a-zA-Z0-9._%+-]*      # 중간: 영숫자 및 허용 특수문자
>     @                       # @ 기호
>     [a-zA-Z0-9]            # 도메인 시작: 영숫자
>     [a-zA-Z0-9.-]*         # 도메인 중간
>     \.                      # 마지막 점
>     [a-zA-Z]{2,}           # TLD: 2자 이상 알파벳
>     $
> """, re.VERBOSE)
> 
> # 테스트
> test_emails = [
>     "user@example.com",         # 정상
>     "user.name+tag@test.co.kr", # 정상 (복잡한 형식)
>     "a@b.c",                    # TLD가 1자 → 실패해야 함
>     "@example.com",             # 로컬 부분 없음 → 실패해야 함
>     "user@.com",                # 도메인 시작이 점 → 실패해야 함
> ]
> 
> for email in test_emails:
>     result = improved.match(email)
>     status = "✅ 유효" if result else "❌ 무효"
>     print(f"{email:<30} {status}")
> # user@example.com               ✅ 유효
> # user.name+tag@test.co.kr        ✅ 유효
> # a@b.c                          ❌ 무효
> # @example.com                   ❌ 무효
> # user@.com                      ❌ 무효
> ```
>
> **핵심 포인트**: 
> - **중첩 수량자 제거**: `([a-zA-Z0-9]+[._-]?)*` 같은 중첩 구조를 `[a-zA-Z0-9._%+-]*`로 평탄화
> - **`re.VERBOSE`**: 주석으로 각 부분의 의도를 명시
> - **테스트 케이스**: 정상 입력과 비정상 입력을 모두 포함하여 검증

> 🔍 **예시 17.6.2: `re`와 `regex` 모듈을 적재적소에 활용하기**
>
> **상황**: 다국어 문서에서 한글 이름, 영문 이름, 숫자를 각각 분리하고, OCR 오류 가능성이 있는 회사명도 찾아야 합니다.
>
> **풀이/설명**:
> ```python
> import re
> import regex
> 
> document = """
> 담당자: 홍길동 (Hong Gildong), 연락처: 010-1234-5678
> 회사: Sarnsung Electronics  (OCR 오류 가능)
> 금액: 1,500,000원
> 참조: 김영희, Younghee Kirn (OCR 오류 가능)
> """
> 
> # 1. 한글 이름 추출 — re 모듈로 충분
> korean_names = re.findall(r'[가-힣]{2,4}', document)
> print(f"한글 이름: {korean_names}")
> # ['홍길동', '김영희']
> 
> # 2. 영문 이름 추출 — re 모듈로 충분
> english_names = re.findall(r'[A-Z][a-z]+(?:\s[A-Z][a-z]+)+', document)
> print(f"영문 이름: {english_names}")
> # ['Hong Gildong', 'Younghee Kirn']
> 
> # 3. 금액 추출 — re 모듈
> amounts = re.findall(r'[\d,]+원', document)
> print(f"금액: {amounts}")
> # ['1,500,000원']
> 
> # 4. OCR 오류 허용 회사명 검색 — regex 모듈의 퍼지 매칭 필요
> company_match = regex.search(
>     r'(?:Samsung Electronics){e<=2}',
>     document, regex.IGNORECASE
> )
> if company_match:
>     print(f"회사명 발견: '{company_match.group()}' (오류: {company_match.fuzzy_counts})")
> # 회사명 발견: 'Sarnsung Electronics' (오류: (0, 0, 1))
> 
> # 5. 순수 문자 vs 숫자 분리 — regex 모듈의 유니코드 카테고리
> letters_only = regex.findall(r'\p{L}{2,}', document)
> numbers_only = regex.findall(r'\p{N}+', document)
> print(f"문자 토큰: {letters_only}")
> print(f"숫자 토큰: {numbers_only}")
> ```
>
> **핵심 포인트**: 모든 작업에 `regex` 모듈을 쓸 필요는 없습니다. 단순한 패턴에는 표준 `re` 모듈을, 유니코드 카테고리나 퍼지 매칭이 필요할 때만 `regex` 모듈을 사용하는 것이 바람직합니다.

> ⚠️ **주의**: 종합 프로젝트를 진행할 때 가장 흔한 실수는 "한 번에 완벽한 패턴을 작성하려는 것"입니다. 반드시 작은 패턴부터 시작하여 점진적으로 확장하고, 각 단계마다 테스트하세요. 또한 정규표현식이 복잡해지면 `re.VERBOSE`를 사용하여 주석을 달아두세요. 미래의 자신(또는 동료)이 감사할 것입니다.

### Section 요약

- 정규표현식 프로젝트는 **요구사항 분석 → 패턴 설계 → 구현 → 테스트 → 최적화** 워크플로우를 따릅니다.
- 마크다운 파서 프로젝트는 앵커, 캡처 그룹, 비탐욕적 수량자, 전후방 탐색 등 다양한 기법을 종합합니다.
- 로그 분석 프로젝트는 명명 그룹, `re.VERBOSE`, `finditer()` 등을 실전에 적용합니다.
- 데이터 정제 파이프라인은 여러 정규표현식을 단계적으로 적용하는 설계 패턴을 보여줍니다.
- 실전에서는 `re`와 `regex`를 적재적소에 활용하고, 복잡한 패턴에는 반드시 주석을 남깁니다.

---

# Part C: 통합 및 정리 (Integration Block)

---

## Chapter 핵심 요약 (Chapter Summary)

이 Chapter에서는 파이썬 정규표현식의 마지막 퍼즐 조각들을 배웠습니다:

- **17.1 `regex` 모듈 소개**: `regex` 모듈은 표준 `re` 모듈의 드롭인 대체로, `import regex as re`만으로 기존 코드를 그대로 활용하면서 강력한 확장 기능에 접근할 수 있습니다.

- **17.2 유니코드 카테고리 매칭**: `\p{L}`, `\p{N}`, `\p{Hangul}`, `\p{Han}` 등의 문법으로 유니코드 속성 기반의 정밀한 문자 매칭이 가능합니다. `\w`보다 훨씬 세밀한 제어를 제공합니다.

- **17.3 소유적 수량자와 원자적 그룹의 실전 사용**: Chapter 15에서 배운 개념을 `regex` 모듈에서 `++`, `*+`, `(?>...)` 구문으로 실제 코드에 적용하여, 재앙적 백트래킹과 ReDoS를 방지할 수 있습니다.

- **17.4 퍼지 매칭과 기타 확장**: `(?:패턴){e<=N}` 문법으로 오탈자 허용 매칭이 가능하며, `partial=True`로 불완전한 입력의 실시간 검증을 수행할 수 있습니다.

- **17.5 정규표현식의 대안**: 문자열 메서드, `parse`, `pyparsing`, PEG 파서 등 상황에 따라 더 적합한 도구를 선택하는 안목을 갖추었습니다. 특히 중첩 구조(HTML/XML/JSON)에는 전용 파서가 필수입니다.

- **17.6 종합 프로젝트**: 마크다운 파서, 로그 분석기, 데이터 정제 파이프라인 등 실전 프로젝트를 통해 설계부터 최적화까지의 전체 워크플로우를 경험했습니다.

---

## 핵심 용어 정리 (Glossary)

| 용어 | 정의 | 관련 Section |
|------|------|-------------|
| `regex` 모듈 | PyPI에서 설치하는 `re` 모듈의 상위 호환 서드파티 라이브러리 | 17.1 |
| 드롭인 대체 | `import` 문 변경만으로 기존 코드가 그대로 동작하는 호환성 | 17.1 |
| 유니코드 일반 카테고리 | 문자를 L(문자), N(숫자), P(구두점), S(기호) 등으로 분류하는 체계 | 17.2 |
| 유니코드 스크립트 | 문자가 속한 문자 체계(Hangul, Han, Latin 등) | 17.2 |
| `\p{카테고리}` | 유니코드 속성 기반 매칭 문법 (`regex` 모듈 전용) | 17.2 |
| 소유적 수량자 구문 | `++`, `*+`, `?+`, `{n,m}+` — 백트래킹 없는 수량자 (`regex` 모듈) | 17.3 |
| 원자적 그룹 구문 | `(?>...)` — 백트래킹 없는 그룹 (`regex` 모듈) | 17.3 |
| 퍼지 매칭 | 오류를 허용하는 근사 매칭 (`{e<=N}`, `{s<=N}` 등) | 17.4 |
| 편집 거리 | 문자열 간 삽입/삭제/치환의 최소 횟수 (레벤슈타인 거리) | 17.4 |
| 부분 매칭 | 불완전한 입력의 매칭 가능성을 판단하는 기능 (`partial=True`) | 17.4 |
| `parse` 라이브러리 | format 문자열의 역방향 도구, 구조화된 텍스트에서 값 추출 | 17.5 |
| `pyparsing` | 파이썬 코드로 문법 기반 파서를 구성하는 라이브러리 | 17.5 |
| PEG 파서 | Parsing Expression Grammar 기반 파서, 재귀적 구조 처리 가능 | 17.5 |
| 정규표현식 파이프라인 | 여러 정제 단계를 순차적으로 적용하는 설계 패턴 | 17.6 |

---

## 개념 연결 맵 (Connection Map)

**이전 Chapter → 이번 Chapter:**

- Chapter 15에서 배운 소유적 수량자와 원자적 그룹의 **개념**을 → 이번 Chapter에서 `regex` 모듈을 통해 **실제 코드**로 사용했습니다.
- Chapter 16에서 경험한 재앙적 백트래킹과 ReDoS 문제를 → 소유적 수량자/원자적 그룹으로 **근본적으로 해결**하는 방법을 배웠습니다.
- Chapter 11에서 배운 유니코드/한국어 처리를 → `\p{Hangul}`, `\p{Han}` 등으로 **더 정밀하게** 수행할 수 있게 되었습니다.
- Chapter 12~14의 실전 응용 경험을 → 종합 프로젝트에서 **통합적으로** 활용했습니다.

**이번 Chapter가 열어주는 다음 단계:**

이 과정의 마지막 Chapter로서, 여러분은 이제 정규표현식의 모든 핵심 지식을 갖추었습니다. 앞으로의 방향:
- 실무 프로젝트에서 정규표현식을 자유롭게 설계하고 최적화할 수 있습니다.
- `regex` 모듈을 포함한 확장 도구를 적재적소에 활용할 수 있습니다.
- 정규표현식이 적합하지 않은 상황을 판단하고 대안을 선택할 수 있습니다.

---

## 자기 점검 질문 (Self-Check Questions)

**Q1.** `regex` 모듈에서 `\p{L}`은 어떤 문자를 매칭하는가?

<details>
<summary>정답</summary>

유니코드의 모든 '문자(Letter)' 카테고리에 해당하는 문자를 매칭합니다. 영문(A-Z, a-z), 한글(가-힣, ㄱ-ㅎ 등), 한자, 히라가나, 키릴 문자 등 모든 언어의 문자가 포함됩니다. 숫자(`0-9`)와 밑줄(`_`)은 포함되지 않습니다. 이는 `\w`(문자+숫자+밑줄)보다 좁은 범위입니다.
</details>

**Q2.** O/X: 소유적 수량자 `a++`는 `(?>a+)`와 동일한 동작을 한다.

<details>
<summary>정답</summary>

**O (맞음)**. 소유적 수량자는 원자적 그룹의 축약형입니다. `a++`는 `(?>a+)`와 정확히 동일하게 동작합니다. 다만, 원자적 그룹은 내부에 더 복잡한 패턴을 포함할 수 있어 더 유연합니다.
</details>

**Q3.** 퍼지 매칭에서 `(?:hello){s<=1,d<=1,e<=2}`의 의미는 무엇인가?

<details>
<summary>정답</summary>

"hello"와 매칭하되, 치환(substitution) 최대 1개, 삭제(deletion) 최대 1개, 전체 오류(error) 최대 2개를 허용한다는 의미입니다. 예를 들어 "helo"(삭제 1개)는 매칭되지만, "hlo"(삭제 2개)는 전체 오류 제한 내이지만 삭제 제한 초과로 매칭되지 않습니다.
</details>

**Q4.** O/X: `\p{Hangul}`은 `[가-힣]`과 정확히 동일한 범위의 문자를 매칭한다.

<details>
<summary>정답</summary>

**X (틀림)**. `\p{Hangul}`은 `[가-힣]`보다 더 넓은 범위를 매칭합니다. `[가-힣]`은 한글 완성형 음절만 매칭하지만, `\p{Hangul}`은 한글 자모(ㄱ-ㅎ, ㅏ-ㅣ), 호환 자모, 반각 자모 등도 포함합니다.
</details>

**Q5.** 다음 중 정규표현식보다 전용 파서가 적합한 작업은? (a) 전화번호 추출 (b) 이메일 검증 (c) HTML 태그 파싱 (d) 로그 분석

<details>
<summary>정답</summary>

**(c) HTML 태그 파싱**. HTML은 중첩 구조(nested structure)를 가지므로 정규표현식으로는 올바르게 파싱할 수 없습니다. BeautifulSoup, lxml 같은 전용 파서를 사용해야 합니다. 나머지 (a), (b), (d)는 정규표현식으로 충분히 처리 가능합니다.
</details>

**Q6.** `parse` 라이브러리에서 `{:d}`는 어떤 역할을 하는가?

<details>
<summary>정답</summary>

정수(decimal integer)를 매칭하고, 매칭된 문자열을 자동으로 파이썬 `int` 타입으로 변환합니다. 예를 들어 `parse("나이: {:d}세", "나이: 30세")`에서 `{:d}`는 "30"을 매칭하고 정수 `30`으로 반환합니다.
</details>

**Q7.** O/X: `regex` 모듈의 `partial=True` 옵션은 오탈자를 허용하는 퍼지 매칭 기능이다.

<details>
<summary>정답</summary>

**X (틀림)**. `partial=True`는 퍼지 매칭과는 다른 기능입니다. 이것은 **불완전한 입력**이 패턴에 매칭될 *가능성이 있는지*를 판단하는 부분 매칭 기능입니다. 예를 들어 패턴이 `\d{3}-\d{4}`일 때, 입력 "010-12"는 아직 완성되지 않았지만 완성될 가능성이 있으므로 `partial=True`에서 매칭 성공으로 판단됩니다. 퍼지 매칭은 `{e<=N}` 문법을 사용합니다.
</details>

---

# Part D: 문제 풀이 (Problem Set)

---

## D1. Practice (연습 문제) — 기본 개념 확인

> **Practice 17.1**: 다음 `regex` 모듈의 유니코드 카테고리 패턴 중, 문자열 `"Hello 세계 123 ©"` 에서 `['Hello', '세계']`를 결과로 반환하는 것은?
>
> (a) `regex.findall(r'\w+', text)`  
> (b) `regex.findall(r'\p{L}+', text)`  
> (c) `regex.findall(r'\p{N}+', text)`  
> (d) `regex.findall(r'\p{Lu}+', text)`
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: (b)
>
> **해설**: `\p{L}+`는 유니코드 문자(Letter)만 매칭합니다. 숫자 "123"과 기호 "©"는 제외됩니다. (a) `\w+`는 숫자 "123"도 포함하므로 `['Hello', '세계', '123']`을 반환합니다. (c) `\p{N}+`는 숫자만 매칭합니다. (d) `\p{Lu}+`는 대문자만 매칭하여 `['H']`만 반환합니다.
> </details>

> **Practice 17.2**: `regex.search(r'(?:test){e<=1}', 'tset')`의 결과에서 `m.fuzzy_counts`의 값은 무엇인가?
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `(0, 0, 2)` — 치환(substitution) 2개
>
> **해설**: "test" → "tset"는 `e`와 `s`의 위치가 바뀐 것으로, 'e'→'s'(인덱스 1)와 's'→'e'(인덱스 2)의 치환 2개입니다. 그런데 `{e<=1}`로 최대 1개 오류만 허용했으므로, 이 매칭은 실제로 **실패**합니다(`None` 반환). `{e<=2}`로 수정해야 매칭되며 그때 `fuzzy_counts`는 `(0, 0, 2)`가 됩니다.
> </details>

> **Practice 17.3**: 다음 중 소유적 수량자의 올바른 표현은?
>
> (a) `a*++`  
> (b) `a++`  
> (c) `a+*`  
> (d) `a**`
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: (b) `a++`
>
> **해설**: 소유적 수량자는 기본 수량자(`+`, `*`, `?`, `{n,m}`) 뒤에 `+`를 **한 개** 붙여 표현합니다. `a++`는 "a를 1회 이상, 소유적으로 매칭"입니다. `a*+`(0회 이상, 소유적)도 유효합니다. (a), (c), (d)는 잘못된 문법입니다.
> </details>

> **Practice 17.4**: `parse` 라이브러리에서 다음 코드의 출력은?
> ```python
> from parse import parse
> result = parse("가격: {:d}원", "가격: 15000원")
> print(type(result[0]), result[0])
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `<class 'int'> 15000`
>
> **해설**: `{:d}` 서식 지정자는 정수를 매칭하고, 결과를 자동으로 파이썬 `int` 타입으로 변환합니다. 따라서 `result[0]`은 문자열 `'15000'`이 아닌 정수 `15000`입니다.
> </details>

> **Practice 17.5**: 다음 중 `\p{Hangul}`은 매칭하지만 `[가-힣]`은 매칭하지 않는 문자는?
>
> (a) '가'  
> (b) '힣'  
> (c) 'ㄱ'  
> (d) 'A'
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: (c) 'ㄱ'
>
> **해설**: `[가-힣]`은 한글 완성형 음절 범위(U+AC00~U+D7A3)만 매칭합니다. 'ㄱ'은 한글 호환 자모(U+3131)로, 이 범위에 포함되지 않습니다. 반면 `\p{Hangul}`은 한글 스크립트에 속하는 모든 문자를 매칭하므로, 자모 'ㄱ'도 포함됩니다. (d) 'A'는 라틴 문자이므로 둘 다 매칭하지 않습니다.
> </details>

> **Practice 17.6**: 다음 코드에서 `result`의 값은?
> ```python
> import regex
> result = regex.search(r'".*+"', '"hello"')
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**: `None`
>
> **해설**: `.*+`는 소유적 수량자로, `.`에 해당하는 모든 문자를 가져간 후 절대 돌려주지 않습니다. 첫 번째 `"` 다음에 `.*+`가 `hello"`를 모두 소유적으로 매칭하므로, 패턴의 마지막 `"`를 매칭할 문자가 남아있지 않아 매칭이 실패합니다.
> </details>

---

## D2. Exercise (응용 문제) — 개념 적용 및 통합

> **Exercise 17.1**: 다국어 텍스트 분류기
>
> 다음 텍스트에서 `regex` 모듈의 유니코드 스크립트 매칭을 사용하여, 각 스크립트별 단어 목록과 단어 수를 출력하는 함수를 작성하세요.
>
> ```python
> text = "Python은 파이썬으로, 日本語では Python と呼びます. Питон — это Python."
> ```
>
> 출력 예시:
> ```
> Latin: ['Python', 'Python', 'Python'] (3개)
> Hangul: ['은', '파이썬으로'] (2개)
> Han: ['日本語'] (1개)
> Hiragana: ['では', 'と', 'びます'] (3개)
> Katakana: ['パイソン'] (1개) -- (원문에 없다면 해당 없음)
> Cyrillic: ['Питон', 'это'] (2개)
> ```
>
> **힌트**: 딕셔너리에 스크립트 이름과 패턴을 저장하고 순회하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: `regex` 모듈의 `\p{Script}+` 패턴을 여러 스크립트에 대해 적용합니다.
>
> **풀이 과정**:
> ```python
> import regex
> 
> def classify_by_script(text):
>     scripts = {
>         'Latin': r'\p{Latin}+',
>         'Hangul': r'\p{Hangul}+',
>         'Han': r'\p{Han}+',
>         'Hiragana': r'\p{Hiragana}+',
>         'Katakana': r'\p{Katakana}+',
>         'Cyrillic': r'\p{Cyrillic}+',
>     }
>     
>     result = {}
>     for script_name, pattern in scripts.items():
>         matches = regex.findall(pattern, text)
>         if matches:
>             result[script_name] = matches
>     
>     return result
> 
> text = "Python은 파이썬으로, 日本語では Python と呼びます. Питон — это Python."
> classification = classify_by_script(text)
> 
> for script, words in classification.items():
>     print(f"{script}: {words} ({len(words)}개)")
> ```
>
> **보충 설명**: `\p{Hiragana}`는 히라가나 문자만 매칭하고, `\p{Katakana}`는 가타카나만 매칭합니다. 일본어 텍스트에서 한자, 히라가나, 가타카나를 정확히 분리할 수 있는 것이 유니코드 스크립트 매칭의 강점입니다.
> </details>

> **Exercise 17.2**: 퍼지 매칭 기반 주소록 검색
>
> 다음 주소록에서, 사용자가 입력한 이름(오탈자 포함 가능)으로 연락처를 검색하는 함수를 작성하세요. 최대 2개의 오류를 허용합니다.
>
> ```python
> contacts = {
>     "홍길동": "010-1234-5678",
>     "김영희": "010-2345-6789",
>     "Alexander": "010-3456-7890",
>     "Christopher": "010-4567-8901",
> }
> 
> # 예: search_contact("홍길둥") → "홍길동: 010-1234-5678"
> # 예: search_contact("Alexnader") → "Alexander: 010-3456-7890"
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 주소록의 각 이름에 대해 퍼지 매칭을 시도하고, 가장 오류가 적은 결과를 반환합니다.
>
> **풀이 과정**:
> ```python
> import regex
> 
> def search_contact(query, contacts, max_errors=2):
>     """오탈자를 허용하는 연락처 검색"""
>     best_match = None
>     best_errors = max_errors + 1
>     
>     for name, phone in contacts.items():
>         # 각 이름에 대해 퍼지 매칭 시도
>         pattern = f'(?:{regex.escape(name)}){{e<={max_errors}}}'
>         m = regex.fullmatch(pattern, query, flags=regex.BESTMATCH)
>         
>         if m:
>             total_errors = sum(m.fuzzy_counts)
>             if total_errors < best_errors:
>                 best_errors = total_errors
>                 best_match = (name, phone, total_errors)
>     
>     if best_match:
>         name, phone, errors = best_match
>         return f"{name}: {phone} (오류 {errors}개)"
>     return "검색 결과 없음"
> 
> contacts = {
>     "홍길동": "010-1234-5678",
>     "김영희": "010-2345-6789",
>     "Alexander": "010-3456-7890",
>     "Christopher": "010-4567-8901",
> }
> 
> print(search_contact("홍길둥", contacts))    # 홍길동: 010-1234-5678 (오류 1개)
> print(search_contact("Alexnader", contacts)) # Alexander: 010-3456-7890 (오류 2개)
> print(search_contact("김영희", contacts))    # 김영희: 010-2345-6789 (오류 0개)
> print(search_contact("XXXXX", contacts))     # 검색 결과 없음
> ```
>
> **보충 설명**: `regex.escape()`로 이름에 포함될 수 있는 특수 문자를 안전하게 이스케이프합니다. `BESTMATCH` 플래그를 사용하면 가능한 한 오류가 적은 매칭을 우선합니다.
> </details>

> **Exercise 17.3**: 안전한 이메일 검증 패턴 작성
>
> `regex` 모듈의 소유적 수량자를 활용하여, ReDoS 공격에 안전한 이메일 검증 패턴을 작성하세요. 다음 테스트 케이스를 모두 통과해야 합니다:
>
> ```python
> valid = ["user@example.com", "a.b+c@test.co.kr", "user123@my-domain.org"]
> invalid = ["@example.com", "user@", "user@.com", "a" * 1000 + "@test.com"]
> ```
>
> **힌트**: 중첩 수량자를 피하고, 가능한 곳에 소유적 수량자를 적용하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 중첩 수량자를 사용하지 않고 평탄한 패턴을 설계하며, 소유적 수량자로 백트래킹을 방지합니다.
>
> **풀이 과정**:
> ```python
> import regex
> import time
> 
> # ReDoS-safe 이메일 패턴 (소유적 수량자 활용)
> email_pattern = regex.compile(r"""
>     ^
>     [a-zA-Z0-9]                    # 시작: 영숫자
>     [a-zA-Z0-9._%+-]*+            # 중간: 소유적으로 매칭
>     @                              # @ 기호
>     [a-zA-Z0-9]                    # 도메인 시작: 영숫자
>     [a-zA-Z0-9.-]*+               # 도메인 중간: 소유적으로 매칭
>     \.                             # 마지막 점
>     [a-zA-Z]{2,}+                  # TLD: 소유적으로 매칭
>     $
> """, regex.VERBOSE)
> 
> valid = ["user@example.com", "a.b+c@test.co.kr", "user123@my-domain.org"]
> invalid = ["@example.com", "user@", "user@.com", "a" * 1000 + "@test.com"]
> 
> print("=== 유효한 이메일 ===")
> for email in valid:
>     result = email_pattern.match(email)
>     print(f"  {email[:40]:<40} {'✅' if result else '❌'}")
> 
> print("\n=== 무효한 이메일 ===")
> for email in invalid:
>     start = time.time()
>     result = email_pattern.match(email)
>     elapsed = time.time() - start
>     display = email[:40] + ('...' if len(email) > 40 else '')
>     print(f"  {display:<40} {'❌' if not result else '✅'} ({elapsed:.6f}초)")
> # 마지막 케이스 (1000글자 + @test.com)도 즉시 처리됨
> ```
>
> **보충 설명**: 소유적 수량자 `*+`를 사용하면, 로컬 부분의 문자가 `@` 전까지 소유적으로 매칭됩니다. 매칭 실패 시 백트래킹 없이 즉시 실패하므로, 아무리 긴 입력이 들어와도 성능이 일정합니다.
> </details>

---

## D3. Problem (심화 문제) — 깊이 있는 사고와 창의적 문제 해결

> **Problem 17.1**: 설정 파일 파서 설계
>
> 다음과 같은 형식의 설정 파일을 파싱하는 도구를 만들어야 합니다. 정규표현식과 대안 도구 중 어떤 것을 사용할지 판단하고 구현하세요.
>
> ```ini
> [database]
> host = localhost
> port = 5432
> name = mydb
> 
> [server]
> host = 0.0.0.0
> port = 8080
> debug = true
> # 이것은 주석입니다
> allowed_origins = http://localhost:3000, http://example.com
> ```
>
> 요구사항:
> 1. 섹션(`[이름]`), 키-값 쌍, 주석을 올바르게 파싱할 것
> 2. 값의 타입을 자동 감지할 것 (정수, 불리언, 문자열, 쉼표로 구분된 목록)
> 3. 정규표현식, `parse`, 문자열 메서드 중 최소 두 가지 방법으로 구현하고 비교할 것
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: INI 파일은 줄 단위로 처리 가능한 단순한 구조이므로, 정규표현식이 매우 적합합니다. 하지만 문자열 메서드만으로도 충분히 구현할 수 있어, 두 방법을 비교하면 좋습니다.
>
> **풀이 전략**: 
> 1. 줄 단위로 읽으면서 각 줄의 유형(빈 줄, 주석, 섹션, 키-값)을 판단
> 2. 값의 타입 자동 변환 로직 구현
> 3. 딕셔너리 구조로 결과 반환
>
> **상세 풀이**:
> ```python
> import re
> 
> def parse_ini_regex(text):
>     """정규표현식 방식 INI 파서"""
>     config = {}
>     current_section = None
>     
>     section_pat = re.compile(r'^\[(\w+)\]$')
>     kv_pat = re.compile(r'^(\w+)\s*=\s*(.+)$')
>     comment_pat = re.compile(r'^\s*#')
>     
>     for line in text.strip().split('\n'):
>         line = line.strip()
>         
>         if not line or comment_pat.match(line):
>             continue
>         
>         section_match = section_pat.match(line)
>         if section_match:
>             current_section = section_match.group(1)
>             config[current_section] = {}
>             continue
>         
>         kv_match = kv_pat.match(line)
>         if kv_match and current_section:
>             key = kv_match.group(1)
>             value = auto_convert(kv_match.group(2).strip())
>             config[current_section][key] = value
>     
>     return config
> 
> def parse_ini_string(text):
>     """문자열 메서드 방식 INI 파서"""
>     config = {}
>     current_section = None
>     
>     for line in text.strip().split('\n'):
>         line = line.strip()
>         
>         if not line or line.startswith('#'):
>             continue
>         
>         if line.startswith('[') and line.endswith(']'):
>             current_section = line[1:-1]
>             config[current_section] = {}
>             continue
>         
>         if '=' in line and current_section:
>             key, _, value = line.partition('=')
>             key = key.strip()
>             value = auto_convert(value.strip())
>             config[current_section][key] = value
>     
>     return config
> 
> def auto_convert(value):
>     """값의 타입을 자동 감지하여 변환"""
>     if value.lower() in ('true', 'false'):
>         return value.lower() == 'true'
>     try:
>         return int(value)
>     except ValueError:
>         pass
>     if ',' in value:
>         return [v.strip() for v in value.split(',')]
>     return value
> ```
>
> **확장 생각**: 
> - 이 수준의 INI 파싱에서는 문자열 메서드와 정규표현식의 효율 차이가 미미합니다. 
> - 값에 `=` 문자가 포함될 수 있는 경우(`value = a=b`), 문자열 메서드의 `partition('=')`이 `re.match()`보다 안전합니다(`partition`은 첫 번째 `=`만 분리).
> - 파이썬 표준 라이브러리의 `configparser` 모듈이 이 작업의 전용 도구입니다. 실무에서는 이를 사용하는 것이 바람직합니다.
> </details>

> **Problem 17.2**: 복잡한 정규표현식 리팩토링
>
> 다음은 실제 코드에서 발견된 URL 검증 정규표현식입니다. 이 패턴을 분석하고, (1) 무엇을 매칭하는지 설명하고, (2) 문제점을 찾고, (3) 개선된 버전을 작성하세요.
>
> ```python
> url_pattern = r'^(https?|ftp)://([a-zA-Z0-9]+([a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{2,}(/[a-zA-Z0-9._~:/?#\[\]@!$&\'()*+,;=-]*)?$'
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 복잡한 패턴을 부분별로 분해하여 분석하고, 성능과 가독성 양면에서 개선합니다.
>
> **풀이 전략**:
>
> **1단계 — 분석:**
> ```
> ^                                      # 문자열 시작
> (https?|ftp)                           # 프로토콜: http, https, ftp
> ://                                    # 구분자
> ([a-zA-Z0-9]+([a-zA-Z0-9-]*           # 도메인 레이블 시작 + 중간
>   [a-zA-Z0-9])?\.)+                    # 도메인 레이블 끝 + 점 (1회 이상 반복)
> [a-zA-Z]{2,}                           # TLD
> (/[a-zA-Z0-9._~:/?#\[\]@!$&'()*+,;=-]*)? # 경로 (선택)
> $                                      # 문자열 끝
> ```
>
> **2단계 — 문제점:**
> - `([a-zA-Z0-9]+([a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+` — 중첩된 캡처 그룹과 수량자로 잠재적 백트래킹 위험
> - 불필요한 캡처 그룹 사용 (매칭만 필요하다면 비캡처로 충분)
> - 가독성이 매우 낮음
>
> **3단계 — 개선:**
> ```python
> import regex
> 
> improved_url = regex.compile(r"""
>     ^
>     (?:https?|ftp)://                      # 프로토콜
>     (?:                                     # 도메인
>         [a-zA-Z0-9]                         #   레이블 시작
>         (?:[a-zA-Z0-9-]*+[a-zA-Z0-9])?     #   레이블 중간+끝 (소유적)
>         \.                                  #   점
>     )++                                     # 1회 이상 반복 (소유적)
>     [a-zA-Z]{2,}+                           # TLD (소유적)
>     (?:/\S*)?                               # 경로 (선택, 간소화)
>     $
> """, regex.VERBOSE)
> ```
>
> **확장 생각**: URL 검증은 RFC 3986 표준이 매우 복잡하여, 완벽한 정규표현식을 만들기 어렵습니다. 실무에서는 `urllib.parse.urlparse()`나 전용 라이브러리(예: `validators`)를 사용하는 것이 더 안전합니다.
> </details>

---

# Part E: 마무리 (Closing Block)

---

## 다음 단계 안내 (What's Next)

축하합니다! 🎉 이 Chapter를 완료함으로써, 여러분은 **"파이썬을 활용한 정규표현식" 과정 전체**를 마쳤습니다.

여러분은 이제:
- 정규표현식의 기본 문법부터 고급 패턴까지 자유자재로 다룰 수 있습니다.
- 파이썬 `re` 모듈과 `regex` 모듈을 상황에 맞게 활용할 수 있습니다.
- 엔진의 내부 원리를 이해하고, 성능 최적화와 보안까지 고려할 수 있습니다.
- 정규표현식의 한계를 알고, 적절한 대안 도구를 선택할 수 있습니다.

**앞으로의 성장 방향:**
- **실전 프로젝트**: 이론을 넘어, 실제 업무에서 정규표현식을 적용하면서 경험을 쌓으세요.
- **다른 환경으로 확장**: JavaScript, Java, Go 등 다른 언어에서의 정규표현식도 큰 어려움 없이 다룰 수 있을 것입니다.
- **커뮤니티 참여**: Stack Overflow, GitHub 등에서 다른 사람의 정규표현식 관련 질문에 답하는 것은 자신의 지식을 공고히 하는 좋은 방법입니다.

---

## 추가 학습 자원 (Further Resources)

- **공식 문서**
  - [Python `re` 모듈 공식 문서](https://docs.python.org/3/library/re.html)
  - [`regex` 모듈 PyPI 페이지](https://pypi.org/project/regex/)
  - [`parse` 라이브러리 문서](https://pypi.org/project/parse/)
  - [`pyparsing` 라이브러리 문서](https://pyparsing-docs.readthedocs.io/)

- **서적**
  - Jeffrey Friedl, *Mastering Regular Expressions* (O'Reilly) — 정규표현식의 바이블로, 엔진 내부 원리와 최적화에 대한 깊은 통찰을 제공합니다.
  - Jan Goyvaerts & Steven Levithan, *Regular Expressions Cookbook* (O'Reilly) — 실전 레시피 모음으로, 다양한 문제별 패턴을 바로 활용할 수 있습니다.

- **온라인 도구**
  - [regex101.com](https://regex101.com) — Python 모드 지원, 실시간 디버거, 패턴 설명 자동 생성
  - [RegexOne](https://regexone.com) — 인터랙티브 정규표현식 학습 사이트
  - [Debuggex](https://www.debuggex.com) — 정규표현식의 시각적 다이어그램 생성

- **심화 주제**
  - [Regular Expression Matching Can Be Simple And Fast](https://swtch.com/~rsc/regexp/regexp1.html) — Russ Cox의 NFA/DFA 엔진 분석 글 (무료)
  - [ReDoS 취약점 데이터베이스](https://makenowjust-labs.github.io/recheck/) — 실제 ReDoS 사례 모음
