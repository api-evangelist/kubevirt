---
title: "Dedicated Migration Networks for Cross-Cluster Live Migration with KubeVirt and EVPN"
url: "https://kubevirt.io//2025/Dedicated-migration-network-for-cross-cluster-live-migration.html"
date: "2025-10-31T00:00:00+00:00"
author: "Miguel Duarte Barroso"
feed_url: "https://kubevirt.io//feed.xml"
---
<h2 id="introduction">Introduction</h2>

<p>In our <a href="https://kubevirt.io/2025/Stretched-layer2-network-between-clusters.html">previous post</a>,
we explored how to stretch Layer 2 networks across multiple KubeVirt clusters
using EVPN and OpenPERouter. While this enables cross-cluster connectivity, VMs
often need to move between clusters. This happens during disaster recovery,
cluster maintenance, resource optimization, or compliance requirements.</p>

<p>Cross cluster live migration moves a running VM from one cluster to another
without stopping it. This generates substantial network traffic and needs
reliable, high-bandwidth connectivity. When you use the same network for both
application traffic and migration, you risk network congestion and security
issues from mixing migration traffic with user data.</p>

<p>A dedicated migration network solves this problem. By configuring a separate
Layer 2 Virtual Network Interface (L2VNI) for migration traffic, you isolate
this critical operation from application networking, improving both security and
performance. Furthermore, the cluster/network admins’ lives are simplified by
making the dedicated migration network an overlay: instead of physically
running and maintaining new cables, configuring switches, and adding network
interfaces to each Kubernetes node (a complex and time-consuming underlay
network expansion), an L2VNI builds upon the existing physical network
infrastructure - admins can define and manage this overlay network logically,
making it a much more agile (and less disruptive) solution for dedicated
migration paths.</p>

<h2 id="why-should-you-have-a-dedicated-migration-network">Why should you have a dedicated migration network</h2>

<p>Dedicated migration networks provide several key advantages:</p>

<ul>
  <li>
    <p><strong>Traffic Isolation</strong>: Migration data flows through a separate network path,
preventing interference with application traffic and allowing for independent
network policies and monitoring.</p>
  </li>
  <li>
    <p><strong>Security Boundaries</strong>: Migration traffic can be encrypted and routed through
dedicated security zones, reducing the attack surface and enabling fine-grained
access controls.</p>
  </li>
  <li>
    <p><strong>Performance Optimization</strong>: Migration networks can be configured with
specific bandwidth allocations, MTU settings, and QoS policies optimized for
bulk data transfer.</p>
  </li>
  <li>
    <p><strong>Operational Visibility</strong>: Separate networks enable dedicated monitoring and
troubleshooting of migration operations without impacting application network
analysis.</p>
  </li>
</ul>

<h2 id="configuring-the-dedicated-migration-network">Configuring the Dedicated Migration Network</h2>

<p>Building on our previous multi-cluster setup, we’ll now add a dedicated
migration network using a separate L2VNI. This configuration assumes you
already have the base clusters and stretched L2 network from the
<a href="https://kubevirt.io/2025/Stretched-layer2-network-between-clusters.html">previous article</a>.</p>

<h3 id="prerequisites">Prerequisites</h3>

<p>Ensure you have:</p>
<ul>
  <li>The multi-cluster testbed from the previous post deployed using
<code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code></li>
  <li>KubeVirt 1.6.2 or higher installed (included in
<code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code>)</li>
  <li>Whereabouts IPAM CNI installed (included in <code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code>)</li>
  <li>The <code class="language-plaintext highlighter-rouge">DecentralizedLiveMigration</code> feature gate enabled (included in
<code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code>)</li>
</ul>

<h3 id="configuring-the-migration-l2vni">Configuring the Migration L2VNI</h3>

<p>Now we’ll create a separate L2VNI dedicated to migration traffic. Note that
we’re using VNI 666 and VRF “rouge” to distinguish this from our application
network (VNI 110, VRF “red”).</p>

<p><strong>NOTE:</strong> this dedicated migration network (implemented by this L2VNI) is
pre-provisioned when you run <code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code>.</p>

<p><strong>Cluster A Migration Network:</strong></p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: L2VNI
metadata:
  name: migration
  namespace: openperouter-system
spec:
  hostmaster:
    autocreate: true
    type: bridge
  l2gatewayip: 192.170.10.1/24
  vni: 666
  vrf: rouge
</span><span class="no">EOF
</span></code></pre></div></div>

<p><strong>Cluster B Migration Network:</strong></p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: openpe.openperouter.github.io/v1alpha1
kind: L2VNI
metadata:
  name: migration
  namespace: openperouter-system
spec:
  hostmaster:
    autocreate: true
    type: bridge
  l2gatewayip: 192.170.10.1/24
  vni: 666
  vrf: rouge
</span><span class="no">EOF
</span></code></pre></div></div>

<h3 id="creating-migration-network-attachment-definitions">Creating Migration Network Attachment Definitions</h3>

<p>Next, we create Network Attachment Definitions (NADs) for the migration
network. Note the reduced MTU of 1400 to account for VXLAN overhead:</p>

<p><strong>Cluster A Migration NAD:</strong></p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: migration-evpn
  namespace: kubevirt
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "name": "migration-evpn",
      "type": "bridge",
      "bridge": "br-hs-666",
      "mtu": 1400,
      "ipam": {
        "type": "whereabouts",
        "range": "192.170.10.0/24",
        "exclude": [
          "192.170.10.1/32",
          "192.170.10.128/25"
        ]
      }
    }
</span><span class="no">EOF
</span></code></pre></div></div>

<p><strong>Cluster B Migration NAD:</strong></p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: migration-evpn
  namespace: kubevirt
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "name": "migration-evpn",
      "type": "bridge",
      "bridge": "br-hs-666",
      "mtu": 1400,
      "ipam": {
        "type": "whereabouts",
        "range": "192.170.10.0/24",
        "exclude": [
          "192.170.10.1/32",
          "192.170.10.0/25"
        ]
      }
    }
</span><span class="no">EOF
</span></code></pre></div></div>

<p><strong>NOTE:</strong> these NADs are already pre-provisioned when you run
<code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code>.</p>

<h4 id="understanding-the-ip-range-strategy">Understanding the IP Range Strategy</h4>

<p>Both clusters define the same 192.170.10.0/24 range but use different
exclusion patterns to avoid IP conflicts:</p>

<ul>
  <li><strong>Cluster A</strong> excludes <code class="language-plaintext highlighter-rouge">192.170.10.128/25</code> (192.170.10.128 to 192.170.10.255),
giving it access to IPs 192.170.10.2 to 192.170.10.127</li>
  <li><strong>Cluster B</strong> excludes <code class="language-plaintext highlighter-rouge">192.170.10.0/25</code> (192.170.10.0 to 192.170.10.127),
giving it access to IPs 192.170.10.128 to 192.170.10.255</li>
  <li>Both exclude <code class="language-plaintext highlighter-rouge">192.170.10.1/32</code> (the gateway IP)</li>
</ul>

<p>This approach ensures that VMs in each cluster get IPs from non-overlapping
ranges while maintaining the same L2 network, allowing seamless migration
without IP conflicts or the need for IP reassignment during the migration
process.</p>

<p>Since all the prerequisites including certificate exchange are handled by
<code class="language-plaintext highlighter-rouge">make deploy-multi-cluster</code>, we can proceed directly to preparing the VM to be
migrated. All the manifests and instructions are available in
<a href="https://github.com/openperouter/openperouter/blob/main/website/content/docs/examples/evpnexamples/kubevirt-multi-cluster.md#l2-vni-as-kubevirt-dedicated-migration-network-for-cross-cluster-live-migration">OpenPERouter cross-cluster live migration examples</a>.</p>

<h2 id="cross-cluster-live-migration-in-action">Cross-Cluster Live Migration in Action</h2>

<p>Now let’s demonstrate cross-cluster live migration using our dedicated
migration network. We’ll create VMs that use both the application network
(evpn) and have an EVPN <code class="language-plaintext highlighter-rouge">L2VNI</code> as the migration network. Keep in mind that the
latter network is <strong>not</strong> plumbed into the VMs! It is used by the KubeVirt
agents (privileged components, which run in the Kubernetes nodes) to move
the migration between the different nodes (which happen to run in different
clusters).</p>

<h3 id="creating-migration-ready-vms">Creating Migration-Ready VMs</h3>

<p><strong>VM in Cluster A (Migration Source):</strong></p>

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
            macAddress: 02:03:04:05:06:07
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
          image: quay.io/kubevirt/fedora-with-test-tooling-container-disk:v1.6.2
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

<p><strong>VM in Cluster B (Migration Target):</strong></p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-1
spec:
  runStrategy: WaitAsReceiver
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
          image: quay.io/kubevirt/fedora-with-test-tooling-container-disk:v1.6.2
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

<p>As you can see, both VM definitions are the same - except the <code class="language-plaintext highlighter-rouge">runStrategy</code>.</p>

<h3 id="performing-the-cross-cluster-live-migration">Performing the Cross Cluster Live Migration</h3>

<p>To live-migrate the VM between clusters, we first need to wait for the source
VM to be ready:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl <span class="nb">wait </span>vm vm-1 <span class="se">\</span>
  <span class="nt">--for</span><span class="o">=</span><span class="nv">condition</span><span class="o">=</span>Ready <span class="nt">--timeout</span><span class="o">=</span>60s
</code></pre></div></div>

<p>After that, we can create the migration receiver in cluster B:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: kubevirt.io/v1
kind: VirtualMachineInstanceMigration
metadata:
  name: migration-target
spec:
  receive:
    migrationID: "cross-cluster-demo"
  vmiName: vm-1
</span><span class="no">EOF
</span></code></pre></div></div>

<p>We need to get the URL for the destination cluster migration agent. This
information will be required to provision the source cluster migration CR.</p>
<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">TARGET_IP</span><span class="o">=</span><span class="si">$(</span><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl get vmim <span class="se">\</span>
  migration-target <span class="nt">-o</span> <span class="nv">jsonpath</span><span class="o">=</span><span class="s1">'{.status.synchronizationAddresses[0]}'</span><span class="si">)</span>
<span class="nb">echo</span> <span class="s2">"Target migration IP: </span><span class="nv">$TARGET_IP</span><span class="s2">"</span>
</code></pre></div></div>

<p>Now that we know the IP of the destination migration controller, we can initiate
the migration from cluster A:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: kubevirt.io/v1
kind: VirtualMachineInstanceMigration
metadata:
  name: migration-source
spec:
  sendTo:
    connectURL: "</span><span class="k">${</span><span class="nv">TARGET_IP</span><span class="k">}</span><span class="sh">:9185"
    migrationID: "cross-cluster-demo"
  vmiName: vm-1
</span><span class="no">EOF
</span></code></pre></div></div>

<p>Monitor the migration progress:</p>

<div class="language-shell highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="c"># Watch migration status in cluster A</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-a kubectl get vmim <span class="se">\</span>
  migration-source <span class="nt">-w</span>

<span class="c"># Watch VM status in cluster B</span>
<span class="nv">KUBECONFIG</span><span class="o">=</span><span class="si">$(</span><span class="nb">pwd</span><span class="si">)</span>/bin/kubeconfig-pe-kind-b kubectl get vm vm-1 <span class="nt">-w</span>
</code></pre></div></div>

<h2 id="conclusion">Conclusion</h2>

<p>Dedicated migration networks are essential for production KubeVirt deployments
that require VM mobility. Without traffic isolation, live migrations compete
with application workloads for bandwidth, potentially degrading service
performance and creating security risks by mixing operational traffic with user
data.</p>

<p>In this post, we have built upon the foundation laid in our
<a href="https://kubevirt.io/2025/Stretched-layer2-network-between-clusters.html">previous article</a>
and enhanced our multi-cluster KubeVirt deployment with cross-cluster live
migration capabilities. We have configured a secondary <code class="language-plaintext highlighter-rouge">L2VNI</code> (VNI 666, VRF
“rouge”) as a dedicated migration network between KubeVirt clusters. This
overlay network provides isolated, high-performance connectivity for migration
operations without requiring additional physical infrastructure. By using EVPN
and OpenPERouter, we demonstrated how cross-cluster live migration works in
practice while maintaining complete separation from application networking.</p>

<p>This setup enables organizations to achieve workload mobility across clusters
with the security, performance, and operational visibility required for
production environments. The overlay approach simplifies management by avoiding
the complexity of physical network expansion while providing the dedicated
bandwidth and monitoring capabilities that enterprise migrations demand.</p>
