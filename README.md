# KubeVirt (kubevirt)

KubeVirt is a CNCF incubating project that extends Kubernetes to run traditional virtual machines alongside containers. It allows users to create, manage, and run VMs using the same Kubernetes APIs and tools used for containers. KubeVirt is ideal for migrating legacy workloads to Kubernetes without requiring application rewriting.

**APIs.json:** [https://kubevirt.io](https://kubevirt.io)

## Scope

- **Type:** Index

## Tags

- Cloud Native
- Incubating
- Kubernetes
- Migration
- Virtual Machines
- Virtualization

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-05-19

## APIs

### KubeVirt VM Management API

KubeVirt extends the Kubernetes API with custom resources for virtual machine management. VirtualMachine resources define VM specifications including CPU, memory, disks, and network interfaces. VirtualMachineInstance tracks running VMs, and VirtualMachineInstanceMigration handles live migrations. The API supports start, stop, pause, migrate, and snapshot operations through standard kubectl commands.

- **Human URL:** [https://kubevirt.io/user-guide/](https://kubevirt.io/user-guide/)

#### Tags

- Kubernetes API
- Live Migration
- Virtual Machines

#### Properties

- [Documentation](https://kubevirt.io/user-guide/)
- [Reference](https://kubevirt.io/api-reference/)
- [OpenAPI](openapi/kubevirt-vm-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/kubevirt-vm.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/kubevirt-vm.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [JSON Schema](json-schema/kubevirt-vm-schema.json) — [JSON Schema](https://json-schema.org/specification)

### KubeVirt Containerized Data Importer API

REST API for the Containerized Data Importer (CDI), which provides facilities for importing and cloning virtual machine disk images into PersistentVolumeClaims for use as KubeVirt VM disks. The CDI API includes DataVolume, DataSource, and StorageProfile resources for managing data import pipelines.

- **Human URL:** [https://kubevirt.io/user-guide/storage/containerized_data_importer/](https://kubevirt.io/user-guide/storage/containerized_data_importer/)

#### Tags

- Data Import
- Kubernetes
- PersistentVolumeClaims
- Storage
- Virtual Machines

#### Properties

- [Documentation](https://kubevirt.io/user-guide/storage/containerized_data_importer/)
- [Reference](https://kubevirt.io/cdi-api-reference/)
- [GitHub Repository](https://github.com/kubevirt/containerized-data-importer)
- [OpenAPI](openapi/kubevirt-cdi-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/kubevirt-cdi.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/kubevirt-cdi.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [Website](https://kubevirt.io/)
- [JSON-LD](json-ld/kubevirt-context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [JSON Schema](json-schema/kubevirt-vm-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [Documentation](https://kubevirt.io/user-guide/)
- [GitHub Organization](https://github.com/kubevirt)
- [GitHub Repository](https://github.com/kubevirt/kubevirt)
- [Blog](https://kubevirt.io/blogs/)
- [Community](https://github.com/kubevirt/community)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
