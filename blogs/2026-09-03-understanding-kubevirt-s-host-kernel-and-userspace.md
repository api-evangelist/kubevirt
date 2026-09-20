---
title: "Understanding KubeVirt’s Host Kernel and Userspace Dependencies"
url: "https://kubevirt.io//2026/understanding-kubevirt-host-kernel-userspace-dependencies.html"
date: "2026-09-03"
author: "Lee Yarwood"
feed_url: "https://kubevirt.io/feed.xml"
---
KubeVirt runs each VM inside a virt-launcher pod. QEMU and libvirt are userspace processes scheduled like any other container, with host kernel devices ( /dev/kvm , /dev/vhost-net , /dev/net/tun ) mapped in. Like any workload that probes kernel device capabilities, a mismatch between the userspace stack and the host kernel can cause failures.
