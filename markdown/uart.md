# Câu 1: Trình bày những khái niệm, lý thuyết cơ bản và ứng dụng về chuẩn giao tiếp truyền thông nối tiếp UART

## 1. Khái niệm UART

UART là viết tắt của **Universal Asynchronous Receiver/Transmitter**, nghĩa là **bộ truyền nhận nối tiếp không đồng bộ vạn năng**.

UART là một chuẩn giao tiếp truyền thông nối tiếp dùng để truyền và nhận dữ liệu giữa hai thiết bị điện tử như vi điều khiển, máy tính, PLC, cảm biến, module truyền thông hoặc các thiết bị đo lường điều khiển.

UART được gọi là giao tiếp **nối tiếp** vì dữ liệu được truyền lần lượt từng bit theo thời gian trên đường dây tín hiệu. UART được gọi là **không đồng bộ** vì bên truyền và bên nhận không dùng chung một dây xung clock riêng. Thay vào đó, hai thiết bị phải được cấu hình cùng tốc độ truyền dữ liệu, gọi là **baud rate**.

## 2. Các tín hiệu cơ bản của UART

Một giao tiếp UART cơ bản thường sử dụng ba dây:

| Ký hiệu | Ý nghĩa  | Chức năng                    |
| ------- | -------- | ---------------------------- |
| TX      | Transmit | Chân truyền dữ liệu          |
| RX      | Receive  | Chân nhận dữ liệu            |
| GND     | Ground   | Mass chung giữa hai thiết bị |

Khi kết nối UART giữa hai thiết bị, phải nối chéo chân TX và RX:

```text
Thiết bị A TX  -------->  Thiết bị B RX
Thiết bị A RX  <--------  Thiết bị B TX
Thiết bị A GND ---------  Thiết bị B GND
```

Hai thiết bị cần nối chung GND để có cùng mức tham chiếu điện áp. Nếu không nối chung GND, dữ liệu truyền nhận có thể bị sai hoặc không giao tiếp được.

## 3. Nguyên lý hoạt động của UART

UART truyền dữ liệu theo từng khung truyền gọi là **frame**. Khi chưa truyền dữ liệu, đường truyền UART thường ở trạng thái mức logic 1. Khi bắt đầu truyền, bên gửi tạo một **start bit** ở mức logic 0 để báo cho bên nhận biết rằng dữ liệu sắp được gửi.

Sau start bit là các bit dữ liệu. Thông thường UART truyền 8 bit dữ liệu, tương ứng với 1 byte. Sau phần dữ liệu có thể có **parity bit**, tức là bit kiểm tra chẵn lẻ để phát hiện lỗi đơn giản. Cuối cùng là **stop bit** để báo kết thúc một khung truyền.

Cấu trúc một frame UART cơ bản:

```text
Start bit | Data bits | Parity bit | Stop bit
```

Trong đó:

* **Start bit**: báo hiệu bắt đầu truyền dữ liệu.
* **Data bits**: chứa dữ liệu cần truyền, thường là 8 bit.
* **Parity bit**: bit kiểm tra lỗi chẵn/lẻ, có thể có hoặc không.
* **Stop bit**: báo hiệu kết thúc khung truyền.

Ví dụ cấu hình UART phổ biến:

```text
9600, 8N1
```

Ý nghĩa:

* **9600**: tốc độ truyền là 9600 bit/giây.
* **8**: có 8 bit dữ liệu.
* **N**: không dùng parity.
* **1**: có 1 stop bit.

## 4. Baud rate trong UART

**Baud rate** là tốc độ truyền dữ liệu của UART, thường tính bằng bit trên giây.

Một số baud rate thường dùng:

| Baud rate      | Ứng dụng                         |
| -------------- | -------------------------------- |
| 9600           | Truyền dữ liệu cơ bản, ổn định   |
| 19200          | Truyền dữ liệu tốc độ trung bình |
| 57600          | Giao tiếp nhanh hơn              |
| 115200         | Debug, giao tiếp với máy tính    |
| 230400 trở lên | Truyền dữ liệu tốc độ cao hơn    |

Hai thiết bị giao tiếp UART bắt buộc phải có cùng baud rate. Nếu một bên đặt 9600 baud còn bên kia đặt 115200 baud thì dữ liệu nhận được sẽ bị sai hoặc hiển thị ký tự lạ.

## 5. Chế độ truyền dữ liệu của UART

UART có thể truyền dữ liệu theo các chế độ sau:

| Chế độ      | Đặc điểm                               |
| ----------- | -------------------------------------- |
| Simplex     | Chỉ truyền dữ liệu một chiều           |
| Half-duplex | Truyền hai chiều nhưng không đồng thời |
| Full-duplex | Truyền và nhận đồng thời               |

Trong thực tế, UART thường hoạt động ở chế độ **full-duplex** vì có hai đường riêng biệt là TX và RX. Một thiết bị có thể vừa gửi dữ liệu qua TX, vừa nhận dữ liệu qua RX.

## 6. Mức điện áp UART

UART có thể hoạt động với nhiều mức điện áp khác nhau tùy thiết bị phần cứng.

Một số mức điện áp thường gặp:

| Loại          | Đặc điểm                                                          |
| ------------- | ----------------------------------------------------------------- |
| UART TTL 5V   | Dùng mức logic 0V và 5V                                           |
| UART TTL 3.3V | Dùng mức logic 0V và 3.3V                                         |
| RS232         | Dùng mức điện áp khác TTL, thường dùng trong thiết bị công nghiệp |
| RS485         | Truyền tín hiệu vi sai, phù hợp truyền xa và chống nhiễu tốt hơn  |

Khi kết nối thiết bị 5V với thiết bị 3.3V cần chú ý mức điện áp logic. Nếu đưa tín hiệu 5V trực tiếp vào chân RX của thiết bị chỉ chịu được 3.3V thì có thể làm hỏng phần cứng. Khi cần thiết, phải dùng mạch chuyển mức logic.

## 7. Ưu điểm của UART

UART có các ưu điểm chính sau:

* Cấu trúc đơn giản, dễ hiểu và dễ sử dụng.
* Chỉ cần ít dây kết nối.
* Không cần dây clock riêng.
* Có sẵn trên hầu hết vi điều khiển như Arduino, STM32, ESP32, PIC, AVR.
* Dễ giao tiếp với máy tính thông qua mạch USB-to-UART.
* Rất phù hợp để debug chương trình và truyền dữ liệu trạng thái.
* Được nhiều module ngoại vi hỗ trợ như GPS, Bluetooth, GSM, WiFi, cảm biến thông minh.

## 8. Nhược điểm của UART

UART cũng có một số hạn chế:

* Thường chỉ phù hợp để giao tiếp trực tiếp giữa hai thiết bị.
* Không có cơ chế định địa chỉ thiết bị như I2C.
* Không phù hợp khi cần kết nối nhiều thiết bị trên cùng một bus.
* Dễ bị lỗi nếu cấu hình sai baud rate, parity hoặc stop bit.
* Khoảng cách truyền ngắn nếu dùng UART TTL.
* Dễ bị nhiễu khi dây dài hoặc môi trường làm việc có nhiều nhiễu điện từ.
* Chuẩn UART cơ bản không có cơ chế kiểm tra và sửa lỗi mạnh.

## 9. Ứng dụng của UART

UART được sử dụng rộng rãi trong các hệ thống đo lường, điều khiển và máy tính nhúng.

Một số ứng dụng phổ biến:

* Giao tiếp giữa vi điều khiển và máy tính để truyền dữ liệu đo hoặc debug.
* Giao tiếp giữa Arduino, STM32, ESP32 với các module cảm biến.
* Giao tiếp với module GPS để nhận dữ liệu vị trí.
* Giao tiếp với module Bluetooth như HC-05, HC-06.
* Giao tiếp với module GSM, GPRS, 4G thông qua tập lệnh AT.
* Giao tiếp giữa Raspberry Pi hoặc Jetson với vi điều khiển trong hệ thống robot.
* Truyền lệnh điều khiển và dữ liệu trạng thái trong hệ thống điều khiển.
* Kết nối PLC, HMI hoặc thiết bị công nghiệp thông qua RS232 hoặc RS485.

Ví dụ trong hệ thống robot:

```text
Raspberry Pi / Jetson
        |
       UART
        |
STM32 / ESP32 / Arduino
        |
Cảm biến, động cơ, gripper, encoder
```

Trong sơ đồ này, Raspberry Pi hoặc Jetson xử lý thuật toán cấp cao, còn STM32 hoặc ESP32 điều khiển phần cứng thời gian thực. UART được dùng để truyền lệnh điều khiển và phản hồi trạng thái giữa hai tầng xử lý.

## 10. Các lỗi thường gặp khi dùng UART

Một số lỗi thường gặp:

| Lỗi                     | Nguyên nhân thường gặp                    | Cách khắc phục                               |
| ----------------------- | ----------------------------------------- | -------------------------------------------- |
| Không nhận được dữ liệu | Nối sai TX/RX                             | Kiểm tra và nối TX với RX                    |
| Dữ liệu bị ký tự lạ     | Sai baud rate                             | Đặt cùng baud rate cho hai thiết bị          |
| Không giao tiếp được    | Chưa nối chung GND                        | Nối GND chung                                |
| Hỏng chân vi điều khiển | Sai mức điện áp 5V/3.3V                   | Dùng mạch chuyển mức logic                   |
| Mất dữ liệu             | Bộ đệm nhận bị tràn hoặc đọc dữ liệu chậm | Dùng interrupt, DMA hoặc giảm tốc độ truyền  |
| Dữ liệu bị nhiễu        | Dây dài hoặc môi trường nhiều nhiễu       | Rút ngắn dây, giảm baud rate hoặc dùng RS485 |

## 11. Kết luận

UART là chuẩn giao tiếp truyền thông nối tiếp không đồng bộ, dùng để truyền và nhận dữ liệu giữa hai thiết bị. UART sử dụng các chân TX, RX và GND. Dữ liệu được truyền theo từng frame gồm start bit, data bits, parity bit và stop bit.

UART có ưu điểm là đơn giản, dễ sử dụng, dễ lập trình và rất phổ biến trong vi điều khiển, cảm biến, máy tính nhúng, PLC và robot. Tuy nhiên, khi sử dụng UART cần chú ý nối đúng TX/RX, nối chung GND, cấu hình đúng baud rate, kiểm tra mức điện áp logic và hạn chế dây truyền quá dài.

Trong kỹ thuật đo lường và điều khiển bằng máy tính, UART là một chuẩn giao tiếp quan trọng vì giúp các thiết bị như vi điều khiển, cảm biến, module truyền thông và máy tính trao đổi dữ liệu một cách đơn giản và hiệu quả.
