---
title: "Boost libraries 정리 중"
date: 2019-02-17 21:42:00 +0900
categories: boost
---
Boost libraries 중 직접 사용해본 라이브러리에 대한 내용을 정리해 두고자 한다.
사용해본 라이브러리는 아래와 같다.

# Boost Log

python 에는 기본적으로 logging 라이브러리가 있는데, C++에는 없을까 하여 사용하게 된 라이브러이다.  
내가 사용해본 기능은 아래와 같다.
1. Thread safe log
2. log rotation
3. 파일과 콘솔에 동시에 출력
4. debug, info, warning 등의 로깅 레벨 설정

# Boost Statechart

State machine을 구현하기 위해 사용하는 라이브러리이다.  
비슷한 라이브러리로 Boost Meta State Chart가 있다.  
처음에는 Boost Meta State Chart가 성능적인 측면에서 더 낫다고 하여 사용해보려고 하였다.

예제 코드(https://theboostcpplibraries.com/boost.msm)를 보고 Macro 기반으로 구현하였으나 State와 Transition의 개수가 늘어날수록 컴파일 시간과 컴파일 시의 메모리 사용량이 엄청나게 늘어나는 문제점이 있었다.  
또한, state machine 전체의 transition table을 한 군데 만들게 구현이 되어 있는데, 이 table의 크기를 일정 수준(50개) 이상으로 늘리기 위해 따로 헤더를 만들어 주어야 한다. (Meta state chart에서는 transition table을 만들기 위해 boost::mpl::list를 사용한다. boost::mpl::list의 default size는 20이며, 추가적인 헤더 선언을 통해 늘릴 수 있으나, boost libraries 에 선언된 건 50까지만 있다. 해당 헤더를 복사한 후 일부분만 수정하여 추가하는 식으로 60, 70, ... 처럼 10 단위로 늘릴 수는 있으나, 상당히 불편하다.)
물론, 컴파일된 결과물은 실행 시간이나 실행 시의 메모리 측면에서 별다른 문제가 없었으나, State와 Transition의 개수가 너무 늘어나서 컴파일을 하기에 PC의 메모리가 모자란 경우가 발생하여 포기하고 Boost Statechart를 사용하기로 하였다.  

Boost statechart는 MSM(Meta State Machine)과 다르게 각 state에서 transition table을 관리한다. 물론, boost statechart에서도 transition table을 관리하기 위해 boost::mpl::list를 사용하나, 각 state에서 transition을 관리하기 때문에, 각각의 boost::mpl::list의 크기가 작게 유지될 수 있다.  
성능은 상대적으로 MSM에 비해 떨어지나, 극단적으로 빠른 state machine이 필요하지 않았기 때문에 크게 체감되지는 않았다. 또한, MSM에서 발생한 복잡한 state machine 컴파일 시 소요되는 시간이나 메모리 사용량은 문제가 발생하지 않았다.

# Boost Asio

Socket 프로그래밍을 하면서 Windows, Linux를 둘 다 지원하기 위해 사용하게 되었다.  
원래 boost asio의 핵심은 비동기 I/O이지만, 아직 작성중인 코드에 비동기 I/O를 적용해야 할 필요가 없었기 때문에 멀티 플랫폼 코드를 구현하는 데 사용하였다.  
기존에는 Windows용 소켓 프로그래밍 코드와 Linux용 소켓 프로그래밍 코드를 별도로 작성한 후, 아래와 같이 플랫폼에 따라 다른 코드로 컴파일되도록 구현하였다. 

    #ifdef _WIN32  
    <Windows socket code>  
    #else  
    <Linux socket code>  
    #endif  

이런 식으로 플랫폼 별로 코드를 따로 만들어 관리하면, 추가적인 변경사항이 생길 때마다 한  플랫폼은 수정하였으나 다른 플랫폼은 수정되지 않는 경우가 발생할 수 있어 관리가 힘들어진다.  
Boost asio를 사용하면 위처럼 플랫폼 별로 코드를 따로 관리할 필요가 없이 하나의 코드로 다양한 플랫폼에서 사용이 가능하다.
