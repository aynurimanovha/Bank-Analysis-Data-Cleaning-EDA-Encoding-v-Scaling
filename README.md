# Bank Marketinq Məlumatları — Data Cleaning, EDA, Encoding və Scaling

## Layihə haqqında

Bu layihə bank marketinq məlumatlarını maşın öyrənməsi üçün hazırlamaq məqsədilə tam data preparation pipeline həyata keçirir:

- İlkin məlumatların yoxlanılması
- Data Cleaning
- Exploratory Data Analysis (EDA)
- Kateqorik dəyişənlərin Encoding edilməsi
- Train/Test bölünməsi
- Feature Scaling
- Data keyfiyyəti və data leakage yoxlamaları
- Model üçün hazır datasetlərin yaradılması

Əsas məqsəd ilkin dataset-i təmiz, rəqəmsal və maşın öyrənməsi üçün hazır train/test datasetlərinə çevirmək və bütün preprocessing prosesini təkrarolunan saxlamaqdır.

---

## Layihənin məqsədləri

Layihə əsasən aşağıdakı dörd tələbə fokuslanır:

1. **Data Cleaning** — missing values, `unknown` kateqoriyalar və xüsusi numeric dəyərləri müəyyən etmək və idarə etmək.
2. **EDA (Exploratory Data Analysis)** — dəyişənlərin paylanmalarını, outlier-ləri, kateqorik dəyişənləri, target balansını və dəyişənlər arasındakı əlaqələri araşdırmaq.
3. **Encoding** — kateqorik dəyişənləri maşın öyrənməsi üçün uyğun rəqəmsal formata çevirmək.
4. **Scaling** — data leakage yaratmadan seçilmiş numeric feature-ləri miqyaslandırmaq.

---

## Dataset

Notebook ilkin dataset-i aşağıdakı faylda gözləyir:

```text
bank.csv
```

Faylın yolu notebook-da dəyişdirilə bilər:

```python
DATA_PATH = "bank.csv"
```

> **Qeyd:** Əgər dataset GitHub repository-sinə əlavə edilməyibsə, notebook-u işlətməzdən əvvəl `bank.csv` faylını layihə qovluğuna yerləşdirmək lazımdır.



---

## Data Cleaning

Notebook ilkin dataset-də aşağıdakı problemləri araşdırır:

- Real `NaN` dəyərləri
- `unknown` kateqoriyaları
- Xüsusi numeric kodlar
- Mənfi və ya şübhəli numeric dəyərlər
- Ümumi data tipləri və paylanmalar

### Cleaning qərarları

| Dəyişən | Görülən tədbir |
|---|---|
| `job` | `unknown` olan sətirlər az paya malik olduğu üçün silinir |
| `education` | `unknown` dəyərləri məlum dəyərlər əsasında hesablanan mode ilə əvəz edilir |
| `contact` | `unknown` ayrıca kateqoriya kimi saxlanılır |
| `poutcome` | Əvvəlki əlaqə/nəticənin olmamasını ifadə etdiyi üçün `unknown` ayrıca kateqoriya kimi saxlanılır |
| `pdays` | Xüsusi `-1` dəyəri transformasiya edilir və əlavə `was_contacted_before` feature-i yaradılır |

Bu qərarlar notebook-un daxilində ayrıca izah edilmişdir.

---

## Exploratory Data Analysis (EDA)

Notebook həm numeric, həm də categorical dəyişənləri əhatə edən strukturlaşdırılmış EDA prosesi həyata keçirir.

### Numeric analiz

Analizə aşağıdakılar daxildir:

- Descriptive statistics
- Mean, median və standard deviation
- Skewness
- Distribution qrafikləri
- Boxplot-lar
- IQR əsaslı outlier aşkarlanması

Outlier-lər əsasən **aşkarlanır və analiz edilir**, avtomatik olaraq silinmir.

### Categorical analiz

Notebook aşağıdakıları araşdırır:

- Kateqoriyaların tezliyi
- Kateqoriyalar üzrə target subscription rate
- Categorical dəyişənlərlə target arasındakı əlaqələr

### Target analizi

Target dəyişəni:

```text
deposit
```

Target-in class distribution-u analiz edilir və daha sonra binary numeric formaya keçirilir:

```text
no  → 0
yes → 1
```

### Correlation analizi

Numeric dəyişənlər arasındakı əlaqələri araşdırmaq və potensial redundant feature-ləri müəyyən etmək üçün həm **Pearson**, həm də **Spearman** correlation analizindən istifadə olunur.

---

## Encoding

Categorical dəyişənlər model təlimindən əvvəl numeric formaya çevrilir.

### Binary Encoding

Aşağıdakı dəyişənlər `0` və `1` olaraq kodlaşdırılır:

```text
deposit
default
housing
loan
```

Məsələn:

```text
no  → 0
yes → 1
```

### Ordinal Encoding

`education` təbii sıralanmasına uyğun olaraq kodlaşdırılır:

```text
primary   → 0
secondary → 1
tertiary  → 2
```

### One-Hot Encoding

Nominal categorical dəyişənlər aşağıdakı üsulla kodlaşdırılır:

```python
pd.get_dummies(..., drop_first=True)
```

Tətbiq olunan dəyişənlər:

```text
job
marital
contact
poutcome
month
```

`drop_first=True` lazımsız dummy-variable redundancy-nin qarşısını almaq üçün istifadə olunur.

---

## Train/Test Split

Encoding prosesindən sonra dataset training və testing hissələrinə bölünür.

Notebook-da:

```python
TEST_SIZE = 0.2
RANDOM_STATE = 42
```

istifadə olunur.

Bölünmə target dəyişəninə görə stratified şəkildə aparılır:

```python
train_test_split(
    X,
    y,
    test_size=TEST_SIZE,
    stratify=y,
    random_state=RANDOM_STATE
)
```

Bu yanaşma training və testing datasetlərində target class distribution-un mümkün qədər qorunmasına kömək edir.

---

## Feature Scaling

Scaling train/test bölünməsindən **sonra** həyata keçirilir.

Default scaler:

```python
SCALER = "standard"
```

Notebook aşağıdakı scaling metodlarını dəstəkləyir:

```text
standard
minmax
robust
```

Default olaraq `StandardScaler` istifadə edilir və z-score standardization həyata keçirilir.

### Vacib: Data Leakage-in qarşısının alınması

Scaler yalnız **training data** üzərində fit edilir:

```python
scaler.fit_transform(X_train[scale_cols])
```

Daha sonra eyni fitted scaler test data-ya tətbiq olunur:

```python
scaler.transform(X_test[scale_cols])
```

Beləliklə test dataset-indən əldə olunan məlumatların training preprocessing prosesinə təsir etməsinin qarşısı alınır.

---

## Scaling tətbiq olunan feature-lər

Aşağıdakı numeric feature-lər mövcud olduqları halda scale edilir:

```text
age
balance
day
duration
campaign
pdays
previous
```

Binary dəyişənlər və one-hot encoded feature-lər scale edilmir.

Target dəyişəni heç vaxt scale edilmir.

---

## Duration Feature

Notebook-da aşağıdakı konfiqurasiya seçimi mövcuddur:

```python
DROP_DURATION = False
```

Aktiv edildikdə `duration` model hazırlığından əvvəl dataset-dən çıxarılır.

Bu seçim notebook-da ayrıca izah edilmişdir, çünki `duration` müəyyən prediction ssenarilərində target ilə bağlı məlumat daşıya bilər. Buna görə feature səssiz şəkildə silinmək əvəzinə configurable saxlanılmışdır.

---

## Final Data Validation

Final datasetlər export edilməzdən əvvəl notebook aşağıdakı yoxlamaları həyata keçirir:

- Missing values
- Qalan non-numeric sütunlar
- Train/test feature sütunlarının uyğunluğu
- Ümumi feature sayı
- Feature correlation və redundancy

Final datasetlərin aşağıdakı xüsusiyyətlərə malik olması nəzərdə tutulur:

- Numeric feature-lər
- Encode edilmiş categorical dəyişənlər
- Düzgün scale edilmiş numeric feature-lər
- Ayrı binary target sütunu

---

## Output Files

Notebook aşağıdakı faylları yaradır.

### `bank_cleaned.csv`

Final model preparation mərhələsindən əvvəl təmizlənmiş dataset.

### `bank_train_ready.csv`

Encode edilmiş və scale edilmiş feature-ləri və target-i ehtiva edən training dataset.

### `bank_test_ready.csv`

Training dataset ilə eyni feature strukturuna malik encode edilmiş və scale edilmiş testing dataset.

Bu iki fayl layihənin əsas **model-ready datasetləri** hesab olunur.

---

## Reproducibility

Əsas konfiqurasiya notebook-un əvvəlində müəyyən edilir:

```python
DATA_PATH     = "bank.csv"
TARGET        = "deposit"
SCALER        = "standard"
DROP_DURATION = False
TEST_SIZE     = 0.2
RANDOM_STATE  = 42
```

Bu yanaşma preprocessing pipeline-ın yenidən icra edilməsini və ehtiyaca uyğun dəyişdirilməsini asanlaşdırır.

---

## İstifadə olunan texnologiyalar

- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- Jupyter Notebook

---

## Workflow

```text
Raw Data
   ↓
Data Inspection
   ↓
Hidden Problem Detection
   ↓
Data Cleaning
   ↓
Exploratory Data Analysis
   ↓
Encoding
   ↓
Train/Test Split
   ↓
Scaling (yalnız train üzərində fit)
   ↓
Validation
   ↓
Model-Ready Data
```

---

## Machine Learning üçün hazırlıq

Bu layihənin əsas məqsədi **data preparation** mərhələsidir. Layihədə ayrıca machine-learning modelinin train və evaluation prosesi həyata keçirilmir.

Hazırlanmış datasetlər növbəti machine-learning mərhələsində istifadə üçün nəzərdə tutulub:

```text
bank_train_ready.csv
bank_test_ready.csv
```

Bu fayllardan model training və evaluation üçün istifadə etmək mümkündür.

---

## Layihəni necə işlətmək olar?

1. Repository-ni clone edin və ya yükləyin.
2. `bank.csv` faylını layihə qovluğuna yerləşdirin.
3. `bank_analysis.ipynb` faylını Jupyter Notebook, JupyterLab və ya VS Code ilə açın.
4. Notebook-u əvvəldən sona qədər işlədin.
5. Təmizlənmiş və model üçün hazır CSV faylları avtomatik yaradılacaq.

---

## Nəticə

Bu layihə bank marketinq dataset-i üçün reproducible preprocessing pipeline təqdim edir. Pipeline aşağıdakı mərhələləri əhatə edir:

- Data Cleaning
- EDA
- Encoding
- Train/Test Split
- Scaling
- Final Validation

Nəticədə yaradılan `bank_train_ready.csv` və `bank_test_ready.csv` faylları machine-learning workflow-un növbəti mərhələsində istifadə üçün hazırlanmışdır.
