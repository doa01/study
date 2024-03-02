# Character Encoding

- graphical character -> number, number -> graphical character
- graphical character = grapheme: single unit of human writing system

## History

- ASCII: American Standard Code for Information Interchange,
  - 1 character -> 1 byte = 8bit
  - 127 code points. only 95 printable
  - English spoken world's standard
  - in other world, each has own encoding(not compatible)
- world wide web
  - hit a problem
  - can't read other world's characters
    => need other way of encoding

## words

- code point: character에 assign된 number. code page라는 table의 position
  - https://en.wikipedia.org/wiki/Code_point
- user perceived character: what a user perceives as a smallest component of their alphabet

## UTF-8

### 3 problems to solve

- more bytes for character -> wasted storage
- in old computers: `0000 0000` -> null. end of string
- backward compatibility: compatible with basic ascii, or less work for english users

### how it works

- graphical character -> code point -> bytes(binary)
- ascii
  - code point = bytes
- utf-8

  - process

    - code point -> 1~4 bytes
    - 1 byte(singleton) / lead unit, trail unit
    - ex)
      - `I♥NY` -> `49 E2 99 A5 4E 59`
      - `I`, `N`, `Y` -> 1 byte(singleton)
      - `♥` -> `E2 99 A5`
    - singleton: `0xxxxxxx`
    - 2 bytes: `110xxxxx`, `10xxxxx`
    - 3 bytes: `1110xxxx`, `10xxxxx`, `10xxxxx`
    - ex)
      - 물: 11101011 10101100 10111100
      - ㅁ: 11100011 10000101 10000001
      - ㅜ: 11100011 10000101 10011100
      - ㄹ: 11100011 10000100 10111001
    - https://en.wikipedia.org/wiki/UTF-8#Encoding

  - english alphabets have same code points as ascii
    - singleton -> 1 byte
    - compatible with ascii

- characters and clusters
  - 1 user perceived character unit != 1 code point
  - https://www.w3.org/International/articles/definitions-characters/#characters
- characters and glyphs
  - https://www.w3.org/International/articles/definitions-characters/#characters
  - A font is a collection of glyphs
  - glyph
    - visual representation of a code point
    - vary with the font used, bold, italic, ...

### ascii, latin1, euc-kr

- latin1 = ascii + western european language symbols
- euc-kr
  - 한글의 완성형 인코딩.
  - 현대 한글의 모든 글자 중 사용 빈도가 높은 글자를 가나다 순으로 배열하여 code point 부여 \
    (가, 각, 간, ...)
  - 없는 글자는 표현이 불가능하다.
- cp949: euc-kr의 확장형

---

## words

- character: minimal unit of text that has semantic value
- character set: collection of elements used to represent.(ex. characters for third grade Chinese child, latin alphabet)
- coded character set: character set mapped to unique numbers. aka. code page
- character repertoire: set of characters that can be represented by a particular coded character set. may be closed or open
- code point: character에 assign된 number. code page라는 table의 position
  - https://en.wikipedia.org/wiki/Code_point
- code space: range of code point
- code unit: minimum bit combination that can be represent a character in charaacter encoding. ex) utf-8 -> 8 bit
- variable width encoding
  https://en.wikipedia.org/wiki/Variable-width_encoding

---

## ref

https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/
