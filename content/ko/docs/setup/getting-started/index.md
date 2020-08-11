---
title: Getting Started
description: Try Istio’s features quickly and easily.
weight: 5
aliases:
    - /docs/setup/kubernetes/getting-started/
    - /docs/setup/kubernetes/
    - /docs/setup/kubernetes/install/kubernetes/
keywords: [getting-started, install, bookinfo, quick-start, kubernetes]
owner: istio/wg-environments-maintainers
test: yes
---

이 가이드는를 통해 Istio를 빠르게 사용해보고 평가할 수 있습니다. 이미 Istio에 익숙하거나 다른 구성 프로필 혹은 다른
[배포 모델](/docs/ops/deployment/deployment-models/)에 관심이 있는 경우 대신 [Customizable Install with `istioctl`](/docs/setup/install/istioctl/)
를 참조하여 설치하세요.

이 단계를 수행하려면 호환되는 Kubernetes 버전을 실행하는 {{< gloss >}}cluster{{< /gloss >}} 가 필요합니다.
[Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)를 사용하거나 [클라우드사업자](/docs/setup/platform-setup/)의 Kubernetes 서비스를 사용할 수 있습니다..

가이드의 순서는 아래와 같습니다. :

1. [Download and install Istio](#download)
1. [Deploy the sample application](#bookinfo)
1. [Open the application to outside traffic](#ip)
1. [View the dashboard](#dashboard)

## Download Istio {#download}

1.  [Istio release]({{< istio_release_url >}})페이지에서 OS에 맞는 파일을 다운로드 받거나 아래 명령으로 최신 버전을 다운받을 수 있습니다. (Linux or macOS):

    {{< text bash >}}
    $ curl -L https://istio.io/downloadIstio | sh -
    {{< /text >}}

    {{< tip >}}
    위의 명령은 Istio의 최신 릴리스를(숫자 기준) 다운로드 합니다.
    특정 버전을 다운로드 하려면 CLI에서 변수를 추가 할 수 있습니다.
    예를 들어 Istio 1.4.3 버전을 다운받고 싶다면 다음을 실행합니다. 
      `curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.4.3 sh -`
    {{< /tip >}}

1.  Istio 패키지 디렉토리로 이동합니다. 이 예시의 경우 `istio-{{< istio_full_version >}}` 입니다. :

    {{< text syntax=bash snip_id=none >}}
    $ cd istio-{{< istio_full_version >}}
    {{< /text >}}

    디렉토리에는 아래 항목을 포함하고 있습니다. :

    - 예제 애플리케이션은 `samples/` 디렉토리에 있습니다.
    - [`istioctl`](/docs/reference/commands/istioctl) 클라이언트 바이너리는 `bin/` 디렉토리에 있습니다.

1.  `istioctl` 클라이언트 환경변수를 사용자 환경에 추가합니다. (Linux or macOS):

    {{< text bash >}}
    $ export PATH=$PWD/bin:$PATH
    {{< /text >}}

## Install Istio {#install}

1.  이 예제에선 `demo` [설정 프로파일](/docs/setup/additional-setup/config-profiles/)을 사용합니다.
    테스트하기 좋은 기본값을 선택하고 있지만, 프러덕션 또는 성능 테스트를 위한 다른 프로필도 준비되어 있습니다.

    {{< text bash >}}
    $ istioctl install --set profile=demo
    ✔ Istio core installed
    ✔ Istiod installed
    ✔ Egress gateways installed
    ✔ Ingress gateways installed
    ✔ Installation complete
    {{< /text >}}

1.  namespace label을 추가하여, 애플리케이션을 배포 할 때 Istio가 Envoy 사이드카 프록시를 자동으로 배포되도록 합니다. :

    {{< text bash >}}
    $ kubectl label namespace default istio-injection=enabled
    namespace/default labeled
    {{< /text >}}

## Deploy the sample application {#bookinfo}

1.  [`Bookinfo` 얘제 애플리케이션](/docs/examples/bookinfo/)을 배포합니다. :

    {{< text bash >}}
    $ kubectl apply -f @samples/bookinfo/platform/kube/bookinfo.yaml@
    service/details created
    serviceaccount/bookinfo-details created
    deployment.apps/details-v1 created
    service/ratings created
    serviceaccount/bookinfo-ratings created
    deployment.apps/ratings-v1 created
    service/reviews created
    serviceaccount/bookinfo-reviews created
    deployment.apps/reviews-v1 created
    deployment.apps/reviews-v2 created
    deployment.apps/reviews-v3 created
    service/productpage created
    serviceaccount/bookinfo-productpage created
    deployment.apps/productpage-v1 created
    {{< /text >}}

1.  애플리케이션이 시작되고, 각 Pod가 ready 상태가 되면 Istio사이드카가 배포됩니다.

    {{< text bash >}}
    $ kubectl get services
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.0.0.212      <none>        9080/TCP   29s
    kubernetes    ClusterIP   10.0.0.1        <none>        443/TCP    25m
    productpage   ClusterIP   10.0.0.57       <none>        9080/TCP   28s
    ratings       ClusterIP   10.0.0.33       <none>        9080/TCP   29s
    reviews       ClusterIP   10.0.0.28       <none>        9080/TCP   29s
    {{< /text >}}

    and

    {{< text bash >}}
    $ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    details-v1-558b8b4b76-2llld       2/2     Running   0          2m41s
    productpage-v1-6987489c74-lpkgl   2/2     Running   0          2m40s
    ratings-v1-7dc98c7588-vzftc       2/2     Running   0          2m41s
    reviews-v1-7f99cc4496-gdxfn       2/2     Running   0          2m41s
    reviews-v2-7d79d5bd5d-8zzqd       2/2     Running   0          2m41s
    reviews-v3-7dbcdcbc56-m8dph       2/2     Running   0          2m41s
    {{< /text >}}

    {{< tip >}}
    다음 단계로 이동하기 전에 이전 명령을 다시 실행하여 모든 Pod의 STATUS가 READY 2 / 2 그리고
     Running 인지 확인하세요. 이 작업은 플랫폼에 따라 몇 분 정도 걸립니다.
    {{< /tip >}}

1.  이 시점에서 모든 작업이 잘 진행됐는지 검증 합니다. 아래 명령을 실행하여 page title HTML태그를 반환하는지 확인합니다. :

    {{< text bash >}}
    $ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -s productpage:9080/productpage | grep -o "<title>.*</title>"
    <title>Simple Bookstore App</title>
    {{< /text >}}

## Open the application to outside traffic {#ip}

Bookinfo 애플리케이션이 배포됐지만 아직 외부에서 접속이 가능하지 않습니다. 외부 접속을 허용하기 위해 
mesh의 edge에 라우팅 경로를 매핑시킬 [Istio Ingress Gateway](/docs/concepts/traffic-management/#gateways)를 배포합니다.

1.  Istio gateway 를 배포하기 위해 아래 명령을 실행합니다. :

    {{< text bash >}}
    $ kubectl apply -f @samples/bookinfo/networking/bookinfo-gateway.yaml@
    gateway.networking.istio.io/bookinfo-gateway created
    virtualservice.networking.istio.io/bookinfo created
    {{< /text >}}

1.  설정에 이상이 없는지 확인합니다. :

    {{< text bash >}}
    $ istioctl analyze
    ✔ No validation issues found when analyzing namespace: default.
    {{< /text >}}

### Determining the ingress IP and ports

다음 순서를 따라서 gateway를 위한 `INGRESS_HOST` 그리고 `INGRESS_PORT` 변수를 설정합니다. 
아래 탭을 선택하여 배포된 플랫폼에 맞는 순서를 진행합니다. :

{{< tabset category-name="gateway-ip" >}}

{{< tab name="Minikube" category-value="external-lb" >}}

Ingress 포트를 설정합니다. :

{{< text bash >}}
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
{{< /text >}}

포트 번호가 환경변수에 매핑됐는지 확인합니다. :

{{< text bash >}}
$ echo "$INGRESS_PORT"
32194
{{< /text >}}

{{< text bash >}}
$ echo "$SECURE_INGRESS_PORT"
31632
{{< /text >}}

Ingress IP를 설정합니다.:

{{< text bash >}}
$ export INGRESS_HOST=$(minikube ip)
{{< /text >}}

IP 주소가 환경변수에 매핑됐는지 확인합니다. :

{{< text bash >}}
$ echo "$INGRESS_HOST"
192.168.4.102
{{< /text >}}

새 터미널 창에서 이 명령을 실행하여 Istio Ingress Gateway로 트래픽을 보내는 Minikube 터널을 시작합니다. :

{{< text bash >}}
$ minikube tunnel
{{< /text >}}

{{< /tab >}}

{{< tab name="Other platforms" category-value="node-port" >}}

배포된 Kubernetes cluster 환경이 External load balancers를 지원하면 아래 명령을 실행합니다. :

{{< text bash >}}
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   172.21.109.129   130.211.10.121  80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
{{< /text >}}

`EXTERNAL-IP` 값이 설정되어 있으면, 그 환경은 External load balancer를 지원하므로 Ingress gateway에 매핑시킬 수 있습니다..
`EXTERNAL-IP` 값이 `<none>` 이라면 (혹은 `<pending>`), Ingress Gateway에 External load balancer를 사용할 수 없습니다..
이럴 경우, service의 [node port](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport)모드를 사용하여 배포할 수 있습니다.

환경에 맞는 방법을 선택하세요 :

**External load balancer를 지원하는 환경이라면 아래 방법을 따릅니다.**

Ingress IP 와 port 를 지정합니다. :

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
{{< /text >}}

{{< warning >}}
몇몇 환경에서 Load balancer는 IP 주소 대신 호스트 이름을 사용하여 노출될 수 있습니다.
이 경우 Ingress gateway의 `EXTERNAL-IP` 값은 IP 주소가 아니라 호스트 이름이며, 
위의 명령은 `INGRESS_HOST` 환경 변수 설정에 실패 합니다.
아래 명령을 사용하여 `INGRESS_HOST` 값을 설정 합니다. :

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
{{< /text >}}

{{< /warning >}}

**External load balancer를 지원하지 않는 환경의 경우 아래 방법을 사용하고 node port 서비스를 사용합니다.**

Ingress IP 와 port 를 지정합니다. :

{{< text bash >}}
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
{{< /text >}}

_GKE:_

{{< text bash >}}
$ export INGRESS_HOST=workerNodeAddress
{{< /text >}}

`Ingressgateway` 서비스의 포트에 대한 TCP 트래픽을 허용하려면 방화벽 규칙을 만들어야합니다. 다음 명령을 실행하여 HTTP 포트, 보안 포트 (HTTPS) 또는 둘 모두에 대한 트래픽을 허용합니다. :

{{< text bash >}}
$ gcloud compute firewall-rules create allow-gateway-http --allow "tcp:$INGRESS_PORT"
$ gcloud compute firewall-rules create allow-gateway-https --allow "tcp:$SECURE_INGRESS_PORT"
{{< /text >}}

_IBM Cloud Kubernetes Service:_

{{< text bash >}}
$ ibmcloud ks workers --cluster cluster-name-or-id
$ export INGRESS_HOST=public-IP-of-one-of-the-worker-nodes
{{< /text >}}

_Docker For Desktop:_

{{< text bash >}}
$ export INGRESS_HOST=127.0.0.1
{{< /text >}}

_Other environments:_

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
{{< /text >}}

{{< /tab >}}

{{< /tabset >}}

1.  `GATEWAY_URL` 설정:

    {{< text bash >}}
    $ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
    {{< /text >}}

1.  IP 주소와 Port 가 정확하게 매핑됐는지 확인합니다. :

    {{< text bash >}}
    $ echo "$GATEWAY_URL"
    192.168.99.100:32194
    {{< /text >}}

### Verify external access {#confirm}

브라우저를 사용하여 Bookinfo 페이지에 접속하여 외부에서 Bookinfo 애플리케이션에 접근이 가능한지 확인합니다.

1.  아래 명령을 실행하여 Bookinfo 페이지의 주소를 확인합니다.

    {{< text bash >}}
    $ echo http://"$GATEWAY_URL/productpage"
    {{< /text >}}

1.  위의 주소를 복사하여 브라우저의 주소창에 붙여넣고 페이지로 이동합니다. 외부에서 접속이 가능하면 페이지가 열립니다.

## View the dashboard {#dashboard}

Istio `demo` 설치로 설치된 [몇 가지](/docs/ops/integrations) 대시보드가 있습니다. 
Kiali 대시 보드는 토폴로지를 표시하고 메시 상태를 표시하여 Service mesh 구조를 이해하는 데 도움이됩니다.

1.  Kiali 대시보드에 접속합니다. 기본 사용자 이름과 비밀번호는 모두 `admin` 입니다.

    {{< text bash >}}
    $ istioctl dashboard kiali
    {{< /text >}}

1.  기본적으로 localhost로 실행되며 초기 ID/PW 는 모두 `admin` 입니다.

    {{< text bash >}}
    $ istioctl dashboard kiali
    {{< /text >}}

1.  외부 서비스로 실행시키려면 아래 Flag를 사용하여 대시보드를 실행할 수 있습니다.

    {{< text bash >}}
    $ istioctl dashboard kiali --address (your-ip) -p (your-port)
    {{< /text >}}

1.  왼쪽 네비게이션 메뉴에서 _Graph_ 를 선택하고 _Namespace_ drop down 메뉴에서 _default_ 를 선택합니다.

    Kiali 대시보드는 `Bookinfo` 예제 애플리케이션의 서비스 간 관계와 함께 mesh에 대한 개요를 보여줍니다. 
    트래픽 흐름을 시각화하는 필터도 제공합니다.

    {{< image link="./kiali-example2.png" caption="Kiali Dashboard" >}}

## Next steps

여기까지 완료한 것을 축하합니다!

아래 작업들은 Istio를 처음 접하는 사용자들이 `demo` 를 사용하여 Istio의 특징을 더 자세히 파악하기에 좋습니다. :

- [Request routing](/docs/tasks/traffic-management/request-routing/)
- [Fault injection](/docs/tasks/traffic-management/fault-injection/)
- [Traffic shifting](/docs/tasks/traffic-management/traffic-shifting/)
- [Querying metrics](/docs/tasks/observability/metrics/querying-metrics/)
- [Visualizing metrics](/docs/tasks/observability/metrics/using-istio-dashboard/)
- [Rate limiting](/docs/tasks/policy-enforcement/rate-limiting/)
- [Accessing external services](/docs/tasks/traffic-management/egress/egress-control/)
- [Visualizing your mesh](/docs/tasks/observability/kiali/)

사용자 정의 Istio를 프러덕션에 적용하기 전 아래 내용을 참고하세요. :

- [Deployment models](/docs/ops/deployment/deployment-models/)
- [Deployment best practices](/docs/ops/best-practices/deployment/)
- [Pod requirements](/docs/ops/deployment/requirements/)
- [General installation instructions](/docs/setup/)

## Join the Istio community

We welcome you to ask questions and give us feedback by joining the
[Istio community](/about/community/join/).

## Uninstall

To delete the `Bookinfo` sample application and its configuration, see
[`Bookinfo` cleanup](/docs/examples/bookinfo/#cleanup).

The Istio uninstall deletes the RBAC permissions and all resources hierarchically
under the `istio-system` namespace. It is safe to ignore errors for non-existent
resources because they may have been deleted hierarchically.

{{< text bash >}}
$ istioctl manifest generate --set profile=demo | kubectl delete --ignore-not-found=true -f -
{{< /text >}}

The `istio-system` namespace is not removed by default.
If no longer needed, use the following command to remove it:

{{< text bash >}}
$ kubectl delete namespace istio-system
{{< /text >}}
