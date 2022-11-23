## Project: Which Debts Are Worth the Bank's Effort?

**Project description:** Sử dụng hồi quy gián đoạn để đánh giá khoản nợ nào đáng thu đối với một ngân hàng.

### 1. Hồi quy gián đoạn: việc thu hồi nợ của ngân hàng
Khi một khoản nợ được coi là không thể thu hồi, ngân hàng vẫn tiếp tục thực hiện các nghiệp vụ để thu hồi món nợ này. Ngân hàng sẽ đánh giá tài khoản nợ và đưa ra mức thu hồi kỳ vọng (expected_recovery_amount). Chiến lược thu hồi được xếp hạng bắt đầu từ Level 0 dựa theo các ngưỡng kỳ vọng thu hồi nợ ($1000, $2000,...). Ở Level 0, ngân hàng không cần bỏ thêm chi phí để thu hồi, bắt đầu từ ngưỡng Level 1 trở đi, mỗi ngưỡng chi phí sẽ tăng thêm $50.

**Câu hỏi:** Liệu các level cao hơn có bị đội chi vượt quá $50 không? Bởi vì khi chi phí để thu hồi khoản nợ lớn hơn kỳ vọng thu hồi thì khoản thu đó là không đáng.

Đầu tiên, tôi upload dataset từ ngân hàng để hiểu và đánh giá sơ bộ.

import pandas as pd
import numpy as np
df = pd.read_csv('datasets/bank_data.csv') 
df.head()



### 2. Assess assumptions on which statistical inference will be based

```javascript
if (isAwesome){
  return true
}
```

### 3. Support the selection of appropriate statistical tools and techniques

<img src="images/dummy_thumbnail.jpg?raw=true"/>

### 4. Provide a basis for further data collection through surveys or experiments

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. 

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
