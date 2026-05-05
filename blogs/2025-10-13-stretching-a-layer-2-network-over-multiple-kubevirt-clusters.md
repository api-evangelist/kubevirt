---
title: "Stretching a Layer 2 network over multiple KubeVirt clusters"
url: "https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html"
date: "2025-10-13T00:00:00+00:00"
author: "Miguel Duarte Barroso"
feed_url: "https://kubevirt.io//feed.xml"
---
<h2 id="introduction">Introduction</h2>
<p>KubeVirt enables you to run virtual machines (VMs) within Kubernetes clusters,
but networking VMs across multiple clusters presents significant challenges.
Current KubeVirt networking relies on cluster-local solutions, which cannot
extend Layer 2 broadcast domains beyond cluster boundaries. This limitation
forces applications that require L2 connectivity to either remain within a
single cluster or undergo complex network reconfiguration when distributed
across clusters.</p>

<p>Integrating with EVPN addresses this fundamental limitation in distributed
KubeVirt deployments: the inability to maintain L2 adjacency between VMs
running on different clusters. By leveraging EVPN’s BGP-based control plane and
advanced MAC/IP advertisement mechanisms, we can now stretch Layer 2 broadcast
domains across geographically distributed KubeVirt clusters, creating a unified
network fabric that treats multiple clusters as a single, cohesive
infrastructure.</p>

<h3 id="why-stretch-l2-networks-across-different-clusters-">Why Stretch L2 Networks across different clusters ?</h3>
<p>The ability to extend L2 domains between KubeVirt clusters unlocks several
critical capabilities that were previously difficult to achieve.
Traditional cluster networking creates isolation boundaries that, while
beneficial for security and resource management, can become barriers when
applications require tight coupling or when operational requirements demand
flexibility in workload placement.</p>

<p>All in all, stretching an L2 domain across cluster boundaries enables use cases
that are fundamental to infrastructure reliability and flexibility, which include:</p>
<ul>
  <li><strong>Cross Cluster Live Migration:</strong> VMs must migrate between clusters without
requiring IP address changes, DNS updates, or application reconfiguration. This
capability is essential for disaster recovery scenarios where VMs must failover
to geographically distant clusters while still maintaining their network
identity and established connections.</li>
  <li><strong>Legacy enterprise applications availability:</strong> many mission-critical
workloads were designed with assumptions about L2 adjacency, such as database
clusters requiring heartbeat mechanisms over broadcast domains, application
servers expecting multicast discovery, or network-attached storage systems
relying on L2 protocols.</li>
  <li><strong>Resource optimization and capacity planning:</strong> organizations can distribute
VM workloads based on compute availability, cost considerations, or compliance
requirements while maintaining the network simplicity that applications expect.
This flexibility becomes particularly valuable in hybrid cloud scenarios where
workloads may need to seamlessly span on-premises KubeVirt clusters and
cloud-hosted instances.</li>
</ul>

<p>This is where the power of EVPN comes into play: by integrating EVPN into the
KubeVirt ecosystem, we can create a sophisticated L2 overlay. Think of it as a
virtual network fabric that stretches across your data centers or cloud
regions, enabling the workloads running in KubeVirt clusters to attach to a
single, unified L2 domain.</p>

<p>In this post, we’ll dive into how this powerful combination works and how it
unlocks true application mobility for your virtualized workloads on Kubernetes.</p>

<h2 id="prerequisites">Prerequisites</h2>
<p>The list below is required to run this demo. This will enable you to run
multiple Kubernetes clusters in your own laptop, interconnected by EVPN using
<a href="https://openperouter.github.io/">openperouter</a>.</p>
<ul>
  <li>container runtime - docker - installed in your system</li>
  <li>git</li>
  <li>make</li>
</ul>

<h2 id="the-testbed">The testbed</h2>
<p>The testbed will be implemented using a physical network deployed in
leaf/spine topology, which is a common two-layer network architecture used in
data centers. It consists of leaf switches that connect to end devices, and
spine switches that interconnect all leaf switches. This way, workloads will
always be (at most) two hops away from one another.</p>

<p align="center">
  <img alt="The testbed" src="https://kubevirt.io/assets/2025-10-13-evpn-integration/01-evpn-integration-testbed.png" width="100%" />
</p>

<p>The diagram highlights the autonomous system (AS) numbers each of the
components will use.</p>

<p>We can infer from the AS numbers provided above that the testbed will feature
eBGP configuration, thus providing routing between different autonomous
systems.</p>

<p>We will setup the testbed using <a href="https://containerlab.dev/">containerlab</a>, and
the Kubernetes clusters are deployed using <a href="https://kind.sigs.k8s.io/">KinD</a>.
The BGP speakers (routers) in each leaf are implemented using
<a href="https://frrouting.org/">FRR</a>.</p>

<h3 id="spawning-the-testbed-on-your-laptop">Spawning the testbed on your laptop</h3>
<p>To spawn the tested in your laptop, you should clone the openperouter repo.</p>
<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code>git clone https://github.com/openperouter/openperouter.git
git checkout c9d591a
<span class="nb">cd </span>openperouter
</code></pre></div></div>

<p>Assuming you have all the <a href="https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html#prerequisites">requirements</a> installed in your
laptop, all you need to do is build the router component, and execute the
<code class="language-plaintext highlighter-rouge">deploy-multi</code> make target. Then, you should be ready to go!</p>
<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code>sysctl <span class="nt">-w</span> fs.inotify.max_user_instances<span class="o">=</span>1024 <span class="c"># might need sudo</span>
make docker-build <span class="o">&amp;&amp;</span> make deploy-multi
</code></pre></div></div>

<p>After running this make target, you should have the testbed deployed as shown
in the testbed’s <a href="https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html#the-testbed">diagram</a>. One thing is missing though: the
autonomous systems in the kind clusters are not configured yet! This will be
configured in the <a href="https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html#configuring-the-kubevirt-clusters">next section</a>.</p>

<p>The kubeconfigs to connect to each cluster can be found in <code class="language-plaintext highlighter-rouge">openperouter</code>’s
<code class="language-plaintext highlighter-rouge">bin</code> directory:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">ls</span> <span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-<span class="k">*</span>
/root/github/openperouter/bin/kubeconfig-pe-kind-a  /root/github/openperouter/bin/kubeconfig-pe-kind-b
</code></pre></div></div>

<p>Before moving to the configuration section, let’s install KubeVirt in both
clusters:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="k">for </span>kubeconfig <span class="k">in</span> <span class="si">$(</span><span class="nb">ls </span>bin/kubeconfig-<span class="k">*</span><span class="si">)</span><span class="p">;</span> <span class="k">do
    </span><span class="nb">echo</span> <span class="s2">"Installing KubeVirt in cluster using KUBECONFIG=</span><span class="nv">$kubeconfig</span><span class="s2">"</span>
    <span class="nv">KUBECONFIG</span><span class="o">=</span><span class="nv">$kubeconfig</span> kubectl apply <span class="nt">-f</span> https://github.com/kubevirt/kubevirt/releases/download/v1.5.2/kubevirt-operator.yaml
    <span class="nv">KUBECONFIG</span><span class="o">=</span><span class="nv">$kubeconfig</span> kubectl apply <span class="nt">-f</span> https://github.com/kubevirt/kubevirt/releases/download/v1.5.2/kubevirt-cr.yaml
    <span class="c"># Patch KubeVirt to allow scheduling on control-planes, so we can test live migration between two nodes</span>
    <span class="nv">KUBECONFIG</span><span class="o">=</span><span class="nv">$kubeconfig</span> kubectl patch <span class="nt">-n</span> kubevirt kubevirt kubevirt <span class="nt">--type</span> merge <span class="nt">--patch</span> <span class="s1">'{"spec": {"workloads": {"nodePlacement": {"tolerations": [{"key": "node-role.kubernetes.io/control-plane", "operator": "Exists", "effect": "NoSchedule"}]}}}}'</span>
    <span class="nv">KUBECONFIG</span><span class="o">=</span><span class="nv">$kubeconfig</span> kubectl <span class="nb">wait</span> <span class="nt">--for</span><span class="o">=</span><span class="nv">condition</span><span class="o">=</span>Available kubevirt/kubevirt <span class="nt">-n</span> kubevirt <span class="nt">--timeout</span><span class="o">=</span>10m
    <span class="nb">echo</span> <span class="s2">"Finished installing KubeVirt in cluster using KUBECONFIG=</span><span class="nv">$kubeconfig</span><span class="s2">"</span>
<span class="k">done</span>
</code></pre></div></div>

<h2 id="configuring-the-kubevirt-clusters">Configuring the KubeVirt clusters</h2>
<p>As indicated in the <a href="https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html#introduction">introduction</a> section, the end goal is to
stretch a layer 2 network across both Kubernetes clusters, using EVPN. Please
refer to the image below for a simple diagram.</p>

<p align="center">
  <img alt="Layer 2 network stretched across both clusters" src="https://kubevirt.io/assets/2025-10-13-evpn-integration/02-stretched-l2-evpn.png" width="100%" />
</p>

<p>In order to stretch an L2 overlay across both cluster we need to:</p>
<ul>
  <li>configure the underlay network</li>
  <li>configure the EVPN VXLAN VNI</li>
</ul>

<p>We will rely on <a href="https://openperouter.github.io/">openperouter</a> for both of
these.</p>

<p>Let’s start with the underlay network, in which we will connect the Kubernetes
clusters to each cluster’s top of rack BGP/EVPN speaker.</p>

<h3 id="configuring-the-underlay-network">Configuring the underlay network</h3>
<p>The first thing we need to do is to finish setting up the testbed by
peering our two Kubernetes clusters with the BGP/EVPN speaker in each cluster’s
top of rack:</p>
<ul>
  <li><code class="language-plaintext highlighter-rouge">kindleaf-a</code> for cluster-a</li>
  <li><code class="language-plaintext highlighter-rouge">kindleaf-b</code> for cluster-b</li>
</ul>

<p>This will require you to specify the expected AS numbers, to define the VXLAN
tunnel endpoint addresses, and also specify which node interface will be used
to connect to external routers.</p>

<p>For that,
you will need to provision the following CRs:</p>

<ul>
  <li>Cluster A.</li>
</ul>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: Underlay
metadata:
  name: underlay
  namespace: openperouter-system
spec:
  asn: 64514
  evpn:
    vtepcidr:  100.65.0.0/24
  nics:
    - toswitch
  neighbors:
    - asn: 64512
      address: 192.168.11.2
</span><span class="no">EOF
</span></code></pre></div></div>

<ul>
  <li>Cluster B.</li>
</ul>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: Underlay
metadata:
  name: underlay
  namespace: openperouter-system
spec:
  asn: 64518
  evpn:
    vtepcidr: 100.65.1.0/24
  routeridcidr: 10.0.1.0/24
  nics:
    - toswitch
  neighbors:
    - asn: 64516
      address: 192.168.12.2
</span><span class="no">EOF
</span></code></pre></div></div>

<h3 id="configuring-the-evpn-vni">Configuring the EVPN VNI</h3>
<p>Once we have configured both Kubernetes cluster’s peering with the external
routers in <code class="language-plaintext highlighter-rouge">kindleaf-a</code> and <code class="language-plaintext highlighter-rouge">kindleaf-b</code>, we can now focus on defining the
<code class="language-plaintext highlighter-rouge">layer2</code> EVPN. For that, we will use openperouter’s <code class="language-plaintext highlighter-rouge">L2VNI</code> CRD.</p>

<p>Execute the following commands to provision the <code class="language-plaintext highlighter-rouge">L2VNI</code> in both clusters:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># provision L2VNI in cluster: pe-kind-a</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: L2VNI
metadata:
  name: layer2
  namespace: openperouter-system
spec:
  hostmaster:
    autocreate: true
    type: bridge
  l2gatewayip: 192.170.1.1/24
  vni: 110
  vrf: red
</span><span class="no">EOF

</span><span class="c"># provision L2VNI in cluster: pe-kind-b</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: L2VNI
metadata:
  name: layer2
  namespace: openperouter-system
spec:
  hostmaster:
    autocreate: true
    type: bridge
  l2gatewayip: 192.170.1.1/24
  vni: 110
  vrf: red
</span><span class="no">EOF
</span></code></pre></div></div>

<p>After this step, we will have created an L2 overlay network on top of the
network fabric. We now need to enable it to be plumbed to the workloads.
Execute the commands below to provision a network attachment definition in both
clusters:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># provision NAD in cluster: pe-kind-a</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: evpn
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "name": "evpn",
      "type": "bridge",
      "bridge": "br-hs-110",
      "macspoofchk": false,
      "disableContainerInterface": true
    }
</span><span class="no">EOF

</span><span class="c"># provision NAD in cluster: pe-kind-b</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: evpn
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "name": "evpn",
      "type": "bridge",
      "bridge": "br-hs-110",
      "macspoofchk": false,
      "disableContainerInterface": true
    }
</span><span class="no">EOF
</span></code></pre></div></div>

<p>Now that we have set up networking for the workloads, we can proceed with
actually instantiating the VMs which will attach to this network overlay.</p>

<h3 id="provisioning-and-running-the-vm-workloads">Provisioning and running the VM workloads</h3>

<p>You will have one VM running in cluster A (vm-1), and another VM running in
cluster B (vm-2).</p>

<p>The VMs will each have one network interface, attached to the layer2 overlay.
The VMs are using bridge binding, and they attach to the overlay using bridge-cni.
Both VMs have static IPs, configured over cloud-init. They are:</p>

<table>
  <thead>
    <tr>
      <th>VM name</th>
      <th>Cluster</th>
      <th>IP address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>vm-1</td>
      <td>pe-kind-a</td>
      <td>192.170.1.3</td>
    </tr>
    <tr>
      <td>vm-2</td>
      <td>pe-kind-b</td>
      <td>192.170.1.30</td>
    </tr>
  </tbody>
</table>

<p>To provision these, follow these steps:</p>

<ol>
  <li>Provision <code class="language-plaintext highlighter-rouge">vm-1</code> in cluster <code class="language-plaintext highlighter-rouge">pe-kind-a</code>:</li>
</ol>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-1
spec:
  runStrategy: Always
  template:
    metadata:
      labels:
        kubevirt.io/vm: vm-1
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      domain:
        devices:
          interfaces:
          - bridge: {}
            name: evpn
          disks:
          - disk:
              bus: virtio
            name: containerdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
        resources:
          requests:
            memory: 2048M
        machine:
          type: ""
      networks:
      - multus:
          networkName: evpn
        name: evpn
      terminationGracePeriodSeconds: 0
      volumes:
      - containerDisk:
          image: quay.io/kubevirt/fedora-with-test-tooling-container-disk:v1.5.2
        name: containerdisk
      - cloudInitNoCloud:
          networkData: |
            version: 2
            ethernets:
              eth0:
                addresses:
                - 192.170.1.3/24
                gateway4: 192.170.1.1
        name: cloudinitdisk
</span><span class="no">EOF
</span></code></pre></div></div>

<ol>
  <li>Provision <code class="language-plaintext highlighter-rouge">vm-2</code> in cluster <code class="language-plaintext highlighter-rouge">pe-kind-b</code>:</li>
</ol>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-2
spec:
  runStrategy: Always
  template:
    metadata:
      labels:
        kubevirt.io/vm: vm-2
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      domain:
        devices:
          interfaces:
          - bridge: {}
            name: evpn
          disks:
          - disk:
              bus: virtio
            name: containerdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
        resources:
          requests:
            memory: 2048M
        machine:
          type: ""
      networks:
      - multus:
          networkName: evpn
        name: evpn
      terminationGracePeriodSeconds: 0
      volumes:
      - containerDisk:
          image: quay.io/kubevirt/fedora-with-test-tooling-container-disk:v1.5.2
        name: containerdisk
      - cloudInitNoCloud:
          networkData: |
            version: 2
            ethernets:
              eth0:
                addresses:
                - 192.170.1.30/24
                gateway4: 192.170.1.1
        name: cloudinitdisk
</span><span class="no">EOF
</span></code></pre></div></div>

<p>We will use <code class="language-plaintext highlighter-rouge">vm-2</code> (which runs in cluster <strong>B</strong>) as the “server”, and <code class="language-plaintext highlighter-rouge">vm-1</code>
(which runs in cluster <strong>A</strong>) as the “client”; however, we first need to wait
for the VMs to become <code class="language-plaintext highlighter-rouge">Ready</code>:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span>bin/kubeconfig-pe-kind-a kubectl <span class="nb">wait </span>vm vm-1 <span class="nt">--for</span><span class="o">=</span><span class="nv">condition</span><span class="o">=</span>Ready <span class="nt">--timeout</span><span class="o">=</span>60s
<span class="nv">KUBECONFIG</span><span class="o">=</span>bin/kubeconfig-pe-kind-b kubectl <span class="nb">wait </span>vm vm-2 <span class="nt">--for</span><span class="o">=</span><span class="nv">condition</span><span class="o">=</span>Ready <span class="nt">--timeout</span><span class="o">=</span>60s
</code></pre></div></div>

<p>Now that we know the VMs are <code class="language-plaintext highlighter-rouge">Ready</code>, let’s confirm the IP address for <code class="language-plaintext highlighter-rouge">vm-2</code>,
and reach into it from the <code class="language-plaintext highlighter-rouge">vm-1</code> VM, which is available in cluster A.</p>

<div class="language-sh highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span>bin/kubeconfig-pe-kind-b kubectl get vmi vm-2 <span class="nt">-ojsonpath</span><span class="o">=</span><span class="s2">"{.status.interfaces[0].ipAddress}"</span>
192.170.1.30
</code></pre></div></div>

<p>Let’s now serve some data. We will use a toy python webserver for that, which serves some files:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="o">[</span>fedora@vm-2 ~]<span class="nv">$ </span><span class="nb">touch</span> <span class="si">$(</span><span class="nb">date</span><span class="si">)</span>
<span class="o">[</span>fedora@vm-2 ~]<span class="nv">$ </span><span class="nb">ls</span> <span class="nt">-la</span>
total 12
drwx------. 1 fedora fedora 122 Oct 13 12:08 <span class="nb">.</span>
drwxr-xr-x. 1 root   root    12 Sep 13  2024 ..
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 12:08:15
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 13
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 2025
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora  18 Jul 21  2021 .bash_logout
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora 141 Jul 21  2021 .bash_profile
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora 492 Jul 21  2021 .bashrc
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 Mon
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 Oct
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 PM
drwx------. 1 fedora fedora  30 Sep 13  2024 .ssh
<span class="nt">-rw-r--r--</span><span class="nb">.</span> 1 fedora fedora   0 Oct 13 12:08 UTC
<span class="o">[</span>fedora@vm-2 ~]<span class="nv">$ </span>python3 <span class="nt">-m</span> http.server 8090
Serving HTTP on 0.0.0.0 port 8090 <span class="o">(</span>http://0.0.0.0:8090/<span class="o">)</span> ...
</code></pre></div></div>

<p>And let’s try to access that from the VM which runs in the other cluster:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span>bin/kubeconfig-pe-kind-a virtctl console vm-1
<span class="c"># password to access the VM is fedora/fedora</span>
<span class="o">[</span>fedora@vm-1 ~]<span class="nv">$ </span>curl 192.170.1.30:8090
&lt;<span class="o">!</span>DOCTYPE HTML PUBLIC <span class="s2">"-//W3C//DTD HTML 4.01//EN"</span> <span class="s2">"http://www.w3.org/TR/html4/strict.dtd"</span><span class="o">&gt;</span>
&lt;html&gt;
&lt;<span class="nb">head</span><span class="o">&gt;</span>
&lt;meta http-equiv<span class="o">=</span><span class="s2">"Content-Type"</span> <span class="nv">content</span><span class="o">=</span><span class="s2">"text/html; charset=utf-8"</span><span class="o">&gt;</span>
&lt;title&gt;Directory listing <span class="k">for</span> /&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;Directory listing <span class="k">for</span> /&lt;/h1&gt;
&lt;hr&gt;
&lt;ul&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">".bash_logout"</span><span class="o">&gt;</span>.bash_logout&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">".bash_profile"</span><span class="o">&gt;</span>.bash_profile&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">".bashrc"</span><span class="o">&gt;</span>.bashrc&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">".ssh/"</span><span class="o">&gt;</span>.ssh/&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"12%3A08%3A15"</span><span class="o">&gt;</span>12:08:15&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"13"</span><span class="o">&gt;</span>13&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"2025"</span><span class="o">&gt;</span>2025&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"Mon"</span><span class="o">&gt;</span>Mon&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"Oct"</span><span class="o">&gt;</span>Oct&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"PM"</span><span class="o">&gt;</span>PM&lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a <span class="nv">href</span><span class="o">=</span><span class="s2">"UTC"</span><span class="o">&gt;</span>UTC&lt;/a&gt;&lt;/li&gt;
&lt;/ul&gt;
&lt;hr&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre></div></div>

<p>As you can see, the VM running in cluster A was able to successfully reach into
the VM running in cluster B.</p>

<h3 id="bonus-track-connecting-to-provider-networks-using-an-l3vni">Bonus track: connecting to provider networks using an L3VNI</h3>
<p>This extra (optional) step showcases how you can import provider network routes
into the Kubernetes clusters - essentially creating an L3 overlay - using
<code class="language-plaintext highlighter-rouge">openperouter</code>s L3VNI CRD.</p>

<p>We will use it to reach into the webserver hosted in <code class="language-plaintext highlighter-rouge">hostA</code> (attached to
<code class="language-plaintext highlighter-rouge">leafA</code> in the <a href="https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html#the-testbed">diagram</a>) from the VMs running in both clusters.
Please refer to the image below to get a better understanding of the scenario.</p>

<p align="center">
  <img alt="Wrap an L3VNI over a stretched L2 EVPN" src="https://kubevirt.io/assets/2025-10-13-evpn-integration/03-wrap-l3vni-over-stretched-l2.png" width="100%" />
</p>

<p>Since we already have configured the <code class="language-plaintext highlighter-rouge">underlay</code> in a
<a href="https://kubevirt.io//2025/Stretched-layer2-network-between-clusters.html#configuring-the-underlay-network">previous step</a>, all we need to do is to
configure the <code class="language-plaintext highlighter-rouge">L3VNI</code>; for that, provision the following CR in <strong>both</strong>
clusters:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># provision L3VNI in cluster: pe-kind-a</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: L3VNI
metadata:
  name: red
  namespace: openperouter-system
spec:
  vni: 100
  vrf: red
</span><span class="no">EOF

</span><span class="c"># provision L3VNI in cluster: pe-kind-b</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: L3VNI
metadata:
  name: red
  namespace: openperouter-system
spec:
  vni: 100
  vrf: red
</span><span class="no">EOF
</span></code></pre></div></div>

<p>This will essentially wrap the existing <code class="language-plaintext highlighter-rouge">L2VNI</code> with an L3 domain - i.e. a
separate Virtual Routing Function (VRF), whose Virtual Network Identifier (VNI)
is 100. This will enable the Kubernetes clusters to reach into services located in
the <code class="language-plaintext highlighter-rouge">red</code> VRF (which have VNI = 100). Services on <code class="language-plaintext highlighter-rouge">hostA</code> and/or <code class="language-plaintext highlighter-rouge">hostB</code> with
VNI = 200 are not accessible, since we haven’t exposed them over EVPN (using an
<code class="language-plaintext highlighter-rouge">L3VNI</code>).</p>

<p>Once we’ve provisioned the aforementioned <code class="language-plaintext highlighter-rouge">L3VNI</code>, we can now check accessing
the webserver located in the host in <code class="language-plaintext highlighter-rouge">leafA</code> - <code class="language-plaintext highlighter-rouge">clab-kind-leafA</code>.</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code>docker <span class="nb">exec </span>clab-kind-hostA_red ip <span class="nt">-4</span> addr show dev eth1
228: eth1@if227: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 9500 qdisc noqueue state UP group default  link-netnsid 1
    inet 192.168.20.2/24 scope global eth1
       valid_lft forever preferred_lft forever
</code></pre></div></div>

<p>Let’s also check the same thing for the <code class="language-plaintext highlighter-rouge">blue</code> VRF - for which we do <strong>not</strong>
have any VNI configuration.</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code>docker <span class="nb">exec </span>clab-kind-hostA_blue ip <span class="nt">-4</span> addr show dev eth1
273: eth1@if272: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 9500 qdisc noqueue state UP group default  link-netnsid 1
    inet 192.168.21.2/24 scope global eth1
       valid_lft forever preferred_lft forever
</code></pre></div></div>

<p>And let’s now access these services from the VMs we have in both clusters.</p>

<p>From <code class="language-plaintext highlighter-rouge">vm-1</code>, in cluster A:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># username/password =&gt; fedora/fedora</span>
virtctl console vm-1
Successfully connected to vm-1 console. The escape sequence is ^]

vm-1 login: fedora
Password:
<span class="o">[</span>fedora@vm-1 ~]<span class="nv">$ </span>curl 192.168.20.2:8090/clientip <span class="c"># we have access to the RED VRF</span>
192.170.1.3:35146
<span class="o">[</span>fedora@vm-1 ~]<span class="nv">$ </span>curl 192.168.21.2:8090/clientip <span class="c"># we do NOT have access to the BLUE VRF</span>
curl: <span class="o">(</span>28<span class="o">)</span> Failed to connect to 192.168.21.2 port 8090 after 128318 ms: Connection timed out
</code></pre></div></div>

<p>From <code class="language-plaintext highlighter-rouge">vm-2</code>, in cluster B:</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># username/password =&gt; fedora/fedora</span>
virtctl console vm-2
Successfully connected to vm-2 console. The escape sequence is ^]

vm-2 login: fedora
Password:
<span class="o">[</span>fedora@vm-2 ~]<span class="nv">$ </span>curl 192.168.20.2:8090/clientip <span class="c"># we have access to the RED VRF</span>
192.170.1.30:52924
<span class="o">[</span>fedora@vm-2 ~]<span class="nv">$ </span>curl 192.168.21.2:8090/clientip <span class="c"># we do NOT have access to the BLUE VRF</span>
curl: <span class="o">(</span>28<span class="o">)</span> Failed to connect to 192.168.21.2 port 8090 after 130643 ms: Connection timed out
</code></pre></div></div>

<h2 id="conclusions">Conclusions</h2>
<p>In this article we have explained EVPN and which virtualization use cases it
can provide.</p>

<p>We have also shown how the <a href="https://openperouter.github.io/">openperouter</a>
<code class="language-plaintext highlighter-rouge">L2VNI</code> CRD can be used to stretch a Layer 2 overlay across multiple Kubernetes
clusters.</p>

<p>Finally, we have also seen how <code class="language-plaintext highlighter-rouge">openperouter</code> <code class="language-plaintext highlighter-rouge">L3VNI</code> can be used to create
Layer 3 overlays, which allows the VMs running in the Kubernetes clusters to
access services in the exposed provider networks.</p>
