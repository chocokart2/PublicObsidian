


Vertex Shaders

Vertex shaders are small programs that are written mainly for transforming the vertices from the vertex buffer into 3D space. There are other calculations that can be done such as calculating normals for each vertex. The vertex shader program will be called by the GPU for each vertex it needs to process. For example, a 5,000 polygon model will run your vertex shader program 15,000 times each frame just to draw that single model. So, if you lock your graphics program to 60 fps it will call your vertex shader 900,000 times a second to draw just 5,000 triangles. As you can tell writing efficient vertex shaders is important.

버텍스 셰이더

버텍스 셰이더는 주로 버텍스 버퍼의 버텍스를 3D 공간으로 변환하기 위해 작성된 작은 프로그램입니다. 각 버텍스에 대한 법선 계산과 같은 다른 계산도 수행할 수 있습니다. 버텍스 셰이더 프로그램은 처리해야 하는 각 버텍스에 대해 GPU에서 호출됩니다. 예를 들어 5,000개의 폴리곤 모델을 그리려면 버텍스 셰이더 프로그램이 매 프레임마다 15,000번 실행됩니다. 따라서 그래픽 프로그램을 60fps로 고정하면 버텍스 셰이더를 초당 900,000번 호출하여 5,000개의 삼각형만 그리게 됩니다. 효율적인 버텍스 셰이더를 작성하는 것이 중요하다는 것을 알 수 있습니다.