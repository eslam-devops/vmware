كلامك ده إسلام هو شرح تفصيلي لإعدادات **vSwitch في VMware vSphere**
خليني أعمله ليك بنظام منظم بالصور + شرح مبسط بالمصري علشان تفهم كل اختيار وانت بتشوفه في الواجهة

---

## 🔧 إعدادات الـ vSwitch في VMware vSphere

أول ما تفتح إعدادات السويتش من خلال
**vSphere Client → Networking → vSwitch0 → Edit Settings**
هيظهر لك صفحة بالشكل ده 👇

📸 **صورة تقريبية**
![vSwitch Properties](https://www.nakivo.com/wp-content/uploads/2019/03/vmware-vswitch-security-settings.png)

---

### 1️⃣ قسم Security

هتلاقي فيه 3 اختيارات أساسية:

#### 🟢 **Promiscuous Mode**

* الاختيارات: **Accept / Reject / Inherit**
* **معناه:** يسمح أو يمنع البورت إنه يشوف كل الترافيك اللي ماشي في السويتش كله.
* **لما تختار Accept:** البورت هيشوف كل البيانات اللي رايحة لباقي الـ VMs.
* **بنستخدمها:** لما نحلل الشبكة أو نعمل sniffing / monitoring.
* **الأفضل تسيبها Reject** علشان الأمن.

📸
![Promiscuous Mode](https://www.vgarethlewis.com/wp-content/uploads/2016/11/VCSA-6_5_Installation_09.png)

---

#### 🟠 **MAC Address Changes**

* الاختيارات: **Accept / Reject / Inherit**
* **معناه:** يسمح أو يمنع تغيير الماك أدرس بتاع الكارت داخل الـ VM.
* **Accept:** ممكن الماك يتغير لو النظام الافتراضي أو الأدوات محتاجة كده.
* **Reject:** يرفض أي تغيير للماك بعد أول مرة.
* **الاختيار الافتراضي:** Accept.

📸
![MAC Address Changes](https://kb.vmware.com/resource/PROD/images/1780513_1.jpg)

---

#### 🔵 **Forged Transmits**

* الاختيارات: **Accept / Reject / Inherit**
* **معناه:** يسمح أو يمنع إرسال ترافيك من كارت عليه MAC مختلف عن الماك المسجل في الـ VM.
* لو اخترتها **Reject**، أي ترافيك فيه MAC مزيف هيتمنع.
* **الأفضل:** تسيبها Reject برضه.

📸
![Forged Transmits](https://kb.vmware.com/resource/PROD/images/1780513_2.jpg)

---

### 2️⃣ قسم Traffic Shaping

بيتحكم في كمية الترافيك اللي تمر على البورت

📸
![Traffic Shaping](https://www.nakivo.com/wp-content/uploads/2019/03/vmware-traffic-shaping-settings.png)

* **Enabled:** تقدر تحدد سرعة الترافيك بالـ Kbps.
* **Average Bandwidth:** السرعة العادية المسموح بها.
* **Peak Bandwidth:** أقصى سرعة ممكنة وقت الضغط.
* **Burst Size:** كمية البيانات اللي مسموح تمر فجأة.
* **لو مش محتاج تعمل تحديد، سيبها Disabled.**

---

### 3️⃣ قسم NIC Teaming

📸
![NIC Teaming](https://www.vgarethlewis.com/wp-content/uploads/2017/01/nic-teaming-settings.png)

هنا أهم جزء في إعدادات السويتش:

#### 🟩 **Load Balancing**

بيحدد إزاي الترافيك يتوزع على كروت النت (NICs)

الخيارات:

1. **Route based on originating virtual port (Default)**

   * الترافيك بيتوزع حسب البورت اللي خارج منه.
   * بسيط وآمن.

2. **Route based on IP hash**

   * حسب الـ IP اللي بيبعت أو بيستقبل.
   * لازم يكون السويتش الفيزيكال شغال بـ **Port Channel / EtherChannel**.

3. **Route based on source MAC hash**

   * التوزيع حسب الماك أدرس.

**الأفضل:** خليك على الافتراضي لو مش عامل bonding فعلي.

---

#### 🟨 **Network Failover Detection**

* **Link status only:** بيكشف لو الكابل مفصول أو الكارت واقع.
* **Beacon probing:** الكروت تبعت إشارات لبعض زي الـ ping علشان يعرفوا لو واحد وقع.
* **الأفضل:** خليك على Link status only لأنه أخف.

---

#### 🟥 **Notify Switches**

* لو خليتها **Yes**: السويتش الفيزيكال يعرف إن كارت وقع فيعمل reroute بسرعة.
* **الأفضل:** خليك Yes دايمًا.

---

#### 🟦 **Failback**

* **Yes:** بعد ما الكارت اللي كان واقع يرجع، الترافيك يرجعله تلقائي.
* **No:** الترافيك يفضل ماشي على الكارت البديل.
* **الأفضل:** Yes علشان كل حاجة ترجع لطبيعتها.

---

🎯 **الخلاصة**

| الإعداد                    | الأفضل تختار             | السبب                       |
| -------------------------- | ------------------------ | --------------------------- |
| Promiscuous Mode           | Reject                   | أمان                        |
| MAC Address Changes        | Accept                   | مرونة                       |
| Forged Transmits           | Reject                   | حماية                       |
| Traffic Shaping            | Disabled                 | لو مش محتاج تحديد سرعة      |
| Load Balancing             | Originating virtual port | ثابت وسهل                   |
| Network Failover Detection | Link status only         | كافي                        |
| Notify Switches            | Yes                      | استجابة أسرع                |
| Failback                   | Yes                      | يرجع الوضع الطبيعي تلقائيًا |

---

تحب أعملك **Visual Diagram فيه كل إعداد بالسهم والتأثير بتاعه** (زي خريطة مبسطة بالأسهم توضّح الترافيك ماشي إزاي حسب الإعدادات)؟
ده هيساعدك جدًا لو ناوي تشرحها أو تعملها في PDF أو مشروع.
