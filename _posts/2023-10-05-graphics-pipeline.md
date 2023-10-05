---
title:  "렌더링 파이프라인 정리"
excerpt: "3D 오브젝트가 화면에 나타나는 과정을 정리한 글"

categories:
  - OpenGL
tags:
  - [OpenGL, Graphics]

toc: true
toc_sticky: true
 
date: 2023-10-05
last_modified_at: 2023-10-05
---
# 렌더링 파이프라인
 
## 서론
유니티, 언리얼과 같은 게임 엔진을 사용하다보면 렌더링 파이프라인이란 것을 한번쯤은 보게 된다. 렌더링 파이프라인이란 그래픽스 파이프라인의 하위 개념으로써, 3차원 물체를 2차원에 투영하는 렌더링 프로세스를 의미한다. 간단히 3D 물체를 모니터에 표현하는 일련의 과정을 의미한다.

## 본론
렌더링 파이프라인은 일반적으로 Input Assembler, Vertex Shader, Tessellation, Geometry Shader, Rasterization, Pixel(Fragment) Shader 으로 이루어진6개의 과정을 거친다. 이 중 Vertex Shader, Tessellation, Geometry Shader, Pixel(Fragment) Shader가 조작가능한 단계이다. 이 외에는 조작이 불가능하다.

### 1. Input Assembly 단계
이 단계에선 CPU가 정점 데이터를 담는 정점 버퍼를 GPU에게 전달해준다. 이 정점 버퍼에는 위치, 노말, 색상, UV를 담고 있다. GPU는 정점 버퍼를 받아 삼각형과 같은 기본 도형(Primitive)로 조립해주기에 Input Assembler 단계라고 한다. 이러한 과정을 거치는 이유는 GPU가 CPU의 자원에 직접 접근하지 못하기 때문이다.
### 2. Vertex Shader 단계
첫번째 과정을 겪으면 조립된 각 도형들은 자신만의 좌표계를 가지고 있다. 그렇기에 하나의 좌표계(View Space)로 모으기 위한 작업이 필요하다. 그러한 과정을 수행하는 곳이 바로 이곳으로, MVP(Model, View, Projection) 변환을 겪게 된다.
### 3. Tesselation 단계
하나의 도형을 여러 개의 도형으로 쪼개는 것을 테셀레이션이라고 한다. 저해상도의 메시가 부드러운 고해상도의 메시로 변하게 되는데, 이것을 이용하면 LOD(Level of detail)를 사용함으로써 성능을 향상시킬 수 있다.
### 4. Geometry Shader 단계
기본 도형에서 점, 선 등을 수정할 수 있게 해주는 쉐이더이다. 필요할 때마다 도형을 수정해 사용할 수 있어서 성능 향상에 도움을 줄 수 있다. 
### 5. Rasterization 단계
3D 도형이 픽셀로 변경되는 과정이다. 정점 사이의 공간은 색 보간을 통해 픽셀에 적절히 반영된다. 이 과정에서는 아직 색깔이 존재하지 않는다.
### 6. Fragment Shader 단계
래스트화된 픽셀에 색을 입혀주는 과정이다. 라이팅과 그림자, 투명도, 텍스쳐 색상은 모두 이 과정에서 입혀지게 된다.

## 결론
이러한 과정을 통틀어 렌더링 파이프라인이라고 한다. OpenGL, DirectX11, Vulkan 등등 다양한 그래픽스 API에서 사용되며, 각자의 구현방식은 다를 수 있다.

---
# 참고
[[Computer Graphics] #18. GPU 테셀레이션 | Dandi (choi-dan-di.github.io)](https://choi-dan-di.github.io/computer-graphics/surface-tessellation/)
<br><br>
[[Computer Graphics] 렌더링 파이프라인 요약 (velog.io)](https://velog.io/@cedongne/Graphics-%EB%A0%8C%EB%8D%94%EB%A7%81-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EC%9A%94%EC%95%BD)
<br><br>
[렌더링 파이프라인 간단 정리 | Rito15](https://rito15.github.io/posts/rendering-pipeline/)
