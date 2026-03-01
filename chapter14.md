# Chapter 14: 로그 분석과 파일 자동화

> **난이도**: ⭐⭐⭐ | **예상 학습 시간**: 3.5시간 | **선수 지식**: Chapter 13 (텍스트 처리와 데이터 정제)

---

# Part A: 도입부

## 학습 목표 (Learning Objectives)

이 Chapter를 완료하면, 당신은 다음을 할 수 있습니다:

- [ ] 서버 로그, 애플리케이션 로그 등 실제 로그 파일을 정규표현식으로 파싱하고 분석할 수 있다
- [ ] 여러 파일을 대상으로 정규표현식 기반 검색과 일괄 치환을 수행하는 스크립트를 작성할 수 있다
- [ ] `os`, `glob`, `pathlib` 등과 정규표현식을 결합하여 파일 시스템 자동화를 구현할 수 있다
- [ ] 정규표현식을 활용한 간단한 텍스트 처리 CLI 도구를 만들 수 있다

---

## 핵심 질문 (Essential Questions)

1. **서버에 쌓이는 수백만 줄의 로그에서 원하는 정보만 골라내려면 어떻게 해야 할까?** — 로그 파일은 일정한 형식을 따르지만, 그 안에서 필요한 데이터를 추출하려면 패턴을 정확히 파악해야 합니다.
2. **수백 개의 파일에서 동시에 특정 텍스트를 찾거나 바꾸려면?** — 하나하나 열어보는 대신, 정규표현식과 파일 시스템 도구를 결합하면 몇 초 만에 처리할 수 있습니다.
3. **반복적인 텍스트 처리 작업을 자동화하는 나만의 도구를 만들 수 있을까?** — `argparse`와 정규표현식을 결합하면, 터미널에서 바로 사용할 수 있는 유틸리티를 만들 수 있습니다.

---

## 개념 지도 (Concept Map)

```
[선수: 정규표현식 문법 전체 (Ch1~Ch4)]
[선수: re 모듈 함수, 매치 객체, 캡처 그룹, 치환/분할 (Ch5~Ch7)]
[선수: 명명 그룹, 전후방 탐색, 플래그, 유니코드 (Ch8~Ch11)]
[선수: 데이터 검증 패턴, 텍스트 정제/추출/변환 파이프라인 (Ch12~Ch13)]
    │
    ▼
[신규: 로그 포맷 이해] ──→ [신규: Apache/Nginx 로그 파싱]
         │                          │
         ▼                          ▼
[신규: 애플리케이션 로그 분석] ──→ [신규: 다중 파일 검색/치환]
                                        │
                                        ▼
                              [신규: 파일명 매칭/리네이밍]
                                        │
                                        ▼
                              [신규: CLI 유틸리티 작성]
    │
    ▼
[결과: 로그 분석 자동화, 파일 시스템 일괄 처리, 재사용 가능한 도구 구축]
```

---

# Part B: 본문 (Main Content)

---

## 14.1 로그 파일의 구조와 패턴

> 💡 **한 줄 요약**: 로그 파일은 일정한 형식을 따르는 텍스트이므로, 그 구조를 파악하면 정규표현식으로 원하는 정보를 정밀하게 추출할 수 있다.

### 직관적 이해

여러분이 건물의 경비실에서 일한다고 상상해 보세요. 출입 기록부에는 매번 같은 형식으로 기록이 남습니다 — *날짜, 시간, 이름, 출입 방향, 목적*. 이 기록부가 수천 페이지에 달한다면, "지난주에 야간에 출입한 사람은 누구인가?"라는 질문에 답하기 위해 한 줄씩 읽어볼 수는 없습니다. 하지만 기록의 *형식*을 알고 있다면, 해당 형식에 맞는 패턴을 만들어 한 번에 걸러낼 수 있습니다.

서버 로그 파일이 바로 이 출입 기록부입니다. 웹 서버, 애플리케이션, 운영체제 등 모든 소프트웨어는 자신의 동작 기록을 로그 파일에 남깁니다. 이 로그는 미리 정해진 형식(포맷)을 따르기 때문에, 정규표현식의 이상적인 처리 대상입니다.

### 핵심 개념 설명

#### 로그 파일이란?

> 📝 **로그 파일(Log File)**: 소프트웨어의 실행 과정에서 발생하는 이벤트(요청, 오류, 상태 변경 등)를 시간순으로 기록한 텍스트 파일. 각 줄(line)이 하나의 이벤트를 나타내는 것이 일반적이다.

로그 파일의 핵심 특징은 **반복적인 구조**입니다. 모든 줄이 동일한 형식을 따르므로, 하나의 정규표현식 패턴으로 모든 줄을 파싱할 수 있습니다.

#### 공통 로그 포맷 (Common Log Format, CLF)

웹 서버 로그의 가장 기본적인 형식은 **CLF(Common Log Format)**입니다:

```
127.0.0.1 - frank [10/Oct/2023:13:55:36 +0900] "GET /index.html HTTP/1.1" 200 2326
```

이 한 줄을 분해하면:

| 필드 | 값 | 의미 |
|------|-----|------|
| 원격 호스트 | `127.0.0.1` | 요청을 보낸 IP 주소 |
| 식별자 | `-` | RFC 1413 식별자 (보통 사용하지 않아 `-`) |
| 사용자 | `frank` | 인증된 사용자 이름 (없으면 `-`) |
| 타임스탬프 | `[10/Oct/2023:13:55:36 +0900]` | 요청 시각 |
| 요청 | `"GET /index.html HTTP/1.1"` | HTTP 메서드, 경로, 프로토콜 |
| 상태 코드 | `200` | HTTP 응답 코드 |
| 응답 크기 | `2326` | 응답 본문의 바이트 수 |

#### Combined Log Format

실무에서 더 자주 사용되는 것은 CLF를 확장한 **Combined Log Format**입니다. CLF 뒤에 **Referer**와 **User-Agent** 두 필드가 추가됩니다:

```
192.168.1.100 - - [10/Oct/2023:13:55:36 +0900] "GET /api/users HTTP/1.1" 200 1234 "https://example.com/dashboard" "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

#### 로그 타임스탬프의 형식

로그 파일에서 타임스탬프는 가장 중요한 필드 중 하나입니다. 로그 종류에 따라 다양한 형식이 사용됩니다:

```python
# Apache/Nginx 형식
# [DD/Mon/YYYY:HH:MM:SS ±HHMM]
"[10/Oct/2023:13:55:36 +0900]"

# syslog 형식
# Mon DD HH:MM:SS
"Oct 10 13:55:36"

# ISO 8601 형식
# YYYY-MM-DDTHH:MM:SS±HH:MM
"2023-10-10T13:55:36+09:00"

# 파이썬 logging 기본 형식
# YYYY-MM-DD HH:MM:SS,mmm
"2023-10-10 13:55:36,123"
```

각 형식에 맞는 패턴을 작성하는 것이 로그 분석의 첫걸음입니다.

#### 로그 레벨

애플리케이션 로그에는 보통 **로그 레벨(Log Level)**이 포함되어 메시지의 심각도를 나타냅니다:

| 레벨 | 의미 |
|------|------|
| `DEBUG` | 디버깅 목적의 상세 정보 |
| `INFO` | 일반적인 운영 정보 |
| `WARNING` | 잠재적 문제 경고 |
| `ERROR` | 오류 발생 |
| `CRITICAL` | 심각한 오류, 시스템 중단 가능 |

```python
import re

# 로그 레벨을 추출하는 기본 패턴
log_line = "2023-10-10 13:55:36,123 ERROR [main] Connection refused: db-server:5432"
match = re.search(r'\b(DEBUG|INFO|WARNING|ERROR|CRITICAL)\b', log_line)
if match:
    print(f"로그 레벨: {match.group(1)}")  # 로그 레벨: ERROR
```

> 🔗 **연결**: `\b`(단어 경계)를 사용하여 로그 레벨 단어만 정확히 매칭합니다 (Chapter 3의 단어 경계 참고). 선택 연산자 `|`로 여러 레벨을 하나의 패턴으로 처리합니다 (Chapter 2 참고).

### 상세 예시

> 🔍 **예시 14.1.1: CLF 로그의 기본 파싱**
>
> **상황**: Apache 웹 서버의 CLF 로그에서 IP 주소와 상태 코드를 추출하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import re
>
> log_line = '192.168.1.100 - frank [10/Oct/2023:13:55:36 +0900] "GET /index.html HTTP/1.1" 200 2326'
>
> # 단계적 접근: 먼저 IP와 상태 코드만 추출
> pattern = r'^(\d{1,3}(?:\.\d{1,3}){3}) .+?" (\d{3}) '
> match = re.search(pattern, log_line)
> if match:
>     print(f"IP: {match.group(1)}")       # IP: 192.168.1.100
>     print(f"상태 코드: {match.group(2)}")  # 상태 코드: 200
> ```
>
> **핵심 포인트**: 로그의 전체 구조를 한 번에 파싱하려 하지 말고, 필요한 필드만 먼저 추출하는 것이 실전에서 효과적입니다. `(?:...)`는 비캡처 그룹으로, IP 주소의 반복 부분을 그룹핑하되 캡처하지는 않습니다.

> 🔗 **연결**: 비캡처 그룹 `(?:...)`은 Chapter 8에서 배웠습니다.

> 🔍 **예시 14.1.2: 타임스탬프 패턴 작성**
>
> **상황**: 다양한 로그 형식의 타임스탬프를 인식하는 패턴을 만들어야 합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> timestamps = [
>     "[10/Oct/2023:13:55:36 +0900]",           # Apache
>     "2023-10-10T13:55:36+09:00",               # ISO 8601
>     "2023-10-10 13:55:36,123",                 # Python logging
>     "Oct 10 13:55:36",                         # syslog
> ]
>
> patterns = {
>     'apache':  r'\[\d{2}/\w{3}/\d{4}:\d{2}:\d{2}:\d{2} [+\-]\d{4}\]',
>     'iso8601': r'\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}[+\-]\d{2}:\d{2}',
>     'python':  r'\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2},\d{3}',
>     'syslog':  r'\w{3} [ \d]\d \d{2}:\d{2}:\d{2}',
> }
>
> for ts in timestamps:
>     for name, pat in patterns.items():
>         if re.search(pat, ts):
>             print(f"'{ts}' → {name} 형식")
>             break
> ```
> **출력**:
> ```
> '[10/Oct/2023:13:55:36 +0900]' → apache 형식
> '2023-10-10T13:55:36+09:00' → iso8601 형식
> '2023-10-10 13:55:36,123' → python 형식
> 'Oct 10 13:55:36' → syslog 형식
> ```
>
> **핵심 포인트**: 각 로그 형식의 타임스탬프에는 고유한 구조적 특징이 있습니다. Apache는 대괄호와 `/` 구분자, ISO 8601은 `T` 구분자, Python logging은 밀리초의 `,` 구분이 특징적입니다.

> ⚠️ **주의**: 로그 파일의 형식은 서버 설정에 따라 달라질 수 있습니다. 실제 로그 파일의 처음 몇 줄을 직접 확인한 후 패턴을 작성하는 습관을 들이세요. "아마 이 형식일 거야"라고 가정하면 예상치 못한 필드나 구분자 때문에 파싱이 실패할 수 있습니다.

### Section 요약

- 로그 파일은 **반복적인 구조**를 가진 텍스트로, 정규표현식 처리에 이상적인 대상이다
- **CLF**(Common Log Format)와 **Combined Log Format**은 웹 서버 로그의 표준 형식이다
- 타임스탬프는 로그 종류에 따라 Apache, ISO 8601, syslog, Python logging 등 다양한 형식이 존재한다
- **로그 레벨**(DEBUG, INFO, WARNING, ERROR, CRITICAL)은 애플리케이션 로그의 핵심 필드이다
- 로그 파싱의 첫 단계는 실제 로그 파일의 형식을 직접 확인하는 것이다

---

## 14.2 Apache/Nginx 로그 분석

> 💡 **한 줄 요약**: 웹 서버 로그를 명명 그룹으로 구조적으로 파싱하면, IP별 접속 통계, 상태 코드 분석, 트래픽 패턴 등 실무에서 가치 있는 정보를 추출할 수 있다.

### 직관적 이해

웹 서버 로그는 "누가, 언제, 무엇을, 어떻게 요청했고, 결과는 어땠는가"를 한 줄에 담고 있습니다. 이것은 마치 상점의 판매 기록과 같습니다 — 고객(IP), 방문 시간(타임스탬프), 무엇을 봤는지(요청 경로), 구매가 성사되었는지(상태 코드)가 모두 기록되어 있죠. 이 기록을 분석하면 "어떤 페이지가 가장 인기 있는가?", "오류가 자주 발생하는 시간대는?", "특정 IP에서 비정상적인 접근이 있는가?" 등의 질문에 답할 수 있습니다.

### 핵심 개념 설명

#### Combined Log Format의 전체 파싱 패턴

Combined Log Format의 모든 필드를 명명 그룹으로 추출하는 패턴을 단계적으로 구성해 봅시다:

> 🔗 **연결**: 명명 그룹 `(?P<name>...)`은 Chapter 8에서 배웠습니다. 이것을 사용하면 결과를 이름으로 참조할 수 있어 코드 가독성이 크게 높아집니다.

```python
import re

# re.VERBOSE를 사용하여 가독성 높은 패턴 작성
clf_pattern = re.compile(r'''
    (?P<ip>\d{1,3}(?:\.\d{1,3}){3})   # IP 주소
    \s
    (?P<ident>\S+)                      # 식별자 (보통 -)
    \s
    (?P<user>\S+)                       # 사용자 (보통 -)
    \s
    \[(?P<timestamp>[^\]]+)\]           # 타임스탬프 [...]
    \s
    "(?P<request>[^"]*)"                # 요청 "..."
    \s
    (?P<status>\d{3})                   # 상태 코드
    \s
    (?P<size>\d+|-)                     # 응답 크기 (없으면 -)
    (?:                                 # Combined 확장 필드 (선택적)
        \s
        "(?P<referer>[^"]*)"            # Referer
        \s
        "(?P<user_agent>[^"]*)"         # User-Agent
    )?
''', re.VERBOSE)
```

> 🔗 **연결**: `re.VERBOSE`(Chapter 10)를 사용하면 복잡한 패턴에 주석과 공백을 추가하여 가독성을 높일 수 있습니다. 이처럼 긴 패턴에서는 사실상 필수입니다.

이 패턴의 핵심 기법을 분해하면:

1. **IP 주소**: `\d{1,3}(?:\.\d{1,3}){3}` — 1~3자리 숫자와 점의 반복
2. **타임스탬프**: `\[([^\]]+)\]` — 대괄호 안의 모든 문자 (대괄호 제외)
3. **요청**: `"([^"]*)"` — 쌍따옴표 안의 모든 문자 (쌍따옴표 제외)
4. **선택적 필드**: `(?:...)?` — Combined 필드가 없는 CLF 로그도 처리 가능

#### 요청 필드의 추가 분해

요청 필드(`"GET /index.html HTTP/1.1"`)는 다시 메서드, 경로, 프로토콜로 분해할 수 있습니다:

```python
def parse_request(request_str):
    """요청 문자열을 메서드, 경로, 프로토콜로 분해"""
    match = re.match(r'(?P<method>\w+)\s+(?P<path>\S+)(?:\s+(?P<protocol>\S+))?', request_str)
    if match:
        return match.groupdict()
    return None

# 사용 예
result = parse_request("GET /api/users?page=2 HTTP/1.1")
print(result)
# {'method': 'GET', 'path': '/api/users?page=2', 'protocol': 'HTTP/1.1'}
```

#### 실전: 로그 파일 분석 스크립트

이제 전체 로그 파일을 읽어서 유용한 통계를 생성하는 스크립트를 작성해 봅시다:

```python
import re
from collections import Counter

def analyze_access_log(filepath):
    """웹 서버 접속 로그를 분석하여 주요 통계를 반환"""

    clf_pattern = re.compile(r'''
        (?P<ip>\d{1,3}(?:\.\d{1,3}){3})
        \s\S+\s\S+\s
        \[(?P<timestamp>[^\]]+)\]
        \s
        "(?P<method>\w+)\s+(?P<path>\S+)\s*[^"]*"
        \s
        (?P<status>\d{3})
        \s
        (?P<size>\d+|-)
    ''', re.VERBOSE)

    ip_counter = Counter()
    status_counter = Counter()
    path_counter = Counter()
    error_lines = []
    total_lines = 0
    parsed_lines = 0

    with open(filepath, 'r', encoding='utf-8') as f:
        for line in f:
            total_lines += 1
            match = clf_pattern.search(line)
            if match:
                parsed_lines += 1
                data = match.groupdict()

                ip_counter[data['ip']] += 1
                status_counter[data['status']] += 1
                path_counter[data['path']] += 1

                # 4xx, 5xx 에러 수집
                if data['status'].startswith(('4', '5')):
                    error_lines.append(line.strip())

    # 결과 출력
    print(f"=== 로그 분석 결과 ===")
    print(f"전체 줄: {total_lines}, 파싱 성공: {parsed_lines}")
    print(f"\n--- 상위 5개 IP ---")
    for ip, count in ip_counter.most_common(5):
        print(f"  {ip}: {count}회")

    print(f"\n--- 상태 코드 분포 ---")
    for status, count in sorted(status_counter.items()):
        print(f"  {status}: {count}회")

    print(f"\n--- 상위 5개 요청 경로 ---")
    for path, count in path_counter.most_common(5):
        print(f"  {path}: {count}회")

    print(f"\n--- 에러 로그 (최근 5건) ---")
    for err in error_lines[-5:]:
        print(f"  {err[:100]}...")

    return {
        'total': total_lines,
        'parsed': parsed_lines,
        'top_ips': ip_counter.most_common(10),
        'status_codes': dict(status_counter),
        'top_paths': path_counter.most_common(10),
        'errors': error_lines,
    }
```

> 🔗 **연결**: `groupdict()`는 Chapter 8에서 배운 명명 그룹의 결과를 딕셔너리로 반환하는 메서드입니다. `Counter`는 파이썬 표준 라이브러리의 `collections` 모듈에서 제공하는 도구로, 요소의 등장 횟수를 자동으로 세어줍니다.

### 상세 예시

> 🔍 **예시 14.2.1: 특정 시간대의 접속 패턴 분석**
>
> **상황**: "오후 2시~4시 사이에 404 오류가 급증했다"는 보고를 받았습니다. 해당 시간대의 404 오류 로그만 추출해야 합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> sample_logs = """192.168.1.10 - - [10/Oct/2023:14:23:15 +0900] "GET /old-page HTTP/1.1" 404 0
> 192.168.1.20 - - [10/Oct/2023:14:45:30 +0900] "GET /api/data HTTP/1.1" 200 1234
> 192.168.1.30 - - [10/Oct/2023:15:10:22 +0900] "GET /missing HTTP/1.1" 404 0
> 192.168.1.10 - - [10/Oct/2023:15:55:01 +0900] "GET /deleted HTTP/1.1" 404 0
> 192.168.1.40 - - [10/Oct/2023:16:05:00 +0900] "GET /home HTTP/1.1" 200 5678
> 10.0.0.1 - - [10/Oct/2023:13:30:00 +0900] "GET /old HTTP/1.1" 404 0"""
>
> # 시간이 14~15시이고, 상태 코드가 404인 줄 추출
> pattern = re.compile(
>     r'(?P<ip>\S+) .+ \[\d{2}/\w{3}/\d{4}:(?P<hour>1[45]):\d{2}:\d{2} [+\-]\d{4}\]'
>     r' "(?P<method>\w+) (?P<path>\S+) [^"]*" 404'
> )
>
> print("=== 14:00~15:59 사이 404 오류 ===")
> for line in sample_logs.strip().split('\n'):
>     match = pattern.search(line)
>     if match:
>         d = match.groupdict()
>         print(f"  [{d['hour']}시] {d['ip']} → {d['path']}")
> ```
> **출력**:
> ```
> === 14:00~15:59 사이 404 오류 ===
>   [14시] 192.168.1.10 → /old-page
>   [15시] 192.168.1.30 → /missing
>   [15시] 192.168.1.10 → /deleted
> ```
>
> **핵심 포인트**: 시간 필터링은 타임스탬프의 시간 부분만 캡처하여 조건을 적용합니다. `1[45]`는 14 또는 15를 매칭하는 간결한 패턴입니다.

> 🔍 **예시 14.2.2: User-Agent 기반 봇 트래픽 식별**
>
> **상황**: 서버 트래픽 중 검색 엔진 봇의 비율을 파악하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import re
> from collections import Counter
>
> sample_logs = [
>     '10.0.0.1 - - [10/Oct/2023:10:00:00 +0900] "GET / HTTP/1.1" 200 5000 "-" "Mozilla/5.0 (compatible; Googlebot/2.1)"',
>     '10.0.0.2 - - [10/Oct/2023:10:00:01 +0900] "GET /page HTTP/1.1" 200 3000 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/118.0"',
>     '10.0.0.3 - - [10/Oct/2023:10:00:02 +0900] "GET /robots.txt HTTP/1.1" 200 100 "-" "Mozilla/5.0 (compatible; bingbot/2.0)"',
>     '10.0.0.4 - - [10/Oct/2023:10:00:03 +0900] "GET /api HTTP/1.1" 200 200 "-" "python-requests/2.28.0"',
>     '10.0.0.5 - - [10/Oct/2023:10:00:04 +0900] "GET / HTTP/1.1" 200 5000 "-" "Mozilla/5.0 (iPhone; CPU iPhone OS 16_0)"',
> ]
>
> # User-Agent에서 봇 키워드를 찾는 패턴
> bot_pattern = re.compile(r'(?i)(googlebot|bingbot|yandexbot|slurp|duckduckbot|baidu)')
> ua_pattern = re.compile(r'"([^"]*)"$')  # 줄 끝의 마지막 따옴표 문자열 = User-Agent
>
> categories = Counter()
>
> for line in sample_logs:
>     ua_match = ua_pattern.search(line)
>     if ua_match:
>         user_agent = ua_match.group(1)
>         if bot_pattern.search(user_agent):
>             categories['봇'] += 1
>         else:
>             categories['사람/기타'] += 1
>
> total = sum(categories.values())
> print("=== 트래픽 분류 ===")
> for cat, count in categories.items():
>     print(f"  {cat}: {count}건 ({count/total*100:.1f}%)")
> ```
> **출력**:
> ```
> === 트래픽 분류 ===
>   봇: 2건 (40.0%)
>   사람/기타: 3건 (60.0%)
> ```
>
> **핵심 포인트**: `(?i)` 인라인 플래그로 대소문자 무시 매칭을 적용하여 다양한 봇 이름 변형을 처리합니다. 로그의 마지막 따옴표 쌍이 User-Agent라는 Combined Format의 구조를 활용했습니다.

> 🔗 **연결**: 인라인 플래그 `(?i)`는 Chapter 10에서 배웠습니다.

> ⚠️ **주의**: 실제 환경에서는 로그 형식이 표준과 약간 다를 수 있습니다. 예를 들어 Nginx의 커스텀 로그 포맷, 로드 밸런서가 추가하는 `X-Forwarded-For` 헤더 등이 있을 수 있습니다. 항상 **실제 로그 샘플**을 먼저 확인하고 패턴을 조정하세요.

### Section 요약

- Combined Log Format의 전체 필드를 **명명 그룹**과 **`re.VERBOSE`**를 활용하여 구조적으로 파싱할 수 있다
- 요청 필드를 추가 분해하여 HTTP 메서드, 경로, 프로토콜을 개별적으로 추출할 수 있다
- `Counter`와 결합하면 IP별 접속 횟수, 상태 코드 분포, 인기 페이지 등의 **통계를 자동 생성**할 수 있다
- 시간대 필터링, 봇 트래픽 식별 등 **조건부 분석**은 패턴에 필터 조건을 직접 포함시켜 구현한다

---

## 14.3 애플리케이션 로그 분석

> 💡 **한 줄 요약**: 파이썬 로그, 에러 트레이스백, JSON 로그 등 애플리케이션 수준의 로그는 웹 서버 로그보다 형식이 다양하지만, 정규표현식으로 체계적으로 파싱할 수 있다.

### 직관적 이해

웹 서버 로그가 "건물 출입 기록"이라면, 애플리케이션 로그는 "건물 내 CCTV 기록"에 가깝습니다. 누가 들어왔는지뿐 아니라, 내부에서 무슨 일이 있었는지를 상세히 기록합니다 — 어떤 함수가 호출되었고, 어디서 오류가 발생했으며, 데이터베이스 연결이 끊어진 시점은 언제인지 등.

애플리케이션 로그는 웹 서버 로그보다 형식이 다양하고, 때로는 한 이벤트가 여러 줄에 걸쳐 기록되기도 합니다(예: 파이썬의 에러 트레이스백). 이러한 다양성에 대응하는 유연한 패턴 설계가 필요합니다.

### 핵심 개념 설명

#### 파이썬 logging 모듈의 기본 로그 형식

파이썬 `logging` 모듈의 기본 출력 형식은 다음과 같습니다:

```
2023-10-10 13:55:36,123 WARNING [module_name] This is a warning message
2023-10-10 13:55:37,456 ERROR [db_handler] Connection timeout after 30s
```

이 형식을 파싱하는 패턴:

```python
import re

python_log_pattern = re.compile(r'''
    (?P<timestamp>\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2},\d{3})
    \s
    (?P<level>DEBUG|INFO|WARNING|ERROR|CRITICAL)
    \s
    \[(?P<module>[^\]]+)\]
    \s
    (?P<message>.+)
''', re.VERBOSE)

log_lines = [
    "2023-10-10 13:55:36,123 WARNING [auth] Failed login attempt for user: admin",
    "2023-10-10 13:55:37,456 ERROR [db_handler] Connection timeout after 30s",
    "2023-10-10 13:55:38,789 INFO [scheduler] Daily backup completed successfully",
]

for line in log_lines:
    match = python_log_pattern.match(line)
    if match:
        d = match.groupdict()
        print(f"[{d['level']:8s}] {d['module']:12s} | {d['message']}")
```

**출력**:
```
[WARNING ] auth         | Failed login attempt for user: admin
[ERROR   ] db_handler   | Connection timeout after 30s
[INFO    ] scheduler    | Daily backup completed successfully
```

#### 에러 트레이스백 파싱

파이썬 에러 트레이스백은 여러 줄에 걸쳐 기록되므로 특별한 처리가 필요합니다:

```
Traceback (most recent call last):
  File "/app/handlers/user.py", line 42, in get_user
    user = db.query(User).filter_by(id=user_id).one()
  File "/app/lib/database.py", line 128, in query
    return session.execute(stmt)
sqlalchemy.exc.OperationalError: connection refused
```

> 🔗 **연결**: 여러 줄에 걸친 패턴을 매칭하려면 `re.DOTALL`(Chapter 10)을 사용하여 `.`이 줄바꿈 문자도 매칭하도록 해야 합니다.

```python
import re

log_text = """2023-10-10 14:00:01,234 INFO [main] Server starting
2023-10-10 14:00:05,567 ERROR [handler] Unhandled exception:
Traceback (most recent call last):
  File "/app/handlers/user.py", line 42, in get_user
    user = db.query(User).filter_by(id=user_id).one()
  File "/app/lib/database.py", line 128, in query
    return session.execute(stmt)
sqlalchemy.exc.OperationalError: connection refused
2023-10-10 14:00:06,789 INFO [main] Retrying connection"""

# 트레이스백 블록 추출
traceback_pattern = re.compile(
    r'Traceback \(most recent call last\):\n'
    r'((?:  .+\n)+)'       # 들여쓰기된 줄들 (파일/코드)
    r'(\S+Error\S*: .+)',   # 예외 클래스: 메시지
    re.MULTILINE
)

for match in traceback_pattern.finditer(log_text):
    frames = match.group(1)
    error = match.group(2)
    print(f"에러: {error}")

    # 각 프레임에서 파일명과 줄 번호 추출
    frame_pattern = re.compile(
        r'File "(?P<file>[^"]+)", line (?P<line>\d+), in (?P<func>\w+)'
    )
    for frame in frame_pattern.finditer(frames):
        d = frame.groupdict()
        print(f"  → {d['file']}:{d['line']} ({d['func']})")
```

**출력**:
```
에러: sqlalchemy.exc.OperationalError: connection refused
  → /app/handlers/user.py:42 (get_user)
  → /app/lib/database.py:128 (query)
```

#### JSON 로그 파싱

최근의 많은 애플리케이션은 구조화된 JSON 형식으로 로그를 출력합니다:

```
{"timestamp": "2023-10-10T14:00:01Z", "level": "ERROR", "service": "auth", "message": "Login failed", "user_id": 12345}
```

JSON 로그는 `json` 모듈로 직접 파싱하는 것이 가장 정확하지만, 정규표현식이 유용한 경우가 있습니다 — 예를 들어 특정 필드만 빠르게 필터링하거나, JSON 형식이 깨진 줄을 사전 필터링할 때입니다:

```python
import re
import json

json_logs = [
    '{"timestamp": "2023-10-10T14:00:01Z", "level": "ERROR", "message": "DB connection failed"}',
    '{"timestamp": "2023-10-10T14:00:02Z", "level": "INFO", "message": "User logged in"}',
    'This line is not valid JSON - system restarted',
    '{"timestamp": "2023-10-10T14:00:03Z", "level": "ERROR", "message": "Timeout"}',
]

# 방법 1: 정규표현식으로 ERROR 로그만 사전 필터링
error_filter = re.compile(r'"level"\s*:\s*"ERROR"')

print("=== ERROR 로그 (정규표현식 필터 후 JSON 파싱) ===")
for line in json_logs:
    if error_filter.search(line):
        try:
            data = json.loads(line)
            print(f"  [{data['timestamp']}] {data['message']}")
        except json.JSONDecodeError:
            print(f"  [파싱 실패] {line[:50]}...")
```

> 💬 **참고**: JSON 로그가 표준 형식을 완벽히 따른다면, 정규표현식 대신 `json.loads()`만으로 파싱하는 것이 더 안전합니다. 정규표현식은 **사전 필터링**(전체 파싱 전에 원하는 줄만 골라내기)이나 **비표준 형식 처리**에 활용하는 것이 좋습니다.

### 상세 예시

> 🔍 **예시 14.3.1: 에러 빈도 분석 스크립트**
>
> **상황**: 애플리케이션 로그에서 시간대별 에러 빈도를 분석하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import re
> from collections import Counter
>
> sample_log = """2023-10-10 08:15:23,100 INFO [app] Request processed
> 2023-10-10 08:30:45,200 ERROR [db] Query timeout
> 2023-10-10 09:12:33,300 ERROR [auth] Invalid token
> 2023-10-10 09:45:12,400 WARNING [app] Slow response: 3.2s
> 2023-10-10 09:50:00,500 ERROR [db] Connection pool exhausted
> 2023-10-10 10:05:22,600 INFO [app] Health check OK
> 2023-10-10 10:15:33,700 ERROR [api] Rate limit exceeded
> 2023-10-10 10:20:11,800 ERROR [db] Query timeout"""
>
> # 에러 로그에서 시간(hour)과 모듈을 추출
> error_pattern = re.compile(
>     r'(?P<date>\d{4}-\d{2}-\d{2})\s(?P<hour>\d{2}):\d{2}:\d{2},\d{3}\s'
>     r'ERROR\s\[(?P<module>\w+)\]\s(?P<message>.+)'
> )
>
> hourly_errors = Counter()
> module_errors = Counter()
> error_messages = Counter()
>
> for line in sample_log.strip().split('\n'):
>     match = error_pattern.search(line)
>     if match:
>         d = match.groupdict()
>         hourly_errors[f"{d['hour']}시"] += 1
>         module_errors[d['module']] += 1
>         error_messages[d['message']] += 1
>
> print("시간대별 에러:")
> for hour, count in sorted(hourly_errors.items()):
>     bar = '█' * count
>     print(f"  {hour}: {bar} ({count})")
>
> print("\n모듈별 에러:")
> for module, count in module_errors.most_common():
>     print(f"  {module}: {count}건")
>
> print("\n에러 메시지 빈도:")
> for msg, count in error_messages.most_common():
>     print(f"  '{msg}': {count}회")
> ```
> **출력**:
> ```
> 시간대별 에러:
>   08시: █ (1)
>   09시: ██ (2)
>   10시: ██ (2)
>
> 모듈별 에러:
>   db: 3건
>   auth: 1건
>   api: 1건
>
> 에러 메시지 빈도:
>   'Query timeout': 2회
>   'Invalid token': 1회
>   'Connection pool exhausted': 1회
>   'Rate limit exceeded': 1회
> ```
>
> **핵심 포인트**: 패턴에 `ERROR`를 리터럴로 포함시켜 에러 줄만 선별적으로 매칭합니다. 한 번의 패턴 매칭으로 시간, 모듈, 메시지를 동시에 추출하여 여러 차원의 통계를 생성할 수 있습니다.

> 🔍 **예시 14.3.2: 멀티라인 스택 트레이스 수집**
>
> **상황**: 로그 파일에서 모든 스택 트레이스를 추출하고, 어떤 예외가 가장 자주 발생하는지 파악해야 합니다.
>
> **풀이/설명**:
> ```python
> import re
> from collections import Counter
>
> log_text = """2023-10-10 14:00:01 INFO Server started
> 2023-10-10 14:00:05 ERROR Unhandled exception:
> Traceback (most recent call last):
>   File "app.py", line 10, in handle
>     result = process(data)
> ValueError: invalid literal for int()
> 2023-10-10 14:00:10 INFO Request processed
> 2023-10-10 14:00:15 ERROR Unhandled exception:
> Traceback (most recent call last):
>   File "app.py", line 25, in connect
>     db.open()
> ConnectionError: refused
> 2023-10-10 14:00:20 ERROR Unhandled exception:
> Traceback (most recent call last):
>   File "app.py", line 10, in handle
>     result = process(data)
> ValueError: invalid literal for int()
> 2023-10-10 14:00:25 INFO Shutdown"""
>
> # 예외 클래스만 빠르게 추출하는 패턴
> # Traceback 블록 뒤의 예외 줄: 들여쓰기 없이 시작하는 XxxError: 형태
> exception_pattern = re.compile(
>     r'^(\w+(?:\.\w+)*(?:Error|Exception|Warning)): (.+)',
>     re.MULTILINE
> )
>
> exceptions = Counter()
> for match in exception_pattern.finditer(log_text):
>     exc_class = match.group(1)
>     exc_msg = match.group(2)
>     exceptions[f"{exc_class}: {exc_msg}"] += 1
>
> print("=== 예외 빈도 ===")
> for exc, count in exceptions.most_common():
>     print(f"  {exc} — {count}회")
> ```
> **출력**:
> ```
> === 예외 빈도 ===
>   ValueError: invalid literal for int() — 2회
>   ConnectionError: refused — 1회
> ```
>
> **핵심 포인트**: `re.MULTILINE`과 `^` 앵커를 사용하여 각 줄의 시작에서 예외 패턴을 찾습니다. `\w+(?:\.\w+)*`는 `sqlalchemy.exc.OperationalError` 같은 점으로 구분된 모듈 경로도 매칭합니다.

> 🔗 **연결**: `re.MULTILINE` 모드에서 `^`는 문자열 전체의 시작이 아니라 각 줄의 시작을 매칭합니다 (Chapter 10 참고).

> ⚠️ **주의**: 에러 트레이스백은 사용자의 입력 데이터를 포함할 수 있으므로, 로그를 외부에 공유할 때는 개인정보가 포함되지 않았는지 확인해야 합니다. 정규표현식을 활용한 민감 정보 마스킹도 좋은 습관입니다:
> ```python
> # 이메일 주소 마스킹
> masked = re.sub(r'[\w.+-]+@[\w-]+\.[\w.]+', '[EMAIL]', log_text)
> # IP 주소 마스킹
> masked = re.sub(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}', '[IP]', masked)
> ```

### Section 요약

- 파이썬 `logging` 모듈의 로그는 타임스탬프, 레벨, 모듈, 메시지의 구조를 가진다
- **에러 트레이스백**은 여러 줄에 걸친 패턴이므로 `re.DOTALL`이나 `re.MULTILINE`을 적절히 활용해야 한다
- **JSON 로그**는 `json` 모듈과 정규표현식을 조합하여 처리하는 것이 효과적이다
- 로그 분석의 핵심은 **필터링 → 추출 → 집계**의 파이프라인이다

---

## 14.4 다중 파일 검색과 일괄 치환

> 💡 **한 줄 요약**: `os`, `glob`, `pathlib` 등 파이썬의 파일 시스템 도구와 정규표현식을 결합하면, 수많은 파일에서 동시에 패턴을 검색하고 일괄 치환하는 강력한 자동화를 구현할 수 있다.

### 직관적 이해

프로젝트 디렉토리에 수백 개의 파이썬 파일이 있고, 모든 파일에서 `print()` 문을 `logger.debug()`로 바꿔야 하는 상황을 상상해 보세요. 하나하나 열어서 수작업으로 바꾸는 것은 비현실적입니다. IDE의 "Find & Replace" 기능을 사용할 수도 있지만, "변수에 할당된 `print`는 건너뛰고, 함수 호출인 `print(...)`만 바꾸기"처럼 복잡한 조건이 필요하다면, 정규표현식 + 파일 순회 스크립트가 가장 유연한 해결책입니다.

### 핵심 개념 설명

#### 파일 시스템 탐색 도구

파이썬에서 디렉토리를 탐색하는 주요 방법 세 가지를 알아봅시다:

```python
import os
import glob
from pathlib import Path

# 1. os.walk() — 재귀적으로 모든 디렉토리 순회
for root, dirs, files in os.walk('/project/src'):
    for filename in files:
        if filename.endswith('.py'):
            filepath = os.path.join(root, filename)
            print(filepath)

# 2. glob.glob() — 와일드카드 패턴으로 파일 찾기
for filepath in glob.glob('/project/src/**/*.py', recursive=True):
    print(filepath)

# 3. pathlib.Path — 객체 지향적 파일 경로 처리
for filepath in Path('/project/src').rglob('*.py'):
    print(filepath)
```

> 💬 **참고**: `glob`의 `*`와 `**`는 정규표현식이 아니라 **글로브 패턴(glob pattern)**이라는 별도의 와일드카드 문법입니다. `*`는 임의의 문자열, `**`는 재귀적 디렉토리 탐색을 의미합니다. 파일명에 대해 정규표현식을 적용하려면 `glob`으로 파일을 먼저 수집한 뒤 `re`로 필터링하면 됩니다.

#### 다중 파일 검색 함수

여러 파일에서 정규표현식 패턴을 검색하는 범용 함수:

```python
import re
from pathlib import Path

def search_files(directory, file_pattern, regex_pattern, encoding='utf-8'):
    """
    디렉토리 내 파일에서 정규표현식 패턴을 검색

    Args:
        directory: 검색할 디렉토리 경로
        file_pattern: 파일 필터 (예: '*.py', '*.txt')
        regex_pattern: 검색할 정규표현식 패턴 (문자열 또는 컴파일된 패턴)
        encoding: 파일 인코딩

    Yields:
        (파일경로, 줄번호, 줄내용, 매치객체) 튜플
    """
    if isinstance(regex_pattern, str):
        regex_pattern = re.compile(regex_pattern)

    for filepath in Path(directory).rglob(file_pattern):
        try:
            with open(filepath, 'r', encoding=encoding) as f:
                for line_num, line in enumerate(f, 1):
                    match = regex_pattern.search(line)
                    if match:
                        yield (filepath, line_num, line.rstrip(), match)
        except (UnicodeDecodeError, PermissionError) as e:
            print(f"  [경고] {filepath}: {e}")

# 사용 예: 모든 Python 파일에서 TODO 주석 찾기
for filepath, line_num, line, match in search_files('.', '*.py', r'#\s*TODO\b.*'):
    print(f"{filepath}:{line_num}: {line}")
```

#### 다중 파일 일괄 치환 함수

```python
import re
from pathlib import Path

def replace_in_files(directory, file_pattern, search_pattern, replacement,
                     encoding='utf-8', dry_run=True):
    """
    여러 파일에서 정규표현식 기반 일괄 치환

    Args:
        directory: 대상 디렉토리
        file_pattern: 파일 필터
        search_pattern: 검색 패턴
        replacement: 치환 문자열 (역참조 가능)
        encoding: 파일 인코딩
        dry_run: True이면 실제 파일을 수정하지 않고 변경 예정 내용만 출력

    Returns:
        변경된 파일 수
    """
    if isinstance(search_pattern, str):
        search_pattern = re.compile(search_pattern)

    modified_count = 0

    for filepath in Path(directory).rglob(file_pattern):
        try:
            with open(filepath, 'r', encoding=encoding) as f:
                original = f.read()

            new_content, count = search_pattern.subn(replacement, original)

            if count > 0:
                modified_count += 1
                print(f"\n📄 {filepath} ({count}건 변경)")

                if dry_run:
                    # 변경 내용 미리보기
                    for line_num, (old_line, new_line) in enumerate(
                        zip(original.splitlines(), new_content.splitlines()), 1
                    ):
                        if old_line != new_line:
                            print(f"  줄 {line_num}:")
                            print(f"    - {old_line.strip()}")
                            print(f"    + {new_line.strip()}")
                else:
                    with open(filepath, 'w', encoding=encoding) as f:
                        f.write(new_content)
                    print(f"  ✅ 저장 완료")

        except (UnicodeDecodeError, PermissionError) as e:
            print(f"  [경고] {filepath}: {e}")

    mode = "미리보기" if dry_run else "실제 적용"
    print(f"\n=== {mode}: 총 {modified_count}개 파일 변경 ===")
    return modified_count
```

> ⚠️ **주의**: 파일 일괄 치환은 **반드시 `dry_run=True`로 먼저 확인**한 후, 문제가 없으면 `dry_run=False`로 실제 적용하세요. 그리고 작업 전에 반드시 **버전 관리(git 등)로 백업**하거나 별도의 복사본을 만들어 두세요. 잘못된 치환은 프로젝트 전체를 망가뜨릴 수 있습니다.

> 🔗 **연결**: `re.subn()`은 Chapter 7에서 배운 함수로, 치환된 결과 문자열과 치환 횟수를 함께 반환합니다.

### 상세 예시

> 🔍 **예시 14.4.1: 프로젝트 전체에서 deprecated 함수 호출 찾기**
>
> **상황**: `get_data()` 함수가 `fetch_data()`로 이름이 바뀌었습니다. 프로젝트 내 모든 `.py` 파일에서 `get_data(` 호출을 찾아야 합니다.
>
> **풀이/설명**:
> ```python
> import re
> from pathlib import Path
>
> # 함수 호출 패턴: get_data( 로 시작하되, 정의(def get_data)는 제외
> pattern = re.compile(r'(?<!def )(?<!def  )\bget_data\s*\(')
>
> # 예시용 가상 파일 내용
> files_content = {
>     'main.py': [
>         'from utils import get_data',
>         'result = get_data(user_id)',
>         'print(result)',
>     ],
>     'utils.py': [
>         'def get_data(uid):',
>         '    return db.query(uid)',
>     ],
>     'test_main.py': [
>         'data = get_data(123)',
>         'assert data is not None',
>     ],
> }
>
> print("=== deprecated 함수 호출 위치 ===")
> for filename, lines in files_content.items():
>     for num, line in enumerate(lines, 1):
>         if pattern.search(line):
>             print(f"  {filename}:{num}: {line}")
> ```
> **출력**:
> ```
> === deprecated 함수 호출 위치 ===
>   main.py:2: result = get_data(user_id)
>   test_main.py:1: data = get_data(123)
> ```
>
> **핵심 포인트**: 부정 후방 탐색 `(?<!def )`으로 함수 정의는 제외하고 호출만 찾습니다. `import` 구문의 `get_data`는 뒤에 `(`가 없으므로 자동으로 제외됩니다.

> 🔗 **연결**: 부정 후방 탐색 `(?<!...)`은 Chapter 9에서 배웠습니다.

> 🔍 **예시 14.4.2: 설정 파일의 버전 번호 일괄 업데이트**
>
> **상황**: 프로젝트 내 모든 `setup.py`, `setup.cfg`, `pyproject.toml` 파일에서 버전 번호를 `1.2.3`에서 `1.3.0`으로 업데이트해야 합니다.
>
> **풀이/설명**:
> ```python
> import re
>
> # 버전 번호 패턴: version = "1.2.3" 또는 version = '1.2.3' 또는 version="1.2.3"
> version_pattern = re.compile(
>     r'(version\s*=\s*["\'])1\.2\.3(["\'])'
> )
> replacement = r'\g<1>1.3.0\g<2>'
>
> # 예시: 치환 동작 확인
> test_lines = [
>     'version = "1.2.3"',
>     "version='1.2.3'",
>     'version="1.2.3"  # current version',
>     'description = "Version 1.2.3 release"',  # 이건 바뀌면 안 됨
> ]
>
> for line in test_lines:
>     result = version_pattern.sub(replacement, line)
>     changed = " ← 변경!" if result != line else ""
>     print(f"  {result}{changed}")
> ```
> **출력**:
> ```
>   version = "1.3.0" ← 변경!
>   version='1.3.0' ← 변경!
>   version="1.3.0"  # current version ← 변경!
>   description = "Version 1.2.3 release"
> ```
>
> **핵심 포인트**: `version=` 뒤에 오는 버전 번호만 치환하므로, 설명 문자열 안의 버전 번호는 변경되지 않습니다. `\g<1>`과 `\g<2>`로 따옴표 종류(쌍따옴표/홑따옴표)를 원래대로 보존합니다.

> 🔗 **연결**: 치환에서 `\g<1>`, `\g<2>` 역참조는 Chapter 7에서 배웠습니다.

### Section 요약

- `os.walk()`, `glob.glob()`, `pathlib.Path.rglob()` 세 가지 방법으로 디렉토리를 탐색할 수 있다
- **다중 파일 검색**은 파일 순회 + 줄 단위 정규표현식 매칭으로 구현한다
- **일괄 치환**은 반드시 `dry_run`(미리보기) 모드를 먼저 실행하고, 백업 후에 적용해야 한다
- 후방 탐색 등 고급 패턴을 활용하면 "함수 정의는 제외하고 호출만 찾기" 같은 정밀한 검색이 가능하다

---

## 14.5 파일명 처리와 리네이밍

> 💡 **한 줄 요약**: 정규표현식으로 파일명의 패턴을 인식하고, 규칙에 따라 일괄 이름 변경(리네이밍)을 자동화할 수 있다.

### 직관적 이해

디지털 카메라로 촬영한 사진 파일이 `IMG_20231010_135536.jpg`, `IMG_20231010_140012.jpg` 같은 형식이라면, 이를 `2023-10-10_01.jpg`, `2023-10-10_02.jpg`처럼 정리된 이름으로 바꾸고 싶을 수 있습니다. 파일이 수백 개라면 수작업은 불가능합니다. 정규표현식으로 파일명의 패턴을 인식하고, `re.sub()`으로 원하는 형식으로 변환하면 됩니다.

### 핵심 개념 설명

#### 파일명에서 정보 추출

```python
import re

filenames = [
    "IMG_20231010_135536.jpg",
    "IMG_20231010_140012.jpg",
    "IMG_20231011_091500.png",
    "document.pdf",
    "IMG_20231011_153045.jpg",
]

# 카메라 파일명에서 날짜와 시간 추출
cam_pattern = re.compile(
    r'IMG_(?P<year>\d{4})(?P<month>\d{2})(?P<day>\d{2})_'
    r'(?P<hour>\d{2})(?P<min>\d{2})(?P<sec>\d{2})'
    r'\.(?P<ext>\w+)$'
)

for name in filenames:
    match = cam_pattern.match(name)
    if match:
        d = match.groupdict()
        print(f"{name} → {d['year']}-{d['month']}-{d['day']} "
              f"{d['hour']}:{d['min']}:{d['sec']} (.{d['ext']})")
    else:
        print(f"{name} → (패턴 불일치)")
```

**출력**:
```
IMG_20231010_135536.jpg → 2023-10-10 13:55:36 (.jpg)
IMG_20231010_140012.jpg → 2023-10-10 14:00:12 (.jpg)
IMG_20231011_091500.png → 2023-10-11 09:15:00 (.png)
document.pdf → (패턴 불일치)
IMG_20231011_153045.jpg → 2023-10-11 15:30:45 (.jpg)
```

#### 안전한 일괄 리네이밍

파일 이름을 실제로 변경하는 스크립트에는 반드시 안전장치가 있어야 합니다:

```python
import re
import os
from pathlib import Path

def batch_rename(directory, pattern, replacement_func, dry_run=True):
    """
    디렉토리 내 파일명을 정규표현식 기반으로 일괄 변경

    Args:
        directory: 대상 디렉토리
        pattern: 파일명에 적용할 정규표현식 (컴파일된 패턴 또는 문자열)
        replacement_func: 매치 객체를 받아 새 파일명을 반환하는 함수
        dry_run: True이면 실제 변경 없이 미리보기만
    """
    if isinstance(pattern, str):
        pattern = re.compile(pattern)

    dir_path = Path(directory)
    rename_plan = []

    for filepath in sorted(dir_path.iterdir()):
        if filepath.is_file():
            match = pattern.match(filepath.name)
            if match:
                new_name = replacement_func(match)
                if new_name != filepath.name:
                    new_path = filepath.parent / new_name
                    rename_plan.append((filepath, new_path))

    if not rename_plan:
        print("변경할 파일이 없습니다.")
        return

    # 충돌 검사: 새 이름이 이미 존재하는지 확인
    for old_path, new_path in rename_plan:
        if new_path.exists() and new_path != old_path:
            print(f"⛔ 충돌: {new_path.name} 이미 존재! 작업을 중단합니다.")
            return

    # 미리보기 또는 실행
    mode = "미리보기" if dry_run else "실행"
    print(f"=== 리네이밍 {mode} ({len(rename_plan)}개 파일) ===")
    for old_path, new_path in rename_plan:
        print(f"  {old_path.name} → {new_path.name}")
        if not dry_run:
            old_path.rename(new_path)

    if not dry_run:
        print("✅ 완료!")
```

사용 예:

```python
# 카메라 파일명을 날짜-번호 형식으로 변환
cam_pattern = re.compile(
    r'IMG_(\d{4})(\d{2})(\d{2})_(\d{2})(\d{2})(\d{2})\.(\w+)$'
)

counter = {}
def camera_rename(match):
    year, month, day = match.group(1), match.group(2), match.group(3)
    ext = match.group(7)
    date_key = f"{year}-{month}-{day}"
    counter[date_key] = counter.get(date_key, 0) + 1
    return f"{date_key}_{counter[date_key]:03d}.{ext}"

# 미리보기
batch_rename('./photos', cam_pattern, camera_rename, dry_run=True)

# 확인 후 실제 실행
# batch_rename('./photos', cam_pattern, camera_rename, dry_run=False)
```

### 상세 예시

> 🔍 **예시 14.5.1: 파일명의 공백과 특수문자 정리**
>
> **상황**: 다운로드 폴더에 `My Document (Final Ver.) - Copy (2).docx` 같은 지저분한 파일명이 가득합니다. 공백을 밑줄로, 특수문자를 제거하여 정리하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import re
>
> filenames = [
>     "My Document (Final Ver.) - Copy (2).docx",
>     "보고서 (수정본) [최종].pdf",
>     "photo  2023--10.jpg",
>     "normal_file.txt",
> ]
>
> def clean_filename(name):
>     """파일명을 정리하여 안전한 형식으로 변환"""
>     # 확장자 분리
>     match = re.match(r'^(.+)\.(\w+)$', name)
>     if not match:
>         base, ext = name, ''
>     else:
>         base, ext = match.group(1), match.group(2)
>
>     # 1단계: 괄호와 내용물 제거 (또는 보존 선택)
>     base = re.sub(r'\s*[\(\[\{][^\)\]\}]*[\)\]\}]\s*', '_', base)
>     # 2단계: 특수문자를 밑줄로 변환
>     base = re.sub(r'[^\w가-힣]', '_', base)
>     # 3단계: 연속 밑줄을 하나로
>     base = re.sub(r'_+', '_', base)
>     # 4단계: 양끝 밑줄 제거
>     base = base.strip('_')
>
>     return f"{base}.{ext}" if ext else base
>
> for name in filenames:
>     cleaned = clean_filename(name)
>     changed = " ← 변경" if cleaned != name else ""
>     print(f"  {name}\n    → {cleaned}{changed}\n")
> ```
> **출력**:
> ```
>   My Document (Final Ver.) - Copy (2).docx
>     → My_Document_Copy.docx ← 변경
>
>   보고서 (수정본) [최종].pdf
>     → 보고서_최종.pdf ← 변경
>
>   photo  2023--10.jpg
>     → photo_2023_10.jpg ← 변경
>
>   normal_file.txt
>     → normal_file.txt
> ```
>
> **핵심 포인트**: 다단계 처리 파이프라인으로 파일명을 순차적으로 정리합니다. 한국어(`가-힣`)를 `\w`와 함께 보존하여 한글 파일명도 안전하게 처리합니다.

> 🔗 **연결**: `[가-힣]`을 이용한 한글 매칭은 Chapter 11에서 배웠으며, 다단계 처리 파이프라인은 Chapter 13에서 학습한 기법입니다.

> 🔍 **예시 14.5.2: 번호 기반 파일 정렬을 위한 제로 패딩**
>
> **상황**: `chapter1.txt`, `chapter2.txt`, ..., `chapter12.txt` 파일들이 있는데, 파일 탐색기에서 `chapter1.txt` 다음에 `chapter10.txt`가 오는 문제가 있습니다. `chapter01.txt`, `chapter02.txt` 형식으로 제로 패딩을 추가하고 싶습니다.
>
> **풀이/설명**:
> ```python
> import re
>
> filenames = [
>     "chapter1.txt", "chapter2.txt", "chapter10.txt",
>     "chapter12.txt", "chapter3.txt", "chapter9.txt"
> ]
>
> # 파일명의 숫자 부분을 2자리 제로 패딩으로 변환
> def zero_pad(match):
>     num = int(match.group(1))
>     ext = match.group(2)
>     return f"chapter{num:02d}.{ext}"
>
> pattern = re.compile(r'chapter(\d+)\.(\w+)$')
>
> print("=== 제로 패딩 리네이밍 ===")
> for name in sorted(filenames):
>     new_name = pattern.sub(lambda m: f"chapter{int(m.group(1)):02d}.{m.group(2)}", name)
>     if new_name != name:
>         print(f"  {name} → {new_name}")
>     else:
>         print(f"  {name} (변경 없음)")
> ```
> **출력**:
> ```
> === 제로 패딩 리네이밍 ===
>   chapter1.txt → chapter01.txt
>   chapter10.txt (변경 없음)
>   chapter12.txt (변경 없음)
>   chapter2.txt → chapter02.txt
>   chapter3.txt → chapter03.txt
>   chapter9.txt → chapter09.txt
> ```
>
> **핵심 포인트**: `re.sub()`에 람다 함수를 전달하여, 캡처된 숫자를 `int()`로 변환한 뒤 `f"...{num:02d}..."`로 포맷팅합니다. 이미 2자리인 숫자(10, 12)는 변경되지 않습니다.

> 🔗 **연결**: `re.sub()`에 함수(callable)를 전달하는 동적 치환은 Chapter 7에서 배웠습니다.

### Section 요약

- 파일명에서 정보를 추출하는 것은 명명 그룹을 활용한 일반적인 패턴 매칭이다
- 일괄 리네이밍에는 반드시 **dry_run(미리보기)**, **충돌 검사**, **백업** 안전장치가 필요하다
- 다단계 `re.sub()` 파이프라인으로 파일명의 공백, 특수문자, 숫자 포맷 등을 정리할 수 있다
- `re.sub()`에 함수를 전달하면 숫자 변환, 제로 패딩 등 동적인 이름 변환이 가능하다

---

## 14.6 정규표현식 CLI 유틸리티

> 💡 **한 줄 요약**: `argparse`와 정규표현식을 결합하면, 터미널에서 바로 사용할 수 있는 재사용 가능한 텍스트 처리 도구를 만들 수 있다.

### 직관적 이해

리눅스의 `grep` 명령어를 사용해 본 적이 있나요? `grep`은 파일에서 특정 패턴을 검색하는 유닉스 도구입니다. 이와 비슷한 기능을 하되, 여러분의 필요에 맞게 **커스터마이징된 도구**를 파이썬으로 만들 수 있습니다. `argparse` 모듈로 커맨드라인 인터페이스를 만들고, 핵심 로직은 정규표현식으로 처리하면 됩니다.

이렇게 만든 도구는 터미널에서 `python mytool.py --pattern "ERROR" --files "*.log"` 형태로 실행할 수 있으며, 반복적인 텍스트 처리 작업을 자동화하는 데 큰 도움이 됩니다.

### 핵심 개념 설명

#### argparse 기초 (최소한의 설명)

> 💬 **참고**: `argparse`는 파이썬 표준 라이브러리의 커맨드라인 인자 파싱 모듈입니다. 여기서는 정규표현식 CLI 도구를 만들기 위한 최소한의 사용법만 다룹니다.

```python
import argparse

# 기본 구조
parser = argparse.ArgumentParser(description='도구 설명')
parser.add_argument('pattern', help='검색할 정규표현식 패턴')
parser.add_argument('files', nargs='+', help='검색할 파일들')
parser.add_argument('-i', '--ignore-case', action='store_true', help='대소문자 무시')
parser.add_argument('-n', '--line-number', action='store_true', help='줄 번호 표시')

args = parser.parse_args()
# args.pattern, args.files, args.ignore_case, args.line_number 으로 접근
```

#### 실전: pygrep — 파이썬 grep 도구

```python
#!/usr/bin/env python3
"""pygrep: 정규표현식 기반 파일 검색 도구"""

import argparse
import re
import sys
from pathlib import Path


def create_parser():
    parser = argparse.ArgumentParser(
        description='정규표현식으로 파일 내용을 검색합니다',
        epilog='예: python pygrep.py "ERROR|WARN" *.log -i -n'
    )
    parser.add_argument('pattern', help='검색할 정규표현식 패턴')
    parser.add_argument('files', nargs='+', help='검색할 파일 (글로브 패턴 가능)')
    parser.add_argument('-i', '--ignore-case', action='store_true',
                        help='대소문자를 구분하지 않음')
    parser.add_argument('-n', '--line-number', action='store_true',
                        help='줄 번호를 함께 출력')
    parser.add_argument('-c', '--count', action='store_true',
                        help='매칭 줄 수만 출력')
    parser.add_argument('-v', '--invert', action='store_true',
                        help='매칭되지 않는 줄 출력')
    parser.add_argument('--color', action='store_true', default=True,
                        help='매칭 부분을 색상으로 강조 (기본값)')
    parser.add_argument('-r', '--recursive', action='store_true',
                        help='하위 디렉토리까지 재귀적으로 검색')
    return parser


def colorize_match(line, pattern):
    """매칭된 부분을 ANSI 색상으로 강조"""
    RED = '\033[91m'
    RESET = '\033[0m'
    return pattern.sub(lambda m: f"{RED}{m.group()}{RESET}", line)


def search_file(filepath, pattern, args):
    """단일 파일에서 패턴 검색"""
    results = []
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            for line_num, line in enumerate(f, 1):
                line = line.rstrip('\n')
                found = bool(pattern.search(line))

                if found != args.invert:  # XOR 로직: invert면 반전
                    results.append((line_num, line))
    except (UnicodeDecodeError, PermissionError):
        pass
    return results


def main():
    parser = create_parser()
    args = parser.parse_args()

    # 플래그 설정
    flags = 0
    if args.ignore_case:
        flags |= re.IGNORECASE

    # 패턴 컴파일
    try:
        pattern = re.compile(args.pattern, flags)
    except re.error as e:
        print(f"잘못된 패턴: {e}", file=sys.stderr)
        sys.exit(1)

    # 파일 목록 확장
    file_list = []
    for file_arg in args.files:
        if args.recursive:
            file_list.extend(Path('.').rglob(file_arg))
        else:
            file_list.extend(Path('.').glob(file_arg))

    if not file_list:
        # 글로브가 아닌 직접 경로로 시도
        file_list = [Path(f) for f in args.files if Path(f).is_file()]

    multi_file = len(file_list) > 1

    total_matches = 0

    for filepath in sorted(file_list):
        results = search_file(filepath, pattern, args)
        total_matches += len(results)

        if args.count:
            if multi_file:
                print(f"{filepath}: {len(results)}")
            else:
                print(len(results))
        else:
            for line_num, line in results:
                parts = []
                if multi_file:
                    parts.append(f"\033[35m{filepath}\033[0m")
                if args.line_number:
                    parts.append(f"\033[32m{line_num}\033[0m")

                display_line = colorize_match(line, pattern) if args.color else line
                parts.append(display_line)

                separator = ':' if args.line_number else ':'
                print(separator.join(parts) if multi_file or args.line_number
                      else display_line)

    sys.exit(0 if total_matches > 0 else 1)


if __name__ == '__main__':
    main()
```

이 도구는 터미널에서 다음과 같이 사용할 수 있습니다:

```bash
# 모든 .log 파일에서 ERROR 찾기 (줄 번호 포함)
python pygrep.py "ERROR" *.log -n

# 대소문자 무시하고 warning 또는 error 검색
python pygrep.py "warning|error" app.log -i -n

# 하위 디렉토리까지 재귀적으로 TODO 주석 검색
python pygrep.py "#\s*TODO" *.py -r -n

# 매칭되지 않는 줄만 출력 (정상 로그만 보기)
python pygrep.py "ERROR|CRITICAL" server.log -v

# 파일별 매칭 줄 수 출력
python pygrep.py "Exception" *.log -c
```

#### 실전: 로그 요약 도구

로그 파일의 주요 통계를 한눈에 보여주는 CLI 도구:

```python
#!/usr/bin/env python3
"""logstat: 로그 파일 통계 요약 도구"""

import argparse
import re
from collections import Counter
from pathlib import Path


def analyze_log(filepath, level_pattern):
    """로그 파일을 분석하여 통계 반환"""
    stats = {
        'total_lines': 0,
        'level_counts': Counter(),
        'hourly': Counter(),
        'top_errors': Counter(),
    }

    with open(filepath, 'r', encoding='utf-8') as f:
        for line in f:
            stats['total_lines'] += 1

            match = level_pattern.search(line)
            if match:
                level = match.group('level')
                stats['level_counts'][level] += 1

                # 시간대 추출 (HH:MM:SS 형태에서 HH만)
                time_match = re.search(r'(\d{2}):\d{2}:\d{2}', line)
                if time_match:
                    stats['hourly'][time_match.group(1)] += 1

                # ERROR 레벨의 메시지 수집
                if level in ('ERROR', 'CRITICAL'):
                    msg_match = re.search(r'(?:ERROR|CRITICAL)\s+\[?\w*\]?\s*(.*)', line)
                    if msg_match:
                        # 메시지의 핵심 부분만 추출 (숫자 등 가변 부분 일반화)
                        msg = re.sub(r'\d+', 'N', msg_match.group(1).strip())
                        stats['top_errors'][msg] += 1

    return stats


def main():
    parser = argparse.ArgumentParser(description='로그 파일 통계 요약')
    parser.add_argument('logfile', help='분석할 로그 파일')
    parser.add_argument('--top', type=int, default=5, help='상위 N개 항목 표시 (기본: 5)')

    args = parser.parse_args()

    level_pattern = re.compile(r'(?P<level>DEBUG|INFO|WARNING|ERROR|CRITICAL)')

    filepath = Path(args.logfile)
    if not filepath.exists():
        print(f"파일을 찾을 수 없습니다: {filepath}")
        return

    stats = analyze_log(filepath, level_pattern)

    print(f"\n📊 로그 분석: {filepath.name}")
    print(f"{'='*50}")
    print(f"전체 줄 수: {stats['total_lines']:,}")

    print(f"\n📋 로그 레벨 분포:")
    for level in ['CRITICAL', 'ERROR', 'WARNING', 'INFO', 'DEBUG']:
        count = stats['level_counts'].get(level, 0)
        if count > 0:
            pct = count / stats['total_lines'] * 100
            bar = '█' * int(pct / 2)
            print(f"  {level:10s} {count:>6,}건 ({pct:5.1f}%) {bar}")

    if stats['hourly']:
        print(f"\n🕐 시간대별 로그 (상위 {args.top}개):")
        for hour, count in stats['hourly'].most_common(args.top):
            print(f"  {hour}시: {count:,}건")

    if stats['top_errors']:
        print(f"\n🔴 상위 에러 메시지 (상위 {args.top}개):")
        for msg, count in stats['top_errors'].most_common(args.top):
            print(f"  [{count:>3}회] {msg[:70]}")

    print()


if __name__ == '__main__':
    main()
```

사용 예:
```bash
python logstat.py server.log --top 10
```

### 상세 예시

> 🔍 **예시 14.6.1: 간단한 텍스트 치환 CLI 도구**
>
> **상황**: `sed`의 `s/패턴/치환/g` 기능처럼 파일에서 정규표현식 치환을 수행하는 간단한 도구를 만들고 싶습니다.
>
> **풀이/설명**:
> ```python
> #!/usr/bin/env python3
> """pysed: 간단한 정규표현식 치환 도구"""
>
> import argparse
> import re
> import sys
>
>
> def main():
>     parser = argparse.ArgumentParser(description='정규표현식 치환 도구')
>     parser.add_argument('pattern', help='검색 패턴')
>     parser.add_argument('replacement', help='치환 문자열')
>     parser.add_argument('file', nargs='?', default='-',
>                         help='입력 파일 (생략 시 표준입력)')
>     parser.add_argument('-i', '--ignore-case', action='store_true')
>     parser.add_argument('--in-place', action='store_true',
>                         help='파일을 직접 수정')
>
>     args = parser.parse_args()
>
>     flags = re.IGNORECASE if args.ignore_case else 0
>     try:
>         regex = re.compile(args.pattern, flags)
>     except re.error as e:
>         print(f"잘못된 패턴: {e}", file=sys.stderr)
>         sys.exit(1)
>
>     # 입력 읽기
>     if args.file == '-':
>         content = sys.stdin.read()
>     else:
>         with open(args.file, 'r', encoding='utf-8') as f:
>             content = f.read()
>
>     # 치환
>     result = regex.sub(args.replacement, content)
>
>     # 출력
>     if args.in_place and args.file != '-':
>         with open(args.file, 'w', encoding='utf-8') as f:
>             f.write(result)
>         print(f"✅ {args.file} 수정 완료", file=sys.stderr)
>     else:
>         print(result, end='')
>
>
> if __name__ == '__main__':
>     main()
> ```
>
> **사용법**:
> ```bash
> # 파일의 내용을 치환하여 출력
> python pysed.py '\bfoo\b' 'bar' input.txt
>
> # 파이프라인에서 사용
> cat data.csv | python pysed.py '(\d{4})-(\d{2})-(\d{2})' '\3/\2/\1'
>
> # 파일 직접 수정
> python pysed.py 'old_function' 'new_function' code.py --in-place
> ```
>
> **핵심 포인트**: `nargs='?'`와 기본값 `'-'`를 사용하여 파일 인자가 없으면 표준입력(`stdin`)에서 읽도록 합니다. 이를 통해 파이프라인(`|`)에서도 사용할 수 있는 유닉스 철학에 부합하는 도구가 됩니다.

> 🔍 **예시 14.6.2: 데이터 추출 CLI 도구**
>
> **상황**: 텍스트 파일에서 이메일, URL, 전화번호 등을 자동으로 추출하는 범용 도구를 만들고 싶습니다.
>
> **풀이/설명**:
> ```python
> #!/usr/bin/env python3
> """extract: 텍스트에서 패턴 기반 데이터 추출"""
>
> import argparse
> import re
> import sys
>
> # 미리 정의된 패턴
> BUILTIN_PATTERNS = {
>     'email':  r'[\w.+-]+@[\w-]+(?:\.[\w-]+)+',
>     'url':    r'https?://\S+',
>     'ipv4':   r'\b\d{1,3}(?:\.\d{1,3}){3}\b',
>     'phone':  r'\b0\d{1,2}[-.\s]?\d{3,4}[-.\s]?\d{4}\b',
>     'date':   r'\b\d{4}[-/]\d{2}[-/]\d{2}\b',
> }
>
>
> def main():
>     parser = argparse.ArgumentParser(description='텍스트에서 데이터 추출')
>     group = parser.add_mutually_exclusive_group(required=True)
>     group.add_argument('-t', '--type',
>                        choices=BUILTIN_PATTERNS.keys(),
>                        help='추출할 데이터 유형')
>     group.add_argument('-p', '--pattern', help='커스텀 정규표현식 패턴')
>     parser.add_argument('file', nargs='?', default='-',
>                         help='입력 파일')
>     parser.add_argument('--unique', action='store_true',
>                         help='중복 제거')
>     parser.add_argument('--sort', action='store_true',
>                         help='결과 정렬')
>
>     args = parser.parse_args()
>
>     # 패턴 결정
>     pat_str = BUILTIN_PATTERNS.get(args.type) or args.pattern
>     try:
>         regex = re.compile(pat_str)
>     except re.error as e:
>         print(f"잘못된 패턴: {e}", file=sys.stderr)
>         sys.exit(1)
>
>     # 입력 읽기
>     if args.file == '-':
>         text = sys.stdin.read()
>     else:
>         with open(args.file, 'r', encoding='utf-8') as f:
>             text = f.read()
>
>     # 추출
>     results = regex.findall(text)
>
>     if args.unique:
>         results = list(dict.fromkeys(results))  # 순서 유지 중복 제거
>     if args.sort:
>         results.sort()
>
>     for item in results:
>         print(item)
>
>
> if __name__ == '__main__':
>     main()
> ```
>
> **사용법**:
> ```bash
> # 파일에서 이메일 추출
> python extract.py -t email contacts.txt --unique --sort
>
> # 로그에서 IP 주소 추출
> python extract.py -t ipv4 access.log --unique
>
> # 커스텀 패턴으로 특정 데이터 추출
> python extract.py -p '\b[A-Z]{2,5}-\d{4,}\b' tickets.txt
>
> # 파이프라인에서 사용
> curl -s https://example.com | python extract.py -t url --unique
> ```
>
> **핵심 포인트**: `add_mutually_exclusive_group`으로 `-t`(내장 패턴)와 `-p`(커스텀 패턴) 중 하나만 선택하게 합니다. 미리 정의된 패턴을 딕셔너리로 관리하여, 자주 쓰는 추출 패턴을 간편하게 사용할 수 있습니다.

> 🔗 **연결**: 이메일, URL, IP 주소, 전화번호 등의 검증/추출 패턴은 Chapter 12에서 설계 원칙과 함께 배웠습니다. 여기서는 그 패턴을 CLI 도구에 통합하여 재사용 가능한 형태로 만들었습니다.

> ⚠️ **주의**: CLI 도구를 배포할 때는 `re.compile()`에서 발생할 수 있는 `re.error` 예외를 반드시 처리하세요. 사용자가 잘못된 패턴을 입력할 가능성이 항상 있으며, 처리하지 않으면 스택 트레이스가 출력되어 사용자 경험이 나빠집니다.

### Section 요약

- `argparse`와 정규표현식을 결합하면 터미널에서 사용하는 **재사용 가능한 텍스트 처리 도구**를 만들 수 있다
- `pygrep`(검색), `pysed`(치환), `extract`(추출) 등 유닉스 도구에 대응하는 커스텀 유틸리티를 구축할 수 있다
- 표준입력(`stdin`) 지원으로 **파이프라인**에서도 활용 가능한 도구를 설계해야 한다
- 미리 정의된 패턴 딕셔너리를 활용하면 자주 쓰는 추출 작업을 간편하게 처리할 수 있다

---

# Part C: 통합 및 정리 (Integration Block)

## Chapter 핵심 요약 (Chapter Summary)

이번 Chapter에서는 지금까지 배운 정규표현식 기법을 **실제 파일과 로그**에 적용하는 방법을 학습했습니다.

| Section | 핵심 내용 |
|---------|----------|
| 14.1 로그 파일의 구조와 패턴 | 로그 파일의 반복적 구조(CLF, Combined, syslog 등)를 이해하고, 타임스탬프와 로그 레벨 패턴을 작성했다 |
| 14.2 Apache/Nginx 로그 분석 | `re.VERBOSE`와 명명 그룹으로 웹 서버 로그를 구조적으로 파싱하고, IP/상태코드/경로 통계를 생성했다 |
| 14.3 애플리케이션 로그 분석 | 파이썬 logging 형식, 멀티라인 트레이스백, JSON 로그 등 다양한 애플리케이션 로그를 처리했다 |
| 14.4 다중 파일 검색과 일괄 치환 | `pathlib`/`glob`과 정규표현식을 결합하여 여러 파일에서 동시에 검색·치환하는 유틸리티를 구축했다 |
| 14.5 파일명 처리와 리네이밍 | 파일명에서 정보 추출, 안전한 일괄 리네이밍, 파일명 정리 파이프라인을 구현했다 |
| 14.6 정규표현식 CLI 유틸리티 | `argparse`와 결합하여 pygrep, pysed, extract 등 재사용 가능한 커맨드라인 도구를 만들었다 |

---

## 핵심 용어 정리 (Glossary)

| 용어 | 정의 | 관련 Section |
|------|------|-------------|
| CLF (Common Log Format) | 웹 서버 접속 로그의 표준 형식. IP, 사용자, 타임스탬프, 요청, 상태 코드, 크기를 포함 | 14.1 |
| Combined Log Format | CLF에 Referer와 User-Agent를 추가한 확장 형식 | 14.1, 14.2 |
| 로그 레벨 (Log Level) | DEBUG, INFO, WARNING, ERROR, CRITICAL 등 로그 메시지의 심각도 | 14.1, 14.3 |
| 트레이스백 (Traceback) | 파이썬 에러 발생 시 호출 스택을 보여주는 여러 줄의 에러 메시지 | 14.3 |
| 글로브 패턴 (Glob Pattern) | `*`, `**`, `?` 등 와일드카드를 사용하는 파일명 매칭 패턴 (정규표현식과 다름) | 14.4 |
| dry_run | 실제 변경을 수행하지 않고 결과만 미리보는 안전 모드 | 14.4, 14.5 |
| 제로 패딩 (Zero Padding) | 숫자 앞에 0을 추가하여 자릿수를 통일하는 기법 (예: 1 → 01) | 14.5 |
| CLI (Command Line Interface) | 터미널/명령 프롬프트에서 텍스트 기반으로 상호작용하는 인터페이스 | 14.6 |
| argparse | 파이썬 표준 라이브러리의 커맨드라인 인자 파싱 모듈 | 14.6 |

---

## 개념 연결 맵 (Connection Map)

**이전 Chapter와의 연결**:
- Chapter 7의 **치환(`re.sub`)과 분할(`re.split`)**은 로그 파싱과 파일 일괄 치환의 기반이 되었습니다
- Chapter 8의 **명명 그룹(`(?P<name>...)`)**과 **`groupdict()`**는 로그 필드를 구조적으로 추출하는 핵심 도구입니다
- Chapter 9의 **전후방 탐색**은 함수 정의/호출 구분 같은 정밀한 다중 파일 검색에 활용되었습니다
- Chapter 10의 **`re.VERBOSE`와 `re.MULTILINE`**은 복잡한 로그 패턴과 멀티라인 처리에 필수적이었습니다
- Chapter 12~13의 **검증 패턴과 텍스트 정제 기법**이 CLI 도구의 내장 패턴과 파일명 정리에 재사용되었습니다

**다음 Chapter로의 연결**:
- Chapter 15에서는 정규표현식 **엔진의 내부 원리**(NFA, 백트래킹)를 배웁니다. 이 Chapter에서 작성한 로그 파싱 패턴이 내부적으로 어떻게 동작하는지 이해할 수 있게 됩니다.
- Chapter 16에서는 **성능 최적화**를 다룹니다. 수백만 줄의 로그를 처리할 때 패턴의 성능이 중요해지며, 이 Chapter에서 경험한 실전 사례가 최적화 학습의 기반이 됩니다.

---

## 자기 점검 질문 (Self-Check Questions)

**Q1.** CLF(Common Log Format)에서 타임스탬프는 어떤 기호로 감싸져 있는가?

<details>
<summary>정답</summary>

대괄호 `[...]`로 감싸져 있습니다. 예: `[10/Oct/2023:13:55:36 +0900]`
</details>

**Q2.** O/X: `glob.glob('**/*.py', recursive=True)`에서 `**`는 정규표현식의 수량자이다.

<details>
<summary>정답</summary>

**X**. `**`는 글로브 패턴(glob pattern)의 재귀적 디렉토리 탐색 와일드카드이며, 정규표현식과는 다른 문법 체계입니다.
</details>

**Q3.** 파일 일괄 치환 시 `dry_run` 옵션의 목적은 무엇인가?

<details>
<summary>정답</summary>

실제 파일을 수정하지 않고, 어떤 변경이 이루어질지 미리 확인하기 위한 안전장치입니다. 의도하지 않은 치환을 방지합니다.
</details>

**Q4.** 파이썬 에러 트레이스백에서 예외 클래스가 위치하는 곳은 트레이스백의 어디인가?

<details>
<summary>정답</summary>

트레이스백의 **마지막 줄**에 위치합니다. `ExceptionClass: error message` 형태로, 들여쓰기 없이 시작합니다.
</details>

**Q5.** O/X: `re.sub()`에 함수를 전달하면, 각 매칭마다 해당 함수가 매치 객체를 인자로 받아 호출된다.

<details>
<summary>정답</summary>

**O**. 함수는 각 매칭마다 호출되며, 매치 객체를 인자로 받아 치환 문자열을 반환합니다. 이를 통해 동적 치환(제로 패딩 등)이 가능합니다.
</details>

**Q6.** CLI 도구에서 `nargs='?'`와 기본값 `'-'`를 사용하는 이유는?

<details>
<summary>정답</summary>

파일 인자가 주어지지 않으면 표준입력(stdin)에서 읽도록 하기 위해서입니다. 이를 통해 `cat file | python tool.py` 같은 파이프라인 활용이 가능해집니다.
</details>

**Q7.** 로그 분석에서 에러 메시지의 숫자 부분을 `re.sub(r'\d+', 'N', msg)`로 일반화하는 이유는?

<details>
<summary>정답</summary>

"Connection timeout after 30s"와 "Connection timeout after 60s"처럼 숫자만 다른 동일한 유형의 에러를 하나로 그룹핑하여 빈도를 정확히 집계하기 위해서입니다.
</details>

---

# Part D: 문제 풀이 (Problem Set)

## D1. Practice (연습 문제) — 기본 개념 확인

> **Practice 14.1**: 다음 Apache CLF 로그에서 IP 주소만 추출하는 정규표현식을 작성하세요.
>
> ```
> 10.0.0.1 - admin [15/Oct/2023:09:30:00 +0900] "GET /home HTTP/1.1" 200 5120
> 172.16.0.50 - - [15/Oct/2023:09:30:01 +0900] "POST /api/login HTTP/1.1" 401 128
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> pattern = r'^(\d{1,3}(?:\.\d{1,3}){3})'
> logs = [
>     '10.0.0.1 - admin [15/Oct/2023:09:30:00 +0900] "GET /home HTTP/1.1" 200 5120',
>     '172.16.0.50 - - [15/Oct/2023:09:30:01 +0900] "POST /api/login HTTP/1.1" 401 128',
> ]
> for line in logs:
>     match = re.match(pattern, line)
>     if match:
>         print(match.group(1))
> # 출력: 10.0.0.1, 172.16.0.50
> ```
> **해설**: CLF에서 IP 주소는 항상 줄의 시작에 위치하므로 `^` 앵커를 사용합니다. `\d{1,3}`은 1~3자리 숫자, `(?:\.\d{1,3}){3}`은 `.숫자`의 3회 반복입니다.
> </details>

> **Practice 14.2**: 다음 파이썬 로그에서 로그 레벨이 `ERROR` 또는 `CRITICAL`인 줄만 필터링하는 코드를 작성하세요.
>
> ```python
> logs = [
>     "2023-10-15 09:00:00,100 INFO [app] Started",
>     "2023-10-15 09:01:00,200 ERROR [db] Timeout",
>     "2023-10-15 09:02:00,300 WARNING [app] Slow",
>     "2023-10-15 09:03:00,400 CRITICAL [core] OOM",
>     "2023-10-15 09:04:00,500 DEBUG [app] Query OK",
> ]
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> pattern = re.compile(r'\b(ERROR|CRITICAL)\b')
> for line in logs:
>     if pattern.search(line):
>         print(line)
> # 출력:
> # 2023-10-15 09:01:00,200 ERROR [db] Timeout
> # 2023-10-15 09:03:00,400 CRITICAL [core] OOM
> ```
> **해설**: `\b`로 단어 경계를 지정하여 정확한 레벨 명칭만 매칭합니다. `|`로 여러 레벨을 OR 조건으로 연결합니다.
> </details>

> **Practice 14.3**: `pathlib`을 사용하여 현재 디렉토리의 모든 `.txt` 파일 이름을 출력하는 코드를 작성하세요 (재귀적 탐색).
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> from pathlib import Path
>
> for filepath in Path('.').rglob('*.txt'):
>     print(filepath)
> ```
> **해설**: `Path('.').rglob('*.txt')`는 현재 디렉토리와 모든 하위 디렉토리에서 `.txt` 확장자를 가진 파일을 찾습니다. `rglob`의 `r`은 recursive(재귀적)를 의미합니다.
> </details>

> **Practice 14.4**: 다음 파일명에서 날짜(YYYYMMDD)를 추출하는 패턴을 작성하세요.
>
> ```python
> filenames = ["report_20231015_v2.pdf", "IMG_20231020_143000.jpg", "notes.txt"]
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> pattern = re.compile(r'(\d{8})')
> for name in filenames:
>     match = pattern.search(name)
>     if match:
>         date_str = match.group(1)
>         print(f"{name} → 날짜: {date_str[:4]}-{date_str[4:6]}-{date_str[6:]}")
>     else:
>         print(f"{name} → 날짜 없음")
> # 출력:
> # report_20231015_v2.pdf → 날짜: 2023-10-15
> # IMG_20231020_143000.jpg → 날짜: 2023-10-20
> # notes.txt → 날짜 없음
> ```
> **해설**: 8자리 연속 숫자 `\d{8}`로 YYYYMMDD 패턴을 매칭합니다. IMG 파일의 경우 시간 부분(143000)도 6자리 숫자이지만, `\d{8}`은 첫 번째 8자리(날짜)를 먼저 매칭합니다.
> </details>

> **Practice 14.5**: `re.sub()`에 함수를 전달하여, 텍스트의 모든 숫자를 2배로 만드는 코드를 작성하세요.
>
> ```python
> text = "I have 3 cats and 12 fish."
> # 기대 결과: "I have 6 cats and 24 fish."
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> text = "I have 3 cats and 12 fish."
> result = re.sub(r'\d+', lambda m: str(int(m.group()) * 2), text)
> print(result)  # I have 6 cats and 24 fish.
> ```
> **해설**: `re.sub()`에 람다 함수를 전달합니다. 각 숫자 매칭마다 함수가 호출되어, 매칭된 문자열을 정수로 변환 → 2배 → 다시 문자열로 변환합니다.
> </details>

> **Practice 14.6**: 다음 Combined Log Format 줄에서 HTTP 상태 코드와 요청 경로를 추출하세요.
>
> ```
> 192.168.0.1 - - [15/Oct/2023:10:00:00 +0900] "GET /api/v2/users?page=3 HTTP/1.1" 200 4567 "https://example.com" "Chrome/118.0"
> ```
>
> <details>
> <summary>정답 및 해설</summary>
>
> **정답**:
> ```python
> import re
> line = '192.168.0.1 - - [15/Oct/2023:10:00:00 +0900] "GET /api/v2/users?page=3 HTTP/1.1" 200 4567 "https://example.com" "Chrome/118.0"'
>
> pattern = r'"(\w+)\s+(\S+)\s+\S+"\s+(\d{3})'
> match = re.search(pattern, line)
> if match:
>     print(f"메서드: {match.group(1)}")  # GET
>     print(f"경로: {match.group(2)}")    # /api/v2/users?page=3
>     print(f"상태: {match.group(3)}")    # 200
> ```
> **해설**: 요청 필드(`"..."`) 안에서 메서드, 경로, 프로토콜을 각각 추출하고, 닫는 따옴표 뒤의 숫자가 상태 코드입니다.
> </details>

---

## D2. Exercise (응용 문제) — 개념 적용 및 통합

> **Exercise 14.1**: 다음 Apache 로그 샘플에서 **상태 코드별 요청 수**와 **404 에러가 발생한 경로 목록**을 출력하는 함수를 작성하세요.
>
> ```python
> sample_logs = """192.168.1.1 - - [15/Oct/2023:10:00:00 +0900] "GET / HTTP/1.1" 200 5000
> 192.168.1.2 - - [15/Oct/2023:10:00:01 +0900] "GET /about HTTP/1.1" 200 3000
> 192.168.1.3 - - [15/Oct/2023:10:00:02 +0900] "GET /nonexist HTTP/1.1" 404 0
> 192.168.1.1 - - [15/Oct/2023:10:00:03 +0900] "POST /api/data HTTP/1.1" 500 128
> 192.168.1.4 - - [15/Oct/2023:10:00:04 +0900] "GET /old-page HTTP/1.1" 404 0
> 192.168.1.2 - - [15/Oct/2023:10:00:05 +0900] "GET /about HTTP/1.1" 304 0
> 192.168.1.5 - - [15/Oct/2023:10:00:06 +0900] "GET /api/users HTTP/1.1" 200 1234"""
> ```
>
> **힌트**: 명명 그룹으로 경로와 상태 코드를 동시에 추출하고, `Counter`로 집계하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 명명 그룹으로 경로(`path`)와 상태 코드(`status`)를 추출 → Counter로 상태 코드 집계 → 404인 경우 경로를 별도 리스트에 수집
>
> **풀이 과정**:
> ```python
> import re
> from collections import Counter
>
> def analyze_status(log_text):
>     pattern = re.compile(
>         r'"(?:\w+)\s+(?P<path>\S+)\s+\S+"\s+(?P<status>\d{3})'
>     )
>
>     status_counter = Counter()
>     not_found_paths = []
>
>     for line in log_text.strip().split('\n'):
>         match = pattern.search(line)
>         if match:
>             status = match.group('status')
>             path = match.group('path')
>             status_counter[status] += 1
>             if status == '404':
>                 not_found_paths.append(path)
>
>     print("상태 코드별 요청 수:")
>     for status, count in sorted(status_counter.items()):
>         print(f"  {status}: {count}건")
>
>     print(f"\n404 에러 경로 ({len(not_found_paths)}건):")
>     for path in not_found_paths:
>         print(f"  {path}")
>
> analyze_status(sample_logs)
> ```
> **출력**:
> ```
> 상태 코드별 요청 수:
>   200: 3건
>   304: 1건
>   404: 2건
>   500: 1건
>
> 404 에러 경로 (2건):
>   /nonexist
>   /old-page
> ```
>
> **보충 설명**: 비캡처 그룹 `(?:...)`으로 HTTP 메서드를 그룹핑만 하고 캡처는 하지 않았습니다. 필요한 필드(경로, 상태 코드)만 명명 그룹으로 캡처하여 효율적으로 처리합니다.
> </details>

> **Exercise 14.2**: 다음 멀티라인 파이썬 에러 로그에서 (1) 에러 발생 시각, (2) 예외 클래스, (3) 에러 메시지를 추출하여 표 형식으로 출력하세요.
>
> ```python
> error_log = """2023-10-15 09:00:00,100 INFO [app] Server started
> 2023-10-15 09:15:30,200 ERROR [handler] Exception occurred:
> Traceback (most recent call last):
>   File "app.py", line 50, in process
>     return int(value)
> ValueError: invalid literal for int() with base 10: 'abc'
> 2023-10-15 09:20:00,300 INFO [app] Request processed
> 2023-10-15 09:45:10,400 ERROR [db] Database error:
> Traceback (most recent call last):
>   File "db.py", line 30, in connect
>     conn = psycopg2.connect(dsn)
> psycopg2.OperationalError: could not connect to server
> 2023-10-15 10:00:00,500 INFO [app] Shutdown"""
> ```
>
> **힌트**: ERROR 줄의 타임스탬프와 트레이스백 마지막 줄의 예외 클래스를 각각 추출한 뒤 연결하세요.
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: 두 단계로 접근 — (1) ERROR 줄 뒤에 이어지는 Traceback 블록을 하나의 단위로 추출, (2) 그 안에서 타임스탬프와 예외 정보를 파싱
>
> **풀이 과정**:
> ```python
> import re
>
> # ERROR 줄 + Traceback 블록을 하나의 단위로 매칭
> error_block_pattern = re.compile(
>     r'(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}),\d+ ERROR .+\n'
>     r'Traceback \(most recent call last\):\n'
>     r'(?:  .+\n)+'
>     r'(?P<exception>\S+): (?P<message>.+)'
> )
>
> print(f"{'시각':^22s} | {'예외 클래스':^30s} | 메시지")
> print("-" * 80)
>
> for match in error_block_pattern.finditer(error_log):
>     d = match.groupdict()
>     print(f"{d['timestamp']:22s} | {d['exception']:30s} | {d['message']}")
> ```
> **출력**:
> ```
> 시각                   | 예외 클래스                       | 메시지
> --------------------------------------------------------------------------------
> 2023-10-15 09:15:30    | ValueError                       | invalid literal for int() with base 10: 'abc'
> 2023-10-15 09:45:10    | psycopg2.OperationalError        | could not connect to server
> ```
>
> **보충 설명**: `(?:  .+\n)+`은 트레이스백의 들여쓰기된 줄(파일명, 코드)을 비캡처로 건너뛰고, 마지막 줄의 예외 정보만 캡처합니다. `\S+`로 `psycopg2.OperationalError` 같은 점이 포함된 예외 클래스도 매칭합니다.
> </details>

> **Exercise 14.3**: `chapter1.txt`, `chapter2.txt`, ..., `chapter15.txt` 파일명 리스트가 주어질 때, 모든 파일명을 `ch01.txt`, `ch02.txt`, ..., `ch15.txt` 형식으로 변환하는 함수를 작성하세요. (실제 파일 변경 없이, 변환 결과만 출력)
>
> ```python
> filenames = [f"chapter{i}.txt" for i in range(1, 16)]
> ```
>
> <details>
> <summary>모범 답안</summary>
>
> **접근 방법**: `re.sub()`에 함수를 전달하여 캡처된 숫자를 `int`로 변환 → 2자리 제로 패딩 적용
>
> **풀이 과정**:
> ```python
> import re
>
> filenames = [f"chapter{i}.txt" for i in range(1, 16)]
> pattern = re.compile(r'chapter(\d+)(\.txt)$')
>
> for name in filenames:
>     new_name = pattern.sub(lambda m: f"ch{int(m.group(1)):02d}{m.group(2)}", name)
>     if new_name != name:
>         print(f"  {name} → {new_name}")
> ```
> **출력**:
> ```
>   chapter1.txt → ch01.txt
>   chapter2.txt → ch02.txt
>   ...
>   chapter9.txt → ch09.txt
>   chapter10.txt → ch10.txt
>   ...
>   chapter15.txt → ch15.txt
> ```
>
> **보충 설명**: 확장자(`.txt`)도 캡처 그룹에 포함시켜 치환 시 보존합니다. `int()`로 변환 후 `:02d` 포맷팅으로 제로 패딩을 적용합니다.
> </details>

---

## D3. Problem (심화 문제) — 깊이 있는 사고와 창의적 문제 해결

> **Problem 14.1**: **종합 로그 분석기**를 구현하세요. 다음 요구사항을 모두 만족하는 `analyze_mixed_log()` 함수를 작성합니다.
>
> **요구사항**:
> 1. 입력은 Apache 로그와 파이썬 애플리케이션 로그가 **혼합**된 텍스트입니다
> 2. 각 줄의 로그 유형(Apache/Python)을 자동 판별합니다
> 3. Apache 로그에서: 상위 3개 IP, 상태 코드 분포를 출력합니다
> 4. Python 로그에서: 레벨별 빈도, 에러 메시지 목록을 출력합니다
> 5. 함수는 분석 결과를 딕셔너리로도 반환합니다
>
> ```python
> mixed_log = """192.168.1.1 - - [15/Oct/2023:10:00:00 +0900] "GET / HTTP/1.1" 200 5000
> 2023-10-15 10:00:01,100 INFO [scheduler] Task started
> 192.168.1.2 - - [15/Oct/2023:10:00:02 +0900] "POST /api HTTP/1.1" 500 0
> 2023-10-15 10:00:03,200 ERROR [db] Connection refused
> 192.168.1.1 - - [15/Oct/2023:10:00:04 +0900] "GET /about HTTP/1.1" 200 3000
> 2023-10-15 10:00:05,300 WARNING [app] Memory usage high: 85%
> 192.168.1.3 - - [15/Oct/2023:10:00:06 +0900] "GET /missing HTTP/1.1" 404 0
> 2023-10-15 10:00:07,400 ERROR [auth] Invalid token for user: admin
> 192.168.1.1 - - [15/Oct/2023:10:00:08 +0900] "GET /api/data HTTP/1.1" 200 1234"""
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 두 가지 로그 형식의 고유한 특징을 패턴으로 구분 — Apache 로그는 IP로 시작, Python 로그는 `YYYY-MM-DD HH:MM:SS,mmm` 타임스탬프로 시작
>
> **풀이 전략**:
> 1. Apache 패턴과 Python 패턴을 각각 정의
> 2. 각 줄에 대해 두 패턴을 순서대로 시도하여 유형 판별
> 3. 유형별로 별도의 Counter/리스트에 데이터 축적
> 4. 결과 출력 및 딕셔너리 반환
>
> **상세 풀이**:
> ```python
> import re
> from collections import Counter
>
> def analyze_mixed_log(log_text):
>     # Apache 로그 패턴
>     apache_pat = re.compile(
>         r'^(?P<ip>\d{1,3}(?:\.\d{1,3}){3})\s.+?"(?P<method>\w+)\s+(?P<path>\S+)\s+\S+"\s+(?P<status>\d{3})'
>     )
>     # Python 로그 패턴
>     python_pat = re.compile(
>         r'^(?P<timestamp>\d{4}-\d{2}-\d{2}\s\d{2}:\d{2}:\d{2}),\d+\s+'
>         r'(?P<level>DEBUG|INFO|WARNING|ERROR|CRITICAL)\s+'
>         r'\[(?P<module>\w+)\]\s+(?P<message>.+)'
>     )
>
>     result = {
>         'apache': {'ips': Counter(), 'status_codes': Counter(), 'count': 0},
>         'python': {'levels': Counter(), 'errors': [], 'count': 0},
>         'unknown': 0,
>     }
>
>     for line in log_text.strip().split('\n'):
>         apache_match = apache_pat.match(line)
>         python_match = python_pat.match(line)
>
>         if apache_match:
>             d = apache_match.groupdict()
>             result['apache']['count'] += 1
>             result['apache']['ips'][d['ip']] += 1
>             result['apache']['status_codes'][d['status']] += 1
>
>         elif python_match:
>             d = python_match.groupdict()
>             result['python']['count'] += 1
>             result['python']['levels'][d['level']] += 1
>             if d['level'] in ('ERROR', 'CRITICAL'):
>                 result['python']['errors'].append(
>                     f"[{d['timestamp']}] [{d['module']}] {d['message']}"
>                 )
>         else:
>             result['unknown'] += 1
>
>     # 결과 출력
>     print("=" * 50)
>     print("📊 혼합 로그 분석 결과")
>     print("=" * 50)
>
>     a = result['apache']
>     print(f"\n🌐 Apache 로그: {a['count']}줄")
>     print("  상위 IP:")
>     for ip, cnt in a['ips'].most_common(3):
>         print(f"    {ip}: {cnt}건")
>     print("  상태 코드:")
>     for code, cnt in sorted(a['status_codes'].items()):
>         print(f"    {code}: {cnt}건")
>
>     p = result['python']
>     print(f"\n🐍 Python 로그: {p['count']}줄")
>     print("  레벨별 빈도:")
>     for level, cnt in p['levels'].most_common():
>         print(f"    {level}: {cnt}건")
>     if p['errors']:
>         print("  에러 목록:")
>         for err in p['errors']:
>             print(f"    {err}")
>
>     if result['unknown'] > 0:
>         print(f"\n⚠️ 인식 불가: {result['unknown']}줄")
>
>     return result
>
> analyze_mixed_log(mixed_log)
> ```
>
> **확장 생각**: 실제 환경에서는 로그 형식이 더 다양할 수 있습니다. JSON 로그, syslog 형식 등을 추가로 지원하려면, 각 형식의 패턴을 리스트에 등록하고 순서대로 시도하는 "형식 자동 탐지" 구조로 확장할 수 있습니다. 이러한 확장 가능한 구조를 설계하는 것이 실전 로그 분석의 핵심 역량입니다.
> </details>

> **Problem 14.2**: **프로젝트 코드 마이그레이션 도구**를 만드세요.
>
> 파이썬 프로젝트에서 다음과 같은 코드 변환을 일괄적으로 수행해야 합니다:
>
> 1. `print(xxx)` → `logger.info(xxx)` (단, 문자열 안의 `print`나 주석의 `print`는 변경 안 함)
> 2. `dict.has_key(x)` → `x in dict`
> 3. `'%s' % var` 형식의 문자열 포맷팅 → `f'{var}'` f-string 변환 (간단한 경우만)
>
> 최소한 변환 1번과 2번을 구현하고, 입력 텍스트에 대해 dry-run 결과를 출력하는 함수를 작성하세요.
>
> ```python
> test_code = '''
> # This script prints data
> print("Hello, World!")
> result = get_data()
> print(result)
> if my_dict.has_key("name"):
>     print(my_dict["name"])
> msg = "print is a function"
> '''
> ```
>
> <details>
> <summary>풀이 가이드</summary>
>
> **핵심 아이디어**: 주석과 문자열 내의 `print`를 구분하기 위해 줄의 구조를 분석해야 합니다. 완벽한 구분은 정규표현식만으로는 어렵지만, 실용적인 수준의 휴리스틱을 적용할 수 있습니다.
>
> **풀이 전략**:
> 1. 주석(`#` 이후)은 치환에서 제외
> 2. 문자열 안의 내용도 제외 (완벽하지는 않지만, `print(`가 줄의 주요 구문인지 판단)
> 3. `has_key` 변환은 패턴 캡처와 역참조로 인자 순서 변환
>
> **상세 풀이**:
> ```python
> import re
>
> def migrate_code(code_text, dry_run=True):
>     lines = code_text.split('\n')
>     changes = []
>
>     for i, line in enumerate(lines):
>         original = line
>
>         # 주석 부분 분리
>         comment_match = re.match(r'^(\s*#.*)$', line)
>         if comment_match:
>             continue  # 주석 줄은 건너뜀
>
>         # 주석이 아닌 부분과 인라인 주석 분리
>         code_part = re.split(r'(?<!["\'])\s*#', line, maxsplit=1)
>         main_code = code_part[0]
>         inline_comment = ('#' + code_part[1]) if len(code_part) > 1 else ''
>
>         # 변환 1: print(...) → logger.info(...)
>         # 줄의 시작(들여쓰기 후)에서 print(로 시작하는 경우만
>         new_code = re.sub(
>             r'^(\s*)print\((.+)\)\s*$',
>             r'\1logger.info(\2)',
>             main_code
>         )
>
>         # 변환 2: dict.has_key(x) → x in dict
>         new_code = re.sub(
>             r'(\w+)\.has_key\(([^)]+)\)',
>             r'\2 in \1',
>             new_code
>         )
>
>         new_line = new_code + (' ' + inline_comment if inline_comment else '')
>
>         if new_line != original:
>             changes.append((i + 1, original, new_line))
>
>     # 결과 출력
>     mode = "미리보기" if dry_run else "적용"
>     print(f"=== 코드 마이그레이션 {mode} ({len(changes)}건 변경) ===\n")
>     for line_num, old, new in changes:
>         print(f"줄 {line_num}:")
>         print(f"  - {old}")
>         print(f"  + {new}")
>         print()
>
>     if not dry_run:
>         # 실제 적용된 전체 코드 반환
>         result_lines = code_text.split('\n')
>         for line_num, old, new in changes:
>             result_lines[line_num - 1] = new
>         return '\n'.join(result_lines)
>
> migrate_code(test_code)
> ```
>
> **확장 생각**: 이 문제는 "정규표현식의 한계"를 체감하는 좋은 사례입니다. 문자열 리터럴 내부의 `print`를 100% 정확히 구분하려면 파이썬 AST(Abstract Syntax Tree) 파서가 필요합니다. 하지만 실무에서는 정규표현식의 **80% 정확도 + dry_run 검증**이면 충분한 경우가 많습니다. Chapter 17에서 `pyparsing` 등 대안 도구를 배우면, 이런 한계를 극복하는 방법을 알게 됩니다.
> </details>

---

# Part E: 마무리 (Closing Block)

## 다음 단계 안내 (What's Next)

이번 Chapter에서 여러분은 정규표현식을 실제 파일과 로그에 적용하여, 자동화된 분석 및 처리 도구를 만드는 경험을 했습니다. 그런데 수백만 줄의 로그를 처리할 때, 어떤 패턴은 순식간에 끝나는 반면 어떤 패턴은 무한히 느린 경우가 있다는 것을 느꼈을 수 있습니다. **Chapter 15: 정규표현식 엔진의 원리**에서는 이 차이가 왜 발생하는지, 정규표현식 엔진이 내부적으로 패턴을 어떻게 처리하는지를 깊이 있게 탐구합니다.

---

## 추가 학습 자원 (Further Resources)

- **Apache 로그 형식 공식 문서**: [https://httpd.apache.org/docs/current/logs.html](https://httpd.apache.org/docs/current/logs.html)
- **파이썬 logging 모듈 문서**: [https://docs.python.org/3/library/logging.html](https://docs.python.org/3/library/logging.html)
- **파이썬 pathlib 가이드**: [https://docs.python.org/3/library/pathlib.html](https://docs.python.org/3/library/pathlib.html)
- **argparse 튜토리얼**: [https://docs.python.org/3/howto/argparse.html](https://docs.python.org/3/howto/argparse.html)
- **GoAccess**: 실시간 웹 로그 분석 도구 — 정규표현식 기반 분석의 실무 사례 참고
