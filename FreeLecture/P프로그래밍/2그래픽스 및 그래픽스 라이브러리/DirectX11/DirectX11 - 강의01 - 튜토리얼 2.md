DirectX 11로 코딩을 시작하기 전에 간단한 코드 프레임워크를 빌드하는 것이 좋습니다. 이 프레임워크는 기본적인 윈도우 기능을 처리하고 DirectX 11을 배우는 목적으로 체계적이고 읽기 쉬운 방식으로 코드를 확장하는 쉬운 방법을 제공합니다. 이 튜토리얼의 의도는 DirectX 11의 다양한 기능을 시도하는 것뿐이므로 의도적으로 프레임워크를 가능한 한 얇게 유지하고 전체 렌더링 엔진을 빌드하지 않습니다. DirectX 11을 확실히 이해하면 최신 그래픽 렌더링 엔진을 빌드하는 방법을 연구할 수 있습니다.

프레임워크

프레임워크는 네 가지 항목으로 시작합니다. 애플리케이션의 진입점을 처리하는 WinMain 함수가 있습니다. 또한 WinMain 함수 내에서 호출되는 전체 애플리케이션을 캡슐화하는 시스템 클래스가 있습니다. 시스템 클래스 내부에는 사용자 입력을 처리하는 입력 클래스와 DirectX 그래픽 코드를 처리하는 애플리케이션 클래스가 있습니다. 프레임워크 설정의 다이어그램은 다음과 같습니다.

![[pic6001.gif]]

이제 프레임워크가 어떻게 설정되는지 살펴보았으니 main.cpp 파일 내의 WinMain 함수부터 살펴보겠습니다.

### WinMain (main.cpp)
##### 역할

##### 함수 흐름도
###### WinMain
SystemClass 객체를 동적 할당(new)하여 생성함.  
System 객체의 Initialize() 메서드를 호출하고 결과를 result에 저장.  
초기화가 성공(true)하면 Run() 메서드를 실행하여 프로그램의 메인 루프를 수행.  
Shutdown() 메서드를 호출하여 리소스를 정리.  
delete 연산자로 System 객체의 메모리를 해제한 후, 포인터를 0(또는 nullptr)로 초기화하여 안전성을 확보.

#### 원문 번역
보시다시피, 우리는 WinMain 함수를 매우 단순하게 유지했습니다. 우리는 시스템 클래스를 생성한 다음 초기화합니다. 문제 없이 초기화되면 시스템 클래스 Run 함수를 호출합니다. Run 함수는 자체 루프를 실행하고 완료될 때까지 모든 애플리케이션 코드를 실행합니다. Run 함수가 완료되면 시스템 객체를 종료하고 시스템 객체를 정리합니다. 따라서 우리는 매우 단순하게 유지하고 전체 애플리케이션을 시스템 클래스 내부에 캡슐화했습니다. 이제 시스템 클래스 헤더 파일을 살펴보겠습니다.
#### 코드
``` C++
////////////////////////////////////////////////////////////////////////////////
// Filename: main.cpp
////////////////////////////////////////////////////////////////////////////////
#include "systemclass.h"


int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR pScmdline, int iCmdshow)
{
	SystemClass* System;
	bool result;
	
	
	// Create the system object.
	System = new SystemClass;

	// Initialize and run the system object.
	result = System->Initialize();
	if(result)
	{
		System->Run();
	}

	// Shutdown and release the system object.
	System->Shutdown();
	delete System;
	System = 0;

	return 0;
}
```
### XXX Systemclass
#### XXXX Systemclass 헤더
##### XXXXX 클래스 역할
##### XXXXX 메서드 목록
SystemClass();
SystemClass(const SystemClass&);
~SystemClass();
bool Initialize();
void Shutdown();
void Run();
LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM);
bool Frame();
void InitializeWindows(int&, int&);
void ShutdownWindows();
##### XXXXX 원문 번역
여기서 우리는 WIN32_LEAN_AND_MEAN을 정의합니다. 우리는 빌드 프로세스를 가속화하기 위해 이렇게 하며, 덜 사용되는 API 중 일부를 제외함으로써 Win32 헤더 파일의 크기를 줄입니다.

이 시점에서 프레임워크의 다른 두 클래스에 대한 헤더를 포함시켜 시스템 클래스에서 사용할 수 있도록 했습니다.

Windows.h가 포함되었으므로 창을 생성/파괴하는 함수를 호출하고 다른 유용한 win32 함수를 사용할 수 있습니다.

클래스의 정의는 매우 간단합니다. WinMain에서 호출된 Initialize, Shutdown, Run 함수가 여기에 정의되어 있습니다. 또한 해당 함수 내부에서 호출되는 일부 개인 함수도 있습니다. 또한 클래스에 MessageHandler 함수를 넣어 애플리케이션이 실행되는 동안 애플리케이션으로 전송되는 Windows 시스템 메시지를 처리합니다. 마지막으로 입력과 그래픽 렌더링을 처리할 두 객체에 대한 포인터가 되는 일부 개인 변수 m_Input과 m_Application이 있습니다.

WndProc 함수와 ApplicationHandle 포인터도 이 클래스 파일에 포함되어 있으므로 Windows 시스템 메시징을 시스템 클래스 내부의 MessageHandler 함수로 다시 리디렉션할 수 있습니다.

이제 시스템 클래스 소스 파일을 살펴보겠습니다.
##### XXXXX .h 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: systemclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _SYSTEMCLASS_H_
#define _SYSTEMCLASS_H_

///////////////////////////////
// PRE-PROCESSING DIRECTIVES //
///////////////////////////////
#define WIN32_LEAN_AND_MEAN

//////////////
// INCLUDES //
//////////////
#include <windows.h>

///////////////////////
// MY CLASS INCLUDES //
///////////////////////
#include "inputclass.h"
#include "applicationclass.h"

////////////////////////////////////////////////////////////////////////////////
// Class name: SystemClass
////////////////////////////////////////////////////////////////////////////////
class SystemClass
{
public:
	SystemClass();
	SystemClass(const SystemClass&);
	~SystemClass();

	bool Initialize();
	void Shutdown();
	void Run();

	LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM);

private:
	bool Frame();
	void InitializeWindows(int&, int&);
	void ShutdownWindows();

private:
	LPCWSTR m_applicationName;
	HINSTANCE m_hinstance;
	HWND m_hwnd;

	InputClass* m_Input;
	ApplicationClass* m_Application;
};


/////////////////////////
// FUNCTION PROTOTYPES //
/////////////////////////
static LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);


/////////////
// GLOBALS //
/////////////
static SystemClass* ApplicationHandle = 0;


#endif
```
#### XXXX Systemclass 구현
##### XXXXX 각 메서드 흐름
###### XXXXXX 메서드 SystemClass()
아무것도 하지 않음
###### XXXXXX 메서드 SystemClass(const SystemClass&);
아무것도 하지 않음
###### XXXXXX 메서드 ~SystemClass();
아무것도 하지 않음
###### XXXXXX 메서드 bool Initialize();
- `screenWidth`와 `screenHeight`를 0으로 초기화합니다.
- `InitializeWindows(screenWidth, screenHeight)`를 호출하여 윈도우를 초기화합니다.
- `m_Input` 객체를 생성한 후 `Initialize` 메서드를 호출합니다.
- `m_Application` 객체를 생성합니다.
- `m_Application->Initialize(screenWidth, screenHeight, m_hwnd)`를 호출합니다.
- `result` 값이 `false`이면 `return false`를 실행하고, `true`이면 `return true`를 실행합니다.
###### XXXXXX 메서드 void Shutdown();
애플리케이션 클래스 객체 해제
- m_Application 객체가 존재하면:
	- m_Application->Shutdown()을 호출하여 애플리케이션 클래스의 종료 작업을 수행합니다.
	- m_Application 객체를 삭제하고, m_Application 포인터를 nullptr로 설정합니다.
입력 객체 해제
- m_Input 객체가 존재하면:
	- m_Input 객체를 삭제하고, m_Input 포인터를 nullptr로 설정합니다.
윈도우 종료
- ShutdownWindows()를 호출하여 윈도우 관련 리소스를 종료합니다.
함수 종료
- 종료 작업을 마친 후 함수가 종료됩니다.
###### XXXXXX 메서드 void Run();
###### XXXXXX 메서드 LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM);
###### XXXXXX 메서드 bool Frame();
###### XXXXXX 메서드 void InitializeWindows(int&, int&);
###### XXXXXX 메서드 void ShutdownWindows();

##### XXXXX 원문 번역
클래스 생성자에서 객체 포인터를 null로 초기화합니다. 이는 이러한 객체의 초기화가 실패하면 Shutdown 함수가 나중에 해당 객체를 정리하려고 시도하기 때문에 중요합니다. 객체가 null이 아니면 유효한 생성된 객체라고 가정하고 정리해야 합니다. 애플리케이션에서 모든 포인터와 변수를 null로 초기화하는 것도 좋은 방법입니다. 그렇게 하지 않으면 일부 릴리스 빌드가 실패합니다.

여기서 빈 복사 생성자와 빈 클래스 소멸자를 만듭니다. 이 클래스에서는 필요하지 않지만 정의되지 않은 경우 일부 컴파일러가 생성해 주는데, 그럴 경우 비어 있는 것이 좋습니다.

클래스 소멸자에서 객체를 정리하지 않는다는 점도 알 수 있을 겁니다. 대신 아래에서 볼 수 있는 Shutdown 함수에서 모든 객체를 정리합니다. 그 이유는 호출되는 것을 신뢰할 수 없기 때문입니다. ExitThread()와 같은 특정 Windows 함수는 클래스 소멸자를 호출하지 않아 메모리 누수가 발생하는 것으로 알려져 있습니다. 물론 지금은 이러한 함수의 더 안전한 버전을 호출할 수 있지만 Windows에서 프로그래밍할 때는 조심하고 있습니다.
- GPT 설명 : Windows 환경에서 `ExitThread()`와 같은 API가 객체의 소멸자를 호출하지 않는 문제가 있기 때문에, 명시적으로 `Shutdown`을 호출하여 리소스를 정리하는 방식이 필요할 수도 있습니다.
- Windows 프로그래밍에서는 특정 API(예: ExitThread(), TerminateThread(), ExitProcess())가 객체의 소멸자를 호출하지 않고 종료될 수 있기 때문에, RAII 패턴을 적용하더라도 정상적인 리소스 해제가 보장되지 않을 수 있습니다.
- Windows API 환경에서는 Shutdown()을 사용하여 명시적으로 리소스를 정리하는 방식이 충분히 정당한 선택이며, 특히 멀티스레딩 환경에서는 더욱 중요합니다. 하지만, 가능하면 ExitThread() 같은 강제 종료 방식보다는 정상적인 스레드 종료(flow control)를 유지하는 것이 가장 좋은 방법입니다.

다음 Initialize 함수는 애플리케이션에 대한 모든 설정을 수행합니다. 먼저 InitializeWindows를 호출하여 애플리케이션에서 사용할 창을 만듭니다. 또한 애플리케이션에서 사용자 입력을 처리하고 화면에 그래픽을 렌더링하는 데 사용할 입력 및 애플리케이션 객체를 모두 만들고 초기화합니다.

Shutdown 함수는 정리를 합니다. 애플리케이션과 입력 객체와 관련된 모든 것을 종료하고 해제합니다. 또한 창을 종료하고 관련된 핸들을 정리합니다.

##### XXXXX .cpp 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: systemclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "systemclass.h"

SystemClass::SystemClass()
{
	m_Input = 0;
	m_Application = 0;
}

SystemClass::SystemClass(const SystemClass& other)
{
}


SystemClass::~SystemClass()
{
}

bool SystemClass::Initialize()
{
	int screenWidth, screenHeight;
	bool result;


	// Initialize the width and height of the screen to zero before sending the variables into the function.
	screenWidth = 0;
	screenHeight = 0;

	// Initialize the windows api.
	InitializeWindows(screenWidth, screenHeight);

	// Create and initialize the input object.  This object will be used to handle reading the keyboard input from the user.
	m_Input = new InputClass;

	m_Input->Initialize();

	// Create and initialize the application class object.  This object will handle rendering all the graphics for this application.
	m_Application = new ApplicationClass;
	
	result = m_Application->Initialize(screenWidth, screenHeight, m_hwnd);
	if(!result)
	{
		return false;
	}
	
	return true;
}

void SystemClass::Shutdown()
{
	// Release the application class object.
	if(m_Application)
	{
		m_Application->Shutdown();
		delete m_Application;
		m_Application = 0;
	}

	// Release the input object.
	if(m_Input)
	{
		delete m_Input;
		m_Input = 0;
	}

	// Shutdown the window.
	ShutdownWindows();
	
	return;
}
```
