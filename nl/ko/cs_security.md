---

copyright:
  years: 2014, 2017
lastupdated: "2017-12-13"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}


# {{site.data.keyword.containerlong_notm}}에 대한 보안
{: #cs_security}

위험성 분석과 보안 보장을 위해 기본 제공 보안 기능을 사용할 수 있습니다. 이러한 기능을 사용하면 클러스터 인프라 및 네트워크 통신을 보호하고 컴퓨팅 리소스를 격리하며 인프라 컴포넌트 및 컨테이너 배치에서 보안 준수를 보장하는 데 도움이 됩니다.
{: shortdesc}

## 클러스터 컴포넌트에 의한 보안
{: #cs_security_cluster}

각 {{site.data.keyword.containerlong_notm}} 클러스터에는 [마스터](#cs_security_master) 및 [작업자](#cs_security_worker) 노드에 기본 제공되는 보안 기능이 있습니다. 방화벽이 있는 경우, 클러스터 외부에서 로드 밸런싱에 액세스해야 하는 경우 또는 회사 네트워크 정책이 공용 인터넷 엔드포인트에 대한 액세스를 방지할 때 로컬 시스템에서 `kubectl` 명령을 실행하려는 경우, [방화벽에서 포트를 여십시오](#opening_ports). 클러스터의 앱을 온프레미스 네트워크 또는 클러스터 외부의 기타 앱에 연결하려면 [VPN 연결을 설정](#vpn)하십시오.
{: shortdesc}

다음 다이어그램에서 Kubernetes 마스터, 작업자 노드 및 컨테이너 이미지에서 그룹화된 보안 기능을 볼 수 있습니다.

<img src="images/cs_security.png" width="400" alt="{{site.data.keyword.containershort_notm}} 클러스터 보안" style="width:400px; border-style: none"/>


  <table summary="테이블의 첫 번째 행에는 두 개의 열이 있습니다. 나머지 행은 왼쪽에서 오른쪽으로 읽어야 하며 1열에는 서버 위치, 2열에는 일치시킬 IP 주소가 있습니다.">
  <caption>표 1. 보안 기능</caption>
  <thead>
  <th colspan=2><img src="images/idea.png" alt="아이디어 아이콘"/> {{site.data.keyword.containershort_notm}}의 기본 제공 클러스터 보안 설정 </th>
  </thead>
  <tbody>
    <tr>
      <td>Kubernetes 마스터</td>
      <td>각 클러스터의 Kubernetes 마스터는 IBM에서 관리하고 가용성이 높습니다. 작업자 노드와의 양방향 보안 통신과 보안 준수를 보장하는 {{site.data.keyword.containershort_notm}} 보안 설정이 포함됩니다. 필요하면 IBM에 의해 업데이트가 수행됩니다. 전용 Kubernetes 마스터는 클러스터에서 모든 Kubernetes 리소스를 중앙 집중식으로 제어하고 모니터합니다. 클러스터의 용량 및 배치 요구사항에 따라 Kubernetes 마스터는 사용 가능한 작업자 노드 간에 배치하도록 컨테이너화된 앱을 자동으로 스케줄링합니다. 자세한 정보는 [Kubernetes 마스터 보안](#cs_security_master)을 참조하십시오. </td>
    </tr>
    <tr>
      <td>작업자 노드</td>
      <td>컨테이너는 클러스터 전용이며 IBM 고객을 위한 컴퓨팅, 네트워크 및 스토리지 격리를 보장하는 작업자 노드에 배치됩니다. {{site.data.keyword.containershort_notm}}는 공용 및 사설 네트워크에서 작업자 노드의 보안을 유지하고 작업자 노드 보안 준수를 보장할 수 있도록 기본 제공되는 보안 기능을 제공합니다. 자세한 정보는 [작업자 노드 보안](#cs_security_worker)을 참조하십시오. 또한 [Calico 네트워크 정책](#cs_security_network_policies)을 추가하여 작업자 노드의 포드에서 허용하거나 차단할 네트워크 트래픽을 구체적으로 지정할 수 있습니다.</td>
     </tr>
     <tr>
      <td>이미지</td>
      <td>클러스터 관리자인 경우에는 Docker 이미지를 저장하고 이를 클러스터 사용자 간에 공유할 수 있는 {{site.data.keyword.registryshort_notm}}의 사용자 고유의 보안 Docker 이미지 저장소를 설정할 수 있습니다. 안전한 컨테이너 배치를 보장하기 위해, 개인용 레지스트리의 모든 이미지는 Vulnerability Advisor에 의해 스캔됩니다. Vulnerability Advisor는 잠재적 취약점을 스캔하고 보안 권장사항을 작성하며 취약점 해결을 위한 지시사항을 제공하는 {{site.data.keyword.registryshort_notm}}의 컴포넌트입니다. 자세한 정보는 [{{site.data.keyword.containershort_notm}}의 이미지 보안](#cs_security_deployment)을 참조하십시오.</td>
    </tr>
  </tbody>
</table>

### Kubernetes 마스터
{: #cs_security_master}

Kubernetes 마스터를 보호하고 클러스터 네트워크 통신을 보호하기 위해 기본 제공되는 Kubernetes 마스터 보안 기능을 검토합니다.
{: shortdesc}

<dl>
  <dt>전체 관리되는 데디케이티드 Kubernetes 마스터</dt>
    <dd>{{site.data.keyword.containershort_notm}}의 모든 Kubernetes 클러스터는 IBM에서 IBM 소유 IBM Cloud 인프라(SoftLayer) 계정으로 관리하는 데디케이티드 Kubernetes 마스터에 의해 제어됩니다. Kubernetes 마스터는 다른 IBM 고객과 공유하지 않는 다음의 데디케이티드 컴포넌트로 설정됩니다. <ul><li>etcd 데이터 저장소: 서비스, 배치, 포드와 같은 클러스터의 모든 Kubernetes 리소스를 저장합니다. Kubernetes ConfigMaps 및 시크릿은 포드에서 실행되는 앱에서 사용할 수 있도록 키 값 쌍으로 저장되는 앱 데이터입니다. etcd의 데이터는 IBM에서 관리하는 암호화된 디스크에 저장되며 데이터 보호와 무결성을 보장하기 위해 포드에 전송될 때 TLS를 통해
암호화됩니다. </li>
    <li>kube-apiserver: Kubernetes 마스터에 대한 작업자 노드의 모든 요청에 대해 기본 시작점 역할을 합니다. 
kube-apiserver는 요청을 유효성 검증하고 처리하며 etcd 데이터 저장소에서 읽고 쓰기가 가능합니다. </li>
    <li>kube-scheduler: 용량 및 성능 요구사항, 하드웨어 및 소프트웨어 정책 제한조건,
연관관계 방지 스펙, 그리고 워크로드 요구사항을 고려하여 포드가 배치될 위치를 결정합니다. 
요구사항과 일치하는 작업자 노드를 찾을 수 없으면 포드가 클러스터에 배치되지 않습니다. </li>
    <li>kube-controller-manager: 원하는 상태를 얻기 위한 상대 포드의 작성과 복제본 세트의 모니터링을 담당합니다. </li>
    <li>OpenVPN: 모든 Kubernetes 마스터 대 작업자 노드 통신을 위해 보안 네트워크 연결을 제공하기 위한 {{site.data.keyword.containershort_notm}} 특정 컴포넌트. </li></ul></dd>
  <dt>모든 작업자 노드 대 Kubernetes 마스터 통신을 위한 TLS 보안 네트워크 연결</dt>
    <dd>Kubernetes 마스터에 대한 네트워크 통신을 보호하기 위해,
{{site.data.keyword.containershort_notm}}는 모든 클러스터에 대한
kube-apiserver 및 etcd 데이터 저장소 컴포넌트와의 양방향 통신을 암호화하는 TLS 인증서를 생성합니다.
이러한 인증서는 클러스터 간에 또는 Kubernetes 마스터 컴포넌트 간에 절대 공유되지 않습니다. </dd>
  <dt>모든 Kubernetes 마스터 대 작업자 노드 통신을 위한 OpenVPN 보안 네트워크 연결</dt>
    <dd>Kubernetes가 `https` 프로토콜을 사용하여 Kubernetes 마스터 및 작업자 노드 간의 통신을 보호하지만, 기본적으로 작업자 노드에서는 인증이 제공되지 않습니다. 이 통신을 보호하기 위해 {{site.data.keyword.containershort_notm}}는 클러스터가 작성될 때 Kubernetes 마스터 및 작업자 노드 간의 OpenVPN 연결을 자동으로 설정합니다. </dd>
  <dt>지속적 Kubernetes 마스터 네트워크 모니터링</dt>
    <dd>모든 Kubernetes 마스터는 프로세스 레벨 서비스 거부(DoS) 공격을 제어하고 처리하기 위해 IBM에 의해 지속적으로 모니터됩니다. </dd>
  <dt>Kubernetes 마스터 노드 보안 준수</dt>
    <dd>{{site.data.keyword.containershort_notm}}는 마스터 노드 보호를 보장하기 위해 적용이 필요한 OS 특정 보안 수정사항 및 Kubernetes에서 발견된 취약점에 대해 Kubernetes 마스터가 배치된 모든 노드를 자동으로 스캔합니다. 취약점이 발견된 경우, {{site.data.keyword.containershort_notm}}는 자동으로 수정사항을 적용하며 사용자를 위해 취약점을 해결합니다. </dd>
</dl>

<br />


### 작업자 노드
{: #cs_security_worker}

작업자 노드 환경을 보호하고 리소스, 네트워크 및 스토리지 격리를 보장하기 위해 기본 제공되는 작업자 노드 보안 기능을 검토합니다.
{: shortdesc}

<dl>
  <dt>컴퓨팅, 네트워크 및 스토리지 인프라 격리</dt>
    <dd>클러스터를 작성할 때 가상 머신이 고객 IBM Cloud 인프라(SoftLayer) 계정 또는 IBM의 데디케이티드 IBM Cloud 인프라(SoftLayer) 계정에서 작업자 노드로 프로비저닝됩니다. 작업자 노드는 클러스터 데디케이티드이며, 기타 클러스터의 워크로드를 호스팅하지 않습니다. </br> 모든 {{site.data.keyword.Bluemix_notm}} 계정은 작업자 노드에서 고품질 네트워크 성능과 격리를 보장할 수 있도록 IBM Cloud 인프라(SoftLayer) VLAN으로 설정됩니다. </br>클러스터에서 데이터의 지속성을 위해 IBM Cloud 인프라(SoftLayer)에서 전용 NFS 기반 파일 스토리지를 프로비저닝할 수 있으며 해당 플랫폼의 기본 제공 데이터 보안 기능을 최대한 활용할 수 있습니다.</dd>
  <dt>안전한 작업자 노드 설정</dt>
    <dd>모든 작업자 노드는 사용자가 변경할 수 없는 Ubuntu 운영 체제로 설정됩니다. 잠재적인 공격으로부터 작업자 노드의 운영 체제를 보호하기 위해 모든 작업자 노드가 Linux iptable 규칙이 적용되는 전문 방화벽 설정으로 구성됩니다. </br> Kubernetes에서 실행되는 모든 컨테이너는 클러스터 작성 중에 모든 작업자 노드에서 구성되는 사전 정의된 Calico 네트워크 정책 설정에 의해 보호됩니다. 이 설정은 작업자 노드와 포드 간의 보안 네트워크 통신을 보장합니다. 컨테이너가 작업자 노드에서 수행할 수 있는 조치를 제한하기 위해 사용자가 작업자 노드에서 [AppArmor 정책 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://kubernetes.io/docs/tutorials/clusters/apparmor/)을 구성하도록 선택할 수 있습니다. </br> SSH 액세스는 작업자 노드에서 사용되지 않습니다. 작업자 노드에서 추가 기능을 설치하려면 모든 작업자 노드에서 실행할 모든 항목에 대해 [Kubernetes 디먼 세트 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset)를 사용하거나 실행해야 하는 일회성 조치에 대해서는 [Kubernetes 작업 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)을 사용하십시오.</dd>
  <dt>Kubernetes 작업자 노드 보안 준수</dt>
    <dd>IBM은 내부 및 외부 보안 자문 팀과 공동 작업하여 잠재적 보안 준수 취약점을 해결합니다. IBM은 운영 체제에 업데이트 및 보안 패치를 배치하기 위해 작업자 노드에 대한 액세스를 유지합니다.
</br> <b>중요</b>: 자동으로 운영 체제에 배치되는 업데이트와 보안 패치가 설치될 수 있도록 정기적으로 작업자 노드를 다시 부팅하십시오. IBM에서는 작업자 노드를 다시 부팅하지 않습니다.
</dd>
  <dt>암호화된 디스크</dt>
  <dd>기본적으로 {{site.data.keyword.containershort_notm}}는 프로비저닝될 때 모든 작업자 노드에 두 개의 로컬 SSD 암호화 데이터 파티션을 제공합니다. 첫 번째 파티션은 암호화되지 않고 _/var/lib/docker_에 마운트되는 두 번째 파티션은 LUKS 암호화 키를 사용하여 프로비저닝될 때 잠금 해제됩니다. 각 Kubernetes 클러스터의 각 작업자는 {{site.data.keyword.containershort_notm}}에서 관리되는 고유한 LUKS 암호화 키가 있습니다. 클러스터를 작성하거나 기존 클러스터에 작업자 노드를 추가하는 경우, 키를 안전하게 가져온 다음 암호화 디스크가 잠금 해제된 후에 버려집니다. <p><b>참고</b>: 암호화는 디스크 I/O 성능에 영향을 줄 수 있습니다. 고성능 디스크 I/O가 필요한 워크로드의 경우, 암호화를 끌지 여부를 결정하는 데 도움이 되도록 암호화가 사용된 클러스터와 사용되지 않은 클러스터를 둘 다 테스트하십시오. </p>
  </dd>
  <dt>IBM Cloud 인프라(SoftLayer) 네트워크 방화벽에 대한 지원</dt>
    <dd>{{site.data.keyword.containershort_notm}}는 모든 [IBM Cloud 인프라(SoftLayer) 방화벽 오퍼링 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://www.ibm.com/cloud-computing/bluemix/network-security)과 호환 가능합니다. {{site.data.keyword.Bluemix_notm}} 퍼블릭에서 사용자는 클러스터에 대한 데디케이티드 네트워크 보안을 제공하고 네트워크 침입을 발견하여 이를 해결하기 위한 사용자 정의 네트워크 정책으로 방화벽을 설정할 수 있습니다. 예를 들어, [Vyatta ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://knowledgelayer.softlayer.com/topic/vyatta-1)가 방화벽 역할을 수행하여 원하지 않는 트래픽을 차단하게 설정하도록 선택할 수 있습니다. 방화벽을 설정할 때 마스터와 작업자 노드가 통신할 수 있도록 각 지역의 [필수 포트와 IP 주소도 개방해야 합니다](#opening_ports). </dd>
  <dt>서비스를 개인용으로 설정하거나 일부 서비스 및 앱을 선택적으로 공용 인터넷에 노출</dt>
    <dd>사용자는 작업자 노드와 포드 간에 보안 통신을 보장하기 위해 서비스와 앱을 개인용으로 유지하고 이 주제에서 설명하는 기본 제공 보안 기능을 최대로 활용하도록 선택할 수 있습니다. 서비스와 앱을 공용 인터넷에 노출하기 위해, 공용으로 안전하게 서비스를 사용할 수 있도록 Ingress 및 로드 밸런서 지원을 활용할 수 있습니다. </dd>
  <dt>작업자 노드와 앱을 온프레미스 데이터 센터에 안전하게 연결</dt>
  <dd>온프레미스 데이터 센터에 작업자 노드와 앱을 연결하려면 Strongswan 서비스, Vyatta Gateway Appliance 또는 Fortigate Appliance로 VPN IPSec 엔드포인트를 구성할 수 있습니다. <br><ul><li><b>Strongswan IPSec VPN 서비스</b>: Kubernetes 클러스터를 온프레미스 네트워크와 안전하게 연결하는 [Strongswan IPSec VPN 서비스 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://www.strongswan.org/)를 설정할 수 있습니다. Strongswan IPSec VPN 서비스는 IPsec(Industry-standard Protocol Security) 프로토콜 스위트를 기반으로 하는 인터넷을 통해 안전한 엔드-투-엔드 통신 채널을 제공합니다. 클러스터와 온프레미스 네트워크 간의 보안 연결을 설정하려면 IPsec VPN 게이트웨이 또는 IBM Cloud 인프라(SoftLayer) 서버가 온프레미스 데이터 센터에 설치되어 있어야 합니다. 그러면 Kubernetes 포드에 [Strongswan IPSec VPN 서비스를 구성 및 배치](cs_security.html#vpn)할 수 있습니다. </li><li><b>Vyatta Gateway Appliance 또는 Fortigate Appliance</b>: 클러스터의 규모가 더 큰 경우, IPSec VPN 엔드포인트를 구성하기 위해 Vyatta Gateway Appliance 또는 Fortigate Appliance를 설정하는 것이 바람직합니다. 자세한 정보는 [클러스터를 온프레미스 데이터 센터에 연결 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://www.ibm.com/blogs/bluemix/2017/07/kubernetes-and-bluemix-container-based-workloads-part4/)의 블로그 게시물을 참조하십시오. </li></ul></dd>
  <dt>클러스터 활동의 지속적 모니터링 및 로깅</dt>
    <dd>표준 클러스터의 경우, 작업자 노드 추가, 롤링 업데이트 진행상태 또는 용량 사용 정보와 같은 모든 클러스터 관련 이벤트를 {{site.data.keyword.containershort_notm}}에서 로그하고 모니터할 수 있으며, {{site.data.keyword.loganalysislong_notm}} 및 {{site.data.keyword.monitoringlong_notm}}으로 전송할 수 있습니다. 로깅 및 모니터링 설정에 대한 정보는 [클러스터 로깅 구성](https://console.bluemix.net/docs/containers/cs_cluster.html#cs_logging) 및 [클러스터 모니터링 구성](https://console.bluemix.net/docs/containers/cs_cluster.html#cs_monitoring)을 참조하십시오. </dd>
</dl>

### 이미지
{: #cs_security_deployment}

기본 제공 보안 기능으로 이미지의 보안과 무결성을 관리합니다.
{: shortdesc}

<dl>
<dt>{{site.data.keyword.registryshort_notm}}의 보안 Docker 개인용 이미지 저장소</dt>
<dd>Docker 이미지를 빌드하고 안전하게 저장하며 이를 클러스터 사용자 간에 공유할 수 있도록 IBM이 호스팅하고 관리하는 멀티 테넌트, 고가용성 및 확장 가능한 개인용 이미지 레지스트리의 사용자 고유의 Docker 이미지 저장소를 설정할 수 있습니다. </dd>

<dt>이미지 보안 준수 </dt>
<dd>{{site.data.keyword.registryshort_notm}}를 사용하는 경우에는 Vulnerability Advisor에서 제공하는 기본 제공 보안 스캐닝을 활용할 수 있습니다. 네임스페이스에 푸시되는 모든 이미지는 알려진 CentOS, Debian, Red Hat 및 Ubuntu 문제의 데이터베이스에 대한 취약점이 자동으로 스캔됩니다. 취약점이 발견되는 경우, Vulnerability Advisor는 이미지 무결성과 보안을 보장하기 위해 이를 해결하는 방법에 대한 지시사항을 제공합니다. </dd>
</dl>

이미지의 취약성 평가를 보려면 [Vulnerability Advisor 문서를 검토](/docs/services/va/va_index.html#va_registry_cli)하십시오.

<br />


## 방화벽에서 필수 포트와 IP 주소 열기
{: #opening_ports}

방화벽에서 특정 포트 및 IP 주소을 열어야 할 수 있는 다음 상황을 검토하십시오.
* 회사 네트워크 정책으로 인해 프록시 또는 방화벽을 통해 공용 인터넷 엔드포인트에 액세스하지 못할 때 로컬 시스템에서 [`bx` 명령을 실행](#firewall_bx)하려는 경우
* 회사 네트워크 정책으로 인해 프록시 또는 방화벽을 통해 공용 인터넷 엔드포인트에 액세스하지 못할 때 로컬 시스템에서 [`kubectl` 명령을 실행](#firewall_kubectl)하려는 경우
* 회사 네트워크 정책으로 인해 프록시 또는 방화벽을 통해 공용 인터넷 엔드포인트에 액세스하지 못할 때 로컬 시스템에서 [`calicoctl` 명령을 실행](#firewall_calicoctl)하려는 경우
* 작업자 노드에 대한 방화벽이 설정되었거나 IBM Cloud 인프라(SoftLayer) 계정에서 방화벽 설정이 사용자 정의되었을 때 [Kubernetes 마스터와 작업자 노드 간의 통신을 허용하려는 경우](#firewall_outbound)
* [클러스터의 외부에서 NodePort 서비스, LoadBalancer 서비스 또는 Ingress에 액세스하려는 경우](#firewall_inbound)

### 방화벽 뒤에서 `bx cs` 명령 실행
{: #firewall_bx}

회사 네트워크 정책이 로컬 시스템에서 프록시 또는 방화벽을 통해 공용 엔드포인트에 액세스하지 못하도록 방지하는 경우 `bx cs` 명령을 실행하려면 {{site.data.keyword.containerlong_notm}}에 대한 TCP 액세스를 허용해야 합니다. 

1. 포트 443에서 `containers.bluemix.net`에 대한 액세스를 허용하십시오. 
2. 연결을 확인하십시오. 액세스가 올바르게 구성되면 출력에 배가 표시됩니다. 
   ```
   curl https://containers.bluemix.net/v1/
   ```
   {: pre}

   출력 예: 
   ```
                                     )___(
                              _______/__/_
                     ___     /===========|   ___
    ____       __   [\\\]___/____________|__[///]   __
    \   \_____[\\]__/___________________________\__[//]___
     \                                                    |
      \                                                  /
   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   ```
   {: screen}

### 방화벽 뒤에서 `kubectl` 명령 실행
{: #firewall_kubectl}

회사 네트워크 정책이 로컬 시스템에서 프록시 또는 방화벽을 통해 공용 엔드포인트에 액세스하지 못하도록 방지하는 경우 `kubectl` 명령을 실행하려면 클러스터에 대한 TCP 액세스를 허용해야 합니다. 

클러스터가 작성될 때 마스터 URL의 포트는 20000 - 32767 내에서 무작위로 지정됩니다. 작성될 수 있는 모든 클러스터에 포트 범위 20000 - 32767을 공개할 수도 있고 기존의 특정 클러스터에만 액세스를 허용할 수도 있습니다. 

시작하기 전에 [`bx cs` 명령 실행](#firewall_bx)에 대한 액세스를 허용하십시오. 

특정 클러스터에 대한 액세스를 허용하려면 다음을 수행하십시오. 

1. {{site.data.keyword.Bluemix_notm}} CLI에 로그인하십시오. 프롬프트가 표시되면
{{site.data.keyword.Bluemix_notm}} 신임 정보를 입력하십시오. 연합 계정이 있는 경우, `--sso` 옵션을 포함하십시오. 

    ```
    bx login [--sso]
    ```
    {: pre}

2. 클러스터가 있는 지역을 선택하십시오. 

   ```
   bx cs region-set
   ```
   {: pre}

3. 클러스터의 이름을 가져오십시오. 

   ```
         bx cs clusters
        ```
   {: pre}

4. 클러스터의 **마스터 URL**을 검색하십시오. 

   ```
   bx cs cluster-get <cluster_name_or_id>
   ```
   {: pre}

   출력 예: 
   ```
   ...
   Master URL:		https://169.46.7.238:31142
   ...
   ```
   {: screen}

5. 포트(예: 이전 예제의 `31142` 포트)에서 **마스터 URL**에 대한 액세스를 허용하십시오. 

6. 연결을 확인하십시오. 

   ```
   curl --insecure <master_URL>/version
   ```
   {: pre}

   명령 예: 
   ```
   curl --insecure https://169.46.7.238:31142/version
   ```
   {: pre}

   출력 예: 
   ```
   {
     "major": "1",
     "minor": "7+",
     "gitVersion": "v1.7.4-2+eb9172c211dc41",
     "gitCommit": "eb9172c211dc4108341c0fd5340ee5200f0ec534",
     "gitTreeState": "clean",
     "buildDate": "2017-11-16T08:13:08Z",
     "goVersion": "go1.8.3",
     "compiler": "gc",
     "platform": "linux/amd64"
   }
   ```
   {: screen}

7. 선택사항: 노출해야 하는 각 클러스터에 대해 이러한 단계를 반복하십시오. 

### 방화벽 뒤에서 `calicoctl` 명령 실행
{: #firewall_calicoctl}

회사 네트워크 정책이 로컬 시스템에서 프록시 또는 방화벽을 통해 공용 엔드포인트에 액세스하지 못하도록 방지하는 경우 `calicoctl` 명령을 실행하려면 Calico 명령에 대한 TCP 액세스를 허용해야 합니다. 

시작하기 전에 [`bx` commands](#firewall_bx) 및 [`kubectl` 명령](#firewall_kubectl)을 실행할 수 있도록 액세스를 허용하십시오. 

1. [`kubectl` 명령](#firewall_kubectl)을 허용하는 데 사용한 마스터 URL에서 IP 주소를 검색하십시오. 

2. ETCD의 포트를 가져오십시오. 

  ```
  kubectl get cm -n kube-system calico-config -o yaml | grep etcd_endpoints
  ```
  {: pre}

3. 마스터 URL IP 주소와 ETCD 포트를 통해 Calico 정책에 대한 액세스를 허용하십시오. 

### 클러스터가 인프라 리소스 및 기타 서비스에 액세스하도록 허용
{: #firewall_outbound}

  1.  클러스터에서 모든 작업자 노드에 대한 공인 IP 주소를 기록해 놓으십시오. 

      ```
    bx cs workers <cluster_name_or_id>
    ```
      {: pre}

  2.  소스 _<each_worker_node_publicIP>_에서 대상 TCP/UDP 포트 범위 20000-32767 및 포트 443으로의 발신 네트워크 트래픽과 다음 IP 주소 및 네트워크 그룹을 허용하십시오. 로컬 시스템이 공용 인터넷 엔드포인트에 액세스하지 못하도록 방지하는 회사 방화벽이 있는 경우, 소스 작업자 노드와 로컬 시스템 둘 다에 대해 이 단계를 수행하십시오. 
      - **중요**: 부트스트랩 프로세스 중에 로드를 밸런싱하려면 지역 내 모든 위치에 대해 포트 443으로의 발신 트래픽을 허용해야 합니다. 예를 들어, 클러스터가 미국 남부에 있는 경우 포트 443에서 모든 위치(dal10, dal12 및 dal13)의 IP 주소로의 트래픽을 허용해야 합니다.
      <p>
  <table summary="테이블의 첫 번째 행에는 두 개의 열이 있습니다. 나머지 행은 왼쪽에서 오른쪽으로 읽어야 하며 1열에는 서버 위치, 2열에는 일치시킬 IP 주소가 있습니다.">
      <thead>
      <th>지역</th>
      <th>위치</th>
      <th>IP 주소</th>
      </thead>
    <tbody>
      <tr>
        <td>AP 북부</td>
        <td>hkg02<br>tok02</td>
        <td><code>169.56.132.234</code><br><code>161.202.126.210</code></td>
       </tr>
      <tr>
         <td>AP 남부</td>
         <td>mel01<br>syd01<br>syd04</td>
         <td><code>168.1.97.67</code><br><code>168.1.8.195</code><br><code>130.198.64.19, 130.198.66.34</code></td>
      </tr>
      <tr>
         <td>중앙 유럽</td>
         <td>ams03<br>fra02<br>mil01<br>par01</td>
         <td><code>169.50.169.106, 169.50.154.194</code><br><code>169.50.56.170, 169.50.56.174</code><br><code>159.122.190.98</code><br><code>159.8.86.149, 159.8.98.170</code></td>
        </tr>
      <tr>
        <td>영국 남부</td>
        <td>lon02<br>lon04</td>
        <td><code>159.122.242.78</code><br><code>158.175.65.170, 158.175.74.170, 158.175.76.2</code></td>
      </tr>
      <tr>
        <td>미국 동부</td>
         <td>tor01<br>wdc06<br>wdc07</td>
         <td><code>169.53.167.50</code><br><code>169.60.73.142</code><br><code>169.61.83.62</code></td>
      </tr>
      <tr>
        <td>미국 남부</td>
        <td>dal10<br>dal12<br>dal13</td>
        <td><code>169.47.234.18, 169.46.7.234</code><br><code>169.47.70.10</code><br><code>169.60.128.2</code></td>
      </tr>
      </tbody>
    </table>
</p>

  3.  작업자 노드에서 {{site.data.keyword.registrylong_notm}}로의 발신 네트워크 트래픽을 허용하십시오.
      - `TCP port 443 FROM <each_worker_node_publicIP> TO <registry_publicIP>`
      - 트래픽을 허용하려는 레지스트리 지역에 대한 모든 주소로 <em>&lt;registry_publicIP&gt;</em>를 대체하십시오. <p>
<table summary="테이블의 첫 번째 행에는 두 개의 열이 있습니다. 나머지 행은 왼쪽에서 오른쪽으로 읽어야 하며 1열에는 서버 위치, 2열에는 일치시킬 IP 주소가 있습니다.">
      <thead>
        <th>컨테이너 지역</th>
        <th>레지스트리 주소</th>
        <th>레지스트리 IP 주소</th>
      </thead>
      <tbody>
        <tr>
          <td>AP 북부, AP 남부</td>
          <td>registry.au-syd.bluemix.net</td>
          <td><code>168.1.45.160/27</code></br><code>168.1.139.32/27</code></td>
        </tr>
        <tr>
          <td>중앙 유럽</td>
          <td>registry.eu-de.bluemix.net</td>
          <td><code>169.50.56.144/28</code></br><code>159.8.73.80/28</code></td>
         </tr>
         <tr>
          <td>영국 남부</td>
          <td>registry.eu-gb.bluemix.net</td>
          <td><code>159.8.188.160/27</code></br><code>169.50.153.64/27</code></td>
         </tr>
         <tr>
          <td>미국 동부, 미국 남부</td>
          <td>registry.ng.bluemix.net</td>
          <td><code>169.55.39.112/28</code></br><code>169.46.9.0/27</code></br><code>169.55.211.0/27</code></td>
         </tr>
        </tbody>
      </table>
</p>

  4.  선택사항: 작업자 노드에서 {{site.data.keyword.monitoringlong_notm}} 및 {{site.data.keyword.loganalysislong_notm}} 서비스로의 발신 네트워크 트래픽을 허용하십시오.
      - `TCP port 443, port 9095 FROM <each_worker_node_publicIP> TO <monitoring_publicIP>`
      - 트래픽을 허용하려는 모니터링 지역에 대한 모든 주소로 <em>&lt;monitoring_publicIP&gt;</em>를 대체하십시오.
      <p><table summary="테이블의 첫 번째 행에는 두 개의 열이 있습니다. 나머지 행은 왼쪽에서 오른쪽으로 읽어야 하며 1열에는 서버 위치, 2열에는 일치시킬 IP 주소가 있습니다.">
      <thead>
        <th>컨테이너 지역</th>
        <th>모니터링 주소</th>
        <th>모니터링 IP 주소</th>
        </thead>
      <tbody>
        <tr>
         <td>중앙 유럽</td>
         <td>metrics.eu-de.bluemix.net</td>
         <td><code>159.122.78.136/29</code></td>
        </tr>
        <tr>
         <td>영국 남부</td>
         <td>metrics.eu-gb.bluemix.net</td>
         <td><code>169.50.196.136/29</code></td>
        </tr>
        <tr>
          <td>미국 동부, 미국 남부, AP 북부</td>
          <td>metrics.ng.bluemix.net</td>
          <td><code>169.47.204.128/29</code></td>
         </tr>
         
        </tbody>
      </table>
</p>
      - `TCP port 443, port 9091 FROM <each_worker_node_publicIP> TO <logging_publicIP>`
      - 트래픽을 허용하려는 로깅 지역에 대한 모든 주소로 <em>&lt;logging_publicIP&gt;</em>를 대체하십시오.
      <p><table summary="테이블의 첫 번째 행에는 두 개의 열이 있습니다. 나머지 행은 왼쪽에서 오른쪽으로 읽어야 하며 1열에는 서버 위치, 2열에는 일치시킬 IP 주소가 있습니다.">
      <thead>
        <th>컨테이너 지역</th>
        <th>로깅 주소</th>
        <th>로깅 IP 주소</th>
        </thead>
        <tbody>
          <tr>
            <td>미국 동부, 미국 남부</td>
            <td>ingest.logging.ng.bluemix.net</td>
            <td><code>169.48.79.236</code><br><code>169.46.186.113</code></td>
           </tr>
          <tr>
           <td>EU 중심부, UK 남부</td>
           <td>ingest-eu-fra.logging.bluemix.net</td>
           <td><code>158.177.88.43</code><br><code>159.122.87.107</code></td>
          </tr>
          <tr>
           <td>AP 남부, AP 북부</td>
           <td>ingest-au-syd.logging.bluemix.net</td>
           <td><code>130.198.76.125</code><br><code>168.1.209.20</code></td>
          </tr>
         </tbody>
       </table>
</p>

  5. 사설 방화벽의 경우 적절한 IBM Cloud 인프라(SoftLayer) 사설 IP 범위를 허용하십시오. [이 링크](https://knowledgelayer.softlayer.com/faq/what-ip-ranges-do-i-allow-through-firewall)의 **백엔드(사설) 네트워크** 섹션부터 참조하십시오.
      - 사용 중인 [지역 내의 위치](cs_regions.html#locations)를 모두 추가하십시오. 
      - dal01 위치(데이터 센터)를 추가해야 합니다. 
      - 클러스터 부트스트랩 프로세스를 허용하려면 포트 80 및 443을 여십시오. 

  6. 데이터 스토리지에 대한 지속적 볼륨 클레임을 작성하려면 클러스터가 있는 위치(데이터 센터)의 [IBM Cloud 인프라(SoftLayer) IP 주소](https://knowledgelayer.softlayer.com/faq/what-ip-ranges-do-i-allow-through-firewall)에 대해 방화벽을 통한 egress 액세스를 허용하십시오. 
      - 클러스터의 위치(데이터 센터)를 찾으려면 `bx cs clusters`를 실행하십시오. 
      - **프론트 엔드(공용) 네트워크**와 **백엔드(사설) 네트워크** 둘 다에 대해 IP 범위에 대한 액세스를 허용하십시오. 
      - **백엔드(사설) 네트워크**를 위해 dal01 위치(데이터 센터)를 추가해야 합니다. 

### 클러스터 외부에서 NodePort, 로드 밸런서 및 Ingress 서비스에 액세스
{: #firewall_inbound}

NodePort, 로드 밸런서 및 Ingress 서비스에 대한 수신 액세스를 허용할 수 있습니다. 

<dl>
  <dt>NodePort 서비스</dt>
  <dd>모든 작업자 노드가 트래픽을 허용할 수 있도록 서비스를 공인 IP 주소에 배치할 때 구성한 포트를 여십시오. 포트를 찾으려면 `kubectl get svc`를 실행하십시오. 포트는 20000 - 32000 범위에 있습니다. <dd>
  <dt>LoadBalancer 서비스</dt>
  <dd>서비스를 로드 밸런스 서비스의 공인 IP 주소에 배치할 때 구성한 포트를 여십시오. </dd>
  <dt>Ingress</dt>
  <dd>Ingress 애플리케이션 로드 밸런서의 IP 주소에 대해 HTTP의 경우 포트 80을 열고 HTTPS의 경우 포트 443을 여십시오. </dd>
</dl>

<br />


## Strongswan IPSec VPN 서비스 Helm 차트로 VPN 연결 설정
{: #vpn}

VPN 연결을 통해 Kubernetes 클러스터의 앱을 온프레미스 네트워크에 안전하게 연결할 수 있습니다. 클러스터 내에서 실행 중인 앱에 클러스터 외부의 앱을 연결할 수도 있습니다. VPN 연결을 설정하려면 Helm 차트를 사용하여 Kubernetes 포드 내에 [Strongswan IPSec VPN 서비스 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://www.strongswan.org/)를 구성하고 배치할 수 있습니다. 그러면 모든 VPN 트래픽은 이 포드를 통해 라우팅됩니다. Strongswan 차트를 설정하는 데 사용되는 Helm 명령에 대한 자세한 정보는 [Helm 문서 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://docs.helm.sh/helm/)를 참조하십시오. 

시작하기 전에:

- [표준 클러스터를 작성하십시오. ](cs_cluster.html#cs_cluster_cli)
- [기존 클러스터를 사용 중인 경우 버전 1.7.4 이상으로 업데이트하십시오. ](cs_cluster.html#cs_cluster_update)
- 클러스터에는 하나 이상의 사용 가능한 공인 로드 밸런서 IP 주소가 있어야 합니다. 
- [Kubernetes CLI를 클러스터에 대상으로 지정하십시오](cs_cli_install.html#cs_cli_configure). 

Strongswan으로 VPN 연결을 설정하려면 다음을 수행하십시오. 

1. 아직 사용되지 않은 경우, 클러스터를 위해 Helm을 설치하고 초기화하십시오. 

    1. [Helm CLI를 설치 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://docs.helm.sh/using_helm/#installing-helm)하십시오. 

    2. Helm을 초기화하고 `tiller`를 설치하십시오. 

        ```
        helm init
        ```
        {: pre}

    3. `tiller-deploy` 포드가 클러스터에서 `Running` 상태인지 확인하십시오. 

        ```
        kubectl get pods -n kube-system -l app=helm
        ```
        {: pre}

        출력 예: 

        ```
        NAME                            READY     STATUS    RESTARTS   AGE
        tiller-deploy-352283156-nzbcm   1/1       Running   0          10m
        ```
        {: screen}

    4. {{site.data.keyword.containershort_notm}} Helm 저장소를 Helm 인스턴스에 추가하십시오. 

        ```
        helm repo add bluemix  https://registry.bluemix.net/helm
        ```
        {: pre}

    5. Strongswan 차트가 Helm 저장소에 나열되는지 확인하십시오. 

        ```
        helm search bluemix
        ```
        {: pre}

2. Strongswan Helm 차트를 위한 기본 구성 설정을 로컬 YAML 파일에 저장하십시오. 

    ```
    helm inspect values bluemix/strongswan > config.yaml
    ```
    {: pre}

3. `config.yaml` 파일을 열고 원하는 VPN 구성에 따라 기본값을 다음과 같이 변경하십시오. 특성이 값에 대해 옵션을 설정한 경우, 그들은 파일에서 각 특성 위의 주석에 나열됩니다. **중요**: 특성을 변경할 필요가 없는 경우 해당 특성 앞에 `#`를 놓아서 주석 처리하십시오. 

    <table>
    <caption>표 2. YAML 파일 컴포넌트 이해</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="아이디어 아이콘"/> YAML 파일 컴포넌트 이해</th>
    </thead>
    <tbody>
    <tr>
    <td><code>overRideIpsecConf</code></td>
    <td>사용할 기존 <code>ipsec.conf</code> 파일이 있는 경우, 중괄호(<code>{}</code>)를 제거하고 이 특성 뒤에 파일의 컨텐츠를 추가하십시오. 파일 컨텐츠는 들여써야 합니다. **참고:** 고유의 파일을 사용하는 경우, <code>ipsec</code>, <code>local</code> 및 <code>remote</code> 섹션의 값은 사용되지 않습니다. </td>
    </tr>
    <tr>
    <td><code>overRideIpsecSecrets</code></td>
    <td>사용할 기존 <code>ipsec.secrets</code> 파일이 있는 경우, 중괄호(<code>{}</code>)를 제거하고 이 특성 뒤에 파일의 컨텐츠를 추가하십시오. 파일 컨텐츠는 들여써야 합니다. **참고:** 고유의 파일을 사용하는 경우, <code>preshared</code> 섹션의 값은 사용되지 않습니다. </td>
    </tr>
    <tr>
    <td><code>ipsec.keyexchange</code></td>
    <td>온프레미스 VPN 터널 엔드포인트가 연결 초기화를 위한 프로토콜로 <code>ikev2</code>를 지원하지 않는 경우, 이 값을 <code>ikev1</code>로 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>ipsec.esp</code></td>
    <td>온프레미스 VPN 터널 엔드포인트가 연결에 사용하는 ESP 암호화/인증 알고리즘의 목록으로 이 값을 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>ipsec.ike</code></td>
    <td>온프레미스 VPN 터널 엔드포인트가 연결에 사용하는 IKE/ISAKMP SA 암호화/인증 알고리즘의 목록으로 이 값을 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>ipsec.auto</code></td>
    <td>클러스터가 VPN 연결을 시작하도록 하려면 이 값을 <code>start</code>로 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>local.subnet</code></td>
    <td>VPN 연결을 통해 온프레미스 네트워크에 노출되어야 하는 클러스터 서브넷 CIDR의 목록으로 이 값을 변경하십시오. 이 목록은 다음 서브넷을 포함할 수 있습니다. <ul><li>Kubernetes 포드 서브넷 CIDR: <code>172.30.0.0/16</code></li><li>Kubernetes 서비스 서브넷 CIDR: <code>172.21.0.0/16</code></li><li>사설 네트워크에서 NodePort 서비스가 노출한 애플리케이션이 있는 경우, 작업자 노드의 사설 서브넷 CIDR입니다. 이 값을 찾으려면 <code>bx cs subnets | grep <xxx.yyy.zzz></code>를 실행하십시오. 여기서 &lt;xxx.yyy.zzz&gt;는 작업자 노드 사설 IP 주소의 처음 세 옥텟입니다. </li><li>사설 네트워크에서 LoadBalancer 서비스가 노출한 애플리케이션이 있는 경우, 클러스터의 사설 또는 사용자 관리 서브넷 CIDR입니다. 이러한 값을 찾으려면 <code>bx cs cluster-get <cluster name> --showResources</code>를 실행하십시오. <b>VLANS</b> 섹션에서 <b>Public</b> 값이 <code>false</code>인 CIDR을 찾으십시오. </li></ul></td>
    </tr>
    <tr>
    <td><code>local.id</code></td>
    <td>VPN 터널 엔드포인트가 연결에 사용하는 로컬 Kubernetes 클러스터 측 문자열 ID로 이 값을 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>remote.gateway</code></td>
    <td>온프레미스 VPN 게이트웨이의 공인 IP 주소로 이 값을 변경하십시오. </td>
    </tr>
    <td><code>remote.subnet</code></td>
    <td>Kubernetes 클러스터가 액세스하도록 허용된 온프레미스 사설 서브넷 CIDR의 목록으로 이 값을 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>remote.id</code></td>
    <td>VPN 터널 엔드포인트가 연결에 사용하는 원격 온프레미스 측 문자열 ID로 이 값을 변경하십시오. </td>
    </tr>
    <tr>
    <td><code>preshared.secret</code></td>
    <td>온프레미스 VPN 터널 엔드포인트 게이트웨이가 연결에 사용하는 사전 공유된 시크릿으로 이 값을 변경하십시오. </td>
    </tr>
    </tbody></table>

4. 업데이트된 `config.yaml` 파일을 저장하십시오. 

5. 업데이트된 `config.yaml` 파일로 Helm 차트를 클러스터에 설치하십시오. 업데이트된 특성은 차트의 구성 맵에 저장됩니다. 

    ```
    helm install -f config.yaml --namespace=kube-system --name=vpn bluemix/strongswan
    ```
    {: pre}

6. 차트 배치 상태를 확인하십시오. 차트가 준비 상태인 경우, 출력의 맨 위 근처에 있는 **STATUS** 필드의 값은 `DEPLOYED`입니다. 

    ```
    helm status vpn
    ```
    {: pre}

7. 차트가 배치된 후 `config.yaml` 파일에서 업데이트된 설정이 사용되었는지 확인하십시오. 

    ```
    helm get values vpn
    ```
    {: pre}

8. 새 VPN 연결을 테스트하십시오. 
    1. 온프레미스 게이트웨이에서 VPN이 활성이 아닌 경우 VPN을 시작하십시오. 

    2. `STRONGSWAN_POD` 환경 변수를 설정하십시오. 

        ```
        export STRONGSWAN_POD=$(kubectl get pod -n kube-system -l app=strongswan,release=vpn -o jsonpath='{ .items[0].metadata.name }')
        ```
        {: pre}

    3. VPN의 상태를 확인하십시오. `ESTABLISHED` 상태는 VPN 연결이 성공임을 의미합니다. 

        ```
        kubectl exec -n kube-system  $STRONGSWAN_POD -- ipsec status
        ```
        {: pre}

        출력 예: 
        ```
        Security Associations (1 up, 0 connecting):
            k8s-conn[1]: ESTABLISHED 17 minutes ago, 172.30.244.42[ibm-cloud]...192.168.253.253[on-prem]
            k8s-conn{2}:  INSTALLED, TUNNEL, reqid 12, ESP in UDP SPIs: c78cb6b1_i c5d0d1c3_o
            k8s-conn{2}:   172.21.0.0/16 172.30.0.0/16 === 10.91.152.128/26
        ```
        {: screen}

        **참고**:
          - 이 Helm 차트를 처음 사용하는 경우 VPN의 상태는 `ESTABLISHED`가 아닐 가능성이 높습니다. 연결이 성공하기 전에 온프레미스 VPN 엔드포인트 설정을 확인하고 3단계로 돌아가서 `config.yaml` 파일을 여러 번 변경해야 할 수도 있습니다. 
          - VPN 포드가 `ERROR` 상태이거나 계속 충돌하고 다시 시작되는 경우, 차트 구성 맵에 있는 `ipsec.conf` 설정의 매개변수 유효성 검증 때문일 수 있습니다. 이 경우에 해당하는지 확인하려면 `kubectl logs -n kube-system $STRONGSWAN_POD`를 실행하여 Strongswan 포드 로그에서 유효성 검증 오류가 있는지 확인하십시오. 유효성 검증 오류가 있는 경우, `helm delete --purge vpn`을 실행하고 3단계로 돌아가서 `config.yaml` 파일에서 잘못된 값을 수정한 후 4 - 8단계를 반복하십시오. 클러스터에 많은 수의 작업자 로드가 있는 경우에는 `helm delete` 및 `helm install`을 실행하는 대신 `helm upgrade`를 사용하여 변경사항을 더 빠르게 적용할 수도 있습니다. 

    4. VPN이 `ESTABLISHED` 상태가 된 후 `ping`으로 연결을 테스트하십시오. 다음 예제는 Kubernetes 클러스터의 VPN 포드에서 온프레미스 VPN 게이트웨이의 사설 IP 주소로 Ping을 전송합니다. 구성 파일에 올바른 `remote.subnet` 및 `local.subnet`이 지정되었는지 확인하고 로컬 서브넷 목록에 Ping을 전송 중인 소스 IP 주소가 포함되었는지 확인하십시오. 

        ```
        kubectl exec -n kube-system  $STRONGSWAN_POD -- ping -c 3  <on-prem_gateway_private_IP>
        ```
        {: pre}

Strongswan IPSec VPN 서비스를 사용하지 않으려면 다음을 수행하십시오. 

1. Helm 차트를 삭제하십시오. 

    ```
    helm delete --purge vpn
    ```
    {: pre}

<br />


## 네트워크 정책
{: #cs_security_network_policies}

모든 Kubernetes 클러스터는 Calico라는 네트워크 플러그인으로 설정됩니다. 모든 작업자 노드의 공용 네트워크 인터페이스 보안을 위해 기본 네트워크 정책이 설정됩니다. 고유 보안 요구사항이 있을 때 Calico 및 기본 Kubernetes 기능을 사용하여 클러스터에 대한 네트워크 정책을 더 구성할 수 있습니다. 이러한 네트워크 정책은 클러스터에서 포드에 허용하거나 차단하고자 하는 네트워크 트래픽을 지정합니다.
{: shortdesc}

클러스터에 대한 네트워크 정책을 작성하기 위해 Calico 및 기본 Kubernetes 기능 중에서 선택할 수 있습니다. Kubernetes 네트워크 정책을 사용하여 시작할 수 있지만, 더 강력한 기능을 위해서는 Calico 네트워크 정책을 사용하십시오. 

<ul>
  <li>[Kubernetes 네트워크 정책 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://kubernetes.io/docs/concepts/services-networking/network-policies/): 포드가 서로 통신할 수 있는지 지정하는 등의 일부 기본 옵션이 제공됩니다. 프로토콜 및 포트에 대한 수신 네트워크 트래픽을 허용하거나 차단할 수 있습니다. 다른 포드에 연결하려고 시도하는 포드의 Kubernetes 네임스페이스 및 레이블을 기반으로 이 트래픽을 필터링할 수 있습니다.</br>이러한 정책은
`kubectl` 명령 또는 Kubernetes API를 사용하여 적용될 수 있습니다. 해당 정책이 적용되면
Calico 네트워크 정책으로 변환되고 Calico는 이러한 정책을 시행합니다. </li>
  <li>[Calico 네트워크 정책 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](http://docs.projectcalico.org/v2.4/getting-started/kubernetes/tutorials/advanced-policy): 이러한 정책은 Kubernetes 네트워크 정책의 수퍼세트이며 다음 기능으로 기본 Kubernetes 기능을 향상시킵니다.
</li>
    <ul><ul><li>Kubernetes 포드 트래픽뿐만 아니라 특정 네트워크 인터페이스에서 네트워크 트래픽을 허용하거나 차단합니다. </li>
    <li>수신(ingress) 및 발신(egress) 네트워크 트래픽을 허용하거나 차단합니다. </li>
    <li>[LoadBalancer 또는 NodePort Kubernetes 서비스에 대한 수신(ingress) 트래픽을 차단](#cs_block_ingress)합니다.</li>
    <li>소스 또는 대상 IP 주소 또는 CIDR을 기반으로 하는 트래픽을 허용하거나 차단합니다. </li></ul></ul></br>

이러한 정책은 `calicoctl` 명령을 사용하여 적용됩니다. Calico는
Kubernetes 작업자 노드에서 Linux iptables 규칙을 설정하여 Calico 정책으로 변환된 Kubernetes 네트워크 정책을 포함하는 해당 정책을
시행합니다. Iptables 규칙은 대상으로 지정된 리소스에 전달되도록 네트워크 트래픽이 충족해야 하는 특성을 정의하기 위해
작업자 노드에 대한 방화벽의 역할을 합니다. </ul>


### 기본 정책 구성
{: #concept_nq1_2rn_4z}

클러스터가 작성되면 공용 인터넷에서 작업자 노드에 대한 수신 트래픽을 제한하기 위해 기본 네트워크 정책이 각 작업자 노드의 공용 네트워크 인터페이스에
대해 자동으로 설정됩니다. 이러한 정책은 포드에서 포드 트래픽에 영향을 미치지 않으며
Kubernetes 노드 포트, 로드 밸런서 및 Ingress 서비스에 대한 액세스를 허용하도록 설정됩니다. 

기본 정책은 포드에 직접 적용되지 않습니다. Calico 호스트 엔드포인트를 사용하여 작업자 노드의 공용 네트워크 인터페이스에 적용됩니다. 호스트 엔드포인트가 Calico에서 작성되면
그 작업자 노드의 네트워크 인터페이스에서 나가고 들어오는 모든 트래픽은 그 트래픽이 정책에서 허용되는 경우를 제외하고 차단됩니다. 

**중요:** 정책을 완전히 이해하고 정책에서 허용하고 있는 트래픽이 필요하지 않다는 것을 아는 경우를 제외하고는 호스트 엔드포인트에 적용된 정책을 제거하지 마십시오. 


 <table summary="테이블의 첫 번째 행에는 두 개의 열이 있습니다. 나머지 행은 왼쪽에서 오른쪽으로 읽어야 하며 1열에는 서버 위치, 2열에는 일치시킬 IP 주소가 있습니다.">
      <caption>표 3. 각 클러스터의 기본 정책</caption>
  <thead>
  <th colspan=2><img src="images/idea.png" alt="아이디어 아이콘"/> 각 클러스터의 기본 정책</th>
  </thead>
  <tbody>
    <tr>
      <td><code>allow-all-outbound</code></td>
      <td>아웃바운드 트래픽을 모두 허용합니다. </td>
    </tr>
    <tr>
      <td><code>allow-bixfix-port</code></td>
      <td>bigfix 앱에 대한 포트 52311의 수신 트래픽이 가능하게 하여 필수 작업자 노드 업데이트를 허용합니다.</td>
    </tr>
    <tr>
      <td><code>allow-icmp</code></td>
      <td>수신 icmp 패킷(ping)을 허용합니다. </td>
     </tr>
    <tr>
      <td><code>allow-node-port-dnat</code></td>
      <td>해당 서비스가 노출된 포드에 수신 노드 포트, 로드 밸런서 및 Ingress 서비스 트래픽을 허용합니다. Kubernetes는 해당 서비스 요청을 올바른 포드에 전달하기 위해 대상 네트워크 주소 변환(DNAT)을
사용하기 때문에 해당 서비스가 공용 인터페이스에서 노출하는 포트를 지정할 필요가 없다는 점을 참고하십시오. 호스트 엔드포인트 정책이 iptables에 적용되기 전에 해당 전달이 이루어집니다. </td>
   </tr>
   <tr>
      <td><code>allow-sys-mgmt</code></td>
      <td>작업자 노드를 관리하는 데 사용되는 특정 IBM Cloud 인프라(SoftLayer) 시스템에 대한 수신 연결을 허용합니다.</td>
   </tr>
   <tr>
    <td><code>allow-vrrp</code></td>
    <td>작업자 노드 사이에서 가상 IP 주소를 모니터링하고 이동하는 데 사용되는 vrrp 패킷을 허용합니다. </td>
   </tr>
  </tbody>
</table>


### 네트워크 정책 추가
{: #adding_network_policies}

대부분의 경우, 기본 정책은 변경할 필요가 없습니다. 고급 시나리오에만 변경사항이 필요할 수 있습니다. 변경사항을 작성해야 하는 경우, Calico CLI를 설치하고 사용자 고유의 네트워크 정책을 작성하십시오.

시작하기 전에:

1.  [{{site.data.keyword.containershort_notm}} 및 Kubernetes CLI를 설치하십시오.](cs_cli_install.html#cs_cli_install)
2.  [라이트 또는 표준 클러스터를 작성하십시오.](cs_cluster.html#cs_cluster_ui)
3.  [Kubernetes CLI를 클러스터에 대상으로 지정하십시오](cs_cli_install.html#cs_cli_configure). 
`--admin` 옵션을 `bx cs cluster-config` 명령에 포함하십시오. 이는 인증서 및 권한 파일을 다운로드하는 데 사용됩니다.
이 다운로드에는 Calico 명령을 실행하는 데 필요한 수퍼유저 역할에 대한 키도 포함됩니다. 

  ```
  bx cs cluster-config <cluster_name> --admin
  ```
  {: pre}

  **참고:** Calico CLI 버전 1.6.1이 지원됩니다.

네트워크 정책을 추가하려면 다음을 수행하십시오. 
1.  Calico CLI를 설치하십시오. 
    1.  [Calico CLI 다운로드 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://github.com/projectcalico/calicoctl/releases/tag/v1.6.1).

        **팁:** Windows를 사용하는 경우, {{site.data.keyword.Bluemix_notm}} CLI와 동일한 디렉토리에 Calico CLI를 설치하십시오. 이 설정을 사용하면 나중에 명령을 실행할 때 일부 파일 경로 변경이 필요하지 않습니다. 

    2.  OSX 및 Linux 사용자의 경우, 다음 단계를 완료하십시오. 
        1.  실행 파일을 /usr/local/bin 디렉토리로 이동하십시오.
            -   Linux:

              ```
               mv /<path_to_file>/calicoctl /usr/local/bin/calicoctl
              ```
              {: pre}

            -   OS X:

              ```
              mv /<path_to_file>/calicoctl-darwin-amd64 /usr/local/bin/calicoctl
              ```
              {: pre}

        2.  파일을 실행 가능하도록 설정하십시오.

            ```
           chmod +x /usr/local/bin/calicoctl
            ```
            {: pre}

    3.  Calico CLI 클라이언트 버전을 확인하여 `calico` 명령이 올바르게 실행되는지 확인하십시오. 

        ```
        calicoctl version
        ```
        {: pre}

2.  Calico CLI를 구성하십시오. 

    1.  Linux 및 OS X의 경우에는 `/etc/calico` 디렉토리를 작성하십시오. Windows의 경우에는 임의의 디렉토리가 사용될 수 있습니다. 

      ```
      sudo mkdir -p /etc/calico/
      ```
      {: pre}

    2.  `calicoctl.cfg` 파일을 작성하십시오. 
        -   Linux 및 OS X:

          ```
          sudo vi /etc/calico/calicoctl.cfg
          ```
          {: pre}

        -   Windows: 텍스트 편집기로 파일을 작성하십시오.

    3.  <code>calicoctl.cfg</code> 파일에 다음 정보를 입력하십시오. 

        ```
      apiVersion: v1
      kind: calicoApiConfig
      metadata:
      spec:
          etcdEndpoints: <ETCD_URL>
          etcdKeyFile: <CERTS_DIR>/admin-key.pem
          etcdCertFile: <CERTS_DIR>/admin.pem
          etcdCACertFile: <CERTS_DIR>/<ca-*pem_file>
      ```
        {: codeblock}

        1.  `<ETCD_URL>`을 검색하십시오. 이 명령이 실패하고 `calico-config not found` 오류가 발생하는 경우 이 [문제점 해결 주제](cs_troubleshoot.html#cs_calico_fails)를 참조하십시오.

          -   Linux 및 OS X:

              ```
              kubectl get cm -n kube-system calico-config -o yaml | grep "etcd_endpoints:" | awk '{ print $2 }'
              ```
              {: pre}

          -   출력 예:

              ```
               https://169.1.1.1:30001
              ```
              {: screen}

          -   Windows:<ol>
            <li>구성 맵에서 calico 구성 값을 가져오십시오. </br><pre class="codeblock"><code>kubectl get cm -n kube-system calico-config -o yaml</code></pre></br>
            <li>`data` 섹션에서 etcd_endpoints 값을 찾으십시오. 예: <code>https://169.1.1.1:30001</code>
            </ol>

        2.  Kubernetes 인증서가 다운로드되는 디렉토리인 `<CERTS_DIR>`을 검색하십시오. 

            -   Linux 및 OS X:

              ```
              dirname $KUBECONFIG
              ```
              {: pre}

                출력 예:

              ```
              /home/sysadmin/.bluemix/plugins/container-service/clusters/<cluster_name>-admin/
              ```
              {: screen}

            -   Windows:

              ```
              ECHO %KUBECONFIG%
              ```
              {: pre}

                출력 예:

              ```
              C:/Users/<user>/.bluemix/plugins/container-service/<cluster_name>-admin/kube-config-prod-<location>-<cluster_name>.yml
              ```
              {: screen}

            **참고**: 디렉토리 경로를 가져오려면 출력의 끝에서 파일 이름 `kube-config-prod-<location>-<cluster_name>.yml`을 제거하십시오.

        3.  <code>ca-*pem_file<code>을 검색하십시오. 

            -   Linux 및 OS X:

              ```
              ls `dirname $KUBECONFIG` | grep "ca-"
              ```
              {: pre}

            -   Windows:<ol><li>이전 단계에서 검색한 디렉토리를 여십시오. </br><pre class="codeblock"><code>C:\Users\<user>\.bluemix\plugins\container-service\&lt;cluster_name&gt;-admin\</code></pre>
              <li> <code>ca-*pem_file</code> 파일을 찾으십시오. </ol>

        4.  Calico 구성이 올바르게 작동하고 있는지 확인하십시오. 

            -   Linux 및 OS X:

              ```
              calicoctl get nodes
              ```
              {: pre}

            -   Windows:

              ```
              calicoctl get nodes --config=<path_to_>/calicoctl.cfg
              ```
              {: pre}

              출력:

              ```
              NAME
              kube-dal10-crc21191ee3997497ca90c8173bbdaf560-w1.cloud.ibm
              kube-dal10-crc21191ee3997497ca90c8173bbdaf560-w2.cloud.ibm
              kube-dal10-crc21191ee3997497ca90c8173bbdaf560-w3.cloud.ibm
              ```
              {: screen}

3.  기존 네트워크 정책을 조사하십시오. 

    -   Calico 호스트 엔드포인트를 보십시오. 

      ```
      calicoctl get hostendpoint -o yaml
      ```
      {: pre}

    -   클러스터에 작성된 모든 Calico 및 Kubernetes 네트워크 정책을 보십시오. 이 목록에는 포드 또는 호스트에 아직 적용되지 않았을 수 있는 정책이 포함됩니다. 네트워크 정책이 적용되려면 Calico 네트워크 정책에 정의된 선택기와 일치하는 Kubernetes 리소스를 찾아야 합니다. 

      ```
      calicoctl get policy -o wide
      ```
      {: pre}

    -   네트워크 정책의 세부사항을 보십시오. 

      ```
      calicoctl get policy -o yaml <policy_name>
      ```
      {: pre}

    -   클러스터에 대한 모든 네트워크 정책의 세부사항을 보십시오. 

      ```
      calicoctl get policy -o yaml
      ```
      {: pre}

4.  트래픽을 허용하거나 차단하도록 Calico 네트워크 정책을 작성하십시오. 

    1.  구성 스크립트(.yaml)를 작성하여 [Calico 네트워크 정책![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](http://docs.projectcalico.org/v2.1/reference/calicoctl/resources/policy)을 정의하십시오. 이러한 구성 파일에는 이러한 정책이 적용되는 포드, 네임스페이스 또는 호스트를 설명하는 선택기가 포함됩니다. 사용자 공유 정책을 작성하려면 이러한 [샘플 Calico 정책 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](http://docs.projectcalico.org/v2.0/getting-started/kubernetes/tutorials/advanced-policy)을 참조하십시오.

    2.  클러스터에 정책을 적용하십시오. 
        -   Linux 및 OS X:

          ```
          calicoctl apply -f <policy_file_name.yaml>
          ```
          {: pre}

        -   Windows:

          ```
          calicoctl apply -f <path_to_>/<policy_file_name.yaml> --config=<path_to_>/calicoctl.cfg
          ```
          {: pre}

### LoadBalancer 또는 NodePort 서비스에 대한 수신 트래픽 차단.
{: #cs_block_ingress}

기본적으로 Kubernetes `NodePort` 및 `LoadBalancer` 서비스는 앱을 모든 퍼블릭 및 프라이빗 클러스터 인터페이스에서 사용 가능하게 하도록 설계되었습니다. 그러나 트래픽 소스 또는 대상을 기반으로 서비스에 대한 수신 트래픽을 차단할 수 있습니다. 트래픽을 차단하려면 Calico `preDNAT` 네트워크 정책을 작성하십시오.

Kubernetes LoadBalancer 서비스는 NodePort 서비스이기도 합니다. LoadBalancer 서비스는 로드 밸런서 IP 주소 및 포트를 통해 앱을 사용할 수 있도록 하고 서비스의 노드 포트를 통해 앱을 사용할 수 있도록 합니다. 클러스터 내의 모든 노드에 대한 모든 IP 주소(공인 및 사설)에서 노드 포트에 액세스할 수 있습니다.

클러스터 관리자는 Calico `preDNAT` 네트워크 정책을 사용하여 다음을 차단할 수 있습니다.

  - NodePort 서비스에 대한 트래픽. LoadBalancer 서비스에 대한 트래픽을 허용됩니다.
  - 소스 어댑터 또는 CIDR을 기반으로 하는 트래픽

Calico `preDNAT` 네트워크 정책에 대한 몇 가지 일반적 사용:

  - 사설 LoadBalancer 서비스의 공용 노드 포트에 대한 트래픽을 차단합니다.
  - [에지 작업자 노드](#cs_edge)를 실행하는 클러스터의 공용 노드 포트에 대한 트래픽을 차단합니다. 노드 포트를 차단하면 에지 작업자 노드가 수신 트래픽을 처리하는 유일한 작업자 노드가 됩니다.

Kubernetes NodePort 및 LoadBalancer 서비스에 대해 생성된 DNAT iptables 규칙으로 인해 기본 Kubernetes 및 Calico 정책을 이러한 서비스의 보호에 적용하기 어렵기 때문에 `preDNAT` 네트워크 정책이 유용합니다.

Calico `preDNAT` 네트워크 정책은 [Calico 네트워크 정책 리소스 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://docs.projectcalico.org/v2.4/reference/calicoctl/resources/policy)를 기반으로 iptables 규칙을 생성합니다.

1. Kubernetes 서비스에 대한 Ingress 액세스를 위한 Calico `preDNAT` 네트워크 정책을 정의하십시오. 

  모든 노드 포트를 차단하는 예:

  ```
  apiVersion: v1
  kind: policy
  metadata:
    name: deny-kube-node-port-services
  spec:
    preDNAT: true
    selector: ibm.role in { 'worker_public', 'master_public' }
    ingress:
    - action: deny
      protocol: tcp
      destination:
        ports:
        - 30000:32767
    - action: deny
      protocol: udp
      destination:
        ports:
        - 30000:32767
  ```
  {: codeblock}

2. Calico preDNAT 네트워크 정책을 적용하십시오. 클러스터 전체에 정책 변경사항을 적용하는 데 약 1분이 걸립니다.

  ```
  calicoctl apply -f deny-kube-node-port-services.yaml
  ```
  {: pre}

<br />



## 네트워크 트래픽을 에지 작업자 노드로 제한
{: #cs_edge}

`dedicated=edge` 레이블을 클러스터의 둘 이상의 작업자 노드에 추가하여 Ingress와 로드 밸런서가 해당 작업자 노드에만 배치되도록 하십시오.

에지 작업자 노드는 외부에서 적은 수의 작업자 노드에 액세스할 수 있게 하고 네트워크 워크로드를 격리함으로써 클러스터의 보안을 향상시킬 수 있습니다. 이러한 작업자 노드가 네트워킹 전용으로 표시되는 경우 다른 워크로드는 작업자 노드의 CPU 또는 메모리를 이용할 수 없고 네트워킹을 방해할 수도 없습니다.

시작하기 전에:

- [표준 클러스터를 작성하십시오. ](cs_cluster.html#cs_cluster_cli)
- 클러스터에 하나 이상의 퍼블릭 VLAN을 가지고 있는지 확인하십시오. 에지 작업자 노드는 프라이빗 VLAN만 있는 클러스터에서는 사용할 수 없습니다.
- [Kubernetes CLI를 클러스터에 대상으로 지정하십시오](cs_cli_install.html#cs_cli_configure). 


1. 클러스터의 작업자 노드를 나열하십시오. **NAME** 열에서 사설 IP 주소를 사용하여 노드를 식별하십시오. 에지 작업자 노드가 될 둘 이상의 작업자 노드를 선택하십시오. 둘 이상의 작업자 노드를 사용하면 네트워킹 리소스 가용성이 향상됩니다.

  ```
  kubectl get nodes -L publicVLAN,privateVLAN,dedicated
  ```
  {: pre}

2. `dedicated=edge`로 작업자 노드의 레이블을 지정하십시오. 작업자 노드가 `dedicated=edge`와 함께 표시되면 모든 후속 Ingress 및 로드 밸런서가 에지 작업자 노드에 배치됩니다.

  ```
  kubectl label nodes <node_name> <node_name2> dedicated=edge
  ```
  {: pre}

3. 클러스터에서 기존 로드 밸런서 서비스를 모두 검색하십시오.

  ```
  kubectl get services --all-namespaces -o jsonpath='{range .items[*]}kubectl get service -n {.metadata.namespace} {.metadata.name} -o yaml | kubectl apply -f - :{.spec.type},{end}' | tr "," "\n" | grep "LoadBalancer" | cut -d':' -f1
  ```
  {: pre}

  출력:

  ```
  kubectl get service -n <namespace> <name> -o yaml | kubectl apply -f
  ```
  {: screen}

4. 이전 단계의 출력을 사용하여 각 `kubectl get service` 행을 복사하고 붙여넣으십시오. 이 명령은 에지 작업자 노드에 로드 밸런서를 다시 배치합니다. 공용 로드 밸런서만 다시 배치해야 합니다.

  출력:

  ```
  service "<name>" configured
  ```
  {: screen}

`dedicated=edge`로 작업자 노드의 레이블을 지정했으며 기존 모든 로드 밸런서와 Ingress를 에지 작업자 노드에 다시 배치했습니다. 그 다음 다른 [워크로드가 에지 작업자 노드에서 실행](#cs_edge_workloads)되지 않도록 하고 [작업자 노드에서 노드 포트에 대한 인바운드 트래픽을 차단](#cs_block_ingress)하십시오.

### 워크로드가 에지 작업자 노드에서 실행되지 않도록 함
{: #cs_edge_workloads}

에지 작업자 노드의 이점 중 하나는 네트워킹 서비스만 실행하도록 이러한 작업자 노드를 지정할 수 있다는 것입니다. `dedicated=edge` 결함 허용을 사용하면 모든 로드 밸런서와 Ingress 서비스가 레이블 지정된 작업자 노드에만 배치됩니다. 하지만 다른 워크로드가 에지 작업자 노드에서 실행되지 않고 작업자 노드 리소스를 이용하지 못하도록 하려면 [Kubernetes 오염 ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)을 사용해야 합니다.

1. `edge` 레이블이 있는 모든 작업자 노드를 나열하십시오.

  ```
  kubectl get nodes -L publicVLAN,privateVLAN,dedicated -l dedicated=edge
  ```
  {: pre}

2. 포드가 작업자 노드에서 실행되지 않도록 하고 작업자 노드에서 `edge` 레이블이 없는 포드를 제거하는 오염을 각 작업자 노드에 적용하십시오. 제거된 포드는 용량이 있는 다른 작업자 노드에 다시 배치됩니다.

  ```
  kubectl taint node <node_name> dedicated=edge:NoSchedule dedicated=edge:NoExecute
  ```

이제 `dedicated=edge` 결함 허용을 사용하는 포드만 에지 작업자 노드에 배치됩니다.
