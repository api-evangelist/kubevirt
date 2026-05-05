---
title: "Announcing the release of KubeVirt v1.8"
url: "https://kubevirt.io//2026/KubeVirt-v1-8-release.html"
date: "2026-03-25T00:00:00+00:00"
author: "KubeVirt Maintainers"
feed_url: "https://kubevirt.io//feed.xml"
---
<p>Author: The Kubevirt Community
Release date and time: 25th March</p>

<p>The <a href="https://kubevirt.io">KubeVirt</a> Community is happy to announce the release of <a href="https://github.com/kubevirt/kubevirt/releases/tag/v1.8.0">v1.8</a>, which aligns with <a href="https://kubernetes.io/blog/2025/12/17/kubernetes-v1-35-release/">Kubernetes v1.35</a>.</p>

<p>This is the third release since we started our VEP (<a href="https://github.com/kubevirt/enhancements?tab=readme-ov-file#kubevirt-enhancements-tracking-and-backlog">Virt Enhancement Proposal</a>) process and, after some shaky starts and concerted iterating, we are really starting to see it settle and find a rhythm in the community. We have had a real boom in proposals for this release, and that trend is likely to continue. It’s wonderful to see new contributors coming forward with exciting ideas and engage with the project to see them through.</p>

<p>You can read the full <a href="https://kubevirt.io/user-guide/release_notes/#v180">release notes</a> in our user-guide, but we have included some highlights in this blog.</p>

<p>For those of you at KubeCon this week, we have a whole bunch of talks, as well as a project kiosk, which we have listed on our <a href="https://github.com/kubevirt/community/wiki/Events#upcoming-conferences-with-one-or-more-kubevirt-sessions">events wiki</a>. 
We are also running our first in-person event: <a href="https://kccnceu2026.sched.com/?searchstring=KubeVirt+Summit&amp;iframe=no">KubeVirt Summit Live at the Cloud Native Theatre</a> on Thursday March 26th.</p>

<h3 id="sig-compute">SIG Compute</h3>
<p>The Confidential Computing Working Group has introduced improvements to support Intel TDX Attestation in KubeVirt; confidential VMs can now certify that they are running on confidential hardware (Intel TDX currently).</p>

<p>Another major milestone is the introduction of Hypervisor Abstraction Layer, which enables KubeVirt to integrate multiple hypervisor backends beyond KVM, while still maintaining the current KVM-first behaviour as default.</p>

<p>And because good things happen in threes, we’ve also enabled AI and HPC workloads in VMs to achieve near-native performance with the introduction of PCIe NUMA topology awareness alongside other resource improvements.</p>

<h3 id="sig-networking">SIG Networking</h3>
<p>The <code class="language-plaintext highlighter-rouge">passt</code> binding has been promoted from a plugin to a core binding. This binding is a significant improvement to an earlier implementation.</p>

<p>Also, you can now live update NAD references without requiring VM restart, allowing you to change a VM’s backing network without disrupting the guest.</p>

<p>And we have decoupled KubeVirt from NAD definitions to reduce API calls made by virt-controller, removing a performance bottleneck for VM activation at scale and improving security by removing permissions. Users should be aware that this is a deprecating process and prepare accordingly.</p>

<h3 id="sig-storage">SIG Storage</h3>
<p>The big news on the storage front is two new features: ContainerPath volume and Incremental Backup with CBT.</p>

<p>ContainerPath volumes allow you to map container paths for VM storage and improve portability and configuration options. This provides an escape hatch for cloud provider credential injection patterns.</p>

<p>Incremental Backup with Changed Block Tracking (CBT) leverages QEMU’s and libvirt backup capabilities providing <strong>storage agnostic</strong> incremental VM backups. By capturing only modified data, the solution eliminates reliance on specific CSI drivers, allowing for faster backup windows and a drastically reduced storage footprint. This not only ensures storage freedom but also minimizes cluster network traffic for peak efficiency.</p>
<h3 id="sig-scale-and-performance">SIG Scale and Performance</h3>

<p>There have been a few test improvements rolled out in SIG Scale and Performance.  First, we have increased the KWOK performance test to 8000 VMIs.  The results have shown the kubevirt control-plane performs well even as VMI counts grow.  On the scale side, when comparing the 100 VMI job to 8000 VMI job, we see some expected memory increases.  The average virt-api memory grows from 140MB to 170MB (+30MB) and average virt-controller memory grows from 65MB to 1400MB (+1335MB).
To determine the memory scaling per Virtual Machine Instance (VMI), we calculate the rate of change on the control-plane in the 100 real VMIs and 8000 KWOK VMIs. This estimates the incremental memory cost for each additional VMI added to the system.</p>

<table>
  <thead>
    <tr>
      <th>Component</th>
      <th>Total Memory Increase 100 to 8000 (Δ)</th>
      <th>Memory Scale per VMI (MB)</th>
      <th>Memory Scale per VMI (KB)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>virt-api</td>
      <td>30 MB</td>
      <td>0.0038 MB</td>
      <td>3.89 KB</td>
    </tr>
    <tr>
      <td>virt-controller</td>
      <td>1335 MB</td>
      <td>0.1690 MB</td>
      <td>173.04 KB</td>
    </tr>
  </tbody>
</table>

<p>We will continue to refine these measurements as they are still estimates and may have some incorrect measurements. Our goal is to eventually publish this along this our comprehensive list of performance and scale benchmarks for each release, which is <a href="https://github.com/kubevirt/kubevirt/blob/main/docs/perf-scale-benchmarks.md">here</a>.</p>

<h3 id="thanks">Thanks!</h3>
<p>A lot of work from a huge amount of people go into these releases. Some contributions are small, such as raising a bug or attending our community meeting, and others are massive, like working on a feature or reviewing PRs. Whatever your part: we thank you.</p>

<p>We had a huge amount of features and the next release is looking to be larger still. If you’re interested in contributing and being a part of this great project, please check out our <a href="https://kubevirt.io/user-guide/contributing/">contributing guide</a> and our <a href="https://github.com/kubevirt/community/blob/main/membership_policy.md">community membership guidelines</a>. Reviewing PRs is a great way to learn and gain experience, but it can sometimes be daunting. If you’d like to be involved but aren’t sure, reach out on our Slack or mailing list; we have some wonderful people in the community who can help you find your feet.</p>
