# OpenShift MicroShift Installation on RHEL 9.2

This guide provides a comprehensive walkthrough for installing **Red Hat OpenShift MicroShift** on RHEL 9.2. MicroShift is a lightweight version of OpenShift designed for edge computing and low-resource environments (VMs), providing a full OpenShift experience (`oc` CLI, Routes, Over-the-air updates) with minimal overhead.

## 📋 Prerequisites

* **System:** RHEL 9.2 with an active Red Hat Subscription.
* **RAM:** Minimum 4GB (5GB+ recommended).
* **Disk:** 20GB+ available space.

---

## 🛠️ Installation Steps

### 1. Enable Repositories & Install Tools

```bash
[ali_wazeer@vbox ~]$ sudo subscription-manager repos --enable=fast-datapath-for-rhel-9-$(arch)-rpms
[ali_wazeer@vbox ~]$ sudo dnf install -y microshift openshift-clients

```

### 2. Configure the Pull Secret (Crucial Step)

MicroShift requires authorization to pull container images from the Red Hat Registry.

1. Download your secret from: [Red Hat Hybrid Cloud Console](https://console.redhat.com/openshift/install/pull-secret).
2. Create the configuration file:

```bash
[ali_wazeer@vbox ~]$ sudo mkdir -p /etc/crio
[ali_wazeer@vbox ~]$ sudo nano /etc/crio/openshift-pull-secret
# Paste your secret here and save (Ctrl+O, Enter, Ctrl+X)

[ali_wazeer@vbox ~]$ sudo chown root:root /etc/crio/openshift-pull-secret
[ali_wazeer@vbox ~]$ sudo chmod 600 /etc/crio/openshift-pull-secret

```

### 3. Start Services & Configure Firewall

```bash
[ali_wazeer@vbox ~]$ sudo systemctl enable --now microshift
[ali_wazeer@vbox ~]$ sudo firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
[ali_wazeer@vbox ~]$ sudo firewall-cmd --permanent --zone=trusted --add-source=169.254.169.1
[ali_wazeer@vbox ~]$ sudo firewall-cmd --reload

```

### 4. Setup Cluster Access (Kubeconfig)

Wait about 5 minutes for MicroShift to initialize, then run:

```bash
[ali_wazeer@vbox ~]$ mkdir -p ~/.kube
[ali_wazeer@vbox ~]$ sudo cp /var/lib/microshift/resources/kubeadmin/kubeconfig ~/.kube/config
[ali_wazeer@vbox ~]$ sudo chown $(id -u):$(id -g) ~/.kube/config

```

### 5. Monitor Cluster Status

```bash
[ali_wazeer@vbox ~]$ watch oc get pods -A
[ali_wazeer@vbox ~]$ oc get nodes

```

---

## 🚀 Bonus: Deploy Your First App (Nginx)

### 1. Create a Project and Deployment

```bash
[ali_wazeer@vbox ~]$ oc new-project demo-web
[ali_wazeer@vbox ~]$ oc create deployment my-web --image=nginx

```

### 2. Expose the App to the Internet

```bash
[ali_wazeer@vbox ~]$ oc expose deployment my-web --port=80
[ali_wazeer@vbox ~]$ oc expose service my-web
[ali_wazeer@vbox ~]$ oc get routes

```

### 3. Access from Host Browser (Windows/macOS)

1. Get the VM IP: `[ali_wazeer@vbox ~]$ hostname -I`
2. Update your Host's `hosts` file (e.g., `C:\Windows\System32\drivers\etc\hosts`) adding:
`YOUR_VM_IP my-web-demo-web.cluster.local`
3. Open `http://my-web-demo-web.cluster.local` in your browser.

---

## 🔍 Troubleshooting & Useful Commands

### Inspection

* **Check System Logs:** `[ali_wazeer@vbox ~]$ sudo journalctl -u microshift -f`
* **Check Service Status:** `[ali_wazeer@vbox ~]$ sudo systemctl status microshift`
* **View Pulled Images:** `[ali_wazeer@vbox ~]$ sudo crictl images`
* **Monitor System Resources:** `[ali_wazeer@vbox ~]$ top`

### Management

* **Restart Services:**
```bash
[ali_wazeer@vbox ~]$ sudo systemctl restart crio
[ali_wazeer@vbox ~]$ sudo systemctl restart microshift
```




* **Stop Services:**
```bash
[ali_wazeer@vbox ~]$ sudo systemctl stop microshift
[ali_wazeer@vbox ~]$ sudo systemctl stop crio
```






### File System & Identity

* **Locate Kubeconfig:** `[ali_wazeer@vbox ~]$ sudo find /var/lib/microshift -name kubeconfig`
* **Verify Access Rights:** `[ali_wazeer@vbox ~]$ sudo ls -l /var/lib/microshift/resources/kubeadmin/kubeconfig`

### Cluster Operations

* **Get Nodes:** `[ali_wazeer@vbox ~]$ oc get nodes`
* **Get All Pods:** `[ali_wazeer@vbox ~]$ oc get pods -A`
* **Describe Specific Pod:** `[ali_wazeer@vbox ~]$ oc describe pod <pod_name> -n <namespace>`

---

**Happy Orchestrating!** 🥳


<img width="1366" height="717" alt="Screenshot (883)" src="https://github.com/user-attachments/assets/ac2ed266-cc11-4f94-9916-244d0c7641ad" />

<img width="1366" height="368" alt="Screenshot (884)" src="https://github.com/user-attachments/assets/d7a23f0e-bd9a-4107-af05-86e114bc20b8" />
