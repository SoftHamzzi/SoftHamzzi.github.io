---
title:  "그래픽스 API 비교 보고서"
excerpt: "그래픽스 API들을 비교하고 정리한 글"

categories:
  - GraphicsDoc
tags:
  - [Documents, Graphics]

toc: true
toc_sticky: true
 
date: 2023-10-06
last_modified_at: 2023-10-06
---
매우 급하게 쓰여진 대학교 과제 글이므로 잘못된 부분이 매우 많을 수 있습니다!  
ChatGPT또한 사용한 점 참고 부탁드립니다.
{: .notice--warning}
# 그래픽스 API 비교 보고서
---
# 1. 서론
그래픽 API는 그래픽 렌더링과 GPU(그래픽 처리 장치)를 활용하여 컴퓨터 화면에 이미지, 애니메이션 및 3D 그래픽을 렌더링하는 데 사용되는 기술이다. 그래픽 API를 큰 틀로보면 DirectX, OpenGL, Vulkan, Metal 이 있다. 그러나 이 보고서에서는 버전, 파생 API에 따라 다시 한번 DirectX11, DirectX12, OpenGL, WebGL, Vulkan, 그리고 Metal 총 6가지로 나눠 비교하고 분석한다. 각 API의 특징, 용도, 성능, 개발 환경 및 장단점을 다루고 비교할 것이다.

# 2. 본론
## 1. DirectX
마이크로소프트에서 개발하였으며, 컴퓨팅 멀티미디어와 관련된 작업을 마이크로소프트 플랫폼에서 처리하기 위한 API이다. DirectX에서 주로 사용하는 버전이 9, 11, 12버전이므로, 11버전과 12버전을 분석해보려 한다.
>### 1) DirectX11
>DirectX의 11버전이다. 주로 게임 개발에 사용되며, 높은 수준의 그래픽 렌더링을 지원한다. DirectX11의 주요 특징으로는 다음과 같다.
첫째, 하이레벨 API. 개발자들이 비교적 쉽게 학습하고 사용할 수 있도록 하이레벨 API로 설계되었다. 이로 인해 상대적으로 빠른 개발이 가능하며, 렌더링 파이프라인과 자원 관리에 대한 많은 부분을 DirectX가 처리해준다.
둘째, 유연한 렌더링 옵션. 다양한 렌더링 기술을 지원한다. 셰이더 프로그래밍을 통해 다양한 시각 효과와 그래픽 품질을 달성할 수 있으며, 게임 환경을 다양하게 표현할 수 있다.
셋째, Windows 전용. 주로 Windows 플랫폼에서 사용되며, 다른 플랫폼에서의 지원은 제한적이다. 따라서 Windows 기반 개발에 적합하다.
>### 2) DirectX12
>DirectX 12는 로우레벨 API로 개발자가 메모리 관리, 스레딩 및 최적화를 직접 다룰 수 있어 하드웨어의 모든 기능을 최대한 활용할 수 있다. 특징은 다음과 같다.
첫째. 로우레벨 API. DirectX12는 로우레벨 API로, 개발자가 메모리 및 스레딩을 직접 관리할 수 있어 최적화에 유리하다.
둘째, 높은 성능. 다중 스레딩과 다중 코어 CPU를 효과적으로 활용하여 높은 성능을 제공한다.
셋째, Windows 전용. 11버전과 마찬가지로 주로 Windows 플랫폼에서 사용되며, 다른 플랫폼에서는 사용이 제한적이다. 그러나 DirectX11과는 다르게 Windows 10 이상부터만 사용하다는 특징이 있다.
>### 3) DirectX11과 DirectX12 비교
>DirectX12가 DirectX11에 비해 이론적으로는 모든 면에서 기술적으로 우월하다. 하지만 DirectX11로 돌리는 것이 프레임이 더 잘나오고 안정적인 게임이 있으며, 오히려 DirectX12의 성능이 떨어지는 경우가 많다.
이러한 현상이 벌어지는 이유는 다음과 같다.
DirectX12는  AMD에서 개발한 Mantle에서 선보인 아이디어를 기반으로 한 로우레벨 API이다. 타사의 기술을 기반으로 한 로우 레벨 API에 호환성이나 안정성이 떨어질 수 밖에 없었다. 그러므로 개발을 할 때 어려운 프로세스 과정을 사용하고, 다른 하드웨어와의 안정성이 떨어지는 이유로 악효과가 생기게 되었다. 또한, 다이렉트X11이 이전 버전인 만큼 안정성이나 개발 환경이 더 간편하다는 이유도 있다.
>|비교 항목|DirectX11|DirectX12|
>|---|---|---|
>|지원 플랫폼|Windows|Windows 10 이상|
>|API 유형|하이레벨 API|로우레벨 API|
>|성능|높은 성능, 오버헤드 존재|낮은 CPU 오버헤드, 뛰어난 최적화|
>|멀티스레딩|제한적|멀티스레딩 지원|
>|개발 언어|C++, C# 등|C++, C#, DirectX12 API|

## 2. OpenGL
OpenGL은 크로스 플랫폼 지원을 강조하는 그래픽 API로, Windows, macOS, Linux 등 다양한 운영 체제에서 사용할 수 있다. 다음은 OpenGL의 주요 특징이다.
첫째. 크로스 플랫폼 지원. OpenGL은 여러 운영 체제에서 사용 가능하므로, 플랫폼에 독립적인 애플리케이션을 개발할 때 유용하다. 이로써 동일한 코드를 여러 플랫폼에 쉽게 이식할 수 있다.
둘째, 언어 독립적. OpenGL은 C, Python, Java, Javascript등 다양한 언어에서 사용할 수 있다.
셋째, 높은 성능. OpenGL은 비교적 간단한 API 구조를 가지고 있어 학습하기가 쉽다. 이는 OpenGL을 처음 접하는 개발자들에게 유용하다.
넷째, 하이레벨 API. 개발자들이 쉽게 학습할 수 있으며, 상대적으로 빠른 개발이 가능하게 한다.

>###	1) WebGL
>WebGL은 웹 브라우저에서 3D 그래픽을 렌더링하기 위한 로우레벨 JavaScript 	API이다. OpenGL ES를 기반으로 브라우저 엔진에 내장된 HTML5 Canvas 요소 위에 	그려진다. 주로 웹 기반 게임 및 시뮬레이션을 개발하는 데 사용된다. 다음은 	WebGL의 주요 특징이다.
첫째, 웹 브라우저 지원. WebGL은 웹 브라우저에서 실행되며, 플러그인 없이 웹에서 	3D 그래픽을 표현할 수 있다.
둘째, 웹 표준. 웹 표준 기술로 지원되어 다양한 플랫폼과 브라우저에서 호환성이 	높다. 이는 웹에서 3D 그래픽을 구현하는데 유용하다.
셋째, 자바스크립트 프로그래밍. 자바스크립트는 메모리 관리를 해주기 때문에, 	수동적으로 할당할 필요가 없다.

## 3. Vulkan
Vulkan은 OpenGL을 개발하였던 크로노스 그룹에서 개발한 로우레벨 그래픽 API이다. 크로스 플랫폼에서 최신 그래픽 기술을 활용하는 데 사용된다. 다음은 Vulkan의 주요 특징이다.
첫째, 로우레벨 API. Vulkan은 메모리 및 스레딩 관리를 직접 할 수 있어 최적화에 유리하다. 개발자가 하드웨어를 직접 제어할 수 있어, 렌더링 성능을 최대한 끌어올릴 수 있다.
둘째, 다중 스레드 및 CPU 활용. 다중 스레딩 및 다중 코어 CPU를 효과적으로 활용하여 성능을 향상시킨다. 특히 멀티스레드 CPU에서 뛰어난 성능을 보여준다.

## 4. Metal
Metal은 Apple의 iOS 및 macOS 플랫폼을 위한 로우레벨 그래픽 API로, Apple 생태계와 통합되어 최적의 성능을 제공한다. 다음은 Metal의 주요 특징이다.
첫째, Apple 플랫폼 지원. Metal은 iOS 및 macOS에서 주로 사용되며, Apple 생태계에서 좋은 성능을 낸다.
둘째, 로우레벨 API. 메모리 및 스레딩 관리를 직접 할 수 있어 최적화에 유리하다. 개발자가 하드웨어를 직접 제어할 수 있어, 렌더링 성능을 최대한 끌어올릴 수 있다.

|비교 항목|DirectX11|DirectX12|OpenGL|WebGL|Vulkan|Metal|
|---|---|---|---|---|---|---|
|개발사|Microsoft|Microsoft|Khronos Group|Khronos Group|Khronos Group|Apple|
|플랫폼|Windows|Windows 10 이상|크로스 플랫폼(Windows, macOS, Linux)|웹 브라우저 기반(크로스 플랫폼)|크로스 플랫폼(Windows, macOS, Linux)|macOS, IOS|
|API 유형|하이레벨 API|로우레벨API|하이레벨API|자바스크립트 API|하이레벨 API|로우레벨 API|
|성능|높은 성능, 오버헤드 존재|낮은 CPU 오버헤드, 뛰어난 최적화|성능 다소 떨어짐|성능 제한|최적화와 높은 성능	높은 성능|Apple 생태계에 최적|
|개발 언어|C++, C# 등|C++, C#, DirectX12 API|C, C++, Python 등|JavaScript|C, C++, Rust 등|Objective-C, Swift|
|개발도구|DirectX SDK, 라이브러리 다수|Windows SDK, DirectX Tool Kit 등|OpenGL SDK|WebGL GPI 및 라이브러리|Vulkan SDK, 라이브러리 다수	|Xcode, Metal Kit|

# 3. 결론
여기까지 그래픽 API들에 대해 알아보았다. DirectX 11 및 DirectX 12는 Windows 플랫폼에 최적화되어 있다. OpenGL 및 OpenGL ES는 크로스 플랫폼 지원을 제공하며, WebGL은 웹 브라우저에서 3D 그래픽을 사용할 때 유용하다. Vulkan은 로우레벨 API로 다중 스레딩과 CPU 활용하여 성능을 향상시키며, Metal은 Apple 생태계에서 뛰어난 성능을 보여준다.  
그러므로 그래픽 API는 개발 플랫폼, 개발자의 경험, 성능 요구 사항 및 프로젝트 목표에 따라 신중히 결정해야 할 것이다.

# 레퍼런스
[1] "제이의 미디어 유니버스." 제이의 IT 월드 다이렉트X 11과 다이렉트X 12 사이에는 정확히 어떤 차이가 있을까? n.d. 수정, 2023년 10월 6일 접속, https://stg1994.tistory.com/934.
[2] "GoFo 혼자하는 코딩." [OpenGL] OpenGL이란? n.d. 수정, 2023년 10월 6일 접속, https://gofo-coding.tistory.com/entry/OpenGL.
[3] "0hyo." WebGL이란? n.d. 수정, 2023년 10월 6일 접속, https://velog.io/@0hyo/WebGL%EC%9D%B4%EB%9E%80.
[4] "삼각형의 소프트웨어 세상." Vulkan이란? n.d. 수정, 2023년 10월 6일 접속, https://blog.naver.com/dmatrix/221808847792.
[5] "Wikipedia." Metal (API). n.d. 수정, 2023년 10월 6일 접속, https://en.wikipedia.org/wiki/Metal_(API).