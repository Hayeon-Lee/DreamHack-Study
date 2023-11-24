# Background: Web Browser

1. 웹 브라우저

	* dreamhack.io 를 입력했을 때 브라우저가 하게 되는 일
		* 웹 브라우저의 주소창에 입력된 주소를 해석 (URL 분석)
		* dreamhack.io 에 해당하는 주소 탐색 (DNS요청)
		* HTTP를 통해 dreamhack.io에 요청
		* dreamhack.io의 HTTP 응답 수신
		* 리소스 다운로드 및 웹 렌더링 (HTML, CSS, JS)

2. URL
	* 정의: 웹에 있는 리소스의 위치를 표현하는 문자열

3. Domain Name
	* Host: 웹 브라우저가 접속할 웹 서버의 주소
		* 구성요소 1) Domain Name
		* 구성요소 2) IP Address 
	* Domain Name을 Host 값으로 이용할 때
		1) 브라우저는 Domain Name Server(DNS)에 도메인 이름을 질의한다.
		2) 이후 DNS가 응답한 IP Address를 사용한다.
		- http://example.com에 접속할 경우, DNS에 질의한 example.com의 IP와 통신하는 것
	* Domain Name에 대한 정보는 MacOS/Linux/Windows에서 nslookup 명령어를 사용해 확인할 수 있다.

4. 웹 렌더링
	* 정의: 서버로부터 받은 리소스를 이용자에게 시각화하는 행위
		* 웹 브라우저는 서버의 응답을 받은 뒤, 리소스의 타입을 확인하여 적절하게 보여준다.
	* 웹 렌더링 엔진에 의해 이뤄진다.
		* 사파리: 웹킷
		* 크롬: 블링크
		* 파이어폭스: 개코
		-> 각 브라우저 별로 속도 차이는 있지만, 기능은 같다.
