# 작업: Chapter 학습 자료의 ASCII 아트를 D2Coding Ligature 폰트에 맞춰 재생성

## 대상 파일
경로: `<여기에 Chapter 파일들이 있는 디렉토리 경로>`
파일 패턴: `Chapter*.md` (또는 실제 파일 패턴)

## 폰트 너비 규칙 (D2Coding Ligature 기준, 엄격 준수)

이 규칙은 절대 위반하면 안 됨. 모든 ASCII 아트는 아래 규칙으로 계산한 "시각적 폭"이 줄마다 정확히 맞아야 함.

- **2 columns (전각)**
  - 한글 음절 (U+AC00~U+D7A3): 가, 나, 다, 안, 녕, ...
  - 한글 자모 (U+3130~U+318F): ㄱ, ㅎ, ㅏ, ㅣ, ...
  - 전각 기호: ※, ○, ●, ◆, ■, □, ★, ☆, ←, →, ↑, ↓ 등
- **1 column (반각)**
  - ASCII 영문: A-Z, a-z
  - ASCII 숫자: 0-9
  - ASCII 기호: ! @ # $ % ^ & * ( ) _ + - = [ ] { } ; ' : " , . / < > ? \ | ` ~
  - 공백: (space)
  - 박스 드로잉 (U+2500~U+257F): ─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼ ═ ║ ╔ ╗ ╚ ╝ ╠ ╣ ╦ ╩ ╬
- **금지 문자** (폭이 불안정하거나 폰트 의존적)
  - 이모지 (📦 🚀 ✅ 등)
  - 중국어/일본어 한자
  - 반각 가타카나
  - 기타 판단이 애매한 기호는 사용 금지하고 ASCII 조합으로 대체

## 작업 절차

각 Chapter 파일에 대해:

1. **파일 읽기 및 분석**
   - 기존 ASCII 아트 블록들의 위치, 목적, 전달하는 정보 파악
   - 각 아트가 설명하는 개념이 무엇인지 주변 문맥으로 이해

2. **ASCII 아트 재설계**
   - 기존 아트의 목적과 정보 전달력을 유지하되, 폭 규칙에 맞게 재작성
   - 박스는 `┌─┐│└─┘` 또는 이중선 `╔═╗║╚═╝` 사용
   - 한글 레이블과 영문/기호가 섞이는 경우, 줄마다 visual width가 정확히 일치해야 함
   - 표/다이어그램에서 세로 구분선(`│`)은 모든 행에서 동일한 column 위치에 있어야 함

3. **검증 (필수)**
   - 매 ASCII 아트마다 아래 Python 함수로 각 줄의 visual width 계산
   - 박스/표의 경우 관련 줄들의 width가 모두 동일한지 assert
   - 검증 실패 시 통과할 때까지 재작성

4. **파일 업데이트**
   - 검증 통과한 ASCII 아트로 원본 파일의 해당 블록만 교체
   - 본문, 목차, 예제 등 아트 외의 내용은 건드리지 않음

## 검증 함수

```python
def visual_width(s: str) -> int:
    """D2Coding Ligature 기준 시각적 폭 계산"""
    width = 0
    for ch in s:
        code = ord(ch)
        if 0xAC00 <= code <= 0xD7A3:      # 한글 음절
            width += 2
        elif 0x3130 <= code <= 0x318F:    # 한글 자모
            width += 2
        elif 0x2600 <= code <= 0x26FF:    # 기타 기호 (대부분 전각)
            width += 2
        elif code in (0x2190, 0x2191, 0x2192, 0x2193, 0x2194, 0x2195):  # 화살표
            width += 2
        elif 0x25A0 <= code <= 0x25FF:    # 기하학적 도형
            width += 2
        elif 0x2500 <= code <= 0x257F:    # 박스 드로잉
            width += 1
        elif code < 0x80:                  # ASCII
            width += 1
        else:
            raise ValueError(f"폭이 불확실한 문자: {ch!r} (U+{code:04X})")
    return width

def verify_box(lines: list[str]) -> None:
    widths = [visual_width(l) for l in lines]
    assert len(set(widths)) == 1, f"폭 불일치: {widths}"
```

## 보고 형식

각 Chapter 완료 시:
- `Chapter XX - [제목]: ASCII 아트 N개 수정 완료`
- 폭 규칙 판단이 애매해서 대체한 문자가 있으면 명시

## 진행 방식
- Chapter 1부터 순차적으로 한 파일씩 처리
- 각 파일 처리 전 기존 아트 개수와 개선 방향 먼저 보고 후 작업 시작
- 전체 파일 일괄 수정 금지 (한 파일씩 확인하며 진행)

먼저 디렉토리를 스캔해서 대상 파일 목록을 보여주고, Chapter 1부터 시작해줘.