# Codex Custom Instructions

## 0. 적용 목적

이 지침은 Codex가 요구사항을 정확히 구현하도록 하기 위한 개발 규칙임.
목표는 **세련되고 간결하며, 요구사항에 정확히 맞는 코드 작성**임.
과잉 개발, 미달 개발, 불필요한 문법 사용, 단일 파일에 class를 몰아넣는 구조를 피하는 것을 우선함.

---

## 1. 핵심 개발 원칙

- 사용자가 요청한 기능만 구현할 것.
- 요청하지 않은 기능, 옵션, 예외 처리, 확장 구조, 로깅, fallback, retry, validation을 추가하지 말 것.
- 미달 구현도 금지함. 요구사항이 명확하면 실제로 동작하는 수준까지 구현할 것.
- 기본 목표는 “버그 예방용 코드”가 아니라 “요청된 기술 구현”임.
- 코드가 실행되기 위한 최소 검사는 허용하되, 과도한 방어 코드는 작성하지 말 것.
- 기존 코드의 구조가 있으면 우선 따르되, 단일 파일 과밀화나 중복 구현이 있으면 요구사항 범위 안에서만 정리할 것.

---

## 2. 과잉 문법 사용 금지

아래 문법은 기본적으로 사용하지 말 것.
요청자가 명시하거나, 사용하지 않으면 구조가 명확히 나빠지는 경우에만 제한적으로 사용할 것.

- `@dataclass`
- `@dataclass(frozen=True)`
- `@property`
- `@classmethod`
- `@staticmethod`
- `from __future__ import annotations`
- 지나치게 복잡한 type alias
- 불필요한 generic 문법
- 불필요한 decorator
- 불필요한 context manager wrapping
- 불필요한 abstract base class
- 불필요한 protocol/interface 흉내

기본적으로는 평범한 class, 일반 instance method, 일반 `__init__`, 명확한 변수, 명확한 함수 호출 흐름을 사용할 것.

예시:

```python
# 지양: 이 정도 데이터 보관에 dataclass/frozen/property/classmethod까지 쓰는 것은 과함
@dataclass(frozen=True)
class Matrix:
    values: list[list[float]]

    @property
    def size(self) -> int:
        return len(self.values)

    @classmethod
    def from_rows(cls, rows):
        return cls(rows)
```

```python
# 권장: 단순하고 직접적인 class 구조
class Matrix:
    def __init__(self, values: list[list[float]]) -> None: # values를 행렬 데이터로 저장하기 위한 생성자임
        self.values = values # 외부에서 받은 2차원 리스트를 객체 내부 상태로 보관함
        self.size = len(values) # 행렬 크기를 자주 쓰므로 생성 시 한 번 계산해 저장함
```

---

## 3. 파일 분리와 class 배치 원칙

단일 파일에 너무 많은 class를 몰아넣지 말 것.
class를 만드는 목적은 책임을 나누고, 파일 구조에서도 읽기 쉽게 분리하기 위함임.

- class는 가능한 한 의미 단위별 파일로 분리할 것.
- 하나의 파일에는 강하게 관련된 class만 둘 것.
- 서로 다른 책임의 class를 `main.py` 하나에 몰아넣지 말 것.
- `main.py`는 실행 흐름, 객체 생성, 모드 선택, 전체 pipeline 연결만 담당하게 할 것.
- 핵심 로직 class는 별도 파일로 분리할 것.
- 파일이 길어지거나 class가 2~3개 이상 섞이면 파일 분리를 우선 검토할 것.
- 단, 너무 작은 class를 무리하게 파일 하나씩 분리하여 탐색 비용을 늘리는 것도 피할 것.

권장 구조 예시:

```text
project/
  main.py                  # 프로그램 시작점, 객체 생성, 실행 흐름만 담당
  matrix.py                # Matrix class
  mac_engine.py            # MacEngine class
  classifier.py            # PatternClassifier class
  pattern_generator.py     # PatternGenerator class
  performance_analyzer.py  # PerformanceAnalyzer class
  data_json_analyzer.py    # DataJsonAnalyzer class
```

`main.py` 예시 역할:

```python
from mac_engine import MacEngine # MAC 연산 담당 class 가져옴
from classifier import PatternClassifier # 판정 담당 class 가져옴


def main() -> None: # 프로그램 시작 흐름만 담당함
    engine = MacEngine() # MAC 연산 객체 생성함
    classifier = PatternClassifier() # 점수 판정 객체 생성함
    # 이후 실행 모드 선택 및 pipeline 연결만 처리함


if __name__ == "__main__": # 이 파일을 직접 실행할 때만 main() 호출함
    main() # 전체 프로그램 흐름 시작함
```

---

## 4. Python 독립 함수 사용 제한

Python 코드 작성 시, class 밖에 단독으로 존재하는 함수는 원칙적으로 `main.py`에서만 사용할 것.

- `main.py`에서는 `main()`, 실행 모드 함수, 간단한 입출력 흐름 함수 정도만 허용함.
- `main.py`가 아닌 파일에서는 독립 함수를 만들지 말고, 관련 class의 instance method로 넣을 것.
- 단독 함수가 여러 class에서 재사용되어야 한다면, 별도 utility 함수로 흩뿌리지 말고 역할을 가진 class로 만들 것.
- `@staticmethod`, `@classmethod`로 억지로 묶지 말고, 필요한 상태가 없는 경우에도 일반 instance method로 구성할 것.
- 단순 상수는 별도 `constants.py` 또는 관련 class 내부/모듈 상단에 둘 수 있음.

예시:

```python
# 지양: main.py가 아닌 파일에 단독 함수가 흩어지는 형태

def normalize_label(label: object) -> str | None:
    ...
```

```python
# 권장: 역할을 가진 class 내부 method로 배치
class LabelNormalizer:
    def normalize(self, label: object) -> str | None: # label이 문자열이 아닐 수도 있으므로 object로 받음, 실패 시 None 반환함
        text = str(label).strip().lower() # 문자열 변환 후 공백 제거, 소문자화하여 비교 기준 통일함
        if text in {"+", "cross"}: # '+' 또는 'cross' 입력은 Cross 라벨로 정규화함
            return "Cross" # 내부 분석에서 사용할 표준 라벨 반환함
        if text == "x": # 'x' 입력은 X 라벨로 정규화함
            return "X" # 내부 분석에서 사용할 표준 라벨 반환함
        return None # 지원하지 않는 라벨이므로 이후 단계에서 유효하지 않은 값으로 처리하게 함
```

---

## 5. 객체지향 설계 원칙

객체지향 언어를 사용할 때는 구현 전에 class 구조와 pipeline을 먼저 설계할 것.

- class는 의도와 책임에 맞게 나눌 것.
- 동일한 기능을 여러 class 안에 중복 구현하지 말 것.
- 재사용 가능한 기능은 한 번만 구현하고 여러 class에서 조합해 사용할 것.
- 상위 class는 하위 구성요소를 조합해서 사용하게 만들 것.
- 기본적으로 상속보다 composition을 우선할 것.
- class를 나누었다면 파일도 의미 있게 나눌 것.

예시 원칙:

- `Calculator`와 `MPU`가 모두 덧셈이 필요하면 각각 `add()`를 따로 만들지 말 것.
- `AddOperation` 같은 재사용 가능한 class를 만들고 둘 다 그 class를 사용하게 할 것.
- `Calculator`, `AddOperation`, `NumberValue`가 있다면 아래처럼 pipeline을 구성할 것.

```text
Calculator
  └── AddOperation
        └── NumberValue
```

이 구조는 상위 class가 하위 class를 재사용하는 형태임.
같은 로직을 여러 위치에 복붙하지 않는 것이 핵심임.

---

## 6. 구현 난이도 기준

- 대학 4학년 ~ 대학원 1학년 수준의 구현을 목표로 할 것.
- 장난감 예제 수준으로 낮추지 말 것.
- 연구/실험 코드에서 바로 이해하고 수정할 수 있는 수준으로 작성할 것.
- 단, framework 수준의 과도한 구조화는 피할 것.
- 세련되고 간결하며, 요구사항에 정확한 구현을 우선할 것.

---

## 7. 시간 복잡도와 자료구조

- 구현 전에 시간 복잡도와 공간 복잡도를 고려할 것.
- 입력 크기가 커질 수 있으면 비효율적인 중첩 반복을 피할 것.
- `dict`, `set`, `list`, `heap`, 정렬, 투 포인터 등 표준 자료구조와 알고리즘을 적절히 사용할 것.
- 비효율적인 `O(n^2)` 구현을 작성하기 전에 `O(n)` 또는 `O(n log n)` 대안이 있는지 확인할 것.
- 알고리즘이 의미 있는 경우, 응답 요약에 Big-O 복잡도를 적을 것.

---

## 8. 라이브러리 사용 원칙

- 잘 쓰지 않는 함수나 obscure library를 가져오지 말 것.
- 표준 라이브러리를 먼저 사용할 것.
- 표준 라이브러리로 부족하면 major library를 사용할 것.
- 외부 라이브러리 추가는 명확히 필요한 경우에만 할 것.
- 외부 라이브러리를 쓰면 왜 필요한지 짧게 설명할 것.

우선순위:

```text
표준 라이브러리
→ major/common library
→ 직접 구현
→ obscure library는 원칙적으로 금지
```

---

## 9. 주석 작성 규칙

코드가 진행되는 과정에 유의하면서, 의미 있는 각 줄에 최대한 상세히 주석을 달 것.
주석은 한국어로 작성하고, 완벽한 문장보다 짧고 자연스러운 **음슴체**를 사용할 것.

- 처리 흐름이 있는 줄에는 inline comment를 우선 달 것.
- 너무 기초적인 문법 자체에는 주석을 달지 않아도 됨.
- 그러나 값 변환, 조건 분기, 반복, 자료구조 선택, class 간 연결, 반환값 의미는 설명할 것.
- 주석은 “무엇을 함”뿐 아니라 “왜 그렇게 함”까지 적을 것.
- 한 줄 주석이 너무 길어지면 바로 위에 block comment로 나눌 것.
- 주석이 코드보다 복잡해지면 안 됨.

권장 주석 스타일:

```python
class LabelNormalizer:
    def normalize(self, label: object) -> str | None: # label이 문자열이 아닐 수도 있으므로 object로 받음, 실패 시 None 반환함
        text = str(label).strip().lower() # str()로 문자열화, strip()으로 양쪽 공백 제거, lower()로 비교 기준 통일함
        if text in {"+", "cross"}: # '+'나 'cross'는 같은 의미로 보고 Cross로 정규화함
            return "Cross" # 내부 판정 로직에서 사용할 표준 라벨 반환함
        if text == "x": # 'x'는 X 패턴 라벨로 처리함
            return "X" # 내부 판정 로직에서 사용할 표준 라벨 반환함
        return None # 그 외 값은 정규화 불가로 보고 이후 분석 단계에서 invalid label로 처리함
```

지양:

```python
index += 1 # index를 1 증가시킴
return result # result 반환함
```

위처럼 코드만 그대로 읽어주는 주석은 달지 말 것.

---

## 10. 테스트 정책

- 테스트는 가볍게 진행할 수 있도록 할 것.
- 요청한 기능을 확인할 수 있는 최소 테스트만 작성할 것.
- 대규모 test suite를 만들지 말 것.
- 정상 케이스를 먼저 확인할 것.
- edge case는 요구사항과 직접 관련 있을 때만 추가할 것.
- 테스트용 코드도 과도한 fixture, mock, parameterization을 피할 것.

---

## 11. 코드 스타일

- 코드 흐름이 한눈에 보이게 작성할 것.
- 변수명, 함수명, class명은 의미 있게 작성할 것.
- 불필요하게 짧은 이름을 쓰지 말 것.
- `i`, `j`, `n`, `x`, `y` 같은 관례적 이름은 짧게 써도 됨.
- 불필요한 중첩을 줄일 것.
- 불필요한 전역 상태를 만들지 말 것.
- 한 함수나 method가 너무 많은 일을 하지 않게 할 것.
- 기존 repository 스타일이 있으면 우선 따를 것.

---

## 12. 응답 요약 형식

구현 후 응답은 짧게 정리할 것.

필수 포함:

1. 무엇을 구현했는지
2. 어떤 파일과 class를 수정했는지
3. class와 파일 분리 구조가 어떻게 되는지
4. 가벼운 테스트 또는 확인 결과
5. 의미 있는 알고리즘이 있으면 시간 복잡도

불필요하게 긴 설명은 피할 것.
요청하지 않은 추가 개선 제안은 하지 말 것.

---

## 13. Codex 작업 지시 템플릿

Codex에게 작업을 줄 때는 아래 형식을 우선 사용할 것.

```md
Goal:
- 구현할 목표를 한 줄로 적음.

Context:
- 관련 파일, 현재 구조, 수정해야 할 범위를 적음.

Constraints:
- 요청하지 않은 기능 추가 금지.
- 방어 코드, 과도한 validation, logging, fallback 추가 금지.
- dataclass, property, classmethod, staticmethod 사용 금지.
- class는 의미 단위별 파일로 분리.
- main.py 외 파일에서는 단독 함수 금지.
- Python에서는 일반 class와 instance method 중심으로 구현.
- 주석은 코드 진행 과정에 맞춰 의미 있는 각 줄에 음슴체로 작성.
- 표준 라이브러리 우선 사용.
- 테스트는 가볍게 작성.

Done when:
- 요청한 기능이 동작함.
- class 중복 구현이 없음.
- 파일 분리가 의미 있게 되어 있음.
- 가벼운 실행 예시 또는 테스트가 통과함.
- 시간 복잡도 설명이 필요한 경우 Big-O로 요약됨.
```
