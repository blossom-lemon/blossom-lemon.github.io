## Project: (SQL)Tối ưu hóa doanh thu bán lẻ đồ thể thao?

**Project description:** Phân tích dữ liệu sản phẩm và đưa ra đề xuất để tối ưu hóa doanh thu cho công ty.

### 1. Counting missing value
Quần áo thể thao và trang phục vận động viên là một ngành công nghiệp khổng lồ, trị giá khoảng 193 tỷ đô la vào năm 2021 với dự báo tăng trưởng mạnh mẽ trong thập kỷ tới! Trong bài thực hành này, tôi đóng vai một nhà phân tích sản phẩm cho một công ty quần áo thể thao trực tuyến. Công ty đặc biệt quan tâm đến việc làm thế nào nó có thể cải thiện doanh thu. Tôi sẽ đi sâu vào dữ liệu sản phẩm bao gồm info, finance, brands, reviews và traffic để đưa ra các đề xuất.

Bắt đầu bằng việc tìm hiểu mức độ hoàn thiện của dữ liệu. Tôi xử lý dữ liệu bị thiếu cũng như các loại dữ liệu số, chuỗi và dấu thời gian để rút ra thông tin chi tiết về các sản phẩm trong cửa hàng trực tuyến. 

```SQL
%%sql
postgresql:///sports

SELECT COUNT(*) AS total_rows,
    COUNT(i.description) AS count_description,
    COUNT (f.listing_price) AS count_listing_price,
    COUNT (t.last_visited) AS count_last_visited
FROM info AS i
    LEFT JOIN finance AS f ON i.product_id = f.product_id
    LEFT JOIN traffic AS t ON i.product_id = t.product_id
```
![image](https://user-images.githubusercontent.com/118591981/203539148-29fbd786-085f-4c54-a5f3-204323f5454d.png)

### 2. Nike vs Adidas pricing
Chúng ta có thể thấy cơ sở dữ liệu chứa tổng cộng 3.179 sản phẩm. Trong số các cột đã xem, chỉ một cột — last_visited — bị thiếu hơn năm phần trăm giá trị. 
Mức giá của các sản phẩm Nike và Adidas khác nhau như thế nào? Trả lời câu hỏi này có thể giúp xây dựng một bức tranh về stocks của công ty và thị trường khách hàng. Tôi chạy một truy vấn để xem xét phân phối của list_price và số lượng cho từng giá, nhóm lại theo thương hiệu.

```SQL
%%sql

SELECT b.brand, f.listing_price::integer, COUNT(f.*)
FROM finance AS f
INNER JOIN brands AS b 
    ON f.product_id = b.product_id
WHERE listing_price > 0
GROUP BY b.brand, f.listing_price
ORDER BY listing_price DESC;
```
![image](https://user-images.githubusercontent.com/118591981/203540164-8d917835-89fb-4e57-934f-632a8d593e06.png)
![image](https://user-images.githubusercontent.com/118591981/203540234-517b44a7-338e-4494-a6d2-566a69dfb2f6.png)
....

### 3. Labeling price ranges
Như vậy có 77 mức giá khác nhau cho các sản phẩm trong cơ sở dữ liệu, điều này làm cho đầu ra của truy vấn cuối cùng khá khó phân tích. Do đó tôi gán nhãn cho các phạm vi giá khác nhau, nhóm theo thương hiệu và khoảng giá. 

```SQL
%%sql
SELECT
    brand, COUNT(*), SUM(revenue) AS total_revenue,
    CASE WHEN listing_price < 42 THEN 'Budget'
        WHEN listing_price < 74 THEN 'Average'
        WHEN listing_price < 129 THEN 'Expensive'
        ELSE 'Elite' END AS price_category
FROM brands AS b
    LEFT JOIN finance AS f USING (product_id)
WHERE brand IS NOT NULL
GROUP BY brand, price_category
ORDER BY total_revenue DESC;
```
![image](https://user-images.githubusercontent.com/118591981/203540873-906cfb1e-fee2-48c6-9cab-bf3ba9b59f69.png)

### 4. Average discount by brand
Việc nhóm các sản phẩm theo thương hiệu và phạm vi giá cho thấy rằng các mặt hàng Adidas tạo ra tổng doanh thu cao hơn trong mọi phạm vi giá! Cụ thể, các sản phẩm Adidas "Elite" có giá từ 129$ trở lên thường tạo ra doanh thu cao nhất, vì vậy công ty có thể tăng doanh thu bằng cách chuyển lượng hàng tồn kho của họ sang các sản phẩm này có tỷ lệ lớn hơn! Tuy nhiên, giá niêm yết có thể không phải là giá cuối cùng mà sản phẩm được bán. Để hiểu rõ hơn về doanh thu, tôi xem xét chiết khấu.

```SQL
%%sql
postgresql:///sports
SELECT
    brand, AVG(discount) AS average_discount
FROM brands AS b
    LEFT JOIN finance AS f USING (product_id)
WHERE brand IS NOT NULL
GROUP BY brand;
```
![image](https://user-images.githubusercontent.com/118591981/203541487-d438bc10-d3df-47de-aad3-82014ee62a8e.png)

### 5. Correlation between revenue and reviews
Có thể thấy không có giảm giá nào được cung cấp cho các sản phẩm của Nike! Ngược lại, không chỉ các sản phẩm của Adidas tạo ra doanh thu cao nhất mà các sản phẩm này còn được giảm giá mạnh! Để cải thiện doanh thu hơn nữa, công ty có thể cố gắng giảm số lượng chiết khấu được cung cấp cho các sản phẩm của Adidas và theo dõi doanh số bán hàng để xem liệu nó có ổn định hay không. Ngoài ra, có thể thử giảm giá nhỏ cho các sản phẩm của Nike. Điều này sẽ làm giảm doanh thu trung bình cho các sản phẩm này, nhưng có thể tăng doanh thu tổng thể nếu số lượng sản phẩm Nike bán ra tăng lên.

Bây giờ tôi muốn xem xét Hệ số tương quan (Correlation) giữa doanh thu và reviews.

```SQL
%%sql
SELECT
    CORR(reviews, revenue) AS review_revenue_corr
FROM reviews AS r
    LEFT JOIN finance AS f USING (product_id);
```
![image](https://user-images.githubusercontent.com/118591981/203542214-39b73cf3-dcda-4f1d-aafe-0144a48f0fa5.png)

### 6. Ratings and reviews by product description length
Như vậy, có một mối tương quan possitif mạnh giữa doanh thu và đánh giá. Điều này có nghĩa là, nếu chúng ta có thể nhận được nhiều đánh giá hơn trên trang web của công ty, thì có thể tăng doanh số bán những mặt hàng đó với số lượng đánh giá lớn hơn. Tiếp theo, tôi xem xét độ dài của mô tả sản phẩm có thể ảnh hưởng đến xếp hạng và đánh giá của sản phẩm hay không — như vậy, công ty có thể đưa ra hướng dẫn nội dung để liệt kê sản phẩm trên trang web của họ và kiểm tra xem điều này có ảnh hưởng đến doanh thu hay không. 

```SQL
%%sql
SELECT
    LENGTH(description)/100 AS description_length,
    ROUND(AVG(rating :: numeric),2) AS average_rating
FROM info AS i
    LEFT JOIN reviews USING (product_id)
WHERE description IS NOT NULL
GROUP BY description_length
ORDER BY description_length;
```
![image](https://user-images.githubusercontent.com/118591981/203542817-a7e66476-f245-41b4-bbe9-26b3e3d5780c.png)

### 7. Reviews by month and brand
Dường như không có một khuôn mẫu rõ ràng nào giữa độ dài của mô tả sản phẩm và xếp hạng của nó. Như đã biết, có mối tương quan tồn tại giữa reviews và revenue, một phương pháp mà công ty có thể thực hiện là chạy thử nghiệm với các quy trình bán hàng khác nhau để khuyến khích khách hàng đánh giá nhiều hơn về các giao dịch mua của họ, chẳng hạn như bằng cách giảm giá một khoản nhỏ cho các giao dịch mua trong tương lai. Tôi muốn xem số lượng đánh giá theo tháng để xem liệu có bất kỳ xu hướng hoặc lỗ hổng nào mà chúng ta có thể tìm cách khai thác hay không.

```SQL
%%sql
SELECT
    brand, 
    EXTRACT (month FROM last_visited) AS month,
    COUNT(*) AS num_reviews
FROM traffic 
    LEFT JOIN reviews USING (product_id)
    LEFT JOIN brands USING (product_id)
WHERE brand IS NOT NULL AND EXTRACT (month FROM last_visited) IS NOT NULL
GROUP BY brand, month
ORDER BY brand, month;
```
![image](https://user-images.githubusercontent.com/118591981/203543485-1cc6efa3-bf01-4fd4-b753-65de08055f94.png)
![image](https://user-images.githubusercontent.com/118591981/203543633-17edb2f7-6a09-4892-a3f2-bc17e4bfb9b1.png)
![image](https://user-images.githubusercontent.com/118591981/203543684-9c2ecc06-9541-4f16-8c9f-4b3b44d27c1b.png)

### 8. Footwear product performance
Có vẻ như số lượng bài đánh giá sản phẩm cao nhất trong quý đầu tiên của năm dương lịch, vì vậy, có thể chạy các thử nghiệm nhằm tăng số lượng bài đánh giá trong chín tháng còn lại!

Trước đó tôi đã tập trung phân tích dựa trên thương hiệu chung là NIKE và ADIDAS. Bây giờ, tôi muốn xem xét loại sản phẩm được bán. Vì không có nhãn cho loại sản phẩm (giày/quần áo/...), tôi tạo CTE để lọc mô tả cho từ khóa, sau đó sử dụng kết quả để tìm hiểu lượng hàng tồn kho của công ty bao gồm các sản phẩm giày dép và doanh thu trung bình do các mặt hàng này tạo ra .

```SQL
%%sql

WITH footwear AS
    (
    SELECT description, revenue
    FROM info 
        LEFT JOIN finance USING (product_id)
    WHERE description ILIKE '%shoe%'
        OR description ILIKE '%trainer%'
        OR description ILIKE '%foot%'
          AND description IS NOT NULL
    )

SELECT
    COUNT(*) AS num_footwear_products,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS median_footwear_revenue
FROM footwear;
```
![image](https://user-images.githubusercontent.com/118591981/203544358-64a84a0d-9d54-437d-97e1-1622da51ded3.png)

### 9. Clothing product performance
Trước đó, tôi đã thấy có 3.117 sản phẩm không thiếu giá trị mô tả (phần 1). Trong đó có 2.700 sản phẩm giày dép, chiếm khoảng 85% hàng tồn kho của công ty. Chúng cũng tạo ra doanh thu trung bình trên $3000 đô la! Tuy nhiên, tôi không có dữ liệu nào để xác định doanh thu trung bình của giày dép là tốt hay xấu so với các sản phẩm khác. Vì vậy, cuối cùng tôi muốn xem xét doanh thu từ giày dép khác với các sản phẩm quần áo như thế nào. 

```SQL
%%sql

WITH footwear AS
    (
    SELECT description, revenue
    FROM info 
        LEFT JOIN finance USING (product_id)
    WHERE description ILIKE '%shoe%'
        OR description ILIKE '%trainer%'
        OR description ILIKE '%foot%'
          AND description IS NOT NULL
    )

SELECT
    COUNT(*) AS num_clothing_products,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS median_clothing_revenue
FROM info 
    INNER JOIN finance USING (product_id)
WHERE description NOT IN (SELECT description FROM footwear);
```
![image](https://user-images.githubusercontent.com/118591981/203545102-532d75f8-9a8a-448c-82a2-15c7e2856cef.png)

