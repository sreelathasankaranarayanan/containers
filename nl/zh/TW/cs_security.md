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


# {{site.data.keyword.containerlong_notm}} 的安全
{: #cs_security}

您可以使用內建的安全特性，以進行風險分析及安全保護。這些特性可協助您保護叢集基礎架構及網路通訊、隔離運算資源，以及確保基礎架構元件和容器部署的安全相符性。
{: shortdesc}

## 依叢集元件的安全
{: #cs_security_cluster}

每一個 {{site.data.keyword.containerlong_notm}} 叢集的[主節點](#cs_security_master)及[工作者節點](#cs_security_worker)都有內建安全特性。如果您有防火牆，則需要從叢集外部存取負載平衡，或者，在組織網路原則阻止存取公用網際網路端點時，從您的本端系統執行 `kubectl` 指令（[開啟防火牆中的埠](#opening_ports)）。如果您要將叢集中的應用程式連接至內部部署網路或叢集外部的其他應用程式，請[設定 VPN 連線功能](#vpn)。
{: shortdesc}

在下圖中，您可以看到依 Kubernetes 主節點、工作者節點及容器映像檔分組的安全特性。

<img src="images/cs_security.png" width="400" alt="{{site.data.keyword.containershort_notm}} 叢集安全" style="width:400px; border-style: none"/>


  <table summary="表格中的第一列跨這兩個直欄。其餘的列應該從左到右閱讀，第一欄為伺服器位置，第二欄則為要符合的 IP 位址。">
  <caption>表 1. 安全特性</caption>
  <thead>
  <th colspan=2><img src="images/idea.png" alt="構想圖示"/> {{site.data.keyword.containershort_notm}} 中的內建叢集安全設定</th>
  </thead>
  <tbody>
    <tr>
      <td>Kubernetes 主節點</td>
      <td>每一個叢集中的 Kubernetes 主節點都是由 IBM 管理，並且具備高可用性。它包括 {{site.data.keyword.containershort_notm}} 安全設定，可確保安全相符性及進出工作者節點的安全通訊。IBM 會視需要執行更新。專用 Kubernetes 主節點可集中控制及監視叢集中的所有 Kubernetes 資源。根據叢集中的部署需求及容量，Kubernetes 主節點會自動排定容器化應用程式，在可用的工作者節點之間進行部署。如需相關資訊，請參閱 [Kubernetes 主節點安全](#cs_security_master)。</td>
    </tr>
    <tr>
      <td>工作者節點</td>
      <td>容器部署於叢集專用的工作者節點，可確保 IBM 客戶的運算、網路及儲存空間隔離。{{site.data.keyword.containershort_notm}} 提供內建安全特性，讓工作者節點在專用及公用網路上保持安全，以及確保工作者節點的安全相符性。如需相關資訊，請參閱[工作者節點安全](#cs_security_worker)。此外，您也可以新增 [Calico 網路原則](#cs_security_network_policies)，以進一步指定您要容許或封鎖進出工作者節點上 Pod 的網路資料流量。</td>
     </tr>
     <tr>
      <td>映像檔</td>
      <td>身為叢集管理者，您可以在 {{site.data.keyword.registryshort_notm}} 中設定您自己的安全 Docker 映像檔儲存庫，您可以在其中儲存 Docker 映像檔並且在叢集使用者之間進行共用。為了確保安全容器部署，「漏洞警告器」會掃描專用登錄中的每個映像檔。「漏洞警告器」是 {{site.data.keyword.registryshort_notm}} 的元件，可掃描以尋找潛在漏洞、提出安全建議，並提供解決漏洞的指示。如需相關資訊，請參閱 [{{site.data.keyword.containershort_notm}} 中的映像檔安全](#cs_security_deployment)。</td>
    </tr>
  </tbody>
</table>

### Kubernetes 主節點
{: #cs_security_master}

檢閱內建 Kubernetes 主節點安全特性，此安全特性用來保護 Kubernetes 主節點，以及保護叢集網路通訊安全。
{: shortdesc}

<dl>
  <dt>完整受管理及專用的 Kubernetes 主節點</dt>
    <dd>{{site.data.keyword.containershort_notm}} 中的每個 Kubernetes 叢集都是由 IBM 透過 IBM 擁有的 IBM Cloud 基礎架構 (SoftLayer) 帳戶所管理的專用 Kubernetes 主節點予以控制。Kubernetes 主節點已設定下列未與其他 IBM 客戶共用的專用元件。<ul><li>etcd 資料儲存庫：儲存叢集的所有 Kubernetes 資源（例如「服務」、「部署」及 Pod）。Kubernetes ConfigMap 及 Secret 是儲存為金鑰值組的應用程式資料，因此，Pod 中執行的應用程式可以使用它們。傳送至 Pod 時，etcd 中的資料會儲存在 IBM 所管理並透過 TLS 加密的已加密磁碟中，以確保資料保護及完整性。</li>
    <li>kube-apiserver：提供為從工作者節點到 Kubernetes 主節點的所有要求的主要進入點。kube-apiserver 會驗證及處理要求，並且可以讀取及寫入 etcd 資料儲存庫。</li>
    <li>kube-scheduler：考量容量及效能需求、軟硬體原則限制、反親緣性規格及工作負載需求，以決定在何處部署 Pod。如果找不到符合需求的工作者節點，則不會在叢集中部署 Pod。</li>
    <li>kube-controller-manager：負責監視抄本集，以及建立對應的 Pod 來達到想要的狀態。</li>
    <li>OpenVPN：{{site.data.keyword.containershort_notm}} 特定元件，提供所有 Kubernetes 主節點與工作者節點的通訊的安全網路連線功能。</li></ul></dd>
  <dt>所有工作者節點與 Kubernetes 主節點的通訊的 TLS 安全網路連線功能</dt>
    <dd>為了保護與 Kubernetes 主節點的網路通訊安全，{{site.data.keyword.containershort_notm}} 會產生 TLS 憑證，以加密每個叢集的 kube-apiserver 及 etcd 資料儲存庫元件的進出通訊。這些憑證絕不會在叢集或 Kubernetes 主節點元件之間共用。</dd>
  <dt>所有 Kubernetes 主節點與工作者節點的通訊的 OpenVPN 安全網路連線功能</dt>
    <dd>雖然 Kubernetes 會使用 `https` 通訊協定來保護 Kubernetes 主節點與工作者節點之間的通訊，但是預設不會提供工作者節點的鑑別。為了保護此通訊，{{site.data.keyword.containershort_notm}} 會在建立叢集時自動設定 Kubernetes 主節點與工作者節點之間的 OpenVPN 連線。</dd>
  <dt>持續 Kubernetes 主節點網路監視</dt>
    <dd>IBM 會持續監視每個 Kubernetes 主節點，以控制及重新修補處理程序層次的「拒絕服務 (DOS)」攻擊。</dd>
  <dt>Kubernetes 主節點節點安全相符性</dt>
    <dd>{{site.data.keyword.containershort_notm}} 會自動掃描每個已部署 Kubernetes 主節點的節點，以尋找需要套用以確保主節點保護的 Kubernetes 及 OS 特定安全修正程式中存在的漏洞。如果找到漏洞，{{site.data.keyword.containershort_notm}} 會自動套用修正程式，並代表使用者來解決漏洞。</dd>
</dl>

<br />


### 工作者節點
{: #cs_security_worker}

檢閱內建的工作者節點安全特性，以保護工作者節點環境，以及確保資源、網路及儲存空間隔離。
{: shortdesc}

<dl>
  <dt>運算、網路及儲存空間基礎架構隔離</dt>
    <dd>當您建立叢集時，會將虛擬機器佈建為客戶 IBM Cloud 基礎架構 (SoftLayer) 帳戶中的工作者節點，或由 IBM 將虛擬機器佈建為專用 IBM Cloud 基礎架構 (SoftLayer) 帳戶中的工作者節點。工作者節點專用於叢集，而且未管理其他叢集的工作負載。</br> 每個 {{site.data.keyword.Bluemix_notm}} 帳戶都已設定 IBM Cloud 基礎架構 (SoftLayer) VLAN，確保工作者節點上的優質網路效能及隔離。</br>若要持續保存叢集中的資料，您可以從 IBM Cloud 基礎架構 (SoftLayer) 佈建專用 NFS 型檔案儲存空間，並運用該平台的內建資料安全特性。</dd>
  <dt>設定安全的工作者節點</dt>
    <dd>每個工作者節點都設定了 Ubuntu 作業系統，且使用者無法變更。為了保護工作者節點的作業系統不會受到潛在攻擊，每個工作者節都會以 Linux iptables 規則所強制執行的專家級防火牆設定來進行配置。</br> 在 Kubernetes 上執行的所有容器都受到預先定義的 Calico 網路原則設定所保護，這些設定是在建立叢集期間，配置於每個工作者節點上。這項設定確保工作者節點與 Pod 之間的安全網路通訊。若要進一步限制容器可以在工作者節點上執行的動作，使用者可以選擇在工作者節點上配置 [AppArmor 原則 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://kubernetes.io/docs/tutorials/clusters/apparmor/)。</br> 工作者節點上會停用 SSH 存取。如果您要在工作者節點上安裝其他特性，則可以針對您要在每個工作者節點上執行的所有項目，使用 [Kubernetes 常駐程式集 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset)，或是針對您必須執行的任何一次性動作，使用 [Kubernetes 工作 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)。</dd>
  <dt>Kubernetes 工作者節點安全相符性</dt>
    <dd>IBM 與內部及外部安全顧問團隊合作，解決潛在的安全相符性漏洞。IBM 會維護對工作者節點的存取，以將更新項目及安全修補程式部署至作業系統。</br> <b>重要事項</b>：請定期重新啟動工作者節點，以確保安裝自動部署至作業系統的更新項目及安全修補程式。IBM 不會將您的工作者節點重新開機。</dd>
  <dt>已加密磁碟</dt>
  <dd>依預設，{{site.data.keyword.containershort_notm}} 在佈建時，會為所有工作者節點提供兩個本端 SSD 已加密資料分割區。第一個分割區未加密，而在使用 LUKS 加密金鑰佈建時，會解除鎖定裝載至 _/var/lib/docker_ 的第二個分割區。每個 Kubernetes 叢集中的每一個工作者節點都有由 {{site.data.keyword.containershort_notm}} 所管理的專屬唯一 LUKS 加密金鑰。當您建立叢集或將工作者節點新增至現有叢集時，會安全地取回金鑰，然後在解除鎖定已加密磁碟之後予以捨棄。
  <p><b>附註</b>：加密可能會影響磁碟 I/O 效能。對於需要高效能磁碟 I/O 的工作負載，請測試已啟用及停用加密的叢集，以協助您決定是否關閉加密。</p>
  </dd>
  <dt>支援 IBM Cloud 基礎架構 (SoftLayer) 網路防火牆</dt>
    <dd>{{site.data.keyword.containershort_notm}} 與所有 [IBM Cloud 基礎架構 (SoftLayer) 防火牆供應項目 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://www.ibm.com/cloud-computing/bluemix/network-security) 相容。在「{{site.data.keyword.Bluemix_notm}} 公用」上，您可以使用自訂網路原則來設定防火牆，以提供叢集的專用網路安全，以及偵測及重新修補網路侵入。例如，您可能選擇設定 [Vyatta ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://knowledgelayer.softlayer.com/topic/vyatta-1) 作為防火牆，並且封鎖不想要的資料流量。當您設定防火牆時，[也必須開啟必要埠及 IP 位址](#opening_ports)（針對每一個地區），讓主節點與工作者節點可以進行通訊。</dd>
  <dt>將服務維持為專用狀態，或選擇性地將服務及應用程式公開給公用網際網路使用</dt>
    <dd>您可以選擇將服務及應用程式維持為專用狀態，並運用本主題所述的內建安全特性，來確保工作者節點與 Pod 之間的安全通訊。若要將服務及應用程式公開給公用網際網路使用，您可以運用 Ingress 及負載平衡器支援，安全地將服務設為可公開使用。</dd>
  <dt>將工作者節點及應用程式安全地連接至內部部署資料中心</dt>
  <dd>若要將工作者節點及應用程式連接至內部部署資料中心，您可以配置具有 Strongswan 服務或者 Vyatta Gateway Appliance 或 Fortigate Appliance 的 VPN IPSec 端點。<br><ul><li><b>Strongswan IPSec VPN 服務</b>：您可以設定 [Strongswan IPSec VPN 服務 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://www.strongswan.org/)，以安全地連接 Kubernetes 叢集與內部部署網路。在根據業界標準網際網路通訊協定安全 (IPsec) 通訊協定套組的網際網路上，Strongswan IPSec VPN 服務提供安全的端對端通訊通道。若要設定叢集與內部部署網路之間的安全連線，您必須已在內部部署資料中心內安裝 IPsec VPN 閘道或 IBM Cloud 基礎架構 (SoftLayer) 伺服器。然後，您可以在 Kubernetes Pod 中[配置及部署 Strongwan IPSec VPN 服務](cs_security.html#vpn)。</li><li><b>Vyatta Gateway Appliance 或 Fortigate Appliance</b>：如果您有較大的叢集，則可能會選擇設定 Vyatta Gateway Appliance 或 Fortigate Appliance，以配置 IPSec VPN 端點。如需相關資訊，請參閱[將叢集連接至內部部署資料中心 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://www.ibm.com/blogs/bluemix/2017/07/kubernetes-and-bluemix-container-based-workloads-part4/) 上的這篇部落格文章。</li></ul></dd>
  <dt>叢集活動的持續監視及記載</dt>
    <dd>對於標準叢集，{{site.data.keyword.containershort_notm}} 可以記載並監視所有叢集相關事件（例如新增工作者節點）、漸進式更新進度或容量使用資訊，並將其傳送至 {{site.data.keyword.loganalysislong_notm}} 及 {{site.data.keyword.monitoringlong_notm}}。如需設定記載及監視的相關資訊，請參閱[配置叢集記載](https://console.bluemix.net/docs/containers/cs_cluster.html#cs_logging)及[配置叢集監視](https://console.bluemix.net/docs/containers/cs_cluster.html#cs_monitoring)。</dd>
</dl>

### 映像檔
{: #cs_security_deployment}

使用內建安全特性來管理映像檔的安全及完整性。
{: shortdesc}

<dl>
<dt>{{site.data.keyword.registryshort_notm}} 中的安全 Docker 專用映像檔儲存庫</dt>
<dd>您可以在多方承租戶中設定您自己的 Docker 映像檔儲存庫、高可用性，以及由 IBM 所管理的可擴充專用映像檔儲存庫，以建置、安全地儲存 Docker 映像檔，並且在叢集使用者之間進行共用。</dd>

<dt>映像檔安全相符性</dt>
<dd>當您使用 {{site.data.keyword.registryshort_notm}} 時，可以運用「漏洞警告器」所提供的內建安全掃描。會自動掃描每個推送至您名稱空間的映像檔，以對照已知 CentOS、Debian、Red Hat 及 Ubuntu 問題資料庫，掃描漏洞。如果發現漏洞，「漏洞警告器」會提供其解決方式的指示，以確保映像檔的完整性及安全。</dd>
</dl>

若要檢視映像檔的漏洞評量，請[檢閱漏洞警告器文件](/docs/services/va/va_index.html#va_registry_cli)。

<br />


## 在防火牆中開啟必要埠及 IP 位址
{: #opening_ports}

請檢閱下列狀況，您在下列狀況時可能需要在防火牆中開啟特定的埠和 IP 位址：
* 當組織網路原則阻止透過 Proxy 或防火牆存取公用網際網路端點時，從本端系統[執行 `bx` 指令](#firewall_bx)。
* 當組織網路原則阻止透過 Proxy 或防火牆存取公用網際網路端點時，從本端系統[執行 `kubectl` 指令](#firewall_kubectl)。
* 當組織網路原則阻止透過 Proxy 或防火牆存取公用網際網路端點時，從本端系統[執行 `calicoctl` 指令](#firewall_calicoctl)。
* 當已針對工作者節點設定防火牆，或是在 IBM Cloud 基礎架構 (SoftLayer) 帳戶中自訂防火牆設定時，[容許 Kubernetes 主節點與工作者節點之間的通訊](#firewall_outbound)。
* [從叢集外部存取 NodePort 服務、LoadBalancer 服務或 Ingress](#firewall_inbound)。

### 在防火牆的保護下執行 `bx cs` 指令
{: #firewall_bx}

如果組織網路原則阻止透過 Proxy 或防火牆從本端系統存取公用端點，則若要執行 `bx cs` 指令，您必須容許 {{site.data.keyword.containerlong_notm}} 的 TCP 存取。

1. 容許埠 443 上對 `containers.bluemix.net` 的存取。
2. 驗證連線。如果已正確配置存取，則會在輸出中顯示船。
   ```
   curl https://containers.bluemix.net/v1/
   ```
   {: pre}

   輸出範例：
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

### 在防火牆的保護下執行 `kubectl` 指令
{: #firewall_kubectl}

如果組織網路原則阻止透過 Proxy 或防火牆從本端系統存取公用端點，則若要執行 `kubectl` 指令，您必須容許叢集的 TCP 存取。

建立叢集時，會從 20000-32767 內隨機指派主要 URL 中的埠。您可以針對任何可能建立的叢集選擇開啟埠範圍 20000-32767，或選擇容許存取特定現有叢集。

開始之前，請容許[執行 `bx cs` 指令](#firewall_bx)的存取。

若要容許存取特定叢集，請執行下列動作：

1. 登入 {{site.data.keyword.Bluemix_notm}} CLI。系統提示時，請輸入您的 {{site.data.keyword.Bluemix_notm}} 認證。如果您有聯合帳戶，請包括 `--sso` 選項。

    ```
    bx login [--sso]
    ```
    {: pre}

2. 選取您叢集所在的地區。

   ```
   bx cs region-set
   ```
   {: pre}

3. 取得叢集的名稱。

   ```
      bx cs clusters
      ```
   {: pre}

4. 擷取叢集的**主要 URL**。

   ```
   bx cs cluster-get <cluster_name_or_id>
   ```
   {: pre}

   輸出範例：
   ```
   ...
   Master URL:		https://169.46.7.238:31142
   ...
   ```
   {: screen}

5. 容許在埠上存取**主要 URL**，例如前一個範例中的埠 `31142`。

6. 驗證連線。

   ```
   curl --insecure <master_URL>/version
   ```
   {: pre}

   範例指令：
   ```
   curl --insecure https://169.46.7.238:31142/version
   ```
   {: pre}

   輸出範例：
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

7. 選用項目：針對您需要公開的每一個叢集，重複這些步驟。

### 在防火牆的保護下執行 `calicoctl` 指令
{: #firewall_calicoctl}

如果組織網路原則阻止透過 Proxy 或防火牆從本端系統存取公用端點，則若要執行 `calicoctl` 指令，您必須容許 Calico 指令的 TCP 存取。

開始之前，請容許執行 [`bx` 指令](#firewall_bx)及 [`kubectl` 指令](#firewall_kubectl)的存取。

1. 從用來容許 [`kubectl` 指令](#firewall_kubectl)的主要 URL 中擷取 IP 位址。

2. 取得 ETCD 的埠。

  ```
  kubectl get cm -n kube-system calico-config -o yaml | grep etcd_endpoints
  ```
  {: pre}

3. 容許透過主要 URL IP 位址及 ETCD 埠存取 Calico 原則。

### 容許叢集存取基礎架構資源及其他服務
{: #firewall_outbound}

  1.  記下叢集中所有工作者節點的公用 IP 位址。

      ```
      bx cs workers <cluster_name_or_id>
      ```
      {: pre}

  2.  容許從來源 _<each_worker_node_publicIP>_ 到目的地 TCP/UDP 埠範圍 20000-32767 和埠 443 以及下列 IP 位址和網路群組的送出網路資料流量。如果您的公司防火牆阻止本端機器存取公用網際網路端點，請針對來源工作者節點及本端機器執行此步驟。
      - **重要事項**：對於地區內的所有位置，您必須容許對埠 443 的送出資料流量，以平衡引導處理程序期間的負載。例如，如果您的叢集是在美國南部，則對於所有位置（dal10、dal12 及 dal13），您必須容許從埠 443 到 IP 位址的資料流量。
      <p>
  <table summary="表格中的第一列跨這兩個直欄。其餘的列應該從左到右閱讀，第一欄為伺服器位置，第二欄則為要符合的 IP 位址。">
  <thead>
      <th>地區</th>
      <th>位置</th>
      <th>IP 位址</th>
      </thead>
    <tbody>
      <tr>
        <td>亞太地區北部</td>
        <td>hkg02<br>tok02</td>
        <td><code>169.56.132.234</code><br><code>161.202.126.210</code></td>
       </tr>
      <tr>
         <td>亞太地區南部</td>
         <td>mel01<br>syd01<br>syd04</td>
         <td><code>168.1.97.67</code><br><code>168.1.8.195</code><br><code>130.198.64.19, 130.198.66.34</code></td>
      </tr>
      <tr>
         <td>歐盟中部</td>
         <td>ams03<br>fra02<br>mil01<br>par01</td>
         <td><code>169.50.169.106, 169.50.154.194</code><br><code>169.50.56.170, 169.50.56.174</code><br><code>159.122.190.98</code><br><code>159.8.86.149, 159.8.98.170</code></td>
        </tr>
      <tr>
        <td>英國南部</td>
        <td>lon02<br>lon04</td>
        <td><code>159.122.242.78</code><br><code>158.175.65.170, 158.175.74.170, 158.175.76.2</code></td>
      </tr>
      <tr>
        <td>美國東部</td>
         <td>tor01<br>wdc06<br>wdc07</td>
         <td><code>169.53.167.50</code><br><code>169.60.73.142</code><br><code>169.61.83.62</code></td>
      </tr>
      <tr>
        <td>美國南部</td>
        <td>dal10<br>dal12<br>dal13</td>
        <td><code>169.47.234.18, 169.46.7.234</code><br><code>169.47.70.10</code><br><code>169.60.128.2</code></td>
      </tr>
      </tbody>
    </table>
</p>

  3.  容許從工作者節點到 {{site.data.keyword.registrylong_notm}} 的送出網路資料流量：
      - `TCP port 443 FROM <each_worker_node_publicIP> TO <registry_publicIP>`
      - 將 <em>&lt;registry_publicIP&gt;</em> 取代為您要容許資料流量的登錄地區的所有位址：
        <p>
<table summary="表格中的第一列跨這兩個直欄。其餘的列應該從左到右閱讀，第一欄為伺服器位置，第二欄則為要符合的 IP 位址。">
  <thead>
        <th>容器地區</th>
        <th>登錄位址</th>
        <th>登錄 IP 位址</th>
      </thead>
      <tbody>
        <tr>
          <td>亞太地區北部、亞太地區南部</td>
          <td>registry.au-syd.bluemix.net</td>
          <td><code>168.1.45.160/27</code></br><code>168.1.139.32/27</code></td>
        </tr>
        <tr>
          <td>歐盟中部</td>
          <td>registry.eu-de.bluemix.net</td>
          <td><code>169.50.56.144/28</code></br><code>159.8.73.80/28</code></td>
         </tr>
         <tr>
          <td>英國南部</td>
          <td>registry.eu-gb.bluemix.net</td>
          <td><code>159.8.188.160/27</code></br><code>169.50.153.64/27</code></td>
         </tr>
         <tr>
          <td>美國東部、美國南部</td>
          <td>registry.ng.bluemix.net</td>
          <td><code>169.55.39.112/28</code></br><code>169.46.9.0/27</code></br><code>169.55.211.0/27</code></td>
         </tr>
        </tbody>
      </table>
</p>

  4.  選用項目：容許從工作者節點到 {{site.data.keyword.monitoringlong_notm}} 及 {{site.data.keyword.loganalysislong_notm}} 服務的送出網路資料流量：
      - `TCP port 443, port 9095 FROM <each_worker_node_publicIP> TO <monitoring_publicIP>`
      - 將 <em>&lt;monitoring_publicIP&gt;</em> 取代為您要容許資料流量的監視地區的所有位址：
        <p><table summary="表格中的第一列跨這兩個直欄。其餘的列應該從左到右閱讀，第一欄為伺服器位置，第二欄則為要符合的 IP 位址。">
  <thead>
        <th>容器地區</th>
        <th>監視位址</th>
        <th>監視 IP 位址</th>
        </thead>
      <tbody>
        <tr>
         <td>歐盟中部</td>
         <td>metrics.eu-de.bluemix.net</td>
         <td><code>159.122.78.136/29</code></td>
        </tr>
        <tr>
         <td>英國南部</td>
         <td>metrics.eu-gb.bluemix.net</td>
         <td><code>169.50.196.136/29</code></td>
        </tr>
        <tr>
          <td>美國東部、美國南部、亞太地區北部</td>
          <td>metrics.ng.bluemix.net</td>
          <td><code>169.47.204.128/29</code></td>
         </tr>
         
        </tbody>
      </table>
</p>
      - `TCP port 443, port 9091 FROM <each_worker_node_publicIP> TO <logging_publicIP>`
      - 將 <em>&lt;logging_publicIP&gt;</em> 取代為您要容許資料流量的記載地區的所有位址：
        <p><table summary="表格中的第一列跨這兩個直欄。其餘的列應該從左到右閱讀，第一欄為伺服器位置，第二欄則為要符合的 IP 位址。">
  <thead>
        <th>容器地區</th>
        <th>記載位址</th>
        <th>記載 IP 位址</th>
        </thead>
        <tbody>
          <tr>
            <td>美國東部、美國南部</td>
            <td>ingest.logging.ng.bluemix.net</td>
            <td><code>169.48.79.236</code><br><code>169.46.186.113</code></td>
           </tr>
          <tr>
           <td>歐盟中部、英國南部</td>
           <td>ingest-eu-fra.logging.bluemix.net</td>
           <td><code>158.177.88.43</code><br><code>159.122.87.107</code></td>
          </tr>
          <tr>
           <td>亞太地區南部、亞太地區北部</td>
           <td>ingest-au-syd.logging.bluemix.net</td>
           <td><code>130.198.76.125</code><br><code>168.1.209.20</code></td>
          </tr>
         </tbody>
       </table>
</p>

  5. 對於專用防火牆，容許適當的 IBM Cloud 基礎架構 (SoftLayer) 專用 IP 範圍。請參閱[此鏈結](https://knowledgelayer.softlayer.com/faq/what-ip-ranges-do-i-allow-through-firewall)，從 **Backend (private) Network** 小節開始。
      - 新增所使用[地區內的所有位置](cs_regions.html#locations)。
      - 請注意，您必須新增 dal01 位置（資料中心）。
      - 開啟埠 80 及 443，以容許叢集引導處理程序。

  6. 若要建立資料儲存空間的持續性磁區宣告，請透過防火牆容許對叢集所在位置（資料中心）的 [IBM Cloud 基礎架構 (SoftLayer) IP 位址](https://knowledgelayer.softlayer.com/faq/what-ip-ranges-do-i-allow-through-firewall)進行 Egress 存取。
      - 若要尋找叢集的位置（資料中心），請執行 `bx cs clusters`。
      - 容許存取**前端（公用）網路**及**後端（專用）網路**的 IP 範圍。
      - 請注意，您必須針對**後端（專用）網路**新增 dal01 位置（資料中心）。

### 從叢集外部存取 NodePort、負載平衡器及 Ingress 服務
{: #firewall_inbound}

您可以容許對 NodePort、負載平衡器及 Ingress 服務進行送入存取。

<dl>
  <dt>NodePort 服務</dt>
  <dd>開啟您在將服務部署至所有工作者節點的公用 IP 位址時所配置的埠，以容許接收資料流量。若要尋找埠，請執行 `kubectl get svc`。該埠在 20000-32000 的範圍內。<dd>
  <dt>LoadBalancer 服務</dt>
  <dd>開啟您在將服務部署至負載平衡器服務的公用 IP 位址時所配置的埠。</dd>
  <dt>Ingress</dt>
  <dd>針對 Ingress 應用程式負載平衡器的 IP 位址，開啟埠 80（適用於 HTTP）或埠 443（適用於 HTTPS）。</dd>
</dl>

<br />


## 使用 Strongswan IPSec VPN 服務 Helm 圖表設定 VPN 連線功能
{: #vpn}

VPN 連線功能可讓您將 Kubernetes 叢集中的應用程式安全地連接至內部部署網路。您也可以將叢集外部的應用程式連接至叢集內部執行的應用程式。若要設定 VPN 連線功能，您可以使用「Helm 圖表」，在 Kubernetes Pod 內部配置及部署 [Strongswan IPSec VPN 服務 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://www.strongswan.org/)。接著會透過此 Pod 遞送所有 VPN 資料流量。如需用來設定 Strongswan 圖表之 Helm 指令的相關資訊，請參閱 [Helm 文件 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://docs.helm.sh/helm/)。

開始之前：

- [建立標準叢集。
](cs_cluster.html#cs_cluster_cli)
- [如果您要使用現有叢集，請將它更新至 1.7.4 版或更新版本。](cs_cluster.html#cs_cluster_update)
- 叢集必須至少有一個可用的公用「負載平衡器」IP 位址。
- [將 Kubernetes CLI 的目標設為叢集](cs_cli_install.html#cs_cli_configure)。

若要使用 Strongswan 設定 VPN 連線功能，請執行下列動作：

1. 如果尚未予以啟用，請安裝及起始設定叢集的 Helm。

    1. [安裝 Helm CLI ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://docs.helm.sh/using_helm/#installing-helm)。

    2. 起始設定 Helm，並安裝 `tiller`。

        ```
        helm init
        ```
        {: pre}

    3. 驗證 `tiller-deploy` Pod 在叢集中的狀態為 `Running`。

        ```
        kubectl get pods -n kube-system -l app=helm
        ```
        {: pre}

        輸出範例：

        ```
        NAME                            READY     STATUS    RESTARTS   AGE
        tiller-deploy-352283156-nzbcm   1/1       Running   0          10m
        ```
        {: screen}

    4. 將 {{site.data.keyword.containershort_notm}} Helm 儲存庫新增至 Helm 實例。

        ```
        helm repo add bluemix  https://registry.bluemix.net/helm
        ```
        {: pre}

    5. 驗證 Strongswan 圖表列在 Helm 儲存庫中。

        ```
        helm search bluemix
        ```
        {: pre}

2. 將「Strongswan Helm 圖表」的預設配置設定儲存在本端 YAML 檔案中。

    ```
    helm inspect values bluemix/strongswan > config.yaml
    ```
    {: pre}

3. 開啟 `config.yaml` 檔案，並根據您要的 VPN 配置，對預設值進行下列變更。如果內容已設定值的選項，則它們會列在檔案中每個內容上方的註解中。**重要事項**：如果您不需要變更內容，請在它前面加上 `#` 來註銷該內容。

    <table>
    <caption>表 2. 瞭解 YAML 檔案元件</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="構想圖示"/> 瞭解 YAML 檔案元件</th>
    </thead>
    <tbody>
    <tr>
    <td><code>overRideIpsecConf</code></td>
    <td>如果您有想要使用的現有 <code>ipsec.conf</code> 檔案，請移除大括弧 (<code>{}</code>)，並在此內容之後新增檔案的內容。檔案內容必須縮排。**附註：**如果您使用自己的檔案，則不會使用 <code>ipsec</code>、<code>local</code> 及 <code>remote</code> 區段的任何值。</td>
    </tr>
    <tr>
    <td><code>overRideIpsecSecrets</code></td>
    <td>如果您有想要使用的現有 <code>ipsec.secrets</code> 檔案，請移除大括弧 (<code>{}</code>)，並在此內容之後新增檔案的內容。檔案內容必須縮排。**附註：**如果您使用自己的檔案，則不會使用 <code>preshared</code> 區段的任何值。</td>
    </tr>
    <tr>
    <td><code>ipsec.keyexchange</code></td>
    <td>如果您的內部部署 VPN 通道端點不支援 <code>ikev2</code> 作為起始設定連線的通訊協定，請將此值變更為 <code>ikev1</code>。</td>
    </tr>
    <tr>
    <td><code>ipsec.esp</code></td>
    <td>將此值變更為內部部署 VPN 通道端點用於連線的 ESP 加密/鑑別演算法清單。</td>
    </tr>
    <tr>
    <td><code>ipsec.ike</code></td>
    <td>將此值變更為內部部署 VPN 通道端點用於連線的 IKE/ISAKMP SA 加密/鑑別演算法清單。</td>
    </tr>
    <tr>
    <td><code>ipsec.auto</code></td>
    <td>如果您希望叢集起始 VPN 連線，請將此值變更為 <code>start</code>。</td>
    </tr>
    <tr>
    <td><code>local.subnet</code></td>
    <td>將此值變更為應該透過 VPN 連線公開到內部部署網路的叢集子網路 CIDR 清單。此清單可能包括下列子網路：<ul><li>Kubernetes Pod 子網路 CIDR：<code>172.30.0.0/16</code></li><li>Kubernetes 服務子網路 CIDR：<code>172.21.0.0/16</code></li><li>如果您在專用網路上具有 NodePort 服務所公開的應用程式，則為工作者節點的專用子網路 CIDR。若要尋找此值，請執行 <code>bx cs subnets | grep <xxx.yyy.zzz></code>，其中 &lt;xxx.yyy.zzz&gt; 是工作者節點專用 IP 位址的前 3 個八位元組。</li><li>如果您在專用網路上具有 LoadBalancer 服務所公開的應用程式，則為叢集的專用或使用者管理子網路 CIDR。若要尋找這些值，請執行 <code>bx cs cluster-get <cluster name> --showResources</code>。在 <b>VLANS</b> 區段中，請尋找 <b>Public</b> 值為 <code>false</code> 的 CIDR。</li></ul></td>
    </tr>
    <tr>
    <td><code>local.id</code></td>
    <td>將此值變更為 VPN 通道端點用於連線的本端 Kubernetes 叢集端的字串 ID。</td>
    </tr>
    <tr>
    <td><code>remote.gateway</code></td>
    <td>將此值變更為內部部署 VPN 閘道的公用 IP 位址。</td>
    </tr>
    <td><code>remote.subnet</code></td>
    <td>將此值變更為容許 Kubernetes 叢集存取的內部部署專用子網路 CIDR 清單。</td>
    </tr>
    <tr>
    <td><code>remote.id</code></td>
    <td>將此值變更為 VPN 通道端點用於連線的遠端內部部署端的字串 ID。</td>
    </tr>
    <tr>
    <td><code>preshared.secret</code></td>
    <td>將此值變更為內部部署 VPN 通道端點閘道用於連線的預先共用密碼。</td>
    </tr>
    </tbody></table>

4. 儲存已更新的 `config.yaml` 檔案。

5. 使用已更新的 `config.yaml` 檔案，將「Helm 圖表」安裝至叢集。已更新的內容會儲存在圖表的配置對映中。

    ```
    helm install -f config.yaml --namespace=kube-system --name=vpn bluemix/strongswan
    ```
    {: pre}

6. 檢查圖表部署狀態。圖表備妥時，輸出頂端附近的 **STATUS** 欄位值為 `DEPLOYED`。

    ```
    helm status vpn
    ```
    {: pre}

7. 部署圖表之後，請驗證已使用 `config.yaml` 檔案中的已更新設定。

    ```
    helm get values vpn
    ```
    {: pre}

8. 測試新的 VPN 連線功能。
    1. 如果內部部署閘道上的 VPN 為非作用中，請啟動 VPN。

    2. 設定 `STRONGSWAN_POD` 環境變數。

        ```
        export STRONGSWAN_POD=$(kubectl get pod -n kube-system -l app=strongswan,release=vpn -o jsonpath='{ .items[0].metadata.name }')
        ```
        {: pre}

    3. 檢查 VPN 的狀態。`ESTABLISHED` 狀態表示 VPN 連線成功。

        ```
        kubectl exec -n kube-system  $STRONGSWAN_POD -- ipsec status
        ```
        {: pre}

        輸出範例：
        ```
        Security Associations (1 up, 0 connecting):
            k8s-conn[1]: ESTABLISHED 17 minutes ago, 172.30.244.42[ibm-cloud]...192.168.253.253[on-prem]
            k8s-conn{2}:  INSTALLED, TUNNEL, reqid 12, ESP in UDP SPIs: c78cb6b1_i c5d0d1c3_o
            k8s-conn{2}:   172.21.0.0/16 172.30.0.0/16 === 10.91.152.128/26
        ```
        {: screen}

        **附註**：
          - 第一次使用此「Helm 圖表」時，VPN 極不可能有 `ESTABLISHED` 狀態。您可能需要檢查內部部署 VPN 端點設定，並回到步驟 3 數次變更 `config.yaml` 檔案，連線才會成功。
          - 如果 VPN Pod 處於 `ERROR` 狀態，或持續損毀並重新啟動，則可能是圖表配置對映中 `ipsec.conf` 設定的參數驗證所造成。若要查看是否為此狀況，請執行 `kubectl logs -n kube-system $STRONGSWAN_POD`，檢查 Strongswan Pod 日誌中是否有任何驗證錯誤。如果發生驗證錯誤，請執行 `helm delete --purge vpn`，並回到步驟 3 以修正 `config.yaml` 檔案中不正確的值，然後重複步驟 4 到 8。如果叢集具有大量工作者節點，您也可以使用 `helm upgrade` 更快速地套用您的變更，而不是執行 `helm delete` 及 `helm install`。

    4. VPN 的狀態為 `ESTABLISHED` 之後，請使用 `ping` 來測試連線。下列範例會將來自 Kubernetes 叢集中 VPN Pod 的 ping 傳送至內部部署 VPN 閘道的專用 IP 位址。請確定已在配置檔中指定正確的 `remote.subnet` 及 `local.subnet`，而且本端子網路清單包括您要從中傳送 ping 的來源 IP 位址。

        ```
        kubectl exec -n kube-system  $STRONGSWAN_POD -- ping -c 3  <on-prem_gateway_private_IP>
        ```
        {: pre}

若要停用 Strongswan IPSec VPN 服務，請執行下列動作：

1. 刪除「Helm 圖表」。

    ```
    helm delete --purge vpn
    ```
    {: pre}

<br />


## 網路原則
{: #cs_security_network_policies}

每個 Kubernetes 叢集都會設定稱為 Calico 的網路外掛程式。已設定預設的網路原則來保護每個工作者節點的公用網路介面。當您有獨特的安全需求時，可以使用 Calico 及原生 Kubernetes 功能來配置叢集的其他網路原則。這些網路原則指定您要容許或封鎖的與叢集中 Pod 之間往來的網路資料流量。
{: shortdesc}

您可以選擇 Calico 與原生 Kubernetes 功能，以建立叢集的網路原則。您可以使用 Kubernetes 網路原則開始，但若需要更健全的功能，請使用 Calico 網路原則。

<ul>
  <li>[Kubernetes 網路原則 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://kubernetes.io/docs/concepts/services-networking/network-policies/)：提供部分基本選項，例如，指定可以彼此通訊的 Pod。對於通訊協定及埠，可以容許或封鎖送入的網路資料流量。可以根據正在嘗試連接至其他 Pod 之 Pod 的標籤及 Kubernetes 名稱空間來過濾此資料流量。</br>您可以使用 `kubectl` 指令或 Kubernetes API 來套用這些原則。這些原則在套用時會轉換為 Calico 網路原則，而 Calico 會強制執行這些原則。</li>
  <li>[Calico 網路原則 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](http://docs.projectcalico.org/v2.4/getting-started/kubernetes/tutorials/advanced-policy)：這些原則是 Kubernetes 網路原則的超集，並且使用下列特性來加強原生 Kubernetes 功能。
</li>
    <ul><ul><li>容許或封鎖特定網路介面的網路資料流量，而不只是 Kubernetes Pod 資料流量。</li>
    <li>容許或封鎖送入 (Ingress) 及送出 (Egress) 的網路資料流量。</li>
    <li>[封鎖對 LoadBalancer 或 NodePort Kubernetes 服務的送入 (ingress) 資料流量](#cs_block_ingress)。</li>
    <li>容許或封鎖根據來源或目的地 IP 位址或 CIDR 的資料流量。</li></ul></ul></br>

您可以使用 `calicoctl` 指令來套用這些原則。Calico 透過在 Kubernetes 工作者節點上設定 Linux iptables 規則，來強制執行這些原則（包括任何會轉換為 Calico 原則的 Kubernetes 網路原則）。iptables 規則作為工作者節點的防火牆，以定義網路資料流量必須符合才能轉遞至目標資源的特徵。</ul>


### 預設原則配置
{: #concept_nq1_2rn_4z}

建立叢集時，會自動設定每一個工作者節點的公用網路介面的預設網路原則，以限制工作者節點來自公用網際網路的送入資料流量。這些原則不會影響 Pod 對 Pod 的資料流量，並且設定以容許存取 Kubernetes NodePort、負載平衡器及 Ingress 服務。

預設原則不會直接套用至 Pod：它們是使用 Calico 主機端點套用至工作者節點的公用網路介面。在 Calico 中建立主機端點時，會封鎖與該工作者節點的網路介面之間往來的所有資料流量，除非原則容許該資料流量。

**重要事項：**除非您完全瞭解原則，並且知道您不需要原則所容許的資料流量，否則請不要移除套用至主機端點的原則。


 <table summary="表格中的第一列跨這兩個直欄。其餘的列應該從左到右閱讀，第一欄為伺服器位置，第二欄則為要符合的 IP 位址。">
  <caption>表 3. 每一個叢集的預設原則</caption>
  <thead>
  <th colspan=2><img src="images/idea.png" alt="構想圖示"/> 每一個叢集的預設原則</th>
  </thead>
  <tbody>
    <tr>
      <td><code>allow-all-outbound</code></td>
      <td>容許所有出埠資料流量。</td>
    </tr>
    <tr>
      <td><code>allow-bixfix-port</code></td>
      <td>容許埠 52311 上對 bigfix 應用程式的送入資料流量，以容許必要的工作者節點更新。</td>
    </tr>
    <tr>
      <td><code>allow-icmp</code></td>
      <td>容許送入的 icmp 封包 (ping)。</td>
     </tr>
    <tr>
      <td><code>allow-node-port-dnat</code></td>
      <td>容許 NodePort、負載平衡器及 Ingress 服務將公開至 pod 的送入 NodePort、負載平衡器及 Ingress 服務的資料流量。請注意，不需要指定這些服務在公用介面上公開的埠，因為 Kubernetes 會使用目的地網址轉譯 (DNAT) 將這些服務要求轉遞至正確的 Pod。該轉遞是在 iptables 套用主機端點原則之前進行。</td>
   </tr>
   <tr>
      <td><code>allow-sys-mgmt</code></td>
      <td>容許用來管理工作者節點之特定 IBM Cloud 基礎架構 (SoftLayer) 系統的送入連線。</td>
   </tr>
   <tr>
    <td><code>allow-vrrp</code></td>
    <td>容許 vrrp 封包，用來監視及移動工作者節點之間的虛擬 IP 位址。</td>
   </tr>
  </tbody>
</table>


### 新增網路原則
{: #adding_network_policies}

在大部分情況下，不需要變更預設原則。只有進階情境會有可能需要變更。如果您發現必須進行變更，請安裝 Calico CLI，並建立自己的網路原則。

開始之前：

1.  [安裝 {{site.data.keyword.containershort_notm}} 及 Kubernetes CLI。](cs_cli_install.html#cs_cli_install)
2.  [建立精簡或標準叢集。](cs_cluster.html#cs_cluster_ui)
3.  [將 Kubernetes CLI 的目標設為叢集](cs_cli_install.html#cs_cli_configure)。請在 `bx cs cluster-config` 指令包含 `--admin` 選項，這用來下載憑證及許可權檔案。此下載還包括「超級使用者」角色的金鑰，您需要此金鑰才能執行 Calico 指令。

  ```
  bx cs cluster-config <cluster_name> --admin
  ```
  {: pre}

  **附註**：支援 Calico CLI 1.6.1 版。

若要新增網路原則，請執行下列動作：
1.  安裝 Calico CLI。
    1.  [下載 Calico CLI ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://github.com/projectcalico/calicoctl/releases/tag/v1.6.1)。

        **提示：**如果您使用的是 Windows，請將 Calico CLI 安裝在與 {{site.data.keyword.Bluemix_notm}} CLI 相同的目錄中。當您稍後執行指令時，此設定可為您省去一些檔案路徑變更。

    2.  若為 OSX 及 Linux 使用者，請完成下列步驟。
        1.  將執行檔移至 /usr/local/bin 目錄。
            -   Linux：

              ```
              mv /<path_to_file>/calicoctl /usr/local/bin/calicoctl
              ```
              {: pre}

            -   OS X：

              ```
              mv /<path_to_file>/calicoctl-darwin-amd64 /usr/local/bin/calicoctl
              ```
              {: pre}

        2.  使檔案成為可執行檔。

            ```
            chmod +x /usr/local/bin/calicoctl
            ```
            {: pre}

    3.  檢查 Calico CLI 用戶端版本，驗證已適當地執行 `calico` 指令。

        ```
        calicoctl version
        ```
        {: pre}

2.  配置 Calico CLI。

    1.  若為 Linux 及 OS X，請建立 `/etc/calico` 目錄。若為 Windows，任何目錄皆可使用。

      ```
      sudo mkdir -p /etc/calico/
      ```
      {: pre}

    2.  建立 `calicoctl.cfg` 檔案。
        -   Linux 及 OS X：

          ```
          sudo vi /etc/calico/calicoctl.cfg
          ```
          {: pre}

        -   Windows：使用文字編輯器建立檔案。

    3.  在 <code>calicoctl.cfg</code> 檔案中，輸入下列資訊。

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

        1.  擷取 `<ETCD_URL>`。如果這個指令失敗，且出現 `calico-config` 錯誤，請參閱這個[疑難排解主題](cs_troubleshoot.html#cs_calico_fails)。

          -   Linux 及 OS X：

              ```
              kubectl get cm -n kube-system calico-config -o yaml | grep "etcd_endpoints:" | awk '{ print $2 }'
              ```
              {: pre}

          -   輸出範例：

              ```
              https://169.1.1.1:30001
              ```
              {: screen}

          -   Windows：<ol>
            <li>從配置對映取得 calico 配置值。</br><pre class="codeblock"><code>kubectl get cm -n kube-system calico-config -o yaml</code></pre></br>
            <li>在 `data` 區段中，找到 etcd_endpoints 值。範例：<code>https://169.1.1.1:30001</code>
            </ol>

        2.  擷取 `<CERTS_DIR>`，這是將 Kubernetes 憑證下載至其中的目錄。

            -   Linux 及 OS X：

              ```
              dirname $KUBECONFIG
              ```
              {: pre}

                輸出範例：

              ```
              /home/sysadmin/.bluemix/plugins/container-service/clusters/<cluster_name>-admin/
              ```
              {: screen}

            -   Windows：

              ```
              ECHO %KUBECONFIG%
              ```
              {: pre}

                輸出範例：

              ```
              C:/Users/<user>/.bluemix/plugins/container-service/<cluster_name>-admin/kube-config-prod-<location>-<cluster_name>.yml
              ```
              {: screen}

            **附註**：若要取得目錄路徑，請移除輸出結尾中的檔名 `kube-config-prod-<location>-<cluster_name>.yml`。

        3.  擷取 <code>ca-*pem_file<code>。

            -   Linux 及 OS X：

              ```
              ls `dirname $KUBECONFIG` | grep "ca-"
              ```
              {: pre}

            -   Windows：<ol><li>開啟您在最後一個步驟中擷取的目錄。</br><pre class="codeblock"><code>C:\Users\<user>\.bluemix\plugins\container-service\&lt;cluster_name&gt;-admin\</code></pre>
              <li> 找出 <code>ca-*pem_file</code> 檔案。</ol>

        4.  驗證 Calico 配置正確運作。

            -   Linux 及 OS X：

              ```
              calicoctl get nodes
              ```
              {: pre}

            -   Windows：

              ```
              calicoctl get nodes --config=<path_to_>/calicoctl.cfg
              ```
              {: pre}

              輸出：

              ```
              NAME
              kube-dal10-crc21191ee3997497ca90c8173bbdaf560-w1.cloud.ibm
              kube-dal10-crc21191ee3997497ca90c8173bbdaf560-w2.cloud.ibm
              kube-dal10-crc21191ee3997497ca90c8173bbdaf560-w3.cloud.ibm
              ```
              {: screen}

3.  檢查現有網路原則。

    -   檢視 Calico 主機端點。

      ```
      calicoctl get hostendpoint -o yaml
      ```
      {: pre}

    -   檢視已為叢集建立的所有 Calico 及 Kubernetes 網路原則。這份清單包括可能尚未套用至任何 Pod 或主機的原則。若要強制執行網路原則，則必須找到符合 Calico 網路原則中所定義之選取器的 Kubernetes 資源。

      ```
      calicoctl get policy -o wide
      ```
      {: pre}

    -   檢視網路原則的詳細資料。

      ```
      calicoctl get policy -o yaml <policy_name>
      ```
      {: pre}

    -   檢視叢集的所有網路原則的詳細資料。

      ```
      calicoctl get policy -o yaml
      ```
      {: pre}

4.  建立要容許或封鎖資料流量的 Calico 網路原則。

    1.  建立配置 Script (.yaml)，以定義 [Calico 網路原則 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](http://docs.projectcalico.org/v2.1/reference/calicoctl/resources/policy)。這些配置檔包含選取器，其說明這些原則適用的 Pod、名稱空間或主機。請參閱這些 [Calico 原則範例 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](http://docs.projectcalico.org/v2.0/getting-started/kubernetes/tutorials/advanced-policy)，以協助您建立自己的原則。

    2.  將原則套用至叢集。
        -   Linux 及 OS X：

          ```
          calicoctl apply -f <policy_file_name.yaml>
          ```
          {: pre}

        -   Windows：

          ```
          calicoctl apply -f <path_to_>/<policy_file_name.yaml> --config=<path_to_>/calicoctl.cfg
          ```
          {: pre}

### 封鎖對 LoadBalancer 或 NodePort 服務的送入資料流量
{: #cs_block_ingress}

依預設，Kubernetes `NodePort` 及 `LoadBalancer` 服務的設計是要讓您的應用程式能夠在所有公用和專用叢集介面上使用。不過，您可以根據資料流量來源或目的地，封鎖對您服務的送入資料流量。若要封鎖資料流量，請建立 Calico `preDNAT` 網路原則。

Kubernetes LoadBalancer 服務也是 NodePort 服務。LoadBalancer 服務可讓您的應用程式透過負載平衡器 IP 位址及埠提供使用，並讓您的應用程式可透過服務的節點埠提供使用。叢集中每個節點的每個 IP 位址（公開和專用）上都可以存取節點埠。

叢集管理者可以使用 Calico `preDNAT` 網路原則來封鎖：

  - 對 NodePort 服務的資料流量。容許 LoadBalancer 服務的資料流量。
  - 根據來源位址或 CIDR 的資料流量。

Calico `preDNAT` 網路原則的一些常見用途：

  - 封鎖對專用 LoadBalancer 服務之公用節點埠的資料流量。
  - 封鎖對執行[邊緣工作者節點](#cs_edge)之叢集上公用節點埠的資料流量。封鎖節點埠可確保邊緣工作者節點是處理送入資料流量的唯一工作者節點。

`preDNAT` 網路原則很有用，因為預設的 Kubernetes 及 Calico 原則很難套用來保護 Kubernetes NodePort 和 LoadBalancer 服務，這是針對這些服務產生之 DNAT iptables 規則的緣故。

Calico `preDNAT` 網路原則會根據 [Calico 網路原則資源 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://docs.projectcalico.org/v2.4/reference/calicoctl/resources/policy)產生 iptables 規則。

1. 針對 Kubernetes 服務的進入存取，定義 Calico `preDNAT` 網路原則。

  封鎖所有節點埠的範例：

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

2. 套用 Calico preDNAT 網路原則。大約需要 1 分鐘，才能在整個叢集中套用原則變更。

  ```
  calicoctl apply -f deny-kube-node-port-services.yaml
  ```
  {: pre}

<br />



## 限制送至邊緣工作者節點的網路資料流量
{: #cs_edge}

將 `dedicated=edge` 標籤新增至叢集中兩個以上的工作者節點，以確保 Ingress 及負載平衡器只會部署至那些工作者節點。

邊緣工作者節點可以藉由容許較少的工作者節點可在外部進行存取，以及隔離網路工作負載，來增進叢集的安全。僅在標示這些工作者節點可以進行網路連線時，其他工作負載才無法取用工作者節點的記憶體，並干擾網路連線。

開始之前：

- [建立標準叢集。
](cs_cluster.html#cs_cluster_cli)
- 確保您的叢集至少具有一個公用 VLAN。僅具有專用 VLAN 的叢集無法使用邊緣工作者節點。
- [將 Kubernetes CLI 的目標設為叢集](cs_cli_install.html#cs_cli_configure)。


1. 列出叢集中的所有工作者節點。請使用來自 **NAME** 直欄的專用 IP 位址來識別節點。至少選取兩個工作者節點作為邊緣工作者節點。使用兩個以上的工作者節點可提高網路資源的可用性。

  ```
  kubectl get nodes -L publicVLAN,privateVLAN,dedicated
  ```
  {: pre}

2. 使用 `dedicated=edge` 來標示工作者節點。在使用 `dedicated=edge` 標示工作者節點之後，所有後續的 Ingress 及負載平衡器都會部署至邊緣工作者節點。

  ```
  kubectl label nodes <node_name> <node_name2> dedicated=edge
  ```
  {: pre}

3. 擷取叢集中所有現有的負載平衡器服務。

  ```
  kubectl get services --all-namespaces -o jsonpath='{range .items[*]}kubectl get service -n {.metadata.namespace} {.metadata.name} -o yaml | kubectl apply -f - :{.spec.type},{end}' | tr "," "\n" | grep "LoadBalancer" | cut -d':' -f1
  ```
  {: pre}

  輸出：

  ```
  kubectl get service -n <namespace> <name> -o yaml | kubectl apply -f
  ```
  {: screen}

4. 使用來自前一個步驟的輸出，複製並貼到每一個 `kubectl get service` 指令行。此指令會將負載平衡器重新部署至邊緣工作者節點。只有公用負載平衡器需要重新部署。

  輸出：

  ```
  service "<name>" configured
  ```
  {: screen}

您已使用 `dedicated=edge` 來標示工作者節點，並已將所有現有的負載平衡器及 Ingress 重新部署至邊緣工作者節點。接下來，請避免其他[工作負載在邊緣工作者節點上執行](#cs_edge_workloads)，以及[封鎖對工作者節點上節點埠的入埠資料流量](#cs_block_ingress)。

### 避免工作負載在邊緣工作者節點上執行
{: #cs_edge_workloads}

邊緣工作者節點的其中一個好處是，這些工作者節點可以指定為僅執行網路服務。使用 `dedicated=edge` 容錯，表示所有負載平衡器及 Ingress 服務都只會部署至已標示的工作者節點。不過，為了避免其他工作負載在邊緣工作者節點上執行，以及取用工作者節點資源，您必須使用 [Kubernetes 污點 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)。

1. 列出所有具有 `edge` 標籤的工作者節點。

  ```
  kubectl get nodes -L publicVLAN,privateVLAN,dedicated -l dedicated=edge
  ```
  {: pre}

2. 將污點套用至每一個工作者節點，以避免 Pod 在工作者節點上執行，並從工作者節點中移除沒有 `edge` 標籤的 Pod。移除的 Pod 會重新部署在有容量的其他工作者節點上。

  ```
  kubectl taint node <node_name> dedicated=edge:NoSchedule dedicated=edge:NoExecute
  ```

現在，僅具有 `dedicated=edge` 容錯的 Pod 才會部署至您的邊緣工作者節點。
