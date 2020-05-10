# KMP
- 문자열 중에 특정 패턴을 찾아내는 문자열 검색 알고리즘
- 문자열에서 검색할 문자열의 접두사 = 접미사가 될 수 있는 가장 긴 부분을 찾아 그 위치에서부터 다시 문자열 검색을 한다.
- 시간복잡도 : O(문자열의 길이 + 검색할 문자열의 길이)
- 검색 과정
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/KMP_1.jpg" width="600px"></p>

## 구현
```java
// 검색할 문자열 : pat, 문자열 원본 : txt
static int kmpMatch(String txt, String pat) {
	int pt = 1;		// txt 커서
	int pp = 0;		// pat 커서
	int[] skip = new int[pat.length() + 1];		// 건너뛰기 표

	// 건너뛰기 표를 만들기
	skip[pt] = 0;
	while (pt != pat.length()) {
		if (pat.charAt(pt) == pat.charAt(pp))
			skip[++pt] = ++pp;
		else if (pp == 0)
			skip[++pt] = pp;
		else
			pp = skip[pp];
	}

	// 검색
	pt = pp = 0;
	while (pt != txt.length() && pp != pat.length()) {
		if (txt.charAt(pt) == pat.charAt(pp)) {
			pt++;
			pp++;
		} else if (pp == 0)
			pt++;
		else
			pp = skip[pp];
	}
		if (pp == pat.length())		// pt - pp를 반환
		return pt - pp;
	return -1;		// 검색 실패
}
```
