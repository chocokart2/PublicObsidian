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
SystemClass는 윈도우 기반 애플리케이션의 전체적인 흐름을 관리하는 클래스입니다. 이 클래스는 애플리케이션을 초기화하고, 입력을 처리하고, 메인 루프에서 주기적으로 시스템 업데이트를 수행하며, 애플리케이션을 종료하는 기능을 담당합니다.

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
- **메시지 구조체 초기화**
    - `msg` 변수를 `ZeroMemory`를 사용하여 초기화한다.
- **루프 시작 (조건: `done == false`)**
    - `done` 변수를 `false`로 설정하고 반복을 시작한다.
- **윈도우 메시지 처리**
    - `PeekMessage`를 사용하여 메시지 큐에서 메시지를 확인하고 가져온다.
    - 메시지가 있으면:
        - `TranslateMessage`를 호출하여 키 입력을 변환한다.
        - `DispatchMessage`를 호출하여 메시지를 윈도우 프로시저로 보낸다.
- **애플리케이션 종료 여부 확인**
    - `msg.message`가 `WM_QUIT`인지 확인한다.
    - `WM_QUIT`이면 `done = true`로 설정하여 루프를 종료한다.
- **프레임 처리**
    - `Frame()` 함수를 호출하여 프레임을 처리한다.
    - `Frame()`의 반환값이 `false`이면 `done = true`로 설정하여 루프를 종료한다.
- **루프 반복 또는 종료**
    - `done`이 `false`면 루프를 반복한다.
    - `done`이 `true`이면 루프를 빠져나가고 함수 실행이 종료된다.
###### XXXXXX 메서드 LRESULT CALLBACK MessageHandler(HWND, UINT, WPARAM, LPARAM);
- **메시지 수신**
    - `MessageHandler` 함수가 호출되며 `umsg` 값(메시지 유형)을 확인한다.
- **키보드 입력 판별**
    - `umsg` 값이 `WM_KEYDOWN`인지 확인한다.
        - **예:**
            - `wparam`(눌린 키 코드)을 `m_Input->KeyDown()` 함수에 전달하여 키가 눌렸음을 기록한다.
            - `0`을 반환하고 함수 종료.
        - **아니오:** 다음 단계로 진행.
- **키보드 해제 판별**
    - `umsg` 값이 `WM_KEYUP`인지 확인한다.
        - **예:**
            - `wparam`(해제된 키 코드)을 `m_Input->KeyUp()` 함수에 전달하여 키가 해제되었음을 기록한다.
            - `0`을 반환하고 함수 종료.
        - **아니오:** 다음 단계로 진행.
- **기타 메시지 처리**
    - 위 두 경우가 아니라면 기본 메시지 처리기(`DefWindowProc`)를 호출하여 메시지를 처리하고 반환한다.
###### XXXXXX 메서드 bool Frame();
- `result` 변수를 선언한다.
- 사용자가 **ESC 키(VK_ESCAPE)** 를 눌렀는지 확인한다.
    - **참(True)**: `false`를 반환하고 종료한다.
    - **거짓(False)**: 다음 단계로 진행한다.
- `m_Application` 객체의 `Frame()` 함수를 호출하여 결과를 `result`에 저장한다.
    - **result가 false인 경우**: `false`를 반환하고 종료한다.
    - **result가 true인 경우**: 다음 단계로 진행한다.
- `true`를 반환하고 종료한다.
###### XXXXXX 메서드 void InitializeWindows(int&, int&);
- **초기 설정**
    - `ApplicationHandle`에 현재 객체를 저장한다.
    - `m_hinstance`에 현재 애플리케이션의 인스턴스를 저장한다.
    - `m_applicationName`을 `"Engine"`으로 설정한다.
- **윈도우 클래스 초기화 및 등록**
    - `WNDCLASSEX` 구조체 `wc`를 기본값으로 설정한다.
    - `WndProc`을 윈도우 프로시저 함수로 지정한다.
    - 아이콘, 커서, 배경 브러시 등을 설정한다.
    - `RegisterClassEx(&wc)`를 호출하여 윈도우 클래스를 등록한다.
- **사용자의 화면 해상도 가져오기**
    - `screenWidth`와 `screenHeight`를 `GetSystemMetrics(SM_CXSCREEN)`, `GetSystemMetrics(SM_CYSCREEN)`를 사용하여 설정한다.
- **전체 화면 또는 창 모드 설정**
    - **전체 화면 모드 (`FULL_SCREEN == true`)**
        - `DEVMODE` 구조체 `dmScreenSettings`를 초기화한다.
        - `dmPelsWidth`, `dmPelsHeight`, `dmBitsPerPel`을 현재 화면 해상도 및 32비트 색상으로 설정한다.
        - `ChangeDisplaySettings(&dmScreenSettings, CDS_FULLSCREEN)`을 호출하여 전체 화면 모드로 변경한다.
        - `posX`와 `posY`를 `(0,0)`으로 설정하여 좌측 상단에 배치한다.
    - **창 모드 (`FULL_SCREEN == false`)**
        - `screenWidth = 800`, `screenHeight = 600`으로 설정한다.
        - `posX`, `posY`를 현재 화면 중앙으로 설정한다.
- **윈도우 생성 및 화면 표시**
    - `CreateWindowEx`를 호출하여 지정된 해상도와 위치로 윈도우를 생성하고 핸들을 `m_hwnd`에 저장한다.
    - `ShowWindow(m_hwnd, SW_SHOW)`, `SetForegroundWindow(m_hwnd)`, `SetFocus(m_hwnd)`를 호출하여 창을 활성화한다.
    - `ShowCursor(false)`를 호출하여 마우스 커서를 숨긴다.
- **함수 종료**
    - `return`을 실행하여 함수가 종료된다.
###### XXXXXX 메서드 void ShutdownWindows();
- **마우스 커서를 보이게 설정한다.**
    - `ShowCursor(true);`를 호출하여 마우스 커서를 화면에 표시한다.
- **전체 화면 모드에서 나갈 경우, 디스플레이 설정을 복원한다.**
    - `FULL_SCREEN`이 `true`이면 `ChangeDisplaySettings(NULL, 0);`을 실행하여 디스플레이 설정을 기본값으로 변경한다.
- **윈도우를 제거한다.**
    - `DestroyWindow(m_hwnd);`를 호출하여 현재 애플리케이션의 창을 제거한다.
    - 이후 `m_hwnd`를 `NULL`로 설정하여 윈도우 핸들을 초기화한다.
- **애플리케이션 인스턴스를 제거한다.**
    - `UnregisterClass(m_applicationName, m_hinstance);`를 실행하여 애플리케이션 클래스 등록을 해제한다.
    - 이후 `m_hinstance`를 `NULL`로 설정하여 인스턴스를 초기화한다.
- **클래스의 전역 포인터를 초기화한다.**
    - `ApplicationHandle = NULL;`을 실행하여 애플리케이션 핸들을 해제한다.
- **함수를 종료한다.**
    - `return;`을 실행하여 `ShutdownWindows` 함수를 종료한다.
###### XXXXXX 메서드 static LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
- **함수 시작**:
    - `WndProc` 함수가 호출되면, 전달된 매개변수(`hwnd`, `umessage`, `wparam`, `lparam`)를 확인한다.
- **메시지 확인**:
    - 전달된 `umessage` 값에 따라 다른 처리를 수행한다.
- **WM_DESTROY 메시지 확인**:
    - `umessage`가 `WM_DESTROY`인 경우:
        - `PostQuitMessage(0)` 함수를 호출하여 프로그램 종료 요청을 보낸다.
        - `0`을 반환하고 함수 종료.
- **WM_CLOSE 메시지 확인**:
    - `umessage`가 `WM_CLOSE`인 경우:
        - `PostQuitMessage(0)` 함수를 호출하여 프로그램 종료 요청을 보낸다.
        - `0`을 반환하고 함수 종료.
- **기타 메시지 처리**:
    - 위의 경우에 해당하지 않는 모든 메시지는 `ApplicationHandle->MessageHandler` 함수를 호출하여 처리한다.
    - 이 함수는 `hwnd`, `umessage`, `wparam`, `lparam`을 인자로 받아 처리한 후 결과를 반환한다.
- **함수 종료**:
    - 최종적으로 처리된 결과를 반환하고 함수 실행을 마친다.

##### XXXXX 원문 번역
클래스 생성자에서 객체 포인터를 null로 초기화합니다. 이는 이러한 객체의 초기화가 실패하면 Shutdown 함수가 나중에 해당 객체를 정리하려고 시도하기 때문에 중요합니다. 객체가 null이 아니면 유효한 생성된 객체라고 가정하고 정리해야 합니다. 애플리케이션에서 모든 포인터와 변수를 null로 초기화하는 것도 좋은 방법입니다. 그렇게 하지 않으면 일부 릴리스 빌드가 실패합니다.

여기서 빈 복사 생성자와 빈 클래스 소멸자를 만듭니다. 이 클래스에서는 필요하지 않지만 정의되지 않은 경우 일부 컴파일러가 생성해 주는데, 그럴 경우 비어 있는 것이 좋습니다.

클래스 소멸자에서 객체를 정리하지 않는다는 점도 알 수 있을 겁니다. 대신 아래에서 볼 수 있는 Shutdown 함수에서 모든 객체를 정리합니다. 그 이유는 호출되는 것을 신뢰할 수 없기 때문입니다. ExitThread()와 같은 특정 Windows 함수는 클래스 소멸자를 호출하지 않아 메모리 누수가 발생하는 것으로 알려져 있습니다. 물론 지금은 이러한 함수의 더 안전한 버전을 호출할 수 있지만 Windows에서 프로그래밍할 때는 조심하고 있습니다.
- GPT 설명 : Windows 환경에서 `ExitThread()`와 같은 API가 객체의 소멸자를 호출하지 않는 문제가 있기 때문에, 명시적으로 `Shutdown`을 호출하여 리소스를 정리하는 방식이 필요할 수도 있습니다.
- Windows 프로그래밍에서는 특정 API(예: ExitThread(), TerminateThread(), ExitProcess())가 객체의 소멸자를 호출하지 않고 종료될 수 있기 때문에, RAII 패턴을 적용하더라도 정상적인 리소스 해제가 보장되지 않을 수 있습니다.
- Windows API 환경에서는 Shutdown()을 사용하여 명시적으로 리소스를 정리하는 방식이 충분히 정당한 선택이며, 특히 멀티스레딩 환경에서는 더욱 중요합니다. 하지만, 가능하면 ExitThread() 같은 강제 종료 방식보다는 정상적인 스레드 종료(flow control)를 유지하는 것이 가장 좋은 방법입니다.

다음 Initialize 함수는 애플리케이션에 대한 모든 설정을 수행합니다. 먼저 InitializeWindows를 호출하여 애플리케이션에서 사용할 창을 만듭니다. 또한 애플리케이션에서 사용자 입력을 처리하고 화면에 그래픽을 렌더링하는 데 사용할 입력 및 애플리케이션 객체를 모두 만들고 초기화합니다.

Shutdown 함수는 정리를 합니다. 애플리케이션과 입력 객체와 관련된 모든 것을 종료하고 해제합니다. 또한 창을 종료하고 관련된 핸들을 정리합니다.

Run 함수는 우리의 애플리케이션이 루프를 돌고 우리가 종료하기로 결정할 때까지 모든 애플리케이션 처리를 수행하는 곳입니다. 애플리케이션 처리가 각 루프라고 불리는 Frame 함수에서 수행됩니다. 이것은 이해하는 것이 중요한 개념인데, 이제 우리 애플리케이션의 나머지 부분도 이것을 염두에 두고 작성해야 하기 때문입니다. 의사 코드는 다음과 같습니다.
```
while not done
    check for windows system messages
    process system messages
    process application loop
    check if user wanted to quit during the frame processing
```

다음 Frame 함수는 애플리케이션의 모든 처리가 이루어지는 곳입니다. 지금까지는 상당히 간단합니다. 사용자가 Esc 키를 눌렀고 종료하려고 하는지 확인하기 위해 입력 객체를 확인합니다. 종료하고 싶지 않으면 애플리케이션 클래스 객체를 호출하여 프레임 처리를 수행하여 해당 프레임의 그래픽을 렌더링합니다.

MessageHandler 함수는 우리가 윈도우 시스템 메시지를 보내는 곳입니다. 이런 식으로 우리는 관심 있는 특정 정보를 들을 수 있습니다. 현재 우리는 키가 눌렸는지 또는 키가 놓였는지 읽고 그 정보를 입력 객체에 전달합니다. 다른 모든 정보는 윈도우 기본 메시지 핸들러로 다시 전달합니다.

InitializeWindows 함수는 렌더링에 사용할 창을 빌드하는 코드를 넣는 곳입니다. 호출하는 함수로 screenWidth와 screenHeight를 반환하여 애플리케이션 전체에서 사용할 수 있습니다. 테두리가 없는 일반 검은색 창을 초기화하기 위해 일부 기본 설정을 사용하여 창을 만듭니다. 이 함수는 FULL_SCREEN이라는 전역 변수에 따라 작은 창을 만들거나 전체 화면 창을 만듭니다. 이것이 true로 설정되면 화면이 사용자의 데스크톱 창 전체를 덮습니다. 이것이 false로 설정되면 화면 중앙에 800x600 창을 만듭니다. 수정하려는 경우를 대비하여 FULL_SCREEN 전역 변수를 applicationclass.h 파일의 맨 위에 두었습니다. 이 파일의 헤더 대신 해당 파일에 전역 변수를 넣은 이유는 나중에 이해가 될 것입니다.

ShutdownWindows는 바로 그렇게 합니다. 화면 설정을 정상으로 되돌리고 창과 관련된 핸들을 해제합니다.

WndProc 함수는 Windows가 메시지를 보내는 곳입니다. 위의 InitializeWindows 함수에서 wc.lpfnWndProc = WndProc로 창 클래스를 초기화할 때 Windows에 이름을 알려주는 것을 알 수 있습니다. SystemClass 내부에 정의된 MessageHandler 함수로 모든 메시지를 보내도록 하여 시스템 클래스에 직접 연결했기 때문에 이 클래스 파일에 포함했습니다. 이렇게 하면 메시징 기능을 클래스에 직접 연결하고 코드를 깔끔하게 유지할 수 있습니다.

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

void SystemClass::Run()
{
	MSG msg;
	bool done, result;


	// Initialize the message structure.
	ZeroMemory(&msg, sizeof(MSG));
	
	// Loop until there is a quit message from the window or the user.
	done = false;
	while(!done)
	{
		// Handle the windows messages.
		if(PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
		{
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}

		// If windows signals to end the application then exit out.
		if(msg.message == WM_QUIT)
		{
			done = true;
		}
		else
		{
			// Otherwise do the frame processing.
			result = Frame();
			if(!result)
			{
				done = true;
			}
		}

	}

	return;
}

bool SystemClass::Frame()
{
	bool result;


	// Check if the user pressed escape and wants to exit the application.
	if(m_Input->IsKeyDown(VK_ESCAPE))
	{
		return false;
	}

	// Do the frame processing for the application class object.
	result = m_Application->Frame();
	if(!result)
	{
		return false;
	}

	return true;
}

LRESULT CALLBACK SystemClass::MessageHandler(HWND hwnd, UINT umsg, WPARAM wparam, LPARAM lparam)
{
	switch(umsg)
	{
		// Check if a key has been pressed on the keyboard.
		case WM_KEYDOWN:
		{
			// If a key is pressed send it to the input object so it can record that state.
			m_Input->KeyDown((unsigned int)wparam);
			return 0;
		}

		// Check if a key has been released on the keyboard.
		case WM_KEYUP:
		{
			// If a key is released then send it to the input object so it can unset the state for that key.
			m_Input->KeyUp((unsigned int)wparam);
			return 0;
		}

		// Any other messages send to the default message handler as our application won't make use of them.
		default:
		{
			return DefWindowProc(hwnd, umsg, wparam, lparam);
		}
	}
}

void SystemClass::InitializeWindows(int& screenWidth, int& screenHeight)
{
	WNDCLASSEX wc;
	DEVMODE dmScreenSettings;
	int posX, posY;


	// Get an external pointer to this object.	
	ApplicationHandle = this;

	// Get the instance of this application.
	m_hinstance = GetModuleHandle(NULL);

	// Give the application a name.
	m_applicationName = L"Engine";

	// Setup the windows class with default settings.
	wc.style         = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;
	wc.lpfnWndProc   = WndProc;
	wc.cbClsExtra    = 0;
	wc.cbWndExtra    = 0;
	wc.hInstance     = m_hinstance;
	wc.hIcon         = LoadIcon(NULL, IDI_WINLOGO);
	wc.hIconSm       = wc.hIcon;
	wc.hCursor       = LoadCursor(NULL, IDC_ARROW);
	wc.hbrBackground = (HBRUSH)GetStockObject(BLACK_BRUSH);
	wc.lpszMenuName  = NULL;
	wc.lpszClassName = m_applicationName;
	wc.cbSize        = sizeof(WNDCLASSEX);
	
	// Register the window class.
	RegisterClassEx(&wc);

	// Determine the resolution of the clients desktop screen.
	screenWidth  = GetSystemMetrics(SM_CXSCREEN);
	screenHeight = GetSystemMetrics(SM_CYSCREEN);

	// Setup the screen settings depending on whether it is running in full screen or in windowed mode.
	if(FULL_SCREEN)
	{
		// If full screen set the screen to maximum size of the users desktop and 32bit.
		memset(&dmScreenSettings, 0, sizeof(dmScreenSettings));
		dmScreenSettings.dmSize       = sizeof(dmScreenSettings);
		dmScreenSettings.dmPelsWidth  = (unsigned long)screenWidth;
		dmScreenSettings.dmPelsHeight = (unsigned long)screenHeight;
		dmScreenSettings.dmBitsPerPel = 32;			
		dmScreenSettings.dmFields     = DM_BITSPERPEL | DM_PELSWIDTH | DM_PELSHEIGHT;

		// Change the display settings to full screen.
		ChangeDisplaySettings(&dmScreenSettings, CDS_FULLSCREEN);

		// Set the position of the window to the top left corner.
		posX = posY = 0;
	}
	else
	{
		// If windowed then set it to 800x600 resolution.
		screenWidth  = 800;
		screenHeight = 600;

		// Place the window in the middle of the screen.
		posX = (GetSystemMetrics(SM_CXSCREEN) - screenWidth)  / 2;
		posY = (GetSystemMetrics(SM_CYSCREEN) - screenHeight) / 2;
	}

	// Create the window with the screen settings and get the handle to it.
	m_hwnd = CreateWindowEx(WS_EX_APPWINDOW, m_applicationName, m_applicationName, 
				WS_CLIPSIBLINGS | WS_CLIPCHILDREN | WS_POPUP,
				posX, posY, screenWidth, screenHeight, NULL, NULL, m_hinstance, NULL);

	// Bring the window up on the screen and set it as main focus.
	ShowWindow(m_hwnd, SW_SHOW);
	SetForegroundWindow(m_hwnd);
	SetFocus(m_hwnd);

	// Hide the mouse cursor.
	ShowCursor(false);

	return;
}

void SystemClass::ShutdownWindows()
{
	// Show the mouse cursor.
	ShowCursor(true);

	// Fix the display settings if leaving full screen mode.
	if(FULL_SCREEN)
	{
		ChangeDisplaySettings(NULL, 0);
	}

	// Remove the window.
	DestroyWindow(m_hwnd);
	m_hwnd = NULL;

	// Remove the application instance.
	UnregisterClass(m_applicationName, m_hinstance);
	m_hinstance = NULL;

	// Release the pointer to this class.
	ApplicationHandle = NULL;

	return;
}

LRESULT CALLBACK WndProc(HWND hwnd, UINT umessage, WPARAM wparam, LPARAM lparam)
{
	switch(umessage)
	{
		// Check if the window is being destroyed.
		case WM_DESTROY:
		{
			PostQuitMessage(0);
			return 0;
		}

		// Check if the window is being closed.
		case WM_CLOSE:
		{
			PostQuitMessage(0);		
			return 0;
		}

		// All other messages pass to the message handler in the system class.
		default:
		{
			return ApplicationHandle->MessageHandler(hwnd, umessage, wparam, lparam);
		}
	}
}
```
### XXX Inputclass 클래스
#### XXXX Inputclass.h 클래스 헤더
##### XXXXX 클래스 역할
키보드 입력과 출력을 담당하는 클래스입니다.
##### XXXXX 메서드 목록
InputClass();
InputClass(const InputClass&);
~InputClass();
void Initialize();
void KeyDown(unsigned int);
void KeyUp(unsigned int);
bool IsKeyDown(unsigned int);
##### XXXXX 원문 번역
튜토리얼을 간단하게 유지하기 위해 DirectInput(훨씬 더 뛰어난)에 대한 튜토리얼을 할 때까지는 일단 윈도우 입력을 사용했습니다. 입력 클래스는 키보드에서 사용자 입력을 처리합니다. 이 클래스는 SystemClass::MessageHandler 함수에서 입력을 받습니다. 입력 객체는 각 키의 상태를 키보드 배열에 저장합니다. 쿼리를 받으면 특정 키가 눌렸는지 호출하는 함수에 알려줍니다. 헤더는 다음과 같습니다.
##### XXXXX .h 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: inputclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _INPUTCLASS_H_
#define _INPUTCLASS_H_


////////////////////////////////////////////////////////////////////////////////
// Class name: InputClass
////////////////////////////////////////////////////////////////////////////////
class InputClass
{
public:
	InputClass();
	InputClass(const InputClass&);
	~InputClass();

	void Initialize();

	void KeyDown(unsigned int);
	void KeyUp(unsigned int);

	bool IsKeyDown(unsigned int);

private:
	bool m_keys[256];
};
```
#### XXXX 클래스 구현
##### XXXXX 각 메서드 흐름
###### XXXXXX 메서드 InputClass();
없음
###### XXXXXX 메서드 InputClass(const InputClass&);
없음
###### XXXXXX 메서드 ~InputClass();
없음
###### XXXXXX 메서드 void Initialize();
m_keys 멤버의 모든 원소를 false로 변경
###### XXXXXX 메서드 void KeyDown(unsigned int);
키코드가 input인 키를 눌러졌음을 표기하기 위해 해당 원소를 true로 변경
###### XXXXXX 메서드 void KeyUp(unsigned int);
키코드가 input인 키가 떼졌음 표기하기 위해 해당 원소를 false로 변경
###### XXXXXX 메서드 bool IsKeyDown(unsigned int);
해당 키가 눌러졌는지를 표현
##### XXXXX 원문 번역
없음
##### XXXXX .cpp 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: inputclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "inputclass.h"


InputClass::InputClass()
{
}


InputClass::InputClass(const InputClass& other)
{
}


InputClass::~InputClass()
{
}


void InputClass::Initialize()
{
	int i;
	

	// Initialize all the keys to being released and not pressed.
	for(i=0; i<256; i++)
	{
		m_keys[i] = false;
	}

	return;
}


void InputClass::KeyDown(unsigned int input)
{
	// If a key is pressed then save that state in the key array.
	m_keys[input] = true;
	return;
}


void InputClass::KeyUp(unsigned int input)
{
	// If a key is released then clear that state in the key array.
	m_keys[input] = false;
	return;
}


bool InputClass::IsKeyDown(unsigned int key)
{
	// Return what state the key is in (pressed/not pressed).
	return m_keys[key];
}
```

### XXX Applicationclass 클래스
#### XXXX Applicationclass.h 클래스 헤더
##### XXXXX 클래스 역할
- **그래픽 관련 기능 캡슐화**
    - 애플리케이션의 그래픽 관련 기능을 담당하는 클래스입니다. 향후 그래픽 객체와 렌더링을 관리할 예정입니다.
- **전역 그래픽 설정 관리**
    - `FULL_SCREEN`, `VSYNC_ENABLED`, `SCREEN_DEPTH`, `SCREEN_NEAR` 등의 전역 변수(Global Settings)를 통해 **화면 모드(전체 화면/창 모드), VSync, 화면 깊이** 등의 그래픽 관련 설정을 정의합니다.
- **애플리케이션의 초기화 및 종료 관리**
    - `Initialize(int, int, HWND)`: 애플리케이션을 초기화하는 함수입니다.
    - `Shutdown()`: 애플리케이션을 종료 및 정리하는 함수입니다.
- **프레임 단위 처리 및 렌더링**
    - `Frame()`: 애플리케이션의 프레임별 작업을 처리하는 함수입니다.
    - `Render()`: 실제 렌더링을 수행하는 함수(현재는 선언만 되어 있음)입니다.
##### XXXXX 메서드 목록
ApplicationClass();
ApplicationClass(const ApplicationClass&);
~ApplicationClass();
bool Initialize(int, int, HWND);
void Shutdown();
bool Frame();
bool Render();
##### XXXXX 원문 번역
애플리케이션 클래스는 시스템 클래스에서 생성되는 다른 객체입니다. 이 애플리케이션의 모든 그래픽 기능은 이 클래스에 캡슐화됩니다. 또한 전체 화면 또는 창 모드와 같이 변경하고 싶을 수 있는 모든 그래픽 관련 글로벌 설정에 대해 이 파일의 헤더를 사용합니다. 현재 이 클래스는 비어 있지만 향후 튜토리얼에서는 모든 그래픽 객체를 포함할 것입니다.
##### XXXXX .h 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: applicationclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _APPLICATIONCLASS_H_
#define _APPLICATIONCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <windows.h>


/////////////
// GLOBALS //
/////////////
const bool FULL_SCREEN = false;
const bool VSYNC_ENABLED = true;
const float SCREEN_DEPTH = 1000.0f;
const float SCREEN_NEAR = 0.3f;

We'll need these four globals to start with.

////////////////////////////////////////////////////////////////////////////////
// Class name: ApplicationClass
////////////////////////////////////////////////////////////////////////////////
class ApplicationClass
{
public:
	ApplicationClass();
	ApplicationClass(const ApplicationClass&);
	~ApplicationClass();

	bool Initialize(int, int, HWND);
	void Shutdown();
	bool Frame();

private:
	bool Render();

private:

};

#endif
```
#### XXXX 클래스 구현
##### XXXXX 각 메서드 흐름
전부 비어 있음
##### XXXXX 원문 번역
지금은 튜토리얼을 위한 프레임워크를 구축하고 있기 때문에 이 클래스는 전혀 비어두었습니다.
##### XXXXX .cpp 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: applicationclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "applicationclass.h"


ApplicationClass::ApplicationClass()
{
}


ApplicationClass::ApplicationClass(const ApplicationClass& other)
{
}


ApplicationClass::~ApplicationClass()
{
}


bool ApplicationClass::Initialize(int screenWidth, int screenHeight, HWND hwnd)
{

	return true;
}


void ApplicationClass::Shutdown()
{

	return;
}


bool ApplicationClass::Frame()
{

	return true;
}


bool ApplicationClass::Render()
{

	return true;
}
```