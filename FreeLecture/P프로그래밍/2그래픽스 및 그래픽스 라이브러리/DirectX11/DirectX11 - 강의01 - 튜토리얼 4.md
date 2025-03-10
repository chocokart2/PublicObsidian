

[[DirectX11 - 버텍스 버퍼]]
[[DirectX11 - 인덱스 버퍼]]
[[DirectX11 - 버텍스 셰이더]]
[[DirectX11 - 픽셀 셰이더]]
[[DirectX11 - HLSL]]
![[pic6005.gif]]

이 튜토리얼을 위해 프레임워크가 업데이트되었습니다. ApplicationClass 아래에 CameraClass, ModelClass, ColorShaderClass라는 세 개의 새로운 클래스를 추가했습니다. CameraClass는 앞서 설명한 뷰 매트릭스를 처리합니다. 월드에서 카메라의 위치를 처리하여 셰이더가 씬을 어디에서 보고 있는지 파악하고 그릴 필요가 있을 때 셰이더에 전달합니다. ModelClass는 3D 모델의 지오메트리를 처리하며, 이 튜토리얼에서는 단순성을 위해 3D 모델을 하나의 삼각형으로만 처리합니다. 마지막으로 ColorShaderClass는 HLSL 셰이더를 호출하여 모델을 화면에 렌더링하는 작업을 담당합니다.

튜토리얼 코드는 먼저 HLSL 셰이더 프로그램을 살펴보는 것으로 시작하겠습니다.

### Color.vs
이것이 첫 번째 셰이더 프로그램이 될 것입니다. 셰이더는 모델의 실제 렌더링을 수행하는 작은 프로그램입니다. 이 셰이더는 HLSL로 작성되어 color.vs 및 color.ps라는 소스 파일에 저장됩니다. 지금은 .cpp 및 .h 파일과 함께 엔진에 파일을 배치했습니다. Visual studio에서 새 필터/폴더를 만들어야 한다는 점에 유의하세요. 그리고 셰이더를 마우스 오른쪽 버튼으로 클릭하고 “속성”을 선택하면 팝업에서 콘텐츠 섹션에 아무것도 없어야 하고 항목 유형 섹션에 “빌드에 참여하지 않음”이라고 표시되어야 하며 그렇지 않으면 주 진입점에 대한 컴파일 오류가 발생합니다.

이제 이 셰이더의 목적은 이 첫 번째 HLSL 튜토리얼에서 가능한 한 간단하게 유지하기 위해 색칠된 삼각형을 그리는 것입니다. 다음은 정점 셰이더의 코드입니다.

셰이더 프로그램에서는 전역 변수로 시작합니다. 이러한 전역 변수는 C++ 코드에서 외부적으로 수정할 수 있습니다. int나 float와 같은 여러 유형의 변수를 사용한 다음 셰이더 프로그램에서 사용하도록 외부적으로 설정할 수 있습니다. 일반적으로 단일 전역 변수일지라도 대부분의 전역 변수를 "cbuffer"라는 버퍼 객체 유형에 넣습니다. 이러한 버퍼를 논리적으로 구성하는 것은 셰이더의 효율적인 실행과 그래픽 카드가 버퍼를 저장하는 방법에 중요합니다. 이 예에서는 각 프레임에서 동시에 업데이트할 것이므로 세 개의 행렬을 동일한 버퍼에 넣었습니다.

C와 비슷하게 우리는 우리만의 유형 정의를 만들 수 있습니다. 우리는 HLSL에서 사용할 수 있는 float4와 같은 다양한 유형을 사용할 것입니다. 이는 셰이더 프로그래밍을 더 쉽고 읽기 쉽게 만듭니다. 이 예에서 우리는 x, y, z, w 위치 벡터와 red, green, blue, alpha 색상을 갖는 유형을 만듭니다. POSITION, COLOR, SV_POSITION은 GPU에 변수의 사용을 전달하는 의미론입니다. 구조가 다른 점은 같지만 정점 셰이더와 픽셀 셰이더의 의미론이 다르기 때문에 여기서 두 개의 다른 구조를 만들어야 합니다. POSITION은 정점 셰이더에서 작동하고 SV_POSITION은 픽셀 셰이더에서 작동하는 반면 COLOR는 두 가지 모두에서 작동합니다. 같은 유형을 두 개 이상 원하면 COLOR0, COLOR1 등과 같이 끝에 숫자를 추가해야 합니다.

정점 셰이더는 GPU가 자신에게 전송된 정점 버퍼의 데이터를 처리할 때 호출됩니다.

ColorVertexShader라는 이름의 이 정점 셰이더는 정점 버퍼의 모든 정점에 대해 호출됩니다. 정점 셰이더에 대한 입력은 정점 버퍼의 데이터 형식과 셰이더 소스 파일의 유형 정의와 일치해야 합니다. 이 경우 VertexInputType입니다. 정점 셰이더의 출력은 픽셀 셰이더로 전송됩니다. 이 경우 출력 유형은 PixelInputType이라고 하며 위에서도 정의되어 있습니다.

이를 염두에 두면 정점 셰이더가 PixelInputType 유형의 출력 변수를 생성한다는 것을 알 수 있습니다. 그런 다음 입력 정점의 위치를 ​​가져와서 월드, 뷰, 투영 행렬에 곱합니다. 이렇게 하면 뷰에 따라 3D 공간에서 렌더링할 올바른 위치에 정점이 배치되고 2D 화면에 배치됩니다. 그런 다음 출력 변수는 입력 색상의 사본을 가져온 다음 픽셀 셰이더에 대한 입력으로 사용될 출력을 반환합니다. 또한 입력 위치의 W 값을 1.0으로 설정한다는 점에 유의하세요. 그렇지 않으면 위치에 대해 XYZ 벡터만 읽기 때문에 정의되지 않습니다.

``` vs
////////////////////////////////////////////////////////////////////////////////
// Filename: color.vs
////////////////////////////////////////////////////////////////////////////////

/////////////
// GLOBALS //
/////////////
cbuffer MatrixBuffer
{
    matrix worldMatrix;
    matrix viewMatrix;
    matrix projectionMatrix;
};

//////////////
// TYPEDEFS //
//////////////
struct VertexInputType
{
    float4 position : POSITION;
    float4 color : COLOR;
};

struct PixelInputType
{
    float4 position : SV_POSITION;
    float4 color : COLOR;
};

////////////////////////////////////////////////////////////////////////////////
// Vertex Shader
////////////////////////////////////////////////////////////////////////////////
PixelInputType ColorVertexShader(VertexInputType input)
{
    PixelInputType output;
    

    // Change the position vector to be 4 units for proper matrix calculations.
    input.position.w = 1.0f;

    // Calculate the position of the vertex against the world, view, and projection matrices.
    output.position = mul(input.position, worldMatrix);
    output.position = mul(output.position, viewMatrix);
    output.position = mul(output.position, projectionMatrix);
    
    // Store the input color for the pixel shader to use.
    output.color = input.color;
    
    return output;
}
```

### Color.ps
픽셀 셰이더는 화면에 렌더링될 폴리곤의 각 픽셀을 그립니다. 이 픽셀 셰이더에서는 PixelInputType을 입력으로 사용하고 최종 픽셀 색상을 나타내는 float4를 출력으로 반환합니다. 이 픽셀 셰이더 프로그램은 매우 간단합니다. 색상의 입력 값과 동일하게 픽셀을 색칠하라고만 하면 됩니다. 픽셀 셰이더는 정점 셰이더 출력에서 ​​입력을 받는다는 점에 유의하세요.
```
////////////////////////////////////////////////////////////////////////////////
// Filename: color.ps
////////////////////////////////////////////////////////////////////////////////


//////////////
// TYPEDEFS //
//////////////
struct PixelInputType
{
    float4 position : SV_POSITION;
    float4 color : COLOR;
};


////////////////////////////////////////////////////////////////////////////////
// Pixel Shader
////////////////////////////////////////////////////////////////////////////////
float4 ColorPixelShader(PixelInputType input) : SV_TARGET
{
    return input.color;
}

```

Colorshaderclass.h
ColorShaderClass는 GPU에 있는 3D 모델을 그리기 위해 HLSL 셰이더를 호출하는 데 사용됩니다.

다음은 버텍스 셰이더와 함께 사용될 cBuffer 유형의 정의입니다. 이 typedef는 모델 데이터가 적절한 렌더링을 위해 셰이더의 typedef와 일치해야 하므로 버텍스 셰이더의 typedef와 정확히 동일해야 합니다.

여기의 함수는 셰이더의 초기화와 종료를 처리합니다. 렌더 함수는 셰이더 매개변수를 설정한 다음 셰이더를 사용하여 준비된 모델 정점을 그립니다.

```c
////////////////////////////////////////////////////////////////////////////////
// Filename: colorshaderclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _COLORSHADERCLASS_H_
#define _COLORSHADERCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <d3d11.h>
#include <d3dcompiler.h>
#include <directxmath.h>
#include <fstream>
using namespace DirectX;
using namespace std;


////////////////////////////////////////////////////////////////////////////////
// Class name: ColorShaderClass
////////////////////////////////////////////////////////////////////////////////
class ColorShaderClass
{
private:
	struct MatrixBufferType
	{
		XMMATRIX world;
		XMMATRIX view;
		XMMATRIX projection;
	};

public:
	ColorShaderClass();
	ColorShaderClass(const ColorShaderClass&);
	~ColorShaderClass();
	bool Initialize(ID3D11Device*, HWND);
	void Shutdown();
	bool Render(ID3D11DeviceContext*, int, XMMATRIX, XMMATRIX, XMMATRIX);

private:
	bool InitializeShader(ID3D11Device*, HWND, WCHAR*, WCHAR*);
	void ShutdownShader();
	void OutputShaderErrorMessage(ID3D10Blob*, HWND, WCHAR*);

	bool SetShaderParameters(ID3D11DeviceContext*, XMMATRIX, XMMATRIX, XMMATRIX);
	void RenderShader(ID3D11DeviceContext*, int);

private:
	ID3D11VertexShader* m_vertexShader;
	ID3D11PixelShader* m_pixelShader;
	ID3D11InputLayout* m_layout;
	ID3D11Buffer* m_matrixBuffer;
};

#endif
```


Colorshaderclass.cpp
평소처럼 클래스 생성자는 클래스의 모든 개인 포인터를 null로 초기화합니다.


Initialize 함수는 셰이더에 대한 초기화 함수를 호출합니다. HLSL 셰이더 파일의 이름을 전달합니다. 이 튜토리얼에서는 color.vs와 color.ps라는 이름을 사용합니다.


Shutdown 함수는 셰이더의 종료를 호출합니다.


Render는 먼저 SetShaderParameters 함수를 사용하여 셰이더 내부의 매개변수를 설정합니다. 매개변수가 설정되면 RenderShader를 호출하여 HLSL 셰이더를 사용하여 녹색 삼각형을 그립니다.


이제 InitializeShader라는 이 튜토리얼의 가장 중요한 함수 중 하나로 시작하겠습니다. 이 함수는 실제로 셰이더 파일을 로드하여 DirectX와 GPU에서 사용할 수 있게 합니다. 또한 레이아웃의 설정과 정점 버퍼 데이터가 GPU의 그래픽 파이프라인에서 어떻게 보일지도 볼 수 있습니다. 레이아웃은 modelclass.h 파일의 VertexType과 color.vs 파일에 정의된 VertexType과 일치해야 합니다.

여기서 셰이더 프로그램을 버퍼로 컴파일합니다. 셰이더 파일 이름, 셰이더 이름, 셰이더 버전(DirectX 11의 경우 5.0), 셰이더를 컴파일할 버퍼를 지정합니다. 셰이더 컴파일에 실패하면 errorMessage 문자열에 오류 메시지를 넣고 다른 함수로 보내 오류를 씁니다. 그래도 실패하고 errorMessage 문자열이 없으면 셰이더 파일을 찾을 수 없다는 의미이고, 그럴 경우 대화 상자를 띄워서 해당 사실을 알려줍니다.

정점 셰이더와 픽셀 셰이더 코드가 버퍼로 성공적으로 컴파일되면, 우리는 그 버퍼를 사용하여 셰이더 객체 자체를 만듭니다. 우리는 이 지점부터 정점 및 픽셀 셰이더와 인터페이스하기 위해 이 포인터를 사용할 것입니다.

다음 단계는 셰이더에서 처리할 정점 데이터의 레이아웃을 만드는 것입니다. 이 셰이더는 위치 및 색상 벡터를 사용하므로 레이아웃에서 둘 다 만들고 둘 다의 크기를 지정해야 합니다. 의미적 이름은 레이아웃에서 가장 먼저 채워야 하는 것으로, 이를 통해 셰이더가 레이아웃의 이 요소 사용을 결정할 수 있습니다. 두 가지 다른 요소가 있으므로 첫 번째 요소에는 POSITION을 사용하고 두 번째 요소에는 COLOR를 사용합니다. 레이아웃의 다음으로 중요한 부분은 Format입니다. 위치 벡터의 경우 DXGI_FORMAT_R32G32B32_FLOAT를 사용하고 색상의 경우 DXGI_FORMAT_R32G32B32A32_FLOAT를 사용합니다. 마지막으로 주의해야 할 것은 버퍼에서 데이터가 어떻게 배치되는지 나타내는 AlignedByteOffset입니다. 이 레이아웃의 경우 처음 12바이트는 위치이고 다음 16바이트는 색상이라고 말하고, AlignedByteOffset은 각 요소가 시작되는 위치를 보여줍니다. AlignedByteOffset에 값을 직접 입력하는 대신 D3D11_APPEND_ALIGNED_ELEMENT를 사용하면 간격을 알아낼 수 있습니다. 이 튜토리얼에서는 필요하지 않으므로 지금은 기본값으로 설정한 다른 설정을 사용합니다.

레이아웃 설명이 설정되면 크기를 구한 다음 D3D 장치를 사용하여 입력 레이아웃을 만들 수 있습니다. 또한 레이아웃이 생성되면 더 이상 필요하지 않으므로 정점 및 픽셀 셰이더 버퍼를 해제합니다.

셰이더를 활용하기 위해 설정해야 하는 마지막 것은 상수 버퍼입니다. 정점 셰이더에서 보았듯이 현재는 상수 버퍼가 하나뿐이므로 여기서 하나만 설정하여 셰이더와 인터페이스할 수 있습니다. 버퍼 사용은 각 프레임마다 업데이트되므로 동적으로 설정해야 합니다. 바인드 플래그는 이 버퍼가 상수 버퍼임을 나타냅니다. CPU 액세스 플래그는 사용과 일치해야 하므로 D3D11_CPU_ACCESS_WRITE로 설정해야 합니다. 설명을 작성하면 상수 버퍼 인터페이스를 만든 다음 SetShaderParameters 함수를 사용하여 셰이더의 내부 변수에 액세스할 수 있습니다.


ShutdownShader는 InitializeShader 함수에서 설정된 네 개의 인터페이스를 해제합니다.


OutputShaderErrorMessage는 정점 셰이더나 픽셀 셰이더를 컴파일할 때 발생하는 오류 메시지를 기록합니다.


SetShaderVariables 함수는 셰이더에서 전역 변수를 더 쉽게 설정하기 위해 존재합니다. 이 함수에서 사용되는 행렬은 ApplicationClass 내부에서 생성되고, 그 후 이 함수가 호출되어 Render 함수 호출 중에 정점 셰이더로 전송됩니다.

DirectX 11에서는 셰이더로 보내기 전에 행렬을 전치해야 합니다.

m_matrixBuffer를 잠그고, 그 안에 새로운 행렬을 설정한 후 잠금을 해제합니다.

이제 HLSL 버텍스 셰이더에 업데이트된 매트릭스 버퍼를 설정합니다.


RenderShader는 Render 함수에서 호출되는 두 번째 함수입니다. SetShaderParameters는 셰이더 매개변수가 올바르게 설정되었는지 확인하기 위해 이 함수보다 먼저 호출됩니다.

이 함수의 첫 번째 단계는 입력 어셈블러에서 입력 레이아웃을 활성화로 설정하는 것입니다. 이렇게 하면 GPU가 정점 버퍼의 데이터 형식을 알 수 있습니다. 두 번째 단계는 이 정점 버퍼를 렌더링하는 데 사용할 정점 셰이더와 픽셀 셰이더를 설정하는 것입니다. 셰이더가 설정되면 D3D 장치 컨텍스트를 사용하여 DrawIndexed DirectX 11 함수를 호출하여 삼각형을 렌더링합니다. 이 함수가 호출되면 녹색 삼각형이 렌더링됩니다.

```
////////////////////////////////////////////////////////////////////////////////
// Filename: colorshaderclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "colorshaderclass.h"

ColorShaderClass::ColorShaderClass()
{
	m_vertexShader = 0;
	m_pixelShader = 0;
	m_layout = 0;
	m_matrixBuffer = 0;
}

ColorShaderClass::ColorShaderClass(const ColorShaderClass& other)
{
}


ColorShaderClass::~ColorShaderClass()
{
}

bool ColorShaderClass::Initialize(ID3D11Device* device, HWND hwnd)
{
	bool result;
	wchar_t vsFilename[128];
	wchar_t psFilename[128];
	int error;


	// Set the filename of the vertex shader.
	error = wcscpy_s(vsFilename, 128, L"../Engine/color.vs");
	if (error != 0)
	{
		return false;
	}

	// Set the filename of the pixel shader.
	error = wcscpy_s(psFilename, 128, L"../Engine/color.ps");
	if (error != 0)
	{
		return false;
	}

	// Initialize the vertex and pixel shaders.
	result = InitializeShader(device, hwnd, vsFilename, psFilename);
	if (!result)
	{
		return false;
	}

	return true;
}

void ColorShaderClass::Shutdown()
{
	// Shutdown the vertex and pixel shaders as well as the related objects.
	ShutdownShader();

	return;
}

bool ColorShaderClass::Render(ID3D11DeviceContext* deviceContext, int indexCount, XMMATRIX worldMatrix, XMMATRIX viewMatrix,
	XMMATRIX projectionMatrix)
{
	bool result;


	// Set the shader parameters that it will use for rendering.
	result = SetShaderParameters(deviceContext, worldMatrix, viewMatrix, projectionMatrix);
	if (!result)
	{
		return false;
	}

	// Now render the prepared buffers with the shader.
	RenderShader(deviceContext, indexCount);

	return true;
}

bool ColorShaderClass::InitializeShader(ID3D11Device* device, HWND hwnd, WCHAR* vsFilename, WCHAR* psFilename)
{
	HRESULT result;
	ID3D10Blob* errorMessage;
	ID3D10Blob* vertexShaderBuffer;
	ID3D10Blob* pixelShaderBuffer;
	D3D11_INPUT_ELEMENT_DESC polygonLayout[2];
	unsigned int numElements;
	D3D11_BUFFER_DESC matrixBufferDesc;


	// Initialize the pointers this function will use to null.
	errorMessage = 0;
	vertexShaderBuffer = 0;
	pixelShaderBuffer = 0;

	// Compile the vertex shader code.
	result = D3DCompileFromFile(vsFilename, NULL, NULL, "ColorVertexShader", "vs_5_0", D3D10_SHADER_ENABLE_STRICTNESS, 0,
		&vertexShaderBuffer, &errorMessage);
	if (FAILED(result))
	{
		// If the shader failed to compile it should have writen something to the error message.
		if (errorMessage)
		{
			OutputShaderErrorMessage(errorMessage, hwnd, vsFilename);
		}
		// If there was  nothing in the error message then it simply could not find the shader file itself.
		else
		{
			MessageBox(hwnd, vsFilename, L"Missing Shader File", MB_OK);
		}

		return false;
	}

	// Compile the pixel shader code.
	result = D3DCompileFromFile(psFilename, NULL, NULL, "ColorPixelShader", "ps_5_0", D3D10_SHADER_ENABLE_STRICTNESS, 0,
		&pixelShaderBuffer, &errorMessage);
	if (FAILED(result))
	{
		// If the shader failed to compile it should have writen something to the error message.
		if (errorMessage)
		{
			OutputShaderErrorMessage(errorMessage, hwnd, psFilename);
		}
		// If there was nothing in the error message then it simply could not find the file itself.
		else
		{
			MessageBox(hwnd, psFilename, L"Missing Shader File", MB_OK);
		}

		return false;
	}

	// Create the vertex shader from the buffer.
	result = device->CreateVertexShader(vertexShaderBuffer->GetBufferPointer(), vertexShaderBuffer->GetBufferSize(), NULL, &m_vertexShader);
	if (FAILED(result))
	{
		return false;
	}

	// Create the pixel shader from the buffer.
	result = device->CreatePixelShader(pixelShaderBuffer->GetBufferPointer(), pixelShaderBuffer->GetBufferSize(), NULL, &m_pixelShader);
	if (FAILED(result))
	{
		return false;
	}

	// Create the vertex input layout description.
// This setup needs to match the VertexType stucture in the ModelClass and in the shader.
	polygonLayout[0].SemanticName = "POSITION";
	polygonLayout[0].SemanticIndex = 0;
	polygonLayout[0].Format = DXGI_FORMAT_R32G32B32_FLOAT;
	polygonLayout[0].InputSlot = 0;
	polygonLayout[0].AlignedByteOffset = 0;
	polygonLayout[0].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
	polygonLayout[0].InstanceDataStepRate = 0;

	polygonLayout[1].SemanticName = "COLOR";
	polygonLayout[1].SemanticIndex = 0;
	polygonLayout[1].Format = DXGI_FORMAT_R32G32B32A32_FLOAT;
	polygonLayout[1].InputSlot = 0;
	polygonLayout[1].AlignedByteOffset = D3D11_APPEND_ALIGNED_ELEMENT;
	polygonLayout[1].InputSlotClass = D3D11_INPUT_PER_VERTEX_DATA;
	polygonLayout[1].InstanceDataStepRate = 0;

	// Get a count of the elements in the layout.
	numElements = sizeof(polygonLayout) / sizeof(polygonLayout[0]);

	// Create the vertex input layout.
	result = device->CreateInputLayout(polygonLayout, numElements, vertexShaderBuffer->GetBufferPointer(),
		vertexShaderBuffer->GetBufferSize(), &m_layout);
	if (FAILED(result))
	{
		return false;
	}

	// Release the vertex shader buffer and pixel shader buffer since they are no longer needed.
	vertexShaderBuffer->Release();
	vertexShaderBuffer = 0;

	pixelShaderBuffer->Release();
	pixelShaderBuffer = 0;

	// Setup the description of the dynamic matrix constant buffer that is in the vertex shader.
	matrixBufferDesc.Usage = D3D11_USAGE_DYNAMIC;
	matrixBufferDesc.ByteWidth = sizeof(MatrixBufferType);
	matrixBufferDesc.BindFlags = D3D11_BIND_CONSTANT_BUFFER;
	matrixBufferDesc.CPUAccessFlags = D3D11_CPU_ACCESS_WRITE;
	matrixBufferDesc.MiscFlags = 0;
	matrixBufferDesc.StructureByteStride = 0;

	// Create the constant buffer pointer so we can access the vertex shader constant buffer from within this class.
	result = device->CreateBuffer(&matrixBufferDesc, NULL, &m_matrixBuffer);
	if (FAILED(result))
	{
		return false;
	}

	return true;
}

void ColorShaderClass::ShutdownShader()
{
	// Release the matrix constant buffer.
	if (m_matrixBuffer)
	{
		m_matrixBuffer->Release();
		m_matrixBuffer = 0;
	}

	// Release the layout.
	if (m_layout)
	{
		m_layout->Release();
		m_layout = 0;
	}

	// Release the pixel shader.
	if (m_pixelShader)
	{
		m_pixelShader->Release();
		m_pixelShader = 0;
	}

	// Release the vertex shader.
	if (m_vertexShader)
	{
		m_vertexShader->Release();
		m_vertexShader = 0;
	}

	return;
}

void ColorShaderClass::OutputShaderErrorMessage(ID3D10Blob* errorMessage, HWND hwnd, WCHAR* shaderFilename)
{
	char* compileErrors;
	unsigned long long bufferSize, i;
	ofstream fout;


	// Get a pointer to the error message text buffer.
	compileErrors = (char*)(errorMessage->GetBufferPointer());

	// Get the length of the message.
	bufferSize = errorMessage->GetBufferSize();

	// Open a file to write the error message to.
	fout.open("shader-error.txt");

	// Write out the error message.
	for (i = 0; i < bufferSize; i++)
	{
		fout << compileErrors[i];
	}

	// Close the file.
	fout.close();

	// Release the error message.
	errorMessage->Release();
	errorMessage = 0;

	// Pop a message up on the screen to notify the user to check the text file for compile errors.
	MessageBox(hwnd, L"Error compiling shader.  Check shader-error.txt for message.", shaderFilename, MB_OK);

	return;
}

bool ColorShaderClass::SetShaderParameters(ID3D11DeviceContext* deviceContext, XMMATRIX worldMatrix, XMMATRIX viewMatrix,
	XMMATRIX projectionMatrix)
{
	HRESULT result;
	D3D11_MAPPED_SUBRESOURCE mappedResource;
	MatrixBufferType* dataPtr;
	unsigned int bufferNumber;

	// Transpose the matrices to prepare them for the shader.
	worldMatrix = XMMatrixTranspose(worldMatrix);
	viewMatrix = XMMatrixTranspose(viewMatrix);
	projectionMatrix = XMMatrixTranspose(projectionMatrix);

	// Lock the constant buffer so it can be written to.
	result = deviceContext->Map(m_matrixBuffer, 0, D3D11_MAP_WRITE_DISCARD, 0, &mappedResource);
	if (FAILED(result))
	{
		return false;
	}

	// Get a pointer to the data in the constant buffer.
	dataPtr = (MatrixBufferType*)mappedResource.pData;

	// Copy the matrices into the constant buffer.
	dataPtr->world = worldMatrix;
	dataPtr->view = viewMatrix;
	dataPtr->projection = projectionMatrix;

	// Unlock the constant buffer.
	deviceContext->Unmap(m_matrixBuffer, 0);

	// Set the position of the constant buffer in the vertex shader.
	bufferNumber = 0;

	// Finanly set the constant buffer in the vertex shader with the updated values.
	deviceContext->VSSetConstantBuffers(bufferNumber, 1, &m_matrixBuffer);

	return true;
}

void ColorShaderClass::RenderShader(ID3D11DeviceContext* deviceContext, int indexCount)
{
	// Set the vertex input layout.
	deviceContext->IASetInputLayout(m_layout);

	// Set the vertex and pixel shaders that will be used to render this triangle.
	deviceContext->VSSetShader(m_vertexShader, NULL, 0);
	deviceContext->PSSetShader(m_pixelShader, NULL, 0);

	// Render the triangle.
	deviceContext->DrawIndexed(indexCount, 0, 0);

	return;
}
```


### Modelclass.h
이전에 언급했듯이 ModelClass는 3D 모델의 지오메트리를 캡슐화하는 역할을 합니다. 이 튜토리얼에서는 단일 녹색 삼각형에 대한 데이터를 수동으로 설정합니다. 또한 삼각형을 렌더링할 수 있도록 정점 및 인덱스 버퍼를 만듭니다.

다음은 이 ModelClass에서 정점 버퍼와 함께 사용될 정점 유형의 정의입니다. 또한 이 typedef는 튜토리얼에서 나중에 살펴볼 ColorShaderClass의 레이아웃과 일치해야 한다는 점에 유의하세요.

여기의 함수는 모델의 정점 및 인덱스 버퍼의 초기화 및 종료를 처리합니다. Render 함수는 모델 지오메트리를 비디오 카드에 넣어 컬러 셰이더로 그릴 준비를 합니다.

ModelClass의 프라이빗 변수는 정점 및 인덱스 버퍼와 각 버퍼의 크기를 추적하기 위한 두 개의 정수입니다. 모든 DirectX 11 버퍼는 일반적으로 제네릭 ID3D11Buffer 유형을 사용하고 처음 생성될 때 버퍼 설명으로 더 명확하게 식별됩니다.

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: modelclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _MODELCLASS_H_
#define _MODELCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <d3d11.h>
#include <directxmath.h>
using namespace DirectX;


////////////////////////////////////////////////////////////////////////////////
// Class name: ModelClass
////////////////////////////////////////////////////////////////////////////////
class ModelClass
{
private:
	struct VertexType
	{
		XMFLOAT3 position;
		XMFLOAT4 color;
	};
public:
	ModelClass();
	ModelClass(const ModelClass&);
	~ModelClass();

	bool Initialize(ID3D11Device*);
	void Shutdown();
	void Render(ID3D11DeviceContext*);

	int GetIndexCount();

private:
	bool InitializeBuffers(ID3D11Device*);
	void ShutdownBuffers();
	void RenderBuffers(ID3D11DeviceContext*);

private:
	ID3D11Buffer* m_vertexBuffer, * m_indexBuffer;
	int m_vertexCount, m_indexCount;
};

#endif
```


### Modelclass.cpp
클래스 생성자는 정점 및 인덱스 버퍼 포인터를 null로 초기화합니다.


Initialize 함수는 정점 및 인덱스 버퍼에 대한 초기화 함수를 호출합니다.


Shutdown 함수는 정점 및 인덱스 버퍼에 대한 종료 함수를 호출합니다.


Render는 ApplicationClass::Render 함수에서 호출됩니다. 이 함수는 RenderBuffers를 호출하여 정점 및 인덱스 버퍼를 그래픽 파이프라인에 배치하여 컬러 셰이더가 렌더링할 수 있도록 합니다.


GetIndexCount는 모델의 인덱스 수를 반환합니다. 컬러 셰이더는 이 모델을 그리기 위해 이 정보가 필요합니다.


InitializeBuffers 함수는 정점 및 인덱스 버퍼를 만드는 것을 처리하는 곳입니다. 일반적으로 모델을 읽어서 해당 데이터 파일에서 버퍼를 만듭니다. 이 튜토리얼에서는 삼각형이 하나뿐이므로 정점 및 인덱스 버퍼에 수동으로 점을 설정합니다.

먼저, 나중에 최종 버퍼를 채우는 데 사용할 정점 및 인덱스 데이터를 보관할 두 개의 임시 배열을 만듭니다.

이제 정점과 인덱스 배열을 삼각형의 세 점과 각 점의 인덱스로 채웁니다.
"점을 그리는 순서는 시계 방향이라는 점에 유의하세요."
반시계 방향으로 하면 삼각형이 반대 방향을 향하고 있다고 생각하고 뒷면 컬링으로 인해 그리지 않습니다. 정점을 GPU에 보내는 순서는 매우 중요하다는 점을 항상 기억하세요. 정점 설명의 일부이므로 색상도 여기에 설정됩니다. 색상을 녹색으로 설정했습니다.

정점 배열과 인덱스 배열이 채워졌으므로 이제 정점 버퍼와 인덱스 버퍼를 만드는 데 사용할 수 있습니다. 두 버퍼를 만드는 방법은 동일합니다. 먼저 버퍼에 대한 설명을 작성합니다. 설명에서 ByteWidth(버퍼 크기)와 BindFlags(버퍼 유형)는 올바르게 채워졌는지 확인해야 하는 내용입니다. 설명을 작성한 후에는 이전에 만든 정점 또는 인덱스 배열을 가리키는 하위 리소스 포인터도 작성해야 합니다. 설명과 하위 리소스 포인터를 사용하면 D3D 장치를 사용하여 CreateBuffer를 호출할 수 있으며 새 버퍼에 대한 포인터가 반환됩니다.

정점 버퍼와 인덱스 버퍼가 생성된 후에는 데이터가 버퍼에 복사되었기 때문에 더 이상 필요하지 않은 정점 및 인덱스 배열을 삭제할 수 있습니다.


ShutdownBuffers 함수는 InitializeBuffers 함수에서 생성된 정점 버퍼와 인덱스 버퍼를 해제합니다.


RenderBuffers는 Render 함수에서 호출됩니다. 이 함수의 목적은 GPU의 입력 어셈블러에서 정점 버퍼와 인덱스 버퍼를 활성 상태로 설정하는 것입니다. GPU에 활성 정점 버퍼가 있으면 셰이더를 사용하여 해당 버퍼를 렌더링할 수 있습니다. 이 함수는 또한 삼각형, 선, 팬 등과 같이 해당 버퍼를 그리는 방법을 정의합니다. 이 튜토리얼에서는 입력 어셈블러에서 정점 버퍼와 인덱스 버퍼를 활성 상태로 설정하고 IASetPrimitiveTopology DirectX 함수를 사용하여 버퍼를 삼각형으로 그려야 한다고 GPU에 알립니다.

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: modelclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "modelclass.h"

ModelClass::ModelClass()
{
	m_vertexBuffer = 0;
	m_indexBuffer = 0;
}

ModelClass::ModelClass(const ModelClass& other)
{
}

ModelClass::~ModelClass()
{
}

bool ModelClass::Initialize(ID3D11Device* device)
{
	bool result;


	// Initialize the vertex and index buffers.
	result = InitializeBuffers(device);
	if (!result)
	{
		return false;
	}

	return true;
}

void ModelClass::Shutdown()
{
	// Shutdown the vertex and index buffers.
	ShutdownBuffers();

	return;
}

void ModelClass::Render(ID3D11DeviceContext* deviceContext)
{
	// Put the vertex and index buffers on the graphics pipeline to prepare them for drawing.
	RenderBuffers(deviceContext);

	return;
}

int ModelClass::GetIndexCount()
{
	return m_indexCount;
}

bool ModelClass::InitializeBuffers(ID3D11Device* device)
{
	VertexType* vertices;
	unsigned long* indices;
	D3D11_BUFFER_DESC vertexBufferDesc, indexBufferDesc;
	D3D11_SUBRESOURCE_DATA vertexData, indexData;
	HRESULT result;

	// Set the number of vertices in the vertex array.
	m_vertexCount = 3;

	// Set the number of indices in the index array.
	m_indexCount = 3;

	// Create the vertex array.
	vertices = new VertexType[m_vertexCount];
	if (!vertices)
	{
		return false;
	}

	// Create the index array.
	indices = new unsigned long[m_indexCount];
	if (!indices)
	{
		return false;
	}

	// Load the vertex array with data.
	vertices[0].position = XMFLOAT3(-1.0f, -1.0f, 0.0f);  // Bottom left.
	vertices[0].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	vertices[1].position = XMFLOAT3(0.0f, 1.0f, 0.0f);  // Top middle.
	vertices[1].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	vertices[2].position = XMFLOAT3(1.0f, -1.0f, 0.0f);  // Bottom right.
	vertices[2].color = XMFLOAT4(0.0f, 1.0f, 0.0f, 1.0f);

	// Load the index array with data.
	indices[0] = 0;  // Bottom left.
	indices[1] = 1;  // Top middle.
	indices[2] = 2;  // Bottom right.

	// Set up the description of the static vertex buffer.
	vertexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	vertexBufferDesc.ByteWidth = sizeof(VertexType) * m_vertexCount;
	vertexBufferDesc.BindFlags = D3D11_BIND_VERTEX_BUFFER;
	vertexBufferDesc.CPUAccessFlags = 0;
	vertexBufferDesc.MiscFlags = 0;
	vertexBufferDesc.StructureByteStride = 0;

	// Give the subresource structure a pointer to the vertex data.
	vertexData.pSysMem = vertices;
	vertexData.SysMemPitch = 0;
	vertexData.SysMemSlicePitch = 0;

	// Now create the vertex buffer.
	result = device->CreateBuffer(&vertexBufferDesc, &vertexData, &m_vertexBuffer);
	if (FAILED(result))
	{
		return false;
	}

	// Set up the description of the static index buffer.
	indexBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	indexBufferDesc.ByteWidth = sizeof(unsigned long) * m_indexCount;
	indexBufferDesc.BindFlags = D3D11_BIND_INDEX_BUFFER;
	indexBufferDesc.CPUAccessFlags = 0;
	indexBufferDesc.MiscFlags = 0;
	indexBufferDesc.StructureByteStride = 0;

	// Give the subresource structure a pointer to the index data.
	indexData.pSysMem = indices;
	indexData.SysMemPitch = 0;
	indexData.SysMemSlicePitch = 0;

	// Create the index buffer.
	result = device->CreateBuffer(&indexBufferDesc, &indexData, &m_indexBuffer);
	if (FAILED(result))
	{
		return false;
	}

	// Release the arrays now that the vertex and index buffers have been created and loaded.
	delete[] vertices;
	vertices = 0;

	delete[] indices;
	indices = 0;

	return true;
}

void ModelClass::ShutdownBuffers()
{
	// Release the index buffer.
	if (m_indexBuffer)
	{
		m_indexBuffer->Release();
		m_indexBuffer = 0;
	}

	// Release the vertex buffer.
	if (m_vertexBuffer)
	{
		m_vertexBuffer->Release();
		m_vertexBuffer = 0;
	}

	return;
}

void ModelClass::RenderBuffers(ID3D11DeviceContext* deviceContext)
{
	unsigned int stride;
	unsigned int offset;


	// Set vertex buffer stride and offset.
	stride = sizeof(VertexType);
	offset = 0;

	// Set the vertex buffer to active in the input assembler so it can be rendered.
	deviceContext->IASetVertexBuffers(0, 1, &m_vertexBuffer, &stride, &offset);

	// Set the index buffer to active in the input assembler so it can be rendered.
	deviceContext->IASetIndexBuffer(m_indexBuffer, DXGI_FORMAT_R32_UINT, 0);

	// Set the type of primitive that should be rendered from this vertex buffer, in this case triangles.
	deviceContext->IASetPrimitiveTopology(D3D11_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

	return;
}
```
### Cameraclass.h
우리는 HLSL 셰이더를 코딩하는 방법, 정점 및 인덱스 버퍼를 설정하는 방법, ColorShaderClass를 사용하여 HLSL 셰이더를 호출하여 해당 버퍼를 그리는 방법을 살펴보았습니다. 그러나 우리가 놓친 한 가지는 이를 그릴 시점입니다. 이를 위해 DirectX 11에 장면을 어디에서 어떻게 보고 있는지 알려주는 카메라 클래스가 필요합니다. 카메라 클래스는 카메라의 위치와 현재 회전을 추적합니다. 위치 및 회전 정보를 사용하여 렌더링을 위해 HLSL 셰이더로 전달되는 뷰 매트릭스를 생성합니다.

CameraClass 헤더는 네 가지 함수만 사용하므로 매우 간단합니다. SetPosition 및 SetRotation 함수는 카메라 객체의 위치와 회전을 설정하는 데 사용됩니다. Render는 카메라의 위치와 회전을 기반으로 뷰 매트릭스를 만드는 데 사용됩니다. 마지막으로 GetViewMatrix는 셰이더가 렌더링에 사용할 수 있도록 카메라 객체에서 뷰 매트릭스를 검색하는 데 사용됩니다.

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: cameraclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _CAMERACLASS_H_
#define _CAMERACLASS_H_


//////////////
// INCLUDES //
//////////////
#include <directxmath.h>
using namespace DirectX;


////////////////////////////////////////////////////////////////////////////////
// Class name: CameraClass
////////////////////////////////////////////////////////////////////////////////
class CameraClass
{
public:
	CameraClass();
	CameraClass(const CameraClass&);
	~CameraClass();

	void SetPosition(float, float, float);
	void SetRotation(float, float, float);

	XMFLOAT3 GetPosition();
	XMFLOAT3 GetRotation();

	void Render();
	void GetViewMatrix(XMMATRIX&);

private:
	float m_positionX, m_positionY, m_positionZ;
	float m_rotationX, m_rotationY, m_rotationZ;
	XMMATRIX m_viewMatrix;
};

#endif
```
### Cameraclass.cpp
클래스 생성자는 카메라의 위치와 회전을 장면의 원점으로 초기화합니다.

SetPosition과 SetRotation 함수는 카메라의 위치와 회전을 설정하는 데 사용됩니다.

GetPosition 및 GetRotation 함수는 호출하는 함수에 카메라의 위치와 회전을 반환합니다.

Render 함수는 카메라의 위치와 회전을 사용하여 뷰 매트릭스를 빌드하고 업데이트합니다. 먼저 up, position, rotation 등에 대한 변수를 설정합니다. 그런 다음 장면의 원점에서 먼저 카메라의 x, y, z 회전을 기준으로 카메라를 회전합니다. 제대로 회전되면 카메라를 3D 공간의 위치로 변환합니다. position, lookAt, up에 올바른 값이 있으면 XMMatrixLookAtLH 함수를 사용하여 현재 카메라 회전 및 변환을 나타내는 뷰 매트릭스를 만들 수 있습니다.

Render 함수가 뷰 매트릭스를 생성하기 위해 호출된 후, 이 GetViewMatrix 함수를 사용하여 호출하는 함수에 업데이트된 뷰 매트릭스를 제공할 수 있습니다. 뷰 매트릭스는 HLSL 버텍스 셰이더에서 사용되는 세 가지 주요 매트릭스 중 하나가 될 것입니다.

```c++
////////////////////////////////////////////////////////////////////////////////
// Filename: cameraclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "cameraclass.h"

CameraClass::CameraClass()
{
	m_positionX = 0.0f;
	m_positionY = 0.0f;
	m_positionZ = 0.0f;

	m_rotationX = 0.0f;
	m_rotationY = 0.0f;
	m_rotationZ = 0.0f;
}


CameraClass::CameraClass(const CameraClass& other)
{
}


CameraClass::~CameraClass()
{
}

void CameraClass::SetPosition(float x, float y, float z)
{
	m_positionX = x;
	m_positionY = y;
	m_positionZ = z;
	return;
}


void CameraClass::SetRotation(float x, float y, float z)
{
	m_rotationX = x;
	m_rotationY = y;
	m_rotationZ = z;
	return;
}

XMFLOAT3 CameraClass::GetPosition()
{
	return XMFLOAT3(m_positionX, m_positionY, m_positionZ);
}


XMFLOAT3 CameraClass::GetRotation()
{
	return XMFLOAT3(m_rotationX, m_rotationY, m_rotationZ);
}

void CameraClass::Render()
{
	XMFLOAT3 up, position, lookAt;
	XMVECTOR upVector, positionVector, lookAtVector;
	float yaw, pitch, roll;
	XMMATRIX rotationMatrix;


	// Setup the vector that points upwards.
	up.x = 0.0f;
	up.y = 1.0f;
	up.z = 0.0f;

	// Load it into a XMVECTOR structure.
	upVector = XMLoadFloat3(&up);

	// Setup the position of the camera in the world.
	position.x = m_positionX;
	position.y = m_positionY;
	position.z = m_positionZ;

	// Load it into a XMVECTOR structure.
	positionVector = XMLoadFloat3(&position);

	// Setup where the camera is looking by default.
	lookAt.x = 0.0f;
	lookAt.y = 0.0f;
	lookAt.z = 1.0f;

	// Load it into a XMVECTOR structure.
	lookAtVector = XMLoadFloat3(&lookAt);

	// Set the yaw (Y axis), pitch (X axis), and roll (Z axis) rotations in radians.
	pitch = m_rotationX * 0.0174532925f;
	yaw = m_rotationY * 0.0174532925f;
	roll = m_rotationZ * 0.0174532925f;

	// Create the rotation matrix from the yaw, pitch, and roll values.
	rotationMatrix = XMMatrixRotationRollPitchYaw(pitch, yaw, roll);

	// Transform the lookAt and up vector by the rotation matrix so the view is correctly rotated at the origin.
	lookAtVector = XMVector3TransformCoord(lookAtVector, rotationMatrix);
	upVector = XMVector3TransformCoord(upVector, rotationMatrix);

	// Translate the rotated camera position to the location of the viewer.
	lookAtVector = XMVectorAdd(positionVector, lookAtVector);

	// Finally create the view matrix from the three updated vectors.
	m_viewMatrix = XMMatrixLookAtLH(positionVector, lookAtVector, upVector);

	return;
}

void CameraClass::GetViewMatrix(XMMATRIX& viewMatrix)
{
	viewMatrix = m_viewMatrix;
	return;
}

```

### Applicationclass.h
ApplicationClass에 이제 세 개의 새로운 클래스가 추가되었습니다. CameraClass, ModelClass, ColorShaderClass에는 헤더가 추가되었고 개인 멤버 변수도 추가되었습니다. ApplicationClass는 프로젝트에 필요한 모든 클래스 객체를 호출하여 장면을 렌더링하는 데 사용되는 주요 클래스라는 점을 기억하세요.
```
////////////////////////////////////////////////////////////////////////////////
// Filename: applicationclass.h
////////////////////////////////////////////////////////////////////////////////
#ifndef _APPLICATIONCLASS_H_
#define _APPLICATIONCLASS_H_


//////////////
// INCLUDES //
//////////////
#include <windows.h>

///////////////////////
// MY CLASS INCLUDES //
///////////////////////
#include "d3dclass.h"
#include "cameraclass.h"
#include "modelclass.h"
#include "colorshaderclass.h"

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
	CameraClass* m_Camera;
	ModelClass* m_Model;
	ColorShaderClass* m_ColorShader;
};

#endif

```
### Applicationclass.cpp
ApplicationClass의 첫 번째 변경 사항은 클래스 생성자에서 카메라, 모델, 색상 셰이더 객체를 null로 초기화하는 것입니다.

또한, 세 개의 새 객체를 생성하고 초기화하기 위해 Initialize 함수도 업데이트되었습니다.


Shutdown도 업데이트되어 세 개의 새로운 객체를 종료하고 해제합니다.


프레임 기능은 이전 튜토리얼과 동일하게 유지되었습니다.


예상했듯이 Render 함수에 가장 많은 변경 사항이 있습니다. 여전히 장면을 지우는 것으로 시작하지만 검은색으로 지워집니다. 그런 다음 카메라 객체에 대한 Render 함수를 호출하여 Initialize 함수에서 설정된 카메라 위치를 기반으로 뷰 행렬을 만듭니다. 뷰 행렬이 생성되면 카메라 클래스에서 복사본을 가져옵니다. 또한 D3DClass 객체에서 월드 및 투영 행렬의 복사본을 가져옵니다. 그런 다음 ModelClass::Render 함수를 호출하여 녹색 삼각형 모델 지오메트리를 그래픽 파이프라인에 넣습니다. 이제 정점이 준비되었으므로 모델 정보와 각 정점을 배치하기 위한 세 개의 행렬을 사용하여 색상 셰이더를 호출하여 정점을 그립니다. 이제 녹색 삼각형이 백 버퍼에 그려집니다. 그러면 장면이 완료되고 EndScene을 호출하여 화면에 표시합니다.

```
////////////////////////////////////////////////////////////////////////////////
// Filename: applicationclass.cpp
////////////////////////////////////////////////////////////////////////////////
#include "applicationclass.h"


ApplicationClass::ApplicationClass()
{
	m_Direct3D = 0;
	m_Camera = 0;
	m_Model = 0;
	m_ColorShader = 0;
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
	if (!result)
	{
		MessageBox(hwnd, L"Could not initialize Direct3D", L"Error", MB_OK);
		return false;
	}

	// Create the camera object.
	m_Camera = new CameraClass;

	// Set the initial position of the camera.
	m_Camera->SetPosition(0.0f, 0.0f, -5.0f);

	// Create and initialize the model object.
	m_Model = new ModelClass;

	result = m_Model->Initialize(m_Direct3D->GetDevice());
	if (!result)
	{
		MessageBox(hwnd, L"Could not initialize the model object.", L"Error", MB_OK);
		return false;
	}

	// Create and initialize the color shader object.
	m_ColorShader = new ColorShaderClass;

	result = m_ColorShader->Initialize(m_Direct3D->GetDevice(), hwnd);
	if (!result)
	{
		MessageBox(hwnd, L"Could not initialize the color shader object.", L"Error", MB_OK);
		return false;
	}

	return true;
}


void ApplicationClass::Shutdown()
{
	// Release the color shader object.
	if (m_ColorShader)
	{
		m_ColorShader->Shutdown();
		delete m_ColorShader;
		m_ColorShader = 0;
	}

	// Release the model object.
	if (m_Model)
	{
		m_Model->Shutdown();
		delete m_Model;
		m_Model = 0;
	}

	// Release the camera object.
	if (m_Camera)
	{
		delete m_Camera;
		m_Camera = 0;
	}

	// Release the Direct3D object.
	if (m_Direct3D)
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
	if (!result)
	{
		return false;
	}

	return true;
}


bool ApplicationClass::Render()
{
	XMMATRIX worldMatrix, viewMatrix, projectionMatrix;
	bool result;


	// Clear the buffers to begin the scene.
	m_Direct3D->BeginScene(0.0f, 0.0f, 0.0f, 1.0f);

	// Generate the view matrix based on the camera's position.
	m_Camera->Render();

	// Get the world, view, and projection matrices from the camera and d3d objects.
	m_Direct3D->GetWorldMatrix(worldMatrix);
	m_Camera->GetViewMatrix(viewMatrix);
	m_Direct3D->GetProjectionMatrix(projectionMatrix);

	// Put the model vertex and index buffers on the graphics pipeline to prepare them for drawing.
	m_Model->Render(m_Direct3D->GetDeviceContext());

	// Render the model using the color shader.
	result = m_ColorShader->Render(m_Direct3D->GetDeviceContext(), m_Model->GetIndexCount(), worldMatrix, viewMatrix, projectionMatrix);
	if (!result)
	{
		return false;
	}

	// Clear the buffers to begin the scene.
	m_Direct3D->BeginScene(0.5f, 0.5f, 0.5f, 1.0f);


	// Present the rendered scene to the screen.
	m_Direct3D->EndScene();
	
	return true;
}
```


이 튜토리얼을 마치기 전에 이 튜토리얼의 렌더링 행렬이 어떻게 작동하는지 이해했는지 확인하고 싶습니다.

color.vs vertex 셰이더를 다시 잠깐 살펴보겠습니다. 해당 함수에 들어오는 각 정점을 월드 행렬, 뷰 행렬, 그리고 투영 행렬로 곱하는 것을 보실 수 있습니다.

정점을 월드 행렬로 곱하면 3D 월드에서 해당 정점을 배치합니다. 월드 행렬에는 이동, 회전, 크기 조정 등과 같은 것을 적용할 수 있습니다. 따라서 이를 통해 3D 공간에서 해당 정점을 올바르게 배치할 수 있습니다.

다음으로 정점을 뷰 행렬로 곱하면 정점의 위치를 ​​다시 현재 보고 있는 위치, 즉 카메라 위치로 업데이트합니다.

마지막으로 정점을 투영 행렬로 곱하면 정점의 위치를 ​​다시 업데이트하여 2D 화면으로 다시 렌더링합니다.