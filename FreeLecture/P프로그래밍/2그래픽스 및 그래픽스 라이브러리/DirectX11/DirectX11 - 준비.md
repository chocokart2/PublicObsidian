https://www.rastertek.com/dx11win10tut01.html
#정보
# 1. 제목 :

# 2. 3줄 요약
- 1.
- 2.
- 3.
# 3. 이해를 위한 사전 지식(빌드업)
- 링크
# 4. 설명
## 4.1. 이론
- 특징
- 구성 요소
- 작동 과정
- 복잡하면 그림으로 설명

Before writing any graphics code we'll need to have the tools to do so.

The first of these tools is a compiler that is preferably built into a nice IDE.

The one that I use is Visual Studio Community 2022.

There are several other IDEs available on that net, and a number of them are free as well.

I'll leave that up to you to decide which one you prefer.

I personally use Visual Studio 2022 Community because it is free and it is an excellent IDE.

You can download Visual Studio 2022 Community from the Visual Studio website.

When you install Visual Studio 2022 make sure to choose the workload that is named: Game development with C++.
> 채찍피티
> Visual Studio 2022를 설치할 때 **"Game development with C++"**이라는 작업 부하(workload)를 선택해야 합니다.
![[Pasted image 20250223010214.png]]

Once you have the IDE installed you can now write DirectX 11 applications.

Please note that other IDEs will need to have either the Windows 10 or Windows 11 SDK installed, but with Visual Studio these are already included in the Game development with C++ workload.

First you need to create a new project.

So start Visual Studio and select "Create New Project".

From here you will want to select "Windows Desktop Application".

Next give the project a name (I called mine Engine) and a location where you want it to store your files and then click on "Create".

Next you can delete the default headers, source, and resource files it creates for you. To add your own files click on "Project" and then select "Add Existing Item" and you can select your own source files or use the ones from these tutorials.
>채찍피티 번역
>다음으로, 기본적으로 생성된 헤더, 소스 및 리소스 파일을 삭제할 수 있습니다.  
>자신만의 파일을 추가하려면 "Project"를 클릭한 후 "Add Existing Item"을 선택하면 됩니다.  
>그런 다음, 직접 만든 소스 파일을 선택하거나 이 튜토리얼의 파일을 사용할 수도 있습니다.

By default in the "Solution Platforms" drop down you should see "x64". If for some reason this says "x86" then pick "x64" instead. This will set your project to 64 bit instead of the default 32 bit which is a requirement for the 64 bit alignment of data types such as matrices.


## 4.2. 예시
- 예시 상황 및 코드 작성
# 5. 추가 링크

## 5.1. 왜 이것이 작동 되는가?
- 링크
## 5.2. 이것은 어디에 활용되는가?
### 5.2.1. 이후 개념
- 링크
### 5.2.2. 해결하려는 문제
- 링크
## 5.3. 교재 및 학습자료 커리큘럼 링크
- 링크
## 5.4. 출처
- 링크
## 5.5. 그 외 추가적인 정보
- 링크


