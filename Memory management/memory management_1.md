# 1. Giới thiệu về Memory Management 

Quản lý bộ nhớ trong nhân Linux (Memory Management in the Linux Kernel) là một trong những thành phần quan trọng và phức tạp nhất. Nó liên quan đến việc quản lý tài nguyên bộ nhớ hệ thống một cách hiệu quả và tối ưu để đảm bảo hoạt động của các ứng dụng và hệ điều hành. 

Trong bài viết này tôi và các bạn sẽ cùng nhau tìm hiểu về quản lý bộ nhớ trong nhân Linux nhé.

# 2. Không gian địa chỉ vật lý là gì?

## 2.1. Khái niệm 

Toàn bộ phạm vi địa chỉ bộ nhớ có thể truy cập được bởi bộ xử lý thường được gọi là không gian địa chỉ vật lý.

Ví dụ hệ thống 32bit có thể có không gian địa chỉ là 2^32 = 4GB. 4G không gian địa chỉ không chỉ được sử dụng bởi ram mà còn được sử dụng bởi những ngoại vi khác (peripheral) ví dụ như:
* RAM
* BIOS
* APIC
* PCI 
* Các thiết bị I/O được ánh xạ bộ nhớ khác

Hình dưới đây sẽ giúp bạn hiểu hơn về tổ chức trong không gian địa chỉ vật lý: 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/26/image_d308b8f07a.png)

## 2.2. Cách theo dõi memory map/physical address space trong Linux

Bằng cách sử dụng cú pháp *sudo cat /proc/iomem* bạn có thể theo dõi memory map của hệ thống:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/26/image_3e736c46cf.png)

Bạn có thể thấy được địa chỉ được sử dụng bởi những thiết bị khác.

# 3. Không gian địa chỉ ảo 

Trên Linux, mọi địa chỉ bộ nhớ đều là ảo. Chúng không trỏ trực tiếp đến bất kỳ địa chỉ nào trong RAM.

Bất cứ khi nào bạn truy cập vào một vị trí bộ nhớ, một cơ chế biên dịch sẽ được thực hiện để khớp với bộ nhớ vật lý tương ứng.

Trên Hệ thống Linux, mỗi quy trình sở hữu một không gian địa chỉ ảo.

Kích thước của không gian địa chỉ ảo là 4GB trên các hệ thống 32 bit (ngay cả trên hệ thống có bộ nhớ vật lý nhỏ hơn 4 GB)

Linux chia không gian địa chỉ ảo này thành

* một vùng dành cho các ứng dụng, được gọi là không gian người dùng (User space addresses).

* một vùng dành cho hạt nhân, được gọi là không gian hạt nhân (Kernel addresses).

Sự phân chia giữa hai vùng này được thiết lập bởi tham số cấu hình hạt nhân có tên là PAGE_OFFSET.

Điều này được gọi là Phân chia 3G/1G:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/26/image_618bb390de.png)

# 4. Pages-PAGE_SIZE marco 

Không gian địa chỉ ảo *(0x00000000 đến 0xffffffff)* được chia thành các trang 4096 byte.

Kích thước trang có thể khác nhau trong các hệ thống khác. Nhưng trên ARM và x86 thì kích thước là cố định.

Kích thước của một trang được xác định trong hạt nhân thông qua macro PAGE_SIZE.

Các trang trong không gian địa chỉ ảo được ánh xạ tới các địa chỉ vật lý bởi Đơn vị quản lý bộ nhớ (MMU), sử dụng bảng trang để thực hiện ánh xạ.

Memory Page/Virtual Page/Page: 
* Chỉ một khối bộ nhớ ảo liền kề có độ dài cố định.
* Cấu trúc dữ liệu hạt nhân để biểu diễn một trang bộ nhớ là struct page

Frame/Page Frame:
* Chỉ một khối bộ nhớ vật lý liền kề có độ dài cố định mà trên đó hệ điều hành ánh xạ một trang bộ nhớ
* Mỗi khung trang được cấp một số khung trang (PFN).
* Với một trang, bạn có thể dễ dàng lấy PFN của trang đó và ngược lại, bằng cách sử dụng macro *page_to_pfn/pfn_to_page*

Page Table:

Cấu trúc dữ liệu hạt nhân và kiến trúc được sử dụng để lưu trữ ánh xạ giữa các địa chỉ ảo và địa chỉ vật lý
Mỗi mục mô tả cặp khóa page/frame

Lệnh để tìm page size 

`getconf PAGESIZE`
hoặc
`getconf PAGE_SIZE`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/26/image_ffb34a2ea5.png)

# 5. Struct page

Kernel đại diện cho mọi trang ảo trên hệ thống với cấu trúc struct page.

Header File: <linux/mmtypes.h>
```c
struct page {
        unsigned long flags;
	atomic_t      _count;
	void          *virtual;
	....
};
````
Flags: Trạng thái của trang: Bẩn, bị khóa trong bộ nhớ.
Values: <linux/page-flags.h>

_count: Số lần sử dụng của trang. Có bao nhiêu tham chiếu đến trang này. Khi trang rảnh, _count là số âm một

virtual: Địa chỉ ảo của trang.

# 6. Page fault

Bộ nhớ hạt nhân được quản lý theo cách khá đơn giản.

Bộ nhớ này không được phân trang theo yêu cầu, nghĩa là đối với mọi phân bổ sử dụng kmalloc() hoặc hàm tương tự, đều có bộ nhớ vật lý thực.

Bộ nhớ hạt nhân không bao giờ bị loại bỏ hoặc phân trang.

Linux sử dụng chiến lược phân bổ lười biếng cho không gian người dùng, chỉ ánh xạ các trang bộ nhớ vật lý khi chương trình truy cập vào đó.

Ví dụ, phân bổ bộ đệm 1 MiB bằng malloc(3) trả về một con trỏ tới một khối địa chỉ bộ nhớ nhưng không có bộ nhớ vật lý thực tế.

Một cờ được đặt trong các mục bảng trang sao cho bất kỳ quyền truy cập đọc hoặc ghi nào đều bị hạt nhân giữ lại. Điều này được gọi là lỗi trang.

Chỉ tại thời điểm này, hạt nhân mới cố gắng tìm một trang bộ nhớ vật lý và thêm trang đó vào ánh xạ bảng trang cho quy trình.

# 7. Không gian địa chỉ ảo của không gian người dùng


![](https://assets.devlinux.vn/uploads/editor-images/2024/11/26/image_1eacf53bfa.png)

# 8. Không gian địa chỉ ảo của Kernel - low mem and high mem 

Nhân Linux có không gian địa chỉ ảo riêng, giống như mọi quy trình chế độ người dùng khác.

Kernel code và cấu trúc dữ liệu phải phù hợp với không gian đó, nhưng bộ phận tiêu thụ không gian địa chỉ nhân lớn nhất là ánh xạ ảo cho bộ nhớ vật lý.

Nhân để truy cập bộ nhớ vật lý trước tiên phải ánh xạ bộ nhớ đó vào không gian địa chỉ ảo của nhân.

Lượng bộ nhớ vật lý tối đa mà nhân xử lý = lượng bộ nhớ có thể được ánh xạ vào phần không gian địa chỉ ảo của nhân - Không gian mà kernel code sử dụng.

Do đó, hệ thống Linux dựa trên x86 có thể hoạt động với tối đa dưới 1 GB bộ nhớ vật lý.

Không gian địa chỉ ảo của nhân (kích thước 1 GB trong phân chia 3G/1G) được chia thành hai phần:

* Bộ nhớ thấp hoặc LOWMEM, là 896 MB đầu tiên

* Bộ nhớ cao hoặc HIGHMEM, được biểu thị bằng 128 MB trên cùng

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/26/image_1cec1000d0.png)

## 8.1. Low memory

896 MB đầu tiên của không gian địa chỉ hạt nhân tạo nên vùng bộ nhớ thấp.

Vào đầu quá trình khởi động, hạt nhân ánh xạ vĩnh viễn 896 MB này

Các địa chỉ kết quả từ ánh xạ này được gọi là địa chỉ logic.

Đây là các địa chỉ ảo, nhưng có thể được chuyển đổi thành địa chỉ vật lý bằng cách trừ đi một độ lệch cố định, vì ánh xạ là vĩnh viễn và được biết trước.

Bạn có thể chuyển đổi địa chỉ vật lý thành địa chỉ logic bằng cách sử dụng macro *__pa(address), sau đó hoàn nguyên bằng macro __va(address)*.

Bộ nhớ thấp khớp với giới hạn dưới của địa chỉ vật lý.

Trên thực tế, để phục vụ các mục đích khác nhau, bộ nhớ hạt nhân được chia thành một vùng.

Sau đó, chúng ta có thể xác định ba vùng bộ nhớ khác nhau trong không gian hạt nhân:

* ZONE_DMA: Vùng này chứa các khung trang bộ nhớ dưới 16 MB, dành riêng cho Truy cập bộ nhớ trực tiếp (DMA)
* ZONE_NORMAL: Vùng này chứa các khung trang bộ nhớ trên 16 MB và dưới 896 MB, * dành cho mục đích sử dụng thông thường
* ZONE_HIGHMEM: Vùng này chứa các khung trang bộ nhớ ở mức 896 MB trở lên

Trên hệ thống 512 MB, sẽ không có ZONE_HIGHMEM, 16 MB cho ZONE_DMA và 496 MB cho ZONE_NORMAL.

## 8.2. High memory

128 MB trên cùng của không gian địa chỉ hạt nhân được gọi là vùng bộ nhớ cao.

Hạt nhân sử dụng vùng này để ánh xạ tạm thời bộ nhớ vật lý trên 1 GB

Khi cần truy cập bộ nhớ vật lý trên 1 GB (hay chính xác hơn là 896 MB), hạt nhân sẽ sử dụng 128 MB đó để tạo ánh xạ tạm thời tới không gian địa chỉ ảo của nó, do đó đạt được mục tiêu có thể truy cập tất cả các trang vật lý.

Bộ nhớ vật lý trên 896 MB được ánh xạ theo yêu cầu tới 128 MB của vùng HIGHMEM.

Ánh xạ để truy cập bộ nhớ cao được hạt nhân tạo ra ngay lập tức và bị hủy khi hoàn tất. Điều này làm cho việc truy cập bộ nhớ cao chậm hơn.

Khái niệm bộ nhớ cao không tồn tại trên các hệ thống 64 bit, do phạm vi địa chỉ rất lớn (2^64), trong đó việc chia 3G/1G không còn hợp lý nữa.

# 9. Memory Allocation Mechanisms

Có một cơ chế phân bổ để đáp ứng mọi loại yêu cầu bộ nhớ.

Tùy thuộc vào mục đích sử dụng bộ nhớ, bạn có thể chọn cơ chế gần nhất với mục tiêu của mình.

Bộ phân bổ chính là bộ phân bổ trang, chỉ hoạt động với các trang (trang là đơn vị bộ nhớ nhỏ nhất mà nó có thể cung cấp)

Sau đó là bộ phân bổ SLAB được xây dựng trên bộ phân bổ trang, lấy các trang từ nó và trả về các thực thể bộ nhớ nhỏ hơn (thông qua các slab và bộ đệm). Đây là bộ phân bổ mà bộ phân bổ kmalloc dựa vào.

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/27/image_9d06fdaae2.png)

## 9.1. Kmalloc 

kmalloc là một hàm phân bổ bộ nhớ hạt nhân, chẳng hạn như malloc() trong không gian người dùng

Bộ nhớ được trả về bởi kmalloc là liền kề trong bộ nhớ vật lý và trong bộ nhớ ảo:

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/27/image_a8ce59533a.png)

kmalloc allocator là bộ phân bổ bộ nhớ chung và cấp cao hơn trong kernel và dựa vào SLAB Allocator

Bộ nhớ trả về từ kmalloc có địa chỉ logic kernel vì nó được phân bổ từ vùng LOW_MEM, trừ khi HIGH_MEM được chỉ định.

Header File: #include <linux/slab.h>

`void *kmalloc(size_t size, int flags);`

* size: chỉ định kích thước của bộ nhớ được phân bổ (tính bằng byte).
* flags: xác định cách thức và vị trí bộ nhớ sẽ được phân bổ.
* Các cờ có sẵn giống như bộ phân bổ trang (GFP_KERNEL, GFP_ATOMIC, GFP_DMA, v.v.)


Giá trị trả về: Khi thành công, trả về địa chỉ ảo của khối được phân bổ, được đảm bảo là liền kề về mặt vật lý. Khi xảy ra lỗi, nó trả về NULL


**Flags: **
* GFP_KERNEL: Đây là cờ chuẩn. Chúng ta không thể sử dụng cờ này trong trình xử lý ngắt vì mã của nó có thể ngủ. Nó luôn trả về bộ nhớ từ vùng LOM_MEM (do đó là địa chỉ logic).

* GFP_ATOMIC: Điều này đảm bảo tính nguyên tử của việc phân bổ. Cờ duy nhất để sử dụng khi chúng ta ở trong ngữ cảnh ngắt.

* GFP_USER: Điều này phân bổ bộ nhớ cho một quy trình không gian người dùng. Sau đó, bộ nhớ sẽ riêng biệt và tách biệt với bộ nhớ được phân bổ cho hạt nhân.

* GFP_HIGHUSER: Điều này phân bổ bộ nhớ từ vùng HIGH_MEMORY.

* GFP_DMA: Điều này phân bổ bộ nhớ từ DMA_ZONE.

## 9.2. Kfree

Hàm kfree được sử dụng để giải phóng bộ nhớ được phân bổ bởi kmalloc. Sau đây là nguyên mẫu của kfree():

`void kfree(const void *ptr);`

Có thể xảy ra tình trạng hỏng bộ nhớ:
* trên một khối bộ nhớ đã được giải phóng
* trên một con trỏ không phải là địa chỉ được trả về từ kmalloc()
Bạn hãy luôn cân bằng các lần phân bổ và giải phóng để đảm bảo rằng kfree() được gọi chính xác một lần trên con trỏ chính xác.

# Kết luận 

Trên đây là những kiến thức cơ bản giúp người mới có thể làm quen với công việc quản lý bộ nhớ, một công việc rất quan trọng với lập trình nhúng. Chúng ta sẽ gặp lại nhau ở bài viết tiếp theo để tìm hiểu thêm về Memory management nhé!
