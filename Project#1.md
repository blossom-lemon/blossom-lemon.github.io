## Project: Which Debts Are Worth the Bank's Effort?

**Project description:** Sử dụng hồi quy gián đoạn để đánh giá khoản nợ nào đáng thu đối với một ngân hàng.

### 1. Hồi quy gián đoạn: việc thu hồi nợ của ngân hàng
Khi một khoản nợ được coi là không thể thu hồi, ngân hàng vẫn tiếp tục thực hiện các nghiệp vụ để thu hồi món nợ này. Ngân hàng sẽ đánh giá tài khoản nợ và đưa ra mức thu hồi kỳ vọng (expected_recovery_amount). Chiến lược thu hồi được xếp hạng bắt đầu từ Level 0 dựa theo các ngưỡng kỳ vọng thu hồi nợ ($1000, $2000,...). Ở Level 0, ngân hàng không cần bỏ thêm chi phí để thu hồi, bắt đầu từ ngưỡng Level 1 trở đi, mỗi ngưỡng chi phí sẽ tăng thêm $50.

**Câu hỏi:** Liệu các level cao hơn có bị đội chi vượt quá $50 không? Bởi vì khi chi phí để thu hồi khoản nợ lớn hơn kỳ vọng thu hồi thì khoản thu đó là không đáng.

Đầu tiên, tôi upload dataset từ ngân hàng để hiểu và đánh giá sơ bộ.

```python
import pandas as pd
import numpy as np
df = pd.read_csv('datasets/bank_data.csv') 
df.head()
```
![image](https://user-images.githubusercontent.com/118591981/203523919-6555282a-7814-428c-ac01-0745b4de8407.png)

### 2. Graphical exploratory data analysis
Ngân hàng đặt ra các ngưỡng của kỳ vọng thu hồi nợ là $1000, $2000,... tương ứng với Level 0, Level 1,... Để phân tích trả lời câu hỏi đặt ra, tôi phân tích vùng từ 0-2000$ tương ứng với Level 0 và 1. Mục đích là phân tích xem ngoài expected_recovery_amount còn có các yếu tố nào khác thay đổi ở ngưỡng 1000$ hay không. 

Tóm tắt về ngưỡng và kỳ vọng thu hồi:

Level 0: Expected recovery amounts >$0 and <=$1000
Level 1: Expected recovery amounts >$1000 and <=$2000
The threshold of $1000 separates Level 0 from Level 1

Trong mục 2 này, tôi phân tích mối quan hệ giữa tuổi (age) của khách hàng và kỳ vọng thu hồi.

```python
from matplotlib import pyplot as plt
import pandas as pd
%matplotlib inline
df = pd.read_csv('datasets/bank_data.csv') 
plt.scatter(x=df['expected_recovery_amount'], y=df['age'], c="g", s=2)
plt.xlim(0, 2000)
plt.ylim(0, 60)
plt.xlabel("Expected Recovery Amount")
plt.ylabel("Age")
plt.legend(loc=2)
plt.show()
```

![image](https://user-images.githubusercontent.com/118591981/203525844-c2111c11-114e-4ac5-9076-bd54cac34bb7.png)

Biểu đồ hạt đã không cho thấy bất kỳ một jump về tuổi nào ở quanh ngưỡng 1000$ của kỳ vọng thu hồi. Để chắc chắn, tôi sẽ phân tích bằng thống kê định lượng ở phần 3.

### 3. Statistical test: age vs. expected recovery amount
Tôi muốn chứng minh rằng biến tuổi (age) và giới tính (sex) không có sự thay đổi đột ngột (jump) nào ở ngưỡng 1000$. Vì cuối cùng, điều tôi dự doán và muốn chững minh là Khoản thu hồi thực tế (actual_recovery_amount) là biến phụ thuộc vào Chiến lược thu hồi (Level).

Để chứng minh biến tuổi không có gián đoạn nào ở quanh ngưỡng 1000$ tôi sẽ khảo sát ở vùng 900-1100$, sử dụng pp Kruskal-Wallis test.

```python
from scipy import stats

era_900_1100 = df.loc[(df['expected_recovery_amount']<1100) & 
                      (df['expected_recovery_amount']>=900)]
by_recovery_strategy = era_900_1100.groupby(['recovery_strategy'])
by_recovery_strategy['age'].describe().unstack()


Level_0_age = era_900_1100.loc[df['recovery_strategy']=="Level 0 Recovery"]['age']
Level_1_age = era_900_1100.loc[df['recovery_strategy']=="Level 1 Recovery"]['age']
stats.kruskal(Level_0_age,Level_1_age) 
```
![image](https://user-images.githubusercontent.com/118591981/203528428-0f9685f7-20f1-42e2-b5ef-c240d52180d7.png)

### 4. Statistical test: sex vs. expected recovery amount
Tôi cũng muốn kiểm tra xem tỷ lệ phần trăm khách hàng là nam/nữ thay đổi ngưỡng $1000 hay không. Bắt đầu bằng phạm vi từ $900 đến $1100 và sau đó điều chỉnh phạm vi này.

Tôi sử dụng crosstab và pp chi-square.

```python
crosstab = pd.crosstab(df.loc[(df['expected_recovery_amount']<1100) &
                              (df['expected_recovery_amount']>=900)]['recovery_strategy'],
                               df['sex'])
print(crosstab)

chi2_stat, p_val, dof, ex = stats.chi2_contingency(crosstab)

print("p_val = ", p_val)
```
![image](https://user-images.githubusercontent.com/118591981/203530109-646735b4-3a49-4082-a1b5-8c364eb62f62.png)

### 5. Exploratory graphical analysis: recovery amount
Như vậy, trực quan và định lượng đều chứng minh không có sự thay đổi gián đoạn nào của biến tuổi và giới tính quanh ngưỡng 1000$. Bây giờ tôi tập trung vào biến actual_recovery_amount.

```python
plt.scatter(x=df['actual_recovery_amount'], y=df['expected_recovery_amount'], c="g", s=2)
plt.xlim(900, 1100)
plt.ylim(0, 2000)
plt.xlabel("Actual Recovery Amount")
plt.ylabel("Expected Recovery Amount")
plt.legend(loc=2)
```
![image](https://user-images.githubusercontent.com/118591981/203530765-c8c46351-14ce-4a50-a97a-e67434282c17.png)

### 6. Statistical analysis: recovery amount
Một lần nữa, tôi sẽ sử dụng bài kiểm tra Kruskal-Wallis.

Trước tiên, tôi sẽ tính toán số tiền thu hồi thực tế trung bình cho những khách hàng ngay dưới và ngay trên ngưỡng bằng cách sử dụng phạm vi từ $900 đến $1100. Sau đó, chúng tôi sẽ thực hiện kiểm tra Kruskal-Wallis để xem liệu số tiền thu hồi thực tế có khác nhau ngay trên và ngay dưới ngưỡng hay không. Sau đó, tôi sẽ lặp lại các bước này cho phạm vi nhỏ hơn từ $950 đến $1050.

```python
by_recovery_strategy['actual_recovery_amount'].describe().unstack()

Level_0_actual = era_900_1100.loc[df['recovery_strategy']=='Level 0 Recovery']['actual_recovery_amount']
Level_1_actual = era_900_1100.loc[df['recovery_strategy']=='Level 1 Recovery']['actual_recovery_amount']
print("Erea 900 - 1100", stats.kruskal(Level_0_actual,Level_1_actual)) 

era_950_1050 = df.loc[(df['expected_recovery_amount']<1050) & 
                      (df['expected_recovery_amount']>=950)]
Level_0_actual = era_950_1050.loc[df['recovery_strategy']=='Level 0 Recovery']['actual_recovery_amount']
Level_1_actual = era_950_1050.loc[df['recovery_strategy']=='Level 1 Recovery']['actual_recovery_amount']
print("Erea 950 - 1050", stats.kruskal(Level_0_actual,Level_1_actual))
```
![image](https://user-images.githubusercontent.com/118591981/203531467-23a42503-94c9-4726-928d-a91261eef44e.png)

### 7. Regression modeling: no threshold
Bây giờ, tôi muốn áp dụng phương pháp dựa trên hồi quy để ước tính tác động của chương trình ở ngưỡng $1000 bằng cách sử dụng dữ liệu nằm ngay trên và dưới ngưỡng. Tôi sẽ xây dựng hai mô hình. Mô hình đầu tiên không có ngưỡng trong khi mô hình thứ hai sẽ bao gồm ngưỡng.

```python
import statsmodels.api as sm

# Define X and y
X = era_900_1100['expected_recovery_amount']
y = era_900_1100['actual_recovery_amount']
X = sm.add_constant(X)

# Build linear regression model
model = sm.OLS(y, X).fit()
predictions = model.predict(X)

# Print out the model summary statistics
model.summary()
```

![image](https://user-images.githubusercontent.com/118591981/203532514-8529161c-04ce-4dc8-948f-23d0ad11cf63.png)
![image](https://user-images.githubusercontent.com/118591981/203532615-32fd42fa-04b0-4dd3-932f-d9f9c4e1cde4.png)

### 9. Regression modeling: adjusting the window
**Hệ số hồi quy cho ngưỡng thực thống kê tác động ước tính khoảng $278. Con số này lớn hơn nhiều so với 50 đô la cho mỗi khách hàng cần thiết khi tăng Level cố gắng thu hồi của ngân hàng.**

Để chắc chắn, tôi lặp lại phân tích này cho phạm vi từ $950 đến $1050 để xem có nhận được kết quả tương tự hay không.

```python
era_950_1050 = df.loc[(df['expected_recovery_amount']<1050) & 
                      (df['expected_recovery_amount']>=950)]

# Define X and y 
X = era_950_1050[['expected_recovery_amount','indicator_1000']]
y = era_950_1050['actual_recovery_amount']
X = sm.add_constant(X)

# Build linear regression model
model = sm.OLS(y,X).fit()

# Print the model summary
model.summary()
```
![image](https://user-images.githubusercontent.com/118591981/203533449-476079a1-642e-440f-a3e7-37fbffe018ff.png)
![image](https://user-images.githubusercontent.com/118591981/203533509-d5a46120-1c27-412c-81eb-5d096cb21aa0.png)

