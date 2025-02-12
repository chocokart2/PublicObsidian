#정보
# 1. 제목 :

# 2. 3줄 요약
- 1. 비페이징 풀은 디스크로 스왑되지 않고, 항상 메모리에 존재해야 하는 프로세스를 위해 존재한다.
- 2. 페이지 폴트를 다룰 수 없는 경우, 지속적인 가용성이 필요한 경우, 시간에 민감한 작업인 경우, 높은 안정성을 요구하는 경우 논 페이지드 풀에 남겨진다.
- 3.
# 3. 이해를 위한 사전 지식(빌드업)
- 페이지에 대한 지식
	- 옵시디언 링크 : [[운영체제 - 가상 메모리 aka 가상 메모리 기법, 페이징 기법]]
	- 옵시디언 링크 : [[운영체제 - 페이징 기법]]
# 4. 설명
윈도우에 작업 관리자를 열어보면 가끔씩 이런 그림이 나옴
![[Pasted image 20250114002003.png]]
메모리 풀은 운영체제 및 장치 드라이버가 데이터 구조를 저장하는데 사용하는 메모리 리소스 역할을 함
비 페이징 풀 : 가상화된 메모리를 사용하지 않고, 항상 실제 메모리를 사용하는 메모리 풀을 의미함

## 4.1. 특징
- 항상 물리적 메모리에 존재
	- 비페이징 풀의 메모리는 페이지 아웃이 불가능하며, 항상 실제 메모리에 물리적으로 존재한다
	- 시스템이 페이지 폴트를 다룰 수 없는 상황에서도 접근할 수 있는 데이터를 저장해야 한다
		- 커널 및 장치 드라이버가 이에 해당
- 시스템 안정성 보장
	- 페이지 폴트가 발생하지 않도록 하여 시스템의 안정성을 높임




- 메모리 풀 관리자는 시스템의 가상 주소 공간 영역을 사용하여 커널 모드에서 작동함
- 커널 및 장치 드라이버는 시스템이 페이지 폴트를 다룰 수 없는 상황에서도 접근할 수 있는 데이터를 저장하기 위해 항상 실제 메모리에 물리적으로 존재하는 비 페이징 풀을 사용함
- 비 페이징 풀에 저장된 일반적인 시스템 데이터 구조
	- 프로세스 및 스레드를 나타내는 커널 및 개체, 뮤텍스, 세마포어 및 이벤트와 같은 동기화 개체, 파일 개체로 표시되는 파일 참조 및 I/O 요청 패킷
	- 비페이징 풀은 시스템의 핵심 기능을 안정적으로 수행하기 위해 필요함.
	- 인터럽트 핸들러: 하드웨어 인터럽트에 응답하는 코드는 즉시 액세스 가능해야 함.
	- 장치 드라이버: 특히 높은 인터럽트 요청 수준(IRQL)에서 작동하는 장치 드라이버의 특정 부분에는 페이지가 불가능한 메모리가 필요함.
	- DMA 버퍼: 직접 메모리 액세스 작업에는 항상 물리적 RAM에 존재하는 메모리가 필요함
	- 필수 커널 구성 요소: 지속적인 가용성이 필요한 Windows 커널의 중요 부분은 비페이지 풀에 저장
	- 시간에 민감한 작업: 페이지 오류로 인한 지연을 감당할 수 없는 모든 커널 모드 코드는 비페이지 풀을 사용
## 4.2. 비페이징 풀 예제 코드




## 5.1. 왜 이것이 작동 되는가?
- 페이징 기법
	- 옵시디언 링크 : [[운영체제 - 페이징 기법]]
## 5.2. 이것은 어디에 활용되는가?
### 5.2.1. 이후 개념
- 링크
### 5.2.2. 해결하려는 문제
- 링크
## 5.3. 교재 및 학습자료 커리큘럼 링크
- 링크
## 5.4. 출처
- 링크
	- https://brunch.co.kr/@leedongins/78
## 5.5. 그 외 추가적인 정보
- 링크


