# Boyer-Moore
- 브루트-포스를 개선한 KMP보다 효율이 더 우수한 문자열 검색 알고리즘
- 문자열을 비교하며 마지막 문자가 패턴에 없는 문자일 경우
    * 패턴의 길이만큼 건너뛴다.
- 문자열을 비교하며 마지막 문자가 패턴에 있는 경우 
    * 패턴의 마지막부터 일치하는 지 검사한 후 일치하지 않으면 건너뛴다.
- 문자열 "ABCXDEZCABACABAC"에서 패턴 "ABAC"를 검색하는 경우의 Boyer-Moore법 검색 과정
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Boyer_Moore_1.jpg" width="600px"></p>
<br>

## 구현
```java
// Boyer-Moore
static int bmMatch(String txt, String pat) {
	int pt;					        // txt 커서
	int pp;					        // pat 커서
	int txtLen = txt.length();      // txt의 문자 개수
	int patLen = pat.length();      // pat의 문자 개수
	int[] skip = new int[Character.MAX_VALUE + 1];      // 건너뛰기 표

// 건너뛰기 표 만들기
	for (pt = 0; pt <= Character.MAX_VALUE; pt++)
		skip[pt] = patLen;
	for (pt = 0; pt < patLen - 1; pt++)
		skip[pat.charAt(pt)] = patLen - pt - 1;     // pt == patLen - 1
// 검색
	while (pt < txtLen) {
		pp = patLen - 1;        // pat의 끝 문자에 주목
			while (txt.charAt(pt) == pat.charAt(pp)) {
			if (pp == 0)
				return pt;      // 검색 성공
			pp--;
			pt--;
		}
		pt += (skip[txt.charAt(pt)] > patLen - pp) ? skip[txt.charAt(pt)] : patLen - pp;
	}
	return -1;      // 검색 실패
}
```
