## Project: (Python) Phân tích trong Marketing: Lập mô hình dự đoán sự rời bỏ của khách hàng

**Project description:** Sử dụng thư viện Pandas để phân tích dữ liệu và Scikit-learn để lập mô hình dự đoán tỉ lệ rời bỏ của khách hàng đối với công ty.
Dataset được lấy từ công ty viễn thông Telco.
### 1. Sử dụng Pandas để explore data
![image](https://user-images.githubusercontent.com/118591981/203953883-0d0e1171-5fc7-4196-9543-6759e29db67b.png)

```Python
print(telco.groupby(['Churn']).mean())
print(telco.groupby(['Churn']).std())
print(telco.groupby('State')['Churn'].value_counts())

import matplotlib.pyplot as plt
import seaborn as sns

sns.distplot(telco['Day_Mins'])
plt.show()
```
![image](https://user-images.githubusercontent.com/118591981/203945587-fc58dc3f-4c08-413a-a76d-5859ecbe9fae.png) ![image](https://user-images.githubusercontent.com/118591981/203945717-c30af29e-c50e-4a30-8b7e-5c862abcf4f0.png)
![image](https://user-images.githubusercontent.com/118591981/203946021-2f4ca9c5-66cd-46de-8244-d39a87536ab4.png)
![image](https://user-images.githubusercontent.com/118591981/203946376-ec0b54b5-08b5-4c0f-885c-c4f5ea732bc1.png)
...
![image](https://user-images.githubusercontent.com/118591981/203946699-ad764842-06ca-43ca-b9fa-b4118514495e.png)

Biến Day_Mins phân bổ gần như theo luật normal distribution. Tương tự với biến Eve_Mins và Night_Mins.

```Python
sns.distplot(telco['Eve_Mins'])
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203947577-d563ea2e-8492-4835-bea0-27cc096b44cb.png)

```Python
sns.distplot(telco['Night_Mins'])
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203947718-726affff-a3b1-4328-9cd1-3d12fdc5b8d9.png)

```Python
sns.distplot(telco['Intl_Mins'])
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203947978-c0472ee3-b0cd-44c8-8741-dfcc8c51d740.png)

Tạo một mô hình hộp với 'Churn' trên trục x và 'CustServ_Calls' trên trục y. Loại bỏ các giá trị ngoại lai khỏi biểu đồ. Thêm biến thứ ba vào biểu đồ này - 'Vmail_Plan' - để trực quan hóa việc có gói thư thoại có ảnh hưởng đến số lượng cuộc gọi dịch vụ khách hàng hoặc rời bỏ hay không.
**Ta thấy có một sự khác biệt rất đáng chú ý ở đây giữa những người rời bỏ và không rời bỏ nhưng không liên quan đến gói thư thoại!** 

```python
sns.boxplot(x = 'Churn',
            y = 'CustServ_Calls',
            data = telco,
            sym = "",
            hue='Vmail_Plan')
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203948600-973cfdb6-3630-4348-96ee-01ecea1ac509.png)

Thấy rằng biến voice mail không ảnh hưởng đến việc rời bỏ, tôi thử thay bằng biến Intl_Plan

```Python
sns.boxplot(x = 'Churn',
            y = 'CustServ_Calls',
            data = telco,
            sym = "",
            hue = "Intl_Plan")
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203949052-e10f3e01-e37d-47a5-9947-55db1c44b346.png)

#### 2. Data Preparation

Đảm bảo về data type và data scale trước khi modeling.

Xem qua về data type của các biến.

![image](https://user-images.githubusercontent.com/118591981/203950335-465f7ee3-60c7-4825-9310-205ce7ae8e74.png)

**Các biến objects là các biến cần phải xử lý đưa về 0 và 1. Đối với biến Churn và Vmail_Plan tôi đưa yes và no về 1 và 0. Đối với biến State tôi sử dụng phương pháp one hot coding đưa về chuỗi nhị phân.** 

```Python
telco['Vmail_Plan'] = telco['Vmail_Plan'].replace({'yes':1,'no':0})
telco['Churn'] = telco['Churn'].replace({'yes':1,'no':0})
telco_state = pd.get_dummies(telco['State'])

print(telco_state.head())
```

Kiểm tra thử đối với biến State đã nhị phân hóa ok chưa.

![image](https://user-images.githubusercontent.com/118591981/203954240-af1ee59c-67a9-41a2-b96a-4ee42a331d69.png)

**Standalize data**

```Python
from sklearn.preprocessing import StandardScaler
telco_scaled = StandardScaler().fit_transform(telco)
telco_scaled_df = pd.DataFrame(telco_scaled, columns=["Intl_Calls", "Night_Mins"])
from sklearn.preprocessing import StandardScaler
print(telco_scaled_df.describe())
```

![image](https://user-images.githubusercontent.com/118591981/203954613-e95c31b7-8bd1-473a-bc3f-39452174d8f5.png)

**Tiếp đó tôi loại bỏ các biến không cần thiết đối với model và tạo biến mới là trung bình cuộc gọi đêm tên 'Avg_Night_Calls'**
```Python
telco = telco.drop(telco[['Area_Code','Phone']], axis=1)
telco['Avg_Night_Calls'] = telco['Night_Mins']/telco['Night_Calls']
```

### 3. Churn Prediction - Kiểm thử model dự đoán sự rời bỏ

**Sử dụng Logistic Regression**

```Python
features = telco.drop('Churn', axis=1)
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression()
clf.fit(telco[features], telco['Churn'])
clf.predict(new_customer) 
```

![image](https://user-images.githubusercontent.com/118591981/203962484-fcfd2e39-bcf9-496c-93fa-593832d65596.png)

**Sử dụng Decision Tree**

```Python
from sklearn.tree import DecisionTreeClassifier
clf = DecisionTreeClassifier()
clf.fit(telco[features],telco['Churn'])
clf.predict(new_customer)
```

![image](https://user-images.githubusercontent.com/118591981/203962484-fcfd2e39-bcf9-496c-93fa-593832d65596.png)

**Training và Testing:** Xây dựng mô hình trên mẫu Training và kiểm thử mô hình đó trên phần mẫu Testing

**Chia mẫu ra thành 2 phần Training và Testing**

Lần 1 tôi thử với test size là 30% mẫu thử.

```Python
from sklearn.model_selection import train_test_split
X = telco.drop('Churn', axis=1)
y = telco['Churn']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)
```

**Training bằng pp Random Forest**

```Python
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier()
clf.fit(X_train, y_train)
clf.score(X_test, y_test)
```

Kết quả:

![image](https://user-images.githubusercontent.com/118591981/203963864-f2864bc7-e56b-4287-96dd-48c7cb362c44.png)

**Confusion Matrix - Ma trận kết quả**

```Python
y_pred = clf.predict(X_test)
from sklearn.metrics import confusion_matrix
confusion_matrix(y_test, y_pred)
```

Kết quả:

![image](https://user-images.githubusercontent.com/118591981/203964361-f93774d3-980d-4048-bfe6-280f689831c8.png)

Lần thử này cho ra 934 dự đoán chính xác, bao gồm 842 âm tính thật và 92 dương tính thật.

**Lần 1 tôi thử với test size là 20% mẫu thử.**

```Python
from sklearn.model_selection import train_test_split
X = telco.drop('Churn', axis=1)
y = telco['Churn']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier()
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)
from sklearn.metrics import confusion_matrix
confusion_matrix(y_test,y_pred)
```

![image](https://user-images.githubusercontent.com/118591981/203965788-627956d7-cb7d-4948-9537-11a0c755d39e.png)

Bộ phân loại này có độ chính xác cao hơn so với bộ phân loại trước đó. (94.1% so với 93.4% trước đó)

**Dùng hàm tính precision_score và recall_score tự động:**

```Python
from sklearn.metrics import precision_score
precision_score(y_test, y_pred)
```

<script.py> output:
    0.8292682926829268

```Python
from sklearn.metrics import recall_score
recall_score(y_test, y_pred)
```

<script.py> output:
    0.6415094339622641
    
**ROC curve**

```Python
y_pred_prob = clf.predict_proba(X_test)[:, 1]
from sklearn.metrics import roc_curve
fpr, tpr, thresholds = roc_curve(y_test, y_pred_prob)
plt.plot(fpr, tpr)
plt.xlabel("False Positive Rate")
plt.ylabel("True Positive Rate")
plt.plot([0, 1], [0, 1], "k--")
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203966936-b7447c2d-6198-40db-b9c0-9ac618fafc3c.png)

**Tính ROC_AUC score**

```Python
from sklearn.metrics import roc_auc_score
roc_auc_score(y_test, y_pred_prob)
```

<script.py> output:
    0.8938011695906432

**Tính F1_score**

```Python
clf = RandomForestClassifier()
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)
from sklearn.metrics import f1_score
f1_score(y_test, y_pred)
```

<script.py> output:
    0.723404255319149

### 4. Tinh chỉnh Model

Có quá nhiều biến trong model, nên tôi muốn nhóm các biến lại để Model có ít biến hơn.

```Python
from sklearn.model_selection import GridSearchCV
param_grid = {'max_features': ['auto', 'sqrt', 'log2']}
grid_search = GridSearchCV(clf, param_grid)
grid_search.fit(X, y)
grid_search.best_params_
```

{'max_features': 'log2'}

```Python
from sklearn.model_selection import RandomizedSearchCV
param_dist = {"max_depth": [3, None],
              "max_features": randint(1, 11),
              "bootstrap": [True, False],
              "criterion": ["gini", "entropy"]}
random_search = RandomizedSearchCV(clf, param_dist)
random_search.fit(X,y)
random_search.best_params_
```

{'bootstrap': True, 'criterion': 'entropy', 'max_depth': None, 'max_features': 10}

```Python
importances = clf.feature_importances_
labels = X.columns[sorted_index]
plt.barh(range(X.shape[1]), importances[sorted_index], tick_label=labels)
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203969707-57fc68b6-2b7c-48e9-a16f-031e58cfcd46.png)

6 biến mới tôi sẽ nhóm lại là: Region_Code, Cost_Call, Total_Charge, Total_Minutes, Total_Calls, Min_Call

Kiểm thử xem model độ chính xác giảm hay tăng với 6 biến này:

```Python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)
clf = RandomForestClassifier()
clf.fit(X_train, y_train)
clf.score(X_test,y_test)
```

<script.py> output:
    0.954

Ta thấy độ chính xác của model precision_score cao hơn: 95.4% so với 94.1% trước đó.

Tính F1_score của model mới:

```Python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)
clf = RandomForestClassifier()
clf.fit(X_train, y_train)
clf.score(X_test,y_test)
```

<script.py> output:
    0.954

Kết quả ra tương tự. 

### Như vậy tôi đã tạo model dự báo sự rời bỏ của khách hàng dựa trên các biến cho trước trong dữ liệu của công ty Telco, nhóm các biến lại chỉ còn 6 biến nâng cao độ chính xác và hiệu suất của mô hình.
