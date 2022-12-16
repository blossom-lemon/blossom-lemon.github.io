## Project: Phân tích Nhân sự - Power BI

**Discription:** Phân tích dữ liệu nhân sự vô cùng đa dạng các phương diện. Trong dự án này, tôi chỉ phân tích 1 vài phương diện phổ biến. Dữ liệu được sử dụng là của công ty Atlas Lab.

[Link Power BI online](https://app.powerbi.com/groups/me/reports/97394649-a696-42ae-85a9-6717fdcfa127?ctid=68cdfebb-157b-4846-ba2f-d196a9124ac0&pbi_source=linkShare&bookmarkGuid=3a8e6d67-aaff-41b8-96b6-858619766787)

Dữ liệu dưới dạng .csv và có 6 bảng, trong đó: 5 bảng là dữ liệu dimension (thông tin tra cứu) (Date, Education Level, Employee, RatingLevel, SatisfactionLevel) và 1 bảng là dữ liệu Facts (Performance Rating - đánh giá hoạt động các mặt của nhân viên). Dữ liệu được nhận đã sẵn sàng được phân tích, tuy nhiên để chắc chắn tôi kiểm tra lại format và tình trạng dữ liệu của mỗi cột trong bảng. Bước này rất nhanh vì dữ liệu gần như là hoàn thiện.

Sau đó tôi lập model xác định key liên kết các bảng.
![image](https://user-images.githubusercontent.com/118591981/208191989-0121b404-520f-4033-976b-1f13a3ebfc38.png)

Các bước tiếp theo khi cần có 1 measure mới cần viết dax function để phân tích tôi để trong bảng measure.
![image](https://user-images.githubusercontent.com/118591981/208192232-fea8e3b9-3cb4-4553-9ae9-2b52845c5218.png)

Nếu cần lập column mới phục vụ cho việc phân tích thì tôi sẽ thêm trực tiếp vào trong bảng đó luôn. Ví dụ như để phân tích độ tuổi tôi lập thêm cột Nhóm Tuổi, hoặc để track rating của 1 nhân viên, tôi lập cột Full Name vì dữ liệu đầu tên được chia làm First và Last name.
![image](https://user-images.githubusercontent.com/118591981/208192642-0e82c0d6-e668-4c26-bb34-a89985eb532c.png)

Trang Overview hiện thị nội dung tổng quan.
![image](https://user-images.githubusercontent.com/118591981/208193041-0b89d136-9748-4497-b2b7-e04868ac21d1.png)

Trang Demographics phân tích độ tuổi, giới tính, lương trung bình, sắc tộc (do công ty này của Mỹ nên có phân biệt màu da) và tình trạng hôn nhân.
![image](https://user-images.githubusercontent.com/118591981/208193381-6d1fc1f2-1725-445b-844b-31f8c505bcdf.png)

Trang Performance Tracking cho phép chọn 1 nhân viên và phân tích rating của người đó.
![image](https://user-images.githubusercontent.com/118591981/208193486-cd5e0ff4-97b3-45fb-9f4c-7d93baf7401b.png)

Trang Attrition phân tích tình trạng nghỉ việc.
![image](https://user-images.githubusercontent.com/118591981/208193580-ef93981a-4f83-464a-a522-18c677e1e969.png)
