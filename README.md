Kubernetes Security Checklist and Requirements

There are many ways to make your cluster secure, but we have chosen only one, the most difficult and controversial in some places. We do not guarantee that it will be completely suitable for your infrastructure, but we hope this checklist can help you include those things that you may have forgotten and left out.

Docker Security Guide копия

    Authentication
        It is recommended to use an IdP server as a provider for user authentication to the Kubernetes API (for example, using OIDC). Cluster administrators are advised not to use service account tokens for authentication.
        It is recommended to use a centralized certificate management service to manage certificates within the cluster (for user and service purposes).
        User accounts should be personalized. The names of the service accounts should reflect the purpose access rights of the accounts.
    Authorization
        For each cluster, a role-based access model should be developed.
        Role-Based Access Control (RBAC) should be configured for the Kubernetes cluster. Rights need to be assigned within the project namespace based on least privilege and separation of duties (RBAC-tool).
        All services should have a unique service account with configured RBAC rights.
        Developers should not have access to a production environment without the approval of the security team.
        It is forbidden to use user impersonation (the ability to perform actions under other accounts).
        It is forbidden to use anonymous authentication, except for /healthz, /readyz, /livez. Exceptions should be agreed upon with the security team.
        Cluster administrators and maintainers should interact with the cluster API and infrastructure services through privileged access management systems (Teleport, Boundary).
        All information systems should be divided into separate namespaces. It is recommended to avoid the situation when the same maintainer team is responsible for different namespaces.
        RBAC Rights should be audited regularly (KubiScan, Krane)
    Secure work with secrets
        Secrets should be stored in third-party storage (HashiCorp Vault, Conjur), or in etcd in encrypted form.
        Secrets should be added to the container using the volumeMount mechanism or the secretKeyRef mechanism. For hiding secrets in source codes, for example, the sealed-secret tool can be used.
    Cluster Configuration Security
        Use TLS encryption between all cluster components.
        Use Policy engine (OPA, Kyverno, jsPolicy, Kubewarden).
        The cluster configuration is recommended to comply with CIS Benchmark except for PSP requirements.
        It is recommended to use only the latest versions of cluster components (CVE list).
        For services with increased security requirements, it is recommended to use a low-level run-time with a high degree of isolation (gVisor, Kata-runtime).
        Cluster Configuration should be audited regularly (Kube-bench, Kube-hunter, Kubestriker)
    Audit and Logging
        Log all cases of changing access rights in the cluster.
        Log all operations with secrets (including unauthorized access to secrets).
        Log all actions related to the deployment of applications and changes in their configuration.
        Log all cases of changing parameters, system settings, or configuration of the entire cluster (including OS level).
        All registered security events (at the cluster level and application level both) should be sent to the centralized audit logging system (SIEM).
        The audit logging system should be located outside the Kubernetes cluster.
        Build observability and visibility processes in order to understand what is happening in infrastructure and services (Luntry, WaveScope)
        Use third-party security monitoring tool on all cluster nodes (Falco, Sysdig, Aqua Enterpise, NeuVector, Prisma Cloud Compute).
    Secure OS configuration
        Host administrators and maintainers should interact with cluster nodes through privileged access management systems (or bastion hosts).
        It is recommended to configure the OS and software following the baseline and standards (CIS, NIST).
        It is recommended to regularly scan packages and configuration for vulnerabilities(OpenSCAP profiles, Lynis).
        It is recommended to regularly update the OS kernel version (CVEhound).
    Network Security
        All namespaces should have NetworkPolicy. Interactions between namespaces should be limited to NetworkPolicy following least privileges principles (Inspektor Gadget).
        It is recommended to use authentication and authorization between all application microservices (Istio, Linkerd, Consul).
        The interfaces of the cluster components and infrastructure tools should not be published on the Internet.
        Infrastructure services, control plane, and data storage should be located in a separate VLAN on isolated nodes.
        External user traffic passing into the cluster should be inspected using WAF.
        It is recommended to separate the cluster nodes interacting with the Internet (DMZ) from the cluster nodes interacting with internal services. Delimitation can be within one cluster, or within two different clusters (DMZ and VLAN).
    Secure configuration of workloads
        Do not run pods under the root account - UID 0.
        Set runAsUser parameter for all applications.
        Set allowPrivilegeEscalation - false.
        Do not run the privileged pod (privileged: true).
        It is recommended to set readonlyRootFilesystem - true.
        Do not use hostPID and hostIPC.
        Do not use hostNetwork.
        Do not use unsafe system calls (sysctl):
            kernel.shm *,
            kernel.msg *,
            kernel.sem,
            fs.mqueue. *,
        Do not use hostPath.
        Use CPU / RAM limits. The values should be the minimum for the containerized application to work.
        Capabilities should be set according to the principle of least privileges (drop 'ALL', after which all the necessary capacities for the application to work are enumerated, while it is prohibited to use:
            CAP_FSETID,
            CAP_SETUID,
            CAP_SETGID,
            CAP_SYS_CHROOT,
            CAP_SYS_PTRACE,
            CAP_CHOWN,
            CAP_NET_RAW,
            CAP_NET_ADMIN,
            CAP_SYS_ADMIN,
            CAP_NET_BIND_SERVICE)
        Do not use the default namespace (default).
        The application should have a seccomp, apparmor or selinux profile according to the principles of least privileges (Udica, Oci-seccomp-bpf-hook, Go2seccomp, Security Profiles Operator).
        Workload configuration should be audited regularly (Kics, Kubeaudit, Kubescape, Conftest, Kubesec, Checkov)
    Secure image development
        Do not use RUN construct with sudo.
        COPY is required instead of ADD instruction.
        Do not use automatic package update via apt-get upgrade, yum update, apt-get dist-upgrade.
        It is necessary to explicitly indicate the versions of the installed packages. The SBOM building tools (Syft) can be used to determine the list of packages.
        Do not store sensitive information (passwords, tokens, certificates) in the Dockerfile.
        The composition of the packages in the container image should be minimal enough to work.
        The port range forwarded into the container should be minimal enough to work.
        It is not recommended to install wget, curl, netcat inside the production application image and container.
        It is recommended to use dockerignore to prevent putting sensitive information inside the image.
        It is recommended to use a minimum number of layers using a multi-stage build.
        It is recommended to use WORKDIR as an absolute path. It is not recommended to use cd instead of WORKDIR.
        It is recommended to beware of recursive copying using COPY . ..
        It is recommended not to use the latest tag.
        When downloading packages from the Internet during the build process, it is recommended to check the integrity of these packages.
        Do not run remote control tools in a container.
        Based on the results of scanning Docker images, an image signature should be generated, which will be verified before deployment (Notary, Cosign).
        Dockerfile should be checked during development by automated scanners (Kics, Hadolint, Conftest).
        All images should be checked in the application lifecycle by automated scanners (Trivy, Clair, Grype).
        Build secure CI and CD as same as suply chain process (SLSA)
