# Openshift-Installation


---

### 🚀 دليل تسطيب OpenShift MicroShift على RHEL 9.2

ما هو MicroShift؟

هو نسخة خفيفة جداً من OpenShift مخصصة لأجهزة الـ Edge والـ VMs الضعيفة، بيقدم لك تجربة OpenShift كاملة (oc command, Routes, Over-the-air updates) بأقل إمكانيات.

#### 1️⃣ المتطلبات الأساسية (Prerequisites)

- نظام **RHEL 9.2** مفعل بـ Subscription.
    
- رامات 4GB على الأقل (يفضل 5GB+).
    
- مساحة هارد 20GB+.
#### 1️⃣ تفعيل المستودعات وتحميل الأدوات

Bash

```
[ali_wazeer@vbox ~]$ sudo subscription-manager repos --enable=fast-datapath-for-rhel-9-$(arch)-rpms
[ali_wazeer@vbox ~]$ sudo dnf install -y microshift openshift-clients
```

#### 2️⃣ إعداد الـ Pull Secret (مفتاح الدخول لـ Red Hat Registry)

بعد ما تجيب السيكرت من موقع Red Hat Console:
#### 2️⃣ إعداد الـ Pull Secret (الخطوة الأهم)

المايكروشيفت مش هيعرف يسحب صور الحاويات من غير "تصريح" من Red Hat:
حمل السيكرت من https://console.redhat.com/openshift/install/pull-secret
أنشئ الملف وحط فيه السيكرت:
Bash

```
[ali_wazeer@vbox ~]$ sudo mkdir -p /etc/crio
[ali_wazeer@vbox ~]$ sudo nano /etc/crio/openshift-pull-secret
# (هنا بنحط السيكرت ونعمل Save)

[ali_wazeer@vbox ~]$ sudo chown root:root /etc/crio/openshift-pull-secret
[ali_wazeer@vbox ~]$ sudo chmod 600 /etc/crio/openshift-pull-secret
```

#### 3️⃣ تشغيل الخدمة وفتح الـ Firewall

Bash

```
[ali_wazeer@vbox ~]$ sudo systemctl enable --now microshift
[ali_wazeer@vbox ~]$ sudo firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
[ali_wazeer@vbox ~]$ sudo firewall-cmd --permanent --zone=trusted --add-source=169.254.169.1
[ali_wazeer@vbox ~]$ sudo firewall-cmd --reload
```

#### 4️⃣ ربط الـ `oc` command بالكلاستر (بعد انتظار 5 دقائق للتحميل)

بمجرد ما المايكروشيفت يكريت ملفات الإعدادات:

Bash

```
[ali_wazeer@vbox ~]$ mkdir -p ~/.kube
[ali_wazeer@vbox ~]$ sudo cp /var/lib/microshift/resources/kubeadmin/kubeconfig ~/.kube/config
[ali_wazeer@vbox ~]$ sudo chown $(id -u):$(id -g) ~/.kube/config
```

#### 5️⃣ متابعة حالة الكلاستر (اللحظات الحاسمة)

هنا بنراقب الصور وهي بتنزل والـ Pods وهي بتقوم:

Bash

```
[ali_wazeer@vbox ~]$ watch oc get pods -A
[ali_wazeer@vbox ~]$ oc get nodes
```

![[Screenshot (884) 1.png]]
---

### 💡 أهم النصائح للنجاح:

- **لو الملف مظهرش:** استخدم `sudo journalctl -u microshift -f` عشان تشوف هو بيحمل في إيه دلوقتي.
- **التأكد من الصور:** ممكن تستخدم `sudo crictl images` عشان تعرف هو نزل كام صورة لحد دلوقتي.
![[Screenshot (883).png]]
---

#### 1️⃣ إنشاء مشروع جديد وتطبيق Nginx

Bash

```
[ali_wazeer@vbox ~]$ oc new-project demo-web
[ali_wazeer@vbox ~]$ oc create deployment my-web --image=nginx
```

#### 2️⃣ متابعة التطبيق لحد ما يبقى Running

Bash

```
[ali_wazeer@vbox ~]$ oc get pods -w
# (استنى لحد ما تلاقي الحالة بقت 1/1 Running واضغط Ctrl+C)
```

#### 3️⃣ تحويل التطبيق لخدمة (Service) وعمل Route (الرابط السحري)

Bash

```
[ali_wazeer@vbox ~]$ oc expose deployment my-web --port=80
[ali_wazeer@vbox ~]$ oc expose service my-web
```

#### 4️⃣ الحصول على رابط الموقع (URL)

Bash

```
[ali_wazeer@vbox ~]$ oc get routes
```

> المخرجات هتكون حاجة شبه كدة: > NAME HOST/PORT PATH SERVICES PORT TERMINATION WILDCARD
> 
> my-web my-web-demo-web.cluster.local my-web 80 None

#### 5️⃣ إزاي تفتحه من متصفح الويندوز (أو الجهاز الـ Host)

بما إننا شغالين على VirtualBox، جهازك الـ Host ميعرفش مين هو `my-web-demo-web.cluster.local`. لازم نساعده:

1. اعرف IP المكنة الـ Linux:
Bash

```
[ali_wazeer@vbox ~]$ hostname -I
```

2. في الويندوز، افتح الـ Notepad بـ Administrator وافتح ملف الـ hosts في المسار:
    
    C:\Windows\System32\drivers\etc\hosts
    
3. ضيف السطر ده في الآخر (استبدل الـ IP بـ IP مكنتك):
    
    192.168.x.x my-web-demo-web.cluster.local
    

**الآن افتح المتصفح واكتب الرابط.. ومبروك عليك صفحة "Welcome to nginx" من قلب الـ MicroShift!** 🥳

---
### 1. أوامر الفحص والـ Troubleshooting (عشان تعرف العطل فين)

دي الأوامر اللي استخدمناها لما كان الملف مش راضي يظهر:

- **عشان تشوف "القلب" بتاع المايكروشيفت بيقول إيه (Logs):**
    
Bash

```
[ali_wazeer@vbox ~]$ sudo journalctl -u microshift -f
```

- **عشان تشوف حالة الخدمة نفسها هل هي Running ولا Failed:**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo systemctl status microshift
```

- **عشان تشوف الصور اللي بتتحمل من النت وحجمها (CRI-O):**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo crictl images
```

- **عشان تشوف استهلاك الرامات والبروسيسور (عشان الـ VM متهنجش):**
    

Bash

```
[ali_wazeer@vbox ~]$ top
```

---

### 2. أوامر الإصلاح والـ Restart

لما كنا بنحب "نزق" الكلاستر عشان تلقط التعديلات:

- **عشان ترستر المحرك بتاع الحاويات (CRIO) والمايكروشيفت:**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo systemctl restart crio
[ali_wazeer@vbox ~]$ sudo systemctl restart microshift
```

- **عشان توقفهم خالص لو حبيت تعدل حاجة:**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo systemctl stop microshift
[ali_wazeer@vbox ~]$ sudo systemctl stop crio
```

---

### 3. أوامر البحث والملفات (الـ File System)

دي اللي استخدمناها عشان نلاقي ملف الـ `kubeconfig` التاييه:

- **عشان تدور على مكان الملف في السيستم كله:**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo find /var/lib/microshift -name kubeconfig
```

- **عشان تتأكد إن الملف موجود في مسار معين:**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo ls -l /var/lib/microshift/resources/kubeadmin/kubeconfig
```

- **عشان تقرأ محتوى الـ Pull Secret وتتأكد إنه مكتوب صح:**
    

Bash

```
[ali_wazeer@vbox ~]$ sudo cat /etc/crio/openshift-pull-secret
```

---

### 4. أوامر الـ `oc` الأساسية (التعامل مع الكلاستر)

دي اللي بنستخدمها بعد ما الكلاستر بتقوم:

- **عشان تشوف حالة النود وهل هي Ready ولا لأ:**
    

Bash

```
[ali_wazeer@vbox ~]$ oc get nodes
```

- **عشان تشوف كل الـ Pods اللي شغالة في الكلاستر كلها (حتى السيستم):**
    

Bash

```
[ali_wazeer@vbox ~]$ oc get pods -A
```

- **عشان تراقب التغييرات لحظة بلحظة (زي ما عملنا في الـ Pods):**
    
![[Screenshot (884).png]]
Bash

```
[ali_wazeer@vbox ~]$ watch oc get pods -A
```

- **عشان تعرف تفاصيل أكتر عن Pod معين لو فيه مشكلة (زي ما عملنا مع ovn):**
    

Bash

```
[ali_wazeer@vbox ~]$ oc describe pod <اسم-البود> -n <النام-سبيس>
```

