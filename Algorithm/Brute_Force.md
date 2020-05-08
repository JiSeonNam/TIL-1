# 브루트-포스법(brute force)
- 문자열 검색의 기초라고 할 수 있다.
- 선형 검색을 확장한 알고리즘으로, 전체 데이터를 탐색하는 방법이다.
- 따라서 효율이 매우 좋지 않다.
- 시간복잡도 : O(문자열의 길이 * 검색할 문자열의 길이)
- 검색 과정
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Algorithm/img/Brute_Force_1.jpg"></p>

## 구현
```java
// 검색할 문자열 : pat, 문자열 원본 : txt
static int bruteForce(String txt, String pat) {
	int pt = 0;		// txt 커서
	int pp = 0;		// pat 커서

	while (pt != txt.length() && pp != pat.length()) {
		if (txt.charAt(pt) == pat.charAt(pp)) {
			pt++;
			pp++;
		} else {
			pt = pt - pp + 1;
			pp = 0;
		}
	}
	if (pp == pat.length())			// 검색 성공
		return pt - pp;
	return -1;						// 검색 실패
}
```