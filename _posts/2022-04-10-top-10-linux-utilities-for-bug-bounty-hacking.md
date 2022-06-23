---
layout: translation
author: Othman Alikhan
published: true
lang: ar
title-ar: أفضل 10 أدوات Linux لاختراق/مكافأة العلة
title-en: Top 10 Linux Utilities For Bug Bounty/Hacking
original-author: Samrat Gupta (Sm4rty)
original-article: >-
  https://sm4rty.medium.com/top-10-linux-utilities-for-bugbounty-hacking-dbef7ae28a28
category: translation
tags:
  - cybersecurity
  - medium
---

![computer](https://miro.medium.com/max/1400/0*N_Aq04ySgNGbgFZE)

## جدول المحتويات

- [المقدمة](#المقدمة)
- [1. الأمر "grep"](#grep)
- [2. الأمر "sed"](#sed)


#### المقدمة

مرحبًا ,أنا سامرات جوبتا المعروف أيضًا باسم (سمارتي), باحث أمني وصياد مكافأة
العلة. في هذا المدونة سأشارك معكم بعض الأدوات الشائعة في Linux OS التي يمكن تكون
مفيدة في اختراق/مكافأة العلة أو في أي أستخدام عام وستوفر وقت كثير بالتأكيد.

سنبدأ بدون إضاعة أي وقت أكثر.


<h3 id="grep" lang="en" dir="ltr">1. GREP:</h3>

يبحث مرشح grep في ملف عن نمط معين من الأحرف, ويعرض كل الأسطر التي
تحتوي على هذا النمط. النمط المبحث في الملف يشار بالاسم regular expression.

يحتوي grep على العديد من حالات الاستخدام العملية وهو بالتأكيد أحد
أوامر Linux المستخدم بالكثير.

بناء جملة لأمر grep:
 
<blockquote lang="en" dir="ltr">
grep [options] pattern [files]
</blockquote>

#### ماذا نمكن نفعل مع grep؟
1. البحث عن نص في ملفات.
2. البحث عن أسماء ملفات باستخدام الامتدادات.
3. البحث عن نمط في ملفات مضغوطة.
4. البحث عن URLs في ملفات المصدر.
5. البحث في جميع الملفات في مجلد ومجلداته الفرعية.

#### قراءة المزيد

<div lang="en" dir="ltr">
1. https://www.makeuseof.com/grep-command-practical-examples/<br>
2. https://www.geeksforgeeks.org/grep-command-in-unixlinux/<br>
3. https://phoenixnap.com/kb/grep-command-linux-unix-examples
</div>


<h3 id="sed" lang="en" dir="ltr">2. SED:</h3>

الكلمة SED في Linux أصله من العبارة **S**tream **ED**itor (محرر دفق),
وهو قادر على العديد من الأعمال على ملف كبحث, تبديل, إضافة أو حذف. عمومًا,
هو مستخدم في تبديل التص؛ وبالزيادة, يمكن يتم استخدامه في لعمليات معالجة النص
الأخرى مثل الإضافة, المسح, البحث, أو أكثر.




<blockquote lang="en" dir="ltr">
sed [options] [script] [inputfile]
</blockquote>

#### ماذا نمكن نفعل مع sed؟
1. استبدال نص.
2. إضافة نص.
3. استخدام regular expressions - استبداتل متقدم.
4. إضافة سطور/مسافات فارغة.

<div lang="en" dir="ltr">
1. https://www.javatpoint.com/linux-sed<br>
2. https://www.linuxtechi.com/20-sed-command-examples-linux-users/<br>
3. https://www.tecmint.com/linux-sed-command-tips-tricks/
</div>


<h3 id="grep" lang="en" dir="ltr">3. AWK:</h3>

ل AWK لغة برمجة نصية تستخدم في معالجة البيانات وإنشاء تقارير.
 لغة البرمجة للأمر awk لا يحتاج إلى مترجم ويسمح للمستخدم باستخدام المتغيرات,
 الدالات الرقمية, الدالات السلسلة, والعوامل المنطقية. 



---
{: data-content="footnotes"}
[^1]: الفرق بين Unix و Linux 