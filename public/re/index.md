# Regular Expression(정규표현식)

<!--more-->
# Regular Expression

복잡한 문자열을 처리할 때 사용하는 기법으로, 문자열을 처리하는 모든 곳에서 사용

## 메타문자

### `[ ]`

- 문자 클래스
- **`[ ]`** 사이의 문자들과 매치
- **`-`** : 두 문자 사이의 범위
- **`^`** : not

```python
1. [abc] # a,b,c중 한개의 문자와 매칭
2. [a-c] # == [abc]
3. [a-zA-Z] # 모든 알파벳
4. [0-9] # 숫자
```

| re | content |
| --- | --- |
| \d | 숫자와 매치, [0-9] |
| \D | 숫자가 아닌 것과 매치, [^0-9] |
| \s | whitespace 문자와 매치, [  \t\n\r\f\v] |
| \S | whitespace 문자가 아닌것과 매치, [^ \t\n\r\f\v] |
| \w | 문자+숫자와 매치, [a-zA-Z0-9_] |
| \W | 문자+숫자가 아닌 것과 매치, [^a-zA-Z0-9_] |

### `.`

- 줄바꿈 문자를 제외한 모든 문자
- 문자 클래스(`**[ ]**`) 내에 Dot(`**.**`) 메타 문자가 사용된다면 이것은 "모든 문자"라는 의미가 아닌 문자 `**.**` 그대로를 의미

```python
1. a.b # a + [모든 문자] + b
2. a[.]b # a.b
```

### `*`, `+`, `{m,n}`

- 반복
- **`*`** : 바로 앞에 있는 문자가 0부터 무한대로 반복될 수 있음
- **`+`** :  바로 앞에 있는 문자가 1부터 무한대로 반복될 수 있음(최소 1번)
- **`{m,n}`**: 바로 앞에 있는 문자가 m부터 n번까지 반복될 수 있음(최소 m번, 최대 n번)

```python
ca*t      # ct, cat,caat,caaaat, ...
ca+t      # cat, caat, caaat, ...
ca{2}t    # caat
ca{2,4}t  # caat, caaat, caaaat
```

### `?`

- **'{0, 1}'**
- 있어도 되고, 없어도 된다

```python
mol?lu    # mollu, molu
```

### `|`

- or

### `^`, `$`,`\A`,`\Z`

- **`^`**: 문자열의 맨 처음과 일치, `re.MULTILINE`사용시 각 줄의 처음과 일치
- **`$`**: 문자열 맨 끝과 일치, `re.MULTILINE`사용시 각 줄의 끝과일치
- **`\A`**: 전체 문자열의 맨 처음과 일치
- **`\Z`**: 전체 문자열의 맨 끝과 일치

### `\b`, `\B`

- 단어 구분자
- `\b`: whitespace로 구분된 단어인 경우 매치
- `\B`: whitespace로 구분된 단어가 아닌 경우 매치

---

## Grouping

- **`( )`**

| group(인덱스) | 설명 |
| --- | --- |
| group(0) | 매치된 전체 문자열 |
| group(1) | 첫 번째 그룹에 해당되는 문자열 |
| group(2) | 두 번째 그룹에 해당되는 문자열 |
| group(n) | n 번째 그룹에 해당되는 문자열 |

---

## module re

- 파이썬을 설치할 때 자동으로 설치되는 기본 라이브러리
- 

| Method | 목적 |
| --- | --- |
| match() | 문자열의 처음부터 정규식과 매치되는지 조사한다. |
| search() | 문자열 전체를 검색하여 정규식과 매치되는지 조사한다. |
| findall() | 정규식과 매치되는 모든 문자열(substring)을 리스트로 돌려준다. |
| finditer() | 정규식과 매치되는 모든 문자열(substring)을 반복 가능한 객체로 돌려준다. |

```python
# match, search: true면 객체 반환, false면 None 반환
print(p.match("python"))
print(p.match("3 python"))
print(p.search("python"))
print(p.search("3 python"))

>>>
<re.Match object; span=(0, 6), match='python'>
None
<re.Match object; span=(0, 6), match='python'>
<re.Match object; span=(2, 8), match='python'>
--------
# match 객체 활용
>>> m = p.match("python")
>>> m.group()            # group: 매치된 문자열 반환
'python'
>>> m.start()            # start: 매치된 문자열의 시작위치 반환
0
>>> m.end()              # end: 매치된 문자열의 끝위치 반환
6
>>> m.span()             # 매치된 문자열의 (시작, 끝) 반환
(0, 6)
```

```python
print(p.findall("life is too short"))
for r in p.finditer("life is too short"):
    print(r)

>>>
['life', 'is', 'too', 'short']
<re.Match object; span=(0, 4), match='life'>
<re.Match object; span=(5, 7), match='is'>
<re.Match object; span=(8, 11), match='too'>
<re.Match object; span=(12, 17), match='short'>
```

---

## 컴파일 옵션

- **DOTALL(S)** - `.` 이 줄바꿈 문자를 포함하여 모든 문자와 매치할 수 있도록 한다.
    
    ```python
    >>> p = re.compile('a.b', re.DOTALL)
    >>> m = p.match('a\nb')
    >>> print(m)
    <re.Match object; span=(0, 3), match='a\nb'>
    ```
    
- **IGNORECASE(I)** - 대소문자에 관계없이 매치할 수 있도록 한다.
    
    ```python
    >>> p = re.compile('[a-z]+', re.I)
    >>> p.match('python')
    <re.Match object; span=(0, 6), match='python'>
    >>> p.match('Python')
    <re.Match object; span=(0, 6), match='Python'>
    >>> p.match('PYTHON')
    <re.Match object; span=(0, 6), match='PYTHON'>
    ```
    
- **MULTILINE(M)** - 여러줄과 매치할 수 있도록 한다. (`^`, `$` 메타문자의 사용과 관계가 있는 옵션이다)
    
    ```python
    import re
    p = re.compile("^python\s\w+", re.MULTILINE)
    
    data = """python one
    life is too short
    python two
    you need python
    python three"""
    
    print(p.findall(data))
    >>> ['python one', 'python two', 'python three']
    ```
    
- **VERBOSE(X)** - verbose 모드를 사용할 수 있도록 한다. (정규식을 보기 편하게 만들수 있고 주석등을 사용할 수 있게된다.)
    
    ```python
    charref = re.compile(r"""
     &[#]                # Start of a numeric entity reference
     (
         0[0-7]+         # Octal form
       | [0-9]+          # Decimal form
       | x[0-9a-fA-F]+   # Hexadecimal form
     )
     ;                   # Trailing semicolon
    """, re.VERBOSE)
    ```
    

---

## sub

- 정규식과 매치되는 부분을 다른 문자로 변경

```python
# [정규식].sub("바꿀 문자열", "대상 문자열")
>>> p = re.compile('(blue|white|red)')
>>> p.sub('colour', 'blue socks and red shoes')
'colour socks and colour shoes'
```
