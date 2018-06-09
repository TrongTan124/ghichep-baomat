## Giới thiệu

burpsuite là một công cụ giúp cho bạn rà quét bảo mật.

## Thiết lập

Đầu tiên, bạn cần phải cài đặt java để chạy công cụ này.

Bạn vào trang [này](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)  để tải file cài đặt môi trường java vào máy tính.

Tôi sử dụng windows 10 nên sẽ tải phiên bản như ảnh sau

![1.download_java](/images/1.download_java.png)

Sau khi tải xong, bạn cài đặt như một chương trình bình thường.

Tiếp theo, bạn tải công cụ burpsuite ở trong thư mục tools của github này về.

Bạn chạy file `burp-loader-keygen.jar` trước, khi có bảng hiện lên, bạn nhấn vào `Run`

![2.install_burp_suite](/images/2.install_burp_suite.png)

Bạn copy license từ bảng trên và paste vào ô hiện ra, sau đó chọn next

![3.setup_burp_suite0](/images/3.setup_burp_suite0.png)

Bạn copy `Activation Request` vào lại bảng đầu tiên để lấy mã `Activation Response`.

Kết quả sau khi active xong, bạn chọn như hình và next --> Start burp

![3.setup_burp_suite1](/images/3.setup_burp_suite1.png)

Giao diện chương trình như sau

![3.setup_burp_suite2](/images/3.setup_burp_suite2.png)

Tới đây, chúng ta sẽ bắt tay vào học cách rà quét.

## Thực hành với HTTP

Đầu tiên, bạn phải xác định được website mục tiêu. Tôi nhắm tới một website không sử dụng giao thức https làm nạn nhân trước. Ở đây, tôi dựng lab một website http 
trước `http://172.16.68.19/login/`

Trên trình duyệt bạn đang sử dụng, bạn cần phải cài đặt một proxy để điều hướng request đi qua `burpsuite`, có nhiều proxy plugin cho bạn sử dụng, tôi chọn `proxy switcher`

![4.proxy_plugin](/images/4.proxy_plugin.png)

Cách cài đặt là vào plugin của chrome và tìm kiếm, sau đó thêm vào trình duyệt, bước này tôi ko hướng dẫn ở đây.

Khi cài xong plugin, bạn vào tab `Manual Proxy` để khai báo proxy điều hướng sang `burpsuite`

![4.proxy_plugin1](/images/4.proxy_plugin1.png)

Trên burp suite, ta kiểm tra và thiết lập proxy tương ứng 

![5.proxy_burp_suite](/images/5.proxy_burp_suite.png)

Sau khi xong, ta chuyển sang tab như hình dưới

![5.proxy_burp_suite1](/images/5.proxy_burp_suite1.png)

Lúc này, bạn vào website mục tiêu và đăng nhập, các gói tin sẽ được chuyển tiếp qua proxy burp suite. Bạn Forward các gói tin ko cần thiết để tìm đúng đến gói tin mà 
bạn vừa làm

![6.capture_package](/images/6.capture_package.png)

Bạn chuột phải và chọn `Send to Repeater`, sau đó chọn vào `Intercept if off` để cho bỏ việc bắt gói. Lúc này, bạn đã có đầy đủ thông tin mà request login vừa được trình duyệt gửi đi.

Bây giờ làm gì tiếp theo. Vâng, bây giờ thì bạn sử dụng cái packet mà vừa bắt được để tấn công. Có thể là bruce force (nếu phía webserver ko giới hạn số lần đăng nhập. hoặc 
đơn giản là khai thác lỗi XSS. Lỗi XSS là lỗi tấn công về phía trình duyệt của người dùng để ăn cắp cookies hoặc thông tin đăng nhập.

Bạn chuyển sang tab Repeater để thực hiện tiếp.

![6.capture_package1](/images/6.capture_package1.png)

Bạn thay đổi thông tin `username` và `password` để gửi lại package này

![6.capture_package2](/images/6.capture_package2.png)

Chọn `Go` sẽ gửi gói tin với thông tin được chỉnh sửa lên webserver, kết quả sẽ ra dữ liệu Raw hoặc được Render cho hiển thị dễ nhìn hơn.

![6.capture_package3](/images/6.capture_package3.png)

Nếu trình duyệt gửi lại cho bạn một popup có giá trị 1 thì xin chúc mừng bạn, website này đang bị lỗi XSS nhé. 

Cách sử dụng Repeater chỉ là thủ công, nếu bạn muốn tự động khai thác thì sao, bạn cũng bắt gói tương tự nhưng thay vì `Send to Repeater` thì giờ bạn `Send to Intruder`. Sau 
đó bạn sang tab `Intruder` để làm tiếp.

![6.capture_package4](/images/6.capture_package4.png)

Lúc này, các dữ liệu được thay đổi phía trình duyệt trước khi gửi lên server sẽ có dấu `§` ở đầu và cuối, bạn chọn `clear §` để xóa các đánh dấu này đi. Mục tiêu đánh dấu là 
báo cho bạn đây là các biến có thể thay đổi trước khi gửi đi. Chúng ta chọn biến nào mong muốn sửa, ở đây tôi chọn username và password

![6.capture_package5](/images/6.capture_package5.png)

Ngoài ra, bạn phải chọn cách thức tấn công đối với loại package này. vì tôi chọn 02 biến nên cách thức tôi sử dụng là `Cluster bombs`

![6.capture_package6](/images/6.capture_package6.png)

Bây giờ sang tiếp tab `Payloads` để tùy chỉnh thông tin cho các biến. Tại đây, bạn chọn định nghĩa cho từng biến, ở payload set ta chọn biến số 1 cho username, 

![6.capture_package7](/images/6.capture_package7.png)

sau đó, phần payload options bạn add các giá trị mong muốn điền vào, có thể sử dụng công cụ điền tự động của burp suite bằng cách chọn `Add from list Usernames`. 
Phần payload processing, chọn các cơ chế xử lý đối với dữ liệu của biến như mã hóa trước khi gửi đi bằng cách chọn như ảnh

![6.capture_package8](/images/6.capture_package8.png)

Làm tương tự với biến thứ 2, nhưng lần này chọn Payload Options là passwords. Sau đó chọn `Start attack`

![6.capture_package9](/images/6.capture_package9.png)

Giờ bạn chờ đợi website mục tiêu lộ sơ hở thôi. :) Việc attack này là vét cạn nên sẽ là tổ hợp của username + password. Nếu máy của bạn đủ mạnh thì cứ để thôi. 

![6.capture_package10](/images/6.capture_package10.png)

Vì là khai thác XSS nên trong phần Payload options bạn nên chọn `Fuzzing XSS`

## Thực hành với HTTPS

Vậy với HTTPS có sử dụng cert thì ta làm cách nào. 

Bạn bật proxy của trình duyệt và của burp suite lên. Từ trình duyệt bạn gõ `burp/` và nhấn enter

![7.https1](/images/7.https1.png)

Tại đây, bạn download cert về. Sau đó, trên trình duyệt chrome, bạn vào cài đặt và tìm đến phần import cert

![7.https2](/images/7.https2.png)

Bạn chọn tab `Trusted Root Certification Authoritíe` và Import cert vừa download trên burp về

Giờ thì bạn đã có cert để đánh lạc hướng trình duyệt khi tải các website sử dụng HTTPS rồi. Bạn sẽ thấy tất cả các website HTTPS sau khi đi qua proxy để là HTTP hết. 
Giờ thì thực hành tương tự với HTTP thôi. :D

**NOTE** Nên tắt Proxy đi khi bạn đã vọc vạch xong. Cảm ơn.

## Tham khảo
