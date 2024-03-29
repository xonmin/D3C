# chap 31 웹은 세부사항이다. 

웹이 있기 전엔 클라이언트-서버 아키텍처가 존재 
- 그 전에는 중앙집중식 미니컴퓨터 존재 ...
- 과거로 쭉 가면서 IT 전체로 시야를 넓히면 웹은 아무것도 바꾸지 않았음 

업무 규칙을 UI로부터 분리해야 하므로, GUI는 세부사항. 이 때 웹은 GUI일 뿐. 
따라서 웹은 세부사항이기 때문에 아키텍트라면 핵심 업무 로직에서 분리된 경계 바깥에 두어야 함 


js의 유효성 검증 및 드래그앤드롭 방식의 AJAX 호출등에 대해서 웹에서 장치 독립성은 비현실적으로 주장할 수 있음 
- 어느정도 옳은 주장(상호작용은 빈번하며, 상호작용은 사용중인 GUI에 따라 방식이 달라짐) 

하지만, UI와 어플리케이션 사이에는 추상화가 가능한 또 다른 경계가 존재한다.
- 업무 로직은 다수의 유스케이스로 구성 
- 각 유스케이스는 사용자를 대신하여 일부 함수를 수행 
- 각 유스케이스는 입력 데이터, 수행할 처리 과정, 출력 데이터를 기반으로 기술 가능 

# chap 32 프레임워크는 세부사항이다. 

사용자와 프레임워크 제작자 사이의 관계는 굉장히 비대칭적 
- 사용자는 프레임워크를 사용하기 위해서 도큐먼트에 리소스를 투자하고 굉장한 큰 헌신을 함 
- 대게 프레임워크를 중심에 두고 사용자의 아키텍처는 그 바깥을 감싸야 한다고 말함 
  - 프레임워크의 기능을 바로 import 사용이 그 예시 
- 하지만 이렇게 되면, 서비스는 프레임워크에 절대적인 제어권이 넘어가게 된다. 


### 위험 요소 
- 프레임워크의 아키텍처는 그다지 깔끔하지 않은 경우가 많음 
- 프레임워크는 애플리케이션의 초기 기능을 만드는 데는 도움이 되지만, 제품이 성숙해짐녀서 프레임워크가 제공하는 기능과 틀을 벗어나게 된다. 
- 프레임워큰느 당신에게 도움되지 않는 방향으로 업그레이드 될 수 있음 
- 새롭고 더 나은 프레임워크가 등장하여 갈아타야 할 경우가 있음 

### 해결 방법
프레임워크와 결혼하지 말라 
- 업무 객체를 만들 때 프레임워크의 기반 클래스에서 파생되지말고 proxy 를 만들어 사용하라 
  - ex) 업무 객체는 절대로 스프링에 대해 알아서는 안된다. 
  - 업무 객체보다는 메인 컴포넌트에서 스프링을 통해 의존성을 주입

### 이제 선언합니다. 
- C++ 일 경우 STL과 결혼해야만 함 
- 자바를 사용한다면 표준 라이브러리와는 반드시 결혼해야 한다. 


