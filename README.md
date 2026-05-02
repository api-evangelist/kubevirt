# KubeVirt (kubevirt)
KubeVirt is a CNCF incubating project that extends Kubernetes to run traditional virtual machines alongside containers using the same APIs and tools. It enables legacy workload migration without rewriting applications.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/kubevirt/refs/heads/main/apis.yml)

## Scope
- **Type:** Index
- **Position:** Consumer
- **Access:** 3rd-Party

## Tags:
 - Virtual Machines, Kubernetes, Virtualization, Migration, Cloud Native, Incubating

## Timestamps
- **Created:** 2026-03-16
- **Modified:** 2026-04-28

## APIs

### KubeVirt VM Management API
KubeVirt extends Kubernetes with CRDs for VM management including VirtualMachine, VirtualMachineInstance, and VirtualMachineInstanceMigration resources supporting start, stop, pause, migrate, and snapshot operations.

**Human URL:** [https://kubevirt.io/user-guide/](https://kubevirt.io/user-guide/)

#### Tags:
 - Virtual Machines, Kubernetes API, Live Migration

#### Properties
- [Documentation](https://kubevirt.io/user-guide/)
- [Reference](https://kubevirt.io/api-reference/)
- [OpenAPI](openapi/kubevirt-vm-openapi.yml)
- [JSONSchema](json-schema/kubevirt-vm-schema.json)

### KubeVirt Containerized Data Importer API
REST API for the Containerized Data Importer (CDI), which provides facilities for importing and cloning virtual machine disk images into PersistentVolumeClaims for use as KubeVirt VM disks.

**Human URL:** [https://kubevirt.io/user-guide/storage/containerized_data_importer/](https://kubevirt.io/user-guide/storage/containerized_data_importer/)

#### Tags:
 - Storage, Data Import, PersistentVolumeClaims, Virtual Machines, Kubernetes

#### Properties
- [Documentation](https://kubevirt.io/user-guide/storage/containerized_data_importer/)
- [Reference](https://kubevirt.io/cdi-api-reference/)
- [GitHubRepository](https://github.com/kubevirt/containerized-data-importer)
- [OpenAPI](openapi/kubevirt-cdi-openapi.yml)

## Common Properties
- [Website](https://kubevirt.io/)
- [Documentation](https://kubevirt.io/user-guide/)
- [GitHub Organization](https://github.com/kubevirt)
- [GitHubRepository](https://github.com/kubevirt/kubevirt)
- [Blog](https://kubevirt.io/blogs/)
- [Community](https://github.com/kubevirt/community)
- [JSONSchema](json-schema/kubevirt-vm-schema.json)
- [JSON-LD](json-ld/kubevirt-context.jsonld)

## Maintainers
**FN:** Kin Lane
**Email:** kin@apievangelist.com
