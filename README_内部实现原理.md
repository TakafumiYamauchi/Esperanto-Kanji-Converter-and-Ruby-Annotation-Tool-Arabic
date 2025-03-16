# دليل متعمق لفهم آلية عمل تطبيق استبدال نصوص الإسبرانتو
## للمبرمجين متوسطي المستوى

هذا الدليل مصمم للمطورين متوسطي المستوى الذين يريدون فهماً عميقاً للبنية البرمجية والآليات الداخلية لتطبيق استبدال نصوص الإسبرانتو بالأحرف الصينية (كانجي) أو إضافة تعليقات توضيحية. سنتعمق في كيفية عمل الكود ومختلف الوحدات البرمجية وتفاعلها مع بعضها البعض.

## 1. الهيكل العام للتطبيق

التطبيق مكون من أربعة ملفات رئيسية تعمل معاً:

1. **main.py**: الملف الرئيسي الذي يشكل واجهة التطبيق الأساسية ويدير عملية استبدال النصوص.
2. **صفحة لإنشاء ملف JSON لاستبدال النصوص بالإسبرانتو بسلاسل نصية (كانجي).py**: صفحة ثانوية تسمح بإنشاء ملفات JSON مخصصة لقواعد الاستبدال.
3. **esp_text_replacement_module.py**: وحدة برمجية تحتوي على الدوال الأساسية لمعالجة نصوص الإسبرانتو واستبدالها.
4. **esp_replacement_json_make_module.py**: وحدة برمجية تساعد في إنشاء وهيكلة ملفات JSON الخاصة بالاستبدال.

يجدر بالذكر أن التطبيق مبني باستخدام إطار عمل Streamlit، الذي يسهل بناء تطبيقات الويب التفاعلية للبيانات والتعلم الآلي باستخدام Python.

## 2. الكشف عن دورة حياة البيانات في التطبيق

لفهم آلية عمل التطبيق، من المفيد تتبع دورة حياة البيانات عبر مكونات مختلفة:

```
[ملف JSON للاستبدال] → [نص الإدخال] → [معالجة وتحويل] → [نص مستبدل مع تعليقات]
```

### آلية التشغيل المبسطة:

1. تحميل ملف JSON يحتوي على قواعد الاستبدال
2. إدخال النص الإسبرانتو المراد معالجته
3. تحليل النص وتطبيق قواعد الاستبدال
4. إظهار النتيجة وإتاحة تنزيلها

لكن هذه العملية تتضمن تعقيدات كثيرة سنتعمق فيها لاحقاً.

## 3. تحليل الملف الرئيسي (main.py)

الملف الرئيسي هو قلب التطبيق. دعنا نحلل الأقسام الرئيسية فيه:

### 3.1 الإستيرادات والإعداد الأولي

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

تبدأ main.py باستيراد المكتبات اللازمة. هناك استخدام لـ:
- **streamlit**: لبناء واجهة المستخدم
- **re**: لاستخدام التعبيرات النمطية (Regular Expressions)
- **json**: للتعامل مع ملفات JSON
- **multiprocessing**: للمعالجة المتوازية

ثم يتم تعيين multiprocessing بالوضع "spawn" لتجنب مشاكل PicklingError:

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass
```

### 3.2 استيراد الدوال من الوحدات المساعدة

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

هنا يتم استيراد الدوال الأساسية من وحدة `esp_text_replacement_module.py` التي تتعامل مع تحويل نصوص الإسبرانتو وعمليات الاستبدال.

### 3.3 تخزين مؤقت لبيانات JSON

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    # ...
```

هذه دالة مزخرفة بـ `@st.cache_data` لتخزين نتائج تحميل ملف JSON مؤقتاً، مما يحسن الأداء عند إعادة تحميل البيانات نفسها. هذه الدالة تقرأ ملف JSON وتستخرج ثلاث قوائم استبدال مختلفة:
1. `replacements_final_list`: قائمة الاستبدال الشاملة
2. `replacements_list_for_localized_string`: قائمة الاستبدال الموضعي
3. `replacements_list_for_2char`: قائمة استبدال الجذور ثنائية الأحرف

### 3.4 إعداد واجهة المستخدم

```python
st.set_page_config(
    page_title="أداة لاستبدال الأحرف (كانجي) في النصوص الإسبرانتية",
    layout="wide"
)
st.title("استبدال نصوص إسبرانتو بالأحرف الصينية (كانجي) أو بإضافة تعليقات توضيحية بتنسيق HTML (نسخة موسعة)")
```

هذا القسم يعد واجهة المستخدم الأساسية باستخدام Streamlit.

### 3.5 تحميل ملف JSON (قواعد الاستبدال)

```python
json_options = ["デフォルトを使用する", "アップロードする"]
selected_option = st.radio(
    "كيف تريد التعامل مع ملف JSON؟ (تحميل قواعد الاستبدال من ملف JSON)",
    json_options,
    format_func=lambda x: "استخدام ملف JSON الافتراضي" if x == "デフォルトを使用する" else "تحميل ملف"
)
```

هنا يختار المستخدم إما تحميل ملف JSON افتراضي أو رفع ملف مخصص. بعد ذلك، يتم تحميل الملف المناسب وتخزين القوائم الثلاث:

```python
if selected_option == "デフォルトを使用する":
    default_json_path = "./Appの运行に使用する各类文件/最终的な替换用リスト(列表)(合并3个JSON文件).json"
    try:
        (replacements_final_list,
         replacements_list_for_localized_string,
         replacements_list_for_2char) = load_replacements_lists(default_json_path)
        # ...
```

### 3.6 تحميل الـ Placeholders

```python
placeholders_for_skipping_replacements: List[str] = import_placeholders(
    './Appの运行に使用する各类文件/占位符(placeholders)_%1854%-%4934%_文字列替换skip用.txt'
)
placeholders_for_localized_replacement: List[str] = import_placeholders(
    './Appの运行に使用する各类文件/占位符(placeholders)_@5134@-@9728@_局部文字列替换结果捕捉用.txt'
)
```

هذا الجزء مهم للغاية. يتم تحميل نوعين من العناصر النائبة (placeholders):
1. `placeholders_for_skipping_replacements`: لاستخدامها مع النص المحاط بعلامات `%..%` (النص الذي يجب تجاهله عند الاستبدال)
2. `placeholders_for_localized_replacement`: لاستخدامها مع النص المحاط بعلامات `@..@` (النص الذي يخضع لاستبدال موضعي)

### 3.7 إعدادات المعالجة المتوازية

```python
st.header("إعدادات متقدمة (المعالجة المتوازية)")
with st.expander("فتح الإعدادات المتعلقة بالمعالجة المتوازية"):
    # ...
    use_parallel = st.checkbox("استخدام المعالجة المتوازية", value=False)
    num_processes = st.number_input(
        "عدد العمليات المتزامنة",
        min_value=2, max_value=4, value=4, step=1
    )
```

هذا القسم يتيح للمستخدم تفعيل المعالجة المتوازية لتسريع معالجة النصوص الكبيرة.

### 3.8 اختيار تنسيق الإخراج

```python
options = {
    'HTML格式_Ruby文字_大小调整': 'HTML格式_Ruby文字_大小调整',
    'HTML格式_Ruby文字_大小调整_汉字替换': 'HTML格式_Ruby文字_大小调整_汉字替换',
    # ...
}
options_arabic_labels = {
    'HTML格式_Ruby文字_大小调整': "تنسيق HTML مع تعليقات Ruby وضبط الحجم",
    # ...
}
```

يعرض هذا القسم خيارات تنسيق الإخراج المختلفة للمستخدم، مع تعيين تسميات عربية مقابلة للخيارات.

### 3.9 معالجة النص وتطبيق الاستبدال

```python
with st.form(key='profile_form'):
    # ...
    if submit_btn:
        st.session_state["text0_value"] = text0
        if use_parallel:
            processed_text = parallel_process(
                text=text0,
                num_processes=num_processes,
                # ...
            )
        else:
            processed_text = orchestrate_comprehensive_esperanto_text_replacement(
                text=text0,
                # ...
            )
```

هذا هو قلب المعالجة. عند النقر على زر الإرسال، يتم تنفيذ إحدى الخطوات التالية:
1. استخدام `parallel_process` إذا كانت المعالجة المتوازية مفعّلة
2. استخدام `orchestrate_comprehensive_esperanto_text_replacement` للمعالجة العادية (غير المتوازية)

بعد ذلك، يتم تطبيق التغييرات على تنسيق أحرف الإسبرانتو (x_to_circumflex, hat_to_circumflex, إلخ) وإضافة رأس وتذييل HTML إذا لزم الأمر.

### 3.10 عرض النتائج وإتاحة التنزيل

```python
if processed_text:
    MAX_PREVIEW_LINES = 250
    lines = processed_text.splitlines()
    # ... (عرض معاينة محدودة للنصوص الطويلة)

    if "HTML" in format_type:
        tab1, tab2 = st.tabs(["معاينة HTML", "النتيجة (كود HTML)"])
        # ...
    else:
        tab3_list = st.tabs(["النص الناتج"])
        # ...

    download_data = processed_text.encode('utf-8')
    st.download_button(
        label="تنزيل النتيجة",
        data=download_data,
        file_name="نتيجة_الاستبدال.html",
        mime="text/html"
    )
```

أخيراً، يتم عرض النتائج في علامات تبويب مختلفة (معاينة HTML أو النص الناتج) وإتاحة تنزيل النتائج كملف HTML.

## 4. تحليل وحدة استبدال النصوص (esp_text_replacement_module.py)

هذه الوحدة هي محرك المعالجة الرئيسي للتطبيق. دعنا نلقي نظرة على المكونات الرئيسية:

### 4.1 قواميس تحويل أحرف الإسبرانتو

```python
x_to_circumflex = {
    'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
    'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'
}
# وقواميس أخرى لتحويلات مختلفة
```

تعريف قواميس لتحويل أحرف الإسبرانتو بين الصيغ المختلفة (صيغة cx، صيغة c^، وصيغة ĉ).

### 4.2 دوال تحويل الأحرف الأساسية

```python
def replace_esperanto_chars(text, char_dict: Dict[str, str]) -> str:
    # استبدال كل حرف حسب القاموس المعطى
    for original_char, converted_char in char_dict.items():
        text = text.replace(original_char, converted_char)
    return text

def convert_to_circumflex(text: str) -> str:
    # تحويل النص إلى صيغة الأحرف ذات العلامة المميزة (ĉ, ĝ, ...)
    text = replace_esperanto_chars(text, hat_to_circumflex)
    text = replace_esperanto_chars(text, x_to_circumflex)
    return text
```

دوال أساسية لتحويل أحرف الإسبرانتو بين الصيغ المختلفة.

### 4.3 دالة الاستبدال الآمن (safe_replace)

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    تتلقى قائمة من الثلاثيات (old, new, placeholder) وتقوم بـ:
    1. استبدال old بـ placeholder
    2. ثم استبدال placeholder بـ new
    هذا يمنع التضارب في حالات الاستبدال المتداخلة
    """
    valid_replacements = {}
    # أولاً old→placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # ثم placeholder→new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

هذه دالة حاسمة في عملية الاستبدال. إنها تستبدل النص بطريقة غير مباشرة عبر عناصر نائبة (placeholders) لتجنب مشكلات الاستبدال المتداخل. على سبيل المثال، لاستبدال "am" بـ "حب" و"ami" بـ "صديق"، يتم أولاً استبدال "ami" بعنصر نائب فريد، ثم استبدال "am" بالنص المطلوب، وأخيراً استبدال العنصر النائب بالنص المطلوب.

### 4.4 معالجة العلامات الخاصة (% و @)

```python
def find_percent_enclosed_strings_for_skipping_replacement(text: str) -> List[str]:
    """استخراج النص المحاط بعلامتي %...% (للتجاهل)"""
    # ...

def create_replacements_list_for_intact_parts(text: str, placeholders: List[str]) -> List[Tuple[str, str]]:
    """إنشاء قائمة استبدال للأجزاء التي يجب تجاهلها"""
    # ...

def find_at_enclosed_strings_for_localized_replacement(text: str) -> List[str]:
    """استخراج النص المحاط بعلامتي @...@ (للاستبدال الموضعي)"""
    # ...

def create_replacements_list_for_localized_replacement(text, placeholders: List[str],
                                                      replacements_list_for_localized_string: List[Tuple[str, str, str]]
                                                      ) -> List[List[str]]:
    """إنشاء قائمة استبدال للأجزاء التي تخضع لاستبدال موضعي"""
    # ...
```

هذه الدوال تتعامل مع العلامات الخاصة التي تسمح للمستخدمين بالتحكم في كيفية استبدال أجزاء معينة من النص.

### 4.5 دالة الاستبدال الشاملة

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text,
    placeholders_for_skipping_replacements: List[str],
    replacements_list_for_localized_string: List[Tuple[str, str, str]],
    placeholders_for_localized_replacement: List[str],
    replacements_final_list: List[Tuple[str, str, str]],
    replacements_list_for_2char: List[Tuple[str, str, str]],
    format_type: str
) -> str:
    """
    دالة رئيسية لاستبدال نص إسبرانتو حسب قواعد متعددة.
    الخطوات:
    1) توحيد المسافات + تحويل أحرف الإسبرانتو إلى صيغة موحدة
    2) استبدال مؤقت للأجزاء المحاطة بـ %...%
    3) استبدال موضعي للأجزاء المحاطة بـ @...@
    4) استبدال شامل
    5) استبدال الجذور ثنائية الأحرف (مرتين)
    6) استرجاع العناصر النائبة
    7) تنسيق HTML إذا لزم الأمر
    """
    # ...
```

هذه هي الدالة المركزية التي تنسق عملية الاستبدال بأكملها. تنفذ خطوات متعددة مترابطة لضمان استبدال نصوص الإسبرانتو بشكل صحيح.

### 4.6 المعالجة المتوازية

```python
def process_segment(lines: List[str], ...) -> str:
    """دالة مساعدة للمعالجة المتوازية"""
    # ...

def parallel_process(text: str, num_processes: int, ...) -> str:
    """
    تقسيم النص إلى أجزاء ومعالجتها بالتوازي
    """
    # ...
```

هذه الدوال تدعم المعالجة المتوازية، مما يسمح بمعالجة النصوص الكبيرة بشكل أسرع.

## 5. تحليل وحدة إنشاء ملفات JSON (esp_replacement_json_make_module.py)

هذه الوحدة تساعد في إنشاء وهيكلة ملفات JSON الخاصة بالاستبدال. إليك المكونات الرئيسية:

### 5.1 دوال قياس عرض النص وإدراج فواصل السطر

```python
def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
    """حساب العرض الإجمالي للنص باستخدام قاموس أعراض الأحرف"""
    # ...

def insert_br_at_half_width(text, char_widths_dict: Dict[str, int]) -> str:
    """إدراج <br> عند منتصف عرض النص تقريباً"""
    # ...

def insert_br_at_third_width(text, char_widths_dict: Dict[str, int]) -> str:
    """إدراج <br> عند ثلث وثلثي عرض النص تقريباً"""
    # ...
```

هذه الدوال تساعد في تنسيق النص، خاصة عند إنشاء تعليقات توضيحية طويلة في HTML.

### 5.2 دالة تنسيق الإخراج

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    """
    دمج نص إسبرانتو (main_text) مع محتوى التعليق (ruby_content)
    بالتنسيق المحدد في format_type
    """
    if format_type == 'HTML格式_Ruby文字_大小调整':
        # حساب نسبة عرض محتوى التعليق إلى عرض النص الرئيسي
        # وتطبيق تنسيق مناسب بناءً على هذه النسبة
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main
        if ratio_1 > 6:
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        # ... المزيد من الحالات
```

هذه الدالة مسؤولة عن تنسيق النص المستبدل حسب التنسيق المحدد. تقوم بحساب نسبة عرض محتوى التعليق إلى عرض النص الرئيسي وتطبيق تنسيق مناسب.

### 5.3 دوال المعالجة المتوازية لبناء قواميس الاستبدال

```python
def process_chunk_for_pre_replacements(chunk: List[List[str]], replacements: List[Tuple[str, str, str]]) -> Dict[str, List[str]]:
    """معالجة جزء من البيانات لإنشاء قاموس استبدال جزئي"""
    # ...

def parallel_build_pre_replacements_dict(E_stem_with_Part_Of_Speech_list: List[List[str]], replacements: List[Tuple[str, str, str]], num_processes: int = 4) -> Dict[str, List[str]]:
    """بناء قاموس استبدال شامل باستخدام المعالجة المتوازية"""
    # ...
```

هذه الدوال تساعد في بناء قواميس الاستبدال بشكل متوازٍ لتحسين الأداء.

## 6. تحليل صفحة إنشاء ملفات JSON

هذه الصفحة تسمح للمستخدمين بإنشاء ملفات JSON مخصصة لقواعد الاستبدال. دعنا نلقي نظرة على العناصر الرئيسية:

### 6.1 تحميل وإعداد البيانات الأولية

```python
verb_suffix_2l = {
    'as':'as', 'is':'is', 'os':'os', 'us':'us','at':'at','it':'it','ot':'ot',
    'ad':'ad','iĝ':'iĝ','ig':'ig','ant':'ant','int':'int','ont':'ont'
}
AN = [['dietan', '/diet/an/', '/diet/an'], ['afrikan', '/afrik/an/', '/afrik/an'], # ... قائمة كلمات تنتهي بـ 'an'
# ... قوائم وبيانات أخرى
```

تعريف بيانات أولية مثل لواحق الأفعال وقوائم الكلمات التي تتبع أنماطاً معينة.

### 6.2 تحميل ملف CSV (جذور الإسبرانتو والترجمات)

```python
csv_choice = st.radio("كيف تريد التعامل مع ملف CSV؟", ("رفع ملف CSV", "استخدام الملف الافتراضي"))
# ... معالجة الخيار المحدد
```

هذا القسم يسمح للمستخدم باختيار كيفية توفير بيانات CSV التي تربط بين جذور الإسبرانتو والترجمات أو الأحرف الصينية.

### 6.3 تحميل ملفات JSON (تجزئة الجذور والنص المستبدل)

```python
json_choice = st.radio("1. كيف تريد التعامل مع ملف JSON الخاص بتجزئة الجذور الإسبرانتية؟", ("رفع ملف JSON", "استخدام الملف الافتراضي"))
# ... معالجة الخيار المحدد

json_choice2 = st.radio("2. كيف تريد التعامل مع ملف JSON الخاص بالنص المستبدل؟", ("رفع ملف JSON", "استخدام الملف الافتراضي"))
# ... معالجة الخيار المحدد
```

يسمح هذا القسم للمستخدم بتحميل ملفات JSON مخصصة أو استخدام الملفات الافتراضية.

### 6.4 إنشاء ملف JSON للاستبدال

```python
if st.button("إنشاء ملف JSON للاستبدال"):
    with st.spinner("جاري إنشاء ملف JSON للاستبدال... يرجى الانتظار قليلاً."):
        # ... خطوات متعددة لإنشاء ملف JSON
```

هذا القسم يقوم بعملية إنشاء ملف JSON عندما ينقر المستخدم على الزر. يتضمن العديد من الخطوات المعقدة مثل:
1. فتح وتحميل بيانات إضافية
2. إنشاء قاموس مؤقت لجذور الإسبرانتو
3. دمج بيانات CSV مع القاموس المؤقت
4. ترتيب قائمة الاستبدال حسب طول الكلمات
5. استخدام العناصر النائبة لتجنب التضارب
6. معالجة البيانات باستخدام `safe_replace` (إما متوازياً أو تسلسلياً)
7. إجراء تعديلات وتحسينات على قاموس الاستبدال
8. إنشاء وتنظيم قوائم الاستبدال النهائية
9. تصدير البيانات كملف JSON

## 7. تفاصيل تقنية مهمة

### 7.1 آلية العناصر النائبة (Placeholders)

أحد الجوانب الحاسمة في التطبيق هو استخدام العناصر النائبة (placeholders) لتجنب مشكلات الاستبدال المتداخل. تعمل هذه الآلية على النحو التالي:

1. العناصر النائبة هي سلاسل نصية فريدة (مثل `$20987$` أو `@20374@`) مخزنة في ملفات نصية منفصلة.
2. عند استبدال النصوص، يتم أولاً استبدال النص الأصلي بعنصر نائب فريد.
3. بعد الانتهاء من جميع الاستبدالات، يتم استبدال العناصر النائبة بالنص المستبدل النهائي.

هذه الآلية تحل مشكلة مهمة: عندما تكون هناك كلمات متداخلة مثل "ami" و "am"، فإن استبدال "am" أولاً سيؤثر على "ami". لكن باستخدام العناصر النائبة، يمكن تجنب هذه المشكلة.

### 7.2 ترتيب الاستبدالات حسب الأولوية

يستخدم التطبيق نظاماً معقداً لترتيب الاستبدالات حسب الأولوية:

```python
# مثال مبسط للترتيب حسب طول الكلمة (الكلمات الأطول لها أولوية أعلى)
temporary_replacements_list_2 = sorted(temporary_replacements_list_1, key=lambda x: x[2], reverse=True)
```

عموماً، ترتيب الأولوية يتبع هذه القواعد:
1. الكلمات الأطول تُستبدل أولاً (لتجنب التداخل مع الكلمات الأقصر)
2. الكلمات الكاملة لها أولوية على الأجزاء والجذور
3. التراكيب الخاصة (مثل انتهاء الكلمة بـ "an" أو "on") لها قواعد أولوية خاصة

هذا الترتيب هو أحد الجوانب الأكثر تعقيداً في التطبيق، ويتطلب عناية خاصة عند تعديل قواعد الاستبدال.

### 7.3 معالجة خصائص لغة الإسبرانتو

يتعامل التطبيق مع خصائص لغة الإسبرانتو بطريقة منهجية:

1. **معالجة الأحرف الخاصة**: تحويل بين صيغ مختلفة للأحرف الخاصة (ĉ، cx، c^).
2. **تحليل الجذور والتصريفات**: تحديد جذور الكلمات وتصريفاتها (مثل جذر "am" مع لاحقة الفعل "as").
3. **معالجة اللواصق**: التعامل مع السوابق واللواحق مثل "an" (للعضوية) و "on" (للكسور).

هذا يتطلب فهماً جيداً لقواعد لغة الإسبرانتو، والتي يتم تطبيقها من خلال القواعد المعقدة في التطبيق.

## 8. المعالجة المتوازية

يدعم التطبيق المعالجة المتوازية لتحسين الأداء مع النصوص الكبيرة. إليك كيفية عملها:

### 8.1 في الصفحة الرئيسية

```python
def parallel_process(
    text: str,
    num_processes: int,
    # ... المعلمات الأخرى
) -> str:
    # تقسيم النص إلى أجزاء
    lines = re.findall(r'.*?\n|.+$', text)
    # ... تحديد عدد الأسطر لكل عملية
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [
                (
                    lines[start:end],
                    # ... المعلمات الأخرى
                )
                for (start, end) in ranges
            ]
        )
    return ''.join(results)
```

هذه الدالة تقسم النص إلى أجزاء (عادة أسطر) ومعالجة كل جزء في عملية منفصلة، ثم تجمع النتائج.

### 8.2 في صفحة إنشاء ملفات JSON

```python
def parallel_build_pre_replacements_dict(
    E_stem_with_Part_Of_Speech_list: List[List[str]],
    replacements: List[Tuple[str, str, str]],
    num_processes: int = 4
) -> Dict[str, List[str]]:
    # تقسيم البيانات
    total_len = len(E_stem_with_Part_Of_Speech_list)
    chunk_size = -(-total_len // num_processes)
    # ... إنشاء الأجزاء
    with multiprocessing.Pool(num_processes) as pool:
        partial_dicts = pool.starmap(
            process_chunk_for_pre_replacements,
            [(chunk, replacements) for chunk in chunks]
        )
    # دمج القواميس الجزئية
    merged_dict = {}
    # ... دمج النتائج
    return merged_dict
```

هذه الدالة تقسم قائمة الكلمات الإسبرانتية وتعالج كل جزء بالتوازي لبناء قاموس استبدال شامل.

## 9. آلية التعليقات التوضيحية (Ruby)

تعليقات Ruby هي تقنية HTML لعرض تعليقات نصية صغيرة فوق النص الرئيسي. يستخدم التطبيق تقنية Ruby بطريقة متطورة:

### 9.1 إنشاء وتنسيق تعليقات Ruby

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    if format_type == 'HTML格式_Ruby文字_大小调整':
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main
        if ratio_1 > 6:
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        # ... المزيد من الحالات
```

هذه الدالة تختار حجم وتنسيق التعليق التوضيحي بناءً على نسبة عرض التعليق إلى عرض النص الرئيسي.

### 9.2 تنسيق CSS المتقدم للتعليقات

```python
ruby_style_head="""<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>大多数の环境中で正常に运行するRuby显示功能</title>
    <style>
    html, body {
      -webkit-text-size-adjust: 100%;
      -moz-text-size-adjust: 100%;
      -ms-text-size-adjust: 100%;
      text-size-adjust: 100%;
    }

      :root {
        --ruby-color: blue;
        --ruby-font-size: 0.5em;
      }
      # ... المزيد من تعريفات CSS
```

تعريف CSS متقدم لضمان عرض التعليقات التوضيحية بشكل صحيح ومتناسق في مختلف المتصفحات.

## 10. دورة الاستبدال الكاملة في صفحة إنشاء ملفات JSON

لفهم كيفية إنشاء ملف JSON للاستبدال، إليك الخطوات المفصلة:

### 10.1 تحميل بيانات الإسبرانتو الأساسية

```python
# (1) فتح قائمة الكلمات الإسبرانتية مع أجزاء الكلام
with open("./Appの运行に使用する各类文件/PEJVO(世界语全部单词列表)'全部'について、词尾(a,i,u,e,o,n等)をcutし、comma(,)で隔てて词性と併せて记录した列表(E_stem_with_Part_Of_Speech_list).json", "r", encoding="utf-8") as g:
    E_stem_with_Part_Of_Speech_list = json.load(g)

# (2) تخزين جذور الإسبرانتو (حوالي 11137) في قاموس
temporary_replacements_dict = {}
with open("./Appの运行に使用する各类文件/世界语全部词根_约11137个_202501.txt", 'r', encoding='utf-8') as file:
    E_roots = file.readlines()
    for E_root in E_roots:
        E_root = E_root.strip()
        if not E_root.isdigit():
            temporary_replacements_dict[E_root] = [E_root, len(E_root)]
```

تبدأ العملية بتحميل البيانات الأساسية: قائمة كلمات الإسبرانتو مع أجزاء الكلام (فعل، اسم، صفة، إلخ) وقائمة جذور الإسبرانتو.

### 10.2 دمج بيانات CSV المستخدم

```python
# (3) استخدام CSV_data_imported لإنشاء قاموس "جذر إسبرانتو → (أحرف صينية أو ترجمة)"
for *, (E*root, hanzi_or_meaning) in CSV_data_imported.iterrows():
    if pd.notna(E_root) and pd.notna(hanzi_or_meaning) \
        and '#' not in E_root and (E_root != '') and (hanzi_or_meaning != ''):
        temporary_replacements_dict[E_root] = [
            output_format(E_root, hanzi_or_meaning, format_type, char_widths_dict),
            len(E_root)
        ]
```

يتم دمج بيانات CSV التي قدمها المستخدم (جذور الإسبرانتو والترجمات أو الأحرف الصينية) مع القاموس المؤقت.

### 10.3 ترتيب وإعداد قائمة الاستبدال

```python
# (4) ترتيب (الكلمات الأطول أولاً)
temporary_replacements_list_1 = []
for old, new in temporary_replacements_dict.items():
    temporary_replacements_list_1.append((old, new[0], new[1]))
temporary_replacements_list_2 = sorted(temporary_replacements_list_1, key=lambda x: x[2], reverse=True)

# (5) استخدام العناصر النائبة لتجنب التضارب
temporary_replacements_list_final = []
for kk in range(len(temporary_replacements_list_2)):
    temporary_replacements_list_final.append([
        temporary_replacements_list_2[kk][0],
        temporary_replacements_list_2[kk][1],
        imported_placeholders_for_global_replacement[kk]
    ])
```

يتم ترتيب قائمة الاستبدال حسب طول الكلمات وإعدادها باستخدام العناصر النائبة.

### 10.4 بناء قاموس الاستبدال الأولي

```python
# (6) استخدام safe_replace لمعالجة البيانات الكبيرة
if use_parallel:
    pre_replacements_dict_1 = parallel_build_pre_replacements_dict(
        E_stem_with_Part_Of_Speech_list,
        temporary_replacements_list_final,
        num_processes
    )
else:
    # ... معالجة تسلسلية مع عرض شريط التقدم
```

يتم بناء قاموس الاستبدال الأولي باستخدام المعالجة المتوازية أو التسلسلية.

### 10.5 تطبيق قواعد محددة لتحسين الاستبدال

```python
# (8) تطبيق قواعد معقدة لتحسين الاستبدال
for i,j in pre_replacements_dict_2.items():
    # j[0]:النص المستبدل, j[1]:جزء الكلام, j[2]:الأولوية
    if j[2]==20000:  # معالجة الجذور ثنائية الأحرف
        if "名词" in j[1]:  # الاسم
            for k in ["o","on",'oj']:
                # ... إنشاء استبدالات إضافية
        if "形容词" in j[1]:  # الصفة
            # ... معالجة الصفات
        # ... معالجة أجزاء الكلام الأخرى
```

يتم تطبيق قواعد محددة لتحسين الاستبدال، مثل معالجة الجذور ثنائية الأحرف وأجزاء الكلام المختلفة.

### 10.6 معالجة إعدادات المستخدم المخصصة

```python
# (9) تطبيق custom_stemming_setting_list (إعدادات تجزئة الجذور)
for i in custom_stemming_setting_list:
    if len(i)==3:
        try:
            esperanto_Word_before_replacement = i[0].replace('/', '')
            if i[1] == "dflt":
                replacement_priority_by_length = len(esperanto_Word_before_replacement)*10000
            # ... معالجة الإعدادات المخصصة
```

يتم تطبيق إعدادات تجزئة الجذور التي قدمها المستخدم.

### 10.7 إنشاء قوائم الاستبدال النهائية

```python
# (11) تحويل pre_replacements_dict_3 إلى قائمة وترتيبها حسب الأولوية
pre_replacements_list_1 = []
for old,new in pre_replacements_dict_3.items():
    if isinstance(new[1], int):
        pre_replacements_list_1.append((old,new[0],new[1]))
pre_replacements_list_2 = sorted(pre_replacements_list_1, key=lambda x: x[2], reverse=True)

# (12) إنشاء ثلاثة أنماط: حالة صغيرة، حالة كبيرة، أول حرف كبير
pre_replacements_list_4 = []
if format_type in ('HTML格式_Ruby文字_大小调整','HTML格式_Ruby文字_大小调整_汉字替换','HTML格式','HTML格式_汉字替换'):
    for old,new,place_holder in pre_replacements_list_3:
        pre_replacements_list_4.append((old,new,place_holder))
        pre_replacements_list_4.append((old.upper(), new.upper(), place_holder[:-1]+'up$'))
        # ... إنشاء نمط أول حرف كبير
```

يتم إنشاء وترتيب قوائم الاستبدال النهائية، بما في ذلك ثلاثة أنماط لكل كلمة: حالة صغيرة، حالة كبيرة، أول حرف كبير.

### 10.8 إنشاء وتصدير ملف JSON النهائي

```python
# (16) تجميع وتصدير ملف JSON النهائي
combined_data = {}
combined_data["全域替换用のリスト(列表)型配列(replacements_final_list)"] = replacements_final_list
combined_data["二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)"] = replacements_list_for_2char
combined_data["局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)"] = replacements_list_for_localized_string
download_data = json.dumps(combined_data, ensure_ascii=False, indent=2)
```

أخيراً، يتم تجميع القوائم الثلاث في ملف JSON واحد وإتاحته للتنزيل.

## 11. تفاصيل آلية الاستبدال في الصفحة الرئيسية

لنفهم بالتفصيل كيفية معالجة النص في الصفحة الرئيسية:

### 11.1 الخطوات الرئيسية في orchestrate_comprehensive_esperanto_text_replacement

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text,
    placeholders_for_skipping_replacements: List[str],
    replacements_list_for_localized_string: List[Tuple[str, str, str]],
    placeholders_for_localized_replacement: List[str],
    replacements_final_list: List[Tuple[str, str, str]],
    replacements_list_for_2char: List[Tuple[str, str, str]],
    format_type: str
) -> str:
    # 1, 2) توحيد المسافات وتحويل أحرف الإسبرانتو
    text = unify_halfwidth_spaces(text)
    text = convert_to_circumflex(text)

    # 3) استبدال مؤقت للأجزاء المحاطة بـ %...%
    replacements_list_for_intact_parts = create_replacements_list_for_intact_parts(text, placeholders_for_skipping_replacements)
    # ... تطبيق الاستبدال

    # 4) استبدال موضعي للأجزاء المحاطة بـ @...@
    tmp_replacements_list_for_localized_string_2 = create_replacements_list_for_localized_replacement(
        text, placeholders_for_localized_replacement, replacements_list_for_localized_string
    )
    # ... تطبيق الاستبدال

    # 5) استبدال عام (باستخدام قائمة replacements_final_list)
    valid_replacements = {}
    for old, new, placeholder in replacements_final_list:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new

    # 6) استبدال الجذور ثنائية الأحرف (مرتين)
    # ... تطبيق استبدال الجذور ثنائية الأحرف

    # 7) استرجاع العناصر النائبة
    # ... استبدال العناصر النائبة بالنص النهائي

    # 8) تنسيق HTML إذا لزم الأمر
    if "HTML" in format_type:
        text = text.replace("\n", "<br>\n")
        text = re.sub(r"   ", "&nbsp;&nbsp;&nbsp;", text)
        text = re.sub(r"  ", "&nbsp;&nbsp;", text)

    return text
```

هذه الدالة هي محرك المعالجة الرئيسي، وتتضمن ثمان خطوات مترابطة لمعالجة النص.

### 11.2 التعامل مع المعالجة المتوازية

```python
if submit_btn:
    st.session_state["text0_value"] = text0
    if use_parallel:
        processed_text = parallel_process(
            text=text0,
            num_processes=num_processes,
            placeholders_for_skipping_replacements=placeholders_for_skipping_replacements,
            replacements_list_for_localized_string=replacements_list_for_localized_string,
            placeholders_for_localized_replacement=placeholders_for_localized_replacement,
            replacements_final_list=replacements_final_list,
            replacements_list_for_2char=replacements_list_for_2char,
            format_type=format_type
        )
    else:
        processed_text = orchestrate_comprehensive_esperanto_text_replacement(
            text=text0,
            # ... نفس المعلمات
        )
```

بناءً على خيار المستخدم، يتم استخدام إما المعالجة المتوازية أو التسلسلية.

### 11.3 تطبيق تنسيق أحرف الإسبرانتو المختار

```python
if letter_type == '上付き文字':  # علامة تمييز فوق الحرف
    processed_text = replace_esperanto_chars(processed_text, x_to_circumflex)
    processed_text = replace_esperanto_chars(processed_text, hat_to_circumflex)
elif letter_type == '^形式':  # صيغة ^
    processed_text = replace_esperanto_chars(processed_text, x_to_hat)
    processed_text = replace_esperanto_chars(processed_text, circumflex_to_hat)
```

بعد المعالجة الرئيسية، يتم تطبيق تنسيق أحرف الإسبرانتو المختار.

### 11.4 إضافة رأس وتذييل HTML

```python
processed_text = apply_ruby_html_header_and_footer(processed_text, format_type)
```

أخيراً، يتم إضافة رأس وتذييل HTML إذا لزم الأمر، بناءً على تنسيق الإخراج المختار.

## 12. أفضل الممارسات لتعديل وتوسيع التطبيق

إذا كنت ترغب في تعديل أو توسيع هذا التطبيق، إليك بعض النصائح المهمة:

### 12.1 تعديل قواعد الاستبدال

لتعديل قواعد الاستبدال دون تغيير الكود:
1. استخدم صفحة إنشاء ملف JSON لإنشاء ملف استبدال جديد
2. قم بتخصيص ملف CSV بجذور الإسبرانتو والترجمات المطلوبة
3. قم بتخصيص ملفات JSON لتجزئة الجذور والنص المستبدل

### 12.2 إضافة تنسيقات إخراج جديدة

لإضافة تنسيق إخراج جديد:
1. أضف الخيار الجديد إلى قاموس `options` و `options_arabic_labels`
2. أضف حالة جديدة إلى دالة `output_format` في `esp_replacement_json_make_module.py`
3. أضف أي تعديلات ضرورية إلى دالة `apply_ruby_html_header_and_footer`

### 12.3 تحسين الأداء

لتحسين أداء التطبيق:
1. استخدم المعالجة المتوازية مع النصوص الكبيرة
2. حسّن خوارزميات الاستبدال (مثل استخدام التعبيرات النمطية المجمعة بدلاً من string.replace)
3. استخدم تخزين مؤقت إضافي للعمليات المتكررة

### 12.4 توسيع دعم اللغات

لدعم لغات إضافية:
1. أنشئ ملفات CSV جديدة تربط جذور الإسبرانتو بالترجمات في اللغة المطلوبة
2. عدّل تعريفات CSS إذا لزم الأمر للتعامل مع خصائص اللغة (مثل الاتجاه من اليمين إلى اليسار)

## 13. فهم تدفق البيانات الكامل

لفهم شامل للتطبيق، إليك مخطط تدفق البيانات الكامل:

### في الصفحة الرئيسية:
1. تحميل ملف JSON (يحتوي على 3 قوائم استبدال)
2. تحميل ملفات العناصر النائبة
3. إدخال نص الإسبرانتو
4. تطبيق الاستبدال (مع معالجة العلامات الخاصة % و @)
5. تطبيق تنسيق أحرف الإسبرانتو
6. إضافة رأس وتذييل HTML
7. عرض وتنزيل النتيجة

### في صفحة إنشاء ملفات JSON:
1. تحميل بيانات CSV (جذور الإسبرانتو والترجمات)
2. تحميل ملفات JSON (تجزئة الجذور والنص المستبدل)
3. تحميل بيانات إضافية (قوائم الكلمات الإسبرانتية، الجذور، إلخ)
4. بناء قاموس استبدال أولي
5. تطبيق قواعد محددة وإعدادات المستخدم
6. إنشاء قوائم الاستبدال النهائية
7. تجميع وتصدير ملف JSON نهائي

## 14. خاتمة

تطبيق استبدال نصوص الإسبرانتو هو نظام معقد ومتطور يجمع بين معالجة النصوص، والتحليل اللغوي، وتنسيق HTML، والمعالجة المتوازية لتقديم أداة قوية للمتعلمين والمترجمين. من خلال فهم البنية الداخلية والآليات المختلفة، يمكنك تعديل وتوسيع التطبيق بكفاءة، أو حتى تطبيق مفاهيمه في مشاريع أخرى.

يُظهر هذا التطبيق أيضاً قوة Streamlit في إنشاء تطبيقات ويب تفاعلية مع واجهة مستخدم بسيطة لمهام معالجة البيانات المعقدة. إنه مثال ممتاز على كيفية تطبيق مفاهيم البرمجة المتقدمة مثل المعالجة المتوازية، وتحليل اللغات، والعناصر النائبة لحل مشكلة عملية.

أتمنى أن يكون هذا الدليل المفصل قد ساعدك في فهم آلية عمل التطبيق بشكل عميق، وأن يكون نقطة انطلاق مفيدة لمزيد من الاستكشاف والتطوير.
