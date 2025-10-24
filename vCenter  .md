تمام إسلام، اللي عندك ده نص كامل لمحاضرة بيشرح **إزاي تعمل Install و Configure vCenter Server** في بيئة VMware vSphere، والخطوات دي تعتبر من أهم وأطول مراحل الكورس.

علشان أظبطه ليك بالصور والترتيب زي ما طلبت، خلينا ننظمه في ٤ مراحل أساسية مع شروحات ورسومات توضيحية لكل مرحلة (هعملها لك بالصور لما تأكد):

---

### 🧩 المرحلة 1: الفكرة العامة

**الهدف:**
تقدر تدير كل الـ ESXi Hosts والـ Virtual Machines من مكان واحد اسمه **vCenter Server**.

**السيناريو:**

* عندك أكتر من سيرفر (ESXi 1, ESXi 2, ESXi 3...)
* كل واحد فيهم عليه Virtual Machines كتير
* بدل ما تدخل على كل واحد لوحده، بتجمعهم في **vCenter** وتديرهم من Dashboard واحدة

**الفايدة:**

* Centralized Management
* Live Migration (vMotion)
* Resource Balancing (DRS)
* HA (High Availability)

📸 الصورة اللي هنعملها:
رسم يوضح 3 سيرفرات ESXi متوصلين بـ vCenter Server واحد، والـ vCenter متثبت على Windows Server.

---

### ⚙️ المرحلة 2: المتطلبات قبل التثبيت

**قبل ما تعمل Install لازم تتأكد من:**

1. الجهاز اللي هيتثبت عليه vCenter يكون عليه **Windows Server (2016 أو 2019)**
2. يكون عندك **ESXi Host** عليه مساحة كفاية (RAM ≥ 10GB)
3. تعمل **DataStore** جديدة على الـ ESXi بمساحة 250–300GB
4. يكون عندك **DNS Server** جاهز وتضيف فيه Record خاص بالـ vCenter الجديد

   * اسم السيرفر (مثلاً: `mega1.it.local`)
   * IP Address (مثلاً: `192.168.1.260`)
5. الـ Windows Server يكون Domain Controller أو على الأقل Join Domain
6. تتأكد من الـ IPs والدومين مكتوبين عندك على ورقة

📸 الصور اللي هنعملها:

* رسم يوضح علاقة DNS وvCenter وESXi
* صورة DataStore جديدة
* مثال على Record في DNS Manager

---

### 🧱 المرحلة 3: تجهيز البيئة

**الخطوات العملية:**

1. افتح الـ ESXi
2. لو ما فيش مساحة كافية → اعمل Maintenance Mode → ضيف Hard Disk جديد
3. بعدين اعمل Datastore جديدة على الـ Hard ده
4. بعد كده اطلع من Maintenance Mode

📸 الصور اللي هنعملها:

* خطوات إضافة Hard Disk في vSphere
* Maintenance Mode
* إنشاء DataStore جديدة

---

### 💿 المرحلة 4: تثبيت vCenter Server

**السوفتوير:**
`VMware-VCSA.iso`

**خطوات التثبيت:**

1. شغل ملف الـ ISO على Windows Server
2. اختار **Install**
3. هتبدأ عملية من مرحلتين:

   * **Stage 1:** Deploy the vCenter Appliance
   * **Stage 2:** Configure the vCenter
4. خلال التثبيت هيسألك عن:

   * الـ ESXi Host اللي هينزل عليه
   * الـ Datastore
   * الـ Network + IP + DNS
   * اسم السيرفر (FQDN اللي أضفته في DNS)
5. بعد الانتهاء تقدر تدخل من المتصفح على:

   ```
   https://<vcenter-ip-or-fqdn>:5480
   ```
6. أو على vSphere Client:

   ```
   https://<vcenter-fqdn>/ui
   ```

📸 الصور اللي هنعملها:

* Installer UI
* خطوات Stage 1 و Stage 2
* واجهة الدخول للـ vCenter بعد التثبيت

---

### 🔍 النتيجة النهائية

يبقى عندك هيكل بالشكل ده:

```
[Windows Server + DNS] 
        ↓

     vCenter Server
        ↓
  ┌───────────────┬───────────────┬───────────────┐
  │ ESXi #1       │ ESXi #2       │ ESXi #3       │
  │ VM1..VMn      │ VM1..VMn      │ VM1..VMn      │
  └───────────────┴───────────────┴───────────────┘
```

---

دلوقتي سؤال بسيط عشان أبدأ أجهز الصور والرسم:

هل تحب الصور تكون **واقعية (سكرينشوت من vSphere وWindows DNS)**
ولا تكون **رسومات توضيحية (Diagram ملون ومنظم زي Presentation)**؟









تمام اسلام
اللي انت كاتبه ده هو نص المحاضرة التانية الخاصة بتثبيت **vCenter Server على ESXi** خطوة بخطوة
خليني أعمل لك تنظيم وشرح عملي بالصور عشان تذاكرها كأنها كورس مصور 👇

---

## 🎥 المحاضرة التانية: تثبيت vCenter Server

---

### 🧩 المرحلة الأولى: بعد انتهاء Setup

بعد ما يخلص الـ Setup الأول (اللي خد تقريبًا نص ساعة)، نضغط
**Continue → Next**

---

### ⏰ إعداد التوقيت (Time)

بيسألك هل عايز السيرفر ياخد التوقيت من ESXi ولا تحدده يدوي؟
اختار
✅ **Synchronize time with the ESXi host**

عشان كله يكون بنفس التوقيت.

📸 **صورة توضيحية:**

* شاشة اختيار التوقيت
* خانة فيها خيار “Use the host time” متعلم عليه صح

---

### 🔐 إعداد SSO (Single Sign-On)

ده أهم جزء.

اللي بتكتبه هنا هو اللي هتستخدمه بعد كده لما تدخل من المتصفح.
يعني لما تفتح
`https://<vcenter_ip_or_fqdn>`
هيطلب منك Username و Password
هم دول اللي بتعملهم هنا.

💡 خليه مثلاً:

```
Domain: vsphere.local
Username: administrator
Password: Egypt@123
```

📸 **صورة توضيحية:**

* شاشة فيها domain name, username, password
* توضيح سهم على كلمة vsphere.local

---

### 📋 Summary + Install

هتجيلك شاشة فيها كل الإعدادات اللي اخترتها.
اضغط:
✅ **Finish**

بعد كده يبدأ يكمل الخطوة التانية من التثبيت.

🕒 خد حوالي نص ساعة تانية.

📸 **صورة توضيحية:**

* شاشة summary
* زر finish بالأسفل

---

### 🌐 الدخول على vCenter من المتصفح

بعد ما السيرفر يخلص التثبيت، بيعرضلك عنوان الدخول
زي مثلاً:

```
https://mega1.it.local
```

أو بالـ IP

```
https://192.168.1.140
```

افتحه في المتصفح (يفضل Firefox أو Chrome).

لو ظهرت رسالة تحذير SSL، دوس على
✅ Advanced → Proceed anyway

📸 **صورة توضيحية:**

* المتصفح بيعرض تحذير "Your connection is not private"
* سهم على زر “Proceed to site”

---

### 🔑 تسجيل الدخول

هتدخل بالـ credentials اللي عملتها قبل:

```
User: administrator@vsphere.local
Pass: Egypt@123
```

بعدها هتدخل على واجهة vCenter Web Client.

📸 **صورة توضيحية:**

* شاشة login عليها vSphere Client logo
* سهم على مكان إدخال اليوزر والباسورد

---

### 💻 أول واجهة للـ vCenter

دي الواجهة اللي هتستخدمها في كل الشغل الجاي.
منها تدير الـ Clusters, Hosts, VMs, Networks, Datastores.

📸 **صورة توضيحية:**

* واجهة vCenter Client بعد الدخول
* القائمة الجانبية فيها (Hosts and Clusters / VMs / Networking)

---

### ⚙️ واجهة الإدارة التانية (Port 5480)

لو دخلت كده:

```
https://mega1.it.local:5480
```

هتفتح صفحة إدارة السيرفر نفسه.
مش الواجهة اللي بتدير منها الـ VMs، دي بتدير حالة السيرفر.

📸 **صورة توضيحية:**

* Dashboard فيها CPU, Memory, Database status
* أزرار Restart / Shutdown

---

### 🧠 ملاحظات مهمة

1. ما تعملش Shutdown من داخل واجهة vCenter نفسها
   الأفضل تعملها من صفحة 5480 لأنها أكثر أمانًا.
2. لو السيرفر بطئ أول مرة، عادي جدًا.
   استنى لحد ما كل الخدمات تبقى باللون الأخضر.
3. لما تكون كل الخدمات OK، ساعتها تقدر تدخل عادي على الواجهة التانية.

📸 **صورة توضيحية:**

* كل الـ Services عليها لون أخضر “Running”

---

### 🔄 Shutdown & Power On

* لما تعمل Shutdown، الـ vCenter بيتقفل
* علشان تشغله تاني:

  * افتح من ESXi Host
  * اعمل Right click على المكنة
  * اختار “Power On”
  * استنى لحد ما ترجع الخدمات كلها online

📸 **صورة توضيحية:**

* شاشة vSphere فيها Virtual Machine → Power → Power On

---

### ✅ خلاصة

* خلصنا تنصيب vCenter بالكامل
* جهزنا الدخول من المتصفح
* عرفنا إدارة السيرفر (port 5480)
* عرفنا طريقة تشغيله وإيقافه بأمان

المحاضرة الجاية هنبدأ نضيف الـ ESXi Hosts ونعمل تنظيم للـ Datacenter.

---

تحب أعمل لك الصور فعليًا وأظبطها واحدة واحدة بنفس ترتيب الشرح دا؟
لو وافقت، هجهزها ليك كأنها slides تعليمية بالشرح المصري جوه كل صورة


تمام يا إسلام، دي الخطوات بالترتيب الصح لتثبيت **VMware vCenter Server Appliance (VCSA)** بالصور 👇
خلينا نمشي خطوة بخطوة كأنك فعلاً بتسطبها على جهازك

---

### **📍 الخطوة 1: شاشة البداية – Start Installation**

![Step 1](https://www.nakivo.com/wp-content/uploads/2016/10/starting_stage_1_of_vmware_vcenter_server_installation.webp)

هنا بتفتح الـ Installer وتختار **Install**
دي أول مرحلة في التثبيت
هيقولك إن العملية بتتم على مرحلتين

* المرحلة 1: نشر (Deploy) الـ vCenter Appliance
* المرحلة 2: إعداد (Setup) البيانات والتكوين

اضغط **Next** وكمّل

---

### **📍 الخطوة 2: إعداد الـ SSO (Single Sign-On)**

![Step 2](https://www.nakivo.com/blog/wp-content/uploads/2023/03/vmware_vcenter_sso_configuration_at_stage_1_of_deployment.webp)

المرحلة دي خاصة بالنظام اللي بتدخل منه بعدين
هتختار اسم الدومين الداخلي مثل
`vsphere.local`
وتكتب اليوزر والباسورد الأساسي اللي هتستخدمه بعدين
مثلاً
`administrator@vsphere.local`
خلي الباسورد قوي
وسجله عندك لأنك هتحتاجه بعد التثبيت

---

### **📍 الخطوة 3: إعداد الشبكة (Network Settings)**

![Step 3](https://www.nakivo.com/wp-content/uploads/2016/10/configuring_network_settings_for_vcsa.webp)

الجزء دا أساسي جدًا
هتختار:

* نوع الشبكة (Network)
* اسم الجهاز (FQDN)
* IP ثابت Static
* Subnet Mask
* Default Gateway
* DNS

تأكد الـ DNS معمول له Record للاسم اللي اخترته
لأن لو DNS مش مظبوط، التثبيت ممكن يفشل

---

### **📍 الخطوة 4: بدء المرحلة الثانية (Stage 2)**

![Step 4](https://www.nakivo.com/wp-content/uploads/2016/10/starting_stage_2_of_vcenter_server_installation.webp)

بعد ما الجهاز يتعمله Deploy، تبدأ المرحلة التانية
هنا بتعمل الإعدادات النهائية
هتظهرلك شاشة فيها:

* اسم النظام
* إعدادات الوقت (NTP)
* تفعيل SSH لو حبيت
* إعداد النسخ الاحتياطي

اضغط **Finish** وابدأ العملية

---

### **📍 الخطوة 5: شاشة التقدم (Deployment Progress)**

![Step 5](https://www.vgarethlewis.com/wp-content/uploads/2016/11/VCSA-6_5_Installation_09.png)

دلوقتي التثبيت بدأ فعليًا
هتشوف شريط تقدم بيتحرك
العملية ممكن تاخد من 15 إلى 30 دقيقة
خلي الجهاز شغال ومتقفلش المتصفح

---

### **📍 الخطوة 6: واجهة الدخول بعد التثبيت (Web Client Login)**

![Step 6](https://support.us.ovhcloud.com/hc/article_attachments/360015614880/connection_interface_w_html5.png)

بعد انتهاء التثبيت، افتح المتصفح وروح على
`https://<IP_or_FQDN>`

هتفتح لك شاشة الدخول دي
اكتب:

* Username: `administrator@vsphere.local`
* Password: اللي اخترته في الخطوة 2

وبكده تكون دخلت على لوحة إدارة vCenter 🎉

---

تحب أعملك نفس الترتيب دا في **PDF منسق بالصور والشرح العربي باللهجة المصرية** علشان تستخدمه كـ Manual وقت التطبيق؟

.
