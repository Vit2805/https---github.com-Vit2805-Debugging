


# 1. Lời nói đầu 

Ở bài viết trước chúng ta đã được tìm hiểu về GDB (GNU Debugger) và cách sử dụng cơ bản của chúng, nếu bạn chưa đọc nó thì bạn có thể qua bài viết về GDB: https://devlinux.vn/post/gnu-1730765457267 để có thể hiểu về những nội dung tôi sắp đề cập trong bài viết này.

Bài viết này sẽ giới thiệu bạn đọc về những lỗi phổ biến trong lập trình C/C++ mà công cụ GDB có thể hỗ trợ bạn. 

# 2. Các lỗi thường gặp trong lập trình C/C++

* Lỗi cú pháp (Syntax errors)
* Lỗi ngữ nghĩa (Semantic errors)
* Lỗi logic (Logic errors)
* Lỗi tràn bộ nhớ (Memory leaks)

# 3. Các lệnh GDB thường sử dụng

* run: Chạy chương trình.
* break <điểm_ngắt> : Đặt điểm ngắt tại một dòng hoặc hàm cụ thể.
* continue: Tiếp tục chạy chương trình điểm ngắt đến điểm ngắt tiếp theo.
* next: Thực thi dòng lệnh tiếp theo.
* step: Bước vào bên trong hàm nếu dòng lệnh hiện tại gọi đến một hàm.
* print <biến>: In giá trị của một biến.
* backtrace: Hiển thị lịch sử gọi hàm.
* quit: Thoát khỏi GDB.


# 4. Ví dụ minh họa cho các trường hợp lỗi
## 4.1. Lỗi cú pháp (syntas errors)

Mặc dù GDB không trực tiếp phát hiện lỗi cú pháp, nhưng nó có thể giúp bạn kiểm tra giá trị các biến và dòng điều khiển để xem lỗi có phải do một biểu thức sai cú pháp hay không.

Ta có một đoạn code sau: 

```c
#include <stdio.h>

int main(){
    int a = 5, b = 6;
    float c = a + b;
    printf("Gia tri cua c la: %d\n", c);
    return 0;
}
```

Biên dịch:

`gcc -g syntas_erros.c -o syntas_erros`

Khởi chạy GDB: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_8d8f1bd453.png)

Chương trình được khởi tạo hai biến a và b với kiểu dữ liệu int, tuy nhiên biến c được khai báo với kiểu float gán bằng a+b. Đây là lỗi liên quan đến việc sử dụng sai kiểu dữ liệu. Sau khi chạy chương trình biến c được in ra màn hình giá trị -7704 (giá trị rác). Từ đó ta có thể xác định được lỗi của chương trình và sửa nó.

Code sau khi sửa:
```c
#include <stdio.h>

int main(){
    int a = 5, b = 6;
    int c = a + b;
    printf("Gia tri cua c la: %d\n", c);
    return 0;
}
```

Kết quả sau khi sửa:
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_8225aa637c.png)

## 4.2. Lỗi ngữ nghĩa trong C(Semantic errors)

Đây là một loại lỗi khá "trớ trêu" trong lập trình C. Chúng không vi phạm bất kì quy tắc cú pháp nào của ngôn ngữ nhưng lại khiến chưuong trình chạy không đúng như ý muốn của người lập trình. Thường thì các lỗi này liên quan đến logic chương trình, cách sử dụng các hàm hoặc hiểu sai về vấn đề mà chương trình cần giải quyết.

Dưới đây là một đoạn code lỗi trong vòng lặp:

```c
//Bài toán tính tổng từ 1 đến n
#include <stdio.h>
#include <stdint.h>

int main(){
    //Khai báo n và khởi tạo tổng sum = 0
    int n = 5;
    int sum =0;
    // Tính tổng từ 1 tới n
    for(int i = 0; i <= n; i++){
        sum += i;
    }
    //In ra giá trị của tổng
    printf("sum = %d\n", sum);
    return 0;
}
```
Kết quả của đoạn code: 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_5939ab3406.png)

Lỗi ở đây là điều kiện dừng của vòng lặp. Với n = 5, vòng lặp sẽ chạy từ i = 0 tới i =  5, dẫn đến kết quả tính toán sai. Điều kiện dừng đúng phải là i < n.

Ta sửa lại đoạn code như sau: 

```c
//Bài toán tính tổng từ 1 đến n
#include <stdio.h>
#include <stdint.h>

int main(){
    //Khai báo n và khởi tạo tổng sum = 0
    int n = 5;
    int sum =0;
    // Tính tổng từ 1 tới n
    for(int i = 0; i < n; i++){
        sum += i;
    }
    //In ra giá trị của tổng
    printf("sum = %d\n", sum);
    return 0;
}
```
Để theo dõi giá trị của tổng sum, ta sử dụng breakpoint ở dòng 15 và sử dụng lệnh r (run) để chạy đến breakpoint.

Cú pháp:
`break 15`

Ta được kết quả: 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_889d1c01e7.png)

Chương trình đã dừng lại tại dòng số 15, ta quan sát kết quả của biến sum = 10, đúng với yêu cầu của đề bài. 

## 4.3. Lỗi logic trong lập trình C (logic erros)

Lỗi logic là một loại lỗi khá tinh vi trong lập trình, khi chương trình chạy đúng cú pháp nhưng lại đưa ra kết quả sai so cới yêu cầu của bài toán. Nguyên nhân thường đến từ việc lập trình viên hiểu sai về vấn đề cần giải quyết, thiết kế thuật toán không chính xác, hoặc có những sai sót trong quá trình triển khai thuật toán đó.

Dưới đây là một đoạn code về lỗi logic 

```c
// Tìm số lớn nhất trong 3 số
#include <stdio.h>

int main() {
    int a = 3, b = 6, c = 10, max;

    // Lỗi logic: Chỉ so sánh a với b
    if (a > b)
        max = a;
    else
        max = b;

    printf("So lon nhat la: %d\n", max);
    return 0;
}

```
Biên dịch và chạy: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_fd5fd0ea3c.png)

Chương trình đang mắc lỗi khi chỉ so sánh giá trị của biến a và b, dẫn đến kết quả của bài toán sai.

Ta sửa lại chúng như sau:

```c
// Tìm số lớn nhất trong 3 số
#include <stdio.h>

int main() {
    int a = 3, b = 6, c = 10, max;

    // Lỗi logic: Chỉ so sánh a với b
    if (a > b && a > c)
        max = a;
    else if (b > c)
        max = b;
    else 
        max = c;
    printf("So lon nhat la: %d\n", max);
    return 0;
}
```
Khởi chạy GDB, đặt breakpoint ở dòng số 15 và quan sát giá trị max: 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_429cb1c4fe.png)

Ta được kết quả max = 10 đúng với logic của bài toán.

## 4.4. Lỗi tràn bộ nhớ (memory leaks)

Lỗi tràn bộ nhớ là một trong những lỗ hổng bảo mật nghiêm trọng trong lập trình, đặc biệt là trong C. Nó xảy ra khi một chương trình cố gắng ghi dữ liệu vào một vùng nhớ có kích thước cố định mà không kiểm tra xem có đủ không gian hay không. Điều này dẫn đến việc dữ liệu bị ghi đè lên các vùng nhớ liền kề, gây ra những hậu quả nghiêm trọng như:
*  Chương trình crash: Chương trình bị dừng đột ngột.
*  Hành vi không móng muốn: Chương trình hoạt động không đúng.
*  Mở cửa hậu: Kẻ tấn công có thể lợi dụng lỗ hổng này để thực thu mã độc hại.

Nguyên nhân gây ra lỗi tràn bộ nhớ: 
*  Không kiểm tra kích thước dữ liệu đầu vào: Khi nhận dữ liệu từ người dùng hoặc một nguồn khác mà không kiểm tra xem kích thước có vượt quá giới hạn cho phép của bộ nhớ đệm hay không.
*  Sử dụng các hàm không an toàn: Một số hàm như gets() có thể gây ra lỗi tràn bộ nhớ nếu không sử dụng cẩn thận.
*  Lỗi tràn số nguyên: Khi một phép tính số học vượt quá giới hạn của kiểu dữ liệu, có thể dẫn đến các hành vi không mong muốn.

### 4.4.1. Ví dụ 1

```c
#include <stdio.h>
#include <string.h>

#define MAX_LENGTH 10

int main() {
    char buffer[MAX_LENGTH];

    printf("Nhap mot chuoi: ");
    fgets(buffer, MAX_LENGTH, stdin);

    printf("Chuoi vua nhap la: %s\n", buffer);

    return 0;
}
```
Để gỡ lỗi đoạn code này bằng GDB, ta thực hiện các bước sau:
1. Biên dịch: gcc -g memory_leaks.c -o memory_leaks
2. Khởi động GDB: gdb memory_leaks
3. Đặt điểm ngắt: break 11
4. Chạy chương trình: run
5. Nhập một chuỗi quá dài: Nhập một chuỗi dài vượt quá MAX_LEGTH
6. Kiểm tra giá trị biến: Sử dụng lệnh print buffer để xem nội dung của buffer
7. Kiểm tra stack: Sử dụng lệnh backtrace để xem lại stack frame 

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_03d0d077b0.png)

Ta nhập một chuỗi "HelloWorld!" đây là giá trị mong đợi, tuy nhiên MAX_LEGTH = 10 nhỏ hơn số kí tự mong đợi. Kết quả là ta được giá trị hiện tại "HelloWorl". Sự khác nhau giữa giá trị hiện tại với giá trị mong đợi rất có thể đã xảy ra lỗi memory leaks.

Để sửa lỗi này, ta cần nhập một chuỗi ngắn hơn: 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_962cd96b9e.png)

Ta thấy chương trình đã in ra đúng với giá trị mong đợi!

### 4.4.2. Ví dụ 2

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int size = 10;
    int *ptr = (int*)malloc(size * sizeof(int)); // Cấp phát bộ nhớ cho mảng 10 số nguyên

    // Sử dụng mảng
    for (int i = 0; i < 10; i++) {
        ptr[i] = i;
        printf("%d \n", ptr[i]);
    }

    printf("Kich thuoc cua vung nho duoc cap phat = %zu byte\n", size * sizeof(int));

    return 0; // Kết thúc chương trình mà không giải phóng
}
```
Trong ví dụ này, mảng ptr được cấp phát động bằng malloc nhưng lại không được giải phóng bằng hàm free trước khi kết thúc chương trình. Điều này dẫn đến việc rò rỉ 40 byte bộ nhớ.

Sửa lại đoạn code bằng cách thêm hàm free(ptr):
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int size = 10;
    int *ptr = (int*)malloc(size * sizeof(int)); // Cấp phát bộ nhớ cho mảng 10 số nguyên

    // Sử dụng mảng
    for (int i = 0; i < 10; i++) {
        ptr[i] = i;
        printf("%d \n", ptr[i]);
    }

    printf("Kich thuoc cua vung nho duoc cap phat = %zu byte\n", size * sizeof(int));

    free(ptr);
    return 0; // Kết thúc chương trình đã giải phóng 
}
```
Kiểm tra vùng nhớ bằng GDB: 
*  Khi chương trình dừng lại ở breakpoint, ta có thể sử dụng lệnh print ptr để xem giá trị của con trỏ ptr.
*  Sử dụng lệnh x/10d ptr để xem 10 số nguyên bắt đầu từ địa chỉ mà ptr đang trỏ tới.

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_a2ec58d29e.png)

Sử dụng lệnh continue để chạy hàm free() và dừng tại breakpoint dòng 18:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/13/image_9b9967a9bf.png)

Ta có thể thấy sau khi giải phóng bộ nhớ thì mảng ptr không được cấp phát bộ nhớ để lưu giữ giá trị nữa, thay vào đó là những giá trị rác.

# 5. Kết luận 

GDB là một công cụ vô cùng hữu ích trong việc gỡ lỗi các chương trình C. Việc thành thạo GDB sẽ giúp bạn tiết kiệm thời gian và tăng hiệu quả trong quá trình phát triển phần mềm. Chúc bạn đọc học tập tốt!