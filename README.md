# KubeVirt

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

KubeVirt is a CNCF incubating project that extends Kubernetes to run traditional virtual machines alongside containers. It allows users to create, manage, and run VMs using the same Kubernetes APIs and tools used for containers. KubeVirt is ideal for migrating legacy workloads to Kubernetes without requiring application rewriting.

**Areas:** Cloud-Native · Incubating · Kubernetes · Migration · Virtual Machines · Virtualization

## KubeVirt on APIs.io

Everything profiled for KubeVirt is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/kubevirt/](https://apis.io/providers/kubevirt/)** — or go straight to one of the APIs or properties below.

### APIs (8)

- [KubeVirt StorageProfiles API](https://apis.io/apis/kubevirt/kubevirt-storageprofiles-api/) — Operations for managing StorageProfile resources. StorageProfiles describe the capabilities of a StorageClass and provide default clone and access mode…
- [KubeVirt VirtualMachineInstanceMigrations API](https://apis.io/apis/kubevirt/kubevirt-virtualmachineinstancemigrations-api/) — Operations for managing live migration of VirtualMachineInstances from one node to another without downtime.
- [KubeVirt VirtualMachineInstances API](https://apis.io/apis/kubevirt/kubevirt-virtualmachineinstances-api/) — Operations for managing VirtualMachineInstance (VMI) resources. A VirtualMachineInstance represents a running virtual machine and tracks its actual state.
- [KubeVirt VMConsole API](https://apis.io/apis/kubevirt/kubevirt-vmconsole-api/) — Subresource endpoints for accessing VM consoles via VNC, serial console, and USB redirection.
- [KubeVirt VMLifecycle API](https://apis.io/apis/kubevirt/kubevirt-vmlifecycle-api/) — Subresource operations for VM lifecycle management including start, stop, pause, unpause, restart, migrate, and adding/removing volumes.
- [KubeVirt Data Sources API](https://apis.io/apis/kubevirt/kubevirt-data-sources-api/) — Operations for managing DataSource resources. A DataSource references an existing DataVolume or VolumeSnapshot as a source for cloning into new DataVolumes.
- [KubeVirt Data Volumes API](https://apis.io/apis/kubevirt/kubevirt-data-volumes-api/) — Operations for managing DataVolume resources. A DataVolume is a managed PersistentVolumeClaim with an integrated import/clone pipeline that automatically…
- [KubeVirt Virtual Machines API](https://apis.io/apis/kubevirt/kubevirt-virtual-machines-api/) — Operations for managing VirtualMachine (VM) resources. A VirtualMachine defines the desired state and configuration of a virtual machine, providing…

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Kubevirt Agentic Access](https://apis.io/agentic-access/kubevirt/kubevirt-agentic-access/)

#### JSON Schema (1)

The data models the API reads and writes.

- [KubeVirt VirtualMachine](https://apis.io/schemas/kubevirt/kubevirt-vm/)

#### JSON-LD (1)

Linked-data contexts for the provider's vocabulary.

- [Kubevirt Context](https://apis.io/jsonld/kubevirt/kubevirt-context/)

#### Collections (10)

Postman and Open Collection request collections.

- [KubeVirt Containerized Data Importer API](https://apis.io/collections/kubevirt/open-kubevirt-cdi/)
- [KubeVirt Containerized Data Importer DataSources API](https://apis.io/collections/kubevirt/open-kubevirt-datasources-api/)
- [KubeVirt Containerized Data Importer DataSources DataVolumes API](https://apis.io/collections/kubevirt/open-kubevirt-datavolumes-api/)
- [KubeVirt Containerized Data Importer DataSources StorageProfiles API](https://apis.io/collections/kubevirt/open-kubevirt-storageprofiles-api/)
- [KubeVirt Containerized Data Importer DataSources VirtualMachineInstanceMigrations API](https://apis.io/collections/kubevirt/open-kubevirt-virtualmachineinstancemigrations-api/)
- [KubeVirt Containerized Data Importer DataSources VirtualMachineInstances API](https://apis.io/collections/kubevirt/open-kubevirt-virtualmachineinstances-api/)
- [KubeVirt Containerized Data Importer DataSources VirtualMachines API](https://apis.io/collections/kubevirt/open-kubevirt-virtualmachines-api/)
- [KubeVirt VM Management API](https://apis.io/collections/kubevirt/open-kubevirt-vm/)
- [KubeVirt Containerized Data Importer DataSources VMConsole API](https://apis.io/collections/kubevirt/open-kubevirt-vmconsole-api/)
- [KubeVirt Containerized Data Importer DataSources VMLifecycle API](https://apis.io/collections/kubevirt/open-kubevirt-vmlifecycle-api/)

#### Governance rules (1)

Spectral rulesets for the provider's contracts.

- [KubeVirt — jsonschema rules](https://apis.io/rules/kubevirt/kubevirt-jsonschema-spectral-rules/)

#### Security (1)

Authentication, trust centre, and vulnerability disclosure.

- [Kubevirt Domain Security](https://apis.io/security/kubevirt/kubevirt-domain-security/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Kubevirt Plans Pricing](https://apis.io/plans/kubevirt/kubevirt-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Kubevirt Rate Limits](https://apis.io/rate-limits/kubevirt/kubevirt-rate-limits/)

#### FinOps (1)

What the API costs to run, by operation.

- [Kubevirt Finops](https://apis.io/finops/kubevirt/kubevirt-finops/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://kubevirt.io/)
- [Documentation](https://kubevirt.io/user-guide/)
- [Blog](https://kubevirt.io/blogs/)
- [GitHub](https://github.com/kubevirt)

## API Evangelist rating

**Kin Score 41.0** — developing · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 64.3 |
| Contract quality | 53.5 |
| Contract governance | 9.8 |
| Operational transparency | 36.8 |
| Developer ergonomics | 21.4 |
| Access clarity | 7.9 |

**Agent readiness 21.6** — agent-aware

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | documented (partial credit) |
| MCP Server | — |
| Auth Clarity | — |
| Idempotency | — |
| Error Semantics | — |
| OpenAPI Examples | — |
| Rate Limit Signal | derived (partial credit) |
| Event Surface Described | — |
| Agent Skills | — |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | — |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | paginated |
| Operation Distinctiveness | distinct |

**Regulatory alignment 10.8** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/kubevirt/](https://apis.io/providers/kubevirt/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for KubeVirt: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/kubevirt/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
