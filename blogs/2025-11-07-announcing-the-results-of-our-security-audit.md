---
title: "Announcing the results of our Security Audit"
url: "https://kubevirt.io//2025/Announcing-KubeVirt-Security-Audit-Results.html"
date: "2025-11-07T00:00:00+00:00"
author: "KubeVirt Maintainers"
feed_url: "https://kubevirt.io//feed.xml"
---
<p>The KubeVirt Community is very pleased to share the results of our security audit, completed through the guidance of the Open Source Technology Improvement Fund (OSTIF) and the technical expertise of Quarkslab.</p>

<p>This is a critical step in KubeVirt moving to Graduation within the CNCF framework, and is the first time the project has been publicly audited.</p>

<p>The audit was conducted by Quarkslab earlier this year, beginning with an architectural review of KubeVirt and the creation of a threat model that identified threat actors, attack scenarios, and attack surfaces of the project. These were used to then test, prod, and poke to uncover and exploit any weak points.</p>

<p>The audit found the following:</p>

<ul>
  <li>15 findings with a Security Impact:
    <ul>
      <li>0 Critical</li>
      <li>1 High
        <ul>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-46xp-26xh-hpqh">CVE-2025-64324</a></li>
        </ul>
      </li>
      <li>7 Medium
        <ul>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-38jw-g2qx-4286">CVE-2025-64432</a></li>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-qw6q-3pgr-5cwq">CVE-2025-64433</a></li>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-ggp9-c99x-54gp">CVE-2025-64434</a></li>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-9m94-w2vq-hcf9">CVE-2025-64435</a></li>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-7xgm-5prm-v5gc">CVE-2025-64436</a></li>
          <li><a href="https://github.com/kubevirt/kubevirt/security/advisories/GHSA-2r4r-5x78-mvqf">CVE-2025-64437</a></li>
        </ul>
      </li>
      <li>4 Low</li>
      <li>3 Informational</li>
    </ul>
  </li>
</ul>

<p>Quarkslab also provided us with a Custom Threat Model and Fix Recommendations, and kept in touch after delivering the audit to help us understand and address the weaknesses they found. One of their team even volunteered their time to help remediate some of these issues, which we greatly appreciated!</p>

<p>These findings were provided to the project maintainers privately with an agreed response time to allow KubeVirt to address them prior to publication.</p>

<p>The KubeVirt maintainers are very happy with these results, as they demonstrate not only the strength and security focus of our community, as well as the payoff of our earlier investment of moving to non-privileged by default, and by being compliant with the standard Kubernetes Security Model, which includes SELinux policies, seccomp and Pod Security Standards. It is worth noting that Kubernetes is also maturing and providing more security features, allowing KubeVirt and other projects in the ecosystem to inherently increase our security.</p>

<p>This all highlights the unique benefits and additional isolation of running virtual machines as containers in addition to the benefits of using virtual machines.</p>

<p>Having your project audited is both nerve-inducing and extremely comforting. The KubeVirt project is deeply invested in following security best practices, and part of these best practices is having your project audited by a third party to find any possible weaknesses before a malicious actor. KubeVirt maintainers appreciate the OSTIF initiative in promoting security of CNCF projects.</p>

<p>You can read the <a href="https://ostif.org/wp-content/uploads/2025/10/KubeVirt_OSTIF_Report_25-06-2150-REP_v1.2.pdf">full Audit Report here</a>.</p>

<p><a href="https://blog.quarkslab.com/kubevirt-security-audit.html">Quarkslab’s blog on the process here</a>.</p>

<p>And <a href="https://ostif.org/kubevirt-audit-complete/">OSTIF’s blog here</a>.</p>

<p>A huge thanks to everyone involved:<br />
<strong>Quarkslab</strong>: Sébastien Rolland, Mihail Kirov, and Pauline Sauder<br />
<strong>OSTIF</strong>: Helen Woeste and Amir Montazery<br />
<strong>KubeVirt</strong>: Jed Lejosne, Ľuboslav Pivarč, Vladik Romanovsky, Federico Fossemò, Stu Gott, Roman Mohr, Fabian Deutsch, and Andrew Burden</p>

<p>We recommend users update their clusters to the latest supported z-stream version of KubeVirt.<br />
See our <a href="https://github.com/kubevirt/sig-release/blob/main/releases/k8s-support-matrix.md">KubeVirt to Kubernetes version support matrix</a> for more information on supported KubeVirt versions.</p>
