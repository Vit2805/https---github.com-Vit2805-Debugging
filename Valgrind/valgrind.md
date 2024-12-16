# 1. Giới thiệu về valgrind
Chủ đề của bài viết hôm nay chúng ta sẽ cùng thảo luện về valgrind. Valgrind là một công cụ mạnh mẽ trong Linux để phân tích và gỡ lỗi các chương trình. Nó chủ yếu được sử dụng để phát hiện các vấn đề liên quan đến quản lý bộ nhớ và hiệu năng trong các chương trình C/C++. Dưới đây là một số thông tin chi tiết về Valgrind, cách sử dụng nó, và các công cụ mà Valgrind cung cấp.

Chúng ta cùng bắt đầu!
# 2. Tổng quan 


![](https://assets.devlinux.vn/uploads/editor-images/2024/11/19/image_f1dc2685eb.png)


# 3. Tính năng

* Có thể chạy trên nhiều nền tảng như x86/Linux, AMD64/Linux và PPC32/Linux.
* Hoạt động với tất cả các bản phân phối Linux chính, bao gồm Red Hat, SuSE, Debian, Gentoo, Slackware, Mandrake, v.v.
* Sử dụng tính năng phát hiện nhị phân động để bạn không cần sửa đổi, biên dịch lại hoặc liên kết lại ứng dụng của mình. Có thể sử dụng trên các chương trình không cần mã nguồn.
* Áp dụng cho các chương trình được viết bằng bất kỳ ngôn ngữ nào, cho dù được biên dịch, biên dịch đúng lúc hay thông dịch. Nó có thể được sủ dụng để gỡ lỗi và phân tích các hệ thống được viết bằng các ngôn ngữ hỗn hợp.
* Công cụ được các chương trình viết bằng nhiều ngôn ngữ sử dụng hơn là C và C++.
* Valgrind sẽ làm cho chương trình chạy chậm hơn đáng kể.
* Valgrind không thể được sử dụng để gỡ lỗi chương trình đang chạy.

# 4. Ví dụ sử dụng

Các tham số quan trọng của Valgrind:

```
--malloc-fill=
--free-fill=
--leak-check=
--show-leak-kinds= 
--track-origins= 
--verbose
--log-file=

```

Chúng ta sẽ qua mục 1.4. để hiểu rõ hơn về các tham số trên.

# 5. Mô tả tham số

## 5.1. Các tham số thường dùng

 `--tool=<toolname> [default: memcheck]` 
* valgrind hỗ trợ nhiều công cụ kiểm tra, công cụ được sử dụng phổ biến nhất là kiểm tra bộ nhớ (memcheck), --tool= được sử dụng để chọn công cụ bạn cần thực thi. Nếu không được chỉ định, mặc định là memcheck.

`--show-leak-kinds=<set> [default: definite, possible]`
*  Đặt loại rò rỉ để kiểm tra: definite, indirect, possible, reachable. Bạn có thể đặt nhiều loại trong 4 loại này, phân tách nhau bằng dấu phẩy.
*  Bạn có thể sử dụng all để sử dụng tất cả các loại vả none để không hiển thị gì.

`--show-reachable=<yes | no>` 
* Kiểm soát xem có phát hiện rò rỉ loại reachable và idirect hay không. Theo mặc định, chỉ các loại definite và possible được phát hiện.
*  Bạn có thể hiểu --show-reachable=yes tương đương với --show-leak-kinds=all

`--show-possibly-lost=<yes | no>` 
* Kiểm soát xem có phát hiện rò rỉ loại possibly hay không.

`--malloc-fill=<hexnumber>`
* Đối với các khối bộ nhớ được phân bổ bằng malloc và new, hexnumber được sử dụng để khởi tạo và điền vào chúng, nhưng những khối được phân bổ bằng calloc thì không được sử dụng.
* Tùy chọn này có thể hữu ích cho những vấn đề hỏng bộ nhớ không dễ phát hiện, giúp dễ dàng quan sát dữ liệu bộ nhớ nào được sử dụng và dữ liệu nào không được sử dụng.

`--free-fill=<hexnumber>` 
* Thường được sử dụng kết hợp với malloc-fill
* Khi giải phóng bộ nhớ bằng cách giải phóng và xóa, khối bộ nhớ sẽ chứa đầy số hex. Sẽ rất thuận lợi để quan sát dữ liệu bố nhớ nào đã được giải phóng và dữ liệu nào chưa.
* Nó có những lợi ích tương tự như --malloc-fill, bộ nhớ đã đầy vẫn không thể truy cập được, nó chỉ ảnh hướng đến nội dung của nó.
`--undef-value-erros=<yes | no>`
* giá trị mặc định là có, có nghĩa là memcheck sẽ báo cáo việc sử dụng các lỗi giá trị không xác định. Điều này ảnh hưởng nhẹ đến tốc độ phát hiện.


`track-origin=<yes | no> [default: no]`
* Lỗi giá trị undef hợp lệ khi được đặt thành có
* Có theo dõi nguồn ngoại lệ bằng cách sử dụng các biến chưa được khởi tạo hay không
* Chỉ thiết lập nó khi bạn chắc chắc muốn phân tích lỗi sử dụng bộ nhớ chưa được khởi tạo. Việc thiết lập thông thường sẽ khiến chương trình thực thi rất chậm.

## 5.2. Hiển thị 

`--leak-check=<no|summary|yes|full>`
* Nếu được đặt thành yes hoặc full, valgrind sẽ mô tả chi tiết từng rò rỉ bộ nhớ sau khi chương trình được gọi kết thúc.
* Mặc định là summary, chỉ báo cáo một số rò rỉ bộ nhớ.


`--verbose`
* Hiển thị chi tiết

`--log-file=<filename>`
`log-fd=<number> [default: 2, stderr]`
* valgrind chuyển nhật ký in vào các tệp hoặc bộ mô tả tệp được chỉ định.
* Nếu không có tham số này, nhật ký của valgrind sẽ được xuất ra cùng với nhật ký của chương trình người dùng, trông sẽ rất lộn xộn.

`--leak-resolution=<low|med|high>`
* Tùy chọn này đặt cách công cụ kiểm tra bộ nhớ, khi phát hiện nhiều rò rỉ bộ nhớ, quy các rò rỉ bộ nhớ này cho cùng một rò rỉ (hợp nhất các rò rỉ gây ra bởi cùng một nguyên nhân)
* Khi được đặt ở mức thấp, cố gắng hợp nhất hai rò rỉ đầu tiên.
* Đặt thành med, đây là bốn cái đầu tiên>
* Giá trị mặc định là cao, cố gắng hợp nhất tất cả các rò rỉ. 
* Tùy chọn này không ảnh hưởng đến khả năng tìm rò rỉ của memcheck mà chỉ ảnh hưởng đến việc hiển thị kết quả phát hiện của nó.

## 5.3. Quy trình con, thư viện tải động và thời gian ghi


`--trace-children=<yes | no> [default: no]`
* Có thể theo dõi các tiến trình con hay không. Nếu đó là chương trình nhiều tiến trình, bạn nên sử dụng chưng năng này. Tuy nhiên, nó sẽ không có nhiều tác động nếu chỉ có một quy trình duy nhất được hoạt động.

`--keep-debuginfo=<yes | no> [default: no]`
* Nếu như chương trình sử dụng thư viện được tải động (dlopen), thông tin gỡ lỗi sẽ bị xóa khi thư viện động được dỡ tải (dlclose). Sau khi bật tùy chọn này, thông tin call stack được giữ lại ngay cả khi thư viện động không được tải.

`--keep-stacktraces=<alloc | free | alloc-and-free | alloc-then-free | none> [default: alloc-and-free]`
* Rò rỉ bộ nhớ không khác gì hơn là sự không khớp giữa ứng dụng và bản phát hành. 
* Nếu ta chỉ tập chung vào vấn đề rò ri bộ nhớ thì thực tế không cần phải xin bản ghi phát hành, vì điều này sẽ chiếm thêm rất nhiều bộ nhớ và tiêu tốn nhiều CPU hơn, khiến chương trình vốn đã chậm lại càng trở nên tồi tệ hơn.


## 5.4. Kiểm tra lỗi và tối ưu hóa bộ nhớ 

`--freelist-vol=<number>`
* Khi một chương trình khách sử dụng free hoặc delete để giải phóng một khối bộ nhớ, khối bộ nhớ sẽ không có sẵn ngay lập tức để phân bổ lại. Nó sẽ chỉ được đặt trong hàng đợi các khối giải phóng (danh sách tự do) và được đánh dấu là không thể truy cập được, điều này tạo điều kiện thuận lợi cho việc phát hiện, khoảng thời gian rất quan trọng, chương trình khách sẽ tủy cập lại vào khối đã giải phóng.
* Tùy chọn này chỉ định kích thước khối byte được chiếm bởi hàng đợi. Mặc định là 20MB.
* Việc tăng tùy chọn này sẽ làm tăng chi phí bộ nhớ của memcheck nhưng khả năng phát hiện các lỗi như vậy cũng sẽ được cải thiện.

`--freelist-big-blocks=<number`
* Khi lấy các khối bộ nhớ có sẽ từ hàng đợi danh sách trống để phân bổ lại. memcheck sẽ lấy ra một khối theo mức độ ưu tiên từ các khối bộ nhớ lớn hơn số đó
* Tùy chọn này ngăn chặn các cuộc gọi thường xuyên đến các khối bộ nhớ nhỏ trong danh sách tự do. Tùy chọn này làm tăng khả năng phát hiện lỗi con trỏ hoang dã đối với các khối bộ nhớ nhỏ.
* Nếu tùy chọn này được đặt thành 0, tất cả các khối sẽ được phân bổ lại trên cơ sở nhập trước, xuất trước.
* Mặc định là 1M.

## 5.5. Các tham số ít được sử dụng hơn

`--partical-loads-ok=<yes | no>`
* Tùy chọn này xác định cách memcheck xử lý việc tải dữ liệu có kích thước từ và căn chỉn từ (từ địa chỉ). Một số khối byte tại các địa chỉ có thể định địa chỉ và một số khác thì không

* Nếu được đặt thành có, việc tải này sẽ không tạo ra bất kỳ lỗi nào. Các byte được tải từ các địa chỉ không hợp lệ sẽ được đánh dấu là chưa được khởi tạo và các byte được tải từ các địa chỉ hợp pháp sẽ được xử lý bình thường.

* Giá trị mặc định là không, có nghĩa là tải từ các địa chỉ không hợp lệ một phần được xử lý giống như tải từ các địa chỉ không hợp lệ hoàn toàn sẽ được báo cáo và các khối byte tương ứng sẽ được đánh dấu là đã khởi tạo.

* Trên thực tế, mã như vậy không tuân theo tiêu chuẩn ISO C/C++. Dù vậy, mã đó phải được sửa chữa. Tùy chọn này nên được sử dụng như là phương sách cuối cùng.

`--workaround-gcc296-bugs=<yes | no>`
* Nếu tùy chọn này được bật, sẽ không có lỗi nào được báo cáo khi đọc hoặc ghi không quá xa con trỏ trên cùng của ngăn xếp (lỗi trong gcc2.96)

* Mặc định là không.

* Cố gắng không kích hoạt tùy chọn này vì nó có thể gây ra các lỗi không dễ phát hiện. Giải pháp là sử dụng GCC mới nhất để khắc phục lỗi này.

* Trên các phiên bản GCC cũ hơn, tùy chọn này có thể được sử dụng.

`--ignore-ranges=0x12-0x34[,0x56-0x78]`
* Địa chỉ giữa các phạm vi này sẽ bị loại bỏ trong quá tình kiểm tra địa chỉ của memcheck
* Có thể có nhiều địa chỉ được phân tách bằng dấu phẩy.

## 5.6. Giải thích 

```
vit@vit-virtual-machine:~$ valgrind -h
usage: valgrind [options] prog-and-args

  tool-selection option, with default in [ ]:
    --tool=<name>             use the Valgrind tool named <name> [memcheck]
                              available tools are:
                              memcheck cachegrind callgrind helgrind drd
                              massif dhat lackey none exp-bbv

  basic user options for all Valgrind tools, with defaults in [ ]:
    -h --help                 show this message
    --help-debug              show this message, plus debugging options
    --help-dyn-options        show the dynamically changeable options
    --version                 show version
    -q --quiet                run silently; only print error msgs
    -v --verbose              be more verbose -- show misc extra info
    --trace-children=no|yes   Valgrind-ise child processes (follow execve)? [no]
    --trace-children-skip=patt1,patt2,...    specifies a list of executables
                              that --trace-children=yes should not trace into
    --trace-children-skip-by-arg=patt1,patt2,...   same as --trace-children-skip=
                              but check the argv[] entries for children, rather
                              than the exe name, to make a follow/no-follow decision
    --child-silent-after-fork=no|yes omit child output between fork & exec? [no]
    --vgdb=no|yes|full        activate gdbserver? [yes]
                              full is slower but provides precise watchpoint/step
    --vgdb-error=<number>     invoke gdbserver after <number> errors [999999999]
                              to get started quickly, use --vgdb-error=0
                              and follow the on-screen directions
    --vgdb-stop-at=event1,event2,... invoke gdbserver for given events [none]
         where event is one of:
           startup exit abexit valgrindabexit all none
    --track-fds=no|yes|all    track open file descriptors? [no]
                              all includes reporting stdin, stdout and stderr
    --time-stamp=no|yes       add timestamps to log messages? [no]
    --log-fd=<number>         log messages to file descriptor [2=stderr]
    --log-file=<file>         log messages to <file>
    --log-socket=ipaddr:port  log messages to socket ipaddr:port
    --enable-debuginfod=no|yes query debuginfod servers for missing
                              debuginfo [yes]

  user options for Valgrind tools that report errors:
    --xml=yes                 emit error output in XML (some tools only)
    --xml-fd=<number>         XML output to file descriptor
    --xml-file=<file>         XML output to <file>
    --xml-socket=ipaddr:port  XML output to socket ipaddr:port
    --xml-user-comment=STR    copy STR verbatim into XML output
    --demangle=no|yes         automatically demangle C++ names? [yes]
    --num-callers=<number>    show <number> callers in stack traces [12]
    --error-limit=no|yes      stop showing new errors if too many? [yes]
    --exit-on-first-error=no|yes exit code on the first error found? [no]
    --error-exitcode=<number> exit code to return if errors found [0=disable]
    --error-markers=<begin>,<end> add lines with begin/end markers before/after
                              each error output in plain text mode [none]
    --show-error-list=no|yes  show detected errors list and
                              suppression counts at exit [no]
    -s                        same as --show-error-list=yes
    --keep-debuginfo=no|yes   Keep symbols etc for unloaded code [no]
                              This allows saved stack traces (e.g. memory leaks)
                              to include file/line info for code that has been
                              dlclose'd (or similar)
    --show-below-main=no|yes  continue stack traces below main() [no]
    --default-suppressions=yes|no
                              load default suppressions [yes]
    --suppressions=<filename> suppress errors described in <filename>
    --gen-suppressions=no|yes|all    print suppressions for errors? [no]
    --input-fd=<number>       file descriptor for input [0=stdin]
    --dsymutil=no|yes         run dsymutil on Mac OS X when helpful? [yes]
    --max-stackframe=<number> assume stack switch for SP changes larger
                              than <number> bytes [2000000]
    --main-stacksize=<number> set size of main thread's stack (in bytes)
                              [min(max(current 'ulimit' value,1MB),16MB)]

  user options for Valgrind tools that replace malloc:
    --alignment=<number>      set minimum alignment of heap allocations [16]
    --redzone-size=<number>   set minimum size of redzones added before/after
                              heap blocks (in bytes). [16]
    --xtree-memory=none|allocs|full   profile heap memory in an xtree [none]
                              and produces a report at the end of the execution
                              none: no profiling, allocs: current allocated
                              size/blocks, full: profile current and cumulative
                              allocated size/blocks and freed size/blocks.
    --xtree-memory-file=<file>   xtree memory report file [xtmemory.kcg.%p]

  uncommon user options for all Valgrind tools:
    --fullpath-after=         (with nothing after the '=')
                              show full source paths in call stacks
    --fullpath-after=string   like --fullpath-after=, but only show the
                              part of the path after 'string'.  Allows removal
                              of path prefixes.  Use this flag multiple times
                              to specify a set of prefixes to remove.
    --extra-debuginfo-path=path    absolute path to search for additional
                              debug symbols, in addition to existing default
                              well known search paths.
    --debuginfo-server=ipaddr:port    also query this server
                              (valgrind-di-server) for debug symbols
    --allow-mismatched-debuginfo=no|yes  [no]
                              for the above two flags only, accept debuginfo
                              objects that don't "match" the main object
    --smc-check=none|stack|all|all-non-file [all-non-file]
                              checks for self-modifying code: none, only for
                              code found in stacks, for all code, or for all
                              code except that from file-backed mappings
    --read-inline-info=yes|no read debug info about inlined function calls
                              and use it to do better stack traces.
                              [yes] on Linux/Android/Solaris for the tools
                              Memcheck/Massif/Helgrind/DRD only.
                              [no] for all other tools and platforms.
    --read-var-info=yes|no    read debug info on stack and global variables
                              and use it to print better error messages in
                              tools that make use of it (Memcheck, Helgrind,
                              DRD) [no]
    --vgdb-poll=<number>      gdbserver poll max every <number> basic blocks [5000]
    --vgdb-shadow-registers=no|yes   let gdb see the shadow registers [no]
    --vgdb-prefix=<prefix>    prefix for vgdb FIFOs [/tmp/vgdb-pipe]
    --run-libc-freeres=no|yes free up glibc memory at exit on Linux? [yes]
    --run-cxx-freeres=no|yes  free up libstdc++ memory at exit on Linux
                              and Solaris? [yes]
    --sim-hints=hint1,hint2,...  activate unusual sim behaviours [none]
         where hint is one of:
           lax-ioctls lax-doors fuse-compatible enable-outer
           no-inner-prefix no-nptl-pthread-stackcache fallback-llsc none
    --scheduling-quantum=<number>  thread-scheduling timeslice in number of
           basic blocks [100000]
    --fair-sched=no|yes|try   schedule threads fairly on multicore systems [no]
    --kernel-variant=variant1,variant2,...
         handle non-standard kernel variants [none]
         where variant is one of:
           bproc android-no-hw-tls
           android-gpu-sgx5xx android-gpu-adreno3xx none
    --merge-recursive-frames=<number>  merge frames between identical
           program counters in max <number> frames) [0]
    --num-transtab-sectors=<number> size of translated code cache [32]
           more sectors may increase performance, but use more memory.
    --avg-transtab-entry-size=<number> avg size in bytes of a translated
           basic block [0, meaning use tool provided default]
    --aspace-minaddr=0xPP     avoid mapping memory below 0xPP [guessed]
    --valgrind-stacksize=<number> size of valgrind (host) thread's stack
                               (in bytes) [1048576]
    --show-emwarns=no|yes     show warnings about emulation limits? [no]
    --require-text-symbol=:sonamepattern:symbolpattern    abort run if the
                              stated shared object doesn't have the stated
                              text symbol.  Patterns can contain ? and *.
    --soname-synonyms=syn1=pattern1,syn2=pattern2,... synonym soname
              specify patterns for function wrapping or replacement.
              To use a non-libc malloc library that is
                  in the main exe:  --soname-synonyms=somalloc=NONE
                  in libxyzzy.so:   --soname-synonyms=somalloc=libxyzzy.so
    --sigill-diagnostics=yes|no  warn about illegal instructions? [yes]
    --unw-stack-scan-thresh=<number>   Enable stack-scan unwind if fewer
                  than <number> good frames found  [0, meaning "disabled"]
                  NOTE: stack scanning is only available on arm-linux.
    --unw-stack-scan-frames=<number>   Max number of frames that can be
                  recovered by stack scanning [5]
    --resync-filter=no|yes|verbose [yes on MacOS, no on other OSes]
              attempt to avoid expensive address-space-resync operations
    --max-threads=<number>    maximum number of threads that valgrind can
                              handle [500]
    --realloc-zero-bytes-frees=yes|no [yes on Linux glibc, no otherwise]
                              should calls to realloc with a size of 0
                              free memory and return NULL or
                              allocate/resize and return non-NULL

  user options for Memcheck:
    --leak-check=no|summary|full     search for memory leaks at exit?  [summary]
    --leak-resolution=low|med|high   differentiation of leak stack traces [high]
    --show-leak-kinds=kind1,kind2,.. which leak kinds to show?
                                            [definite,possible]
    --errors-for-leak-kinds=kind1,kind2,..  which leak kinds are errors?
                                            [definite,possible]
        where kind is one of:
          definite indirect possible reachable all none
    --leak-check-heuristics=heur1,heur2,... which heuristics to use for
        improving leak search false positive [all]
        where heur is one of:
          stdstring length64 newarray multipleinheritance all none
    --show-reachable=yes             same as --show-leak-kinds=all
    --show-reachable=no --show-possibly-lost=yes
                                     same as --show-leak-kinds=definite,possible
    --show-reachable=no --show-possibly-lost=no
                                     same as --show-leak-kinds=definite
    --xtree-leak=no|yes              output leak result in xtree format? [no]
    --xtree-leak-file=<file>         xtree leak report file [xtleak.kcg.%p]
    --undef-value-errors=no|yes      check for undefined value errors [yes]
    --track-origins=no|yes           show origins of undefined values? [no]
    --partial-loads-ok=no|yes        too hard to explain here; see manual [yes]
    --expensive-definedness-checks=no|auto|yes
                                     Use extra-precise definedness tracking [auto]
    --freelist-vol=<number>          volume of freed blocks queue     [20000000]
    --freelist-big-blocks=<number>   releases first blocks with size>= [1000000]
    --workaround-gcc296-bugs=no|yes  self explanatory [no].  Deprecated.
                                     Use --ignore-range-below-sp instead.
    --ignore-ranges=0xPP-0xQQ[,0xRR-0xSS]   assume given addresses are OK
    --ignore-range-below-sp=<number>-<number>  do not report errors for
                                     accesses at the given offsets below SP
    --malloc-fill=<hexnumber>        fill malloc'd areas with given value
    --free-fill=<hexnumber>          fill free'd areas with given value
    --keep-stacktraces=alloc|free|alloc-and-free|alloc-then-free|none
        stack trace(s) to keep for malloc'd/free'd areas       [alloc-and-free]
    --show-mismatched-frees=no|yes   show frees that don't match the allocator? [yes]
    --show-realloc-size-zero=no|yes  show realocs with a size of zero? [yes]

  Extra options read from ~/.valgrindrc, $VALGRIND_OPTS, ./.valgrindrc

  Memcheck is Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
  Valgrind is Copyright (C) 2000-2017, and GNU GPL'd, by Julian Seward et al.
  LibVEX is Copyright (C) 2004-2017, and GNU GPL'd, by OpenWorks LLP et al.

  Bug reports, feedback, admiration, abuse, etc, to: www.valgrind.org.


```

# 6.  Ví dụ sử dụng valgrind

Để hiểu rõ hơn về valgrind tôi có một số ví dụ sau: 

## 6.1. Ví dụ 1
Giả sử bạn có chương trình C có tên exam.c:

```c
#include <stdlib.h>

int main() {
    int *arr = (int *)malloc(10 * sizeof(int));  // Cấp phát bộ nhớ
    arr[10] = 100;  // Lỗi truy cập ngoài vùng nhớ (ngoài phạm vi cấp phát)
    free(arr);  // Giải phóng bộ nhớ
    return 0;
}

```
Biên dịch: 
`gcc -o exam exam.c`
Sau khi biên dịch, ta sử dụng valgrind để phân tích, cú pháp:
`valgrind --leak-check=full ./exam`
Kết quả đầu ra từ valgrind như sau: 
![](https://assets.devlinux.vn/uploads/editor-images/2024/11/20/image_9ece4f91a5.png)


Đọc hiểu kết qủa từ valgrind: 
* Invalid write of size 4: Có lỗi truy cập ngoài vùng nhớ. Chương trình cố gắng ghi dữ liệu kích thước 4 bytes ngoài phạm vi được cấp phát.
* Address 0x5203040 is 0 bytes after a block of size 40 alloc'd: Địa chỉ bộ nhớ không hợp lệ.
* HEAP SUMMARY: Thông tin tóm tắt về việc sử dụng bộ nhớ heap.
* in use at exit: Cho biết bộ nhớ vẫn còn sử dụng khi chương trình kết thúc.
* total heap usage: Tổng số lần cấp phát và giải phóng bộ nhớ.

=> Đây là lỗi truy cập bộ nhớ ngoài phạm vi được cho phép.

## 6.2. Ví dụ 2
Code: 
#include <stdlib.h>
#include <stdio.h>

int main() {
    int *ptr = (int *)malloc(5 * sizeof(int));  // Cấp phát bộ nhớ cho mảng 5 phần tử
    if (ptr == NULL) {
        fprintf(stderr, "Không thể cấp phát bộ nhớ!\n");
        return 1;
    }

    ptr[0] = 10;
    ptr[1] = 20;
    ptr[2] = 30;

    // Không gọi free(ptr), dẫn đến rò rỉ bộ nhớ
    return 0;
}
Biên dịch: 
`gcc -o memory_leak memory_leak.c`

Sử dụng valgrind để tìm lỗi: 
`valgrind --leak-check=full ./memory_leak`

![](https://assets.devlinux.vn/uploads/editor-images/2024/11/20/image_ef19b55025.png)

Giải thích kết quả: 
* Valgrind báo cáo rằng có 20bytes bộ nhớ bị mất. Lỗi này xảy ra khi không có lệnh free(ptr) để giải phóng bộ nhớ được cấp phát.

=> Đây là lỗi memory leak

# 7. Kết luận 

Vừa rồi chúng ta đã cùng nhau tìm hiểu về cách sử dụng cơ bản valgrind. Hy vọng chúng ta sẽ gặp lại nhau ở những bài viết sắp tới và chúc bạn học tốt!