# دليل فني شامل حول آلية عمل تطبيق استبدال نصوص الإسبرانتو بالأحرف الصينية (كانجي)

## مقدمة

هذا الدليل موجه للمبرمجين من المستوى المتوسط الذين يتحدثون العربية ويرغبون في فهم آلية عمل تطبيق استبدال نصوص الإسبرانتو بالأحرف الصينية (كانجي) بشكل عميق. سنقوم بتحليل هيكلية التطبيق، والتدفق البرمجي، والآليات الرئيسية التي يعتمد عليها بعيدًا عن مجرد واجهة المستخدم.

يتكون التطبيق من أربعة ملفات رئيسية:
1. `main.py` - الملف الرئيسي للتطبيق
2. `صفحة لإنشاء ملف JSON لاستبدال النصوص بالإسبرانتو بسلاسل نصية (كانجي).py` - صفحة إضافية لإنشاء ملفات JSON
3. `esp_text_replacement_module.py` - وحدة معالجة واستبدال النصوص الإسبرانتية
4. `esp_replacement_json_make_module.py` - وحدة مساعدة لإنشاء ملفات JSON

## 1. نظرة عامة على البنية المعمارية للتطبيق

التطبيق مبني على إطار عمل Streamlit لإنشاء واجهات ويب تفاعلية باستخدام Python. يعتمد على مبدأ التدفق من أعلى إلى أسفل، حيث يتم إعادة تنفيذ الشيفرة البرمجية بأكملها في كل مرة يتفاعل فيها المستخدم مع الواجهة.

### المكونات الرئيسية:

1. **واجهة المستخدم (UI)**: مبنية باستخدام Streamlit، توفر عناصر تفاعلية مثل أزرار الرفع والتنزيل، والنماذج، والمؤشرات، إلخ.

2. **معالجة النصوص**: تتم من خلال مجموعة من الدوال في الوحدات الفرعية، تقوم بتحليل النص الإسبرانتي وتطبيق قواعد الاستبدال عليه.

3. **التخزين المؤقت والتصدير**: يتيح التطبيق تخزين النتائج مؤقتًا وتصديرها بتنسيقات مختلفة.

4. **المعالجة المتوازية**: يدعم التطبيق المعالجة المتوازية عند التعامل مع النصوص الكبيرة أو عمليات إنشاء ملفات JSON المعقدة.

## 2. تحليل عميق للملف الرئيسي (main.py)

### الاستيرادات والإعدادات الأولية

في بداية الملف، نجد استيرادات لمختلف المكتبات والوحدات:

```python
import streamlit as st
import re
import io
import json
import pandas as pd
from typing import List, Dict, Tuple, Optional
import streamlit.components.v1 as components
import multiprocessing
```

يتم استيراد دوال خاصة من الوحدات الفرعية:

```python
from esp_text_replacement_module import (
    x_to_circumflex,
    x_to_hat,
    hat_to_circumflex,
    circumflex_to_hat,
    replace_esperanto_chars,
    import_placeholders,
    orchestrate_comprehensive_esperanto_text_replacement,
    parallel_process,
    apply_ruby_html_header_and_footer
)
```

### إعداد معالجة متعددة المعالجات (Multiprocessing)

يتبع ذلك إعداد المعالجة المتوازية باستخدام `multiprocessing`:

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # يتم تجاهل الخطأ إذا كانت طريقة البدء محددة بالفعل
```

ملاحظة: يتم استخدام طريقة "spawn" بدلاً من طريقة "fork" الافتراضية لتجنب أخطاء PicklingError التي قد تحدث عند استخدام Streamlit مع multiprocessing.

### تخزين مؤقت للبيانات

يتم تعريف دالة `load_replacements_lists` مع مزخرف `@st.cache_data` لتخزين نتائج تحميل ملف JSON مؤقتًا وتسريع الأداء:

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """
    تحميل ملف JSON وإرجاع ثلاث قوائم:
    1) replacements_final_list
    2) replacements_list_for_localized_string
    3) replacements_list_for_2char
    """
    # ... تفاصيل التنفيذ
```

### تكوين الصفحة وواجهة المستخدم

```python
st.set_page_config(
    page_title="أداة لاستبدال الأحرف (كانجي) في النصوص الإسبرانتية",
    layout="wide"
)
st.title("استبدال نصوص إسبرانتو بالأحرف الصينية (كانجي) أو بإضافة تعليقات توضيحية بتنسيق HTML (نسخة موسعة)")
```

### التدفق الرئيسي للتطبيق

تدفق العمليات في الملف الرئيسي:

1. **تحميل ملف JSON (قواعد الاستبدال)**:
   ```python
   if selected_option == "استخدام الملف الافتراضي":
       # تحميل الملف الافتراضي
   else:
       # تحميل ملف مرفوع من المستخدم
   ```

2. **تحميل placeholders (العلامات البديلة)**:
   ```python
   placeholders_for_skipping_replacements = import_placeholders(
       './Appの运行に使用する各类文件/占位符(placeholders)_%1854%-%4934%_文字列替换skip用.txt'
   )
   ```

3. **ضبط الإعدادات المتقدمة** (المعالجة المتوازية)

4. **تحديد تنسيق الإخراج**:
   ```python
   format_type = options[selected_display]
   ```

5. **إدخال النص** (يدويًا أو من ملف)

6. **معالجة النص** داخل نموذج:
   ```python
   if submit_btn:
       # المعالجة المتوازية أو التسلسلية
       if use_parallel:
           processed_text = parallel_process(...)
       else:
           processed_text = orchestrate_comprehensive_esperanto_text_replacement(...)
   ```

7. **عرض النتائج وتوفير التنزيل**:
   ```python
   if processed_text:
       # عرض معاينة
       # تقديم زر للتنزيل
   ```

## 3. تحليل آلية معالجة النصوص (esp_text_replacement_module.py)

هذه الوحدة تحتوي على الدوال الأساسية لمعالجة نصوص الإسبرانتو واستبدالها.

### قواميس تحويل الأحرف

```python
x_to_circumflex = {
    'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
    'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'
}
# ... المزيد من القواميس للتحويلات المختلفة
```

### الدوال الأساسية

1. **استبدال أحرف الإسبرانتو**:
   ```python
   def replace_esperanto_chars(text, char_dict: Dict[str, str]) -> str:
       # استبدال أحرف وفقًا للقاموس المعطى
   ```

2. **التحويل إلى تنسيق حرف العلة (circumflex)**:
   ```python
   def convert_to_circumflex(text: str) -> str:
       # تحويل تنسيقات مثل cx أو c^ إلى ĉ
   ```

3. **الاستبدال الآمن**:
   ```python
   def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
       """
       استلام قائمة من (old, new, placeholder) وإجراء استبدال مرحلي:
       old → placeholder → new
       """
   ```

### آلية العلامات البديلة (Placeholders)

النقطة المحورية في نظام الاستبدال هو استخدام علامات بديلة (placeholders) لتجنب مشاكل الاستبدال المتداخل. تعمل كالتالي:

1. يتم أولاً استبدال النص الأصلي بعلامة بديلة فريدة
2. ثم يتم استبدال جميع العلامات البديلة بالنص النهائي

مثال:
```
"amiko" → "$PLACE123$" → "<ruby>amik<rt>صديق</rt></ruby>o"
```

هذا يحل مشكلة استبدال النصوص المتداخلة، حيث قد يؤدي الاستبدال المباشر إلى نتائج غير متوقعة.

### آلية المعالجة الشاملة

الدالة `orchestrate_comprehensive_esperanto_text_replacement` هي الدالة الرئيسية التي تنسق عملية الاستبدال:

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text,
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type
) -> str:
```

تتضمن خطوات المعالجة:
1. توحيد المسافات وتحويل أحرف الإسبرانتو
2. معالجة المناطق المحاطة بـ `%...%` (للتجاهل)
3. معالجة المناطق المحاطة بـ `@...@` (للاستبدال الموضعي)
4. الاستبدال الشامل باستخدام `replacements_final_list`
5. الاستبدال المخصص لجذور الكلمات المكونة من حرفين
6. استعادة العلامات البديلة بالقيم النهائية

### المعالجة المتوازية

الدالة `parallel_process` تقسم النص إلى أجزاء وتعالجها بشكل متوازٍ:

```python
def parallel_process(
    text: str,
    num_processes: int,
    # ... المعلمات الأخرى
) -> str:
```

تقوم بتقسيم النص إلى أجزاء بناءً على عدد الأسطر، ثم معالجة كل جزء في عملية منفصلة، وأخيرًا دمج النتائج.

## 4. إنشاء ملفات JSON للاستبدال (صفحة لإنشاء ملف JSON...)

هذا الملف يشكل الصفحة الفرعية للتطبيق، حيث يمكن للمستخدمين إنشاء ملفات JSON مخصصة للاستبدال.

### العملية الأساسية:

1. **استيراد البيانات**:
   - ملف CSV يحتوي على أزواج من (جذر إسبرانتو، ترجمة/كانجي)
   - ملف JSON لقواعد تجزئة الجذور الإسبرانتية
   - ملفات إضافية للإعدادات

2. **معالجة البيانات**:
   - بناء قواميس للاستبدال
   - تطبيق قواعد التحويل
   - حساب أولويات الاستبدال (استنادًا إلى طول الكلمة وغيرها من العوامل)

3. **إنشاء قوائم الاستبدال الثلاث**:
   - `replacements_final_list` (للاستبدال الشامل)
   - `replacements_list_for_localized_string` (للاستبدال الموضعي)
   - `replacements_list_for_2char` (لجذور الكلمات المكونة من حرفين)

4. **تصدير ملف JSON النهائي**

### الجوانب التقنية البارزة:

#### معالجة الجذور الخاصة

يولي التطبيق اهتمامًا خاصًا لمعالجة:
- اللواحق الفعلية (as, is, os, us, at, it, ot)
- جذور الكلمات المكونة من حرفين
- الكلمات التي تنتهي بـ "an" أو "on"

```python
# مثال على معالجة اللواحق الفعلية
verb_suffix_2l_2 = {}
for original_verb_suffix, replaced_verb_suffix in verb_suffix_2l.items():
    verb_suffix_2l_2[original_verb_suffix] = safe_replace(replaced_verb_suffix, temporary_replacements_list_final)
```

#### حساب أولويات الاستبدال

يستخدم التطبيق نظامًا معقدًا لأولويات الاستبدال:
```python
replacement_priority_by_length = len(esperanto_Word_before_replacement) * 10000
```

الكلمات الأطول لها أولوية أعلى في الاستبدال، مما يمنع استبدال الأجزاء الجزئية قبل استبدال الكلمات الكاملة.

## 5. وحدة esp_replacement_json_make_module.py

هذه الوحدة توفر دوالًا مساعدة لإنشاء ملفات JSON للاستبدال.

### الدوال الرئيسية:

1. **دوال تنسيق الإخراج**: تحدد كيفية تنسيق النص المستبدل:
   ```python
   def output_format(main_text, ruby_content, format_type, char_widths_dict):
       # إرجاع النص بالتنسيق المطلوب بناءً على format_type
   ```

   تتضمن العديد من التنسيقات مثل:
   - HTML مع Ruby
   - HTML مع تعديل حجم Ruby
   - تنسيق الأقواس
   - استبدال بسيط

2. **قياس عرض النص**:
   ```python
   def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
       # حساب عرض النص بالبكسل استنادًا إلى قاموس عرض الحروف
   ```

3. **إدخال فواصل الأسطر**:
   ```python
   def insert_br_at_half_width(text, char_widths_dict: Dict[str, int]) -> str:
       # إدخال علامات <br> عند نصف عرض النص
   ```

4. **المعالجة المتوازية لبناء قواميس الاستبدال**:
   ```python
   def parallel_build_pre_replacements_dict(
       E_stem_with_Part_Of_Speech_list,
       replacements,
       num_processes = 4
   ):
       # بناء قاموس الاستبدال الأولي باستخدام المعالجة المتوازية
   ```

## 6. المفاهيم التقنية الرئيسية في التطبيق

### 1. نظام علامات التبويب (Placeholders)

يعتمد التطبيق بشكل كبير على نظام علامات التبويب لحل مشكلة الاستبدالات المتداخلة. يوجد ثلاثة أنواع من العلامات:
- علامات للتجاهل (`%...%`)
- علامات للاستبدال الموضعي (`@...@`)
- علامات داخلية للاستبدال الشامل

### 2. المعالجة المتوازية

يستخدم التطبيق مكتبة `multiprocessing` من Python لتحسين الأداء:
- في المعالجة الرئيسية للنص (`parallel_process`)
- في إنشاء قواميس الاستبدال (`parallel_build_pre_replacements_dict`)

### 3. أولويات الاستبدال

نظام معقد لتحديد ترتيب الاستبدالات:
- استنادًا إلى طول الكلمة (الكلمات الأطول لها أولوية أعلى)
- معالجة خاصة للواحق (`as`, `is`, `os`, ...)
- معالجة خاصة للكلمات التي تنتهي بـ "an" أو "on"

### 4. التخزين المؤقت في Streamlit

استخدام `@st.cache_data` لتخزين نتائج معالجة البيانات الكبيرة مؤقتًا:
```python
@st.cache_data
def load_replacements_lists(json_path: str):
    # ...
```

### 5. تعديل أحجام Ruby حسب عرض النص

آلية ذكية لتعديل حجم تعليقات Ruby استنادًا إلى نسبة عرض التعليق إلى عرض النص الأصلي:
```python
if ratio_1 > 6:
    return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
elif ratio_1 > (9/3):
    return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
# ...
```

## 7. التدفق البرمجي بالتفصيل

### 1. تدفق الصفحة الرئيسية (main.py)

1. **بدء التطبيق**:
   - تكوين الصفحة والعنوان
   - استيراد الوحدات اللازمة

2. **تحميل البيانات**:
   - تحميل ملف JSON للاستبدال (افتراضي أو مخصص)
   - تحميل ملفات placeholders

3. **ضبط الإعدادات**:
   - إعدادات المعالجة المتوازية
   - اختيار تنسيق الإخراج
   - اختيار مصدر إدخال النص

4. **معالجة النص**:
   - قراءة النص المدخل (من مربع النص أو ملف مرفوع)
   - تطبيق المعالجة المناسبة (متوازية أو تسلسلية)
   - تحويل أحرف الإسبرانتو حسب الإعدادات

5. **عرض النتائج**:
   - عرض معاينة HTML أو النص العادي
   - توفير زر لتنزيل النتيجة

### 2. تدفق إنشاء ملف JSON

1. **جمع البيانات المدخلة**:
   - تحميل ملف CSV للجذور والترجمات
   - تحميل ملفات JSON للإعدادات

2. **بناء قواميس الاستبدال**:
   - تحويل بيانات CSV إلى قائمة استبدال مؤقتة
   - دمج مع الإعدادات المخصصة

3. **المعالجة المتقدمة**:
   - تطبيق قواعد اللواحق والبادئات
   - معالجة الحالات الخاصة (مثل "an" و "on")
   - تعيين أولويات الاستبدال

4. **إنشاء القوائم النهائية**:
   - إنشاء replacements_final_list
   - إنشاء replacements_list_for_localized_string
   - إنشاء replacements_list_for_2char

5. **تصدير النتيجة**:
   - دمج القوائم في ملف JSON واحد
   - توفير زر للتنزيل

## 8. تقنيات متقدمة في التطبيق

### 1. التعبيرات النمطية (Regular Expressions)

يستخدم التطبيق التعبيرات النمطية لمعالجة النص:
```python
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')
AT_PATTERN = re.compile(r'@(.{1,18}?)@')
```

تُستخدم هذه التعبيرات لتحديد أجزاء النص التي يجب تجاهلها أو معالجتها بشكل مختلف.

### 2. HTML الديناميكي

ينشئ التطبيق HTML بشكل ديناميكي مع تعديل الأنماط حسب المحتوى:
```python
ruby_style_head = """<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <style>
    <!-- أنماط CSS ديناميكية -->
    </style>
  </head>
  <body>
"""
```

### 3. حساب عرض النص

يستخدم التطبيق قاموسًا يحتوي على عرض كل حرف بالبكسل، مما يسمح بحساب دقيق لعرض النص وتحديد أماكن فواصل الأسطر:
```python
def measure_text_width_Arial16(text, char_widths_dict):
    total_width = 0
    for ch in text:
        char_width = char_widths_dict.get(ch, 8)
        total_width += char_width
    return total_width
```

### 4. تقنيات التعامل مع الكلمات المركبة في الإسبرانتو

الإسبرانتو لغة تركيبية يتم فيها تشكيل الكلمات من مجموعة من الجذور واللواحق. يتعامل التطبيق مع هذا من خلال:
- تحديد أولويات خاصة للجذور واللواحق
- معالجة خاصة للكلمات المشتقة

## 9. نصائح لتطوير التطبيق

إذا كنت ترغب في تعديل أو تطوير هذا التطبيق، إليك بعض النصائح:

### 1. إضافة لغات جديدة

لإضافة لغة جديدة للتطبيق:
1. إنشاء ملف CSV جديد يربط بين جذور الإسبرانتو والترجمات باللغة الجديدة
2. تعديل العناصر النصية في واجهة المستخدم
3. تعديل النصوص في ملفات العلامات البديلة إذا لزم الأمر

### 2. تحسين المعالجة المتوازية

يمكن تحسين المعالجة المتوازية من خلال:
1. تحسين استراتيجية تقسيم النص (قد يكون التقسيم حسب الجمل أفضل من التقسيم حسب الأسطر)
2. استخدام تقنيات متقدمة مثل concurrent.futures للتحكم بشكل أفضل في العمليات المتوازية

### 3. تعزيز قدرات معالجة النص

يمكن تعزيز قدرات معالجة النص من خلال:
1. إضافة دعم للتحليل الصرفي الآلي للإسبرانتو
2. استخدام خوارزميات أكثر تطورًا لتحديد أولويات الاستبدال
3. تحسين التعامل مع الكلمات غير المعروفة

### 4. تطوير واجهة المستخدم

يمكن تطوير واجهة المستخدم من خلال:
1. إضافة إمكانية التحرير المباشر لقواعد الاستبدال
2. توفير معاينة فورية للتغييرات
3. إضافة إمكانية حفظ الإعدادات المفضلة للمستخدم

## 10. تحديات تقنية وحلولها

### 1. مشكلة الاستبدالات المتداخلة

**المشكلة**: عند استبدال الكلمات، قد تتداخل الاستبدالات إذا كانت إحدى الكلمات جزءًا من كلمة أخرى.

**الحل**: نظام العلامات البديلة (placeholders)، حيث يتم استبدال النص الأصلي بعلامة بديلة فريدة أولاً، ثم استبدال العلامات البديلة بالنص النهائي.

### 2. تعقيد قواعد الإسبرانتو

**المشكلة**: الإسبرانتو لغة تركيبية مع قواعد معقدة للجذور واللواحق.

**الحل**: تعريف قواعد خاصة للعناصر المختلفة (اللواحق الفعلية، الجذور المكونة من حرفين، إلخ) وتطبيقها بأولويات محددة.

### 3. أداء معالجة النصوص الكبيرة

**المشكلة**: بطء في معالجة النصوص الكبيرة.

**الحل**: المعالجة المتوازية لتقسيم المهام بين عدة معالجات، مع آليات التخزين المؤقت لتجنب إعادة المعالجة.

### 4. التعامل مع تنسيقات مختلفة لأحرف الإسبرانتو

**المشكلة**: توجد عدة طرق لكتابة أحرف الإسبرانتو الخاصة (ĉ, ĝ, ...).

**الحل**: قواميس تحويل متعددة للتعامل مع جميع التنسيقات المحتملة (x_to_circumflex, hat_to_circumflex, ...).


## 11. خلاصة (تتمة)

تطبيق استبدال نصوص الإسبرانتو بالأحرف الصينية (كانجي) هو نظام معقد ولكنه مصمم بشكل جيد، يوضح استخدام تقنيات متقدمة في معالجة النصوص وتطوير واجهات المستخدم التفاعلية. يجمع التطبيق بين فهم عميق للغة الإسبرانتو وتقنيات برمجية متطورة لتوفير أداة تعليمية فعّالة.

من خلال دراسة هذا التطبيق، يمكننا تعلم العديد من التقنيات المتقدمة مثل نظام العلامات البديلة للاستبدال، والمعالجة المتوازية، وتكييف عرض HTML بشكل ديناميكي، وآليات التخزين المؤقت الفعّالة.

## 12. تحليل تفصيلي للوظائف الرئيسية

### 1. دالة orchestrate_comprehensive_esperanto_text_replacement

هذه الدالة هي قلب عملية الاستبدال، وتنسق بين جميع مراحل المعالجة:

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text,
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type
) -> str:
    # 1, 2) توحيد المسافات + تحويل أحرف الإسبرانتو إلى شكل حرف العلة
    text = unify_halfwidth_spaces(text)
    text = convert_to_circumflex(text)

    # 3) استبدال أجزاء %...% بعلامات بديلة مؤقتًا (للتجاهل)
    replacements_list_for_intact_parts = create_replacements_list_for_intact_parts(
        text, placeholders_for_skipping_replacements
    )
    # ترتيب حسب طول النص (الأطول أولاً) لتجنب التداخل
    sorted_replacements_list_for_intact_parts = sorted(
        replacements_list_for_intact_parts, key=lambda x: len(x[0]), reverse=True
    )
    for original, place_holder_ in sorted_replacements_list_for_intact_parts:
        text = text.replace(original, place_holder_)

    # 4) الاستبدال الموضعي للأجزاء المحاطة بـ @...@
    tmp_replacements_list_for_localized_string_2 = create_replacements_list_for_localized_replacement(
        text, placeholders_for_localized_replacement, replacements_list_for_localized_string
    )
    sorted_replacements_list_for_localized_string = sorted(
        tmp_replacements_list_for_localized_string_2, key=lambda x: len(x[0]), reverse=True
    )
    for original, place_holder_, replaced_original in sorted_replacements_list_for_localized_string:
        text = text.replace(original, place_holder_)

    # 5) الاستبدال الشامل باستخدام replacements_final_list
    valid_replacements = {}
    for old, new, placeholder in replacements_final_list:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new

    # 6) استبدال الجذور المكونة من حرفين (مرتين)
    valid_replacements_for_2char_roots = {}
    for old, new, placeholder in replacements_list_for_2char:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements_for_2char_roots[placeholder] = new

    valid_replacements_for_2char_roots_2 = {}
    for old, new, placeholder in replacements_list_for_2char:
        if old in text:
            place_holder_second = "!" + placeholder + "!"
            text = text.replace(old, place_holder_second)
            valid_replacements_for_2char_roots_2[place_holder_second] = new

    # 7) استعادة العلامات البديلة بالنص النهائي (بترتيب عكسي)
    for place_holder_second, new in reversed(valid_replacements_for_2char_roots_2.items()):
        text = text.replace(place_holder_second, new)
    for placeholder, new in reversed(valid_replacements_for_2char_roots.items()):
        text = text.replace(placeholder, new)
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)

    # استعادة العلامات البديلة للاستبدال الموضعي والتجاهل
    for original, place_holder_, replaced_original in sorted_replacements_list_for_localized_string:
        text = text.replace(place_holder_, replaced_original.replace("@",""))
    for original, place_holder_ in sorted_replacements_list_for_intact_parts:
        text = text.replace(place_holder_, original.replace("%",""))

    # 8) تنسيق HTML إذا لزم الأمر
    if "HTML" in format_type:
        text = text.replace("\n", "<br>\n")
        text = re.sub(r"   ", "&nbsp;&nbsp;&nbsp;", text)
        text = re.sub(r"  ", "&nbsp;&nbsp;", text)

    return text
```

هذه الدالة تظهر نهجًا متعدد المراحل للاستبدال النصي، وهو ضروري للتعامل مع التعقيدات في اللغة الإسبرانتية.

### 2. دالة parallel_process

تقوم هذه الدالة بتنفيذ المعالجة المتوازية للنصوص الكبيرة:

```python
def parallel_process(
    text: str,
    num_processes: int,
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type
) -> str:
    # التحقق مما إذا كانت المعالجة المتوازية ضرورية
    if num_processes <= 1:
        return orchestrate_comprehensive_esperanto_text_replacement(...)

    # تقسيم النص إلى أسطر
    lines = re.findall(r'.*?\n|.+$', text)
    num_lines = len(lines)

    if num_lines <= 1:
        return orchestrate_comprehensive_esperanto_text_replacement(...)

    # تحديد نطاقات التقسيم
    lines_per_process = max(num_lines // num_processes, 1)
    ranges = [(i * lines_per_process, (i + 1) * lines_per_process) for i in range(num_processes)]
    ranges[-1] = (ranges[-1][0], num_lines)  # تعديل النطاق الأخير ليشمل جميع الأسطر المتبقية

    # إنشاء مجمع عمليات ومعالجة كل جزء
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [
                (
                    lines[start:end],
                    placeholders_for_skipping_replacements,
                    replacements_list_for_localized_string,
                    placeholders_for_localized_replacement,
                    replacements_final_list,
                    replacements_list_for_2char,
                    format_type
                )
                for (start, end) in ranges
            ]
        )

    # دمج النتائج
    return ''.join(results)
```

تُظهر هذه الدالة نمطًا شائعًا في المعالجة المتوازية: تقسيم المهمة الكبيرة (النص) إلى أجزاء أصغر، ومعالجة كل جزء بشكل منفصل، ثم دمج النتائج.

### 3. آلية إنشاء ملف JSON للاستبدال

الخطوات الرئيسية في عملية إنشاء ملف JSON:

```python
# (1) فتح قاموس الجذور الإسبرانتية مع أقسام الكلام
with open("./Appの运行に使用する各类文件/PEJVO(世界语全部单词列表)'全部'について、词尾(a,i,u,e,o,n等)をcutし、comma(,)で隔てて词性と併せて记录した列表(E_stem_with_Part_Of_Speech_list).json", "r", encoding="utf-8") as g:
    E_stem_with_Part_Of_Speech_list = json.load(g)

# (2) إنشاء قاموس مؤقت لجميع جذور الإسبرانتو
temporary_replacements_dict = {}
with open("./Appの运行に使用する各类文件/世界语全部词根_约11137个_202501.txt", 'r', encoding='utf-8') as file:
    E_roots = file.readlines()
    for E_root in E_roots:
        E_root = E_root.strip()
        if not E_root.isdigit():
            temporary_replacements_dict[E_root] = [E_root, len(E_root)]

# (3) تحديث القاموس باستخدام بيانات CSV
for *, (E*root, hanzi_or_meaning) in CSV_data_imported.iterrows():
    if pd.notna(E_root) and pd.notna(hanzi_or_meaning) \
       and '#' not in E_root and (E_root != '') and (hanzi_or_meaning != ''):
        temporary_replacements_dict[E_root] = [
            output_format(E_root, hanzi_or_meaning, format_type, char_widths_dict),
            len(E_root)
        ]

# (4) ترتيب حسب طول الكلمة (الأطول أولاً)
temporary_replacements_list_1 = []
for old, new in temporary_replacements_dict.items():
    temporary_replacements_list_1.append((old, new[0], new[1]))
temporary_replacements_list_2 = sorted(temporary_replacements_list_1, key=lambda x: x[2], reverse=True)

# (5) إنشاء قائمة الاستبدال مع العلامات البديلة
temporary_replacements_list_final = []
for kk in range(len(temporary_replacements_list_2)):
    temporary_replacements_list_final.append([
        temporary_replacements_list_2[kk][0],
        temporary_replacements_list_2[kk][1],
        imported_placeholders_for_global_replacement[kk]
    ])

# (6) بناء قاموس الاستبدال الأولي باستخدام المعالجة المتوازية أو التسلسلية
if use_parallel:
    pre_replacements_dict_1 = parallel_build_pre_replacements_dict(
        E_stem_with_Part_Of_Speech_list,
        temporary_replacements_list_final,
        num_processes
    )
else:
    # تنفيذ تسلسلي مع شريط التقدم
    # ...

# (7-8) تطبيق قواعد اللواحق والبادئات وحالات خاصة أخرى
# ...

# (9) تطبيق إعدادات تجزئة الجذور المخصصة
if len(custom_stemming_setting_list) > 0:
    # ...

# (10) تطبيق إعدادات النص المستبدل المخصصة
if len(user_replacement_item_setting_list) > 0:
    # ...

# (11-15) إنشاء القوائم النهائية
# ...

# (16) دمج القوائم في كائن JSON واحد
combined_data = {}
combined_data["全域替换用のリスト(列表)型配列(replacements_final_list)"] = replacements_final_list
combined_data["二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)"] = replacements_list_for_2char
combined_data["局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)"] = replacements_list_for_localized_string

download_data = json.dumps(combined_data, ensure_ascii=False, indent=2)
```

تُظهر هذه العملية كيفية بناء قواعد استبدال معقدة من مصادر بيانات متعددة، مع معالجة الحالات الخاصة وتطبيق أولويات محددة.

## 13. آليات هامة يجب فهمها

### 1. آلية safe_replace

هذه الدالة أساسية في التطبيق، وتضمن الاستبدال السليم دون تداخل:

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    تستلم قائمة من (old, new, placeholder) وتجري استبدالًا مرحليًا:
    old → placeholder → new
    """
    valid_replacements = {}
    # أولًا old→placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # ثم placeholder→new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

تعمل هذه الدالة على مرحلتين:
1. استبدال النص الأصلي بعلامة بديلة فريدة
2. استبدال العلامات البديلة بالنص النهائي

هذا يتجنب مشكلة الاستبدالات المتداخلة، على سبيل المثال:
- لنفترض أننا نريد استبدال "ami" بـ "صَدِيق" و "amiko" بـ "صَدِيق"
- إذا قمنا باستبدال "ami" أولًا، فإن "amiko" ستصبح "صَدِيقko"، وهو ليس ما نريده
- باستخدام العلامات البديلة، سيتم استبدال "amiko" بـ "$PLACE123$" أولًا، ثم "ami" بـ "$PLACE456$"، ثم استبدال كل علامة بديلة بالنص المقابل

### 2. آلية معالجة الجذور المكونة من حرفين

الإسبرانتو تعتمد بشكل كبير على اللواحق والبادئات المكونة من حرفين. يعالج التطبيق هذه بطريقة خاصة:

```python
suffix_2char_roots=['ad', 'ag', 'am', 'ar', 'as', 'at', 'av', /* ... */]
prefix_2char_roots=['al', 'am', 'av', 'bo', 'di', /* ... */]
standalone_2char_roots=['al', 'ci', 'da', 'de', /* ... */]

# إنشاء قائمة استبدال للواحق المكونة من حرفين
replacements_list_for_suffix_2char_roots = []
for i in range(len(suffix_2char_roots)):
    replaced_suffix = remove_redundant_ruby_if_identical(
        safe_replace(suffix_2char_roots[i], temporary_replacements_list_final)
    )
    replacements_list_for_suffix_2char_roots.append([
        "$"+suffix_2char_roots[i],
        "$"+replaced_suffix,
        "$"+imported_placeholders_for_2char_replacement[i]
    ])
    # إضافة إصدارات بأحرف كبيرة وحرف أول كبير
    # ...

# تكرار العملية للبادئات والكلمات المستقلة المكونة من حرفين
# ...

# دمج جميع القوائم
replacements_list_for_2char = (
    replacements_list_for_standalone_2char_roots
    + replacements_list_for_suffix_2char_roots
    + replacements_list_for_prefix_2char_roots
)
```

هذا النهج يضمن التعامل المناسب مع العناصر المختلفة في اللغة الإسبرانتية.

### 3. آلية تنسيق الإخراج

تعتمد طريقة تنسيق الإخراج على نوع التنسيق المختار والنسبة بين عرض النص الأصلي والترجمة:

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    if format_type == 'HTML格式_Ruby文字_大小调整':
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main

        if ratio_1 > 6:
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/3):
            return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
        # ... المزيد من الحالات

    elif format_type == 'HTML格式_Ruby文字_大小调整_汉字替换':
        # ... (عكس الأدوار بين main_text و ruby_content)

    elif format_type == 'HTML格式':
        return f'<ruby>{main_text}<rt>{ruby_content}</rt></ruby>'

    elif format_type == 'HTML格式_汉字替换':
        return f'<ruby>{ruby_content}<rt>{main_text}</rt></ruby>'

    elif format_type == '括弧(号)格式':
        return f'{main_text}({ruby_content})'

    elif format_type == '括弧(号)格式_汉字替换':
        return f'{ruby_content}({main_text})'

    elif format_type == '替换后文字列のみ(仅)保留(简单替换)':
        return f'{ruby_content}'
```

هذه الدالة توضح كيفية تكييف الإخراج حسب احتياجات المستخدم ونسبة عرض النص.

## 14. تطوير وتوسيع التطبيق

إذا كنت مهتمًا بتطوير هذا التطبيق أو توسيعه، إليك بعض المجالات التي يمكنك استكشافها:

### 1. دعم لغات إضافية

يمكن توسيع التطبيق ليدعم لغات أخرى إلى جانب العربية، اليابانية، الصينية، إلخ. لتحقيق ذلك:

1. أنشئ ملف CSV يحتوي على جذور الإسبرانتو والترجمات باللغة الجديدة
2. قم بتحديث واجهة المستخدم لتعكس اللغة الجديدة
3. تأكد من أن التطبيق يتعامل بشكل صحيح مع أنظمة الكتابة المختلفة (مثل LTR/RTL)

### 2. دمج تحليل صرفي متقدم للإسبرانتو

يمكن تحسين دقة الاستبدال من خلال دمج محلل صرفي متقدم للإسبرانتو:

1. استخدام مكتبات مثل NLTK أو SpaCy مع تدريبها على الإسبرانتو
2. تطوير قواعد أكثر دقة لتحليل الكلمات المركبة
3. إضافة ميزة التعرف التلقائي على الجذور غير المعروفة

### 3. تحسين واجهة المستخدم

1. إضافة خيار التحرير المباشر للاستبدالات
2. تضمين معاينة فورية للتغييرات
3. إضافة وضع تفاعلي للتعلم، حيث يمكن للمستخدمين النقر على الكلمات لرؤية معلومات إضافية

### 4. تحسين أداء المعالجة

1. استخدام تقنيات مثل Cython لتسريع العمليات الحرجة
2. تحسين استراتيجيات التخزين المؤقت
3. تطوير خوارزميات استبدال أكثر كفاءة

### 5. ميزات تعليمية إضافية

1. إضافة تمارين تفاعلية لتعلم جذور الإسبرانتو
2. دمج قاموس إسبرانتو-عربي
3. إضافة ميزة النطق للكلمات الإسبرانتية

## 15. مكتبات Streamlit وتكاملها

من المهم فهم كيفية استخدام التطبيق لميزات Streamlit المختلفة:

### استخدام مكونات Streamlit المتقدمة

```python
# عرض HTML مخصص
components.html(preview_text, height=500, scrolling=True)

# زر تنزيل
st.download_button(
    label="تنزيل النتيجة",
    data=download_data,
    file_name="نتيجة_الاستبدال.html",
    mime="text/html"
)

# علامات التبويب
tab1, tab2 = st.tabs(["معاينة HTML", "النتيجة (كود HTML)"])
with tab1:
    # محتوى علامة التبويب الأولى
```

### استخدام التخزين المؤقت

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    # ... تحميل البيانات
```

يتيح مزخرف `@st.cache_data` تخزين نتائج الدالة مؤقتًا، مما يحسن الأداء بشكل كبير عند التعامل مع ملفات JSON كبيرة.

### استخدام حالة الجلسة

```python
if submit_btn:
    st.session_state["text0_value"] = text0
    # ... معالجة النص
```

يتيح `st.session_state` الاحتفاظ بالقيم بين إعادات تحميل الصفحة، مما يوفر تجربة مستخدم أفضل.

## 16. التفكير الختامي

تطبيق استبدال نصوص الإسبرانتو بالأحرف الصينية (كانجي) يوضح كيفية معالجة مشكلة معقدة باستخدام مجموعة من التقنيات البرمجية المتقدمة. من خلال دراسة هذا التطبيق، يمكننا تعلم:

1. **نهج متعدد المراحل لمعالجة النصوص**: تقسيم المشكلة المعقدة إلى خطوات أصغر وأكثر قابلية للإدارة.

2. **استخدام العلامات البديلة لتجنب التداخل**: حل ذكي لمشكلة شائعة في الاستبدال النصي.

3. **المعالجة المتوازية لتحسين الأداء**: كيفية تقسيم المهام الكبيرة للاستفادة من معالجات متعددة.

4. **التخزين المؤقت للبيانات**: كيفية تحسين الأداء من خلال تجنب إعادة المعالجة.

5. **واجهة مستخدم تفاعلية**: كيفية بناء واجهة سهلة الاستخدام لمهمة معقدة.

يمكن تطبيق العديد من هذه التقنيات والأفكار في مشاريع أخرى لمعالجة النصوص أو تطوير تطبيقات تعليم اللغات.

تطبيق رائع مثل هذا يوضح كيف يمكن للتكنولوجيا أن تسهل تعلم اللغات وتجسير الفجوات بين أنظمة الكتابة المختلفة، مما يجعل تعلم لغة عالمية مثل الإسبرانتو أكثر سهولة وإمتاعًا للناطقين بالعربية وغيرها من اللغات.
