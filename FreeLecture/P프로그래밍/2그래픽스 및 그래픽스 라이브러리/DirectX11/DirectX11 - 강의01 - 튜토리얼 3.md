해당 강의의 출처는 다음과 같습니다. 정말 감사드립니다.
https://www.rastertek.com/dx11win10tut03.html

프레임워크

우리는 Direct3D 시스템 기능을 모두 처리할 프레임워크에 또 다른 클래스를 추가할 것입니다. 우리는 클래스 D3DClass를 호출할 것입니다. 저는 아래의 프레임워크 다이어그램을 업데이트했습니다:

![[pic6003.gif]]

보시다시피 D3DClass는 ApplicationClass 내부에 위치합니다. 이전 튜토리얼에서는 모든 새로운 그래픽 관련 클래스가 ApplicationClass에 캡슐화된다고 언급했으며, 이것이 새로운 D3DClass에 가장 적합한 위치인 이유입니다. 이제 ApplicationClass에 적용된 변경 사항을 살펴보겠습니다.

D3DClass를 장착했습니다
### XXX ApplicationClass 업데이트
#### XXXX 클래스 헤더
##### XXXXX 클래스 역할
추가적으로 d3dclass 헤더가 추가되고, D3DClass객체를 멤버로 가지기 시작합니다.
##### XXXXX 메서드 목록
변경 없음
##### XXXXX 원문 번역
첫 번째 변경 사항은 다음과 같습니다. windows.h의 include를 제거하고 대신 새로운 d3dclass.h를 포함했습니다.

두 번째 변경 사항은 m_Direct3D라고 부르는 D3DClass에 대한 새로운 프라이빗 포인터입니다. 궁금하시다면 모든 클래스 변수에 접두사 m_을 사용합니다. 이렇게 하면 코딩할 때 어떤 변수가 클래스의 멤버이고 어떤 변수가 아닌지 빠르게 기억할 수 있습니다.
##### XXXXX .h 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename:  applicationclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _APPLICATIONCLASS_H_
#define _APPLICATIONCLASS_H_

///////////////////////
// MY CLASS INCLUDES //
///////////////////////
#include "d3dclass.h"

/////////////
// GLOBALS //
/////////////
const bool FULL_SCREEN = false;
const bool VSYNC_ENABLED = true;
const float SCREEN_DEPTH = 1000.0f;
const float SCREEN_NEAR = 0.3f;


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
	D3DClass* m_Direct3D;
};

#endif
```
#### XXXX 클래스 구현
##### XXXXX 각 메서드 흐름
###### XXXXXX 메서드 생성자 ApplicationClass()
m_Direct3D 포인터에 널 포인터를 넣습니다.
###### XXXXXX 메서드 Initialize(int screenWidth, int screenHeight, HWND hwnd)
- Initialize 메서드는 화면의 너비(screenWidth), 높이(screenHeight), 윈도우 핸들(hwnd)을 입력으로 받는다.
- D3DClass의 인스턴스를 동적으로 생성하여 m_Direct3D 멤버 변수에 저장한다.
- m_Direct3D의 Initialize 메서드를 호출하여 Direct3D를 초기화한다.
	- 초기화 시, VSYNC 설정, 전체 화면 여부, 화면의 깊이(SCREEN_DEPTH), 근접 평면(SCREEN_NEAR) 등의 파라미터를 전달한다.
- 초기화가 실패하면 메시지 박스를 띄우고 false를 반환한다.
- 초기화가 성공하면 true를 반환한다.
###### XXXXXX 메서드 Shutdown()
- **Direct3D 객체 해제**
    - `m_Direct3D`가 존재하는지 확인한다.
    - 존재한다면, `m_Direct3D`의 `Shutdown` 메서드를 호출하여 관련 리소스를 해제한다.
    - 이후 `m_Direct3D` 객체를 삭제하고, 포인터를 `0`(nullptr)으로 초기화하여 메모리 접근 오류를 방지한다.
###### XXXXXX 메서드 Frame()
- **변수 선언**
    - `bool` 타입의 지역 변수 `result`가 선언된다.
- **그래픽 렌더링 수행**
    - `Render()` 메서드를 호출하고, 그 반환 값을 `result` 변수에 저장한다.
- **렌더링 결과 검증**
    - `result` 값이 `false`이면, `Frame()` 메서드는 `false`를 반환하고 종료된다.
- **정상적인 흐름에서의 반환**
    - 모든 과정이 성공적으로 완료되면, `Frame()` 메서드는 `true`를 반환한다.
###### XXXXXX 메서드 
- **버퍼 초기화 및 장면 준비**
    - `BeginScene(float, float, float, float)`를 호출하여 화면을 특정 색상(회색, RGBA: 0.5, 0.5, 0.5, 1.0)으로 초기화하고, 렌더링을 위한 버퍼를 정리한다.
- **장면 출력**
    - `EndScene()`을 호출하여 렌더링이 완료된 장면을 화면에 출력한다.
- **정상 종료 반환**
    - 렌더링이 정상적으로 수행되었음을 나타내기 위해 `true` 값을 반환한다.
###### XXXXXX 메서드 Render()
##### XXXXX 원문 번역
이전 튜토리얼에서 기억하시겠지만 이 클래스는 전혀 비어 있었고 코드도 없었습니다. 이제 D3DClass 멤버가 있으므로 ApplicationClass 내부에 코드를 채워 D3DClass 객체를 초기화하고 종료합니다. 또한 Render 함수에서 BeginScene과 EndScene에 대한 호출을 추가하여 Direct3D를 사용하여 창에 그리게 됩니다.

그래서, 가장 첫 번째 변경 사항은 클래스 생성자에 있습니다. 여기서 우리는 모든 클래스 포인터에서 하는 것처럼 안전상의 이유로 포인터를 null로 초기화합니다.

두 번째 변경 사항은 ApplicationClass 내부의 Initialize 함수에 있습니다. 여기서 D3DClass 객체를 생성한 다음 D3DClass Initialize 함수를 호출합니다. 이 함수에 화면 너비, 화면 높이, 창 핸들, Applicationclass.h 파일의 네 가지 전역 변수를 보냅니다. D3DClass는 이러한 모든 변수를 사용하여 Direct3D 시스템을 설정합니다. d3dclass.cpp 파일을 살펴보면 이에 대해 더 자세히 설명하겠습니다.

다음 변경 사항은 ApplicationClass의 Shutdown 함수에 있습니다. 모든 그래픽 객체의 종료가 여기서 발생하므로 이 함수에 D3DClass 종료를 배치했습니다. 포인터가 초기화되었는지 여부를 확인합니다. 초기화되지 않았다면 결코 설정되지 않았다고 가정하고 종료하려고 하지 않아도 됩니다. 그렇기 때문에 클래스 생성자에서 모든 포인터를 null로 설정하는 것이 중요합니다. 포인터가 초기화되었음을 발견하면 D3DClass를 종료하려고 시도한 다음 포인터를 정리합니다.

Frame 함수가 업데이트되어 이제 각 프레임에서 Render 함수를 호출합니다.

이 클래스의 마지막 변경 사항은 Render 함수에 있습니다. m_Direct3D 객체를 호출하여 화면을 회색으로 지웁니다. 그런 다음 EndScene을 호출하여 회색이 창에 표시됩니다.
##### XXXXX .cpp 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: applicationclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "applicationclass.h"

ApplicationClass::ApplicationClass()
{
	m_Direct3D = 0;
}


ApplicationClass::ApplicationClass(const ApplicationClass& other)
{
}


ApplicationClass::~ApplicationClass()
{
}

bool ApplicationClass::Initialize(int screenWidth, int screenHeight, HWND hwnd)
{
	bool result;


	// Create and initialize the Direct3D object.
	m_Direct3D = new D3DClass;

	result = m_Direct3D->Initialize(screenWidth, screenHeight, VSYNC_ENABLED, hwnd, FULL_SCREEN, SCREEN_DEPTH, SCREEN_NEAR);
	if(!result)
	{
		MessageBox(hwnd, L"Could not initialize Direct3D", L"Error", MB_OK);
		return false;
	}

	return true;
}

void ApplicationClass::Shutdown()
{
	// Release the Direct3D object.
	if(m_Direct3D)
	{
		m_Direct3D->Shutdown();
		delete m_Direct3D;
		m_Direct3D = 0;
	}

	return;
}

bool ApplicationClass::Frame()
{
	bool result;


	// Render the graphics scene.
	result = Render();
	if(!result)
	{
		return false;
	}

	return true;
}

bool ApplicationClass::Render()
{
	// Clear the buffers to begin the scene.
	m_Direct3D->BeginScene(0.5f, 0.5f, 0.5f, 1.0f);


	// Present the rendered scene to the screen.
	m_Direct3D->EndScene();

	return true;
}
```

### XXX 클래스 D3dclass
#### XXXX 클래스 헤더
##### XXXXX 클래스 역할
**D3DClass의 역할**
`d3dclass.h` 파일을 기반으로 분석하면, `D3DClass`는 Direct3D 11을 사용하여 그래픽을 렌더링하는 역할을 수행합니다. 주요 기능은 다음과 같습니다.
- Direct3D 장치 및 컨텍스트 초기화 (`ID3D11Device`, `ID3D11DeviceContext` 등).
- 화면 버퍼 설정 및 관리 (`IDXGISwapChain` 등).
- 렌더링 과정 (`BeginScene`, `EndScene`).
- 카메라 및 변환 행렬 관리 (`GetProjectionMatrix`, `GetWorldMatrix`, `GetOrthoMatrix`).
- 그래픽 카드 정보 조회 (`GetVideoCardInfo`).
- 뷰포트 및 백버퍼 설정 (`SetBackBufferRenderTarget`, `ResetViewport`).
**전체적인 프레임워크 역할**
- `SystemClass`가 애플리케이션의 실행을 관리하며, `InputClass`가 키보드/마우스 입력을 처리.
- `ApplicationClass`는 주요 게임 또는 그래픽 애플리케이션 로직을 담당.
- `D3DClass`는 Direct3D 렌더링을 지원하여 화면에 그래픽을 출력.
##### XXXXX 메서드 목록
D3DClass();
D3DClass(const D3DClass&);
~D3DClass();
bool Initialize(int, int, bool, HWND, bool, float, float);
void Shutdown();
void BeginScene(float, float, float, float);
void EndScene();
ID3D11Device* GetDevice();
ID3D11DeviceContext* GetDeviceContext();
void GetProjectionMatrix(XMMATRIX&);
void GetWorldMatrix(XMMATRIX&);
void GetOrthoMatrix(XMMATRIX&);
void GetVideoCardInfo(char*, int&);
void SetBackBufferRenderTarget();
void ResetViewport();
##### XXXXX 원문 번역
헤더에서 첫 번째는 이 객체 모듈을 사용할 때 연결할 라이브러리를 지정하는 것입니다. 첫 번째 라이브러리에는 DirectX 11에서 3D 그래픽을 설정하고 그리기 위한 모든 Direct3D 기능이 포함되어 있습니다. 두 번째 라이브러리에는 모니터의 재생 빈도, 사용 중인 비디오 카드 등에 대한 정보를 얻기 위해 컴퓨터의 하드웨어와 인터페이스하는 도구가 포함되어 있습니다. 세 번째 라이브러리에는 다음 튜토리얼에서 다룰 셰이더를 컴파일하는 기능이 포함되어 있습니다.

다음으로 할 일은 이 개체 모듈에 링크하는 라이브러리에 대한 헤더와 DirectX 유형 정의 및 수학 기능에 대한 헤더를 포함시키는 것입니다.

D3DClass의 클래스 정의는 여기서 가능한 한 간단하게 유지됩니다. 일반 생성자, 복사 생성자 및 소멸자가 있습니다. 그리고 더 중요한 것은 Initialize 및 Shutdown 함수가 있습니다. 이것이 이 튜토리얼에서 주로 집중할 것입니다. 그 외에 이 튜토리얼에 중요하지 않은 몇 가지 도우미 함수와 d3dclass.cpp 파일을 살펴볼 때 살펴볼 여러 개인 멤버 변수가 있습니다. 지금은 Initialize 및 Shutdown 함수가 우리에게 중요한 것이라는 것을 깨달으십시오.

이미 Direct3D에 익숙한 사람이라면 이 클래스에 뷰 매트릭스 변수가 없다는 것을 알아차릴 수 있을 것입니다. 그 이유는 향후 튜토리얼에서 살펴볼 카메라 클래스에 넣을 것이기 때문입니다.
##### XXXXX .h 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: d3dclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _D3DCLASS_H_
#define _D3DCLASS_H_

/////////////
// LINKING //
/////////////
#pragma comment(lib, "d3d11.lib")
#pragma comment(lib, "dxgi.lib")
#pragma comment(lib, "d3dcompiler.lib")

//////////////
// INCLUDES //
//////////////
#include <d3d11.h>
#include <directxmath.h>
using namespace DirectX;

////////////////////////////////////////////////////////////////////////////////
// Class name: D3DClass
////////////////////////////////////////////////////////////////////////////////
class D3DClass
{
public:
    D3DClass();
    D3DClass(const D3DClass&);
    ~D3DClass();

    bool Initialize(int, int, bool, HWND, bool, float, float);
    void Shutdown();
	
    void BeginScene(float, float, float, float);
    void EndScene();

    ID3D11Device* GetDevice();
    ID3D11DeviceContext* GetDeviceContext();

    void GetProjectionMatrix(XMMATRIX&);
    void GetWorldMatrix(XMMATRIX&);
    void GetOrthoMatrix(XMMATRIX&);

    void GetVideoCardInfo(char*, int&);

    void SetBackBufferRenderTarget();
    void ResetViewport();

private:
    bool m_vsync_enabled;
    int m_videoCardMemory;
    char m_videoCardDescription[128];
    IDXGISwapChain* m_swapChain;
    ID3D11Device* m_device;
    ID3D11DeviceContext* m_deviceContext;
    ID3D11RenderTargetView* m_renderTargetView;
    ID3D11Texture2D* m_depthStencilBuffer;
    ID3D11DepthStencilState* m_depthStencilState;
    ID3D11DepthStencilView* m_depthStencilView;
    ID3D11RasterizerState* m_rasterState;
    XMMATRIX m_projectionMatrix;
    XMMATRIX m_worldMatrix;
    XMMATRIX m_orthoMatrix;
    D3D11_VIEWPORT m_viewport;
};

#endif
```
#### XXXX 클래스 구현
##### XXXXX 각 메서드 흐름
###### XXXXXX 메서드 D3DClass()
- Direct3D 스왑 체인(`m_swapChain`)을 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- Direct3D 디바이스(`m_device`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- Direct3D 디바이스 컨텍스트(`m_deviceContext`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- 렌더 타겟 뷰(`m_renderTargetView`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- 깊이 스텐실 버퍼(`m_depthStencilBuffer`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- 깊이 스텐실 상태(`m_depthStencilState`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- 깊이 스텐실 뷰(`m_depthStencilView`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
- 래스터라이저 상태(`m_rasterState`)를 초기화하여 사용하지 않는 상태(0)로 설정합니다.
###### XXXXXX 메서드 D3DClass(const D3DClass& other)
아무것도 하지 않음
###### XXXXXX 메서드 ~D3DClass()
아무것도 하지 않음
###### XXXXXX 메서드 Initialize(int screenWidth, int screenHeight, bool vsync, HWND hwnd, bool fullscreen, float screenDepth, float screenNear)
#### **1. 초기 설정**

- `vsync` 설정을 저장한다.
- DirectX 그래픽 인터페이스 팩토리를 생성한다.
- 기본 그래픽 어댑터(비디오 카드)를 생성한다.
- 어댑터의 출력을 열거하여 모니터 정보를 가져온다.

#### **2. 디스플레이 모드 및 비디오 카드 정보 설정**

- 어댑터 출력의 디스플레이 모드 목록을 가져온다.
- 현재 해상도(`screenWidth`, `screenHeight`)와 일치하는 디스플레이 모드를 찾아 새로 고침 빈도(분자/분모)를 저장한다.
- 비디오 카드 정보를 가져와 전용 비디오 메모리 크기를 저장한다.
- 비디오 카드 이름을 문자열로 변환하여 저장한다.
- 사용이 끝난 리소스(`displayModeList`, `adapterOutput`, `adapter`, `factory`)를 해제한다.

#### **3. 스왑 체인(Swap Chain) 생성**

- `DXGI_SWAP_CHAIN_DESC` 구조체를 초기화하고 스왑 체인 설정을 구성한다.
    - 백 버퍼 개수, 크기, 픽셀 형식, 새로 고침 빈도, 다중 샘플링, 창 핸들, 전체 화면 여부 등을 설정한다.
- Direct3D 11 디바이스, 디바이스 컨텍스트 및 스왑 체인을 생성한다.
- 백 버퍼를 가져와 렌더 타겟 뷰를 생성한 후, 백 버퍼를 해제한다.

#### **4. 깊이/스텐실 버퍼 및 뷰 설정**

- 깊이 버퍼(`D3D11_TEXTURE2D_DESC`)를 설정하고, 해당 텍스처를 생성한다.
- 깊이/스텐실 상태(`D3D11_DEPTH_STENCIL_DESC`)를 설정하고, 스텐실 상태 객체를 생성한 후 이를 디바이스 컨텍스트에 바인딩한다.
- 깊이/스텐실 뷰(`D3D11_DEPTH_STENCIL_VIEW_DESC`)를 설정하고, 해당 뷰를 생성한 후 렌더 타겟 뷰와 함께 바인딩한다.

#### **5. 래스터라이저(Rasterizer) 설정**

- `D3D11_RASTERIZER_DESC` 구조체를 초기화하고, 폴리곤을 어떻게 렌더링할지 설정한다.
- 래스터라이저 상태를 생성한 후, 디바이스 컨텍스트에 적용한다.

#### **6. 뷰포트(Viewport) 설정**

- `D3D11_VIEWPORT` 구조체를 초기화하고 화면 크기에 맞게 설정한 후, 디바이스 컨텍스트에 적용한다.

#### **7. 카메라 및 렌더링 행렬(Matrix) 설정**

- 3D 렌더링을 위한 원근 투영 행렬(`m_projectionMatrix`)을 생성한다.
- 월드 행렬(`m_worldMatrix`)을 단위 행렬로 초기화한다.
- 2D 렌더링을 위한 직교 투영 행렬(`m_orthoMatrix`)을 생성한다.

#### **8. 초기화 완료**

- 모든 초기화가 정상적으로 완료되었으면 `true`를 반환한다.
###### XXXXXX 메서드 Shutdown()
- **전체 화면 모드 해제**
    - `m_swapChain`이 존재하면 `SetFullscreenState(false, NULL)`을 호출하여 창 모드로 변경
- **그래픽 리소스 해제 (역순 처리)**
    - `m_rasterState`가 존재하면 `Release()`를 호출 후 `0`으로 설정
    - `m_depthStencilView`가 존재하면 `Release()`를 호출 후 `0`으로 설정
    - `m_depthStencilState`가 존재하면 `Release()`를 호출 후 `0`으로 설정
    - `m_depthStencilBuffer`가 존재하면 `Release()`를 호출 후 `0`으로 설정
    - `m_renderTargetView`가 존재하면 `Release()`를 호출 후 `0`으로 설정
- **Direct3D 인터페이스 해제**
    - `m_deviceContext`가 존재하면 `Release()`를 호출 후 `0`으로 설정
    - `m_device`가 존재하면 `Release()`를 호출 후 `0`으로 설정
    - `m_swapChain`이 존재하면 `Release()`를 호출 후 `0`으로 설정
###### XXXXXX 메서드 BeginScene(float red, float green, float blue, float alpha)
- **입력 값 설정:**
    - `red`, `green`, `blue`, `alpha` 값을 입력받음.
- **배열 생성 및 초기화:**
    - `color[4]` 배열을 선언함.
    - 배열에 `red`, `green`, `blue`, `alpha` 값을 차례로 저장함.
- **백 버퍼 초기화:**
    - `m_deviceContext->ClearRenderTargetView(m_renderTargetView, color)` 호출.
    - `color` 값을 사용하여 렌더 타겟 뷰를 초기화함.
- **깊이 버퍼 초기화:**
    - `m_deviceContext->ClearDepthStencilView(m_depthStencilView, D3D11_CLEAR_DEPTH, 1.0f, 0)` 호출.
    - 깊이 버퍼를 1.0f로, 스텐실 버퍼를 0으로 설정하여 초기화함.
###### XXXXXX 메서드 EndScene()
- **메서드 시작**: `D3DClass::EndScene()` 메서드가 호출된다.
- **화면 출력 준비**: 백 버퍼의 렌더링이 완료되었으므로 화면에 출력한다.
- **VSYNC 여부 확인**: `m_vsync_enabled` 값에 따라 두 가지 경로로 분기한다.
    - **VSYNC 활성화 (`m_vsync_enabled == true`)**:
        - 화면 주사율에 맞춰 렌더링을 동기화한다.
        - `m_swapChain->Present(1, 0);` 호출.
    - **VSYNC 비활성화 (`m_vsync_enabled == false`)**:
        - 가능한 한 빠르게 화면을 갱신한다.
        - `m_swapChain->Present(0, 0);` 호출.
###### XXXXXX 메서드 GetDevice()
- m_device를 반환
###### XXXXXX 메서드 GetDeviceContext()
- m_deviceContext를 반환
###### XXXXXX 메서드 GetProjectionMatrix(XMMATRIX& projectionMatrix)
- 매개변수 projectionMatrix를 m_projectionMatrix로 설정함.
###### XXXXXX 메서드 GetWorldMatrix(XMMATRIX& worldMatrix)
- 매개변수 worldMatrix를 m_worldMatrix로 설정함
###### XXXXXX 메서드 GetOrthoMatrix(XMMATRIX& orthoMatrix)
- 매개변수 orthoMatrix를 m_orthoMatrix로 설정함
###### XXXXXX 메서드 GetVideoCardInfo(char* cardName, int& memory)
- 그래픽 카드의 이름과 그래픽 카드의 메모리 양을 매개변수로 기록함.
###### XXXXXX 메서드 SetBackBufferRenderTarget()
- m_deviceContext 객체의 OMSetRenderTargets(1, &m_renderTargetView, m_depthStencilView)를 호출한다.
###### XXXXXX 메서드 ResetViewport()
- m_deviceContext 객체의 RSSetViewports(1, &m_viewport)를 호출한다.
##### XXXXX 원문 번역
그래서, 대부분의 클래스처럼 우리는 클래스 생성자에서 모든 멤버 포인터를 null로 초기화하는 것으로 시작합니다. 헤더 파일의 모든 포인터는 모두 여기에서 고려되었습니다.

Initialize 함수는 DirectX 11용 Direct3D의 전체 설정을 담당합니다. 여기에 필요한 모든 코드와 향후 튜토리얼을 용이하게 하는 몇 가지 추가 내용을 넣었습니다. 이를 단순화하고 일부 항목을 제거할 수도 있었지만 이 모든 내용을 이 함수에 전념하는 단일 튜토리얼에서 다루는 것이 더 나을 것입니다.

이 함수에 제공된 screenWidth 및 screenHeight 변수는 SystemClass에서 만든 창의 너비와 높이입니다. Direct3D는 이를 사용하여 초기화하고 동일한 창 크기를 사용합니다. hwnd 변수는 창에 대한 핸들입니다. Direct3D는 이전에 만든 창에 액세스하기 위해 이 핸들이 필요합니다. fullscreen 변수는 창 모드 또는 전체 화면에서 실행 중인지 여부입니다. Direct3D는 올바른 설정으로 창을 만들기 위해 이 변수도 필요합니다. screenDepth 및 screenNear 변수는 창에서 렌더링될 3D 환경에 대한 깊이 설정입니다. vsync 변수는 Direct3D가 사용자 모니터 새로 고침 빈도에 따라 렌더링할지 아니면 가능한 한 빨리 렌더링할지 여부를 나타냅니다.

Direct3D를 초기화하기 전에 비디오 카드/모니터에서 재생 빈도를 가져와야 합니다. 각 컴퓨터는 약간 다를 수 있으므로 해당 정보를 쿼리해야 합니다. 분자와 분모 값을 쿼리한 다음 설정 중에 DirectX에 전달하면 적절한 재생 빈도가 계산됩니다. 이렇게 하지 않고 재생 빈도를 모든 컴퓨터에 없는 기본값으로 설정하면 DirectX는 버퍼 플립 대신 blit을 수행하여 성능을 저하시키고 디버그 출력에 성가신 오류를 발생시킵니다.

이제 우리는 재생률에 대한 분자와 분모를 가지고 있습니다. 어댑터를 사용하여 검색하는 마지막 항목은 비디오 카드의 이름과 비디오 메모리의 양입니다.

이제 새로 고침 빈도의 분자와 분모와 비디오 카드 정보를 저장했으므로 해당 정보를 가져오는 데 사용된 구조체와 인터페이스를 해제할 수 있습니다.

이제 시스템에서 새로 고침 빈도를 얻었으므로 DirectX 초기화를 시작할 수 있습니다. 먼저 스왑 체인의 설명을 작성합니다. 스왑 체인은 그래픽이 그려질 프런트 및 백 버퍼입니다. 일반적으로 단일 백 버퍼를 사용하여 모든 그리기를 수행한 다음 프런트 버퍼로 스왑하여 사용자 화면에 표시합니다. 그래서 스왑 체인이라고 합니다.

스왑 체인 설명의 다음 부분은 새로 고침 빈도입니다. 새로 고침 빈도는 백 버퍼를 프런트 버퍼로 초당 몇 번 그리는지입니다. applicationclass.h 헤더에서 vsync가 true로 설정된 경우 새로 고침 빈도가 시스템 설정(예: 60hz)으로 고정됩니다. 즉, 초당 60회만 화면을 그립니다(시스템 새로 고침 빈도가 60보다 큰 경우 더 높음). 그러나 vsync를 false로 설정하면 초당 가능한 한 자주 화면을 그리지만 이로 인해 일부 시각적 아티팩트가 발생할 수 있습니다.

스왑 체인 설명을 설정한 후에는 기능 수준이라는 또 다른 변수를 설정해야 합니다. 이 변수는 DirectX에 사용할 버전을 알려줍니다. 여기서 기능 수준을 DirectX 11인 11.0으로 설정합니다. 여러 버전을 지원하거나 하위 하드웨어에서 실행하려는 경우 DirectX의 하위 수준 버전을 사용하려면 이 값을 10 또는 9로 설정할 수 있습니다.

이제 스왑 체인 설명과 기능 수준이 채워졌으므로 스왑 체인, Direct3D 장치 및 Direct3D 장치 컨텍스트를 만들 수 있습니다. Direct3D 장치와 Direct3D 장치 컨텍스트는 매우 중요하며 모든 Direct3D 함수에 대한 인터페이스입니다. 이 시점부터는 거의 모든 것에 장치와 장치 컨텍스트를 사용합니다.

이전 버전의 DirectX에 익숙한 이 글을 읽고 있는 분들은 Direct3D 장치는 알아볼 수 있지만 새로운 Direct3D 장치 컨텍스트는 생소할 것입니다. 기본적으로 Direct3D 장치의 기능을 가져와 두 개의 다른 장치로 분할했으므로 이제 둘 다 사용해야 합니다.

사용자에게 DirectX 11 비디오 카드가 없는 경우 이 함수 호출은 장치와 장치 컨텍스트를 만들지 못합니다. 또한 DirectX 11 기능을 직접 테스트하고 DirectX 11 비디오 카드가 없는 경우 D3D_DRIVER_TYPE_HARDWARE를 D3D_DRIVER_TYPE_REFERENCE로 바꾸면 DirectX가 비디오 카드 하드웨어 대신 CPU를 사용하여 그립니다. 이 방법은 속도가 1/1000이지만 모든 컴퓨터에 아직 DirectX 11 비디오 카드가 없는 사람들에게는 좋습니다.

때때로 기본 비디오 카드가 DirectX 11과 호환되지 않으면 이 장치를 만드는 호출이 실패합니다. 일부 머신은 기본 카드를 DirectX 10 비디오 카드로, 보조 카드를 DirectX 11 비디오 카드로 가질 수 있습니다. 또한 일부 하이브리드 그래픽 카드는 기본 카드를 저전력 Intel 카드로, 보조 카드를 고전력 Nvidia 카드로 사용하는 방식으로 작동합니다. 이를 해결하려면 기본 장치를 사용하지 않고 대신 머신의 모든 비디오 카드를 열거하고 사용자가 사용할 카드를 선택한 다음 장치를 만들 때 해당 카드를 지정해야 합니다.

이제 스왑 체인이 있으므로 백 버퍼에 대한 포인터를 가져온 다음 스왑 체인에 연결해야 합니다. CreateRenderTargetView 함수를 사용하여 백 버퍼를 스왑 체인에 연결합니다.

또한 깊이 버퍼 설명을 설정해야 합니다. 이를 사용하여 폴리곤이 3D 공간에서 제대로 렌더링될 수 있도록 깊이 버퍼를 만듭니다. 동시에 깊이 버퍼에 스텐실 버퍼를 첨부합니다. 스텐실 버퍼는 모션 블러, 볼륨 그림자 및 기타 효과와 같은 효과를 얻는 데 사용할 수 있습니다.

이제 해당 설명을 사용하여 깊이/스텐실 버퍼를 만듭니다. CreateTexture2D 함수를 사용하여 버퍼를 만드는 것을 알 수 있을 것입니다. 따라서 버퍼는 단지 2D 텍스처입니다. 그 이유는 폴리곤이 정렬되고 래스터화되면 이 2D 버퍼에서 색상이 있는 픽셀이 되기 때문입니다. 그런 다음 이 2D 버퍼가 화면에 그려집니다.

이제 깊이 스텐실 설명을 설정해야 합니다. 이를 통해 Direct3D가 각 픽셀에 대해 어떤 유형의 깊이 테스트를 수행할지 제어할 수 있습니다.

설명을 작성했으므로 이제 깊이 스텐실 상태를 만들 수 있습니다.

생성된 깊이 스텐실 상태를 사용하여 이제 적용되도록 설정할 수 있습니다. 장치 컨텍스트를 사용하여 설정합니다.

다음으로 만들어야 할 것은 깊이 스텐실 버퍼의 뷰에 대한 설명입니다. 이렇게 하는 이유는 Direct3D가 깊이 버퍼를 깊이 스텐실 텍스처로 사용하도록 하기 위해서입니다. 설명을 채운 후 CreateDepthStencilView 함수를 호출하여 이를 만듭니다.

이것을 만들었으므로 이제 OMSetRenderTargets를 호출할 수 있습니다. 이렇게 하면 렌더 타겟 뷰와 깊이 스텐실 버퍼가 출력 렌더 파이프라인에 바인딩됩니다. 이렇게 하면 파이프라인이 렌더링하는 그래픽이 이전에 만든 백 버퍼에 그려집니다. 그래픽이 백 버퍼에 기록되면 전면으로 스왑하고 사용자 화면에 그래픽을 표시할 수 있습니다.

이제 렌더 타겟이 설정되었으므로 향후 튜토리얼을 위해 장면을 더 많이 제어할 수 있는 몇 가지 추가 기능을 계속할 수 있습니다. 먼저 래스터라이저 상태를 만들 것입니다. 이를 통해 폴리곤이 렌더링되는 방식을 제어할 수 있습니다. 장면을 와이어프레임 모드로 렌더링하거나 DirectX에서 폴리곤의 앞면과 뒷면을 모두 그리도록 할 수 있습니다. 기본적으로 DirectX에는 이미 래스터라이저 상태가 설정되어 아래와 정확히 동일하게 작동하지만 직접 설정하지 않는 한 변경할 수 있는 제어권이 없습니다.

Direct3D가 클립 공간 좌표를 렌더 대상 공간에 매핑할 수 있도록 뷰포트도 설정해야 합니다. 이것을 창의 전체 크기로 설정합니다.

이제 투영 행렬을 만들 것입니다. 투영 행렬은 3D 장면을 이전에 만든 2D 뷰포트 공간으로 변환하는 데 사용됩니다. 장면을 렌더링하는 데 사용될 셰이더에 전달할 수 있도록 이 행렬의 사본을 보관해야 합니다.

또한 월드 매트릭스라는 또 다른 매트릭스를 만들 것입니다. 이 매트릭스는 객체의 정점을 3D 장면의 정점으로 변환하는 데 사용됩니다. 이 매트릭스는 또한 3D 공간에서 객체를 회전, 이동 및 크기 조정하는 데 사용됩니다. 처음부터 매트릭스를 단위 매트릭스로 초기화하고 이 객체에 복사본을 보관합니다. 복사본은 렌더링을 위해 셰이더에 전달되어야 합니다.

여기서 일반적으로 뷰 행렬을 만듭니다. 뷰 행렬은 장면을 바라보는 위치를 계산하는 데 사용됩니다. 카메라라고 생각하면 되고, 이 카메라를 통해서만 장면을 볼 수 있습니다. 목적 때문에 나중에 튜토리얼에서 카메라 클래스에 만들 것입니다. 논리적으로 더 잘 맞고 지금은 건너뛸 수 있기 때문입니다.

마지막으로 Initialize 함수에서 설정할 것은 정사 투영 행렬입니다. 이 행렬은 화면에 사용자 인터페이스와 같은 2D 요소를 렌더링하는 데 사용되므로 3D 렌더링을 건너뛸 수 있습니다. 나중에 튜토리얼에서 2D 그래픽과 글꼴을 화면에 렌더링하는 방법을 살펴볼 때 이 행렬이 사용되는 것을 볼 수 있습니다.



Shutdown 함수는 Initialize 함수에서 사용된 모든 포인터를 해제하고 정리합니다. 꽤 간단합니다. 하지만 그렇게 하기 전에 포인터를 해제하기 전에 스왑 체인을 창 모드로 강제로 전환하는 호출을 넣었습니다. 이렇게 하지 않고 전체 화면 모드에서 스왑 체인을 해제하려고 하면 몇 가지 예외가 발생합니다. 따라서 이런 일이 발생하지 않도록 Direct3D를 종료하기 전에 항상 창 모드를 강제로 실행합니다.



D3DClass에는 몇 가지 도우미 함수가 있습니다. 처음 두 개는 BeginScene과 EndScene입니다. BeginScene은 각 프레임의 시작 부분에서 새 3D 장면을 그릴 때마다 호출됩니다. 하는 일은 버퍼를 초기화하여 비어 있고 그릴 준비가 되도록 하는 것입니다. 다른 함수는 Endscene으로, 모든 그리기가 각 프레임의 끝에서 완료되면 스왑 체인에 3D 장면을 표시하라고 지시합니다.



다음 함수는 단순히 Direct3D 장치와 Direct3D 장치 컨텍스트에 대한 포인터를 가져옵니다. 이러한 도우미 함수는 프레임워크에서 자주 호출됩니다.



다음 세 개의 헬퍼 함수는 프로젝션, 월드, 오쏘그래픽 행렬의 사본을 호출 함수에 제공합니다. 대부분의 셰이더는 렌더링을 위해 이러한 행렬이 필요하므로 외부 객체가 이러한 행렬의 사본을 쉽게 얻을 수 있는 방법이 필요했습니다. 이 튜토리얼에서는 이러한 함수를 호출하지 않지만 코드에 있는 이유를 설명하려고 합니다.



이 헬퍼 함수는 비디오 카드 이름과 비디오 메모리 양을 참조로 반환합니다. 비디오 카드 이름을 아는 것은 다양한 구성에서 디버깅하는 데 도움이 될 수 있습니다.



마지막 두 개의 도우미 함수는 나중에 렌더 투 텍스처 튜토리얼에서 사용될 예정입니다.
##### XXXXX .cpp 코드
```C++
////////////////////////////////////////////////////////////////////////////////
// Filename: d3dclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "d3dclass.h"

D3DClass::D3DClass()
{
	m_swapChain = 0;
	m_device = 0;
	m_deviceContext = 0;
	m_renderTargetView = 0;
	m_depthStencilBuffer = 0;
	m_depthStencilState = 0;
	m_depthStencilView = 0;
	m_rasterState = 0;
}

D3DClass::D3DClass(const D3DClass& other)
{
}

D3DClass::~D3DClass()
{
}

bool D3DClass::Initialize(int screenWidth, int screenHeight, bool vsync, HWND hwnd, bool fullscreen, float screenDepth, float screenNear)
{
	HRESULT result;
	IDXGIFactory* factory;
	IDXGIAdapter* adapter;
	IDXGIOutput* adapterOutput;
	unsigned int numModes, i, numerator, denominator;
	unsigned long long stringLength;
	DXGI_MODE_DESC* displayModeList;
	DXGI_ADAPTER_DESC adapterDesc;
	int error;
	DXGI_SWAP_CHAIN_DESC swapChainDesc;
	D3D_FEATURE_LEVEL featureLevel;
	ID3D11Texture2D* backBufferPtr;
	D3D11_TEXTURE2D_DESC depthBufferDesc;
	D3D11_DEPTH_STENCIL_DESC depthStencilDesc;
	D3D11_DEPTH_STENCIL_VIEW_DESC depthStencilViewDesc;
	D3D11_RASTERIZER_DESC rasterDesc;
	float fieldOfView, screenAspect;


	// Store the vsync setting.
	m_vsync_enabled = vsync;

// Create a DirectX graphics interface factory.
	result = CreateDXGIFactory(__uuidof(IDXGIFactory), (void**)&factory);
	if(FAILED(result))
	{
		return false;
	}

	// Use the factory to create an adapter for the primary graphics interface (video card).
	result = factory->EnumAdapters(0, &adapter);
	if(FAILED(result))
	{
		return false;
	}

	// Enumerate the primary adapter output (monitor).
	result = adapter->EnumOutputs(0, &adapterOutput);
	if(FAILED(result))
	{
		return false;
	}

	// Get the number of modes that fit the DXGI_FORMAT_R8G8B8A8_UNORM display format for the adapter output (monitor).
	result = adapterOutput->GetDisplayModeList(DXGI_FORMAT_R8G8B8A8_UNORM, DXGI_ENUM_MODES_INTERLACED, &numModes, NULL);
	if(FAILED(result))
	{
		return false;
	}

	// Create a list to hold all the possible display modes for this monitor/video card combination.
	displayModeList = new DXGI_MODE_DESC[numModes];
	if(!displayModeList)
	{
		return false;
	}

	// Now fill the display mode list structures.
	result = adapterOutput->GetDisplayModeList(DXGI_FORMAT_R8G8B8A8_UNORM, DXGI_ENUM_MODES_INTERLACED, &numModes, displayModeList);
	if(FAILED(result))
	{
		return false;
	}

	// Now go through all the display modes and find the one that matches the screen width and height.
	// When a match is found store the numerator and denominator of the refresh rate for that monitor.
	for(i=0; i<numModes; i++)
	{
		if(displayModeList[i].Width == (unsigned int)screenWidth)
		{
			if(displayModeList[i].Height == (unsigned int)screenHeight)
			{
				numerator = displayModeList[i].RefreshRate.Numerator;
				denominator = displayModeList[i].RefreshRate.Denominator;
			}
		}
	}
// Get the adapter (video card) description.
	result = adapter->GetDesc(&adapterDesc);
	if(FAILED(result))
	{
		return false;
	}

	// Store the dedicated video card memory in megabytes.
	m_videoCardMemory = (int)(adapterDesc.DedicatedVideoMemory / 1024 / 1024);

	// Convert the name of the video card to a character array and store it.
	error = wcstombs_s(&stringLength, m_videoCardDescription, 128, adapterDesc.Description, 128);
	if(error != 0)
	{
		return false;
	}
	// Release the display mode list.
	delete [] displayModeList;
	displayModeList = 0;

	// Release the adapter output.
	adapterOutput->Release();
	adapterOutput = 0;

	// Release the adapter.
	adapter->Release();
	adapter = 0;

	// Release the factory.
	factory->Release();
	factory = 0;
	
	// Initialize the swap chain description.
	ZeroMemory(&swapChainDesc, sizeof(swapChainDesc));

	// Set to a single back buffer.
	swapChainDesc.BufferCount = 1;

	// Set the width and height of the back buffer.
	swapChainDesc.BufferDesc.Width = screenWidth;
	swapChainDesc.BufferDesc.Height = screenHeight;

	// Set regular 32-bit surface for the back buffer.
	swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;

	// Set the refresh rate of the back buffer.
	if(m_vsync_enabled)
	{
		swapChainDesc.BufferDesc.RefreshRate.Numerator = numerator;
		swapChainDesc.BufferDesc.RefreshRate.Denominator = denominator;
	}
	else
	{
		swapChainDesc.BufferDesc.RefreshRate.Numerator = 0;
		swapChainDesc.BufferDesc.RefreshRate.Denominator = 1;
	}

	// Set the usage of the back buffer.
	swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;

	// Set the handle for the window to render to.
	swapChainDesc.OutputWindow = hwnd;

	// Turn multisampling off.
	swapChainDesc.SampleDesc.Count = 1;
	swapChainDesc.SampleDesc.Quality = 0;

	// Set to full screen or windowed mode.
	if(fullscreen)
	{
		swapChainDesc.Windowed = false;
	}
	else
	{
		swapChainDesc.Windowed = true;
	}

	// Set the scan line ordering and scaling to unspecified.
	swapChainDesc.BufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED;
	swapChainDesc.BufferDesc.Scaling = DXGI_MODE_SCALING_UNSPECIFIED;

	// Discard the back buffer contents after presenting.
	swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;

	// Don't set the advanced flags.
	swapChainDesc.Flags = 0;

	// Set the feature level to DirectX 11.
	featureLevel = D3D_FEATURE_LEVEL_11_0;

	// Create the swap chain, Direct3D device, and Direct3D device context.
	result = D3D11CreateDeviceAndSwapChain(NULL, D3D_DRIVER_TYPE_HARDWARE, NULL, 0, &featureLevel, 1, 
					       D3D11_SDK_VERSION, &swapChainDesc, &m_swapChain, &m_device, NULL, &m_deviceContext);
	if(FAILED(result))
	{
		return false;
	}

	// Get the pointer to the back buffer.
	result = m_swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (LPVOID*)&backBufferPtr);
	if(FAILED(result))
	{
		return false;
	}

	// Create the render target view with the back buffer pointer.
	result = m_device->CreateRenderTargetView(backBufferPtr, NULL, &m_renderTargetView);
	if(FAILED(result))
	{
		return false;
	}

	// Release pointer to the back buffer as we no longer need it.
	backBufferPtr->Release();
	backBufferPtr = 0;

	// Initialize the description of the depth buffer.
	ZeroMemory(&depthBufferDesc, sizeof(depthBufferDesc));

	// Set up the description of the depth buffer.
	depthBufferDesc.Width = screenWidth;
	depthBufferDesc.Height = screenHeight;
	depthBufferDesc.MipLevels = 1;
	depthBufferDesc.ArraySize = 1;
	depthBufferDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
	depthBufferDesc.SampleDesc.Count = 1;
	depthBufferDesc.SampleDesc.Quality = 0;
	depthBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	depthBufferDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
	depthBufferDesc.CPUAccessFlags = 0;
	depthBufferDesc.MiscFlags = 0;

	// Create the texture for the depth buffer using the filled out description.
	result = m_device->CreateTexture2D(&depthBufferDesc, NULL, &m_depthStencilBuffer);
	if(FAILED(result))
	{
		return false;
	}

	// Initialize the description of the stencil state.
	ZeroMemory(&depthStencilDesc, sizeof(depthStencilDesc));

	// Set up the description of the stencil state.
	depthStencilDesc.DepthEnable = true;
	depthStencilDesc.DepthWriteMask = D3D11_DEPTH_WRITE_MASK_ALL;
	depthStencilDesc.DepthFunc = D3D11_COMPARISON_LESS;

	depthStencilDesc.StencilEnable = true;
	depthStencilDesc.StencilReadMask = 0xFF;
	depthStencilDesc.StencilWriteMask = 0xFF;

	// Stencil operations if pixel is front-facing.
	depthStencilDesc.FrontFace.StencilFailOp = D3D11_STENCIL_OP_KEEP;
	depthStencilDesc.FrontFace.StencilDepthFailOp = D3D11_STENCIL_OP_INCR;
	depthStencilDesc.FrontFace.StencilPassOp = D3D11_STENCIL_OP_KEEP;
	depthStencilDesc.FrontFace.StencilFunc = D3D11_COMPARISON_ALWAYS;

	// Stencil operations if pixel is back-facing.
	depthStencilDesc.BackFace.StencilFailOp = D3D11_STENCIL_OP_KEEP;
	depthStencilDesc.BackFace.StencilDepthFailOp = D3D11_STENCIL_OP_DECR;
	depthStencilDesc.BackFace.StencilPassOp = D3D11_STENCIL_OP_KEEP;
	depthStencilDesc.BackFace.StencilFunc = D3D11_COMPARISON_ALWAYS;

	// Create the depth stencil state.
	result = m_device->CreateDepthStencilState(&depthStencilDesc, &m_depthStencilState);
	if(FAILED(result))
	{
		return false;
	}

	// Set the depth stencil state.
	m_deviceContext->OMSetDepthStencilState(m_depthStencilState, 1);

	// Initialize the depth stencil view.
	ZeroMemory(&depthStencilViewDesc, sizeof(depthStencilViewDesc));

	// Set up the depth stencil view description.
	depthStencilViewDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
	depthStencilViewDesc.ViewDimension = D3D11_DSV_DIMENSION_TEXTURE2D;
	depthStencilViewDesc.Texture2D.MipSlice = 0;

	// Create the depth stencil view.
	result = m_device->CreateDepthStencilView(m_depthStencilBuffer, &depthStencilViewDesc, &m_depthStencilView);
	if(FAILED(result))
	{
		return false;
	}

	// Bind the render target view and depth stencil buffer to the output render pipeline.
	m_deviceContext->OMSetRenderTargets(1, &m_renderTargetView, m_depthStencilView);

	// Setup the raster description which will determine how and what polygons will be drawn.
	rasterDesc.AntialiasedLineEnable = false;
	rasterDesc.CullMode = D3D11_CULL_BACK;
	rasterDesc.DepthBias = 0;
	rasterDesc.DepthBiasClamp = 0.0f;
	rasterDesc.DepthClipEnable = true;
	rasterDesc.FillMode = D3D11_FILL_SOLID;
	rasterDesc.FrontCounterClockwise = false;
	rasterDesc.MultisampleEnable = false;
	rasterDesc.ScissorEnable = false;
	rasterDesc.SlopeScaledDepthBias = 0.0f;

	// Create the rasterizer state from the description we just filled out.
	result = m_device->CreateRasterizerState(&rasterDesc, &m_rasterState);
	if(FAILED(result))
	{
		return false;
	}

	// Now set the rasterizer state.
	m_deviceContext->RSSetState(m_rasterState);

	// Setup the viewport for rendering.
	m_viewport.Width = (float)screenWidth;
	m_viewport.Height = (float)screenHeight;
	m_viewport.MinDepth = 0.0f;
	m_viewport.MaxDepth = 1.0f;
	m_viewport.TopLeftX = 0.0f;
	m_viewport.TopLeftY = 0.0f;

	// Create the viewport.
	m_deviceContext->RSSetViewports(1, &m_viewport);

	// Setup the projection matrix.
	fieldOfView = 3.141592654f / 4.0f;
	screenAspect = (float)screenWidth / (float)screenHeight;

	// Create the projection matrix for 3D rendering.
	m_projectionMatrix = XMMatrixPerspectiveFovLH(fieldOfView, screenAspect, screenNear, screenDepth);

	// Initialize the world matrix to the identity matrix.
	m_worldMatrix = XMMatrixIdentity();

	// Create an orthographic projection matrix for 2D rendering.
	m_orthoMatrix = XMMatrixOrthographicLH((float)screenWidth, (float)screenHeight, screenNear, screenDepth);

	return true;
}

void D3DClass::Shutdown()
{
	// Before shutting down set to windowed mode or when you release the swap chain it will throw an exception.
	if(m_swapChain)
	{
		m_swapChain->SetFullscreenState(false, NULL);
	}

	if(m_rasterState)
	{
		m_rasterState->Release();
		m_rasterState = 0;
	}

	if(m_depthStencilView)
	{
		m_depthStencilView->Release();
		m_depthStencilView = 0;
	}

	if(m_depthStencilState)
	{
		m_depthStencilState->Release();
		m_depthStencilState = 0;
	}

	if(m_depthStencilBuffer)
	{
		m_depthStencilBuffer->Release();
		m_depthStencilBuffer = 0;
	}

	if(m_renderTargetView)
	{
		m_renderTargetView->Release();
		m_renderTargetView = 0;
	}

	if(m_deviceContext)
	{
		m_deviceContext->Release();
		m_deviceContext = 0;
	}

	if(m_device)
	{
		m_device->Release();
		m_device = 0;
	}

	if(m_swapChain)
	{
		m_swapChain->Release();
		m_swapChain = 0;
	}

	return;
}

void D3DClass::BeginScene(float red, float green, float blue, float alpha)
{
	float color[4];


	// Setup the color to clear the buffer to.
	color[0] = red;
	color[1] = green;
	color[2] = blue;
	color[3] = alpha;

	// Clear the back buffer.
	m_deviceContext->ClearRenderTargetView(m_renderTargetView, color);
    
	// Clear the depth buffer.
	m_deviceContext->ClearDepthStencilView(m_depthStencilView, D3D11_CLEAR_DEPTH, 1.0f, 0);

	return;
}


void D3DClass::EndScene()
{
	// Present the back buffer to the screen since rendering is complete.
	if(m_vsync_enabled)
	{
		// Lock to screen refresh rate.
		m_swapChain->Present(1, 0);
	}
	else
	{
		// Present as fast as possible.
		m_swapChain->Present(0, 0);
	}

	return;
}

ID3D11Device* D3DClass::GetDevice()
{
	return m_device;
}


ID3D11DeviceContext* D3DClass::GetDeviceContext()
{
	return m_deviceContext;
}

void D3DClass::GetProjectionMatrix(XMMATRIX& projectionMatrix)
{
	projectionMatrix = m_projectionMatrix;
	return;
}


void D3DClass::GetWorldMatrix(XMMATRIX& worldMatrix)
{
	worldMatrix = m_worldMatrix;
	return;
}

void D3DClass::GetVideoCardInfo(char* cardName, int& memory)
{
	strcpy_s(cardName, 128, m_videoCardDescription);
	memory = m_videoCardMemory;
	return;
}

void D3DClass::GetOrthoMatrix(XMMATRIX& orthoMatrix)
{
	orthoMatrix = m_orthoMatrix;
	return;
}

void D3DClass::SetBackBufferRenderTarget()
{
	// Bind the render target view and depth stencil buffer to the output render pipeline.
	m_deviceContext->OMSetRenderTargets(1, &m_renderTargetView, m_depthStencilView);

	return;
}


void D3DClass::ResetViewport()
{
	// Set the viewport.
	m_deviceContext->RSSetViewports(1, &m_viewport);

	return;
}
```




![[Pasted image 20250223192135.png]]
