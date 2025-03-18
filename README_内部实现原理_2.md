# الدليل الفني لتطبيق استبدال النصوص الإسبرانتية بالأحرف الصينية (كانجي)

## مقدمة

هذا الدليل الفني موجه للمبرمجين المتوسطين في العالم العربي الذين يرغبون في فهم آلية عمل تطبيق استبدال النصوص الإسبرانتية بالأحرف الصينية (كانجي) بشكل عميق. على افتراض أنك على دراية بواجهة المستخدم الرسومية للتطبيق، سنركز هنا على الجوانب التقنية والبرمجية للتطبيق.

يتكون التطبيق من أربعة ملفات Python رئيسية:
1. **main.py** - الملف الرئيسي للتطبيق
2. **صفحة لإنشاء ملف JSON لاستبدال النصوص بالإسبرانتو بسلاسل نصية (كانجي).py** - صفحة إنشاء ملفات JSON
3. **esp_text_replacement_module.py** - وحدة برمجية لمعالجة واستبدال النصوص
4. **esp_replacement_json_make_module.py** - وحدة برمجية لإنشاء ملفات JSON للاستبدال

## نظرة عامة على التقنيات المستخدمة

يعتمد التطبيق على عدة تقنيات وأدوات:

1. **Streamlit**: منصة لإنشاء تطبيقات ويب تفاعلية باستخدام Python
2. **Pandas**: لمعالجة البيانات الجدولية (CSV)
3. **Regular Expressions (re)**: للبحث والاستبدال في النصوص
4. **Multiprocessing**: للمعالجة المتوازية وتحسين الأداء
5. **JSON**: لتخزين ونقل قواعد الاستبدال

## هيكل الملفات ودورها

### 1. main.py

هذا هو الملف الرئيسي الذي يبدأ منه التطبيق. يحتوي على:

```
main.py
├── استيراد المكتبات الضرورية
├── استيراد الوظائف من الوحدات البرمجية الأخرى
├── تعريف دالة load_replacements_lists لتحميل ملفات JSON
├── إعداد صفحة Streamlit
├── تحميل قواعد الاستبدال (من ملف JSON مرفوع أو افتراضي)
├── تحميل العناصر النائبة (placeholders)
├── إعدادات المعالجة المتوازية
├── اختيار تنسيق الإخراج
├── اختيار مصدر النص الإدخالي
└── معالجة النص واستبدال الجذور وعرض النتائج
```

### 2. صفحة لإنشاء ملف JSON لاستبدال النصوص بالإسبرانتو بسلاسل نصية (كانجي).py

هذا الملف يحتوي على واجهة مستخدم مخصصة لإنشاء ملفات JSON التي تحدد قواعد الاستبدال:

```
صفحة لإنشاء ملف JSON...
├── استيراد المكتبات الضرورية
├── تعريف المتغيرات العالمية (القوائم والجذور)
├── إعداد صفحة Streamlit
├── تنزيل الملفات النموذجية
├── اختيار تنسيق الإخراج
├── تحميل ملف CSV (جذور إسبرانتو -> ترجمات)
├── تحميل ملفات JSON (قواعد تجزئة الجذور)
├── إعدادات المعالجة المتوازية
└── إنشاء ملف JSON النهائي للاستبدال
```

### 3. esp_text_replacement_module.py

وحدة برمجية توفر الوظائف الأساسية لاستبدال النصوص:

```
esp_text_replacement_module.py
├── تعريف قواميس تحويل الأحرف الإسبرانتية
├── دوال أساسية لتحويل الأحرف
├── دالة safe_replace للاستبدال الآمن
├── دوال للتعامل مع العناصر النائبة (placeholders)
├── دوال للتعامل مع علامات % و @
├── دالة orchestrate_comprehensive_esperanto_text_replacement
└── دوال المعالجة المتوازية
```

### 4. esp_replacement_json_make_module.py

وحدة برمجية تحتوي على وظائف لإنشاء ملفات JSON للاستبدال:

```
esp_replacement_json_make_module.py
├── قواميس تحويل الأحرف الإسبرانتية
├── دوال تحويل الأحرف
├── دوال قياس عرض النص وإدراج <br>
├── دالة output_format لتحديد تنسيق الإخراج
├── دوال مساعدة للتعامل مع السلاسل النصية والعناصر النائبة
└── دوال المعالجة المتوازية لبناء قواميس الاستبدال
```

## الآليات الأساسية للتطبيق

### 1. نظام الاستبدال بالعناصر النائبة (Placeholder Replacement System)

من أهم الآليات في هذا التطبيق هو نظام الاستبدال بالعناصر النائبة. إليك شرح مفصل:

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    تأخذ قائمة من ثلاثيات (old, new, placeholder) وتقوم بالاستبدال على مرحلتين:
    1. استبدال old -> placeholder
    2. استبدال placeholder -> new
    """
    valid_replacements = {}
    # المرحلة 1: old -> placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # المرحلة 2: placeholder -> new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

**لماذا نستخدم هذه الطريقة؟**

هذه الطريقة تحل مشكلة "التداخل" في عمليات الاستبدال. مثلاً، لو لدينا:
- استبدال "abc" بـ "123"
- استبدال "ab" بـ "XY"

إذا استبدلنا "ab" أولاً، ستصبح "abc" هي "XYc"، وبالتالي لن نستطيع استبدال "abc" بـ "123".

الحل هو استخدام العناصر النائبة الفريدة كمرحلة وسيطة. يتم استبدال "abc" بـ "$12345$" و"ab" بـ "$67890$"، ثم استبدال "$12345$" بـ "123" و"$67890$" بـ "XY".

### 2. معالجة الأحرف الإسبرانتية

توفر الوحدات البرمجية آليات لتحويل الأحرف الإسبرانتية بين ثلاثة أشكال:

1. **الأحرف مع علامات مميزة**: ĉ, ĝ, ĥ, ĵ, ŝ, ŭ
2. **صيغة x**: cx, gx, hx, jx, sx, ux
3. **صيغة ^**: c^, g^, h^, j^, s^, u^

```python
# قواميس التحويل
x_to_circumflex = {'cx': 'ĉ', 'gx': 'ĝ', ...}
circumflex_to_x = {'ĉ': 'cx', 'ĝ': 'gx', ...}
# ... المزيد من القواميس

def replace_esperanto_chars(text, char_dict: Dict[str, str]) -> str:
    for original_char, converted_char in char_dict.items():
        text = text.replace(original_char, converted_char)
    return text

def convert_to_circumflex(text: str) -> str:
    text = replace_esperanto_chars(text, hat_to_circumflex)
    text = replace_esperanto_chars(text, x_to_circumflex)
    return text
```

### 3. آلية المعالجة المتوازية (Parallel Processing)

لتحسين أداء التطبيق مع النصوص الكبيرة، يتم استخدام المعالجة المتوازية:

```python
def parallel_process(
    text: str,
    num_processes: int,
    placeholders_for_skipping_replacements: List[str],
    # ... المزيد من المعاملات
) -> str:
    if num_processes <= 1:
        # استخدام المعالجة العادية
        return orchestrate_comprehensive_esperanto_text_replacement(...)
    
    # تقسيم النص إلى أجزاء
    lines = re.findall(r'.*?\n|.+$', text)
    # ... إعداد نطاقات المعالجة
    
    # إنشاء مجمع عمليات (process pool)
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [
                (
                    lines[start:end],
                    # ... المزيد من المعاملات
                )
                for (start, end) in ranges
            ]
        )
    # دمج النتائج من جميع العمليات
    return ''.join(results)
```

تتم عملية المعالجة المتوازية عبر هذه الخطوات:
1. تقسيم النص إلى مجموعات من الأسطر
2. إنشاء مجمع عمليات متعددة
3. معالجة كل مجموعة بشكل متوازٍ
4. دمج النتائج في نص واحد

### 4. آلية علامتي % و @ للتحكم بالاستبدال

يستخدم التطبيق الآليتين التاليتين للتحكم الدقيق بالاستبدال:

#### علامة % لتجاهل الاستبدال

```python
# تعريف النمط
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')

def find_percent_enclosed_strings_for_skipping_replacement(text: str) -> List[str]:
    """استخراج جميع النصوص المحاطة بـ % (بحد أقصى 50 حرفاً)"""
    matches = []
    used_indices = set()
    for match in PERCENT_PATTERN.finditer(text):
        start, end = match.span()
        if start not in used_indices and end-2 not in used_indices:
            matches.append(match.group(1))
            used_indices.update(range(start, end))
    return matches
```

#### علامة @ للاستبدال الموضعي

```python
# تعريف النمط
AT_PATTERN = re.compile(r'@(.{1,18}?)@')

def find_at_enclosed_strings_for_localized_replacement(text: str) -> List[str]:
    """استخراج جميع النصوص المحاطة بـ @ (بحد أقصى 18 حرفاً)"""
    matches = []
    used_indices = set()
    for match in AT_PATTERN.finditer(text):
        start, end = match.span()
        if start not in used_indices and end-2 not in used_indices:
            matches.append(match.group(1))
            used_indices.update(range(start, end))
    return matches
```

### 5. آلية إنشاء وتطبيق تعليقات Ruby (تنسيق HTML)

التطبيق يولد تعليقات Ruby HTML بأحجام مختلفة حسب طول النص:

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    """تحويل النص الأساسي والترجمة إلى التنسيق المطلوب"""
    if format_type == 'HTML格式_Ruby文字_大小调整':
        # حساب النسب وتطبيق فئات CSS المناسبة
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main
        if ratio_1 > 6:
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/3):
            return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
        # ... المزيد من الحالات
```

الدالة `apply_ruby_html_header_and_footer` تضيف أنماط CSS المناسبة حسب التنسيق المختار:

```python
def apply_ruby_html_header_and_footer(processed_text: str, format_type: str) -> str:
    if format_type in ('HTML格式_Ruby文字_大小调整','HTML格式_Ruby文字_大小调整_汉字替换'):
        ruby_style_head = """<!DOCTYPE html>
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
      html {
        font-size: 100%; /* 多くのブラウザは16px相当が標準 */
      }
      .text-M_M {
        font-size: 1rem!important; 
        font-family: Arial, sans-serif;
        line-height: 2.0 !important;  /* text-M_Mのline-heightとrubyのline-heightは一致させる必要がある。 */
        display: block; /* ブロック要素として扱う */
        position: relative;
      }
    """
    # ... المزيد من الأنماط والحالات
```

## التدفق العام للتطبيق

لفهم كيفية عمل التطبيق بشكل متكامل، إليك تدفق المعالجة الأساسي:

### 1. المعالجة في الصفحة الرئيسية (main.py)

1. تحميل ملف JSON الذي يحتوي على قواعد الاستبدال
2. تحميل العناصر النائبة (placeholders) من الملفات
3. اختيار مصدر النص الإدخالي (إدخال يدوي أو تحميل ملف)
4. معالجة النص باستخدام الدالة `orchestrate_comprehensive_esperanto_text_replacement` أو `parallel_process` إذا تم تفعيل المعالجة المتوازية
5. تطبيق تنسيق الأحرف الإسبرانتية المطلوب (ĉ أو cx أو c^)
6. تطبيق تنسيق HTML إذا لزم الأمر
7. عرض النتائج وإتاحة التنزيل

### 2. الدالة الرئيسية للاستبدال

الدالة `orchestrate_comprehensive_esperanto_text_replacement` هي قلب عملية الاستبدال:

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
    # 1, 2) توحيد المسافات وتحويل الأحرف الإسبرانتية
    text = unify_halfwidth_spaces(text)
    text = convert_to_circumflex(text)
    
    # 3) استبدال مؤقت للأجزاء المحاطة بـ %
    replacements_list_for_intact_parts = create_replacements_list_for_intact_parts(text, placeholders_for_skipping_replacements)
    sorted_replacements_list_for_intact_parts = sorted(replacements_list_for_intact_parts, key=lambda x: len(x[0]), reverse=True)
    for original, place_holder_ in sorted_replacements_list_for_intact_parts:
        text = text.replace(original, place_holder_)
    
    # 4) استبدال موضعي للأجزاء المحاطة بـ @
    tmp_replacements_list_for_localized_string_2 = create_replacements_list_for_localized_replacement(
        text, placeholders_for_localized_replacement, replacements_list_for_localized_string
    )
    sorted_replacements_list_for_localized_string = sorted(tmp_replacements_list_for_localized_string_2, key=lambda x: len(x[0]), reverse=True)
    for original, place_holder_, replaced_original in sorted_replacements_list_for_localized_string:
        text = text.replace(original, place_holder_)
    
    # 5) استبدال شامل (old, new, placeholder)
    valid_replacements = {}
    for old, new, placeholder in replacements_final_list:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
            
    # ... المزيد من خطوات الاستبدال والتحويل
    
    return text
```

### 3. عملية إنشاء ملف JSON (صفحة لإنشاء ملف JSON...)

1. تحميل ملف CSV يحتوي على جذور الإسبرانتو والترجمات
2. تحميل ملفات JSON التي تحدد قواعد تجزئة الجذور
3. إنشاء قاموس مؤقت يحتوي على جميع جذور الإسبرانتو
4. تطبيق بيانات CSV على القاموس
5. فرز القائمة حسب طول الكلمات (الأطول أولاً)
6. إنشاء قائمة الاستبدال النهائية باستخدام العناصر النائبة
7. معالجة بيانات الجذور والاستبدال (برزي، موازي أو تسلسلي)
8. معالجة الحالات الخاصة مثل AN و ON والبادئات واللواحق
9. تطبيق إعدادات المستخدم المخصصة
10. إنشاء قوائم الاستبدال النهائية وحفظها في ملف JSON

## تقنيات وأنماط البرمجة المستخدمة

### 1. استخدام التعليمات النمطية Type Hints

يستخدم التطبيق تعليمات نمطية لتحسين قابلية القراءة والتوثيق:

```python
from typing import List, Dict, Tuple, Optional

def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    # ...
```

### 2. نمط المُزَخرَف (Decorator Pattern)

استخدام مُزَخرَف Streamlit `@st.cache_data` لتخزين نتائج الدوال المؤقتة:

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """
    تحميل ملف JSON وإرجاع ثلاث قوائم كمجموعة:
    1) replacements_final_list
    2) replacements_list_for_localized_string
    3) replacements_list_for_2char
    """
    # ...
```

### 3. تقنية المعالجة المتوازية

استخدام وحدة `multiprocessing` لتحسين الأداء:

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # تم تعيين method البدء بالفعل
```

### 4. التعبيرات النمطية (Regular Expressions)

استخدام التعبيرات النمطية للبحث والاستبدال المعقد:

```python
# أنماط للبحث عن النصوص المحاطة بـ % و @
PERCENT_PATTERN = re.compile(r'%(.{1,50}?)%')
AT_PATTERN = re.compile(r'@(.{1,18}?)@')

# نمط للبحث عن تعليقات Ruby متطابقة
IDENTICAL_RUBY_PATTERN = re.compile(r'<ruby>([^<]+)<rt class="XXL_L">([^<]+)</rt></ruby>')
```

## تحليل معمق للخوارزميات الرئيسية

### 1. خوارزمية الاستبدال الرئيسية

الخوارزمية الرئيسية للاستبدال (في `orchestrate_comprehensive_esperanto_text_replacement`) تعمل بالترتيب التالي:

1. **توحيد وتحويل الأحرف**: توحيد المسافات وتحويل الأحرف الإسبرانتية إلى النموذج الموحد (ĉ)
2. **تحديد الأجزاء غير القابلة للاستبدال**: تحديد النصوص المحاطة بـ % واستبدالها بعناصر نائبة
3. **الاستبدال الموضعي**: تطبيق الاستبدال على النصوص المحاطة بـ @ فقط
4. **الاستبدال الشامل**: تطبيق قائمة الاستبدال الرئيسية على النص بالكامل
5. **استبدال الجذور ثنائية الأحرف**: تطبيق قواعد خاصة للجذور ثنائية الأحرف (مرتين لضمان التطبيق الصحيح)
6. **استعادة العناصر النائبة**: استبدال العناصر النائبة بالقيم المقابلة
7. **إضافة تنسيقات HTML**: تحويل السطور الجديدة إلى `<br>` والمسافات إلى `&nbsp;` إذا لزم الأمر

التعقيد الزمني للخوارزمية هو O(n*m) حيث n هو طول النص وm هو عدد قواعد الاستبدال.

### 2. خوارزمية بناء قاموس الاستبدال

في ملف إنشاء JSON، هناك خوارزمية معقدة لبناء قواعد الاستبدال:

```python
def parallel_build_pre_replacements_dict(
    E_stem_with_Part_Of_Speech_list: List[List[str]],
    replacements: List[Tuple[str, str, str]],
    num_processes: int = 4
) -> Dict[str, List[str]]:
    # تقسيم البيانات إلى أجزاء
    total_len = len(E_stem_with_Part_Of_Speech_list)
    if total_len == 0:
        return {}
    chunk_size = -(-total_len // num_processes)
    chunks = []
    # ... إعداد الأجزاء
    
    # معالجة الأجزاء بالتوازي
    with multiprocessing.Pool(num_processes) as pool:
        partial_dicts = pool.starmap(
            process_chunk_for_pre_replacements,
            [(chunk, replacements) for chunk in chunks]
        )
    
    # دمج النتائج
    merged_dict = {}
    for partial_d in partial_dicts:
        for E_root, val in partial_d.items():
            # ... دمج القواميس الجزئية
    return merged_dict
```

هذه الخوارزمية تستفيد من المعالجة المتوازية لتسريع بناء قاموس الاستبدال الضخم.

## الجوانب التقنية المتقدمة

### 1. قياس عرض النص وإدراج فواصل الأسطر

يستخدم التطبيق خوارزمية لقياس عرض النص وإدراج فواصل الأسطر `<br>`:

```python
def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
    """حساب العرض الإجمالي للنص باستخدام قاموس عرض الأحرف"""
    total_width = 0
    for ch in text:
        char_width = char_widths_dict.get(ch, 8)
        total_width += char_width
    return total_width

def insert_br_at_half_width(text, char_widths_dict: Dict[str, int]) -> str:
    """إدراج <br> عند منتصف عرض النص"""
    total_width = measure_text_width_Arial16(text, char_widths_dict)
    half_width = total_width / 2
    current_width = 0
    insert_index = None
    for i, ch in enumerate(text):
        char_width = char_widths_dict.get(ch, 8)
        current_width += char_width
        if current_width >= half_width:
            insert_index = i + 1
            break
    if insert_index is not None:
        result = text[:insert_index] + "<br>" + text[insert_index:]
    else:
        result = text
    return result
```

### 2. معالجة اللواحق والبادئات الخاصة

يحتوي التطبيق على آليات خاصة لمعالجة اللواحق والبادئات في اللغة الإسبرانتية:

```python
verb_suffix_2l = {
    'as':'as', 'is':'is', 'os':'os', 'us':'us','at':'at','it':'it','ot':'ot',
    'ad':'ad','iĝ':'iĝ','ig':'ig','ant':'ant','int':'int','ont':'ont'
}

suffix_2char_roots=['ad', 'ag', 'am', 'ar', 'as', 'at', 'av', 'di', 'ec', 'eg', 'ej', 'em', 'er', 'et', 'id', 'ig', 'il', 'in', 'ir', 'is', 'it', 'lu', 'nj', 'op', 'or', 'os', 'ot', 'ov', 'pi', 'te', 'uj', 'ul', 'um', 'us', 'uz','ĝu','aĵ','iĝ','aĉ','aĝ','ŝu','eĥ']

prefix_2char_roots=['al', 'am', 'av', 'bo', 'di', 'du', 'ek', 'el', 'en', 'fi', 'ge', 'ir', 'lu', 'ne', 'ok', 'or', 'ov', 'pi', 're', 'te', 'uz','ĝu','aĉ','aĝ','ŝu','eĥ']

standalone_2char_roots=['al', 'ci', 'da', 'de', 'di', 'do', 'du', 'el', 'en', 'fi', 'ha', 'he', 'ho', 'ia', 'ie', 'io', 'iu', 'ja', 'je', 'ju','ke', 'la', 'li', 'mi', 'ne', 'ni', 'nu', 'ok', 'ol', 'po', 'se', 'si', 've', 'vi','ŭa','aŭ','ĉe','ĝi','ŝi','ĉu']
```

هذه القوائم تُستخدم لإنشاء قواعد استبدال خاصة للواحق والبادئات والجذور المستقلة.

### 3. تكبير الحرف الأول في تعليقات Ruby

توجد آلية خاصة لتكبير الحرف الأول في تعليقات Ruby HTML:

```python
RUBY_PATTERN = re.compile(
    r'^'
    r'(.*?)'
    r'(<ruby>)'
    r'([^<]+)'
    r'(<rt[^>]*>)'
    r'([^<]*?(?:<br>[^<]*?){0,2})'
    r'(</rt>)'
    r'(</ruby>)?'
    r'(.*)'
    r'$'
)

def capitalize_ruby_and_rt(text: str) -> str:
    """تكبير الحرف الأول في تعليقات Ruby"""
    def replacer(match):
        # ... استخراج المجموعات ومعالجتها
        if g1.strip():
            return g1.capitalize() + g2 + g3 + g4 + g5 + g6 + (g7 if g7 else '') + g8
        else:
            parent_text = g3.capitalize()
            rt_text = g5.capitalize()
            return g1 + g2 + parent_text + g4 + rt_text + g6 + (g7 if g7 else '') + g8
    replaced_text = RUBY_PATTERN.sub(replacer, text)
    if replaced_text == text:
        replaced_text = text.capitalize()
    return replaced_text
```

### 4. إزالة تعليقات Ruby المتكررة

هناك آلية لإزالة تعليقات Ruby المتكررة (عندما يكون النص والتعليق متطابقين):

```python
IDENTICAL_RUBY_PATTERN = re.compile(r'<ruby>([^<]+)<rt class="XXL_L">([^<]+)</rt></ruby>')

def remove_redundant_ruby_if_identical(text: str) -> str:
    """إزالة تعليقات Ruby المتكررة عندما يكون النص والتعليق متطابقين"""
    def replacer(match: re.Match) -> str:
        group1 = match.group(1)
        group2 = match.group(2)
        if group1 == group2:
            return group1
        else:
            return match.group(0)
    replaced_text = IDENTICAL_RUBY_PATTERN.sub(replacer, text)
    return replaced_text
```

## الاستفادة من Streamlit

### 1. تقسيم التطبيق إلى صفحات

يستخدم التطبيق نظام الصفحات في Streamlit:

1. **main.py**: الصفحة الرئيسية
2. **صفحة لإنشاء ملف JSON...**: صفحة إضافية في مجلد "pages"

### 2. استخدام عناصر Streamlit التفاعلية

يستخدم التطبيق مجموعة متنوعة من عناصر Streamlit التفاعلية:

```python
# نماذج الإدخال
st.text_area("أدخل النص هنا", height=150)
st.file_uploader("اختر ملفًا", type=["txt", "csv", "md"])
st.radio("اختر خيارًا", ["الخيار 1", "الخيار 2"])
st.selectbox("اختر من القائمة", ["الخيار 1", "الخيار 2"])
st.checkbox("تفعيل الخيار", value=False)
st.number_input("أدخل رقمًا", min_value=2, max_value=4)

# أزرار وتنزيلات
st.form_submit_button("إرسال")
st.download_button("تنزيل النتيجة", data=processed_text)

# عناصر العرض
st.write("نص توضيحي")
st.success("تمت العملية بنجاح")
st.error("حدث خطأ")
st.warning("تحذير")
st.info("معلومات")

# مكونات HTML
components.html(preview_text, height=500, scrolling=True)
```

### 3. استخدام التخزين المؤقت (Cache)

يستخدم التطبيق التخزين المؤقت لتحسين الأداء:

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """تحميل ملف JSON وتخزين النتائج مؤقتًا"""
    with open(json_path, 'r', encoding='utf-8') as f:
        data = json.load(f)
    # ... معالجة البيانات
    return (
        replacements_final_list,
        replacements_list_for_localized_string,
        replacements_list_for_2char,
    )
```

## نصائح لتوسيع وتحسين التطبيق

إذا كنت ترغب في توسيع أو تحسين هذا التطبيق، إليك بعض النقاط التي يمكنك التركيز عليها:

### 1. إضافة دعم لغات جديدة

يمكنك توسيع التطبيق لدعم لغات أخرى غير العربية والصينية:

1. إنشاء ملفات CSV تحتوي على جذور إسبرانتو والترجمات بلغات أخرى
2. تعديل وظيفة `output_format` لدعم تنسيقات جديدة
3. تحديث واجهة المستخدم لعرض خيارات اللغات الجديدة

### 2. تحسين أداء المعالجة المتوازية

يمكن تحسين أداء المعالجة المتوازية:

1. تحسين استراتيجية تقسيم النص
2. تنفيذ تقنيات التخزين المؤقت للاستبدالات المتكررة
3. استخدام تقنيات أكثر تقدمًا للمزامنة وإدارة الذاكرة

### 3. إضافة ميزات جديدة

هناك العديد من الميزات التي يمكن إضافتها:

1. دعم التحويل العكسي (من الأحرف الصينية/العربية إلى إسبرانتو)
2. إضافة إحصاءات عن عمليات الاستبدال
3. تطوير واجهة أكثر تفاعلية مع معاينة فورية
4. إضافة دعم للذكاء الاصطناعي للتحسين التلقائي لقواعد الاستبدال

### 4. تحسين الواجهة وتجربة المستخدم

يمكن تحسين واجهة المستخدم:

1. إضافة سمة داكنة/فاتحة
2. تطوير واجهة أكثر استجابة للأجهزة المحمولة
3. إضافة دعم للغات واجهة المستخدم المتعددة
4. تحسين تنسيق مخرجات HTML

## خاتمة

هذا التطبيق هو مثال رائع على تطبيق Streamlit متقدم يجمع بين معالجة النصوص، التعبيرات النمطية، المعالجة المتوازية، وتوليد HTML. فهم آليات عمله سيساعدك على:

1. تطوير تطبيقات Streamlit أكثر تعقيدًا
2. تنفيذ تقنيات معالجة النصوص المتقدمة
3. استخدام المعالجة المتوازية لتحسين الأداء
4. فهم كيفية التعامل مع اللغات المختلفة والأحرف الخاصة

آمل أن يكون هذا الدليل الفني قد ساعدك على فهم آلية عمل هذا التطبيق المعقد بشكل أعمق.