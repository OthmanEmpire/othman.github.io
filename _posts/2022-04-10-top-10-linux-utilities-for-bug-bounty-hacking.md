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
العلة. في هذا المدونة سأشارك بعض الأدوات الشائعة في Linux OS التي يمكن يكون
مفيدة في الاختراق/المكافأة العلة أو في أي أستخدام عام وستوفر وقت كثير بالتأكيد.

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

#### ماذا يمكننا أن نفعل مع grep؟

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
