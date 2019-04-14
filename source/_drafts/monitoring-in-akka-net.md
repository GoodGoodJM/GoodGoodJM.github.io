---
title: monitoring-in-akka-net
catecories: AKKA.NET
tags:
  - AKKA.NET
  - Monitoring
---

## AKKA.NET을 사용하는걸 가장 힘들게 하는 요소 - 흐름의 파악
AKKA.NET을 쓰면서, 가장 아쉬운점은 시각적으로 액터의 생성, 메세지를 파악할 수 있는 유틸리티가 없다는 점이었다.

TOA를 개발하면서 몇번의 병목, 에러 해결 과정중 Actor모델에서 가장 어려운 점이 `어디서 어떤 순서로 문제가 발생하느냐`였다. 로그를 찍짜니 어디부터 찍을 지 엄두도 안나며, 디버깅은 사실 메세지를 넘기면서부터는 불가능한 수준이었다.

Erlang에는 [ERLANG PERFORMANCE LAB](http://www.erlang.pl/)같은 시각적 도구가 많지만, 특히 AKKA.NET에는 거의 없다.

AKKA.NET Monitoring 프로젝트가 있긴하지만, 사용하려면 모든 Actor에 Trace 메소드를 호출해주어야 하므로 귀찮고, 이또한 메세지가 어떤 흐름으로 어디로 향하는지, 액터가 어떤 흐름으로 생성되는지 같은 요소는 측정하기 어렵고 시각화 하기도 힘들다.

때문에 AKKA.NET의 Actor LifeCycle, Message Flow를 추적하여 시각화 하는 라이브러리를 만드려고 한다.

## AKKA.Tracker의 목표

- 최소한의 코드 수정으로 Tracing이 되어야 한다.
- Matric을 File로 추출해줄 수 있어야 한다.
- Visualize하여 메세지의 흐름과 양을 측정할 수 있어야 한다.

## 참고문서