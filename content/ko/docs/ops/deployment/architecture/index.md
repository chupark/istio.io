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
모든 mesh 트래픽에 대한 telemetry 를 수집하고 보고합니다..

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

This sidecar deployment allows Istio to enforce policy decisions and extract
rich telemetry which can be sent to monitoring systems to provide information
about the behavior of the entire mesh.

The sidecar proxy model also allows you to add Istio capabilities to an
existing deployment without requiring you to rearchitect or rewrite code.

Some of the Istio features and tasks enabled by Envoy proxies include:

* Traffic control features: enforce fine-grained traffic control with rich
  routing rules for HTTP, gRPC, WebSocket, and TCP traffic.

* Network resiliency features: setup retries, failovers, circuit breakers, and
  fault injection.

* Security and authentication features: enforce security policies and enforce
  access control and rate limiting defined through the configuration API.

* Pluggable extensions model based on WebAssembly that allows for custom policy
  enforcement and telemetry generation for mesh traffic.

### Istiod

Istiod provides service discovery, configuration and certificate management.

Istiod converts high level routing rules that control traffic behavior into
Envoy-specific configurations, and propagates them to the sidecars at runtime.
Pilot abstracts platform-specific service discovery mechanisms and synthesizes
them into a standard format that any sidecar conforming with the
[Envoy API](https://www.envoyproxy.io/docs/envoy/latest/api/api) can consume.

Istio can support discovery for multiple environments such as Kubernetes,
Consul, or VMs.

You can use Istio's
[Traffic Management API](/docs/concepts/traffic-management/#introducing-istio-traffic-management)
to instruct Istiod to refine the Envoy configuration to exercise more granular control
over the traffic in your service mesh.

Istiod [security](/docs/concepts/security/) enables strong service-to-service and
end-user authentication with built-in identity and credential management. You
can use Istio to upgrade unencrypted traffic in the service mesh. Using
Istio, operators can enforce policies based on service identity rather than
on relatively unstable layer 3 or layer 4 network identifiers.
Additionally, you can use [Istio's authorization feature](/docs/concepts/security/#authorization)
to control who can access your services.

Istiod acts as a Certificate Authority (CA) and generates certificates to allow
secure mTLS communication in the data plane.
