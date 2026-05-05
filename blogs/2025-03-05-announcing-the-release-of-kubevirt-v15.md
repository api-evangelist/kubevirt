---
title: "Announcing the release of KubeVirt v1.5"
url: "https://kubevirt.io//2025/KubeVirt-v1-5_release.html"
date: "2025-03-05T00:00:00+00:00"
author: "KubeVirt Maintainers"
feed_url: "https://kubevirt.io//feed.xml"
---
<p>The KubeVirt Community is pleased to announce the release of <a href="https://github.com/kubevirt/kubevirt/releases/tag/v1.5.0">KubeVirt v1.5</a>. This release aligns with <a href="https://kubernetes.io/blog/2024/12/11/kubernetes-v1-32-release/">Kubernetes v1.32</a> and is the seventh KubeVirt release to follow the Kubernetes release cadence.</p>

<p>This release sees the project adding some features that are aligned with more traditional virtualization platforms, such as enhanced volume and VM migration, increased CPU performance, and more precise network state control.</p>

<p>You can read the full <a href="https://kubevirt.io/user-guide/release_notes/#v150">release notes</a> in our user-guide, but we have included some highlights in this blog.</p>

<h3 id="breaking-change">Breaking change</h3>
<p>Please be aware that in v1.5 we have <a href="https://github.com/kubevirt/kubevirt/pull/13497">introduced a change</a> that affects permissions of namespace admins to trigger live migrations. As a hardening measure (principle of least privilege), the right of creating, editing and deleting <code class="language-plaintext highlighter-rouge">VirtualMachineInstanceMigrations</code> are no longer assigned by default to namespace admins.</p>

<p>For more information, see our post on the <a href="https://kubevirt.io/2025/Hardening-VMIM.html">KubeVirt blog</a>.</p>

<h3 id="feature-ga">Feature GA</h3>

<p>This release marks the graduation of a number of features to GA; deprecating the feature gate and now enabled by default:</p>

<ul>
  <li><a href="https://kubevirt.io/user-guide/storage/volume_migration/">Migration Update Strategy and Volume Migration</a>: Storage migration can be useful in the cases where the users need to change the underlying storage, for example, if the storage class has been deprecated, or there is a new more performant driver available.</li>
  <li><a href="https://kubevirt.io/user-guide/compute/resources_requests_and_limits/">Auto Resource Limits</a>: Automatically apply CPU limits to a VMI.</li>
  <li>VM Live Update Features: This feature underpins hotplugging of CPU, memory, and volume resources.</li>
  <li><a href="https://kubevirt.io/user-guide/network/network_binding_plugins/">Network Binding Plugin</a>: A modular plugin which integrates with KubeVirt to implement a network binding.</li>
</ul>

<h3 id="compute">Compute</h3>

<p>You can now specify the number of <a href="https://kubevirt.io/user-guide/storage/disks_and_volumes/#iothreads">IOThreads to use</a> through virtqueue mapping to improve CPU performance. We also added <a href="https://github.com/kubevirt/kubevirt/pull/13606">virtio video support for amd64</a> as well as the <a href="https://github.com/kubevirt/kubevirt/pull/13208">ability to reset VMs</a>, which provides the means to restart the guest OS without requiring a new pod to be scheduled.</p>

<h3 id="networking">Networking</h3>

<p>You can now <a href="https://kubevirt.io/user-guide/network/interfaces_and_networks/#link-state-management">dynamically control the link state</a> (up/down) of a network interface.</p>

<h3 id="scale-and-performance">Scale and Performance</h3>

<p>A comprehensive list of performance and scale benchmarks for the release is <a href="https://github.com/kubevirt/kubevirt/blob/main/docs/perf-scale-benchmarks.md">available here</a>. A notable change added to the benchmarks was the <a href="https://github.com/kubevirt/kubevirt/pull/13250">virt-handler resource utilization metrics</a>. This metric gives the avg, max and min memory/cpu utilization per VMI that is scheduled on the node where virt-handler is running. Another notable shoutout from the benchmark document is changing how <a href="https://github.com/kubevirt/kubevirt/pull/12716">list calls are tracked</a>. KubeVirt clients were misreporting watch calls as list calls, which was fixed in this release.</p>

<h3 id="storage">Storage</h3>

<p>With this release you can now migrate <a href="https://kubevirt.io/user-guide/storage/volume_migration/">hotplugged volumes</a>. You can also migrate VMIs with a volume shared using virtiofs. And we <a href="https://github.com/kubevirt/kubevirt/pull/13713">addressed a recent change in libvirt</a> that was preventing some NFS shared volumes from migrating by providing shared filesystem paths upfront.</p>

<h3 id="thank-you-for-your-contribution">Thank you for your contribution!</h3>
<p>A lot of work from a <a href="https://kubevirt.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&amp;var-period_name=v1.4.0%20-%20now&amp;var-metric=contributions&amp;var-repogroup_name=All&amp;var-country_name=All&amp;var-companies=All">huge amount of people</a> goes into a release. A huge thank you to the 350+ people who contributed to this v1.5 release.</p>

<p>And if you’re interested in contributing to the project and being a part of the next release, please check out our <a href="https://kubevirt.io/user-guide/contributing/">contributing guide</a> and our <a href="https://github.com/kubevirt/community/blob/main/membership_policy.md">community membership guidelines</a>.</p>

<p>Contributing needn’t be designing a new feature or committing to a <a href="https://github.com/kubevirt/enhancements">Virtualization Enhancement Proposal</a>, there is always a need for reviews, help with our docs and website, or submitting good quality bugs. Every little bit counts.</p>
