
Index Buffers

Index buffers are related to vertex buffers. Their purpose is to record the location of each vertex that is in the vertex buffer. The GPU then uses the index buffer to quickly find specific vertices in the vertex buffer. The concept of an index buffer is similar to the concept using an index in a book, it helps find the topic you are looking for at a much higher speed. The DirectX documentation says that using index buffers can also increase the possibility of caching the vertex data in faster locations in video memory. So, it is highly advised to use these for performance reasons as well.

인덱스 버퍼

인덱스 버퍼는 버텍스 버퍼와 관련이 있습니다.
버텍스 버퍼의 목적은 버텍스 버퍼에 있는 각 버텍스의 위치를 기록하는 것입니다.
그런 다음 GPU는 인덱스 버퍼를 사용하여 버텍스 버퍼에서 특정 버텍스를 빠르게 찾습니다.
인덱스 버퍼의 개념은 책에서 색인을 사용하는 개념과 유사하며, 원하는 주제를 훨씬 빠른 속도로 찾는 데 도움이 됩니다.
DirectX 문서에 따르면 인덱스 버퍼를 사용하면 비디오 메모리의 더 빠른 위치에 버텍스 데이터를 캐싱할 가능성이 높아질 수 있다고 합니다.
따라서 성능상의 이유로도 이를 사용하는 것이 좋습니다.