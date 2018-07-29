# Intro

---

이 글은 "Detecting Concurrency Bugs in Multithreaded Programs" 라는 주제로 진행된 세미나의 내용을 간략히 정리한 것이다. 특히 이번 포스팅은 Multithread programming에 대한 이해를 목표로 하고 있다.

본격적으로 Multithreaded programming에 대한 내용을 다루기에 앞서, Concurrency Bug가 무엇인지 정의하고 넘어가고자 한다. 왜냐하면 우리가 궁극적으로 다루고자 하는 내용은 Concurrency Bug를 탐지하는 것이기 때문이다.

> Concurrency Bug is a bug that occurs when Multithread is used.  

   



​     

   



​     



# Multithreaded Programming

---

Multithreaded Programming이란 여러 개의 스레드를 사용하여 동작하는 프로그램을 말한다. 하나 이상의 스레드를 이용한 프로그램을 짜본 적이 있는 사람이라면, 그것이 생각보다 쉽지 않은 일이라는 것을 경험을 통해 알 수 있을 것이다.

실제로, Multithreaded Programming은 어렵다. 그 이유는 이러한 종류의 프로그램이 가지고 있는 특징과 밀접한 관련이 있다. 다음으로는 Multithreaded Programming이 가지고 있는 특징에 대해 살펴보며 동시에 그것이 가지는 어려움에 대해 알아보려 한다.

  

## 1) Non-deterministic

Multithread programing은 non-deterministic하다.   
이 말은 여러 개의 스레드를 사용하는 상황에서는 같은 입력에 대하여 동일한 출력을 기대하기. 어렵다는 것을 의미한다. 


영화 **매트릭스**를 본 적이 있는가?  
위 영화에서는 로봇이 사람을 지배하고자 한다. 로봇은 사람이 행동할 수 있는 경우의 수를 예상하고 사람으로 하여금 자신이 원하는 행동을 취하게 만든다. 하지만, 몇몇 사람들은 로봇이 통제하는 대로 움직여주지 않고 스스로의 의지에 따라 행동한다.  

여기서 필자가 강조하고 싶은 것은 몇몇의 통제되지 않는 사람들이다. 로봇이 통제하는 대로 움직여주지 않는 사람들은 프로그래머의 생각대로 동작해주지 않는 Multithreaded Program과 같다고 할 수 있겠다.

  

## 2) Number of execution path가 기하급수적으로 증가한다.

이는 Thread scheduling이 임의로 이루어진다는 가정을 하기 때문이다.   
Number of execution path는 발생할 수 있는 상황을 의미하며 그에 따라 발생 가능한 오류도 많아짐을 의미한다.  
게다가, 수많은 execution path 중 잘못된 것이 하나라도 있으면 그것은 오류로 직결된다고 볼 수 있기 때문에 오류가 없는 상황을 보장하는 것은 쉽지 않은 일이 분명하다.

​     



​     

   





# Background of Multithreaded Programming

----

## Moore's Law began to fall apart

20'c 초반부터 moore's law가 성립하지 않기 시작했다.



> **Moore's law**란? 
> 반도체 집적도가 1년 반마다 2배씩 증가한다는 법칙이다. 집적도가 증가한다는 것은 트랜지스터 간의 간격이 좁아져서 소켓이 더 빨라질 수 있다는 것을 의미하며 궁극적으로는 성능의 향상과 직결되었다.
>
> 즉, Moore's law가 성립할 당시에는 시간이 흐름에 따라 하드웨어의 성능이 점점 좋아졌다. 그렇기 때문에 어떤 경우에는 더 나은 프로그램을 만들기 위해서 개발을 하는 것보다 하드웨어의 성능이 좋아지기를 기다리는 것이 더 효과적이라는 말도 있었다.



Moore's law가 더 이상 성립할 수 없었던 데는 크게 두 가지 이유가 있었다.

1. **Semi-Container**를 만드는 어려움

   반도체의 집적도는 일정 수준에 도달하고 난 후로는 그 이상 증가하지 못했다. 왜냐하면 semi-container를 만드는 것이 어려웠기 때문이다.  
    c.f.) 양자역학에서 어느 정도로 빛을 쪼개다 보면 빛이 non-linear하게 나타나는 현상

2. 발열로 인한 **임피던스**

   전기가 원하치 않는 때에 흘러버려서 오류가 발생할 수 있다.

  

   

## Free Lunch is over

Moore's law가 더 이상 성립하지 않게 된 시점부터 진보적인 하드웨어는 한계를 맞이하게 되었고, 마침내 소프트웨어로 경쟁을 하는 것이 관건이 되었다. 더 복잡하고 빠른 소프트웨어를 만들기 위해서 점점 더 많은 수의 코어가 사용되었고, 그 코어를 모두 사용할 수 있는 multithreaded programming이 각광 받기 시작했다.

이 사실은 20'c 초반에 multithreaded programming으로 구현된 application이 대폭 증가한 것을 통해 다시 한 번 확인할 수 있다. 즉, 20'c 초부터 하드웨어의의 변화에 맞춰서 소프트웨어의 발전이 필요했던 것이다.

  

  

## But, Multithread Programming is difficult 

경쟁력을 획득하기 위해 병렬 프로그래밍을 사용하는 것은 불가피했다. 하지만, 병렬 프로그래밍은 쉬운 것이 아니었고 쉽게 해결하기 어려운 무수한 오류를 만들어 냈다.  
일부 회사는 제한된 시간에 프로그램을 완성하기 위해 병렬 프로그래밍으로 부터 발생하는 오류를 해결하는 것보다 싱글코어만을 사용하여 안전성 있는 프로그램을 만드는 것을 선택하기도 하였다. (하드웨어는 멀티코어임에도 불구하고 말이다.)

자연스럽게 Multithread를 사용하는 소프트웨어의 버그를 탐지하기 위한 기술이 발달하였다.   
''다양한 thread schedule을 통해 버그를 찾아보자!'' 라는 생각을 바탕으로 기술은 점차 발전하였고, 시행착오 끝에 여러 가지 방법으로 Multithread program을 테스트할 필요가 있다는 결론이 지어졌다. 그럼에도 불구하고, 예외적인 상황은 언제나 발생할 수 있다는 게 해결해야 할 과제로 남아 있다.