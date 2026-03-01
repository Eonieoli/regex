# Chapter 13: 텍스트 처리와 데이터 정제

> **난이도**: ⭐⭐⭐ | **예상 학습 시간**: 3.5시간 | **선수 지식**: Chapter 12 (데이터 검증 패턴)

---

# Part A: 도입부

## 학습 목표 (Learning Objectives)

이 Chapter를 완료하면, 당신은 다음을 할 수 있습니다:

- [ ] 비정형 텍스트에서 구조화된 데이터를 추출하는 정규표현식 파이프라인을 설계할 수 있다
- [ ] 공백 정규화, 특수문자 제거, 포맷 통일 등 데이터 클리닝 작업을 정규표현식으로 수행할 수 있다
- [ ] CSV, TSV, 키-값 쌍 등 구조화된 텍스트를 정규표현식으로 파싱할 수 있다
- [ ] 날짜 형식 변환, 네이밍 컨벤션 변환 등 포맷 변환 패턴을 구현할 수 있다
- [ ] 여러 패턴을 조합하여 복합 텍스트 처리 워크플로우를 구성할 수 있다

---

## 핵심 질문 (Essential Questions)

1. **현실 세계의 데이터는 왜 이렇게 "지저분"하고, 정규표현식은 이 혼란을 어떻게 질서로 바꿀 수 있을까?**
2. **하나의 패턴으로 해결할 수 없는 복잡한 텍스트 처리를, 여러 패턴을 연결하는 "파이프라인"으로 어떻게 우아하게 풀 수 있을까?**
3. **"추출"과 "변환"은 어떻게 다르며, 각각 어떤 정규표현식 기법이 적합할까?**

---

## 개념 지도 (Concept Map)

```
[선수: Ch1~Ch11의 모든 정규표현식 문법]
    │
    ├── [선수: Ch7 - re.sub(), re.split(), 역참조 치환, 함수 치환]
    ├── [선수: Ch8 - 명명 그룹, 비캡처 그룹, groupdict()]
    ├── [선수: Ch9 - 전후방 탐색]
    ├── [선수: Ch11 - 유니코드/한국어 처리]
    └── [선수: Ch12 - 검증 패턴 설계, 정밀도/재현율 트레이드오프]
            │
            ▼
    ┌─────────────────────────────────────────┐
    │     Chapter 13: 텍스트 처리와 데이터 정제     │
    ├─────────────────────────────────────────┤
    │                                         │
    │  [신규] 텍스트 정규화(Normalization)       │
    │    └─ 공백 정리, 특수문자 처리, 대소문자 통일  │
    │                                         │
    │  [신규] 데이터 추출 패턴                    │
    │    └─ 비정형 텍스트 → 구조화된 정보          │
    │                                         │
    │  [신규] 포맷 변환 패턴                     │
    │    └─ 날짜/전화번호/네이밍 컨벤션 변환        │
    │                                         │
    │  [신규] 구조화된 텍스트 파싱                 │
    │    └─ CSV, 키-값 쌍, 설정 파일              │
    │                                         │
    │  [신규] 다단계 처리 파이프라인               │
    │    └─ 여러 re.sub()를 순차 연결             │
    │                                         │
    └─────────────────────────────────────────┘
            │
            ▼
    [다음: Ch14 - 로그 분석과 파일 자동화]
```

이 Chapter에서는 지금까지 배운 정규표현식의 모든 도구를 실전 데이터 처리에 적용합니다. Chapter 12에서 "데이터가 올바른지 *검증*"하는 법을 배웠다면, 이 Chapter에서는 **지저분한 데이터를 깨끗하게 *변환*하고, 비정형 텍스트에서 필요한 정보를 *추출*하는** 방법을 다룹니다.

---

# Part B: 본문 (Main Content)

---

## 13.1 텍스트 정규화

> 💡 **한 줄 요약**: 들쭉날쭉한 텍스트 데이터를 일관된 형태로 통일하는 것이 정규화이며, 이는 모든 데이터 처리의 첫 번째 단계입니다.

### 직관적 이해

도서관에서 책을 정리하는 사서를 떠올려 봅시다. 기증받은 책들이 도착하면 먼지를 털고, 찢어진 표지를 수선하고, 도서 번호를 붙인 다음에야 서가에 꽂을 수 있습니다. 데이터도 마찬가지입니다. 웹에서 스크래핑한 텍스트, 사용자가 입력한 데이터, 오래된 시스템에서 내보낸 파일 — 이런 "현실의 데이터"는 대부분 지저분합니다.

```
"   홍 길동   "          ← 앞뒤 공백, 이름 사이 불필요한 공백
"서울특별시  강남구  역삼동" ← 연속 공백
"가격: ₩1,000,000원"    ← 통화 기호와 단위 혼재
"email: Hong@Test.COM"  ← 대소문자 불일치
"전화: 010 1234 5678"   ← 구분자 불일치 (공백 vs 하이픈)
```

이런 데이터를 분석하거나 비교하려면, 먼저 **일관된 형태로 통일**해야 합니다. 이 과정을 **텍스트 정규화(text normalization)**라고 합니다.

> 📝 **텍스트 정규화(Text Normalization)**: 형태가 다양한 텍스트 데이터를 하나의 일관된 표준 형식으로 변환하는 과정. 공백 정리, 대소문자 통일, 특수문자 제거/통일 등이 포함됩니다.

왜 정규표현식이 이 작업에 적합할까요? 파이썬의 `str.strip()`이나 `str.replace()`로도 간단한 정규화는 가능하지만, **패턴 기반의 복잡한 정규화**는 정규표현식이 압도적으로 편리합니다. 예를 들어 "연속된 공백을 하나로 줄이기"는 `str.replace()`로는 반복 호출이 필요하지만, 정규표현식으로는 한 줄입니다.

### 핵심 개념 설명

텍스트 정규화는 크게 세 가지 영역으로 나뉩니다.

**① 공백 정규화**

가장 흔한 정규화 작업입니다. 데이터에서 발생하는 공백 문제는 다양합니다.

```python
import re

text = "  Hello   World  \t  Python  \n  "

# 1) 연속 공백 → 단일 공백
result = re.sub(r'\s+', ' ', text)
print(repr(result))  # ' Hello World Python '

# 2) 앞뒤 공백까지 제거하려면 strip()과 결합
result = re.sub(r'\s+', ' ', text).strip()
print(repr(result))  # 'Hello World Python'
```

> 🔗 **연결**: `re.sub(pattern, repl, string)`은 Chapter 7에서 배운 치환 함수입니다. `\s+`는 Chapter 3의 사전 정의 문자 클래스(`\s`)와 Chapter 4의 수량자(`+`)를 조합한 것입니다.

여기서 `\s+`는 "하나 이상의 연속된 공백 문자(스페이스, 탭, 줄바꿈 등)"를 의미합니다. 이것을 단일 공백 `' '`로 치환하면, 어떤 종류의 공백이 몇 개 연속되든 깔끔하게 정리됩니다.

**줄바꿈은 보존하면서 같은 줄의 연속 공백만 정리**하고 싶다면:

```python
text = "Hello   World\n  Python   Regex  "
result = re.sub(r'[^\S\n]+', ' ', text)  # \n을 제외한 공백만 대상
print(repr(result))  # 'Hello World\n Python Regex '
```

`[^\S\n]`은 "공백 문자이지만(`\S`의 부정) 줄바꿈은 아닌" 문자를 의미합니다. 이 패턴은 약간 복잡해 보이지만, "줄바꿈을 보존하는 공백 정리"라는 실무에서 매우 흔한 요구사항을 해결합니다.

> 🔗 **연결**: `[^\S\n]`에서 `[^...]`는 Chapter 3의 부정 문자 클래스입니다. `\S`(공백이 아닌 문자)를 부정하면 결국 공백 문자가 되고, 거기서 `\n`을 추가로 제외하는 이중 부정 기법입니다.

**② 특수문자 처리**

데이터에 포함된 불필요한 특수문자를 제거하거나 통일하는 작업입니다.

```python
# HTML 태그 제거 (간단한 버전)
html_text = "<p>Hello <b>World</b></p>"
clean = re.sub(r'<[^>]+>', '', html_text)
print(clean)  # 'Hello World'

# 문장부호 외 특수문자 제거
text = "Hello! @World# $Python% ^Regex&"
clean = re.sub(r'[^\w\s!?.,]', '', text)
print(clean)  # 'Hello! World Python Regex'

# 연속된 구두점 정리
text = "정말요???!!! 대단해요...."
clean = re.sub(r'([!?.])\1+', r'\1', text)
print(clean)  # '정말요?! 대단해요.'
```

마지막 예시에서 `([!?.])\1+`는 느낌표, 물음표, 마침표 중 하나를 캡처한 뒤, 같은 문자가 하나 이상 반복되는 것을 찾습니다. 이것을 `\1`(캡처된 한 글자)로 치환하면 중복이 제거됩니다.

> 🔗 **연결**: `\1`은 패턴 내 역참조(Chapter 8)와 치환 문자열의 역참조(Chapter 7) 모두에서 사용됩니다. 여기서는 패턴과 치환 양쪽에서 활용하고 있습니다.

**③ 대소문자 및 전각/반각 통일**

```python
# 이메일 도메인 부분만 소문자로
def lower_domain(m):
    return m.group(1) + m.group(2).lower()

email = "User@EXAMPLE.COM"
normalized = re.sub(r'(.+@)(.+)', lower_domain, email)
print(normalized)  # 'User@example.com'
```

> 🔗 **연결**: `re.sub()`에 함수를 전달하는 동적 치환은 Chapter 7에서 배운 기법입니다.

### 상세 예시

> 🔍 **예시 13.1.1: 게시판 댓글 정규화**
>
> **상황**: 온라인 게시판의 댓글 데이터를 분석하려 하는데, 사용자들이 입력한 텍스트가 매우 지저분합니다.
> ```python
> comment = "  ㅋㅋㅋㅋㅋ   정말    웃기다 ㅎㅎㅎㅎㅎ!!!!!!   "
> ```
>
> **풀이/설명**:
> ```python
> import re
>
> comment = "  ㅋㅋㅋㅋㅋ   정말    웃기다 ㅎㅎㅎㅎㅎ!!!!!!   "
>
> # Step 1: 앞뒤 공백 제거 + 연속 공백 → 단일 공백
> step1 = re.sub(r'\s+', ' ', comment).strip()
> # 'ㅋㅋㅋㅋㅋ 정말 웃기다 ㅎㅎㅎㅎㅎ!!!!!!'
>
> # Step 2: 연속 반복 글자를 최대 2개로 제한
> step2 = re.sub(r'(.)\1{2,}', r'\1\1', step1)
> # 'ㅋㅋ 정말 웃기다 ㅎㅎ!!'
>
> print(step2)
> ```
> **핵심 포인트**: `(.)\1{2,}`는 "임의의 한 문자가 3번 이상 연속 반복"되는 것을 찾고, `\1\1`로 치환하여 최대 2번까지만 허용합니다. 이것은 자연어 처리(NLP)에서 매우 흔한 전처리 기법입니다.

> 🔍 **예시 13.1.2: 주소 데이터 정규화**
>
> **상황**: 여러 출처에서 수집한 주소 데이터의 형식이 제각각입니다.
> ```python
> addresses = [
>     "서울특별시  강남구   역삼동  123-45",
>     "서울 강남구 역삼동 123-45",
>     "서울특별시 강남구  역삼동123-45",
> ]
> ```
>
> **풀이/설명**:
> ```python
> import re
>
> def normalize_address(addr):
>     # 연속 공백 → 단일 공백
>     addr = re.sub(r'\s+', ' ', addr).strip()
>     # 동/리 이름과 번지 사이에 공백 확보
>     addr = re.sub(r'([동리가로])(\d)', r'\1 \2', addr)
>     return addr
>
> for addr in addresses:
>     print(normalize_address(addr))
> # 서울특별시 강남구 역삼동 123-45
> # 서울 강남구 역삼동 123-45
> # 서울특별시 강남구 역삼동 123-45
> ```
> **핵심 포인트**: `([동리가로])(\d)` 패턴은 "동/리/가/로"로 끝나는 지명 바로 뒤에 숫자가 붙어있는 경우를 찾아 공백을 삽입합니다. 캡처 그룹을 사용해 원래 내용을 보존하면서 형식만 통일합니다.

### 흔한 실수와 주의사항

> ⚠️ **주의**: `re.sub(r'\s+', ' ', text)`는 줄바꿈(`\n`)도 공백으로 바꿉니다. 줄 구조를 보존해야 하는 경우에는 `\s` 대신 `[ \t]+`(스페이스와 탭만)를 사용하세요.

> ⚠️ **주의**: HTML 태그를 `<[^>]+>`로 제거하는 것은 간단한 경우에만 적합합니다. 속성값에 `>`가 포함된 복잡한 HTML에서는 제대로 동작하지 않을 수 있습니다. 실무에서 HTML 처리가 필요하면 `BeautifulSoup` 같은 전용 라이브러리를 사용하는 것이 안전합니다.

### Section 요약

- **텍스트 정규화**는 비일관적인 텍스트를 표준 형식으로 통일하는 과정이다
- `re.sub(r'\s+', ' ', text).strip()`은 가장 기본적이고 빈번한 공백 정규화 패턴이다
- `(.)\1{2,}` → `\1\1` 패턴으로 반복 글자를 제한할 수 있다
- 특수문자 처리, 대소문자 통일, 구두점 정리 등은 모두 `re.sub()`으로 구현한다
- 정규화는 "단계별로" 수행하는 것이 디버깅과 유지보수에 유리하다

---

## 13.2 데이터 추출 패턴

> 💡 **한 줄 요약**: 비정형 텍스트에서 이름, 날짜, 금액, 수량 등 의미 있는 정보를 구조화된 형태로 뽑아내는 것이 데이터 추출입니다.

### 직관적 이해

신문 기사 더미에서 "모든 회사의 분기 매출 수치"만 뽑아야 한다고 상상해 보세요. 사람이라면 기사를 읽으며 "삼성전자는 3분기 매출 76조원을 기록했다"와 같은 문장에서 눈으로 숫자를 찾아 적어 내겠지만, 수천 건의 기사를 처리해야 한다면 불가능에 가깝습니다.

정규표현식은 이 작업을 자동화합니다. 핵심은 추출하려는 정보의 **"형태적 특징"을 패턴으로 표현**하는 것입니다. 금액은 숫자 뒤에 "원"이나 "달러"가 오고, 날짜는 특정 형식(2024-01-15, 2024/1/15, 2024년 1월 15일)을 따르며, 이메일은 `@`를 포함합니다.

Chapter 12에서는 이런 패턴으로 데이터가 "올바른지 *검증*"했습니다. 이 Section에서는 같은 패턴을 사용해 텍스트 속에서 데이터를 **"*추출*"**하는 데 초점을 맞춥니다.

> 🔗 **연결**: Chapter 12에서 설계한 검증 패턴(이메일, 전화번호, 날짜 등)이 여기서 추출 패턴의 기반이 됩니다. 검증에서는 `re.fullmatch()`로 전체 일치를 확인했지만, 추출에서는 `re.findall()`이나 `re.finditer()`로 텍스트 속의 모든 매칭을 찾습니다.

### 핵심 개념 설명

**검증 vs 추출의 핵심 차이**

| 구분 | 검증 (Validation) | 추출 (Extraction) |
|------|-------------------|-------------------|
| 목적 | "이 문자열이 패턴에 맞는가?" | "이 텍스트 안에 패턴에 맞는 부분이 어디 있는가?" |
| 주요 함수 | `re.fullmatch()`, `re.match()` | `re.findall()`, `re.finditer()` |
| 입력 | 검사 대상 문자열 하나 | 긴 텍스트 (문서, 로그, 웹페이지 등) |
| 결과 | 성공/실패 (True/False) | 매칭된 값들의 리스트/이터레이터 |

**금액 추출 패턴**

```python
import re

text = """
삼성전자는 3분기 매출 76조 5,000억원을 기록했다.
LG전자의 매출은 약 21조 7,000억원이었다.
영업이익은 1조 2,000억원으로 전년 대비 15% 증가했다.
제품 가격은 1,299,000원이다.
"""

# 금액 추출: 숫자(조/억/만 포함)와 "원" 결합
amounts = re.findall(r'\d[\d,]*(?:\s*[조억만]\s*[\d,]*)*\s*원', text)
print(amounts)
# ['76조 5,000억원', '21조 7,000억원', '1조 2,000억원', '1,299,000원']
```

이 패턴을 분해해 봅시다:
- `\d[\d,]*` — 숫자로 시작, 이후 숫자나 쉼표 반복 (예: `76`, `5,000`, `1,299,000`)
- `(?:\s*[조억만]\s*[\d,]*)*` — 선택적으로 "조/억/만" 단위와 추가 숫자 (예: `조 5,000억`)
- `\s*원` — 마지막에 "원"

> 🔗 **연결**: `(?:...)`는 Chapter 8에서 배운 비캡처 그룹입니다. `findall()`에서 비캡처 그룹을 사용하면 전체 매칭이 반환됩니다.

**명명 그룹을 활용한 구조화 추출**

단순히 매칭된 문자열을 가져오는 것을 넘어, **각 구성 요소를 이름 붙여 추출**하면 후속 처리가 훨씬 편리합니다.

```python
text = """
주문일: 2024-03-15, 상품: 노트북, 금액: 1,299,000원, 수량: 2개
주문일: 2024-03-16, 상품: 마우스, 금액: 35,000원, 수량: 5개
"""

pattern = re.compile(
    r'주문일:\s*(?P<date>\d{4}-\d{2}-\d{2}),\s*'
    r'상품:\s*(?P<product>[^,]+),\s*'
    r'금액:\s*(?P<price>[\d,]+)원,\s*'
    r'수량:\s*(?P<qty>\d+)개'
)

for m in pattern.finditer(text):
    info = m.groupdict()
    print(info)
# {'date': '2024-03-15', 'product': '노트북', 'price': '1,299,000', 'qty': '2'}
# {'date': '2024-03-16', 'product': '마우스', 'price': '35,000', 'qty': '5'}
```

> 🔗 **연결**: `(?P<name>...)`은 Chapter 8의 명명 그룹, `groupdict()`은 명명 그룹의 결과를 딕셔너리로 반환하는 메서드입니다.

이렇게 추출된 딕셔너리는 바로 판다스 DataFrame으로 변환하거나, JSON으로 저장하거나, 데이터베이스에 삽입할 수 있습니다. 정규표현식이 **비정형 데이터와 구조화된 데이터 사이의 다리** 역할을 하는 것입니다.

### 상세 예시

> 🔍 **예시 13.2.1: 뉴스 기사에서 날짜 추출**
>
> **상황**: 뉴스 기사 텍스트에서 다양한 형식의 날짜를 모두 추출해야 합니다.
> ```python
> article = """
> 2024년 3월 15일 - 정부는 2024.03.20까지 제출하라고 발표했다.
> 지난 2023-12-25에 시행된 정책은 2024/1/1부터 효력이 발생한다.
> """
> ```
>
> **풀이/설명**:
> ```python
> import re
>
> article = """
> 2024년 3월 15일 - 정부는 2024.03.20까지 제출하라고 발표했다.
> 지난 2023-12-25에 시행된 정책은 2024/1/1부터 효력이 발생한다.
> """
>
> # 다양한 날짜 형식을 OR로 결합
> date_pattern = re.compile(
>     r'(?P<y1>\d{4})년\s*(?P<m1>\d{1,2})월\s*(?P<d1>\d{1,2})일'  # 한국어 형식
>     r'|(?P<y2>\d{4})[./-](?P<m2>\d{1,2})[./-](?P<d2>\d{1,2})'   # 구분자 형식
> )
>
> for m in date_pattern.finditer(article):
>     if m.group('y1'):  # 한국어 형식 매칭
>         print(f"{m.group('y1')}-{m.group('m1').zfill(2)}-{m.group('d1').zfill(2)}")
>     else:              # 구분자 형식 매칭
>         print(f"{m.group('y2')}-{m.group('m2').zfill(2)}-{m.group('d2').zfill(2)}")
>
> # 출력:
> # 2024-03-15
> # 2024-03-20
> # 2023-12-25
> # 2024-01-01
> ```
> **핵심 포인트**: 선택 연산자(`|`)로 여러 형식을 하나의 패턴으로 통합했습니다. 명명 그룹을 사용하면 어떤 형식이 매칭됐는지에 따라 분기 처리가 가능합니다.

> 🔍 **예시 13.2.2: 이력서에서 정보 추출**
>
> **상황**: 텍스트 형식의 이력서에서 이름, 이메일, 전화번호를 자동으로 추출하는 프로그램을 작성합니다.
> ```python
> resume = """
> 이름: 김철수
> 이메일: chulsoo.kim@example.com
> 연락처: 010-1234-5678
> 경력: 5년 (2019~2024)
> """
> ```
>
> **풀이/설명**:
> ```python
> import re
>
> resume = """
> 이름: 김철수
> 이메일: chulsoo.kim@example.com
> 연락처: 010-1234-5678
> 경력: 5년 (2019~2024)
> """
>
> patterns = {
>     'name': r'이름\s*:\s*(?P<name>[가-힣]{2,5})',
>     'email': r'이메일\s*:\s*(?P<email>[\w.+-]+@[\w-]+\.[\w.]+)',
>     'phone': r'연락처\s*:\s*(?P<phone>\d{2,3}-\d{3,4}-\d{4})',
>     'experience': r'경력\s*:\s*(?P<years>\d+)년',
> }
>
> info = {}
> for key, pat in patterns.items():
>     m = re.search(pat, resume)
>     if m:
>         info[key] = m.group(m.lastgroup)
>
> print(info)
> # {'name': '김철수', 'email': 'chulsoo.kim@example.com',
> #  'phone': '010-1234-5678', 'experience': '5'}
> ```
> **핵심 포인트**: 여러 패턴을 딕셔너리에 저장하고 반복문으로 순회하면, 추출 로직이 깔끔하고 확장 가능해집니다. 새로운 필드를 추출하려면 딕셔너리에 패턴을 추가하기만 하면 됩니다.

### 흔한 실수와 주의사항

> ⚠️ **주의**: `findall()`에 캡처 그룹이 있으면 전체 매칭이 아닌 **캡처 그룹의 내용만** 반환됩니다. 전체 매칭과 캡처 그룹을 모두 활용하려면 `finditer()`를 사용하세요. 캡처가 필요 없는 그룹에는 `(?:...)`를 사용하면 `findall()`의 동작이 달라지지 않습니다.

> ⚠️ **주의**: 추출 패턴은 너무 엄격하면 유효한 데이터를 놓치고, 너무 느슨하면 잘못된 데이터를 가져옵니다. Chapter 12에서 배운 **정밀도와 재현율의 트레이드오프**를 항상 고려하세요. 추출 후 결과를 샘플링하여 검증하는 습관이 중요합니다.

### Section 요약

- 데이터 추출은 비정형 텍스트에서 구조화된 정보를 뽑아내는 작업이다
- **검증**은 `fullmatch()`/`match()`를, **추출**은 `findall()`/`finditer()`를 사용한다
- 명명 그룹(`(?P<name>...)`)과 `groupdict()`를 활용하면 추출 결과를 바로 딕셔너리로 변환할 수 있다
- 여러 형식을 선택 연산자(`|`)로 하나의 패턴에 통합하면 다양한 포맷을 한번에 처리할 수 있다
- 추출 패턴을 딕셔너리로 관리하면 유지보수와 확장이 쉬워진다

---

## 13.3 포맷 변환

> 💡 **한 줄 요약**: 데이터의 *내용*은 동일하게 유지하면서 *형식*만 다른 표준으로 바꾸는 것이 포맷 변환이며, 캡처 그룹 역참조가 핵심 도구입니다.

### 직관적 이해

같은 날짜라도 한국에서는 "2024년 3월 15일", 미국에서는 "03/15/2024", ISO 표준에서는 "2024-03-15"로 씁니다. 같은 정보이지만 형식이 다릅니다. 데이터를 다른 시스템으로 넘기거나, 여러 출처의 데이터를 통합할 때는 이 형식을 맞춰야 합니다.

정규표현식의 **"캡처 + 역참조 치환"**은 이 작업에 완벽하게 들어맞습니다. 원본에서 의미 있는 부분(년, 월, 일)을 캡처하고, 치환 문자열에서 원하는 순서와 형식으로 **재배치**하면 됩니다.

> 🔗 **연결**: Chapter 7에서 배운 `re.sub()`의 역참조 치환(`\1`, `\g<name>`)이 이 Section의 핵심 도구입니다.

### 핵심 개념 설명

**① 날짜 형식 변환**

```python
import re

# "MM/DD/YYYY" → "YYYY-MM-DD" (미국식 → ISO)
text = "Date: 03/15/2024, Deadline: 12/31/2024"
result = re.sub(
    r'(\d{2})/(\d{2})/(\d{4})',
    r'\3-\1-\2',
    text
)
print(result)  # 'Date: 2024-03-15, Deadline: 2024-12-31'
```

`(\d{2})/(\d{2})/(\d{4})`는 세 그룹을 캡처합니다: 월(그룹1), 일(그룹2), 년(그룹3). 치환 문자열 `\3-\1-\2`는 이 그룹들을 "년-월-일" 순서로 재배치합니다.

명명 그룹을 사용하면 더 읽기 쉽습니다:

```python
result = re.sub(
    r'(?P<month>\d{2})/(?P<day>\d{2})/(?P<year>\d{4})',
    r'\g<year>-\g<month>-\g<day>',
    text
)
```

**② 전화번호 형식 통일**

```python
# 다양한 구분자 → 하이픈 형식으로 통일
phones = [
    "010.1234.5678",
    "010 1234 5678",
    "01012345678",
    "010-1234-5678",
]

for phone in phones:
    unified = re.sub(
        r'(\d{3})[.\s-]?(\d{4})[.\s-]?(\d{4})',
        r'\1-\2-\3',
        phone
    )
    print(unified)
# 010-1234-5678  (4번 모두 동일한 결과)
```

`[.\s-]?`는 구분자(마침표, 공백, 하이픈)가 있을 수도 없을 수도 있는 상황을 처리합니다. 어떤 형식이든 세 부분을 캡처한 후 하이픈으로 재결합합니다.

**③ 네이밍 컨벤션 변환**

프로그래밍에서 자주 필요한 변환입니다.

```python
# camelCase → snake_case
def camel_to_snake(name):
    # 대문자 앞에 언더스코어 삽입
    result = re.sub(r'(?<=[a-z0-9])(?=[A-Z])', '_', name)
    return result.lower()

print(camel_to_snake("getUserName"))      # 'get_user_name'
print(camel_to_snake("parseHTMLContent")) # 'parse_h_t_m_l_content'  ← 연속 대문자 문제!
```

연속 대문자(예: `HTML`)를 올바르게 처리하려면 패턴을 개선해야 합니다:

```python
def camel_to_snake(name):
    # 1단계: 연속 대문자 뒤에 소문자가 오는 경우 (HTMLContent → HTML_Content)
    result = re.sub(r'([A-Z]+)([A-Z][a-z])', r'\1_\2', name)
    # 2단계: 소문자/숫자 뒤에 대문자가 오는 경우 (get_User → get_user)
    result = re.sub(r'([a-z0-9])([A-Z])', r'\1_\2', result)
    return result.lower()

print(camel_to_snake("getUserName"))      # 'get_user_name'
print(camel_to_snake("parseHTMLContent")) # 'parse_html_content'  ✓
print(camel_to_snake("getURLParser"))     # 'get_url_parser'      ✓
```

> 🔗 **연결**: `(?<=[a-z0-9])(?=[A-Z])`은 Chapter 9에서 배운 후방 탐색과 전방 탐색의 조합입니다. 소문자/숫자 뒤에 대문자가 오는 **"위치"**를 찾되, 문자 자체는 소비하지 않으므로 언더스코어만 삽입할 수 있습니다.

반대 방향의 변환도 가능합니다:

```python
# snake_case → camelCase
def snake_to_camel(name):
    return re.sub(r'_([a-z])', lambda m: m.group(1).upper(), name)

print(snake_to_camel("get_user_name"))  # 'getUserName'
print(snake_to_camel("parse_html"))     # 'parseHtml'
```

여기서 `lambda m: m.group(1).upper()`는 `re.sub()`에 함수를 전달하여, 매칭된 소문자를 대문자로 변환합니다.

### 상세 예시

> 🔍 **예시 13.3.1: 한국어 날짜 → ISO 형식 변환**
>
> **상황**: 공문서에서 추출한 날짜가 "2024년 3월 5일" 형식이며, 이를 데이터베이스 저장용 ISO 형식("2024-03-05")으로 변환해야 합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "접수일: 2024년 3월 5일, 마감일: 2024년 12월 31일"
>
> def convert_kr_date(m):
>     year = m.group('year')
>     month = m.group('month').zfill(2)   # '3' → '03'
>     day = m.group('day').zfill(2)       # '5' → '05'
>     return f"{year}-{month}-{day}"
>
> result = re.sub(
>     r'(?P<year>\d{4})년\s*(?P<month>\d{1,2})월\s*(?P<day>\d{1,2})일',
>     convert_kr_date,
>     text
> )
> print(result)  # '접수일: 2024-03-05, 마감일: 2024-12-31'
> ```
> **핵심 포인트**: 단순 역참조(`\g<year>-\g<month>-\g<day>`)로는 한 자리 월/일을 두 자리로 패딩(`zfill(2)`)할 수 없습니다. 이런 경우 **함수 치환**을 사용하면 변환 로직을 자유롭게 적용할 수 있습니다.

> 🔍 **예시 13.3.2: 금액 표기 변환**
>
> **상황**: 한국 원화 표기("1,234,567원")를 영문 회계 표기("₩1,234,567") 로 변환합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> text = "매출 1,234,567원, 비용 987,654원, 순이익 246,913원"
>
> result = re.sub(
>     r'([\d,]+)원',
>     r'₩\1',
>     text
> )
> print(result)  # '매출 ₩1,234,567, 비용 ₩987,654, 순이익 ₩246,913'
> ```
> **핵심 포인트**: `([\d,]+)원`으로 숫자 부분을 캡처한 뒤, 치환 문자열 `₩\1`에서 통화 기호를 앞에 붙이고 "원" 접미사를 제거합니다. 간단하지만 실무에서 자주 필요한 패턴입니다.

### 깊이 있는 이해

> 🔬 **Deep Dive**: 포맷 변환 함수의 체계적 설계
>
> *이 부분은 심화 내용입니다. 처음 학습 시 건너뛰어도 됩니다.*
>
> 실무에서는 다양한 입력 형식을 하나의 출력 형식으로 통일해야 하는 경우가 많습니다. 이때 변환 함수를 체계적으로 설계하면 재사용성이 높아집니다:
>
> ```python
> import re
> from typing import Callable
>
> def create_format_converter(
>     input_pattern: str,
>     output_template: str | Callable
> ) -> Callable[[str], str]:
>     """입력 패턴과 출력 형식을 받아 변환 함수를 반환"""
>     compiled = re.compile(input_pattern)
>     def converter(text: str) -> str:
>         return compiled.sub(output_template, text)
>     return converter
>
> # 변환기 생성
> us_to_iso = create_format_converter(
>     r'(\d{2})/(\d{2})/(\d{4})',
>     r'\3-\1-\2'
> )
>
> phone_normalizer = create_format_converter(
>     r'(\d{3})[.\s-]?(\d{4})[.\s-]?(\d{4})',
>     r'\1-\2-\3'
> )
>
> # 사용
> print(us_to_iso("Date: 03/15/2024"))        # 'Date: 2024-03-15'
> print(phone_normalizer("010.1234.5678"))      # '010-1234-5678'
> ```
>
> 이 패턴은 변환 로직을 데이터(패턴 + 템플릿)로 분리하므로, 새로운 변환 규칙을 추가할 때 코드를 수정하지 않고 설정만 추가하면 됩니다.

### 흔한 실수와 주의사항

> ⚠️ **주의**: 역참조 번호(`\1`, `\2`)는 **패턴에서 여는 괄호의 순서**로 결정됩니다. 중첩된 그룹이 있으면 번호가 혼동되기 쉬우므로, 복잡한 치환에서는 명명 그룹(`\g<name>`)을 사용하는 것이 안전합니다.

> ⚠️ **주의**: 치환 문자열에서 `\1`은 파이썬 문자열 이스케이프로 해석될 수 있습니다. 반드시 raw string(`r'\1'`)을 사용하세요. `'\1'`은 ASCII 코드 1번 문자가 됩니다.

### Section 요약

- 포맷 변환은 캡처 그룹으로 의미 부분을 추출하고, 역참조로 재배치하는 패턴이다
- 단순 재배치는 `\1`, `\g<name>` 역참조로 충분하다
- 값 변환이 필요한 경우(패딩, 대소문자 변경 등)는 함수 치환을 사용한다
- camelCase ↔ snake_case 변환은 전후방 탐색(위치 기반 삽입)과 함수 치환의 조합이다
- 변환 규칙을 데이터로 분리하면 유지보수와 확장이 쉬워진다

---

## 13.4 구조화된 텍스트 파싱

> 💡 **한 줄 요약**: CSV, 키-값 쌍, 설정 파일 등 일정한 규칙을 가진 텍스트를 정규표현식으로 분해하여 프로그램이 다룰 수 있는 자료구조로 변환하는 것이 파싱입니다.

### 직관적 이해

"파싱(parsing)"은 텍스트를 구조적으로 **분해**하여 각 부분의 의미를 파악하는 과정입니다. 우리가 "이름: 홍길동, 나이: 30"이라는 문장을 읽으면 자연스럽게 "이름은 홍길동이고 나이는 30"이라고 이해합니다. 컴퓨터도 이런 구조를 프로그래밍적으로 처리하려면, 텍스트를 `{'이름': '홍길동', '나이': '30'}`같은 자료구조로 변환해야 합니다.

> 📝 **텍스트 파싱(Text Parsing)**: 일정한 구조를 가진 텍스트를 규칙에 따라 분해하여, 프로그램에서 활용 가능한 자료구조(딕셔너리, 리스트 등)로 변환하는 과정.

> 💬 **참고**: 정규표현식은 **정규 문법(regular grammar)**까지만 처리할 수 있습니다. 중첩 구조(예: HTML, JSON, 프로그래밍 언어)는 정규표현식의 이론적 한계를 넘어서므로, 전용 파서를 사용해야 합니다. 이 Chapter에서는 정규표현식으로 효과적으로 파싱할 수 있는 범위의 텍스트를 다룹니다. 정규표현식의 한계와 대안 도구는 Chapter 17에서 자세히 다룹니다.

### 핵심 개념 설명

**① 키-값 쌍 파싱**

설정 파일이나 로그에서 흔히 볼 수 있는 형식입니다.

```python
import re

config_text = """
host = localhost
port = 8080
debug = true
database_url = postgres://user:pass@localhost/mydb
# 이것은 주석입니다
app_name = My Application
"""

# 키-값 쌍 추출 (주석과 빈 줄 무시)
pattern = re.compile(r'^(?!#)(\w+)\s*=\s*(.+)$', re.MULTILINE)

config = {}
for m in pattern.finditer(config_text):
    key = m.group(1)
    value = m.group(2).strip()
    config[key] = value

print(config)
# {'host': 'localhost', 'port': '8080', 'debug': 'true',
#  'database_url': 'postgres://user:pass@localhost/mydb',
#  'app_name': 'My Application'}
```

패턴을 분석해 봅시다:
- `^(?!#)` — 줄의 시작이 `#`이 아닌 경우만 (주석 제외)
- `(\w+)` — 키 이름 캡처 (영문, 숫자, 언더스코어)
- `\s*=\s*` — 등호와 그 주변 공백
- `(.+)$` — 값 전체 캡처 (줄 끝까지)

> 🔗 **연결**: `(?!#)`은 Chapter 9의 부정 전방 탐색입니다. `re.MULTILINE`은 Chapter 10에서 배웠으며, 이 플래그를 사용하면 `^`가 각 줄의 시작에 매칭됩니다.

**② 간단한 CSV 파싱**

```python
# 기본 CSV 파싱 (따옴표 없는 간단한 경우)
csv_line = "홍길동,30,서울,개발자"
fields = re.split(r',', csv_line)
print(fields)  # ['홍길동', '30', '서울', '개발자']
```

하지만 현실의 CSV에는 필드 안에 쉼표가 포함될 수 있고, 이런 필드는 따옴표로 감쌉니다:

```python
csv_line = '홍길동,30,"서울특별시, 강남구",개발자'

# 따옴표 안의 쉼표를 무시하는 패턴
fields = re.findall(r'"([^"]*)"|\s*([^,]+)', csv_line)
# 결과: [('', '홍길동'), ('', '30'), ('서울특별시, 강남구', ''), ('', '개발자')]

# 튜플 정리: 비어있지 않은 쪽을 선택
cleaned = [quoted or unquoted for quoted, unquoted in fields]
print(cleaned)  # ['홍길동', '30', '서울특별시, 강남구', '개발자']
```

이 패턴에서 `"([^"]*)"` 부분이 따옴표로 감싼 필드를, `([^,]+)` 부분이 일반 필드를 처리합니다. 선택 연산자(`|`)로 두 경우를 모두 처리합니다.

> 💬 **참고**: 실무에서 CSV를 파싱할 때는 파이썬 표준 라이브러리의 `csv` 모듈을 사용하는 것이 더 안전하고 편리합니다. 정규표현식으로 CSV를 파싱하는 것은 학습 목적이거나, `csv` 모듈이 처리하지 못하는 비표준 형식을 다룰 때 유용합니다.

**③ 구분자 기반 텍스트 분할**

`re.split()`은 고정 구분자뿐 아니라 **패턴 기반 구분자**로 분할할 수 있습니다.

```python
# 여러 종류의 구분자로 분할
text = "apple;banana, cherry | grape   melon"
items = re.split(r'\s*[;,|]\s*|\s{2,}', text)
print(items)  # ['apple', 'banana', 'cherry', 'grape', 'melon']
```

이 패턴은 "세미콜론/쉼표/파이프(양쪽 공백 포함) 또는 2개 이상의 연속 공백"을 구분자로 인식합니다.

> 🔗 **연결**: `re.split()`은 Chapter 7에서 배운 함수입니다. 캡처 그룹이 구분자 패턴에 있으면 구분자도 결과에 포함된다는 점을 기억하세요.

**④ 반복되는 블록 파싱**

여러 줄에 걸친 반복 블록을 파싱하는 것은 좀 더 복잡합니다.

```python
text = """
[사원정보]
이름: 김철수
부서: 개발팀
직급: 과장

[사원정보]
이름: 이영희
부서: 디자인팀
직급: 대리
"""

# 각 블록을 먼저 분리
blocks = re.findall(
    r'\[사원정보\]\s*\n(.*?)(?=\[사원정보\]|\Z)',
    text,
    re.DOTALL
)

# 각 블록에서 키-값 추출
employees = []
kv_pattern = re.compile(r'^(\w+)\s*:\s*(.+)$', re.MULTILINE)

for block in blocks:
    emp = {}
    for m in kv_pattern.finditer(block):
        emp[m.group(1)] = m.group(2).strip()
    if emp:
        employees.append(emp)

print(employees)
# [{'이름': '김철수', '부서': '개발팀', '직급': '과장'},
#  {'이름': '이영희', '부서': '디자인팀', '직급': '대리'}]
```

이 예시는 두 단계로 파싱합니다:
1. `\[사원정보\]\s*\n(.*?)(?=\[사원정보\]|\Z)` — 각 `[사원정보]` 블록의 내용을 분리
2. `^(\w+)\s*:\s*(.+)$` — 블록 안에서 키-값 쌍을 추출

> 🔗 **연결**: `(?=\[사원정보\]|\Z)`는 Chapter 9의 전방 탐색입니다. 다음 블록 시작 또는 문자열 끝을 "보기만 하고" 소비하지 않으므로, 각 블록의 경계를 정확히 잡습니다. `re.DOTALL`은 Chapter 10에서 배운 플래그로, `.`이 줄바꿈도 매칭하게 합니다.

### 상세 예시

> 🔍 **예시 13.4.1: 환경 변수 파일(.env) 파싱**
>
> **상황**: 웹 프로젝트의 `.env` 파일을 파싱하여 딕셔너리로 변환해야 합니다.
> ```
> # Database settings
> DB_HOST=localhost
> DB_PORT=5432
> DB_NAME="my_database"
> DB_PASSWORD='s3cr3t_p@ss!'
>
> # App settings
> APP_DEBUG=true
> APP_SECRET_KEY="abc-123-xyz"
> ```
>
> **풀이/설명**:
> ```python
> import re
>
> env_text = """# Database settings
> DB_HOST=localhost
> DB_PORT=5432
> DB_NAME="my_database"
> DB_PASSWORD='s3cr3t_p@ss!'
>
> # App settings
> APP_DEBUG=true
> APP_SECRET_KEY="abc-123-xyz"
> """
>
> # 주석과 빈 줄을 제외하고, 키=값 추출
> # 값은 따옴표로 감싸져 있을 수도 있음
> pattern = re.compile(
>     r'^([A-Z_]+)=(?:"([^"]*)"|\'([^\']*)\'|(.+))$',
>     re.MULTILINE
> )
>
> env = {}
> for m in pattern.finditer(env_text):
>     key = m.group(1)
>     # 세 가지 값 형태 중 매칭된 것을 선택
>     value = m.group(2) or m.group(3) or m.group(4)
>     env[key] = value.strip() if value else ''
>
> for k, v in env.items():
>     print(f"{k} = {v}")
> # DB_HOST = localhost
> # DB_PORT = 5432
> # DB_NAME = my_database
> # DB_PASSWORD = s3cr3t_p@ss!
> # APP_DEBUG = true
> # APP_SECRET_KEY = abc-123-xyz
> ```
> **핵심 포인트**: 값이 큰따옴표, 작은따옴표, 또는 따옴표 없이 올 수 있는 세 가지 경우를 선택 연산자(`|`)로 처리합니다. 따옴표가 있는 경우 따옴표 안의 내용만 캡처하여 따옴표를 자동으로 제거합니다.

> 🔍 **예시 13.4.2: HTTP 쿼리 스트링 파싱**
>
> **상황**: URL의 쿼리 스트링을 파싱하여 딕셔너리로 변환합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> url = "https://example.com/search?q=python+regex&page=2&lang=ko&sort=date"
>
> # 쿼리 스트링 부분 추출
> qs_match = re.search(r'\?(.+)$', url)
> if qs_match:
>     qs = qs_match.group(1)
>     # 키=값 쌍 추출
>     params = dict(re.findall(r'([^&=]+)=([^&]*)', qs))
>     print(params)
> # {'q': 'python+regex', 'page': '2', 'lang': 'ko', 'sort': 'date'}
> ```
> **핵심 포인트**: `re.findall()`에 두 개의 캡처 그룹이 있으면 튜플의 리스트가 반환됩니다. 이것을 바로 `dict()`에 전달하면 딕셔너리가 됩니다. 이 패턴은 간결하면서도 효과적입니다.

### 흔한 실수와 주의사항

> ⚠️ **주의**: 정규표현식으로 JSON이나 XML을 파싱하려 하지 마세요. 이들은 재귀적 중첩 구조를 가지므로 정규표현식의 이론적 한계를 벗어납니다. 파이썬의 `json` 모듈이나 `xml.etree.ElementTree`를 사용하세요.

> ⚠️ **주의**: `re.split()`에서 캡처 그룹을 사용하면 구분자도 결과에 포함됩니다. 구분자가 결과에 포함되지 않기를 원하면 비캡처 그룹 `(?:...)`을 사용하세요.
> ```python
> re.split(r'(,)', 'a,b,c')     # ['a', ',', 'b', ',', 'c']  ← 구분자 포함
> re.split(r'(?:,)', 'a,b,c')   # ['a', 'b', 'c']            ← 구분자 미포함
> re.split(r',', 'a,b,c')       # ['a', 'b', 'c']            ← 캡처 없으면 미포함
> ```

### Section 요약

- 구조화된 텍스트 파싱은 규칙적인 형식의 텍스트를 프로그램용 자료구조로 변환하는 작업이다
- 키-값 쌍은 `(\w+)\s*=\s*(.+)` 패턴으로 추출한다
- CSV 파싱에서 따옴표 내 쉼표를 처리하려면 `"([^"]*)"` 패턴을 선택 연산자로 결합한다
- 반복 블록은 "블록 분리 → 블록 내 파싱"의 2단계 접근이 효과적이다
- JSON, XML 등 중첩 구조는 정규표현식이 아닌 전용 파서를 사용해야 한다

---

## 13.5 다단계 처리 파이프라인

> 💡 **한 줄 요약**: 복잡한 텍스트 처리를 한 번에 해결하려 하지 말고, 작은 변환 단계를 순차적으로 연결하는 "파이프라인"으로 구성하면 유지보수와 디버깅이 쉬워집니다.

### 직관적 이해

자동차 조립 공장의 **생산 라인(assembly line)**을 떠올려 봅시다. 한 사람이 자동차 한 대를 처음부터 끝까지 만들지 않습니다. 대신 "프레임 조립 → 엔진 장착 → 도장 → 내장재 설치 → 검수"처럼 각 단계가 자기 역할만 담당합니다. 각 단계는 단순하지만, 이들이 연결되면 복잡한 완성품이 만들어집니다.

텍스트 처리도 마찬가지입니다. 지저분한 원본 텍스트를 한 번의 정규표현식으로 깔끔하게 변환하려 하면, 패턴이 극도로 복잡해지고 디버깅이 불가능해집니다. 대신 **각 단계가 하나의 명확한 변환만 담당하는 파이프라인**을 구성하면, 각 단계가 간단하고 이해하기 쉬워집니다.

> 📝 **정규표현식 파이프라인(Regex Pipeline)**: 여러 개의 정규표현식 변환(`re.sub()`, `re.findall()` 등)을 순차적으로 연결하여 복잡한 텍스트 처리를 수행하는 설계 패턴. 각 단계의 출력이 다음 단계의 입력이 됩니다.

### 핵심 개념 설명

**파이프라인의 기본 구조**

가장 간단한 형태는 `re.sub()`를 순차적으로 호출하는 것입니다:

```python
import re

def clean_text(text):
    # 단계 1: HTML 태그 제거
    text = re.sub(r'<[^>]+>', '', text)
    # 단계 2: HTML 엔티티 변환
    text = re.sub(r'&amp;', '&', text)
    text = re.sub(r'&lt;', '<', text)
    text = re.sub(r'&gt;', '>', text)
    text = re.sub(r'&nbsp;', ' ', text)
    # 단계 3: 연속 공백 정리
    text = re.sub(r'\s+', ' ', text)
    # 단계 4: 앞뒤 공백 제거
    text = text.strip()
    return text

html = "<p>Hello &amp; <b>World</b></p>  &nbsp; Welcome!"
print(clean_text(html))
# 'Hello & World Welcome!'
```

이 코드는 잘 동작하지만, 단계가 많아지면 함수가 길어지고 규칙 관리가 어려워집니다. 더 **체계적인 파이프라인 구조**를 만들어 봅시다.

**규칙 기반 파이프라인**

변환 규칙을 **데이터(리스트)**로 분리하면 유지보수가 훨씬 쉬워집니다:

```python
import re

def apply_pipeline(text, rules):
    """변환 규칙 리스트를 순서대로 적용"""
    for pattern, replacement in rules:
        text = re.sub(pattern, replacement, text)
    return text

# 변환 규칙 정의 (패턴, 치환 문자열)
cleaning_rules = [
    (r'<[^>]+>', ''),           # HTML 태그 제거
    (r'&amp;', '&'),             # HTML 엔티티 변환
    (r'&lt;', '<'),
    (r'&gt;', '>'),
    (r'&nbsp;', ' '),
    (r'\s+', ' '),               # 연속 공백 → 단일 공백
]

html = "<p>Hello &amp; <b>World</b></p>  &nbsp; Welcome!"
result = apply_pipeline(html, cleaning_rules).strip()
print(result)  # 'Hello & World Welcome!'
```

이 구조의 장점:
- **규칙 추가/제거**가 리스트 편집만으로 가능
- **규칙 순서 변경**도 리스트 재배치만으로 가능
- **규칙 재사용**이 쉬움 (여러 파이프라인에서 같은 규칙 공유)
- **디버깅** 시 각 단계의 중간 결과를 쉽게 출력 가능

**디버깅 가능한 파이프라인**

개발 중에는 각 단계의 중간 결과를 확인할 수 있으면 매우 유용합니다:

```python
import re

def apply_pipeline(text, rules, debug=False):
    """변환 규칙 리스트를 순서대로 적용 (디버그 모드 지원)"""
    if debug:
        print(f"[입력] {text!r}")
    for i, (pattern, replacement) in enumerate(rules, 1):
        text = re.sub(pattern, replacement, text)
        if debug:
            print(f"[단계 {i}] 패턴: {pattern!r} → {text!r}")
    return text

# 디버그 모드로 실행
html = "<p>Hello &amp; <b>World</b></p>  &nbsp; Welcome!"
result = apply_pipeline(html, cleaning_rules, debug=True).strip()
```

출력:
```
[입력] '<p>Hello &amp; <b>World</b></p>  &nbsp; Welcome!'
[단계 1] 패턴: '<[^>]+>' → 'Hello &amp; World  &nbsp; Welcome!'
[단계 2] 패턴: '&amp;' → 'Hello & World  &nbsp; Welcome!'
[단계 3] 패턴: '&lt;' → 'Hello & World  &nbsp; Welcome!'
[단계 4] 패턴: '&gt;' → 'Hello & World  &nbsp; Welcome!'
[단계 5] 패턴: '&nbsp;' → 'Hello & World   Welcome!'
[단계 6] 패턴: '\\s+' → 'Hello & World Welcome!'
```

**함수 치환을 포함하는 고급 파이프라인**

단순 문자열 치환뿐 아니라, 함수 치환이 필요한 단계도 있을 수 있습니다:

```python
import re

def apply_advanced_pipeline(text, rules):
    """패턴 + 문자열/함수 치환 모두 지원"""
    for rule in rules:
        pattern = rule['pattern']
        replacement = rule['replacement']
        flags = rule.get('flags', 0)
        text = re.sub(pattern, replacement, text, flags=flags)
    return text

# 고급 규칙 정의
advanced_rules = [
    {'pattern': r'<[^>]+>', 'replacement': ''},
    {'pattern': r'&amp;', 'replacement': '&'},
    {'pattern': r'\b[A-Z]+\b',
     'replacement': lambda m: m.group().capitalize(),  # 모두 대문자 → 첫글자만 대문자
    },
    {'pattern': r'\s+', 'replacement': ' '},
]
```

### 상세 예시

> 🔍 **예시 13.5.1: 웹 스크래핑 데이터 정제 파이프라인**
>
> **상황**: 웹 페이지에서 스크래핑한 상품 설명 텍스트를 깔끔하게 정리해야 합니다.
> ```python
> raw = """
> <div class="desc">
>   <p>★★★ 최고의   상품! ★★★</p>
>   <p>가격: ￦49,900원 (배송비 무료!!!)  </p>
>   <p>색상: 블랙/화이트/레드  &amp; 블루</p>
>   <p>사이즈:   S / M / L / XL  </p>
>   <p>※ 교환/반품은   7일 이내...</p>
> </div>
> """
> ```
>
> **풀이/설명**:
> ```python
> import re
>
> raw = """
> <div class="desc">
>   <p>★★★ 최고의   상품! ★★★</p>
>   <p>가격: ￦49,900원 (배송비 무료!!!)  </p>
>   <p>색상: 블랙/화이트/레드  &amp; 블루</p>
>   <p>사이즈:   S / M / L / XL  </p>
>   <p>※ 교환/반품은   7일 이내...</p>
> </div>
> """
>
> cleaning_rules = [
>     (r'<[^>]+>', ''),               # 1. HTML 태그 제거
>     (r'&amp;', '&'),                 # 2. HTML 엔티티 변환
>     (r'★+', ''),                     # 3. 장식 문자 제거
>     (r'([!?.])\1+', r'\1'),          # 4. 반복 구두점 제거
>     (r'[ \t]+', ' '),                # 5. 연속 공백 → 단일 공백 (줄바꿈 보존)
>     (r'^\s+|\s+$', '', ),            # 6. 각 줄 앞뒤 공백 — 아래에서 줄 단위 처리
> ]
>
> # 줄 단위 처리 + 빈 줄 제거
> def clean_product_description(raw_text):
>     # 기본 파이프라인 적용
>     text = raw_text
>     for pattern, replacement in cleaning_rules[:5]:
>         text = re.sub(pattern, replacement, text)
>     
>     # 줄 단위 정리
>     lines = text.strip().split('\n')
>     lines = [line.strip() for line in lines if line.strip()]
>     
>     return '\n'.join(lines)
>
> print(clean_product_description(raw))
> # 최고의 상품!
> # 가격: ￦49,900원 (배송비 무료!)
> # 색상: 블랙/화이트/레드 & 블루
> # 사이즈: S / M / L / XL
> # ※ 교환/반품은 7일 이내...
> ```
> **핵심 포인트**: 파이프라인의 각 단계는 하나의 명확한 역할만 담당합니다. 규칙 순서가 중요합니다 — HTML 태그를 먼저 제거해야 태그 안의 공백이 불필요하게 보존되지 않습니다.

> 🔍 **예시 13.5.2: 고객 데이터 정규화 + 추출 파이프라인**
>
> **상황**: 고객 문의 이메일에서 고객 정보를 자동으로 추출하고 정규화하는 파이프라인을 구축합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> email_body = """
> 안녕하세요. 주문 관련 문의드립니다.
>
> 이름:   홍   길동
> 전화: 010.1234.5678
> 주문번호: ORD-2024-00123
> 주문일자: 2024년 3월 15일
> 금액: 1,250,000원
>
> 배송지 변경 부탁드립니다.
> 새 주소: 서울시  강남구  역삼동  123-45
> """
>
> # ==== 1단계: 정규화 파이프라인 ====
> normalization_rules = [
>     (r'[ \t]{2,}', ' '),           # 연속 공백 정리
>     (r'(\d{3})[.\s](\d{4})[.\s](\d{4})', r'\1-\2-\3'),  # 전화번호 통일
> ]
>
> normalized = email_body
> for pattern, replacement in normalization_rules:
>     normalized = re.sub(pattern, replacement, normalized)
>
> # ==== 2단계: 추출 파이프라인 ====
> extraction_patterns = {
>     'name': r'이름\s*:\s*([가-힣\s]+)',
>     'phone': r'전화\s*:\s*(\d{3}-\d{4}-\d{4})',
>     'order_no': r'주문번호\s*:\s*(ORD-\d{4}-\d+)',
>     'order_date': r'주문일자\s*:\s*(\d{4}년\s*\d{1,2}월\s*\d{1,2}일)',
>     'amount': r'금액\s*:\s*([\d,]+원)',
>     'address': r'주소\s*:\s*(.+)',
> }
>
> customer_info = {}
> for field, pattern in extraction_patterns.items():
>     m = re.search(pattern, normalized)
>     if m:
>         customer_info[field] = m.group(1).strip()
>
> # ==== 3단계: 후처리 (값 변환) ====
> if 'name' in customer_info:
>     # 이름 내 불필요한 공백 제거
>     customer_info['name'] = re.sub(r'\s+', '', customer_info['name'])
>
> if 'order_date' in customer_info:
>     # 한국어 날짜 → ISO 형식
>     m = re.match(r'(\d{4})년\s*(\d{1,2})월\s*(\d{1,2})일',
>                  customer_info['order_date'])
>     if m:
>         customer_info['order_date'] = (
>             f"{m.group(1)}-{m.group(2).zfill(2)}-{m.group(3).zfill(2)}"
>         )
>
> if 'amount' in customer_info:
>     # "1,250,000원" → 정수 1250000
>     amount_str = re.sub(r'[,원]', '', customer_info['amount'])
>     customer_info['amount'] = int(amount_str)
>
> for k, v in customer_info.items():
>     print(f"  {k}: {v}")
> # name: 홍길동
> # phone: 010-1234-5678
> # order_no: ORD-2024-00123
> # order_date: 2024-03-15
> # amount: 1250000
> # address: 서울시 강남구 역삼동 123-45
> ```
> **핵심 포인트**: 이 예시는 **정규화 → 추출 → 후처리**라는 전형적인 3단계 파이프라인을 보여줍니다. 각 단계의 역할이 명확히 구분되어 있으므로, 특정 단계에 문제가 있으면 해당 단계만 수정하면 됩니다.

### 깊이 있는 이해

> 🔬 **Deep Dive**: 파이프라인 설계 원칙
>
> *이 부분은 심화 내용입니다. 처음 학습 시 건너뛰어도 됩니다.*
>
> 효과적인 텍스트 처리 파이프라인을 설계할 때 고려할 원칙들:
>
> **1. 순서 의존성 (Order Dependency)**
> 규칙의 순서가 결과에 영향을 미칩니다. 일반적으로:
> - **제거(삭제)** 규칙을 먼저 적용 (노이즈 제거)
> - **정규화(통일)** 규칙을 다음에 적용 (형식 통일)
> - **추출** 규칙을 마지막에 적용 (정리된 데이터에서 추출)
>
> **2. 멱등성 (Idempotency)**
> 좋은 파이프라인은 같은 입력에 두 번 적용해도 결과가 변하지 않아야 합니다:
> ```python
> # 멱등적인 규칙
> r'\s+' → ' '  # 이미 단일 공백이면 그대로
>
> # 비멱등적인 규칙 (주의 필요)
> r'(\w)' → r'\1\1'  # 적용할 때마다 글자가 두 배로
> ```
>
> **3. 테스트 가능성 (Testability)**
> 각 단계를 독립적으로 테스트할 수 있도록 설계합니다:
> ```python
> def test_pipeline_step(pattern, replacement, test_cases):
>     """각 규칙을 독립적으로 테스트"""
>     for input_text, expected in test_cases:
>         result = re.sub(pattern, replacement, input_text)
>         assert result == expected, \
>             f"Failed: {input_text!r} → {result!r} (expected {expected!r})"
>     print(f"패턴 {pattern!r}: 모든 테스트 통과")
>
> test_pipeline_step(r'\s+', ' ', [
>     ("a  b", "a b"),
>     ("a\t\tb", "a b"),
>     ("a b", "a b"),  # 이미 정규화된 경우
> ])
> ```

### 흔한 실수와 주의사항

> ⚠️ **주의**: 파이프라인의 **규칙 순서**가 매우 중요합니다. 예를 들어, 공백 정규화(`\s+` → ` `)를 HTML 태그 제거 전에 적용하면, `<div class="a">` 같은 태그의 내부 공백이 먼저 처리되어 태그 제거 패턴이 깨질 수 있습니다.

> ⚠️ **주의**: 하나의 초거대 정규표현식으로 모든 것을 해결하려는 유혹에 빠지지 마세요. 100자 이상의 단일 패턴보다, 20자짜리 패턴 5개를 순차 적용하는 것이 읽기 쉽고, 디버깅하기 쉽고, 유지보수하기 쉽습니다.

### Section 요약

- **파이프라인**은 여러 정규표현식 변환을 순차적으로 연결하는 설계 패턴이다
- 변환 규칙을 리스트로 분리하면 추가/제거/순서 변경이 쉬워진다
- `debug=True` 옵션으로 각 단계의 중간 결과를 확인할 수 있게 하면 디버깅이 편리하다
- 전형적인 파이프라인 구조는 **노이즈 제거 → 정규화 → 추출 → 후처리** 순서이다
- 단일 복잡 패턴보다 여러 간단한 패턴의 조합이 항상 바람직하다

---

# Part C: 통합 및 정리 (Integration Block)

---

## Chapter 핵심 요약 (Chapter Summary)

이 Chapter에서는 정규표현식을 활용한 실전 텍스트 처리의 네 가지 핵심 영역을 학습했습니다.

**13.1 텍스트 정규화**: 지저분한 텍스트를 일관된 형태로 통일하는 방법을 배웠습니다. `\s+` → `' '`로 공백을 정리하고, `(.)\1{2,}` → `\1\1`로 반복 문자를 제한하며, 특수문자 제거와 대소문자 통일 기법을 익혔습니다.

**13.2 데이터 추출 패턴**: 비정형 텍스트에서 금액, 날짜, 연락처 등 구조화된 정보를 추출하는 방법을 배웠습니다. 명명 그룹과 `groupdict()`를 활용하면 추출 결과를 바로 딕셔너리로 변환할 수 있습니다.

**13.3 포맷 변환**: 캡처 그룹의 역참조 치환을 활용해 날짜 형식 변환, 전화번호 통일, camelCase ↔ snake_case 변환 등을 구현했습니다. 단순 재배치는 역참조로, 값 변환이 필요한 경우는 함수 치환으로 처리합니다.

**13.4 구조화된 텍스트 파싱**: 키-값 쌍, CSV, 환경 변수 파일 등을 파싱하여 딕셔너리나 리스트로 변환하는 방법을 배웠습니다. 반복 블록은 "블록 분리 → 블록 내 파싱"의 2단계 접근이 효과적입니다.

**13.5 다단계 처리 파이프라인**: 복잡한 텍스트 처리를 여러 간단한 단계로 분해하여 순차 적용하는 파이프라인 설계 패턴을 학습했습니다. 규칙을 데이터로 분리하면 유지보수와 확장이 쉬워집니다.

---

## 핵심 용어 정리 (Glossary)

| 용어 | 정의 | 관련 Section |
|------|------|-------------|
| 텍스트 정규화 (Text Normalization) | 다양한 형태의 텍스트를 하나의 일관된 표준 형식으로 변환하는 과정 | 13.1 |
| 데이터 추출 (Data Extraction) | 비정형 텍스트에서 구조화된 정보를 패턴 기반으로 찾아내는 작업 | 13.2 |
| 포맷 변환 (Format Conversion) | 데이터의 내용은 유지하면서 표현 형식만 다른 표준으로 바꾸는 작업 | 13.3 |
| 텍스트 파싱 (Text Parsing) | 규칙적인 구조의 텍스트를 분해하여 프로그래밍 자료구조로 변환하는 과정 | 13.4 |
| 정규표현식 파이프라인 (Regex Pipeline) | 여러 정규표현식 변환을 순차적으로 연결하여 복잡한 처리를 수행하는 설계 패턴 | 13.5 |
| 텍스트 토큰화 (Tokenization) | 텍스트를 의미 있는 단위(토큰)로 분리하는 과정 | 13.2, 13.4 |

---

## 개념 연결 맵 (Connection Map)

**이전 Chapter와의 연결**:
- Chapter 7(치환과 분할)에서 배운 `re.sub()`과 역참조 치환이 이 Chapter의 **모든 Section에서 핵심 도구**로 활용됨
- Chapter 8(고급 그룹핑)의 명명 그룹과 `groupdict()`가 **데이터 추출**(13.2)의 핵심
- Chapter 9(전후방 탐색)이 **포맷 변환**(13.3)의 위치 기반 삽입과 **파싱**(13.4)의 블록 경계 인식에 활용됨
- Chapter 12(데이터 검증)의 패턴 설계 경험이 **추출 패턴 설계**의 기반

**다음 Chapter와의 연결**:
- 이 Chapter의 텍스트 처리 기법들은 Chapter 14(로그 분석과 파일 자동화)에서 **실제 파일 단위로 확장**됩니다
- 파이프라인 설계 패턴은 Chapter 14에서 **대량 파일 일괄 처리**에 적용됩니다
- 복잡한 패턴의 성능 문제는 Chapter 16(성능 최적화)에서 다루게 됩니다

---

## 자기 점검 질문 (Self-Check Questions)

**Q1.** `re.sub(r'\s+', ' ', "a  b\n\nc")`의 결과는 무엇인가요?

<details>
<summary>정답</summary>

`'a b c'` — `\s+`는 줄바꿈을 포함한 모든 공백 문자를 매칭하므로, `\n\n`도 단일 공백으로 치환됩니다.
</details>

**Q2.** 검증(validation)에 주로 사용하는 함수와 추출(extraction)에 주로 사용하는 함수는 각각 무엇인가요?

<details>
<summary>정답</summary>

검증: `re.fullmatch()` 또는 `re.match()` / 추출: `re.findall()` 또는 `re.finditer()`
</details>

**Q3.** `re.sub(r'(\d{2})/(\d{2})/(\d{4})', r'\3-\1-\2', "03/15/2024")`의 결과는?

<details>
<summary>정답</summary>

`'2024-03-15'` — 그룹1(03), 그룹2(15), 그룹3(2024)을 `\3-\1-\2` 순서로 재배치합니다.
</details>

**Q4.** `re.split()`에 캡처 그룹이 포함된 패턴을 사용하면 어떤 차이가 있나요?

<details>
<summary>정답</summary>

구분자에 캡처 그룹이 있으면 구분자 자체도 결과 리스트에 포함됩니다. 예: `re.split(r'(,)', 'a,b')` → `['a', ',', 'b']`. 구분자를 제외하려면 비캡처 그룹 `(?:,)` 또는 그룹 없이 `,`를 사용합니다.
</details>

**Q5.** 정규표현식으로 JSON을 파싱하면 안 되는 이유는 무엇인가요?

<details>
<summary>정답</summary>

JSON은 재귀적 중첩 구조(객체 안에 객체, 배열 안에 배열)를 가지며, 이는 정규표현식이 처리할 수 있는 정규 문법의 범위를 넘어섭니다. 파이썬의 `json` 모듈 같은 전용 파서를 사용해야 합니다.
</details>

**Q6.** 파이프라인에서 규칙의 순서가 중요한 이유를 한 가지 예시와 함께 설명하세요.

<details>
<summary>정답</summary>

HTML 태그 제거와 공백 정규화의 순서가 대표적입니다. 태그 제거를 먼저 해야 `<div class="a">`의 내부 공백이 태그와 함께 제거됩니다. 공백 정규화를 먼저 하면 태그 내부 공백이 변경되어 태그 제거 패턴이 의도대로 동작하지 않을 수 있습니다.
</details>

**Q7.** `(.)\1{2,}`를 `\1\1`로 치환하면 어떤 효과가 있나요?

<details>
<summary>정답</summary>

임의의 문자가 3번 이상 연속 반복되는 경우를 최대 2번까지만 허용합니다. 예: `"ㅋㅋㅋㅋㅋ"` → `"ㅋㅋ"`, `"!!!"` → `"!!"`. SNS 데이터나 채팅 로그 전처리에 흔히 사용됩니다.
</details>

---

# Part D: 문제 풀이 (Problem Set)

---

## D1. Practice (연습 문제) — 기본 개념 확인

> **Practice 13.1**: 다음 문자열에서 연속된 공백을 단일 공백으로 변환하고, 앞뒤 공백을 제거하세요.
> ```python
> text = "   파이썬   정규표현식   입문   "
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> result = re.sub(r'\s+', ' ', text).strip()
> # '파이썬 정규표현식 입문'
> ```
> **해설**: `\s+`는 하나 이상의 연속 공백을 매칭하고, 이를 단일 공백으로 치환합니다. `.strip()`으로 앞뒤 공백을 최종 제거합니다.
> </details>

> **Practice 13.2**: 다음 텍스트에서 모든 금액(숫자+"원")을 추출하세요.
> ```python
> text = "사과 1,500원, 바나나 2,000원, 포도 8,900원"
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> amounts = re.findall(r'[\d,]+원', text)
> # ['1,500원', '2,000원', '8,900원']
> ```
> **해설**: `[\d,]+`는 숫자와 쉼표의 조합을, `원`은 리터럴 문자를 매칭합니다. `findall()`은 텍스트 내 모든 매칭을 리스트로 반환합니다.
> </details>

> **Practice 13.3**: 다음 날짜를 "YYYY/MM/DD" → "YYYY-MM-DD" 형식으로 변환하세요.
> ```python
> text = "시작일: 2024/03/15, 종료일: 2024/12/31"
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> result = re.sub(r'(\d{4})/(\d{2})/(\d{2})', r'\1-\2-\3', text)
> # '시작일: 2024-03-15, 종료일: 2024-12-31'
> ```
> **해설**: 세 그룹(년, 월, 일)을 캡처한 뒤, 슬래시를 하이픈으로 바꿔 역참조로 재배치합니다.
> </details>

> **Practice 13.4**: 다음 HTML 텍스트에서 태그를 제거하여 순수 텍스트만 추출하세요.
> ```python
> html = "<h1>제목</h1><p>본문 <b>강조</b> 텍스트</p>"
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> clean = re.sub(r'<[^>]+>', '', html)
> # '제목본문 강조 텍스트'
> ```
> **해설**: `<[^>]+>`는 `<`로 시작하고 `>` 이외의 문자가 하나 이상 온 뒤 `>`로 끝나는 부분, 즉 HTML 태그를 매칭합니다. 이를 빈 문자열로 치환하면 태그가 제거됩니다.
> </details>

> **Practice 13.5**: 다음 키-값 텍스트를 딕셔너리로 변환하세요.
> ```python
> text = "name=홍길동&age=30&city=서울"
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> params = dict(re.findall(r'([^&=]+)=([^&]*)', text))
> # {'name': '홍길동', 'age': '30', 'city': '서울'}
> ```
> **해설**: `([^&=]+)=([^&]*)`는 `&`와 `=`가 아닌 문자로 된 키와 값을 캡처합니다. `findall()`이 튜플 리스트를 반환하고, `dict()`이 이를 딕셔너리로 변환합니다.
> </details>

> **Practice 13.6**: 다음 텍스트에서 반복되는 문장부호를 하나로 줄이세요.
> ```python
> text = "정말요???!!! 대단해요.... 와우!!!"
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> result = re.sub(r'([!?.])\1+', r'\1', text)
> # '정말요?! 대단해요. 와우!'
> ```
> **해설**: `([!?.])`은 구두점 하나를 캡처하고, `\1+`는 같은 문자의 반복을 매칭합니다. `\1`로 치환하면 하나만 남습니다. `???`는 `?`로, `!!!`는 `!`로, `....`는 `.`으로 줄어듭니다.
> </details>

> **Practice 13.7**: 다음 전화번호들을 모두 "XXX-XXXX-XXXX" 형식으로 통일하세요.
> ```python
> phones = ["01012345678", "010.1234.5678", "010 1234 5678"]
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> for phone in phones:
>     unified = re.sub(r'(\d{3})[.\s-]?(\d{4})[.\s-]?(\d{4})', r'\1-\2-\3', phone)
>     print(unified)
> # 010-1234-5678
> # 010-1234-5678
> # 010-1234-5678
> ```
> **해설**: `[.\s-]?`는 구분자(마침표, 공백, 하이픈)가 있을 수도 없을 수도 있는 상황을 처리합니다. 세 숫자 그룹을 캡처한 후 하이픈으로 재결합합니다.
> </details>

---

## D2. Exercise (응용 문제) — 개념 적용 및 통합

> **Exercise 13.1**: 다음 비정형 텍스트에서 **모든 이메일 주소를 추출**하고, 도메인 부분을 소문자로 정규화하여 리스트로 반환하는 함수를 작성하세요.
> ```python
> text = """
> 문의: support@Example.COM
> 담당자: admin@COMPANY.co.kr
> CC: user.name+tag@Gmail.Com
> """
> ```
>
> **힌트**: 추출(`finditer`)과 함수 치환 또는 후처리를 결합하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: `finditer()`로 이메일을 모두 찾은 뒤, 각 이메일의 `@` 뒤 부분만 소문자로 변환합니다.
>
> **풀이 과정**:
> ```python
> import re
>
> def extract_normalized_emails(text):
>     email_pattern = re.compile(r'[\w.+-]+@[\w-]+\.[\w.]+')
>     emails = []
>     for m in email_pattern.finditer(text):
>         email = m.group()
>         # @ 기준으로 분리하여 도메인만 소문자 처리
>         local, domain = email.split('@', 1)
>         emails.append(f"{local}@{domain.lower()}")
>     return emails
>
> text = """
> 문의: support@Example.COM
> 담당자: admin@COMPANY.co.kr
> CC: user.name+tag@Gmail.Com
> """
>
> print(extract_normalized_emails(text))
> # ['support@example.com', 'admin@company.co.kr', 'user.name+tag@gmail.com']
> ```
>
> **정답**: `['support@example.com', 'admin@company.co.kr', 'user.name+tag@gmail.com']`
>
> **보충 설명**: 이메일 주소의 로컬 파트(@ 앞)는 대소문자를 구분할 수 있으므로 원래 대로 유지하고, 도메인 파트만 소문자로 변환하는 것이 RFC 표준에 더 가깝습니다.
> </details>

> **Exercise 13.2**: 다음 텍스트에서 **camelCase로 된 변수명을 모두 찾아 snake_case로 변환**하세요. 변수명은 소문자로 시작하고 대문자를 하나 이상 포함합니다.
> ```python
> code = """
> let userName = getUserName();
> const maxRetryCount = 3;
> var isActive = checkStatus(userId);
> print(totalAmount);
> """
> ```
>
> **힌트**: camelCase 패턴을 먼저 추출한 뒤, 각각을 snake_case로 변환하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: camelCase 단어를 찾는 패턴을 작성하고, `re.sub()`에 함수를 전달하여 변환합니다.
>
> **풀이 과정**:
> ```python
> import re
>
> def camel_to_snake(name):
>     result = re.sub(r'([a-z0-9])([A-Z])', r'\1_\2', name)
>     return result.lower()
>
> code = """
> let userName = getUserName();
> const maxRetryCount = 3;
> var isActive = checkStatus(userId);
> print(totalAmount);
> """
>
> # camelCase 단어 찾기: 소문자 시작 + 대문자 포함
> result = re.sub(
>     r'\b([a-z]+(?:[A-Z][a-z0-9]*)+)\b',
>     lambda m: camel_to_snake(m.group()),
>     code
> )
> print(result)
> # let user_name = get_user_name();
> # const max_retry_count = 3;
> # var is_active = check_status(user_id);
> # print(total_amount);
> ```
>
> **정답**: 모든 camelCase 변수/함수명이 snake_case로 변환됨
>
> **보충 설명**: `[a-z]+(?:[A-Z][a-z0-9]*)+`는 "소문자로 시작하고, 대문자+소문자 조합이 하나 이상 뒤따르는 단어"를 매칭합니다. 이 패턴은 `let`, `var`, `const`, `print` 같은 순수 소문자 단어는 매칭하지 않습니다.
> </details>

> **Exercise 13.3**: 다음 반복 블록 형식의 텍스트를 파싱하여 **딕셔너리의 리스트**로 변환하세요.
> ```python
> data = """
> ---
> 제품: 노트북
> 가격: 1,500,000원
> 재고: 25
> ---
> 제품: 마우스
> 가격: 35,000원
> 재고: 150
> ---
> 제품: 키보드
> 가격: 89,000원
> 재고: 80
> """
> ```
>
> **힌트**: `---`를 구분자로 블록을 분리한 뒤, 각 블록에서 키-값을 추출하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 2단계 파싱 — 블록 분리 후 블록 내 키-값 추출
>
> **풀이 과정**:
> ```python
> import re
>
> data = """
> ---
> 제품: 노트북
> 가격: 1,500,000원
> 재고: 25
> ---
> 제품: 마우스
> 가격: 35,000원
> 재고: 150
> ---
> 제품: 키보드
> 가격: 89,000원
> 재고: 80
> """
>
> # 1단계: --- 구분자로 블록 분리
> blocks = re.split(r'-{3,}', data)
>
> # 2단계: 각 블록에서 키-값 추출
> kv_pattern = re.compile(r'^(.+?)\s*:\s*(.+)$', re.MULTILINE)
> products = []
>
> for block in blocks:
>     block = block.strip()
>     if not block:
>         continue
>     product = {}
>     for m in kv_pattern.finditer(block):
>         product[m.group(1).strip()] = m.group(2).strip()
>     if product:
>         products.append(product)
>
> print(products)
> # [{'제품': '노트북', '가격': '1,500,000원', '재고': '25'},
> #  {'제품': '마우스', '가격': '35,000원', '재고': '150'},
> #  {'제품': '키보드', '가격': '89,000원', '재고': '80'}]
> ```
>
> **정답**: 3개의 딕셔너리가 담긴 리스트
>
> **보충 설명**: `re.split(r'-{3,}', data)`로 3개 이상의 연속 하이픈을 구분자로 사용합니다. 빈 블록을 `if not block: continue`로 건너뛰는 것이 중요합니다. 키-값 패턴에서 `(.+?)`의 비탐욕적 수량자는 첫 번째 `:`까지만 매칭하게 합니다(값에 `:`이 포함될 수 있으므로).
> </details>

> **Exercise 13.4**: 다음 텍스트 정제 파이프라인을 구현하세요. 원본 텍스트에 아래 규칙을 순서대로 적용합니다:
> 1. HTML 태그 제거
> 2. `&nbsp;` → 공백, `&amp;` → `&` 변환
> 3. 연속 공백을 단일 공백으로
> 4. 각 줄의 앞뒤 공백 제거
> 5. 빈 줄 제거
>
> ```python
> raw = """
> <div>
>   <h1>  공지사항  </h1>
>   <p>신규 서비스 &amp; 업데이트&nbsp;&nbsp;안내</p>
>   <p>  </p>
>   <p>자세한 내용은   <a href="#">여기</a>를   참조하세요.  </p>
> </div>
> """
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 규칙을 리스트로 정의하고 순차 적용하는 파이프라인 구현
>
> **풀이 과정**:
> ```python
> import re
>
> raw = """
> <div>
>   <h1>  공지사항  </h1>
>   <p>신규 서비스 &amp; 업데이트&nbsp;&nbsp;안내</p>
>   <p>  </p>
>   <p>자세한 내용은   <a href="#">여기</a>를   참조하세요.  </p>
> </div>
> """
>
> # 단순 치환 규칙
> sub_rules = [
>     (r'<[^>]+>', ''),        # 1. HTML 태그 제거
>     (r'&nbsp;', ' '),         # 2a. &nbsp; → 공백
>     (r'&amp;', '&'),          # 2b. &amp; → &
>     (r'[ \t]+', ' '),         # 3. 연속 공백 → 단일 공백 (줄바꿈 보존)
> ]
>
> text = raw
> for pattern, replacement in sub_rules:
>     text = re.sub(pattern, replacement, text)
>
> # 4+5. 줄 단위 처리: 앞뒤 공백 제거 + 빈 줄 제거
> lines = [line.strip() for line in text.split('\n') if line.strip()]
> result = '\n'.join(lines)
>
> print(result)
> # 공지사항
> # 신규 서비스 & 업데이트 안내
> # 자세한 내용은 여기를 참조하세요.
> ```
>
> **정답**: 깔끔하게 정리된 3줄의 텍스트
>
> **보충 설명**: 3단계에서 `\s+` 대신 `[ \t]+`를 사용하여 줄바꿈을 보존한 것이 핵심입니다. 줄바꿈을 보존해야 4~5단계에서 줄 단위 처리가 가능합니다.
> </details>

---

## D3. Problem (심화 문제) — 깊이 있는 사고와 창의적 문제 해결

> **Problem 13.1**: **범용 텍스트 정제 클래스**를 설계하세요. 다음 요구사항을 만족해야 합니다:
>
> 1. 규칙을 순서대로 등록할 수 있어야 합니다 (패턴 + 치환 문자열 또는 함수)
> 2. 등록된 규칙을 순서대로 적용하는 `process()` 메서드가 있어야 합니다
> 3. 디버그 모드에서 각 단계의 중간 결과를 출력해야 합니다
> 4. 규칙을 이름으로 관리하여 특정 규칙을 비활성화할 수 있어야 합니다
>
> 이 클래스를 사용하여 "HTML 태그가 포함된 한국어 텍스트"를 정제하는 파이프라인을 구성하고 테스트하세요.
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 규칙을 딕셔너리 리스트로 관리하고, 각 규칙에 이름과 활성화 여부를 추가합니다.
>
> **풀이 전략**: 클래스 설계 → 규칙 등록 메서드 → 처리 메서드 → 디버그 지원
>
> **상세 풀이**:
> ```python
> import re
>
> class TextCleaner:
>     def __init__(self, debug=False):
>         self.rules = []
>         self.debug = debug
>
>     def add_rule(self, name, pattern, replacement, enabled=True):
>         """변환 규칙 추가"""
>         self.rules.append({
>             'name': name,
>             'pattern': re.compile(pattern) if isinstance(pattern, str) else pattern,
>             'replacement': replacement,
>             'enabled': enabled,
>         })
>         return self  # 메서드 체이닝 지원
>
>     def disable_rule(self, name):
>         """이름으로 규칙 비활성화"""
>         for rule in self.rules:
>             if rule['name'] == name:
>                 rule['enabled'] = False
>         return self
>
>     def enable_rule(self, name):
>         """이름으로 규칙 활성화"""
>         for rule in self.rules:
>             if rule['name'] == name:
>                 rule['enabled'] = True
>         return self
>
>     def process(self, text):
>         """등록된 규칙을 순서대로 적용"""
>         if self.debug:
>             print(f"[입력] {text!r}\n")
>         for rule in self.rules:
>             if not rule['enabled']:
>                 if self.debug:
>                     print(f"  [건너뜀] {rule['name']}")
>                 continue
>             pattern = rule['pattern']
>             text = pattern.sub(rule['replacement'], text)
>             if self.debug:
>                 print(f"  [{rule['name']}] → {text!r}")
>         return text
>
> # 사용 예시
> cleaner = TextCleaner(debug=True)
> cleaner.add_rule('html_tags', r'<[^>]+>', '')
> cleaner.add_rule('html_entities', r'&amp;', '&')
> cleaner.add_rule('nbsp', r'&nbsp;', ' ')
> cleaner.add_rule('repeat_chars', r'(.)\1{2,}', r'\1\1')
> cleaner.add_rule('multi_space', r'\s+', ' ')
>
> raw = "<p>안녕하세요!!! &amp; 반갑습니다ㅋㅋㅋㅋ&nbsp;</p>"
> result = cleaner.process(raw).strip()
> print(f"\n[최종] {result}")
> # [최종] 안녕하세요!! & 반갑습니다ㅋㅋ
> ```
>
> **확장 생각**: 이 패턴을 더 발전시키면, 규칙을 YAML/JSON 파일에서 로드하거나, 규칙별 적용 횟수 통계를 수집하거나, 규칙 실행 순서를 동적으로 변경하는 기능을 추가할 수 있습니다.
> </details>

> **Problem 13.2**: 다음과 같은 **비정형 회의록 텍스트**에서 정보를 추출하여 구조화된 JSON 형식의 딕셔너리로 변환하는 프로그램을 작성하세요.
>
> ```python
> minutes = """
> === 주간 회의록 ===
> 일시: 2024년 3월 15일 (금) 오후 2:00~3:30
> 장소: 본사 3층 대회의실
> 참석자: 김철수(팀장), 이영희(과장), 박민수(대리), 정수진(사원)
>
> [안건 1] 신규 프로젝트 킥오프
> - 프로젝트명: Project Alpha
> - 예산: 5,000만원
> - 일정: 2024-04-01 ~ 2024-09-30
> - 담당: 이영희
>
> [안건 2] 서버 마이그레이션
> - 현재 상태: AWS EC2 (t3.large)
> - 목표: AWS ECS Fargate 전환
> - 예상 비용 절감: 월 200만원
> - 담당: 박민수
>
> [결정사항]
> 1. Project Alpha 4월 1일 착수 확정
> 2. 서버 마이그레이션 POC 3월 말 완료
> 3. 다음 회의: 2024년 3월 22일
> """
> ```
>
> 추출할 정보: 일시, 장소, 참석자 목록, 각 안건의 세부 정보, 결정사항 목록
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 문서를 논리적 블록(헤더, 안건, 결정사항)으로 분리한 뒤, 각 블록에 맞는 파싱 로직을 적용합니다.
>
> **풀이 전략**: 메타데이터 추출 → 안건 블록 분리 → 안건 내 상세 추출 → 결정사항 추출
>
> **상세 풀이**:
> ```python
> import re
>
> minutes = """
> === 주간 회의록 ===
> 일시: 2024년 3월 15일 (금) 오후 2:00~3:30
> 장소: 본사 3층 대회의실
> 참석자: 김철수(팀장), 이영희(과장), 박민수(대리), 정수진(사원)
>
> [안건 1] 신규 프로젝트 킥오프
> - 프로젝트명: Project Alpha
> - 예산: 5,000만원
> - 일정: 2024-04-01 ~ 2024-09-30
> - 담당: 이영희
>
> [안건 2] 서버 마이그레이션
> - 현재 상태: AWS EC2 (t3.large)
> - 목표: AWS ECS Fargate 전환
> - 예상 비용 절감: 월 200만원
> - 담당: 박민수
>
> [결정사항]
> 1. Project Alpha 4월 1일 착수 확정
> 2. 서버 마이그레이션 POC 3월 말 완료
> 3. 다음 회의: 2024년 3월 22일
> """
>
> result = {}
>
> # 1. 메타데이터 추출
> date_m = re.search(r'일시:\s*(.+)', minutes)
> if date_m:
>     result['일시'] = date_m.group(1).strip()
>
> place_m = re.search(r'장소:\s*(.+)', minutes)
> if place_m:
>     result['장소'] = place_m.group(1).strip()
>
> # 참석자 파싱: "이름(직급)" 형태
> attendees_m = re.search(r'참석자:\s*(.+)', minutes)
> if attendees_m:
>     result['참석자'] = re.findall(
>         r'(\w+)\((\w+)\)',
>         attendees_m.group(1)
>     )
>     # [('김철수', '팀장'), ('이영희', '과장'), ...]
>
> # 2. 안건 블록 추출
> agenda_blocks = re.findall(
>     r'\[안건\s*(\d+)\]\s*(.+?)\n(.*?)(?=\[안건|\[결정|\Z)',
>     minutes,
>     re.DOTALL
> )
>
> result['안건'] = []
> for num, title, body in agenda_blocks:
>     agenda = {'번호': int(num), '제목': title.strip()}
>     # 각 안건의 상세 항목 추출
>     details = re.findall(r'-\s*(.+?):\s*(.+)', body)
>     for key, value in details:
>         agenda[key.strip()] = value.strip()
>     result['안건'].append(agenda)
>
> # 3. 결정사항 추출
> decisions_m = re.search(r'\[결정사항\]\s*\n(.*?)$', minutes, re.DOTALL)
> if decisions_m:
>     result['결정사항'] = re.findall(
>         r'\d+\.\s*(.+)',
>         decisions_m.group(1)
>     )
>
> # 결과 출력
> import json
> print(json.dumps(result, ensure_ascii=False, indent=2))
> ```
>
> **확장 생각**: 실무에서는 회의록 형식이 완전히 일관적이지 않을 수 있습니다. 누락된 필드, 다른 구분자, 들여쓰기 변화 등에 대응하려면 각 추출 단계에 오류 처리와 유연한 패턴(예: `\s*` 추가, 선택적 그룹)을 넣어야 합니다. 또한 추출 결과의 품질을 검증하는 단계(누락된 필드 확인 등)도 파이프라인에 포함하면 좋습니다.
> </details>

> **Problem 13.3**: **다국어 텍스트 정규화 파이프라인**을 구축하세요. 다음 텍스트에는 한국어, 영어, 숫자, 특수문자, 이모지가 혼합되어 있습니다. 아래 요구사항을 모두 만족하는 정규화 함수를 작성하세요.
>
> 요구사항:
> 1. HTML 태그 제거
> 2. 이모지 제거 (유니코드 범위 활용)
> 3. 한국어 반복 글자 최대 2개로 제한 (예: "ㅋㅋㅋㅋ" → "ㅋㅋ")
> 4. 영어 반복 글자 최대 2개로 제한 (예: "sooo" → "soo")
> 5. 연속 구두점 최대 1개로 제한
> 6. 전각 문자를 반각으로 변환 (예: "！" → "!", "？" → "?")
> 7. 연속 공백 정리
>
> ```python
> messy = '<p>ㅋㅋㅋㅋ sooo goood!!! 😂😂 이것은 정말 대단해요！！ 와우~~~   </p>'
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 각 요구사항을 독립적인 규칙으로 구현한 뒤 파이프라인으로 연결합니다.
>
> **풀이 전략**: 요구사항별 패턴 설계 → 순서 결정 → 테스트
>
> **상세 풀이**:
> ```python
> import re
>
> def normalize_multilingual(text):
>     rules = [
>         # 1. HTML 태그 제거
>         (r'<[^>]+>', ''),
>         # 2. 이모지 제거 (일반적인 이모지 유니코드 범위)
>         (r'[\U0001F600-\U0001F64F\U0001F300-\U0001F5FF'
>          r'\U0001F680-\U0001F6FF\U0001F1E0-\U0001F1FF'
>          r'\U00002702-\U000027B0\U0001F900-\U0001F9FF'
>          r'\U0000FE00-\U0000FE0F\U00002600-\U000026FF]+', ''),
>         # 3+4. 반복 글자 제한 (한국어+영어 통합)
>         (r'(.)\1{2,}', r'\1\1'),
>         # 5. 연속 구두점 제한
>         (r'([!?.,~])\1+', r'\1'),
>         # 6. 전각 → 반각 (주요 문장부호)
>         (r'！', '!'),
>         (r'？', '?'),
>         (r'，', ','),
>         (r'．', '.'),
>         (r'～', '~'),
>         # 7. 연속 공백 정리
>         (r'\s+', ' '),
>     ]
>
>     for pattern, replacement in rules:
>         text = re.sub(pattern, replacement, text)
>
>     return text.strip()
>
> messy = '<p>ㅋㅋㅋㅋ sooo goood!!! 😂😂 이것은 정말 대단해요！！ 와우~~~   </p>'
> print(normalize_multilingual(messy))
> # 'ㅋㅋ soo good! 이것은 정말 대단해요! 와우~'
> ```
>
> **확장 생각**: 이모지 유니코드 범위는 지속적으로 확장되므로, 하드코딩된 범위는 완벽하지 않습니다. Chapter 17에서 다룰 `regex` 모듈의 `\p{Emoji}` 유니코드 카테고리를 사용하면 더 정확한 이모지 매칭이 가능합니다. 또한 전각/반각 변환은 `unicodedata` 모듈의 `normalize()` 함수가 더 체계적인 해법을 제공합니다.
> </details>

---

# Part E: 마무리 (Closing Block)

---

## 다음 단계 안내 (What's Next)

이 Chapter에서는 정규표현식을 활용해 **텍스트를 정규화하고, 데이터를 추출하고, 형식을 변환하고, 구조화된 텍스트를 파싱하는** 핵심 기법들을 익혔습니다.

다음 **Chapter 14: 로그 분석과 파일 자동화**에서는 이러한 텍스트 처리 기법들을 **실제 파일과 파일 시스템 수준으로 확장**합니다. Apache/Nginx 서버 로그를 파싱하고, 여러 파일을 대상으로 일괄 검색과 치환을 수행하고, 파일명을 패턴에 따라 자동으로 변경하는 등 실무 자동화 시나리오를 다루게 됩니다.

---

## 추가 학습 자원 (Further Resources)

- **실습 도구**: [regex101.com](https://regex101.com) — Python 모드에서 이 Chapter의 모든 패턴을 실시간으로 테스트할 수 있습니다
- **공식 문서**: [Python re 모듈 — re.sub()](https://docs.python.org/3/library/re.html#re.sub) — `re.sub()`의 모든 옵션과 동작을 확인하세요
- **NLP 전처리**: 텍스트 정규화를 더 깊이 학습하고 싶다면, 자연어 처리(NLP) 분야의 텍스트 전처리 기법들을 살펴보세요. `nltk`, `spacy` 같은 라이브러리가 정규표현식과 함께 활용됩니다
- **데이터 정제 실무**: pandas의 `str.replace()`는 내부적으로 정규표현식을 지원합니다. 이 Chapter의 패턴들을 DataFrame의 컬럼에 바로 적용할 수 있습니다
