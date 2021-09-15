## GRASP

### GRASP - General Responsibility Assignment Software Patterns
- 책임 기반 객체지향 관점에서 객체에 책임을 할당하기 위한 패턴을 정리한 것을 의미
- [책임할당에 기반한 객체 설계 원칙](http://contents.kocw.or.kr/KOCW/document/2014/Seowon/SongHaesang/08.pdf) : applying uml and patterns 책의 GRASP 설명 정리한 내용
- GRASP : GRASP을 잘 설명하고 있는 영어로 된 문서

### Information Expert
- 역할을 수행할 수 있는 정보를 가지고 있는 객체에 역할을 부여하자.
- Problem: What is a basic principle by which to assign responsibilities to objects?
- Solution: Assign a responsibility to the class that has the information needed to fulfill it.

### Creator
- 객체의 생성은 생성되는 객체의 컨텍스트를 알고 있는 다른 객체가 있다면, 컨텍스트를 알고 있는 객체에 부여하자.
- Problem: Who creates object A?
- Solution: Assign class B the responsibility to create object A if one of these is true (more is better)
  - B contains or compositely aggregates A
  - B records A
  - B closely uses A
  - B has the initializing data for A
  
### Controller
- 시스템 이벤트(사용자의 요청)를 처리할 객체를 만들자. 시스템, 서브시스템으로 들어오는 외부 요청을 처리하는 객체를 만들어 사용하라.
- Problem: What first object beyond the UI layer receives and coordinates “controls” a system operation?
- Solution: Assign the responsibility to an object representing one of these choices:
  - Represents the overall “system”, “root object”, device that the software is running within, or a major subsystem (these are all variations of a facade controller)
  - Represents a use case scenario within which the system operation occurs (a use case or session controller)

### Low Coupling
- 객체들간, 서브 시스템들간의 상호의존도가 낮게 역할을 부여하자.
- Problem: How to reduce the impact of change? How to support low dependency and increased reuse?
- Solution: Assign responsibilities so that (unnecessary) coupling remains low. Use this principle to evaluate alternatives.

### High Cohesion
- 각 객체가 밀접하게 연관된 역할들만 가지도록 역할을 부여하자.
- Problem: How to keep objects focused, understandable, manageable and as a side effect support Low Coupling?
- Solution: Assign a responsibility so that cohesion remains high. Use this to evaluate alternatives.

### Indirection
- 두 객체 사이의 직접적인 Coupling을 피하고 싶으면, 그 사이에 다른 객체를 사용하라.
- Problem: Where to assign a responsibility to avoid direct coupling between two or more things?
- Solution: Assign the responsibility to an intermediate object to mediate between other components or services so that they are not directly coupled.

### Polymorphism
- 객체의 종류에 따라 행동양식이 바뀐다면, Polymorphism 기능을 사용하자.
- Problem: How handle alternatives based on type?
- Solution: When related alternatives or behaviors vary by type (class), assingn responsibility for the behavior (using polymorphi operations) to the types for which the behavior varies.

### Pure Fabrication
- Information Expert 패턴을 적용하면 Low Coupling과 High Cohesion의 원칙이 깨어진다면, 기능적인 역할을 별도로 한 곳으로 모으자.
- Problem: What object should have the responsibility, when you do not want to viloate High Cohesion and Low Coupling but solutions offered by other principles are not appopriate?>
- Solution: Assign a highly cohesive set of responsibilites to an artifical or convenience class that does not represent a problem domain conecept.

### Protected Variations
- 변경될 여지가 있는 곳에 안정된 인터페이스를 정의해서 사용하자.
- Problem: How to design objects, subsystems and systems so that the variations or instability in these elements does not have an undesirable impact on other elements?
- Solution: Identify points of predicted variation or instability, assign responsibilities to create a stable interface around them.
