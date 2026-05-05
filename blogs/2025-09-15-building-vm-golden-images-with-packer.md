---
title: "Building VM golden images with Packer"
url: "https://kubevirt.io//2025/Building-VM-golden-image-with-Packer.html"
date: "2025-09-15T00:00:00+00:00"
author: "Ben Oukhanov"
feed_url: "https://kubevirt.io//feed.xml"
---
<h2 id="introduction">Introduction</h2>

<p>Creating and maintaining VM golden images can be time-consuming, often
requiring local virtualization tools and manual setup. With <a href="https://kubevirt.io">KubeVirt</a>
running inside your <a href="https://kubernetes.io">Kubernetes</a> cluster, you can
manage virtual machines alongside your containers, but it lacks automation
for creating consistent, reusable VM images.</p>

<p>That’s where <a href="https://packer.io">Packer</a> and the new KubeVirt plugin come
in. The plugin lets you build VM images directly in Kubernetes, enabling you
to automate OS installation from ISO, customize the VM during build, and produce
a reusable bootable volume, all without leaving your cluster.</p>

<h2 id="prerequisites">Prerequisites</h2>

<p>Before you begin, make sure you have the following installed:</p>

<ul>
  <li><a href="https://developer.hashicorp.com/packer/install">Packer</a></li>
  <li><a href="https://kubernetes.io/docs/tasks/tools">Kubernetes</a></li>
  <li><a href="https://kubevirt.io/user-guide/cluster_admin/installation">KubeVirt</a></li>
  <li><a href="https://kubevirt.io/user-guide/storage/containerized_data_importer/#install-cdi">Containerized Data Importer (CDI)</a></li>
</ul>

<h2 id="plugin-features">Plugin Features</h2>

<p>The Packer plugin for KubeVirt offers a variety of features that simplify
the VM golden image creation process:</p>

<ul>
  <li><strong>HCL Template</strong>: Define infrastructure as code for easy versioning and reuse using <a href="https://developer.hashicorp.com/packer/docs/templates/hcl_templates">HCL templates</a>.</li>
  <li><strong>ISO Installation</strong>: Build VM golden images from an ISO file using the <code class="language-plaintext highlighter-rouge">kubevirt-iso</code> builder.</li>
  <li><strong>ISO Media Files</strong>: Include additional files (e.g., configs, scripts, and more) during the installation process.</li>
  <li><strong>Boot Command</strong>: Automate the VM boot process via a <a href="https://en.wikipedia.org/wiki/VNC">VNC</a> connection with a predefined set of commands.</li>
  <li><strong>Integrated SSH/WinRM Access</strong>: Provision and customize VMs via <a href="https://man7.org/linux/man-pages/man1/ssh.1.html">SSH</a> or <a href="https://learn.microsoft.com/en-us/windows/win32/winrm/portal">WinRM</a>.</li>
</ul>

<p><strong>Note</strong>: This plugin is currently in pre-release and actively under development by
<a href="https://www.redhat.com">Red Hat</a> and <a href="https://www.hashicorp.com">HashiCorp</a> together.</p>

<h2 id="plugin-components">Plugin Components</h2>

<p>The core component of this plugin is the <code class="language-plaintext highlighter-rouge">kubevirt-iso</code> builder. This builder
allows you to start from an ISO file and create a VM golden image directly
on your Kubernetes cluster.</p>

<h3 id="builder-design">Builder Design</h3>

<p align="center">
  <img alt="Design" src="https://kubevirt.io/assets/2025-09-15-Packer-Plugin/kubevirt-iso-builder-design.png" width="1125" />
</p>

<p>This diagram shows the workflow for building a bootable volume in a
Kubernetes cluster using Packer with the KubeVirt plugin.</p>

<ol>
  <li>Creates a temporary VM from an ISO image.</li>
  <li>Runs provisioning using either the <a href="https://developer.hashicorp.com/packer/docs/provisioners/shell">Shell</a> or <a href="https://developer.hashicorp.com/packer/integrations/hashicorp/ansible/latest/components/provisioner/ansible">Ansible</a> provisioner.</li>
  <li>Clones the VM’s disk to create a reusable bootable volume (<a href="https://kubevirt.io/user-guide/storage/disks_and_volumes/#datavolume">DataVolume and DataSource</a>).</li>
</ol>

<p>This bootable volume can then be reused to instantiate new VMs without
repeating the installation.</p>

<h2 id="step-by-step-example-building-a-fedora-vm-image">Step-by-Step Example: Building a Fedora VM Image</h2>

<p>The following Packer template (Fedora 42) demonstrates key features:</p>

<ul>
  <li>ISO-based installation using the <code class="language-plaintext highlighter-rouge">kubevirt-iso</code> builder.</li>
  <li>Embedded configuration file to automate the installation.</li>
  <li>Sending boot commands to inject <code class="language-plaintext highlighter-rouge">ks.cfg</code> in GRUB.</li>
  <li>SSH provisioning with a <a href="https://developer.hashicorp.com/packer/docs/provisioners/shell">Shell</a> provisioner.</li>
  <li>Full integration with <a href="https://kubevirt.io/user-guide/user_workloads/instancetypes">InstanceTypes and Preferences</a>.</li>
</ul>

<p>Follow these steps to build a Fedora VM image inside your Kubernetes cluster.</p>

<h3 id="step-1-export-kubeconfig-variable">Step 1: Export KubeConfig Variable</h3>

<p>Export your <a href="https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#the-kubeconfig-environment-variable">KubeConfig</a>
variable, which is also used by the Packer plugin:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">export </span><span class="nv">KUBECONFIG</span><span class="o">=</span>~/.kube/config
</code></pre></div></div>

<p>This is required to communicate with your Kubernetes cluster.</p>

<h3 id="step-2-deploy-iso-datavolume">Step 2: Deploy ISO DataVolume</h3>

<p>Create a <a href="https://kubevirt.io/user-guide/storage/disks_and_volumes/#datavolume">DataVolume</a> to import the Fedora ISO into your cluster’s storage:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>kubectl apply <span class="nt">-f</span> - <span class="o">&lt;&lt;</span><span class="no">EOF</span><span class="sh">
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: fedora-42-x86-64-iso
  annotations:
    #
    # This annotation triggers immediate binding of the PVC,
    # speeding up provisioning.
    #
    cdi.kubevirt.io/storage.bind.immediate.requested: "true"
spec:
  source:
    http:
      #
      # Please check if this URL link is valid, in case the import fails.
      # If so, please modify the URL here below.
      #
      url: "https://download.fedoraproject.org/pub/fedora/linux/releases/42/Server/x86_64/iso/Fedora-Server-dvd-x86_64-42-1.1.iso"
  pvc:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 3Gi
</span><span class="no">EOF
</span></code></pre></div></div>

<h4 id="alternative-upload-a-local-iso">Alternative: Upload a local ISO</h4>

<p>Instead of importing from a URL, you can upload a local ISO
using the <a href="https://kubevirt.io/user-guide/user_workloads/virtctl_client_tool">virtctl</a> client tool:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>virtctl image-upload dv fedora-42-x86-64-iso <span class="se">\</span>
  <span class="nt">--size</span><span class="o">=</span>3Gi <span class="se">\</span>
  <span class="nt">--image-path</span><span class="o">=</span>./Fedora-Server-dvd-x86_64-42-1.1.iso <span class="se">\</span>
</code></pre></div></div>

<p>The <a href="https://fedoraproject.org/server/download">Fedora Server 42 ISO</a> is available on Fedora’s official website.</p>

<h3 id="step-3-create-kickstart-file">Step 3: Create Kickstart File</h3>

<p>This <a href="https://en.wikipedia.org/wiki/Kickstart_(Linux)">Kickstart</a> file automates
Fedora installation, enabling unattended VM setup.</p>

<p>Create a file named <code class="language-plaintext highlighter-rouge">ks.cfg</code> with the following configuration:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">cat</span> <span class="o">&gt;</span> ks.cfg <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
cdrom
text
firstboot --disable
lang en_US.UTF-8
keyboard us
timezone Europe/Paris --utc
selinux --enforcing
rootpw root
firewall --enabled --ssh
network --bootproto dhcp
user --groups=wheel --name=user --password=root --uid=1000 --gecos="user" --gid=1000

bootloader --location=mbr --append="net.ifnames=0 biosdevname=0 crashkernel=no"

zerombr
clearpart --all --initlabel
autopart --type=lvm

poweroff

%packages --excludedocs
@core
qemu-guest-agent
openssh-server
%end

%post
systemctl enable --now sshd
systemctl enable --now qemu-guest-agent
%end
</span><span class="no">EOF
</span></code></pre></div></div>

<p>This configuration enables SSH to provision the temporary VM, and <a href="https://qemu-project.gitlab.io/qemu/interop/qemu-ga.html">QEMU Guest Agent</a>
to have a better integration with KubeVirt itself.</p>

<h3 id="step-4-create-packer-template">Step 4: Create Packer Template</h3>

<p>Create an example of the Packer template (<code class="language-plaintext highlighter-rouge">fedora.pkr.hcl</code>):</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nb">cat</span> <span class="o">&gt;</span> fedora.pkr.hcl <span class="o">&lt;&lt;</span> <span class="sh">'</span><span class="no">EOF</span><span class="sh">'
packer {
  required_plugins {
    kubevirt = {
      source  = "github.com/hashicorp/kubevirt"
      version = "&gt;= 0.8.0"
    }
  }
}

variable "kube_config" {
  type    = string
  default = "</span><span class="k">${</span><span class="nv">env</span><span class="p">(</span><span class="s2">"KUBECONFIG"</span><span class="p">)</span><span class="k">}</span><span class="sh">"
}

variable "namespace" {
  type    = string
  default = "vm-images"
}

variable "name" {
  type    = string
  default = "fedora-42-rand-85"
}

source "kubevirt-iso" "fedora" {
  # Kubernetes configuration
  kube_config   = var.kube_config
  name          = var.name
  namespace     = var.namespace

  # ISO configuration
  iso_volume_name = "fedora-42-x86-64-iso"

  # VM type and preferences
  disk_size          = "10Gi"
  instance_type      = "o1.medium"
  preference         = "fedora"
  os_type            = "linux"

  # Default network configuration
  networks {
    name = "default"

    pod {}
  }

  # Files to include in the ISO installation
  media_files = [
    "./ks.cfg"
  ]

  # Boot process configuration
  # A set of commands to send over VNC connection
  boot_command = [
    "&lt;up&gt;e",                            # Modify GRUB entry
    "&lt;down&gt;&lt;down&gt;&lt;end&gt;",                # Navigate to kernel line
    " inst.ks=hd:LABEL=OEMDRV:/ks.cfg", # Set kickstart file location
    "&lt;leftCtrlOn&gt;x&lt;leftCtrlOff&gt;"        # Boot with modified command line
  ]
  boot_wait                 = "10s"     # Time to wait after boot starts
  installation_wait_timeout = "15m"     # Timeout for installation to complete

  # SSH configuration
  communicator      = "ssh"
  ssh_host          = "127.0.0.1"
  ssh_local_port    = 2020
  ssh_remote_port   = 22
  ssh_username      = "user"
  ssh_password      = "root"
  ssh_wait_timeout  = "20m"
}

build {
  sources = ["source.kubevirt-iso.fedora"]

  provisioner "shell" {
    inline = [
      "echo 'Install packages, configure services, or tweak system settings here.'",
    ]
  }
}
</span><span class="no">EOF
</span></code></pre></div></div>

<h3 id="step-5-export-vm-image-optional">Step 5: Export VM Image (Optional)</h3>

<p>Optionally, export the newly created disk image and package it
into a <a href="https://kubevirt.io/user-guide/storage/disks_and_volumes/#containerdisk">containerDisk</a>
so it can be shared across multiple Kubernetes clusters.</p>

<h4 id="required-dependencies">Required Dependencies</h4>

<p>Install these tools on the machine running Packer:</p>

<ul>
  <li><code class="language-plaintext highlighter-rouge">virtctl</code>: Exports the VM image from the KubeVirt cluster.</li>
  <li><code class="language-plaintext highlighter-rouge">qemu-img</code>: Converts raw images to qcow2 format.</li>
  <li><code class="language-plaintext highlighter-rouge">gunzip</code>: Decompresses exported VM images.</li>
  <li><code class="language-plaintext highlighter-rouge">podman</code>: Builds and pushes container images.</li>
</ul>

<h4 id="example">Example</h4>

<p>Add a <code class="language-plaintext highlighter-rouge">shell-local</code> post-processor to the Packer build, which runs after the build is completed:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>variable <span class="s2">"registry"</span> <span class="o">{</span>
  <span class="nb">type</span>    <span class="o">=</span> string
  default <span class="o">=</span> <span class="s2">"quay.io/containerdisks"</span>
<span class="o">}</span>

variable <span class="s2">"registry_username"</span> <span class="o">{</span>
  <span class="nb">type</span>      <span class="o">=</span> string
  sensitive <span class="o">=</span> <span class="nb">true</span>
<span class="o">}</span>

variable <span class="s2">"registry_password"</span> <span class="o">{</span>
  <span class="nb">type</span>      <span class="o">=</span> string
  sensitive <span class="o">=</span> <span class="nb">true</span>
<span class="o">}</span>

variable <span class="s2">"image_tag"</span> <span class="o">{</span>
  <span class="nb">type</span>    <span class="o">=</span> string
  default <span class="o">=</span> <span class="s2">"latest"</span>
<span class="o">}</span>

build <span class="o">{</span>
  ...

  post-processor <span class="s2">"shell-local"</span> <span class="o">{</span>
    inline <span class="o">=</span> <span class="o">[</span>
      <span class="c"># Export VM disk image from PVC</span>
      <span class="s2">"virtctl -n </span><span class="k">${</span><span class="nv">var</span><span class="p">.namespace</span><span class="k">}</span><span class="s2"> vmexport download </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">-export --pvc=</span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2"> --output=</span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.img.gz"</span>,

      <span class="c"># Decompress exported VM image</span>
      <span class="s2">"gunzip -k </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.img.gz"</span>,

      <span class="c"># Convert raw image to qcow2 (smaller and more efficient format)</span>
      <span class="s2">"qemu-img convert -c -O qcow2 </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.img </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.qcow2"</span>,

      <span class="c"># Generate Containerfile</span>
      <span class="s2">"echo 'FROM scratch' &gt; </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.Containerfile"</span>,
      <span class="s2">"echo 'COPY </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.qcow2 /disk/' &gt;&gt; </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.Containerfile"</span>,

      <span class="c"># Login to registry</span>
      <span class="s2">"podman login -u </span><span class="k">${</span><span class="nv">var</span><span class="p">.registry_username</span><span class="k">}</span><span class="s2"> -p </span><span class="k">${</span><span class="nv">var</span><span class="p">.registry_password</span><span class="k">}</span><span class="s2"> </span><span class="k">${</span><span class="nv">var</span><span class="p">.registry</span><span class="k">}</span><span class="s2">"</span>,

      <span class="c"># Build and push image</span>
      <span class="s2">"podman build -t </span><span class="k">${</span><span class="nv">var</span><span class="p">.registry</span><span class="k">}</span><span class="s2">/</span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">:</span><span class="k">${</span><span class="nv">var</span><span class="p">.image_tag</span><span class="k">}</span><span class="s2"> -f </span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">.Containerfile ."</span>,
      <span class="s2">"podman push </span><span class="k">${</span><span class="nv">var</span><span class="p">.registry</span><span class="k">}</span><span class="s2">/</span><span class="k">${</span><span class="nv">var</span><span class="p">.name</span><span class="k">}</span><span class="s2">:</span><span class="k">${</span><span class="nv">var</span><span class="p">.image_tag</span><span class="k">}</span><span class="s2">"</span>
    <span class="o">]</span>
  <span class="o">}</span>
<span class="o">}</span>
</code></pre></div></div>

<p>Sensitive credentials such as registry usernames and passwords should never be hardcoded in templates.</p>

<h3 id="step-6-initialize-packer-plugin">Step 6: Initialize Packer Plugin</h3>

<p>Run the following command once to install the Packer plugin:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>packer init fedora.pkr.hcl
</code></pre></div></div>

<p>This downloads and sets up the KubeVirt plugin automatically.</p>

<h3 id="step-7-run-packer-build">Step 7: Run Packer Build</h3>

<p>Finally, run a build to create a new VM golden image with:</p>

<div class="language-bash highlighter-rouge"><div class="highlight"><pre class="highlight"><code>packer build fedora.pkr.hcl
</code></pre></div></div>

<p>Packer will create a new VM golden image in your Kubernetes cluster.</p>

<h2 id="conclusion">Conclusion</h2>

<p>In this walkthrough, you built a Fedora VM golden image inside Kubernetes
using Packer and the KubeVirt plugin. You defined an ISO source, automated
installation with Kickstart configuration and provisioned the VM over SSH
— all within your Kubernetes cluster.</p>

<p>From here, you can:</p>

<ul>
  <li>Reuse the bootable volume to launch new VMs instantly.</li>
  <li>Integrate Packer builds into your CI/CD pipelines.</li>
  <li>Adapt the same process to build images for other operating systems, such as <a href="https://github.com/hashicorp/packer-plugin-kubevirt/tree/main/examples/builder/kubevirt-iso/rhel">RHEL</a> and <a href="https://github.com/hashicorp/packer-plugin-kubevirt/tree/main/examples/builder/kubevirt-iso/windows">Windows</a>.</li>
</ul>

<p>The plugin is still in pre-release, but it already offers a streamlined way
to create consistent VM images inside Kubernetes.</p>

<p>Give it a try and share your feedback or contributions on <a href="https://github.com/hashicorp/packer-plugin-kubevirt">GitHub</a>!</p>
