---
layout: translation
author: Othman Alikhan
published: true
lang: ar
title-ar: كيفية حماية SSH باستخدام Fail2Ban على Ubuntu 22.04
title-en: How to Protect SSH with Fail2Ban on Ubuntu 22.04
original-author: Alex Garnett
original-article: >-
  https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-22-04
category: translation
tags:
  - cybersecurity
  - linux
  - ubuntu
  - digitalocean
---

![pizza](https://www.digitalocean.com/_next/static/media/intro-to-cloud.d49bc5f7.jpeg)


## جدول المحتويات


- [المقدمة](#المقدمة)
- [1. المتطلبات الأساسية](#prerequisites)
- [2. الخطوة الأولى - تثبيت](#step1)
- [3. الخطوة الثاني - تكوين](#step2)
- [4. الخطوة الثالث - اختبار سياسات الحظر (اختياري)](#step3)
- [5. الاستنتاج](#conclusion)


#### المقدمة

⁧SSH هي طريقة مشهورة لتشبيك إلى خادم سحابي (cloud server). هي متين, وقابل للتوسيع - كما يتم تطوير معاير تشفير (encryption standards) جديدة , يمكن استخدامها لإنشاء مفاتيح SSH جديدة, مع ضمان بقاء البروتوكول الأساسي آمنًا. ومع ذلك, لا يوجد بروتوكول أو حزمة برامج (software stack) مضمونة تمامًا, وSSH التي يتم نشره على نطاق واسع عبر الإنترنت تعني أنها تمثل **سطحه هجوم** أو **متجه هجوم** يمكن للأشخاص من خلاله محاولة الوصول.

أي خدمة متعرض على الشبكة هي هدف محتمل بهذه الطريقة. لو قمت بمراجعة السجلات الخاصة بخدمة SSH التي تعمل على أي خادم يتم تداوله على نطاق واسع, غالبََا ما ستشاهده متكررََا, محاولات منهجية لتسجيل الدخول التي تمثل هجمات القوة الغاشمة (brute force attacks) من قبل المستخدمين والروبوتات على حد سواء. برغم أنك يمكنك إجراء بعض التحسينات على خدمة SSH لتقلل الاحتمال على هذه الهجومات أن تنجح إلى قريب من صفر, مثل تعطيل مصادقة كلمة السر لمفاتيح SSH, لا يزال تحمل مسؤولية طفيفة ومستمرة.  

عمليات نشر الإنتاج على نطاق واسع لمن هذه المسؤولية غير مقبولة على الإطلاق غالبََا ينفذون VPN مثل WireGuard أمام خدمتهم SSH, ليكون مستحيل أن تشبك مباشرََا إلى منفذ (port) SSH الافتراضي 22 من الأنترنت الخارجي بدون تجريد برامج إضافي أو بوابات (gateways). هذه الحلول VPN هي موثوق على نطاق واسع, ولكن تزيد التعقيد, وتقدر تكسر بعض الأتمتة  أو الخطافات البرمجية الصغيرة.

قبل أو أيضافََا إلى الالتزام بإعداد VPN كامل, يمكنك تنفذ أداة تسمى Fail2ban.⁧ Fail2ban قادر أن يقلل هجومات القوة الغاشم (brute force attacks) بشكل كبير من خلال إنشاء قواعد تتغير ذاتي إعدادات الجدار الحماية إلى منع IPs مخصًص بعد عدد معين من المحاولات التسجيل الدخول الفاشلة. سيسمح هذا لخادمك بتقوية نفسه ضد المحاولات الوصول دون تدخل منك. 

في هذه الدليل, سترى كيفية تثبيت واستخدام Fail2ban 22.04 على خادم Ubuntu 22.04.


<h3 id="prerequisites" lang="ar" dir="rtl">المتطلبات الأساسية</h3>

لإكمال هذه الدليل, تحتاج:

- خادم Ubuntu 22.04 ومستخدم غير جذر (non-root) له امتيازات sudo. يمكنك معرفة المزيد حول كيفية إعداد مستخدم بهذه الامتيازات في الدليل "Initial Server Setup with Ubuntu 22.04 guide".
- اختياريََا, خادم ثاني يمكنك أن تشبك إليه من الخادم الأول, لتجرب أن تحظر منه.


<h3 id="step1" lang="ar" dir="rtl"> الخطوة الأولى - تثبيت Fail2ban</h3>


‏Fail2ban متوفر في مستودعات برامج Ubuntu. أبدأ بتشغيل الأوامر التالي مع مستخدم غير جذري لتحديث قوائم الحزم الخاص بك وتثبيت Fail2ban:

```md
sudo apt update
sudo apt install fail2ban
```

سيقوم Fail2ban بإعداد خدمة في الخلف بعد تثبيتها. ولكن, هي معطل افتراضيََا, لأن بعض إعداداتها الافتراضي تقدر تسبب تأثيرات غير مرغوب. يمكنك التحقق من هذا باستخدام الأمر `systemctl`:


```sh
systemctl status fail2ban.service
```

```sh
Output
○ fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; disabled; vendor preset: enabled
     Active: inactive (dead)
       Docs: man:fail2ban(1)
```

يمكنك تبدأ Fail2ban فورََا, ولكن, سنراجع بعض مميزاتها أولََا.


<h3 id="step2" lang="ar" dir="rtl">الخطوة الثاني - تكوين Fail2ban</h3>

الخدمة Fail2ban يخزن تعديدات الملفات في مجلد `/etc/fail2ban`. هناك ملف مع افتراضات اسمه `jail.conf`. أذهب إلى ذلك المجلد وأطبع العشرة السطور الأول من الملف باستخدام `head -20`:

```sh
cd /etc/fail2ban
head -20 jail.conf
```

```sh
Output
#
# WARNING: heavily refactored in 0.9.0 release.  Please review and
#          customize settings for your setup.
#
# Changes:  in most of the cases you should not modify this
#           file, but provide customizations in jail.local file,
#           or separate .conf files under jail.d/ directory, e.g.:
#
# HOW TO ACTIVATE JAILS:
#
# YOU SHOULD NOT MODIFY THIS FILE.
#
# It will probably be overwritten or improved in a distribution update.
#
# Provide customizations in a jail.local file or a jail.d/customisation.local.
# For example to change the default bantime for all jails and to enable the
# ssh-iptables jail the following (uncommented) would appear in the .local file.
# See man 5 jail.conf for details.
#
# [DEFAULT]
```

كما سترى, الأسطر الأولى من هذا الملف معلقة (commented out) - يبدؤون برمز `#` التي تشير إلى أنها يجب أن تقرأ تعليقات بدلََا من الإعدادات. كما سترى أيضََا, هذه التعليقات يوجهك إلى عدم تغيير هذا الملف مباشرةََ. بدلََا من ذلك, عندك خياران: إمَّا إنشاء ملفَّات تعريف فردية لFail2ban في ملفات متعددة داخل `jail.d/`. أو إنشاء وتجميع كل الإعدادات المحلية الخاص بك في ملف `jail.local`. تحدث الملف `jail.conf` بشكل دوري عندما تحدث Fail2ban, ويتم استخدامه كمصدر إعدادات افتراضية التي لم تقم بإنشاء أي تجاوزات لها.  

في هذه الدليل التعليمي, ستقوم بإنشاء `jail.local`. تستطيع أن تفعل ذلك بنسخ `jail.conf`:

```sh
sudo cp jail.conf jail.local
```

الآن تستطيع تبدأ في تغيير الإعدادات. أفتح الملف في `nano` أو أحب محرر النصوص:

```sh
sudo nano jail.local
```

أثناء تصفحك في الملف, هذه الدليل تقوم بمراجعة بعض الخيارات التي ترغب في تحديتها. الإعدادات تحت القسم `[DEFAULT]` قريب من أعلى الملف سيطبِّق إلى جميع الخدمات التي تدعم Fail2ban. في مكان آخر في الملف, توجد رؤوس ل`[sshd]` وخدمات آخرى, التي تضمن خدمة-معين إعدادات التي تطبق فوق الافتراضية.   

```md
# /etc/fail2ban/jail.local

[DEFAULT]
. . .
bantime = 10m
. . .
```

المعلمة `bantime` تحدد طول وقت حظر المستخدم عند فشل في التحقيق, وقياسها في الثواني. مبدأيا, هي محددة إلى 10 دقائق.

```md
# /etc/fail2ban/jail.local

[DEFAULT]
. . .
findtime = 10m
maxretry = 5
. . .
```

المعلمتان التاليتان هما `findtime` و`maxtry`. يعملان مع بعض لتهيئة الشروط التي بموجبها يتم العثور على العميل كمستخدم غير صالح يجب حظره.

المتغير `maxretry` يحدد عدد مرات التي يتعين على العميل المصادقة عليها خلال فترة زمنية محددة بواسطة `findtime`, قبل أن يحظر. مع الإعدادات الافتراضية, الخدمة fail2ban تحظر عميل عنده 5 محاولات لتسجيل دخول فاشلة في مدة 10 دقائق. 

```
# /etc/fail2ban/jail.local

[DEFAULT]
. . .
destemail = root@localhost
sender = root@<fq-hostname>
mta = sendmail
. . .
```

لو تحتاج أن تصلك تنبيهات بريد إلكتروني لما Fail2ban يفعل عملية, يجب عليك تقييم إعدادات `destemail`, و`sendername`, و`mta`. ال`destemail` معلمة تحدد عنوان البريد الإلكتروني التي تجب أن توصل رسائل حظر. ال`sendername` تحدد قيمة الحقل "From" في الرسالة. المعلمة `mta` يتحكم أي خدمة بريد يستخدم لما يرسل رسالة. افتراضيًا, هذا `sendmail`, ولكن يمكن ترغب أن تستخدم Postfix أو حل بريد آخر.

```
# /etc/fail2ban/jail.local

[DEFAULT]
. . .
action = $(action_)s
. . .
```

هذه المعلمة تقوم بتكوين الإجراء التي Fail2ban تأخذ لما تريد حظر مستخدم. القيمة `action_` معرف في الملف قبل هذه المعلمة. الأمر الافتراضي هو تحديث إعدادات الجدار الحماية
إلى منع حركة المرور من المستخدم المخالف حتى ينتهي وقت الحظر.

هناك نصوص `action_` آخرى متوفر افتراضيا التي تقدر أن تتبدل `$(action)` من فوق:

```
### /etc/fail2ban/jail.local
…
# ban & send an e-mail with whois report to the destemail.
action_mw = %(action_)s
            %(mta)s-whois[sender="%(sender)s", dest="%(destemail)s", protocol="%(protocol)s", chain="%(chain)s"]

# ban & send an e-mail with whois report and relevant log lines
# to the destemail.
action_mwl = %(action_)s
             %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]

# See the IMPORTANT note in action.d/xarf-login-attack for when to use this action
#
# ban & send a xarf e-mail to abuse contact of IP address and include relevant log lines
# to the destemail.
action_xarf = %(action_)s
             xarf-login-attack[service=%(__name__)s, sender="%(sender)s", logpath="%(logpath)s", port="%(port)s"]

# ban IP on CloudFlare & send an e-mail with whois report and relevant log lines
# to the destemail.
action_cf_mwl = cloudflare[cfuser="%(cfemail)s", cftoken="%(cfapikey)s"]
                %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]
…
```

على سبيل المثال, `action_mw` يأخذ فعل ويرسل ربريد إلكتروني, `action_mwl` يأخذ فعل, يرسل بريد إلكتروني, ويضمن التسجيل (logging), و`action_cf_mwl` يفعل كل الفوق أيضافََا إلى إرسال تحديث إلى Cloudflare API المتعلق بحسابك إلى حظر المخالف هناك, أيضََا.


<h4 id="prerequisites" lang="ar" dir="rtl"> إعدادات فردية للسجن</h4>

التالي هي الجزء للملف التكوين التي تعمل مع الخدامات الفردية. هي محددة برؤوس الأقسام, ك `[sshd]`.

كل هذه الأقسام تحتاج تمكينها فردية بإضافة العبرة `enabled = true` تحت القسم, مع الإعدادات الأخرى.  


```
# /etc/fail2ban/jail.local

[jail_to_enable]
. . .
enabled = true
. . .
```

افتراضيًا, الخدمة SSH ممكَِن وكل الباقي غير ممكَِن. بعض الإعدادات الأخرى المحدودة هنا هي ال`filter` التي تكون مستخدم لتقرير لو سطر في السجل يشير إلى محاولة فاشلة للدخول وال`logpath` التي توضح fail2ban أين الموقع للسجلات للخدمة المعين.

القيمة `filter` هي في الواقع مرجع إلى ملف موجود في `/etc/fail2ban/filter.d` مجلد, مع الإمداد `.conf` محذوف. هذه الملفات تحتوي regular expressions (اختصار شائع لتحليل النصوص) التي تقرر لو سطر في السجل هي محاولة فاشلة للدخول. نحن لم نغطي هذه الملفات في بدقة في هذه المدونة, لأن الملفات معقدة والإعدادات الافتراضية تحلل الأسطر المناسبة.  

ولكن, تقدر ترى أي نوع من المرشحات متوفرة من خلال النظر إلى المجلد:

```sh
ls /etc/fail2ban/filter.d
```

لو ترى ملف يظهر معلق ما الخدمة التي تستخدمه, يمكنك أن تفتحه بمحرر النصوص (text editor). معظم الملفات عندهم تعليقات واضحة ويمكن أن تعرف على الأقل أي نوع الحالة البرنامج النصي تحمي منها. معظم هذه المرشحات عندهم أقسام مناسبة (غير متمكن) في الملف `jail.conf` التي تقدر نتمكن في `jail.local` لو أردنا. 

على سبيل المثال, تخيل أنك تخدم موقع باستخدام Nginx وتلاحظت أن جزء من موقعك المحمي بكلمة سر معرض بمحاولات تسجيل الدخول. يمكنك أن تعدد fail2ban أن تستخدم الملف  `nginx-http-auth.conf` للتحقق من هذه الشرط داخل الملف `/var/log/nginx/error.log`.

هذا بالواقع جاهز في القسم مسمى `[nginx-http-auth]` في ملفك `etc/fail2ban/jail.conf/`. تحتاج فقط أن تزيد المعلمة `enabled`:

```
# /etc/fail2ban/jail.local

. . .
[nginx-http-auth]

enabled = true
. . .
```

عند الانتهاء من التحرير, احفظ الملف وأغلقه. في هذه النقطة, يمكنك أن تمكن خدمتك Fail2ban ليشتغل تلقائيًا من الآن. أولًا, شغل `systemctl enable`:

```sh
sudo systemctl enable fail2ban
```

ثم, أبدأها يدويًا لأول مرة مع `systemctl start`:

```sh
sudo systemctl start fail2ban
```

يمكنك أن تحقق أنها تعمل مع `systemctl status`:

```sh
sudo systemctl status fail2ban
```

```
Output
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enab>
     Active: active (running) since Mon 2022-06-27 19:25:15 UTC; 3s ago
       Docs: man:fail2ban(1)
   Main PID: 39396 (fail2ban-server)
      Tasks: 5 (limit: 1119)
     Memory: 12.9M
        CPU: 278ms
     CGroup: /system.slice/fail2ban.service
             └─39396 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

Jun 27 19:25:15 fail2ban22 systemd[1]: Started Fail2Ban Service.
Jun 27 19:25:15 fail2ban22 fail2ban-server[39396]: Server ready
```

في الخطوة التالي, ستوضح عملية Fail2ban أثناء العمل.



<h3 id="step3" lang="ar" dir="rtl"> الخطوة الثالث - اختبار سياسات الحظر (اختياري)</h3>

من خادم آخر, واحد ما تحتاج إليه أن تدخل على خادمك الFail2ban في المستقبل, يمكنك تختبر القواعد عن طريق حظر الخادم الثاني. بعد الدخول إلى الخادم الثاني, حاول SSH إلى الخادم Fail2ban. يمكنك أن تشبك باستخدام اسم غير موجود:

```
ssh blah@your_server
```

ادخل أحرف عشوائية في المطالبة بكلمة المرور. كرر هذا عدة مرات. عند نقطة معين, الخطأ الذي ستظهر ستغير من `Permission denied` إلى `Connection refused`. هذا يشير أن خادمك الثاني تم حظره من الخادم Fail2ban.

على خادمك Fail2ban, يمكنك ترى القواعد الجديد بالاطلاع إلى إنتاج `iptables`. `iptables` أمر للتفاعل مع المنفذ والجدار الحماية على خادمك. لو اتبعت المدونة لDigitalOcean لإعداد خادم, ستقوم باستخدام`ufw` لإدارة قواعد جدار الحماية على مستوى أعلى. تشغيل `iptables -S` سيظهر كل القواعد الجدار الحماية الذي `ufw` قام بإنشائه.

```sh
sudo iptables -S
```


```
Output
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT ACCEPT
-N f2b-sshd
-N ufw-after-forward
-N ufw-after-input
-N ufw-after-logging-forward
-N ufw-after-logging-input
-N ufw-after-logging-output
-N ufw-after-output
-N ufw-before-forward
-N ufw-before-input
-N ufw-before-logging-forward
-N ufw-before-logging-input
-N ufw-before-logging-output
…
```

إذا قمت بتوجيه إخراج `iptables -S` إلى `grep` للبحث عن القواعد التي تحتوي السلسلة `f2b`, سترى القواعد التي أضيفت بfail2ban:

```sh
sudo iptables -S | grep f2b
```

```
Output
-N f2b-sshd
-A INPUT -p tcp -m multiport --dports 22 -j f2b-sshd
-A f2b-sshd -s 134.209.165.184/32 -j REJECT --reject-with icmp-port-unreachable
-A f2b-sshd -j RETURN
```

السطر التي تحتوي `REJECT --reject-with icmp-port-unreachable` ستكون مضيف بFail2ban ويجب أن يعكس عنوان IP الخاص بخادمك الثاني.


<h3 id="conclusion" lang="ar" dir="rtl">الاستنتاج</h3>

ستكون قادر الآن أن تعدد بعض السياسة للحظر لخدماتك. Fail2ban طريقة مستفيدة أن تحمي أي خادم تستخدم تسجيل دخول. لو أردت أن تتعلم أكثر من كيف تعمل fail2ban, تقدر تفحص مدوننا على [كيف قواعد وملفات fail2ban تشتغل](https://www.digitalocean.com/community/articles/how-fail2ban-works-to-protect-services-on-a-linux-server).

للمزيد من المعلومات على كيف تستخدم fail2ban لتحني خادمات آخرى, يمكنك تقرأ عن [كيف تحمي خادم Nginx مع Fail2ban على Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-an-nginx-server-with-fail2ban-on-ubuntu-14-04) و [كيف تحمي خادم Apache مع Fail2ban على Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-an-apache-server-with-fail2ban-on-ubuntu-14-04) 