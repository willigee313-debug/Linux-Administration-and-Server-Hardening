# Flashcards — Virtualization & Containers

Spaced-repetition cards for the Virtualization and Containers modules (RHCSA/LFCS/Linux+/LPIC-1 prep), covering KVM/QEMU/libvirt, VirtualBox, LXC/LXD, and the Docker/Podman/Kubernetes container stack.

## Hypervisors & KVM Fundamentals

What is the key architectural difference between a Type-1 and a Type-2 hypervisor?::Type-1 (bare-metal) runs directly on hardware with no general-purpose host OS underneath (KVM, ESXi, Hyper-V); Type-2 (hosted) runs as an application on top of a host OS (VirtualBox, VMware Workstation).

Which flags in /proc/cpuinfo indicate Intel VT-x and AMD-V hardware virtualization support?::`vmx` for Intel VT-x, `svm` for AMD-V.

Which kernel character device does QEMU open to request hardware-accelerated VM creation via KVM?::/dev/kvm

What is the division of labor between QEMU and KVM in a KVM-accelerated guest?::KVM (kvm.ko + kvm-intel/kvm-amd.ko) supplies kernel-level hardware-assisted CPU virtualization; QEMU supplies userspace device emulation (disk, NIC, chipset, GPU) for the guest.

How do you confirm a running libvirt domain is actually using KVM hardware acceleration and not falling back to software emulation?::Check `virsh dumpxml <domain>` for `<domain type="kvm">`; `type="qemu"` means pure software (TCG) emulation.

## Nested Virtualization

What modprobe option enables nested KVM virtualization on Intel and AMD hosts respectively?::`options kvm-intel nested=1` (Intel) and `options kvm-amd nested=1` (AMD), typically in /etc/modprobe.d/kvm.conf.

Which libvirt guest CPU mode is the simplest way to pass VMX/SVM extensions through to an L1 guest for nested virtualization?::`<cpu mode='host-passthrough'>` (CLI equivalent: `-cpu host`).

## libvirt / virsh Management

What virsh command lists all domains, including those that are defined but stopped?::virsh list --all

What is the default libvirt connection URI for host-wide (privileged) management versus per-user unprivileged VMs?::qemu:///system (privileged, root libvirtd) vs qemu:///session (unprivileged, per-user).

What virsh command flattens an external snapshot overlay chain back into the base disk after validating a change?::virsh blockcommit <domain> <disk> --active --pivot

## Snapshots, Storage & Networking

Why should internal qcow2 snapshots generally be avoided on a running (live) libvirt domain in modern setups?::Modern libvirt/QEMU no longer supports live internal disk snapshots well; use external (disk-only) snapshots for anything live instead.

What qemu-img command creates a copy-on-write linked-clone overlay backed by a read-only golden image?::qemu-img create -f qcow2 -F qcow2 -b golden-image.qcow2 clone.qcow2

What distinguishes a libvirt "isolated" virtual network from the default NAT network in its XML definition?::An isolated network has no `<forward>` element at all, so guests have no route out; NAT networks have `<forward mode="nat">`.

In libvirt storage pools, what is the difference between a volume's "capacity" and its "allocation"?::Capacity is the logical/virtual size; allocation is the actual bytes consumed on the backing store — thin-provisioned volumes report capacity, not real usage, in `virsh vol-info`.

Which VBoxManage networking mode fully isolates a pair of VMs from both the host and the physical LAN?::Internal Network (`VBoxManage modifyvm <vm> --nic1 intnet --intnet1 <name>`) — even the host itself cannot reach VMs on it.

## LXC & LXD

In LXC, what is the difference between a privileged and an unprivileged container with respect to UID 0?::In a privileged container, UID 0 inside equals UID 0 on the host; in an unprivileged container, a user namespace maps container UID 0 to an unprivileged host UID via /etc/subuid and /etc/subgid.

What is the difference between an LXD "container" instance and an LXD "virtual-machine" instance?::A container shares the host kernel (namespaces/cgroups); a virtual-machine (`lxc launch --vm`) gets its own kernel via QEMU/KVM.

## Container Fundamentals

What four Linux kernel primitives make containers possible?::Namespaces (isolate what a process can see), cgroups (limit what it can consume), union/overlay filesystem (layered storage), and capabilities + seccomp + LSM (restrict what it's allowed to do).

What is the difference between a Docker image and a Docker container?::An image is an immutable, read-only layered template; a container is a running (or stopped) instance of that image plus a thin writable layer.

Which OCI specification defines the HTTP API a container registry must implement?::distribution-spec

## Docker Engine, Lifecycle & Images

Why is membership in the local `docker` group considered root-equivalent on the host?::A docker group member can run something like `docker run -v /:/hostroot -it alpine chroot /hostroot sh`, mounting the entire host filesystem and gaining root — the daemon itself runs as root.

Which /etc/docker/daemon.json key disables inter-container communication on the default bridge network?::"icc": false

What Docker container exit code typically indicates the process was OOM-killed?::137 (SIGKILL, often triggered by exceeding the --memory limit).

Which docker run flag caps the number of PIDs a container may create, blunting fork-bomb attacks?::--pids-limit

Why is storing a secret via ENV or ARG in a Dockerfile a security risk?::The secret persists in the image's layer history and remains visible via docker inspect/docker history even if a later RUN "deletes" it.

What Dockerfile technique keeps compilers and build tooling out of the final runtime image?::Multi-stage builds — using COPY --from=<stage> to copy only the compiled artifact into a slim final stage.

Which Docker network type provides automatic embedded DNS resolution between containers by name?::A user-defined bridge network (the default `bridge` network does not register container names for DNS).

Which Docker mount type stores data only in host RAM and is wiped when the container stops?::tmpfs mount

## Podman, Rootless & Buildah

What is the fundamental architectural difference between Podman and Docker?::Podman is daemonless (each container is a fork/exec child monitored by conmon); Docker relies on a persistent, root-owned background daemon (dockerd).

What kernel mechanism allows a rootless container's UID 0 to map to an unprivileged host UID?::User namespaces, configured via delegated ranges in /etc/subuid and /etc/subgid.

What Buildah command creates a mutable "working container" from a base image that you can inspect and modify before committing?::buildah from <image>

## Container Security & Escape

Why is running a container with --privileged equivalent to giving it root on the host?::It disables seccomp and AppArmor/SELinux confinement, grants all Linux capabilities, and allows access to all host devices under /dev.

Which Linux capability, combined with a writable cgroup v1 mount, enables the classic release_agent container breakout technique?::CAP_SYS_ADMIN

Why should /var/run/docker.sock never be bind-mounted into an untrusted container?::It grants full control of the Docker daemon, which runs as root — equivalent to unrestricted host root access.

## Kubernetes Basics

What is the smallest deployable unit in Kubernetes?::A Pod — one or more containers sharing a network namespace and storage volumes.

Which Kubernetes control-plane component is the only one that talks directly to etcd?::kube-apiserver

Why are Kubernetes Secret objects not a substitute for a dedicated secrets manager by default?::They are base64-encoded, not encrypted, unless encryption-at-rest is explicitly configured for etcd.

## Related

- [Virtualization](../Virtualization/Readme.md)
- [Containers](../Containers/Readme.md)
- [Exam Preparation](Readme.md)
- [Linux Administration & Server Hardening](../Readme.md)
