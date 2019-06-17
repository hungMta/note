# Cài đặt thanh toán với Apple Pay

![alt text](https://cdn.shopifycloud.com/help/assets/manual/settings/payments/applepay-banner-art-649c8d0cebd461aaff7b542679b98f73c98a9ef597410ea5b9f55502e31a39a0.jpg)

Apple Pay là dịch vụ thanh toán di động được Apple Inc cung cấp trên các thiết bị iPhone 6 hoặc mới hơn.


## Cài đặt
Để cài đặt Apple Pay trong ứng dụng, cần phải hoàn thành 3 bước sau:
+ Tạo merchant ID.
+ Tạo Payment Processing Certificate.
+ Enable Apple Pay trong Xcode.

### Tạo Merchant ID. 
Merchant ID là một định danh được đăng ký với Apple, định danh này sẽ là
duy nhất. Merchant ID không bao giờ hết hạn và nó có thể sự dụng cho nhiều website hay iOS app.
1. Trong phần [Certificates, Identifiers & Profiles]() chọn `Identifiers`, sau đó chọn  Add  (+) ở góc trên bên trái
2. Chọn Merchant IDs, chọn Continue.
3. Điền mô tả và  định danh cho Merchant ID, chọn Continue.
4. Xem lại thông tin đã cài đặt, chọn Register

### Tạo Payment Processing Certificate
Payment processing certificate liên kết với merchant ID và được sử dụng để mã hoá thông tin thanh toán. Payment processing certificate sẽ hết hạn sau 25 tháng, sau khi hết hạn thì có thể tạo mới một certificate khác.
1. Trong phần [Certificates, Identifiers & Profiles](), chọn `Identifiers`
2. Ở phần `Identifiers` chọn mục `Merchant IDs`.
3. Chọn Edit ở merchant id mà đã tạo trước đó.
4. Ở phần Apple Pay Payment Processing Certificate chọn `Create Certificate`
5. Tạo một [Certificate Signing Request](https://help.apple.com/developer-account/#/devbfa00fef7?sub=dev103e030bb) trên máy Mac của mình
6. Upload Certificate ở step 4.
7. chọn Done.

### Enable Apple Pay trong Xcode
![alt text](https://help.apple.com/xcode/mac/9.3/en.lproj/Art/ca_enable_apple_pay.png)
**Enable Apple Pay**
1. Trong mục [project editor], chọn target -> chọn `Capbilites`
2. Chọn mục `Apple Pay`, chọn On.

**Tạo merchant identifier**
1. Trong mục Apple Pay, chọn Add (+)
2. Điền định danh cho merchant id -> chọn OK.

Ta vừa cài đặt thành công Apple Pay trong app, phần tiếp mình sẽ test apple pay với sandbox.

## Test Apple Pay với SandBox
### Sandbox
Môi trường Sandbox cho phép developer kiểm thử tích hợp apple pay trong ứng dụng với test credit và debit card. Hiện tại sandbox chỉ hỗ trợ test Apple Pay trên các khu vực 
![alt text](http://prntscr.com/nxextq)

### Tạo Sandbox tester account
1. Đăng nhập vào [App Store Connect](https://appstoreconnect.apple.com/)
2. Ở phần hompage, chọn Users and Access
3. Ở mục Sandbox, chọn Testers.
4. Chọn (+) để thêm mới một tester account
5. Điền thông tin tester: email, giới tính, tình trạng hôn nhân..... Lưu ý: email đăng kí phải là email chưa được đăng ký Apple ID.
6. Đăng xuất Apple ID trên điện thoại, và login bằng tài khoản tester vừa đăng ký ở trên.

### Thêm Test Card vào ví
1. Đảm bảo rằng bạn đã login tài khoản iCloud bằng tài khoản test vừa đăng ký
2. Chọn mục `Wallet & Apple Pay` trong phần setting. (Nế bạn không tìm thấy mục này thì là do máy của bạn đang để Region không cho phép thanh toán Apple Pay. Hãy chọn lại Region của máy sang các khu vực được hỗ trợ.)
3. Chọn thêm Credit or Debit Card
4. Điền thông tin [thẻ test](https://developer.apple.com/apple-pay/sandbox-testing/) đã được cung cấp trên trang chủ
5. Bây giờ bạn đã có thể test apple pay.

Sau khi đã cài đặt đầy đủ môi trường và tài khoản test, thì bạn có thể triển khai code và test thanh toán Apple Pay với ứng dụng của mình.


### Coding
Sau khi cài đặt Apple Pay thành công giờ đến bước triển khai code. 
Tạo một ViewController đơn giản như sau

![alt text](http://prntscr.com/o2psmm)

import thư viện Passkit trong ViewController để thực hiện thanh toán của Apple Pay.

```swift
import Passkit
```
Định nghĩa merchantId và các loại thẻ thanh toán 
```swift
let supportedPaymentNetworks = [PKPaymentNetwork.visa, PKPaymentNetwork.masterCard]
let applePayMerchantID = "merchant.jp.pay.test"
```
Việc thanh toán được xử lý qua lớp `PKPaymentAuthorizationViewController`, lớp này cần truyền vào là một PKPaymentRequest
```swift
  @IBAction func applePayOnClick(_ sender: Any) {
        print("applePay")
        let request = createPaymentRequest()
        let applePayController = PKPaymentAuthorizationViewController(paymentRequest: request)!
        applePayController.delegate = self
        self.present(applePayController, animated: true) {
            print("done")
        }
    }
```

Tạo `PaymentRequest`
```swift
    func createPaymentRequest() -> PKPaymentRequest {
        let request = PKPaymentRequest()
        request.merchantIdentifier = applePayMerchantID
        request.supportedNetworks = supportedPaymentNetworks
        request.merchantCapabilities = PKMerchantCapability.capability3DS
        request.countryCode = "VN"
        request.currencyCode = "USD"
        request.paymentSummaryItems = [
            PKPaymentSummaryItem(label: "Apple", amount: 9.99),
            PKPaymentSummaryItem(label: "Total", amount: 9.99)
        ]
        
        return request
    }
```
Các thuộc tính của PaymentRequest được cài đặt như sau:

1. `merchantIdentifier`, `supportedNetworks` : merchantId mà bạn đã cài đặt trước đó và loại card được phép thanh toán.
2. `merchantCapabilities`: tiêu chuẩn bảo mật, `MerchantCapability.capability3DS` là tiêu chuẩn bảo mật thường dùng ở Mỹ.
3.  `countryCode`, `currencyCode`: mã vùng và mã tiền.
4. `paymentSummaryItems`: Danh sách các item thanh toán, item cuối cùng được coi như  tổng số tiền phải thanh toán.

implement delegate xử lý thanh toán
```swift
  func paymentAuthorizationViewControllerDidFinish(_ controller: PKPaymentAuthorizationViewController) {
        controller.dismiss(animated: true, completion: nil)
    }
    
    func paymentAuthorizationViewController(_ controller: PKPaymentAuthorizationViewController, didAuthorizePayment payment: PKPayment, handler completion: @escaping (PKPaymentAuthorizationResult) -> Void) {
        completion(.init(status: .success, errors: nil))
    }
```

1. `paymentAuthorizationViewControllerDidFinish`: được gọi khi hoàn thành thanh toán.
2. `paymentAuthorizationViewController`: xử lý xác thực người dùng thanh toán. Xác thực thành công thì phải gọi `completion(.init(status: .success, errors: nil))` để hoàn thành thanh toán.
Giao diện thanh toán như sau

![alt text](http://prntscr.com/o2q5a0)



# Tham Khảo
1. https://developer.apple.com/documentation/passkit/apple_pay/setting_up_apple_pay_requirements?language=objc
2. https://developer.apple.com/apple-pay/sandbox-testing/

3. https://www.raywenderlich.com/2113-apple-pay-tutorial-getting-started