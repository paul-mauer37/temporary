# Semantic Web(시맨틱 웹) - 의미론적 웹



> 컴퓨터가 이해하고 상호 소통할 수 있는 지능형 웹 
>
> 기존의 키워드 기반 검색은 Syntactic Web으로 네이버의 검색 횟수별 우선순위 처리가 있다.

리소스에 대한 정보와 자원 사이의 **관계-의미 정보(Semanteme)를 해석**하고 **시맨틱 정보를 활용**해 복잡한 의미적 연결과 이에 기반한 처리가 가능하다.

* 수많은 웹 페이지에 Metadata(메타데이터)를 부여하여 '의미'와 **'관련성'**을 갖는 DB로 구축한다(**온톨로지**)

* 방법 - 태깅

  * Semantic 요소 : form, table, img 태그 등

  * Non-Semantic 요소 : div, span 태그 등

    > HTML5에서 시맨틱 웹 기능이 향상됨(ex. header 태그)
    >
    > Microsoft, Flicker, Rojo 등에서 태깅 기술과 방법에 대한 연구가 진행되고 있다.



> cf. 시맨틱 웹으로서 HTML5 ref) https://sir.kr/g5_tip/14891





### Ontology(온톨로지)



"정보 자원을 컴퓨터가 해석할 수 있는 **시맨틱**으로 표현한 특정 영역의 **메타데이터**"

Ontology(온톨로지)는 사물 간의 관계 및 여러 개념을 컴퓨터가 처리할 수 있는 형태로 표현한 것이다. 온톨로지는 개념화 과정을 거쳐 완성되는데, 실생활에서 예를 살펴보자.

![img](https://t1.daumcdn.net/cfile/tistory/2524554852EF56D011)



![img](https://t1.daumcdn.net/cfile/tistory/26183F4C52EF572023)



![img](https://t1.daumcdn.net/cfile/tistory/271B6C3C52EF579524)



![img](https://t1.daumcdn.net/cfile/tistory/221EF83952EF57F529)



![img](https://t1.daumcdn.net/cfile/tistory/25494F3452EF586F11)



개념화 과정을 거쳐 완성된 온톨로지를 통해 우리는 "우유를 마시지 못하는 골다공증환자에게 무엇을 서비스해야 하나?"와 같은 의문에 "우유는 칼슘이 들어있기 때문에 두유로 대체할 수 있다"는 결론을 얻을 수 있다.

다시 정의해, **온톨로지는 개념과 개념 간의 관계로 특정 영역 혹은 세계를 표현한 것이다.**



---



## 시맨틱 웹 기술의 2가지 흐름



* RDF(Resource Description Framework)를 기반으로 한 온톨로지 기술
* ISO(국제표준화기구)중심의 Topic Map(토픽 맵) 기술



#### W3C 중심 - RDF 기반의 온톨로지 기술

**RDF**는 특정 자원에 대한 메타데이터를 기술하는 **XML기반 프레임워크**

* 자원, 속성, 속성 값을 하나의 record단위로 취급하는 'Triple개념'
  * 자원 - 주어(Subject)
  * 속성 - 술어(Predicate)
  * 속성 값 - 목적어(Object)

기초가 되는 RDF를 기반으로, 기술 대상이 되는 record가 어떤 클래스에 속하며, 해당 record를 기술하는데 필요한 속성에 대한 정의를 위한 스키마 언어가 **RDF-S**이다.

> RDF-S는 클래스와 클래스 간의 관계, 속성과 속성 간의 관계 그리고 기계 처리 가능한 형태로 메타데이터의 속성과 클래스 간의 관계 표현이 가능하다.

모델링 요소를 보다 확장하고 강화한 것이 온톨로지 언어(**DAML+OIL**, **OWL(Ontology Web Language)**)



#### ISO 중심 - Topic Map

RDF와 마찬가지로 **XML을 기반**으로 하며 XTM(XML Topic Maps) 언어를 사용해 정보와 지식의 분산 관리를 지원한다.

* 지식측과 정보층의 이중 구조
  * 지식측 - 상위 계층 && 토픽(topic)과 토픽간의 연계(association)로 구성(즉, 특정 주제와 주제들 간의 관계)
  * 정보층 - 디지털 콘텐츠를 나타낸다
  * 지식측과 정보층은 Occurrence를 통해 상호 연결 



---



시맨틱 웹 구현을 위한 DB 중요성 이유































































