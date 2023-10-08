# Doing it Manually

## Control-Plane Initialziation
- `sudo -i`
- `dnf -y install wget`
- `vim /etc/NetworkManager/conf.d/rke2-canal.conf`
    ```
    [keyfile]
    unmanaged-devices=interface-name:cali*;interface-name:flannel*
    ```
- `systemctl reload NetworkManager`
- `mkdir -p /etc/rancher/rke2`
- `mkdir -p /etc/rancher/rke2/config.yaml.d`
- `mkdir -p /var/lib/rancher/rke2/server/manifests`
- `mkdir -p /var/lib/rancher/rke2/agent/pod-manifests`
- `mkdir -p /var/lib/rancher/rke2/agent/images`
- `mkdir -p /etc/systemd/system`
- `mkdir -p /usr/local/bin`
- `wget -O /usr/local/bin/rke2-v1.28.2+rke2r1 https://github.com/rancher/rke2/releases/download/v1.28.2%2Brke2r1/rke2.linux-amd64`
- `mkdir -p /var/lib/rancher/rke2/agent/images`
- `wget -O /var/lib/rancher/rke2/agent/images/rke2-images-core.linux-amd64.tar.gz https://github.com/rancher/rke2/releases/download/v1.28.2%2Brke2r1/rke2-images-core.linux-amd64.tar.gz`
- `wget -O /var/lib/rancher/rke2/agent/images/rke2-images-cilium.linux-amd64.tar.gz https://github.com/rancher/rke2/releases/download/v1.28.2%2Brke2r1/rke2-images-cilium.linux-amd64.tar.gz`
- `wget -O /var/lib/rancher/rke2/agent/images/rke2-images-canal.linux-amd64.tar.gz https://github.com/rancher/rke2/releases/download/v1.28.2%2Brke2r1/rke2-images-canal.linux-amd64.tar.gz`
- Copy HelmChart Manifests to /var/lib/rancher/rke2/server/manifests if there are any
    - Cilium as an example
- Copy Pod Manifests to /var/lib/rancher/rke2/agent/pod-manifests if there are any
    - Kube-VIP as an example
- `ln /usr/local/bin/rke2-v1.28.2+rke2r1 /usr/local/bin/rke2`
- `ln /usr/local/bin/rke2-v1.28.2+rke2r1 /usr/local/bin/kubectl`
- `ln /usr/local/bin/rke2-v1.28.2+rke2r1 /usr/local/bin/crictl`
- `ln /usr/local/bin/rke2-v1.28.2+rke2r1 /usr/local/bin/ctr`
- `vim /etc/rancher/rke2/config.yaml`
    ```
    cluster-init: true
    node-ip: '10.10.111.224'
    write-kubeconfig-mode: '0644'
    tls-san:
    - '10.10.111.150'
    # Make a etcd snapshot every 2 hours
    etcd-snapshot-schedule-cron: ' */2 * * *'
    # Keep 56 etcd snapshorts (equals to 2 weeks with 6 a day)
    etcd-snapshot-retention: 56
    cni:
    - canal
    disable-kube-proxy: 'false'
    cluster-cidr: '10.42.0.0/16'
    service-cidr: '10.43.0.0/16'
    cluster-dns: '10.43.0.10'
    selinux: 'true'
    disable:
    - rke2-ingress-nginx
    kubelet-arg:
    - 'max-pods=100'
    - 'eviction-hard=memory.available<250Mi'
    - 'eviction-soft=memory.available<1Gi'
    - 'eviction-soft-grace-period=memory.available=2m'
    - 'kube-reserved=cpu=200m,memory=500Mi'
    - 'system-reserved=cpu=200m,memory=500Mi'
    ```
- `vim /etc/systemd/system/rke2.service`
    ```
    [Unit]
    Description=Lightweight Kubernetes
    Documentation=https://rke2.io
    Wants=network-online.target
    After=network-online.target

    [Service]
    Type=notify
    ExecStartPre=-/sbin/modprobe br_netfilter
    ExecStartPre=-/sbin/modprobe overlay
    ExecStart=/usr/local/bin/rke2 server --config /etc/rancher/rke2/config.yaml 
    KillMode=process
    Delegate=yes
    LimitNOFILE=1048576
    LimitNPROC=infinity
    LimitCORE=infinity
    TasksMax=infinity
    TimeoutStartSec=0
    Restart=always
    RestartSec=5s

    [Install]
    WantedBy=multi-user.target
    ```
- `vim /usr/local/bin/rke2-uninstall.sh`
    ```
    #!/bin/sh
    set -x
    [ $(id -u) -eq 0 ] || exec sudo $0 $@

    /usr/local/bin/rke2-killall.sh

    if which systemctl; then
        systemctl disable rke2
        systemctl reset-failed rke2
        systemctl daemon-reload
    fi
    if which rc-update; then
        rc-update delete rke2 default
    fi

    rm -f /etc/systemd/system/rke2.service

    remove_uninstall() {
        rm -f /usr/local/bin/rke2-uninstall.sh
    }
    trap remove_uninstall EXIT

    if (ls /etc/systemd/system/rke2*.service || ls /etc/init.d/rke2*) >/dev/null 2>&1; then
        set +x; echo 'Additional rke2 services installed, skipping uninstall of rke2'; set -x
        exit
    fi

    for cmd in kubectl crictl ctr; do
        if [ -L /usr/local/bin/$cmd ]; then
            rm -f /usr/local/bin/$cmd
        fi
    done

    for bin in /usr/local/bin/rke2*; do
        if [ -f "${bin}" ]; then
            rm -f "${bin}"
        fi
    done

    rm -rf /etc/rancher/rke2
    rm -rf /run/rke2
    rm -rf /run/flannel
    rm -rf /var/lib/rancher/rke2
    rm -rf /var/lib/kubelet
    rm -f /usr/local/bin/rke2-killall.sh

    if type yum >/dev/null 2>&1; then
        yum remove -y rke2-selinux
        rm -f /etc/yum.repos.d/rancher-rke2-common*.repo
    fi
    ```
- `vim /usr/local/bin/rke2-killall.sh`
    ```
    #!/bin/sh

    [ $(id -u) -eq 0 ] || exec sudo $0 $@

    for bin in /var/lib/rancher/rke2/data/**/bin/; do
        [ -d $bin ] && export PATH=$PATH:$bin:$bin/aux
    done

    set -x

    for service in /etc/systemd/system/rke2*.service; do
        [ -s $service ] && systemctl stop $(basename $service)
    done

    for service in /etc/init.d/rke2*; do
        [ -x $service ] && $service stop
    done

    pschildren() {
        ps -e -o ppid= -o pid= | \
        sed -e 's/^\s*//g; s/\s\s*/\t/g;' | \
        grep -w "^$1" | \
        cut -f2
    }

    pstree() {
        for pid in $@; do
            echo $pid
            for child in $(pschildren $pid); do
                pstree $child
            done
        done
    }

    killtree() {
        kill -9 $(
            { set +x; } 2>/dev/null;
            pstree $@;
            set -x;
        ) 2>/dev/null
    }

    getshims() {
        ps -e -o pid= -o args= | sed -e 's/^ *//; s/\s\s*/\t/;' | grep -w 'rke2/data/[^/]*/bin/containerd-shim' | cut -f1
    }

    killtree $({ set +x; } 2>/dev/null; getshims; set -x)

    do_unmount_and_remove() {
        awk -v path="$1" '$2 ~ ("^" path) { print $2 }' /proc/self/mounts | sort -r | xargs -r -t -n 1 sh -c 'umount "$0" && rm -rf "$0"'
    }

    do_unmount_and_remove '/run/rke2'
    do_unmount_and_remove '/var/lib/rancher/rke2'
    do_unmount_and_remove '/var/lib/kubelet/pods'
    do_unmount_and_remove '/var/lib/kubelet/plugins'
    do_unmount_and_remove '/run/netns/cni-'

    # Remove CNI namespaces
    ip netns show 2>/dev/null | grep cni- | xargs -r -t -n 1 ip netns delete

    # Delete network interface(s) that match 'master cni0'
    ip link show 2>/dev/null | grep 'master cni0' | while read ignore iface ignore; do
        iface=${iface%%@*}
        [ -z "$iface" ] || ip link delete $iface
    done
    ip link delete cni0
    ip link delete flannel.1
    rm -rf /var/lib/cni/
    iptables-save | grep -v KUBE- | grep -v CNI- | iptables-restore
    ```
- `chmod +x /usr/local/bin/*`
- `wget -O rke2-selinux-0.15-1.el9.noarch.rpm /root/ https://github.com/rancher/rke2-selinux/releases/download/v0.15.latest.1/rke2-selinux-0.15-1.el9.noarch.rpm`
- `dnf -y install /root/rke2-selinux-0.15-1.el9.noarch.rpm`
- `restorecon /usr/local/bin/rke2`
- `systemctl daemon-reload`
- `systemctl enable --now rke2.service`