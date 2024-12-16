# 1. Giới thiệu 

Phần 2 của quản lý dữ liệu, chúng ta sẽ tiếp tục tìm hiểu về các cách thức quản lý bộ nhớ!

# 2. Zone

Nhân Linux chia RAM vật lý thành một số vùng bộ nhớ khác nhau: zone

Những vùng bộ nhớ (zone) nào phụ thuộc vào việc máy của bạn là 32-bit hay 64-bit và mức độ phức tạp của nó

Dưới đây là các zones:
1. DMA: bộ nhớ thấp 16 MB.
* Tại thời điểm này, nó tồn tại vì lý do lịch sử.
* đã có phần cứng chỉ có thể thực hiện DMA vào vùng bộ nhớ vật lý này.

2. DMA32: 
* chỉ tồn tại trong Linux 64 bit.
* là bộ nhớ thấp 4 GB, có thể ít hoặc nhiều hơn.
* t tồn tại vì quá trình chuyển đổi sang máy 64 bit có bộ nhớ lớn đã tạo ra một lớp phần cứng chỉ có thể thực hiện DMA vào bộ nhớ thấp 4 GB.

3. Bình thường: khác nhau trên máy 32 bit và 64 bit.
* Trên máy 64 bit, tất cả đều là RAM từ 4 GB trở lên
* Trên máy 32 bit, tất cả đều là RAM từ 16 MB đến 896 MB vì lý do phức tạp và có phần lịch sử

Lưu ý rằng điều này ngụ ý rằng máy có nhân 64 bit có thể có lượng bộ nhớ Bình thường rất nhỏ trừ khi chúng có RAM lớn hơn đáng kể so với 4 GB.
Ví dụ, một máy 2 GB chạy kernel 64 bit sẽ không có bộ nhớ Normal nào cả trong khi một máy 4 GB sẽ chỉ có một lượng nhỏ bộ nhớ Normal.

4. HighMem:
* chỉ tồn tại trên Linux 32 bit.
* là tất cả RAM trên 896 MB, bao gồm RAM trên 4 GB trên các máy đủ lớn.

Trong mỗi vùng, Linux sử dụng bộ phân bổ buddy-system để phân bổ và giải phóng bộ nhớ vật lý.

# 3. Buddy System allocator

Bộ nhớ được chia thành các khối trang lớn, trong đó mỗi khối là lũy thừa của hai số trang (2^thứ tự).
Tất cả các trang trống được chia thành 11 danh sách (MAX_ORDER), mỗi danh sách chứa danh sách các trang 2^thứ tự.

Khi yêu cầu phân bổ được thực hiện cho một kích thước cụ thể, hệ thống buddy sẽ tìm kiếm trong danh sách thích hợp cho một khối trống và trả về địa chỉ của khối đó, nếu có.

Tuy nhiên, nếu không tìm thấy khối trống, hệ thống sẽ di chuyển để kiểm tra trong danh sách bậc cao tiếp theo để tìm khối lớn hơn, nếu có, hệ thống sẽ chia khối bậc cao thành các phần bằng nhau được gọi là buddy, trả về một khối cho bộ phân bổ và xếp hàng khối thứ hai vào danh sách bậc thấp hơn.

Khi cả hai khối buddy đều trống vào một thời điểm nào đó trong tương lai, chúng sẽ được hợp nhất để tạo thành một khối lớn hơn

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/27/image_054188b5e3.png)

# 4. Virtual Kernel Memory Layout

Bạn có thể theo dõi sự bố trí bộ nhớ ảo của nhân ở lệnh 
`dmesg | grep -A 10 'virtual kernel memory layout'`
hoặc 
`Documentation/x86/x86_64/mm.rst`

# 5. Một số hàm khác 

## 5.1. Ksize 

kmalloc có thể làm tròn nội bộ các phân bổ và trả về nhiều bộ nhớ hơn yêu cầu.

ksize() có thể được sử dụng để xác định lượng bộ nhớ thực tế được phân bổ.

Người gọi có thể sử dụng bộ nhớ bổ sung này, mặc dù ban đầu đã chỉ định một lượng bộ nhớ nhỏ hơn với lệnh gọi kmalloc.

Hàm này không thường xuyên cần thiết; người gọi đến kmalloc() thường biết những gì họ đã phân bổ. Tuy nhiên, nó có thể hữu ích trong các tình huống mà một hàm cần biết kích thước của một đối tượng và không có thông tin đó trong tay.

## 5.2. Kzalloc

kzalloc hoạt động giống như kmalloc, nhưng cũng xóa bộ nhớ về 0.
`void *kzalloc(size_t size, gfp_t flags);`
## 5.3. krealloc 

Bộ nhớ được phân bổ bởi kmalloc() có thể được thay đổi kích thước bằng cách:

`void *krealloc(const void *p, size_t new_size, gfp_t flags);`

# 6. vmalloc 

Bộ nhớ được trả về bởi vmalloc chỉ liền kề trong bộ nhớ ảo và không liền kề trong bộ nhớ vật lý

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/28/image_19c5149b6c.png)

Bộ nhớ trả về luôn đến từ vùng HIGH_MEM.

# 7. So sánh giữa vmalloc và kmalloc 



|  | vmalloc | kmalloc |
| -------- | -------- | -------- |
| Physical Memory     |  Phân bổ các trang liền kề về mặt vật lý nhưng không nhất thiết phải liền kề về mặt vật lý   | Đảm bảo các trang liền kề về mặt vật lý và liền kề về mặt ảo     |
| Low mem and high mem     | Trả về từ Bộ nhớ cao    | Trả về từ Bộ nhớ thấp     |
|  Usage    | Bộ nhớ được trả về Không thể được sử dụng bởi các thiết bị phần cứng    |  Bộ nhớ được trả về Có thể được sử dụng bởi các thiết bị phần cứng (DMA, PCI)    |
| Interrupt Context     | không thể được sử dụng trong bối cảnh ngắt    | có thể được sử dụng trong bối cảnh ngắt với 'GFP_ATOMIC'     |
| Allocator     | Sử dụng trực tiếp Bộ phân bổ trang    | Sử dụng bộ phân bổ slab, sau đó sử dụng Bộ phân bổ trang     |
| Overhead     | nhiều chi phí chung hơn, vì mỗi vmalloc yêu cầu thay đổi bảng trang và một bộ đệm tìm kiếm bản dịch không hợp lệ.    | ít chi phí chung hơn     |
| Size    | Hữu ích cho việc phân bổ bộ nhớ lớn và không yêu cầu liên tục vật lý    | Không thể cung cấp bộ nhớ lớn     |


# 8. Kernel Stack 

Trong Hệ thống Linux, mỗi tiến trình có 2 stacks:
* User stack
* Kernel stack 

User stack in x86: Nằm trong không gian địa chỉ người dùng (0-3GB trong x86 32 bit)
Kernel Stack in x86: Nằm trong không gian địa chỉ hạt nhân (3GB-4GB trong x86 32 bit)

Không gian người dùng được hưởng sự xa xỉ của một ngăn xếp lớn, phát triển động, trong khi hạt nhân không có sự xa xỉ như vậy

Kernel stack nhỏ và cố định.

Kích thước của Kernel stack cho mỗi tiến trình phụ thuộc vào cả kiến trúc và tùy chọn thời gian biên dịch.

Khi tùy chọn được bật, mỗi tiến trình chỉ được cấp một trang duy nhất - 4KB trên kiến trúc 32 bit và 8KB trên kiến trúc 64 bit.

Tại sao chỉ có một trang?
1. Tiêu thụ ít bộ nhớ hơn cho mỗi tiến trình
2. Khi thời gian hoạt động tăng lên, ngày càng khó tìm thấy hai trang vật lý liền kề chưa được phân bổ.

Theo truyền thống, trình xử lý ngắt cũng sử dụng Kernel stack của quy trình mà chúng ngắt.

Vì nó đặt ra những ràng buộc chặt chẽ hơn đối với ngăn xếp hạt nhân vốn đã nhỏ hơn. các nhà phát triển hạt nhân đã triển khai một tính năng mới: ngăn xếp ngắt. Ngắt sử dụng ngăn xếp riêng của chúng. Nó chỉ sử dụng một trang duy nhất cho mỗi bộ xử lý.

Bây giờ, chúng ta có kích thước Kernel stack là 16KB từ Linux 3.15 trong x86_64

# 9. Config frame warn 

Tùy chọn cấu hình hạt nhân này truyền một tùy chọn cho trình biên dịch để khiến nó phát ra cảnh báo khi phát hiện kích thước ngăn xếp tĩnh cho một thói quen lớn hơn ngưỡng đã chỉ định.

Nó yêu cầu phiên bản gcc 4.4 trở lên để hoạt động

Tùy chọn gcc được sử dụng là "-Wframe-larger-than=xxx".

Theo mặc định, CONFIG_FRAME_WARN có giá trị là 1024, nhưng bạn có thể đặt thành bất kỳ giá trị nào từ 0 đến 8192.

Kernel Linux định nghĩa kích thước ngăn xếp là 8192 byte cho mỗi process. Tổng các khung ngăn xếp của tất cả các hàm đang hoạt động không được tràn 8192 byte.

Cảnh báo này không đảm bảo rằng bạn sẽ tràn không gian ngăn xếp; nó chỉ cho thấy rằng hàm này làm cho khả năng tràn cao hơn (khi được sử dụng cùng với các hàm khung lớn khác hoặc với nhiều hàm nhỏ hơn). 