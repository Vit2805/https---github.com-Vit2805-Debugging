# Hướng dẫn gỡ lỗi bằng GDB
## 1. Lời nói đầu 
Mục đích của bài viết này là để cung cấp cho bạn đọc cái nhìn tổng quan về kiến thức GDB. Bài viết này sẽ hướng dẫn gỡ lỗi bằng GDB.

GDB là một công cụ sửa lỗi rất dễ dàng sử dụng và mạnh mẽ trong Linux. GDB có thể gỡ lỗi C, C++, Go, Java, Object-c, PHP và các ngôn ngữ khác. Đối với lập trình viên C/C++ làm việc trên Linux thì GDB là một công cụ không thể thiếu.

## 2. Giới thiệu về GDB 

Mặc dù là một công cụ sửa lỗi chế độ dòng lệnh (Command line mode) nhưng khả năng của nó mạnh mẽ tới mức bạn khó có thể tưởng tượng được, GDB cho phép người dùng quan sát cấu trúc bên trong và mức sử dụng bộ nhớ của chương trình khi đang chạy.

Nhìn chung, GDB sẽ giải quyết giúp bạn bốn vấn đề chính sau:
>  1. Khởi động và chạy chương trình cần gỡ lỗi theo cách tùy chỉnh.
>  2. Bạn có thể đặt điểm dừng (breakpoint) bằng cách chỉ định vị trí và biểu thức điều kiện 
>  3. Theo dõi giá trị khi chương trình bị tạm dừng.
>  4. Tự động thay đổi môi trường thực thi của chương trình.

## 3. Các thao tác lệnh cơ bản 

Có rất nhiều lệnh trong GDB, nhưng chúng ta chỉ cần nắm vững khoảng mười lệnh trong số đó là có thể hoàn thành công việc gỡ lỗi chương trình cơ bản hàng ngày.
| Tập tin <tên tập tin>  | Tải tệp chương trình thực thi đang được gỡ lỗi |
| :---: | :---: |
| run | Khởi động lại tập tin đang chạy |
| start | Thực hiện một bước, chạy chương trình, dừng ở câu lệnh thực thi đầu tiên |
| list | Xem mã gốc, viết tắt |
| set | Đặt giá trị của một biến |
| next | Gỡ lỗi từng bước (từng quy trình, thực hiện chức năng trực tiếp), viết tắt là n | 
| step | Gỡ lỗi từng bước (từng câu lệnh, chuyển sang thực thi nội bộ của một hàm tùy chỉnh), viết tắt là s |
| backtrace | Tìm kiếm Frame và mối quan hệ thứ bậc của funtion call, viết tắt là bt |
| frame | Stack frame viết tắt cho chức năng chuyển đổi, viết tắt là f |
| info | Kiểm tra giá trị của biến cục bộ bên trong hàm, viết tắt là i | 
| finish | Kết thúc hàm hiện tại và quay lại điểm gọi hàm |
| continue | Tiếp tục chạy, viết tắt là c |
| print | In ra giá trị và địa chỉ, viết tắt là p | 
| quit | Thoát GDB, viết tắt là q | 

Lệnh gdb có nhiều lệnh nội bộ. Nhập "help" tại dấu nhắc lệnh gdb "(gdb)" để xem tất cả cách lệnh nội bộ và hướng dẫn sử dụng của chúng.
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/ubuntu_Running_Oracle_VM_Virtual_Box_11_5_2024_9_38_04_AM_748b789b4d.png)

## 4. Xác định xem tệp có chứa thông tin gỡ lỗi hay không 

Để gỡ lỗi chương trình C/C++, trước tiên bạn hãy sử dụng gdb để gỡ lỗi chương tình trong quá trình biên dịch. Khi sử dụng gcc để dịch mã nguồn, bạn phải thêm tham số "-g". Giữ nguyên thông tin gỡ lỗi, nếu không bạn không thể sử dụng GDB để gỡ lỗi.

Có một trường hợp là có một tệp nhị phân được biên dịch và bạn không chắc liệu tham số -g và gỡ lỗi GDB hay không. Trong trường hợp này, bạn có thể sử dụng lệnh sau để xác minh:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/ubuntu_Running_Oracle_VM_Virtual_Box_11_5_2024_1_21_58_PM_9f0584d49b.png)

Nếu không có thông tin gỡ lỗi, thông tin này sẽ xuất hiện: 

 `(No debugging symbols found in **main**)`

/home/vit/ThucHanh/code/main là đường dẫn của chương trình 

Nếu chức năng gỡ lỗi được bật, lời nhắc sau sẽ xuất hiện:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/ubuntu_Running_Oracle_VM_Virtual_Box_11_5_2024_2_15_03_PM_da3806a2b0.png)

`Reading symbols from **main**...`


Cho biết có thể gỡ lỗi GDB.

Bạn cũng có thể sử dụng readelf để kiểm tra xem tệp thực thi có khả năng sửa lỗi hay không.

`readelf -S main|grep debug`


![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/ubuntu_Running_Oracle_VM_Virtual_Box_11_5_2024_2_23_36_PM_d2dbf0a298.png)

Nếu có debug tức là có chức năng debug. Lưu ý rằng nếu không có khả năng gỡ lỗi thì không thể gỡ lỗi được.

Bắc đầu vào vấn đề, GDB bắt đầu gỡ lỗi.

## 5. Khởi động và chạy chương trình không có tham số ở chế độ gỡ lỗi

Dưới đây là một ví dụ về gỡ lỗi GDB trong Linux ngôn ngữ C: 
main.c

```c
#include <stdio.h>

void print(int i){
	printf("Xin chao, Vit %d! \n", i);
}
int main(){
	for(int i = 1; i < 3; i++){
		print(i);
	}
}
```
Biên dịch: 
```
gcc -g main.c -o main 
```

Lệnh "gdb" sau khởi động GDB và trước tiên sẽ hiển thị mô tả GDB:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_f9c690edeb.png)


Dòng cuối cùng "(gdb)" ở trên là đường dẫn lệnh nội bộ GDB, chờ người dùng nhập lệnh GDB.

Phần dưới đây sử dụng lệnh "file" để tải chương trình chương trình chính đã được gỡ lỗi (chính ở đây là đầu tpệp thực thi bằng quá trình biên dịch gcc trước đó):

Nếu dòng cuối cùng nhắc *Reading symbols from main...*, điều đó có nghĩa là nó đã được tải thành công.

Dưới đây sử dụng lệnh "r" để thực thi (run) tệp đã gỡ lỗi. Vì không có điểm dừng nào được đặt nên nó sẽ thực thi trực tiếp tới cuối chương trình:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_a68cc5752a.png)

## 6. Gỡ lỗi và khởi động chương trình với các tham số

Giả sử có chương trình sau đây cần được bắt đầu với các tham số: 

```c 
#include <stdio.h>

int main(int argc, char const *argv[]){
	if(1 >= argc){
		printf("usage: hello name\n ");
		return 0;
	}
	printf("hello, Vit %s\n", argv[1]);
	return 0;
}
```

Biên dịch:

```
gcc -g main.c -o main 
```

Làm thế nào để bắt đầu gỡ lỗi trong tình huống này? Chỉ cần mang theo thông số khi bạn cần r.

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_86ae195db0.png)

## 7. Gỡ lỗi tập tin lỗi

Core Dump: Core có nghĩa là bộ nhớ và Dump có nghĩa là vứt nó đi hoặc xếp chồng lên nhau (lỗi phân đoạn). Khi phát triển và sử dụng các chương trình Unix, đôi khi chương trình bị hỏng một cách khó hiểu mà không có bất kì lời nhắc nào (đôi khi nó nhắc nhở kết xuất lõi). Tại thời điểm này, bạn có thể kiểm tra xem một tệp có dạng số core.process có được tạo hay không. Hệ thống sẽ loại bỏ nội dung bộ nhớ khi chương trình không hoạt động. Nó có thể được sử dụng làm tại liệu tham khảo để gỡ lỗi chương trình và có thể giúp chúng tôi xác định các vấn đề trong chương trình. Vậy làm cách nào để bạn có thể tạo file Core?

## 8. Tạo phương thức Core

Để tạo các điều kiện cho coredump, trước tiên bạn cần xác nhận ulimit -c của phiên hiện tại. Nếu nó bằng 0, coredump tương ứng sẽ không được tạo và cần được sửa đổi và thiết lập.

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_e47b391789.png)

Ngay cả khi lõi chương trình bị loại bỏ, sẽ không còn tệp lõi nào. Chúng ta cần cho phép tạo các tệp lõi và đặt kích thước lõi thành không giới hạn:

```
ulimit -c unlimited
```

## 9. Thay đổi đường dẫn tạo kết xuất lõi 

Bởi vì kết xuất lõi sẽ được tạo trong thư mục làm việc của chương trình theo mặc định, nhưng một số chương trình có thể chuyển đổi thư muc, dẫn đến đường tạo kết xuất lõi không đều.

Vì vậy tốt nhất bạn nên tự tạo một thư mục để lưu trữ file lõi đã tạo.

Tôi đã tạo thư mục/data/coredump, thư mục coredump trong data thư mục gốc.

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_68a1d1f320.png)

Gọi lệnh sau: 
```
echo /data/coredump/core.%e.%p> /proc/sys/kernel/core_pattern
```
Đường dẫn tạo tệp lõi sẽ được thay đổi và tự động được đặt trong thư mục /data/coredump.%e đại diện tên chương trình, %p đại điện cho id tiến trình

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_4b459895cc.png)

Mã kiểm tra: 

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
	int i = 0; 
	scanf("%d", i);
	printf("hello, Vit %d\n", i);
	return 0;
}

```

Biên dịch và chạy: 


![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_0233e17d50.png)

Sau khi chạy, kết quả hiển thị lỗi Segmentation fault. Chương trình gặp sự cố khi quét bên trong chức năng chính. "&" phải được thêm vào trước i.

Lúc này bạn hãy vào thư mục /data/coredump để xem lõi đã tạo

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_0c2d440715.png)

Sau đó dùng debug core, lệnh là gdb core.test.25733 thì hiển thị như sau 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_18e400b082.png)

Tại thời điểm này, bạn có thể gõ lệnh backtrace(bt) để xem stack frame và mối quan hệ phân cấp của function call.

Bạn có thể gỡ lỗi nó bằng lệnh sau: 
``` After the gdb executable program exe enters the gdb environment, type the core-file core name and type the command bt to view the accurate information.```

Chương trình thực thi gdb 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_2d5c0468e8.png)

Sau khi vào môi trường gdb

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/5/image_f0b5376175.png)

Nhập lệnh bt

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_66c1ed62bb.png)

Bạn có thể thấy rằng stack frame gần đây nhất lưu trữ lệnh gọi đến thao tác IO và bạn có thể thấy rằng có lỗi trên dòng 5 của hàm chính.

Cho đến nay, đây là phương pháp tạo và gỡ lỗi cấu hình tệp cốt lõi.

## 10. Tóm tắt

Tại thời điểm này, phương pháp gỡ lỗi khưởi động GDB của chúng ta đã hoàn tất và các phương pháp gỡ lỗi và tạo cấu hình tệp lõi đã hoàn tất. Tiếp theo, chúng ta sẽ nói về cài đặt điểm dừng, gỡ lỗi từng bước, v.v. 

## 11. Cài đặt Breakpoint và xem source code

### 11.1. Lời nói đầu 

Trong bài viết trước về gỡ lỗi khởi động GDB, chúng ta đã nói về nhiều cách khác nhau để gỡ lỗi khởi động GDB. Trong phát triển phần mềm môi trường Linux, GDB là công cụ gỡ lỗi chính, được sử dụng để gỡ lỗi chương trình C/C++. Bài viết này chủ yếu nói về cài đặt điểm dừng (breakpoint) và xem mã nguồn (source code)  bằng GDB.

### 11.2. Tại sao chúng ta lại cần đặt breakpoint?

Khi ta muốn xem nội dung biến, stack status, v.v., chúng ta có thể chị định các breakpoint. Việc thực thi chương trình sẽ bị tạm dừng khi đạt đến breakpoint. Lệnh break được sử dụng để đặt breakpoint và tên viết tắt của nó là b. Sau khi đặt nó, chúng ta có thể theo dõi việc thực hiện chương trình gần điểm dừng một cách chi tiết hơn.

### 11.3. Đặt breakpoint theo số dòng

Định dạng: 

 `break [line number]`

số dòng ngắt, điểm ngắt được đặt ở đầu dòng, lưu ý: **dòng mã này chưa được thực thi**

Nếu chương trình của bạn được viết bằng C/C++ thì bạn có thể đặt điểm dừng ở dạng "file name: linenumber". Ví dụ như sau:

```c
#include <stdio.h>

void judge_sd(int num
{
	if ((num & 1) == 0 )
	{
		printf("%d is even\n", num);
		return;
	}
	else
	{
		printf("%d is odd\n", num);
		return;
	}
}
int main (int argc, char const *argv[])
{
	judge_sd(0);
	judge_sd(1);
	judge_sd(4);
	
	return 0;
}
```
Biên dịch: 

``` 
gcc -g test.c -o test 

```

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_90c158ffef.png)


Kiểm tra gdb:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_12deb28e85.png)

break tên file: số dòng, phù hợp với trường hợp có nhiều file nguồn.

Trong ví dụ (gdb) b test.c:19, một breakpoint được đặt. Vị trí của điểm ngắt là dòng 19 của tệp test.c. Khi sử dụng lệnh r để thực thi tập lệnh, nó sẽ tạm dừng khi đến dòng 19. Lưu ý là dòng mã này không được thực thi.

### 11.4. Đặt breakpoint thông qua các hàm 

Định dang: 

`break[function name]`

tên hàm break, điểm ngắt được đặt ở dấu hàm và dòng chứa điểm dừng không được thực thi:

Điểm dừng cũng có thể được đặt tại các chức năng:

`b judge_sd`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_783243f645.png)

### 11.5. Đặt điểm dừng có điều kiện 

Nếu bạn đặt điểm dừng theo phương pháp trên, việc thực thi sẽ tạm dừng mỗi khi đạt đến vị trí điểm dừng. Đôi khí nó có thể gây khó chịu. Chúng tôi chỉ muốn tạm dừng trong các điều kiện cụ thị. Đây là lúc việc thiết lập điểm dừng dựa trên các điều kiện trở nên hữu ích.  Hình thức đặt điểm dừng có điều kiện là thêm điều kiện if sau hình thức đặt điểm dừng cơ bản. Ví dụ như sau:

`break test.c: 7 if num>0`


![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_99fd91f890.png)

khi num > 0 thì chương trình sẽ dừng lại ở dòng 7.

### 11.6. Xem breakpoint

Cú pháp: 

`info breakpoint`

Bạn có thể sử dụng điểm dừng thông tin để xem trạng thái điểm dừng. Chứa thông tin như điểm dừng nào đã được đặt và số lần điểm dừng được đặt. Ví dụ như sau: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_eab763117c.png)

Nó sẽ liệt kê tất cả những điểm dừng đã đặt, mỗi điểm dừng có một nhãn đâị diện cho điểm dừng.

### 11.7. Xóa breakpoint

Cú pháp: 

`delete breakpoint`

Chúng ta có thể xóa các điểm dừng vô dụng. Định dạng của lệnh đã xóa là xóa số breakpoint. Cột num trong kết quả hiện thị bởi lệnh `info breakpoint` thông tin là số.  Một ví dụ về việc xóa breakpoint như sau:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_13d5b24e43.png)

### 11.8. Xem source code

Sau khi thiết lập điểm dừng, chương trình sẽ tạm dừng khi đạt đến điểm dừng. Khi bị tạm dừng, chúng ta có thể xem mã gần điểm dừng. Lệnh con để xem mã là list, viết tắt là l.

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_07aa2eee76.png)

### 11.9. Chỉ định số dòng để xem code 

Cú pháp: 

`list dòng đầu tiên, dòng cuối cùng`

Ví dụ: liệt kê mã nguồn từ dòng 12 đến dòng 15:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_336cc5962f.png)

### 11.10. Liệt kê mã nguồn của tệp được chỉ định

Khi thực hiện lệnh list trước đó, mã nguồn của test.c được liệt kê theo mặc định. Nếu bạn muốn xem mã nguồn của tệp được chỉ định thì sao? Đây là cách cho bạn:

`list [tên tệp + số dòng hoặc tên hàm]`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_f7e266267c.png)

### 11.11. Tóm tắt 

Bài viết trên giới thiệu về cài đặt điểm dừng và xem mã nguồn trong quá trình gỡ lỗi GDB. Cài đặt điểm dừng có thể giúp chúng ta dễ dàng quan sát các biến, ngăn xếp và thông tin khác sau này cũng như chuẩn bị cho việc định vị và gỡ lỗi tiếp theo. Khi xem mã nguồn, bạn có thể xem mã liên quan bằng cách chỉ định số dòng hoặc tên phương thức. 

## 12. Gỡ lỗi từng bước (step by step)

### 12.1. Lời nói đầu

Trong hai bài viết trước, chúng tã đã bắt đầu gỡ lỗi GDB, đặt điểm dừng gỡ lỗi GDB và xem mã nguồn. Chúng ta đã hiểu cách gỡ lỗi khởi động cơ bản của GDB, đặt breakpoint. xem source, v.v, Nếu bạn chưa biết những nội dung này thì nên xem lại nội dung trước đó.

Sau khi hiểu mã gần breakpoint, bạn có thể sử dụng thực thi từng bước để thực thi từng câu lệnh một. Phần sau đây sẽ giới thiệu gỡ lỗi từng bước và cài đặt các biến.

### 12.2. Gỡ lỗi từng bước

Đây chính xác là debug code. Như thường lệ, hãy bắt đầu với những dòng code: 

```c
#include <stdio.h>

void judge_sd(int num
{
	if ((num & 1) == 0 )
	{
		printf("%d is even\n", num);
		return;
	}
	else
	{
		printf("%d is odd\n", num);
		return;
	}
}
int main (int argc, char const *argv[])
{
	judge_sd(0);
	judge_sd(1);
	judge_sd(4);
	
	return 0;
}
```
Biên dịch: 

``` 
gcc -g test.c -o test 

```
Chức năng của chương trình tương đối đơn giản nên tôi sẽ không giải thích ở đây. Sau khi hiểu mã gần điểm ngắt, bạn có thể sử dụng thực thi từng bước để thực hiện thực thi từng câu lệnh một. Bạn có thể xem kết quả thực hiện bất cứ lúc nào. Có hai lệnh để thực hiện từng bước, đó là `step` và `next`. Chúng ta có thể đặt nhiều breakpoint hoặc các breakpoint có thể được đặt trong vòng lặp. Trong trường hợp này, chúng ta có thể sử dụng lệnh `continue`. Sự khác biệt giữa ba lệnh này là:

```
1.  next (viết tắt là n) dùng để thực thi câu lệnh tiếp theo sau khi chương trình bị gián đoạn
2.  step (viết tắt là s) câu lệnh có thể bước một bước bên trong hàm tương ứng
3.  continue (viết tắt là c) nó sẽ tiếp tục thực hiện chương trình cho đến khi gặp lại breakpoint

```
### 12.3. Single step into step 

Sử dụng list (viết tắt là l) để liệt kê mã nguồn, ví dụ:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_fa65a8a511.png)

Bắt đầu gỡ lỗi trước:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_c9b1eafbd7.png)

Thực hiện next:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_0f2a6240e4.png)

Lệnh next dùng để tiếp tục thực hiện câu lệnh tiếp theo sau khi chương trình bị gián đoạn. 

### 12.4. Lệnh continue 

Chúng ta có thể gặp nhiều điểm dừng tỏng một vòng lặp. Lúc này, chúng ta nên làm gì nếu muốn bỏ qua điểm dừng này hoặc thậm chí bỏ qua nhiều điểm dừng và tiếp tục thực hiện? Bạn có thể sử dụng lệnh `continue`. Chức năng của nó là tiếp tục thực hiện từ điểm tạm dừng. Code mẫu như sau: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_2e30ca2ca2.png)

### 12.5. Lệnh skip 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/6/image_27073dd761.png)

Theo hình trên bạn có thể thấy sau khi sử dụng skip thì hàm Judge_sd sẽ không được nhập vào. Ưu điểm của Skip có thể bỏ qua một số chức năng hoặc tập tin mà bạn không muốn chú ý đến.

Nếu bạn muốn xóa bỏ qua, hãy sử dụng delete[num].

## 13. Tóm tắt 

Qua những ví dụ ở trên, tôi tin rằng độc giả đã hiểu cơ bản về việc gỡ lỗi chương trình C/C++ thông qua GDB. Nếu bạn muốn có thêm kỹ năng debug, vui lòng theo dõi thêm những bài viết sắp tới của chúng mình.
