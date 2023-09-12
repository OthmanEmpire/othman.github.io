---
layout: translation
author: Othman Alikhan
published: true
lang: ar
title-ar: كيفية تثبت وتكون OpenLDAP وphpLDAPadmin على Ubuntu 16.04
title-en: How To Install and Configure OpenLDAP and phpLDAPadmin on Ubuntu 16.04
original-author: Brian Boucheron
original-date: 2017-06-01
original-article: >-
  https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-openldap-and-phpldapadmin-on-ubuntu-16-04
category: translation
tags:
  - sysadmin
  - linux
  - ubuntu
  - digitalocean
---

![intro](https://www.digitalocean.com/_next/static/media/intro-to-cloud.d49bc5f7.jpeg)

## جدول المحتويات


- [المقدمة](#المقدمة)
- [1. المتطلبات الأساسية](#prerequisites)
- [2. الخطوة الأولى - تثبيت وتكوين خادم LDAP](#step1)
- [3. الخطوة الثاني - تثبيت وتكوين الواجهة الويب phpLDAPadmin](#step2)
- [4. الخطوة الثالثة - تسجيل الدخول إلى واجهة الويب phpLDAPadmin](#step3)
- [5. الخطوة الرابعة - تكوين تشفير StartTLS LDAP](#step4)
- [6. الخاتمة](#conclusion)


<h3 id="introduction" lang="ar" dir="rtl">المقدمة</h3>

بروتوكول الوصول إلى الدليل الخلفي (LDAP) هي بروتوكول عياري مصمم لإدارة ووصول إلى معلومات الدليل عبر الشبكة. يمكن استخدامها لتخزين أي معلومة, ولكن غالبًا مستخدم كنظام مركزي لمصادقة أو  للبريد الإكتروني وأدلة هاتق الخاصة بالشركة.

في هذا الدليل, سنناقش كيف تثبت وتكون خادم OpenLDAP على Ubuntu 16.04. ثم نقوم بتثبيت phpLDAPadmin, واجهة ويب للاطلاع وتغيير معلومات LDAP. ثم نقوم بتأمين واجهة الويب وخدمة LDAP مع شهادات SSL من Let's Encrypt, وهي شركة توفر شهادات مجانيًا وتلقائيًا.


<h3 id="prerequisites" lang="ar" dir="rtl">المتطلبات الأساسية</h3>

قبل أن نبدأ هذا الدليل, يجب أن تكون عندك خادم Ubuntu 16.04 معدد مع Apache وPHP. يمكنك أن تتبع دليلنا التعليمي [كيفية تثبيت Linux, Apache, MySQL, PHP (LAMP) مكدَّس على Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04), مع حذف خطوة 2 لعدم احتياج قاعدة بيانات خادم MySQL.

بالإضافة, لأننا سنقوم بإدخال كلمات سرية إلى واجهة الويب, يجب علينا تأمين Apache مع تشفير SSL. اقرأ [كيف تأمن Apache مع Let's Encrypt على خادم Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-16-04) لتنزيل وتكوين شهادات SSL مجانية. تحتاج اسم نطاق لإكمال هذه الخطوة. سنستخدم نفس هذه الشهادات لتوفير اتصالات LDAP آمن بالإضافة.

> ملاحظة: الدليل لLet's Encrypt يفترض أن خادمك يمكن وصول إليه من الإنترنت العام. لو هذا ليس الحال, يجب عليك أن تستخدم موفر شهادات آخر أو ربما المرجع المصدق الخاص بمؤسستك. في كلتا الحالتان, يمكنك أن تكمل هذا الدليل مع تغييرات قليلة, معظمهن معلقة بمسار الملفات أو اسم الملفات للشهادات.


<h3 id="step1" lang="ar" dir="rtl">الخطوة الأولى - تثبيت وتكوين خادم LDAP</h3>

الخطوة الأولى لنا هي تثبيت خادم LDAP ومع يعض الأدوات المساعدة المعلقة. كل الحزم التي نحتاج موفرة في مستودعات الافتراضية لUbuntu.

قوم بتسجيل الدخول إلى خادمك. لأنها أول مرة نستخدم `apt-get` في هذا الجلسة, نقوم بتحديت فهرس الحزم المحلي لنا, ثم نثبت الحزم التي نريدها:

```sh
sudo apt-get update
sudo apt-get install slapd ldap-utils
```

عند التثبيت, سيطلب منك تحديد وتأكيد كلمة السر للمدير LDAP. يمكنك أن تحدد أي شيء هنا, لأن عندك فرصة لتحدثه خلال لحظات.

بما أن قمنا بتثبيت الحزم, سنقوم بتكونه مرة أخر. الحزمة `slapd` عندها الإمكانية أن تطلب أسألة مهمة للتكوين, ولكن افتراضيًا يتم تخطيها أثناء عملية التثبيت. يمكننا الوصول إلى جميع المطالبات بإخبار نظامنا لإعادة تكوين الحزمة:

```sh
sudo dpkg-reconfigure slapd
```

هناك أسئلة كثيرة تحتاج إجابة في هذه العملية. سنتقبل معظمهم افتراضيًا. لنمر على الأسئلة:

<div dir="ltr">
    <ul>
        <li>Omit OpenLDAP server configuration? <code>No</code></li>
        <li>DNS domain name?
            <ul>
                <li>هذا الاختيار يحدد البنية الأساسية لممسار مجلدك. اقرأ الرسالة لتفهم بالضبط كيف هذا منفِّذ. يمكنك أن تختار أي قيمة ترغب, حتى لو لم تكون مالك النطاق. لكن في هذا الدليل العلمي نفترض أن عندك اسم نطاق لخادمك نشيط, يجب أن تسخدم ذلك. سنستخدم المثال example.com في هذا الدليل.</li>
            </ul>
        </li>
        <li>Organization name?
            <ul>
                <li>لهذا الدليل, نستخدم المثال كاسم لمؤسسنا. يمكنك أن تحدد أي شيء ترغب مناسب.</li>
            </ul>
        </li>
        <li>Administrator password? <code>ادخل كلمة سر قوي مرتين</code></li>
        <li>Database backend? <code>MDB</code></li>
        <li>Remove the database when slapd is purged? <code>No</code></li>
        <li>Move old database? <code>Yes</code></li>
        <li>Allow LDAPv2 protocol? <code>No</code></li>
    </ul>
</div>



في هذا المرحلة, خادمك LDAP تكون مكون وعامل. افتح المنفذ LDAP على جدار الحماية لك لتسمح عميل من الخارج أن يوصلوا:

```sh
sudo ufw allow ldap
```

سنجرب اتصال LDAP لنا مع `ldapwhoami`, التي تجب أن ترجع اسم المستخدم المتصل بنا:

```sh
ldapwhoami -H ldap:// -x
```

```md
Output
anonymous
```

الاسم `anonymous` النتيجة التي نتوقها, لأن استخدمنا `ldapwhoami` بدون تسجيل دخول إلى خادم LDAP. هذا يعني أن الخادم نشيط ويرد على المطالبات. سنقوم بإعداد واجهة ويب لإدارة بينات LDAP.

<h3 id="step2" lang="ar" dir="rtl">الخطوة الثاني - تثبيت وتكوين الواجهة الويب phpLDAPadmin</h3>

حتى لو أن يمكنك إدارة LDAP من موجه الأوامر, معظم المستخدمين سيجدون أنه أسهل أن تستخدم واجهة الويب. نحم نقوم بتثبيت phpLDAPadmin, برنامج PHP التي توفر هذا القدرة.

المستودعات Ubuntu عندهن حزمات phpLDAPadmin. لتقوم بتثبيت مع `apt-get`:

```sh
sudo apt-get install phpldapadmin
```

هذا يقوم بتثبيت التطبيق, وتمكن تكوينات Apache الضرورية, وإعادة تحميل Apache.

هذا الخادم الويب الآن مكون لتخديم التطبيق, ولكن نحتاج تغييرات إضافية. نريد أن نكون phpLDAPadmin ليستخدم نطاقنا, وأن لا يملئ المعلومات لتسجيل الدخول LDAP تلقائيًّا.

نبدأ بفتح ملف التكوين الأساسي مع حقوق الجذر في محرر النصوص لك:

```sh
sudo nano /etc/phpldapadmin/config.php
```

ابحث لسطر يبدأ مع `$servers->setValue('server','name'`. في `nano` يمكنك أن تبحث سلسلة بكتابة `CTRL-W`, ثم السلسلة, ثم `ENTER`. سيتم وضع المؤشر على السطر الصحيح.

هذا السطر اسم العرض لخادم LDAP لك, المستخدم في واجهة الويب للرؤوس والرسائل عن الخادم. اختر إي شيء مناسب هنا:

```md
# ملف: /etc/phpldapadmin/config.php

$servers->setValue('server','name','Example LDAP');
```

ثم, حرك تحت إلى سطر `$servers->setValue('server','base'`. هذا التكوين يخبر phpLDAPadmin ما هو جذر التدرج الهرمي لLDAP. هذا مبني على حسب القيمة التي حددناها عند إعادة تكوين حزم `slapd`. في مثالنا قمنا باختيار `example.com` ونحتاج أن نترجم هذا إلى بناء جملة LDAP بوضع كل جزء النطاق (كل شيء إلا النقطة) إلى رمز `dc=`:

```md
# ملف: /etc/phpldapadmin/config.php

$servers->setValue('server','base', array('dc=example,dc=com'));
```

الآن ابحث عن سطر التكوين `bind_id` لتسجيل الدخول وقوم بتعليقه مع `#` في بداية السطر:

```md
# ملف: /etc/phpldapadmin/config.php

#$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
```

هذا الاختيار يقوم بتعبئة تفاصيل تسجيل الدخول في الواجهة الويب. هذا معلومات يجب عدم مشاركه لو صفحة phpLDAPadmin يكون متاحة للعام.

آخر شيء نحتاج أن نعدل هي إعدادات التي تحكم رؤية بعض الرسائل التحذيرية لphpLDAPadmin. افترضيًّا التطبيق يبين العديد من الرسائل التحذيرية عن ملفات قالب. هذا لا يؤثر على استخدامنا البرنامج. يمكننا أن نخبيهم ببحث إلى معلمة `hide_template_warning`, إزالة التعليق في السطر التي تحتويه, وتحديده إلى `true`:

```md
# ملف: /etc/phpldapadmin/config.php

$config->custom->appearance['hide_template_warning'] = true;
```

هذا الشيء الأخير التي نحتاج أن نعدله. احفظ واغلق الملف لتنتهي. لا نحتاج إعادة تشعيل أي شيء للتغيرات أن تطبق.

ثم نقوم بتسجيل الدخول إلى phpLDAPadmin.


<h3 id="step3" lang="ar" dir="rtl">الخطوة الثالثة - تسجيل الدخول إلى واجهة الويب phpLDAPadmin</h3>

بعد إجراء تغييرات التكوين اللازمة على phpLDAPadmin, يمكننا أن نبدأ باستخدامه. اذهب إلى التطبيق في مصفحتك الويب. تأكد من تبديل نطاقك في السطر التحت:

```md
https://example.com/phpldapadmin
```

سيتم تحميل الصفحة الهبوط لphpLDAPadmin. انقر على رابط `login` في القائمة على اليسار في الصفحة. سيتم تقديم نموذج تيسجيل الدخول:

![](https://assets.digitalocean.com/articles/install-openldap/phpldapadmin-login-screen.png)

‏`Login DN` هو اسم المستخدم التي ستستعمله. يحتوي اسم الحساب كقسم `cn=`, واسم النطاق التي اخترته للخادم مكسور إلى أقسام في `dc=` كمعبر في الخطوات السابق. الحساب المدير الافتراضي التي قمنا بإعداده أثناء التثبيت تسمى `admin`, ولذلك في مثالنا نكتب اللاحق:

```md
cn=admin,dc=example,dc=com
```

وبعد إدخال السلسلة المناسبة لنطاقك, اكتب كلمة سر المدير التي قمت بإنشائها عند التكوين, ثم انقر على الزر `Authentication`.

ستنقلك إلى الواجهة الرئيسية.

![](https://assets.digitalocean.com/articles/install-openldap/phpldapadmin-interface.png)

عند هذا المرحلة, تكون مسجل للدخول إلى واجهة phpLDAPadmin. عندك القردة لإضافة مستخدمين, الوحدات التنظيمية, المجموعات, والعلاقات.

‏LDAP مرونة في كيفية تقدر تبني بيناتك ومجلداتك كتدرج هرمي. يمكنك تنشأ أي نوع من البنية ترغبه وكذلك تنشأ قواعد لكيفية تفاعلها.

لأن هذه العملية نفسها على Ubuntu 16.04 كما كان على الإصدار السابقة, يمكنك أن تتبع الخطوات الموضحة في قسم *Add Organizational Units, Groups, and Users* من الدليل [تثبيت LDAP لUbuntu 12.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-a-basic-ldap-server-on-an-ubuntu-12-04-vps#add-organizational-units-groups-and-users).

تلك الخطوات تعمل جيدًا على هذا التثبيت لphpLDAPadmin, لذلك اتبع لحصل على تدريب على العمل مع الواجهة وتعلم كيف تبني بيناتك.

الآن لأننا قمنا بتسجيل الدخول ونعرف الواجهة الويب, نأخذ الوقت لنوفر الخادم LDAP أمان أكثر.


<h3 id="step4" lang="ar" dir="rtl">الخطوة الرابعة - تكوين تشفير StartTLS LDAP</h3>

حتى لو قمنا بتشفير واجهة الويب لنا, عميل LDAP من الخارج لا زال يتصلوا إلى الخادم ويرسلوا معلومات بنص عادي. نستخدم شهاداتنا SSL Let's Encrypt لإضافة تشفير إلى خادم LDAP لنا.


<h4>نسخ الشهادات Let's Encrypt</h4>

لأن البرنامج الخفي `slapd` تعمل كمستخدم openldap, وشهادات Let's Encrypt تقرأ فقط من مستخدم الجذر root, نحتاج أن نطبق تعديلات لتسمح `slapd` الوصول إلى الشهادات. نقوم بإنشاء برنامج نصي قصير سيتم نسخ الشهادات إلى `/etc/ssl`, مجلد عيار للنظام لشهادات SSL ومفاتيح. السبب الذي نقوم بإنشاء برنامج نصي ليفعل هذا, بدلًا من إدخال الأوامر يدويًّا, لأننا نحتاج أن نعيد هذه العملية تلقائيًّا لما شهادات Let's Encrypt تجدَّد. سنقوم بتحديث المهمة cron `certbot` لاحقًا لتمكين ذلك.

أولًا, افتح ملف نص جديد للبرنامج النصي:

```sh
sudo nano /usr/local/bin/renew.sh
```

هذا يفتح ملف نص فارغ. الصق البرنامج النصي التالي. تأكد من تحديث الجزء `SITE=example.com` ليوافق مع مكان تخزين شهادات Let's Encrypt. يمكنك العثور على القيمة الصحيحة عم طريق إدراج مجلد الشهادات باستخدام `sudo ls /etc/letsencrypt/live`.

```sh
# ملف: /usr/local/bin/renew.sh

#!/bin/sh

SITE=example.com

# move to the correct let's encrypt directory
cd /etc/letsencrypt/live/$SITE

# copy the files
cp cert.pem /etc/ssl/certs/$SITE.cert.pem
cp fullchain.pem /etc/ssl/certs/$SITE.fullchain.pem
cp privkey.pem /etc/ssl/private/$SITE.privkey.pem

# adjust permissions of the private key
chown :ssl-cert /etc/ssl/private/$SITE.privkey.pem
chmod 640 /etc/ssl/private/$SITE.privkey.pem

# restart slapd to load new certificates
systemctl restart slapd
```

هذا البرنامج النصي يحرك مجلد شهادات Let's Encrypt, ينسخ الملفات إلى `/etc/ssl`, ثم يحدث حقوق مفتاح الخاص لسمحه من القراءة من مجموع النظام ssl-cert. بالإضافة يقوم بإعادة تشغيل `slapd`, التي تأكد أن شهادات الجديدة تحمل لما هذا البرنامج النصي تشغل من مهمة cron `certbot`.

احفظ واغلق الملف, ثم عدله ليكون قابل للتنفيذ:

```sh
sudo chmod u+x /usr/local/bin/renew.sh
```

ثم قوم بتشغيل البرنامج النصي مع `sudo`:

```sh
sudo /usr/local/bin/renew.sh
```

تأكد أن البرنامج النصي تم تشعيل وتم تحديد الملفات الجديدة في `/etc/ssl`:

```sh
sudo su -c 'ls -al /etc/ssl/{certs,private}/example.com*'
```

الأمر `sudo` أعلى مختلف قليلًا من العادي. ال`su -c '...'` جزء يحيط الأمر `ls` كامل في موجه الأوامر جذري قبل تنفيذه. لولا ما قمنا بهذا, سيتم تشغيل اسم الملف البدل `*` مع حقوق مستخدمك الغير `sudo`, وتفشل لأن `/etc/ssl/private` ليس قادر أن يقرأ بمستخدمك.

‏`ls` تطبع التفاصيل عن ثلاثة ملفات. تأكد أن الملك والحقوق يظهر صحيح:

```md
Output
-rw-r--r-- 1 root root     1793 May 31 13:58 /etc/ssl/certs/example.com.cert.pem
-rw-r--r-- 1 root root     3440 May 31 13:58 /etc/ssl/certs/example.com.fullchain.pem
-rw-r----- 1 root ssl-cert 1704 May 31 13:58 /etc/ssl/private/example.com.privkey.pem
```

ثم نقوم بأتمتة هذا باستخدام `certbot`.


<h4>تحديث مهمة Cron لتجديد Certbot</h4>

نريد أن نحدث مهمة cron المعلق ب`certbot` ليشتغل البرنامج النصي عندما الشهادات تجدد:

```sh
sudo crontab -e
```

يجب أن يكون لديك خط تجديد `certbot`. قم بتعديل السر كمثل التحت:

```
15 3 * * * /usr/bin/certbot renew --quiet --renew-hook /usr/local/bin/renew.sh
```

احفظ واغلق crontab. الآن لما `certbot` يجدد الشهادات, البرنامج النصي لنا يشتغل وينسخ الملفات, يعدل الحقوق, وإعادة تشغيل خادم `slapd`.


<h4>تكوين slapd ليوفر اتصالات آمن</h4>

نحتاج أن نضيف المستخجم openldap إلى المجموعة ssl-cert حتى يتمكن `slapd` من قراءة المفتاح الخاص:

```sh
sudo usermod -aG ssl-cert openldap
```

قم بإعادة تشغيل `slapd` حتى يلتقط المجموعة الجديدة:

```sh
sudo systemctl restart slapd
```

أخيرًا, نحتاج أن نكون `slapd` ليستخدم هذه الشهادات والمفاتيح. لنفعل هذا, نضع جميع تغيرات التكوين إلى ملف LDIF - مختصر من LDAP Data Interchange Format - وثم نحمل التغيرات إلى خادمنا LDAP مع الأمر `ldapmodify`.

افتح ملف LDIF جديد

```sh
cd ~
nano ssl.ldif
```

هذا يفتح ملف فارغ. الصق التالي في الملف, مع تحديث إلى أسماء الملفات لنعكس نطاقك:

```
‎ملف: ssl.ldif

dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/example.com.fullchain.pem
-
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/example.com.cert.pem
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/example.com.privkey.pem
```

احفظ واغلق الملف, ثم قوم بتطبيق التغييرات مع `ldapmodify`

```sh
sudo ldapmodify -H ldapi:// -Y EXTERNAL -f ssl.ldif
```

```
Output
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
```

لم نحتاج إعادة تحميل `slapd` لتحميل الشهادات الجديدة, هذا يحدث تلقائيًّا لما نحدث التكوين مع `ldapmodify`. قم بتشغيل الأمر `ldapwhoami` مرة أخرى, لتأكد. هذا المرة نحتاج أن نستخدم الاسم المضيف المناسب وإضافة الاختيار `-ZZ` لنجبر اتصال آمن:

```sh
ldapwhoami -H ldap://example.com -x -ZZ
```

نريد الاسم المضيف الكامل لما نستخدم اتصال آمن لأن العميل يتأكد أن اسم المضيف يوافق الاسم المضيف في الشهادة. هذا يمنع هجومات الرجل-في-الوسط حيث يمكن للمهاجم أن يعترض اتصالك ويتحل شخصية خادمك.

الأمر `ldapwhoami` يجب أن يرجع `anonymous`, بدون أي أخطاء. نحن فمنا بتفشير اتصالنا LDAP بنجاح.


<h3 id="conclusion" lang="ar" dir="rtl">الخاتمة</h3>

في هذه الدليل التعليمي قمنا بتثبيت وتكوين خادم OpenLDAP `slapd`, والواجهة الويب phpLDAPadmin. قمنا أيضًا بإعادة تشفير على كلا خادمان, وتحديث `certbot` لعمل تلقائيًا مع `slapd` ليجدد الشهادات Let's Encrypt.

 النظام التي قمنا بتعديده مرن للغاية ويمكنك أن تصمم المخطط التنظيمي لك وإدارة مجموعات من الموارد حسب مطالباتك. للمزيد من المعلومات على إدارة LDAP, مع تشميل أوامر السطر أكثر وتقنيات, اقرأ الدليل التعليمي لنا [كيفية إدارة واستخدام خوادم LDAP مع أدوات OpenLDAP](https://www.digitalocean.com/community/tutorials/how-to-manage-and-use-ldap-servers-with-openldap-utilities). للمزيد من المعلومات المتعمقة عن تأمين خادم LDAP, مع تشميل كيف تجبر جميع العملاء أن يستخدموا اتصال آمن, اقرأ [كيفية تشفر اتصالات OpenLDAP باستخدام STARTTLS](https://www.digitalocean.com/community/tutorials/how-to-encrypt-openldap-connections-using-starttls).