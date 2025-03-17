# شرح تفصيلي لآلية عمل تطبيق تحويل نصوص الإسبرانتو إلى الأحرف الصينية (كانجي) والتعليقات التوضيحية

## المقدمة

مرحباً بك في الشرح التقني المفصل لآلية عمل تطبيق تحويل نصوص الإسبرانتو إلى الأحرف الصينية (كانجي) والتعليقات التوضيحية. هذا الدليل موجه للمبرمجين متوسطي المستوى الذين يرغبون في فهم البنية البرمجية الداخلية للتطبيق وكيفية تنفيذ وظائفه المختلفة.

سنقوم بتحليل الأجزاء الأساسية للتطبيق، وشرح التقنيات المستخدمة، والتعمق في آليات معالجة النصوص وأنماط البرمجة المطبقة. هذا الدليل يفترض أنك على دراية بأساسيات Python وتريد فهم كيفية بناء تطبيق معقد نسبياً باستخدام إطار عمل Streamlit.

## المحتويات

1. [نظرة عامة على بنية التطبيق](#نظرة-عامة-على-بنية-التطبيق)
2. [تحليل الملفات الرئيسية](#تحليل-الملفات-الرئيسية)
3. [آليات معالجة النصوص](#آليات-معالجة-النصوص)
4. [إدارة وتجهيز ملفات JSON](#إدارة-وتجهيز-ملفات-json)
5. [تقنيات تحسين الأداء](#تقنيات-تحسين-الأداء)
6. [واجهة المستخدم بواسطة Streamlit](#واجهة-المستخدم-بواسطة-streamlit)
7. [آليات معالجة الأحرف الخاصة بالإسبرانتو](#آليات-معالجة-الأحرف-الخاصة-بالإسبرانتو)
8. [إرشادات للتخصيص والتطوير](#إرشادات-للتخصيص-والتطوير)

## نظرة عامة على بنية التطبيق

يتكون التطبيق من أربعة ملفات Python رئيسية:

1. **main.py**: الصفحة الرئيسية للتطبيق، تحتوي على واجهة المستخدم الأساسية ومنطق معالجة النصوص.
2. **صفحة لإنشاء ملف JSON لاستبدال النصوص بالإسبرانتو بسلاسل نصية (كانجي).py**: صفحة ثانوية مخصصة لإنشاء ملفات JSON التي تحتوي على قواعد الاستبدال.
3. **esp_text_replacement_module.py**: وحدة تحتوي على وظائف معالجة النصوص الأساسية ومعالجة الإسبرانتو.
4. **esp_replacement_json_make_module.py**: وحدة تحتوي على وظائف لإنشاء وتحويل ملفات JSON للاستبدال.

### مخطط تدفق البيانات

```
+------------------+      +--------------------+      +----------------+
| ملفات CSV/JSON   | ---> | معالجة وتحويل      | ---> | قواعد استبدال  |
| (مدخلات)         |      | البيانات           |      | (JSON)         |
+------------------+      +--------------------+      +----------------+
                                   |
                                   v
+------------------+      +--------------------+      +----------------+
| نص الإسبرانتو    | ---> | معالجة النص        | ---> | النص بعد       |
| (مدخل المستخدم)  |      | باستخدام القواعد   |      | الاستبدال      |
+------------------+      +--------------------+      +----------------+
```

### المكونات الرئيسية

#### مكتبات خارجية أساسية
- **Streamlit**: لإنشاء واجهة المستخدم التفاعلية والويب
- **Pandas**: لمعالجة ملفات CSV
- **Multiprocessing**: للمعالجة المتوازية وتحسين الأداء
- **JSON**: للتعامل مع ملفات القواعد

#### أدوات معالجة النصوص
- محولات الأحرف الخاصة بالإسبرانتو (ĉ، ĝ، ĥ، ĵ، ŝ، ŭ)
- آليات الاستبدال الآمن باستخدام placeholders
- معالجة العلامات الخاصة (% للتجاهل، @ للاستبدال الموضعي)

#### تنسيقات الإخراج
- HTML مع تعليقات Ruby
- HTML مع تعيير حجم التعليقات
- تنسيق الأقواس
- الاستبدال البسيط

## تحليل الملفات الرئيسية

### الملف الرئيسي: main.py

هذا هو نقطة الدخول الرئيسية للتطبيق ويحتوي على معظم منطق واجهة المستخدم وتنسيق العرض.

#### الوظائف الرئيسية في main.py

1. **load_replacements_lists**: تحميل قوائم الاستبدال من ملف JSON:
```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """
    تحميل ملف JSON وإرجاع ثلاث قوائم استبدال:
    1) replacements_final_list
    2) replacements_list_for_localized_string
    3) replacements_list_for_2char
    """
    with open(json_path, 'r', encoding='utf-8') as f:
        data = json.load(f)
    replacements_final_list = data.get(
        "全域替换用のリスト(列表)型配列(replacements_final_list)", []
    )
    # ... باقي الدالة
```

هذه الدالة مهمة لأنها:
- مزينة بـ `@st.cache_data` للتخزين المؤقت (caching) لتحسين الأداء
- تقرأ ملف JSON وتستخرج ثلاث قوائم للاستبدال مختلفة
- تستخدم لتهيئة قواعد الاستبدال الرئيسية للتطبيق

2. **معالجة النص وتطبيق الاستبدال**:

التدفق العام لمعالجة النص في الملف الرئيسي:

```
تحميل ملف JSON --> تجهيز قوائم الاستبدال --> استلام نص الإدخال -->
معالجة النص (موازية أو تسلسلية) --> تطبيق تنسيق الإخراج --> عرض النتيجة
```

آلية المعالجة الرئيسية تعتمد على دالة `orchestrate_comprehensive_esperanto_text_replacement` من وحدة `esp_text_replacement_module.py` أو الإصدار المتوازي `parallel_process` إذا تم تفعيل المعالجة المتوازية.

### صفحة إنشاء ملفات JSON

هذه الصفحة متخصصة في إنشاء ملفات JSON التي تحتوي على قواعد الاستبدال من ملفات CSV الأصلية.

#### الآلية الأساسية:

1. **استيراد بيانات CSV**:
```python
CSV_data_imported = pd.read_csv(csv_buffer, encoding="utf-8", usecols=[0, 1])
```

2. **تحويل وتوسيع البيانات**: تأخذ جذور الإسبرانتو الأساسية وتوسعها لتشمل الاشتقاقات المختلفة:
```python
for *, (E*root, hanzi_or_meaning) in CSV_data_imported.iterrows():
    if pd.notna(E_root) and pd.notna(hanzi_or_meaning) \
       and '#' not in E_root and (E_root != '') and (hanzi_or_meaning != ''):
        temporary_replacements_dict[E_root] = [
            output_format(E_root, hanzi_or_meaning, format_type, char_widths_dict),
            len(E_root)
        ]
```

3. **معالجة اللواحق والبادئات**: التعامل مع اللواحق والبادئات الخاصة بالإسبرانتو
```python
# معالجة الفعل المضارع، الماضي، المستقبل وغيرها
for k1,k2 in verb_suffix_2l_2.items():
    pre_replacements_dict_3[esperanto_Word_before_replacement + k1] = [Replaced_String + k2, replacement_priority_by_length+len(k1)*10000]
```

4. **إنشاء قوائم متعددة للاستبدال**:
   - قائمة للاستبدال الشامل
   - قائمة للاستبدال الموضعي
   - قائمة خاصة بالجذور ثنائية الأحرف

### وحدة esp_text_replacement_module.py

هذه الوحدة تحتوي على وظائف معالجة النصوص الأساسية.

#### الوظائف الأساسية:

1. **safe_replace**: الاستبدال الآمن باستخدام العلامات المؤقتة (placeholders):
```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    تستقبل قائمة ثلاثيات (الأصلي، الجديد، العلامة المؤقتة)،
    وتقوم بإجراء استبدال على مرحلتين: الأصل → العلامة المؤقتة → الجديد
    """
    valid_replacements = {}
    # أولا: الأصلي → العلامة المؤقتة
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # ثم: العلامة المؤقتة → الجديد
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

هذه الدالة مهمة جدًا لأنها تمنع مشاكل الاستبدال المتداخل، مثل استبدال جزء من كلمة تم استبدالها بالفعل.

2. **دالة التنسيق الشاملة (orchestrate_comprehensive_esperanto_text_replacement)**:
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
    # تنفيذ مراحل الاستبدال المتعددة
    # ...
```

هذه الدالة تنسق عملية الاستبدال الكاملة، وتتضمن:
1. توحيد المسافات
2. تحويل أحرف الإسبرانتو إلى الشكل القياسي
3. معالجة الأجزاء المحاطة بـ `%...%` (للتجاهل)
4. معالجة الأجزاء المحاطة بـ `@...@` (للاستبدال الموضعي)
5. إجراء الاستبدال العام
6. إجراء استبدال الجذور ثنائية الأحرف
7. استعادة العلامات المؤقتة
8. تنسيق الإخراج النهائي

3. **المعالجة المتوازية**:
```python
def parallel_process(
    text: str,
    num_processes: int,
    # المعلمات الأخرى...
) -> str:
    # تقسيم النص إلى أجزاء
    # تنفيذ المعالجة المتوازية
    # دمج النتائج
```

هذه الدالة تقسم النص إلى أقسام وتعالجها بشكل متوازٍ لتسريع المعالجة، مما يفيد عند التعامل مع النصوص الكبيرة.

### وحدة esp_replacement_json_make_module.py

تحتوي هذه الوحدة على دوال تساعد في إنشاء ملفات JSON للاستبدال.

#### الوظائف الأساسية:

1. **output_format**: تنسيق المخرجات حسب النوع المطلوب:
```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    """
    تجمع نص الإسبرانتو (main_text) مع الترجمة/الكانجي (ruby_content)
    بالتنسيق المحدد (format_type)
    """
    if format_type == 'HTML格式_Ruby文字_大小调整':
        # قياس العرض وتنسيق Ruby مع ضبط الحجم
        # ...
    elif format_type == 'HTML格式':
        return f'<ruby>{main_text}<rt>{ruby_content}</rt></ruby>'
    elif format_type == 'HTML格式_汉字替换':
        return f'<ruby>{ruby_content}<rt>{main_text}</rt></ruby>'
    # ...والمزيد من التنسيقات
```

هذه الدالة مهمة لأنها تحول النص الأصلي والترجمة إلى التنسيق المطلوب، مع تكييف حجم تعليقات Ruby حسب طول النص عند الحاجة.

2. **المعالجة المتوازية لبناء قواعد الاستبدال**:
```python
def parallel_build_pre_replacements_dict(
    E_stem_with_Part_Of_Speech_list: List[List[str]],
    replacements: List[Tuple[str, str, str]],
    num_processes: int = 4
) -> Dict[str, List[str]]:
    # تقسيم البيانات
    # معالجة متوازية
    # دمج النتائج
```

هذه الدالة تستخدم المعالجة المتوازية لبناء قاموس الاستبدال بشكل أسرع.

## آليات معالجة النصوص

### آلية الاستبدال الآمن باستخدام العلامات المؤقتة (Placeholders)

أحد أكبر التحديات في معالجة النصوص هو تجنب الاستبدالات المتداخلة أو المتعارضة. يستخدم التطبيق آلية ذكية للتغلب على هذه المشكلة:

1. **تعريف العلامات المؤقتة**: مجموعات من السلاسل النصية غير المحتمل وجودها في النص الأصلي.
2. **استبدال على مرحلتين**:
   - المرحلة الأولى: استبدال النص الأصلي بعلامة مؤقتة
   - المرحلة الثانية: استبدال العلامة المؤقتة بالنص النهائي

```python
# مثال من التطبيق
for old, new, placeholder in replacements_final_list:
    if old in text:
        text = text.replace(old, placeholder)
        valid_replacements[placeholder] = new

for placeholder, new in valid_replacements.items():
    text = text.replace(placeholder, new)
```

### معالجة العلامات الخاصة (% و @)

التطبيق يدعم آليتين خاصتين لتحكم المستخدم في الاستبدال:

1. **استخدام علامة %**: لتجاهل أجزاء معينة من النص تماماً
```python
def find_percent_enclosed_strings_for_skipping_replacement(text: str) -> List[str]:
    # استخراج الأجزاء المحاطة بـ %...%
    # ...
```

2. **استخدام علامة @**: للاستبدال الموضعي داخل جزء محدد
```python
def create_replacements_list_for_localized_replacement(text, placeholders: List[str],
                                                      replacements_list_for_localized_string: List[Tuple[str, str, str]]
                                                     ) -> List[List[str]]:
    # معالجة الأجزاء المحاطة بـ @...@
    # ...
```

### معالجة أحرف الإسبرانتو الخاصة

اللغة الإسبرانتو تحتوي على أحرف خاصة (ĉ، ĝ، ĥ، ĵ، ŝ، ŭ). يوفر التطبيق عدة طرق للتحويل بين تمثيلات مختلفة:

```python
# قواميس التحويل
x_to_circumflex = {'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ', /* ... */}
circumflex_to_x = {'ĉ': 'cx', 'ĝ': 'gx', 'ĥ': 'hx', 'ĵ': 'jx', 'ŝ': 'sx', 'ŭ': 'ux', /* ... */}
x_to_hat = {'cx': 'c^', 'gx': 'g^', /* ... */}
# ... المزيد من قواميس التحويل

# دالة التحويل
def replace_esperanto_chars(text, char_dict: Dict[str, str]) -> str:
    for original_char, converted_char in char_dict.items():
        text = text.replace(original_char, converted_char)
    return text
```

## إدارة وتجهيز ملفات JSON

### آلية إنشاء ملفات JSON للاستبدال

التطبيق يتبع خطوات محددة لإنشاء ملفات JSON للاستبدال:

1. **قراءة البيانات الأولية**: قراءة ملف CSV الذي يحتوي على جذور الإسبرانتو والترجمات/الكانجي المقابلة
2. **توسيع البيانات**: إضافة اشتقاقات مختلفة (مثل صيغ الفعل المختلفة)
3. **تحديد أولويات الاستبدال**: ترتيب قواعد الاستبدال حسب الطول (الأطول أولاً) وأنواع أخرى من الأولويات
4. **إنشاء ثلاث قوائم استبدال مختلفة**:
   - قائمة للاستبدال الشامل
   - قائمة للاستبدال الموضعي
   - قائمة خاصة بالجذور ثنائية الأحرف
5. **حفظ القوائم الثلاث في ملف JSON واحد**

### بنية ملف JSON للاستبدال

ملف JSON النهائي يحتوي على ثلاثة أقسام رئيسية:

```json
{
  "全域替换用のリスト(列表)型配列(replacements_final_list)": [
    ["am", "<ruby>am<rt class=\"M_M_M\">حب</rt></ruby>", "$20987$"]
    /* ... المزيد من قواعد الاستبدال */
  ],
  "局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)": [
    /* قواعد الاستبدال الموضعية */
  ],
  "二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)": [
    /* قواعد الاستبدال للجذور ثنائية الأحرف */
  ]
}
```

كل قاعدة استبدال هي عبارة عن ثلاثية (triplet) تتكون من:
1. النص الأصلي
2. النص البديل (بالتنسيق المحدد)
3. العلامة المؤقتة (placeholder) للاستبدال الآمن

## تقنيات تحسين الأداء

### استخدام التخزين المؤقت (Caching)

التطبيق يستخدم تقنية التخزين المؤقت من Streamlit لتحسين الأداء:

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    # ...
```

المزين `@st.cache_data` يخزن نتائج الدالة في الذاكرة، بحيث لا يتم إعادة تحميل ملف JSON في كل مرة يتم فيها استدعاء الدالة مع نفس المعلمات.

### المعالجة المتوازية

التطبيق يدعم المعالجة المتوازية لتسريع معالجة النصوص الكبيرة:

```python
def parallel_process(
    text: str,
    num_processes: int,
    # ... باقي المعلمات
) -> str:
    # تقسيم النص إلى أجزاء
    lines = re.findall(r'.*?\n|.+$', text)
    # حساب الأجزاء لكل عملية
    lines_per_process = max(num_lines // num_processes, 1)
    # ... باقي المنطق

    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [
                (
                    lines[start:end],
                    # ... باقي المعلمات
                )
                for (start, end) in ranges
            ]
        )
    return ''.join(results)
```

المعالجة المتوازية تعمل عن طريق:
1. تقسيم النص إلى أجزاء (عادة حسب الأسطر)
2. معالجة كل جزء في عملية منفصلة
3. دمج النتائج مرة أخرى

### إعداد بيئة المعالجة المتوازية

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # تجاوز الخطأ إذا تم ضبط طريقة البدء بالفعل
```

هذا الجزء مهم لتجنب أخطاء Pickling في Streamlit عند استخدام المعالجة المتوازية.

## واجهة المستخدم بواسطة Streamlit

### بناء الواجهة الرئيسية

التطبيق يستخدم Streamlit لإنشاء واجهة المستخدم التفاعلية:

```python
st.set_page_config(
    page_title="أداة لاستبدال الأحرف (كانجي) في النصوص الإسبرانتية",
    layout="wide"
)

st.title("استبدال نصوص إسبرانتو بالأحرف الصينية (كانجي) أو بإضافة تعليقات توضيحية بتنسيق HTML (نسخة موسعة)")
```

### العناصر التفاعلية الرئيسية

1. **اختيار ملف JSON**:
```python
selected_option = st.radio(
    "كيف تريد التعامل مع ملف JSON؟ (تحميل قواعد الاستبدال من ملف JSON)",
    json_options,
    format_func=lambda x: "استخدام ملف JSON الافتراضي" if x == "デフォルトを使用する" else "تحميل ملف"
)
```

2. **اختيار تنسيق الإخراج**:
```python
display_options = list(options.keys())
selected_display = st.selectbox(
    "اختر تنسيق الإخراج (مطابق للتنسيق الذي تم تعريفه في ملف JSON للاستبدال):",
    display_options,
    format_func=lambda key: options_arabic_labels[key]
)
```

3. **النموذج الرئيسي لإدخال النص**:
```python
with st.form(key='profile_form'):
    # عناصر النموذج
    text0 = st.text_area(
        "الرجاء إدخال نص الإسبرانتو هنا",
        height=150,
        value=initial_text
    )
    # ... عناصر أخرى
    submit_btn = st.form_submit_button('إرسال')
    cancel_btn = st.form_submit_button("إلغاء")
```

### عرض النتائج

```python
if "HTML" in format_type:
    tab1, tab2 = st.tabs(["معاينة HTML", "النتيجة (كود HTML)"])
    with tab1:
        components.html(preview_text, height=500, scrolling=True)
    with tab2:
        st.text_area("كود HTML الناتج:", preview_text, height=300)
else:
    tab3_list = st.tabs(["النص الناتج"])
    with tab3_list[0]:
        st.text_area("النتيجة:", preview_text, height=300)
```

هذا الجزء يعرض النتائج بطريقتين مختلفتين بناءً على نوع التنسيق:
- إذا كان HTML، يعرض معاينة مباشرة للHTML ونص الكود
- وإلا، يعرض النص الناتج فقط

## آليات معالجة الأحرف الخاصة بالإسبرانتو

### تحويلات الأحرف

التطبيق يدعم ثلاثة أشكال للأحرف الخاصة بالإسبرانتو:

1. **الشكل القياسي (ĉ، ĝ، ...)**: الأحرف بالعلامات فوقها
2. **صيغة x (cx, gx, ...)**: شائعة في الكتابة الرقمية
3. **صيغة ^ (c^, g^, ...)**: بديل آخر شائع

```python
def convert_to_circumflex(text: str) -> str:
    # تحويل النص إلى الشكل القياسي بالعلامات فوق الحروف
    text = replace_esperanto_chars(text, hat_to_circumflex)
    text = replace_esperanto_chars(text, x_to_circumflex)
    return text
```

### تطبيق تنسيق HTML والـ Ruby

```python
def apply_ruby_html_header_and_footer(processed_text: str, format_type: str) -> str:
    """
    إضافة ترويسة وتذييل HTML حسب نوع التنسيق المحدد
    """
    if format_type in ('HTML格式_Ruby文字_大小调整','HTML格式_Ruby文字_大小调整_汉字替换'):
        ruby_style_head = """<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <!-- ... CSS styles ... -->
  </head>
  <body>
  <p class="text-M_M">
"""
        ruby_style_tail = "</p></body></html>"
    # ... المزيد من الأنماط
```

هذه الدالة تضيف ترويسة HTML وCSS لضبط مظهر تعليقات Ruby، وخاصة تعديل حجمها بناءً على نسبة طول التعليق إلى النص الأصلي.

## إرشادات للتخصيص والتطوير

### إضافة لغة جديدة

لإضافة لغة جديدة للتطبيق، ستحتاج إلى:

1. **إنشاء ملف CSV** يربط بين جذور الإسبرانتو والترجمات بلغتك
2. **تخصيص خيارات العرض** في قائمة `options_arabic_labels` لإضافة تسميات بلغتك
3. **تعديل واجهة المستخدم** لعرض النصوص باللغة الجديدة

### تخصيص قواعد الاستبدال

يمكنك تخصيص قواعد الاستبدال عن طريق:

1. **تعديل ملف CSV** لتغيير الترجمات
2. **تعديل ملف JSON لتجزئة الجذور** للتحكم في كيفية تقسيم الكلمات
3. **إنشاء قواعد مخصصة** للكلمات الخاصة عن طريق ملف JSON للنص المستبدل

### أفكار للتطوير

1. **إضافة دعم القوالب**: إمكانية حفظ واستدعاء إعدادات مختلفة بسهولة
2. **تحسين واجهة المستخدم**: إضافة مزيد من التخصيص والخيارات التفاعلية
3. **دعم تنسيقات إضافية**: إضافة تنسيقات جديدة أو خيارات عرض متقدمة
4. **تحسين الأداء**: المزيد من تقنيات التحسين للنصوص الكبيرة جداً

---

هذا الشرح التقني المفصل يوفر نظرة عميقة على آلية عمل التطبيق وكيفية تنفيذ وظائفه المختلفة. إذا كان لديك أي أسئلة محددة حول جوانب معينة من التطبيق، فلا تتردد في طلب المزيد من التفاصيل.
