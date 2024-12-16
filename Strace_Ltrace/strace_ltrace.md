


# 1. Giới thiệu về Strace và Ltrace
***Strace*** và ***ltrace*** là hai công cụ vô cùng hữu ích trong việc gỡ lỗi và hiểu rõ hơn về hành vi của các chương trình trên hệ thống ***Unix-like*** (như Linux). Chúng giúp ta theo dõi các gọi hệ thống và gọi thư viện mà một chương trình thực hiện, cung cấp những thông tin chi tiết về cách chương trình tương tác với hệ điều hành và các thư viện khác.

Trong bài viết tôi và các bạn sẽ cùng nhau tìm hiểu những nội dung sau: 
1. Strace và Ltrace được dùng để giải quyết các vấn đề gì?
2. Strace và Ltrace được sử dụng như thế nào?
3. Một số ví dụ sử dụng Strace và Ltrace.

Chúng ta cùng bắt đầu nào!

# 2. Strace 

## 2.1. Strace dùng để giải quyết vấn đề gì? 

Trong phần này, ta sẽ tìm hiểu về cách mà chương trình hoạt động:

1. Xác định nguyên nhân gây lỗi:
* Lỗi truy cập file: Khi một chương trình không thể đọc hoặc ghi vào một file, strace sẽ cho thấy chính xác gọi hệ thống nào bị lỗi (open, read, write, ...), quyền truy cập của quá trình, và thông báo lỗi từ hệ điều hành.
* Lỗi kết nối mạng: Nếu chương trình gặp vấn đề khi kết nối đến một máy chủ, strace sẽ hiển thị các gọi hệ thống liên quan đến mạng (connect, send, recv, ...), địa chỉ IP, cổng, và thông báo lỗi.
* Lỗi phân bổ bộ nhớ: Khi chương trình bị treo hoặc sập do hết bộ nhớ, strace sẽ giúp ta xác định các gọi hệ thống liên quan đến phân bổ bộ nhớ (malloc, free, ...), kích thước bộ nhớ được yêu cầu, và điểm xảy ra lỗi.
* Lỗi liên quan đến tín hiệu: Nếu chương trình bị ngắt bởi một tín hiệu (signal), strace sẽ cho biết tín hiệu nào được gửi đến, và chương trình phản ứng như thế nào.
2. Hiểu rõ hơn về hoạt động của chương trình:
* Theo dõi luồng thực thi: Bằng cách xem các gọi hệ thống theo thứ tự, ta có thể hiểu rõ hơn về cách một chương trình thực hiện các tác vụ của mình.
Phân tích hiệu năng: Strace giúp xác định các gọi hệ thống tốn nhiều thời gian nhất, từ đó tối ưu hóa hiệu năng của chương trình.
* Kiểm tra tính bảo mật: Strace có thể giúp phát hiện các lỗ hổng bảo mật tiềm ẩn, chẳng hạn như việc một chương trình truy cập vào các file hoặc tài nguyên hệ thống mà nó không được phép.
3. So sánh hành vi của các phiên bản chương trình:
* Bằng cách chạy strace trên các phiên bản khác nhau của cùng một chương trình, ta có thể so sánh sự khác biệt trong cách chúng tương tác với hệ điều hành và tìm ra nguyên nhân gây ra các lỗi mới.

## 2.2. Cách sử dụng Strace 

Dưới đây là những cú pháp cơ bản giúp bạn làm quen với ***Strace***: 

`strace [tùy chọn] lệnh [đối số]`

Trong đó: 
* tùy chọn: Các tùy chọn để điều khiển cách strace hoạt động.
* lệnh: Lệnh mà bạn muốn theo dõi.
* đối số: Các đối số truyền vào cho lệnh đó.

Các tùy chọn thường dùng của strace
- p pid: Theo dõi một tiến trình đang chạy (được xác định bởi PID).
- f: Theo dõi các tiến trình con được tạo ra.
- o file: Ghi output của strace vào file.
- t: In dấu thời gian cho mỗi gọi hệ thống.
- c: Hiển thị thống kê về các gọi hệ thống.
- e trace=event1,event2,...: Chỉ theo dõi các sự kiện cụ thể (ví dụ: open, read, write).



## 2.3. Ví dụ sử dụng strace

### 2.3.1. Bài toán

Ta có một đoạn code sau:

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    printf("Starting program\n");

    int num;
    FILE *file_ptr;
    
    file_ptr = fopen("my_output.txt", "w");

    if(file_ptr == NULL)
    {
        printf("Error opening file!");
        exit(1);
    }
    fprintf(file_ptr, "Hello world!\n");
    fclose(file_ptr);

    printf("Done! \n");
    return 0;
}
```
Chúng ta tiến hành biên dịch và sử dụng strace để thực thi chương trình write_file và hiển thị tất cả các system call mà chương trình này thực hiện ra màn hình: 

Cú pháp: 

`strace ./write_file`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/14/image_a597d2261d.png)


### 2.3.2. Strace syscalls Output

Sau khi biên dịch và tôi muốn kiểm tra đầu ra của chương trình và ghi vào một file cụ thể, ví dụ tôi muốn đầu ra chương trình của mình được ghi vào file out_write_file.txt thì ta thực hiện cú pháp như sau: 

`strace -o out_write_file ./strace`

Kết quả như sau: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_9a13900ab1.png)

### 2.3.3. Phân tích output của strace 

Nếu như bạn đang cố gắng phân tích output, hãy sử dụng cú pháp dưới đây để theo dõi output dễ dàng hơn: 

`vi out_write_file`

Kết quả in ra màn hình: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_c54d49813e.png)

Những ví dụ tôi khoanh tròn trong hình trên thực chất là những giá trị trả về của system calls,
ví dụ nếu tôi nhìn vào giá trị trả về của  **write(1, "Starting program\n", 17)** thì giá trị trả về của system call sẽ là số lượng các kí tự mà nó viết trong trường hợp này. Một ví dụ khác về giá trị trả về của **close(3)** khi thành công là bằng 0 hay giá trị trả về của **mmap(0x769e9dbb0000, 323584, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1b0000)**  Thì thực chất là các con trỏ trả về vị trí được ánh xạ trên bộ nhớ.

### 2.3.4. Theo dõi các system calls cụ thể 

Khi bạn muốn thu hẹp phạm vi theo dõi, chỉ tập trung vào những cuộc gọi hệ thống mà bạn quan tâm, giảm thiểu lượng thông tin xuất ra, giúp bạn dễ dàng tìm thấy những gì mình cần. Thì đây là cú pháp: 

`strace -e trace=sự_kiện1,sự_kiện2,... lệnh`

Ví dụ tôi chỉ muốn theo dõi sự kiện **close**:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_7952ce8ebc.png)

hoặc sự kiện **write**:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_87bc7833f3.png)

Khi tôi muốn loại trừ hoặc bỏ theo dõi một sự kiện nào đó, cụ thể là **mmap** thì ta có thể sử dụng cú pháp: 

`strace -e trace ='!mmap' ./write_file` 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_c11f342fad.png)

Bạn có thể thấy syscall **mmap** đã được loại bỏ.


### 2.3.5. Counting system calls

Cú pháp: 

`strace -c ./write_file`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_7dc101e5d3.png)


Chức năng chính:

* Đếm các cuộc gọi hệ thống: Đếm số lần mỗi cuộc gọi hệ thống được gọi bởi một tiến trình.
* Tính toán thời gian sử dụng: Ước tính thời gian tiêu tốn cho mỗi loại cuộc gọi hệ thống (ví dụ: các hoạt động trên file, các hoạt động mạng).
* Xác định các điểm nghẽn hiệu suất: Bằng cách phân tích số lần gọi và thời gian sử dụng, bạn có thể xác định chính xác các khu vực mà tiến trình có thể đang tiêu tốn quá nhiều thời gian.


### 2.3.6. Following forks

Tôi có một đoạn code ví dụ sau: 

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

void fork_example()
{
    //child procces because return value zero
    if(fork() == 0)
    {
        printf("Hello from Child! \n");
    }
    // parent product because return value non-zero
    else
    {
        printf("Hello from Parent! \n");
    }
}
int main()
    {
        fork_example();
        return 0;
    }
```
Tùy chọn -f của strace được sử dụng để theo dõi không chỉ tiến trình chính mà còn cả các tiến trình con được tạo ra bởi nó. Điều này rất hữu ích khi bạn muốn hiểu cách một chương trình tạo ra và quản lý các tiến trình con, đặc biệt trong các trường hợp phức tạp như các chương trình đa luồng hoặc các chương trình sử dụng các thư viện tạo tiến trình con.

Khi tôi muốn theo dõi tiến trình con trong đoạn code kia, ta sử dụng cú pháp:
`strace -o fork_out.txt -f ./write_file`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_bc4068ad12.png)

Lệnh này sẽ theo dõi cả tiến trình chính fork_write và tất cả các tiến trình con mà nó tạo ra. Output của strace sẽ hiển thị các cuộc gọi hệ thống của tất cả các tiến trình này, giúp bạn hiểu rõ hơn về cách chương trình hoạt động.

Tùy chọn -f của strace là một công cụ mạnh mẽ để hiểu rõ hơn về hành vi của các chương trình đa tiến trình. Bằng cách sử dụng nó, bạn có thể xác định các vấn đề tiềm ẩn, tối ưu hóa hiệu năng và cải thiện độ ổn định của chương trình.

# 3. Ltrace 

## 3.1. Ltrace dùng để giải quyết vấn đề gì? 

**ltrace** là một công cụ dòng lệnh được sử dụng để theo dõi các cuộc gọi thư viện của một chương trình. Nói cách khác, nó cho phép bạn "nghe lén" các cuộc gọi hàm đến các thư viện động mà chương trình sử dụng. Điều này rất hữu ích trong việc gỡ lỗi, hiểu cách một chương trình tương tác với các thư viện và tìm ra các vấn đề tiềm ẩn.

Khác với **strace** tập trung vào các cuộc gọi hệ thống, **ltrace** lại tập trung vào các cuộc gọi hàm cấp cao hơn, thường được cung cấp bởi các thư viện.


Tại sao sử dụng ltrace?
* Gỡ lỗi: Giúp xác định các lỗi liên quan đến việc gọi hàm thư viện.
* Phân tích hiệu năng: Giúp tìm ra những hàm nào đang tiêu tốn nhiều thời gian.
* Hiểu rõ hành vi của chương trình: Giúp hiểu cách chương trình tương tác với các thư viện.
* Kiểm tra bảo mật: Giúp tìm ra các lỗ hổng bảo mật liên quan đến việc sử dụng thư viện.


## 3.2. Cách sử dụng Ltrace

Cú pháp cơ bản: 

`ltrace [tùy chọn] lệnh [đối số]`

Các tùy chọn thường dùng
* -f: Theo dõi các tiến trình con.
* -o file: Ghi output vào file.
* -S: Hiển thị các chuỗi đối số và giá trị trả về.
* -s N: Cắt ngắn các chuỗi thành N ký tự.
* -l: Liệt kê các thư viện mà chương trình đang sử dụng.
* -D: In ra các định nghĩa của các hàm.

## 3.3. Ví dụ sử dụng ltrace

### 3.3.1. Chương trình 

```c
#include <stdio.h>

int main (){
    printf("Hello World!");
    return 0;
}

```

### 3.3.2. Biên dịch chương trình 

`gcc -o helloworld helloworld.c`

### 3.3.3. Gọi ltrace 

Cú pháp: 

`ltrace ./helloworld`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_b8e8c277e6.png)

### 3.3.4. In System calls

Cú pháp: 

`ltrace -S ./helloworld`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_3e7679a7e5.png)

### 3.3.5. Xác định thời gian tiêu thụ 

Cú pháp: 

`ltrace -c ./helloworld`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/15/image_8f70187e9e.png)


# So sánh Strace và Ltrace 

| Tính năng | Ltrace | Strace |
| -------- | -------- | -------- |
| Mục tiêu     | 	Các cuộc gọi thư viện    | Các cuộc gọi hệ thống     |
|   Cấp độ   | 	 Cấp cao hơn  |  	Cấp thấp hơn    |
|  Sử dụng phổ biến    | Gỡ lỗi các vấn đề liên quan đến thư viện	   |   Gỡ lỗi chung, phân tích hiệu năng   |

# Kết luận 

Sau khi đã cùng nhau tìm hiểu, strace và ltrace là những công cụ vô cùng hữu ích cho các lập trình viên và người quản trị hệ thống. Bằng cách hiểu rõ cách sử dụng hai công cụ này, bạn có thể giải quyết hiệu quả các vấn đề liên quan đến chương trình, cải thiện hiệu năng và bảo mật của hệ thống.

Chúc bạn đọc học tập tốt và hẹn gặp lại ở những bài viết sau!
