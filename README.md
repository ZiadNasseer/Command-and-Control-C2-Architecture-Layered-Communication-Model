#🔹 Command and Control (C2) Architecture – Layered Communication Model

#🧠 C2 Communication & Execution – Layered Diagram

```text
┌──────────────────────────────────────────────┐
│ Layer 7 – Result / Exfiltration              │
│ • Send execution results                     │
│ • Logs / Output / Status                     │
│ • Same channel as inbound                    │
│ Protocol: HTTP / DNS / WS                    │
│ Port: 80 / 443 / 53                          │
└──────────────────────────────────────────────┘
                    ▲
                    │
┌──────────────────────────────────────────────┐
│ Layer 6 – Execution (Host Level)              │
│ • Process execution                          │
│ • API calls                                  │
│ • Memory / RAM activity                      │
│ • File or fileless actions                   │
│ Protocol: OS syscalls                        │
│ Port: —                                      │
└──────────────────────────────────────────────┘
                    ▲
                    │
┌──────────────────────────────────────────────┐
│ Layer 5 – Tasking & Command Logic             │
│ • Command parsing                            │
│ • Job queue                                  │
│ • Parameters                                 │
│ • Result correlation                         │
│ Protocol: Logical (JSON / Binary / Custom)   │
│ Port: — (inside payload)                     │
└──────────────────────────────────────────────┘
                    ▲
                    │
┌──────────────────────────────────────────────┐
│ Layer 4 – Beaconing & Session Management      │
│ • Agent check-in                             │
│ • Session ID                                 │
│ • State tracking                             │
│ • Reconnect logic                            │
│ Protocol: Logical (over HTTP/DNS/WS)          │
│ Port: same as transport                      │
└──────────────────────────────────────────────┘
                    ▲
                    │
┌──────────────────────────────────────────────┐
│ Layer 3 – Communication Protocol              │
│ • Message format                             │
│ • Request / Response                         │
│ • Session style                              │
│ Protocols: HTTP / HTTPS / DNS / WebSocket    │
│ Ports: 80 / 443 / 53                          │
└──────────────────────────────────────────────┘
                    ▲
                    │
┌──────────────────────────────────────────────┐
│ Layer 2 – Encryption & Obfuscation            │
│ • TLS / App encryption                       │
│ • Encoding                                   │
│ • Timing (sleep / jitter)                    │
│ Protocols: TLS / Custom crypto               │
│ Ports: same as Layer 3                       │
└──────────────────────────────────────────────┘
                    ▲
                    │
┌──────────────────────────────────────────────┐
│ Layer 1 – Network / Transport                 │
│ • Raw data transfer                          │
│ • Routing                                    │
│ • Connectivity                               │
│ Protocols: TCP / UDP                         │
│ Ports: 80 / 443 / 53 / High ports             │
└──────────────────────────────────────────────┘
```

#ports & Layers & Protocols & job

| Layer | البروتوكول   | البورت    | الوظيفة |
| ----- | ------------ | --------- | ------- |
| 1     | TCP/UDP      | 80/443/53 | نقل     |
| 2     | TLS/Encoding | same      | حماية   |
| 3     | HTTP/DNS/WS  | 80/443/53 | تواصل   |
| 4     | Beacon       | logical   | Session |
| 5     | Tasking      | logical   | أوامر   |
| 6     | Execution    | —         | تشغيل   |
| 7     | Exfil        | same      | نتائج   |

#🔄 Runtime Flow (من التشغيل للتنفيذ)

```text
[ Agent Start ]
      ↓
Layer 1 → Network connection
      ↓
Layer 2 → Encrypt & shape traffic
      ↓
Layer 3 → Protocol request
      ↓
Layer 4 → Beacon + Session
      ↓
Layer 5 → Receive Task
      ↓
Layer 6 → Execute on Host
      ↓
Layer 7 → Send Result back
```

#🛡️ نقاط الكشف (Detection Mapping)

Network      → IP / ASN / Reputation
Encryption   → TLS Fingerprint / Entropy
Protocol     → Header & Protocol anomalies
Beaconing    → Timing patterns
Tasking      → Network → Execution correlation
Execution    → Behavior + RAM analysis
Exfiltration → Outbound data spikes

```text
__________________________________________
| السهم   | معناه                         |
| ------- | ----------------------------- |
| L1 → L2 | فتح قناة → تغليف البيانات     |
| L2 → L3 | بيانات مشفرة → بروتوكول مفهوم |
| L3 → L4 | رسالة → Session               |
| L4 → L5 | Session → Command             |
| L5 → L6 | Command → Execution           |
| L6 → L7 | Execution → Result            |
| L7 → L3 | Result → نفس قناة الاتصال     |
__________________________________________
```

#C2 Commmunication & Execution Layers

🧠 C2 Communication & Execution Layers

(من لحظة ما الـ Agent “يصحى” لحد ما الأمر يتنفّذ)

🟦 Layer 1 — Network / Transport Layer

طبقة النقل الأساسية

البروتوكولات الشائعة (Conceptual):

TCP

UDP

(أحيانًا ICMP بشكل غير تقليدي)

البورتات (كنطاقات معروفة):

Web traffic: 80 / 443

DNS: 53

Custom services: High ports

الوظيفة:

نقل البيانات الخام

فتح قناة اتصال من الأساس

ضمان الوصول من الشبكة للوجهة

من منظور الكشف 🛡️:

IP Reputation

ASN / Hosting Provider

Odd outbound connections

Long-lived connections

🟦 Layer 2 — Encryption & Obfuscation Layer

كيف البيانات “شكلها” إيه

التقنيات (مفهوميًا):

TLS

Application-level encryption

Encoding (Base-like concepts)

Timing patterns (Sleep / Burst)

البورت:

غالبًا نفس بورت النقل (مثلاً HTTPS)

الوظيفة:

حماية المحتوى

منع الفحص السطحي

تقليل القراءة المباشرة للبيانات

الكشف 🛡️:

TLS fingerprint mismatch

Entropy analysis

Regular beacon timing

Encrypted-but-nonstandard payloads

🟦 Layer 3 — Communication Protocol Layer

لغة الكلام نفسها

بروتوكولات:

HTTP / HTTPS

DNS (Query/Response style)

WebSocket

Custom binary protocol

البورت:

HTTP → 80

HTTPS → 443

DNS → 53

WS → غالبًا 443

الوظيفة:

تنظيم الطلب والرد

تعريف Session

إرسال Tasks واستقبال Results

الكشف 🛡️:

HTTP header anomalies

DNS query length/pattern

Protocol misuse

Request frequency

🟦 Layer 4 — Beaconing & Session Layer

إدارة “مين موجود”

آليات:

Beacon (check-in)

Session ID

State (idle / busy)

Reconnect logic

البروتوكول:

فوق HTTP / DNS / WS

Logical layer مش network layer

الوظيفة:

معرفة إن الـ Agent شغال

استلام أوامر

إرسال نتائج

الكشف 🛡️:

Periodic callbacks

Same endpoint contacted repeatedly

Session reuse patterns

🟦 Layer 5 — Tasking & Command Layer

طبقة الأوامر

المحتوى:

Command ID

Parameters

Execution flags

Job queues

البروتوكول:

داخل Payload مش ظاهر للشبكة

JSON / Binary / Custom format

الوظيفة:

تحديد “نعمل إيه”

ترتيب وتنظيم التنفيذ

ربط النتيجة بالأمر

الكشف 🛡️:

Command-result correlation

Abnormal process spawning after traffic

Network → Execution chains

🟦 Layer 6 — Execution Layer (On Host)

التشغيل الفعلي

ما يحدث:

Process creation

API calls

Memory usage

File or fileless execution

لا يوجد بورت هنا

دي Host Layer

الوظيفة:

تنفيذ الأمر

جمع بيانات

تعديل حالة النظام

الكشف 🛡️:

Behavior detection

Syscall patterns

Parent/child process anomalies

RAM execution

🟦 Layer 7 — Result & Exfiltration Layer

إرجاع النتائج

البروتوكولات:

نفس قناة الاتصال

Same protocol used inbound

البورت:

نفس البورت المفتوح

الوظيفة:

إرسال Output

Logs

Status codes

الكشف 🛡️:

Data size anomalies

Upload-like behavior

Outbound bursts after idle

#🧠 Windows Internals — الصورة الكبيرة

# أي برنامج (سواء عادي أو خبيث) لازم يعدّي من نفس الطبقات

```text
Application (User Mode)
        ↓
Windows API (Win32)
        ↓
NTDLL
        ↓
System Calls
        ↓
Kernel Mode
        ↓
Hardware
```
```text
+-----------------------------+
|     Application (EXE)       |
|        User Mode            |
+-------------+---------------+
              |
              v
+-----------------------------+
|      Windows API (Win32)    |
|  File / Process / Network  |
+-------------+---------------+
              |
              v
+-----------------------------+
|            NTDLL            |
|  NT Functions (Bridge)      |
+-------------+---------------+
              |
              v
+-----------------------------+
|        System Call           |
|   User → Kernel Transition  |
+-------------+---------------+
              |
              v
+=============================+
|         Kernel Mode         |
|       Windows Kernel        |
+=============+===============+
              |
              -

|           |           |          |
v           v           v          v
+------+   +---------+  +-------+  +---------+
| Proc |   | Memory  |  | Files |  | Network |
| Mgr  |   | Manager |  | NTFS  |  | TCP/IP  |
+------+   +---------+  +-------+  +---------+
                      |
                      v
                +-----------+
                |    RAM    |
                +-----------+

#🧠 Windows Memory Internals (RAM) — Deep Dive

الصورة الكبيرة الأول

الـ RAM في ويندوز مش مساحة واحدة
دي منظومة صارمة جدًا اسمها:

Virtual Memory System

يعني:

كل برنامج شايف نفسه لوحده

وكل عنوان ذاكرة “وهمي”

والويندوز هو اللي بيفصل ويترجم

🧱 1️⃣ Virtual Memory (VM)
يعني إيه Virtual؟

أي Process عنده:

Virtual Address Space خاص بيه

غالبًا 64-bit = مساحة ضخمة جدًا

مش شرط تكون محمّلة فعلًا في RAM

Process A: 0x00000000 ─────────── 0x7FFFFFFFFFFF
Process B: 0x00000000 ─────────── 0x7FFFFFFFFFFF

🧭 2️⃣ Address Translation (MMU)

الترجمة بتحصل عن طريق:

CPU

Memory Management Unit (MMU)

Page Tables

```text
Virtual Address
      ↓
Page Table
      ↓
Physical Address (RAM)
```

📌 أي Access:

يتراجع

يتشيّك

يتسمح أو يترفَض

📦 3️⃣ Pages (وحدة الذاكرة)

الويندوز يقسم الذاكرة إلى:

Pages

الحجم غالبًا: 4KB

كل Page ليها:

Address

Permissions

State

🔐 4️⃣ Page Permissions (دي خطيرة جدًا)

كل Page ممكن تكون:

Permission	معنى
READ	قراءة
WRITE	كتابة
EXECUTE	تنفيذ
NO ACCESS	ممنوع

📌 مثال مهم:

Code → READ + EXECUTE
Data → READ + WRITE

لو العكس حصل → 🚨

وده أساس كشف:

Code Injection

In-memory execution

🧠 5️⃣ Page States (حالة الصفحة)
State	شرح
Free	مش مستخدمة
Reserved	محجوزة
Committed	مستخدمة فعليًا
Swapped	رايحة Disk

📌 Process ممكن يكون شايف مساحة كبيرة
بس فعليًا مستخدم جزء صغير

🧵 6️⃣ Stack vs Heap
🧵 Stack

لكل Thread

أوامر التنفيذ

Local variables

سريع جدًا

محدود

📦 Heap

Dynamic memory

Objects

Buffers

أكبر

أخطر

📌 أغلب المشاكل والسلوكيات الغريبة بتطلع من Heap

🧬 7️⃣ Memory Regions

داخل أي Process:

```text
+---------------------+
| Image (EXE, DLLs)   |
+---------------------+
| Heap                |
+---------------------+
| Stack (per thread)  |
+---------------------+
| Shared Memory       |
+---------------------+
| Mapped Files        |
+---------------------+
```

🧠 8️⃣ In-Memory Code Execution (مفهومي)

من منظور دفاعي:

الكود الطبيعي:

جاي من File

ليه Image Mapping

الكود المشبوه:

موجود في RAM فقط

مفيش File backing

Permissions غريبة

📌 EDR بيبص هنا بالذات

🧿 9️⃣ Copy-on-Write (COW)

تقنية ذكية:

Processes تشارك نفس DLL

أول ما حد يكتب → نسخة خاصة

📌 بتحافظ على RAM
وبرضه نقطة مراقبة مهمة

🧯 1️⃣0️⃣ Page Faults

لما:

Process يطلب Page مش موجودة في RAM

اللي يحصل:

CPU يوقف

Kernel يدخل

Page تتجاب من Disk

التنفيذ يكمل

📌 كثرة Page Faults = سلوك غير طبيعي أحيانًا

🛡️ 1️⃣1️⃣ EDR بيبص على إيه في الرام؟
مؤشرات خطيرة:

Executable memory without file

RWX pages

Code in Heap

Self-modifying memory

Memory permission changes

Threads بتبدأ من عناوين غريبة

📌 السلوك أهم من الكود

🧠 1️⃣2️⃣ ليه “Fileless” مش فعليًا Fileless؟

لأن:

مفيش حاجة بتشتغل من غير RAM

والرام:

لها Permissions

لها Traces

لها Patterns

Fileless = Diskless مش Memory-less

#🔥 الخلاصة الذهبية

الرام في ويندوز محكومة بقوانين صارمة

أي برنامج:

لازم يطلب Memory

لازم يحدد Permissions

لازم ينفّذ من Pages مسموحة

والـ EDR:
واقف عند كل خطوة

#🧠 Windows Process Memory Layout (RAM)

```text
┌──────────────────────────────────────────────┐
│            Virtual Address Space              │
│          (Private per Process)                │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ Image Mapping (EXE / DLLs)               │  │
│  │ - Code Sections (RX)                    │  │
│  │ - Data Sections (RW)                    │  │
│  │ - Imports / Exports                     │  │
│  │ - PE Headers                            │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ Heap (Dynamic Memory)                   │  │
│  │ - malloc / new / VirtualAlloc           │  │
│  │ - Objects / Buffers                     │  │
│  │ - Often RW                              │  │
│  │ ⚠ High-risk area (behavior analysis)   │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ Stack (per Thread)                      │  │
│  │ - Function calls                        │  │
│  │ - Return addresses                     │  │
│  │ - Local variables                      │  │
│  │ - Grows downward ↓                     │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ Shared Memory                           │  │
│  │ - IPC                                  │  │
│  │ - Shared Sections                      │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │ Memory Mapped Files                     │  │
│  │ - Files mapped into RAM                │  │
│  │ - DLL backing                           │  │
│  └────────────────────────────────────────┘  │
│                                              │
└──────────────────────────────────────────────┘
```

#🔄 Virtual → Physical Translation (تحت الغطا)
#📌 لو permission غلط → Access Violation

```text
[ CPU Instruction ]
        ↓
[ Virtual Address ]
        ↓
[ Page Table ]
        ↓
[ Physical Page (RAM) ]
        ↓
[ Permission Check ]
   (R / W / X ?)
```

#📦 Page Level View (مهم جدًا)

```text
┌─────────────── Page (4 KB) ───────────────┐
│ Virtual Address: 0x7FF6XXXX                │
│ Physical Address: 0x1A3F0000               │
│ State: Committed                           │
│ Permissions: R / W / X                     │
│ Backed by: File | Pagefile | Anonymous    │
└───────────────────────────────────────────┘
```

#🧬 Page States Diagram

```text
[ Free ]
   ↓
[ Reserved ]
   ↓
[ Committed ] ←── Used by process
   ↓
[ Swapped ]   ←── Pagefile (Disk)
```

#🧵 Thread + Stack View
#📌 كل Thread = Stack خاص

```text
Process
 ├── Thread 1
 │     └── Stack
 │          ├── Function A
 │          ├── Function B
 │          └── Return Addr
 │
 ├── Thread 2
 │     └── Stack
 │
 └── Thread 3
       └── Stack
```

#🛡️ EDR / Defender View (من الرام)

```text
[ Memory Events ]
      │
      ├─ RW → RX change
      ├─ Executable Heap
      ├─ No File-backed Code
      ├─ Thread start @ Heap
      ├─ API usage patterns
      └─ Timing anomalies
```

#🧠 الخلاصة البصرية
#أي تنفيذ لازم يعدّي من الرام
#والرام = مراقبة + Permissions + آثار

```text
Disk (Files)
   ↓ load
Image Mapping
   ↓ alloc
Heap / Stack
   ↓ permission
Executable Pages
   ↓ execution
CPU
```
