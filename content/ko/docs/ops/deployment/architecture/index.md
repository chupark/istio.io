---
title: Architecture
description: Describes Istio's high-level architecture and design goals.
weight: 10
aliases:
  - /docs/concepts/architecture
  - /docs/ops/architecture
owner: istio/wg-environments-maintainers
test: n/a
---

Istio service mesh 는 **data plane** 과 **control
plane** 으로 분리되어 있습니다..

* **data plane** 은 사이드카로 배치된 지능형 
([Envoy](https://www.envoyproxy.io/)) 프록시의 집합으로 구성됩니다. 
이러한 프록시들은 마이크로서비스 사이의 모든 네트워크 통신을 조정하고 제어합니다. 또한 
모든 mesh 트래픽에 대한 telemetry 를 수집하고 보고합니다.

* **control plane** 은 트래픽을 라우팅하도록 프록시를 관리하고 구성합니다..

아래 그림은 각 평면을 구성하는 다양한 구성 요소를 보여준다. :

{{< image width="80%"
    link="./arch.svg"
    alt="The overall architecture of an Istio-based application."
    caption="Istio Architecture"
    >}}

## Components

다음 섹션에서는 Istio의 각 행심 구성 요소에 대한 간략한 설명을 제공합니다.

### Envoy

Istio 는 [Envoy](https://envoyproxy.github.io/envoy/) 프록시의 확장 버전을 사용합니다.
Envoy는 C++로 개발된 고성능 프록시로 Service mesh의 모든 서비스의 인바운드 및 아웃바운드 트래픽을 조정합니다.
Envoy프록시는 Data plane과 상호 작용하는 유일한 Istio 구성 요소입니다.

Envoy프록시는 서비스의 사이드카로 배치되며, Envoy의 다양한 내장 기능으로 서비스를 논리적으로 확장합니다.   
예시:

* Dynamic service discovery
* Load balancing
* TLS termination
* HTTP/2 and gRPC proxies
* Circuit breakers
* Health checks
* Staged rollouts with %-based traffic split
* Fault injection
* Rich metrics

사이드카로 배치함으로써 Istio가 정책을 설정하고 모니터링 시스템으로 telemetry를 전송합니다. 
모니터링 시스템은 전달받은 풍부한 메트릭으로 mesh의 행동 정보를 감시합니다.

또한 Istio는 사이드카로 작동하므로 코드를 재설계하거나 다시 작성할 필요 없이 Istio기능을 바로 추가할 수 있습니다..

Envoy 프록시에 의해 활성화되는 Istio기능 예시 :

* 트레픽 제어 기능: HTTP, gRPC, WebSocket, TCP traffic에 대한 다양한 라우팅 규칙으로 세분화된 트래픽 제어 수행.

* 네트워크 복원 기능: setup retries, failovers, circuit breakers, and
  fault injection.

* 보안 및 인증 기능: API를 사용하여 정의된 보안정책, 접근 제어 정책, 속도 제한 정책을 적용 합니다.

* 메쉬 트래픽에 대한 사용자 지정 정책 시행 및 원격 측정 생성을 허용하는 WebAssembly 기반 플러그 가능 확장 모델.

### Istiod

Istiod는 service discovery 설정과 인증서 관리 기능을 제공합니다.

Istiod 높은 수준의 라우팅 규칙은 Envoy-specific 구성으로 변환하고 런타임에 사이드카로 전파합니다..
Pilot은 플랫폼 별 서비스 검색 메커니즘을 추상화 하고 [Envoy API](https://www.envoyproxy.io/docs/envoy/latest/api/api)를 준수하는 
모든 사이드카가 사용할 수 있는 표준 형식으로 합성합니다.

Istio 는Kubernetes, Consul, VM과 같은 여러 환경에 대하여 discovery를 지원합니다.

Istio의 
[Traffic Management API](/docs/concepts/traffic-management/#introducing-istio-traffic-management)
를 사용하여 Istod에서 Envoy 구성을 구체화하여 service mesh의 트래픽을 세밀하게 제어하도록 지시할 수 있습니다.  


Istiod [security](/docs/concepts/security/)는 내장 된 ID 및 자격 증명 관리를 통해 강력한 서비스 대 서비스 및 최종 사용자 인증을 지원합니다. 
Istio를 사용하여 서비스 메시에서 암호화되지 않은 트래픽을 업그레이드 할 수 있습니다. 
운영자는 Istio를 사용하여 상대적으로 불안정한 계층 3 또는 계층 4 네트워크 식별자가 아닌 서비스 ID를 기반으로 정책을 시행 할 수 있습니다. 
릴리스 0.5부터 Istio의 [Istio's authorization feature](/docs/concepts/security/#authorization) 기능을 사용하여 서비스에 액세스 할 수있는 사용자를 제어 할 수 있습니다.

Istiod는 인증 기관 (CA)의 역할을하며 data plane에서 안전한 mTLS 통신을 허용하는 인증서를 생성합니다.
