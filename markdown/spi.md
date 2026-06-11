# Câu 1: Trình bày những khái niệm, lý thuyết cơ bản và ứng dụng về chuẩn giao tiếp truyền thông nối tiếp SPI

## 1. Khái niệm SPI

SPI là viết tắt của **Serial Peripheral Interface**, nghĩa là **giao tiếp nối tiếp với thiết bị ngoại vi**.

SPI là một chuẩn giao tiếp truyền thông nối tiếp đồng bộ, thường dùng để truyền dữ liệu giữa vi điều khiển với các thiết bị ngoại vi như cảm biến, màn hình, thẻ nhớ, bộ nhớ Flash, ADC, DAC hoặc module truyền thông.

SPI được gọi là giao tiếp **nối tiếp** vì dữ liệu được truyền lần lượt từng bit theo thời gian. SPI được gọi là giao tiếp **đồng bộ** vì quá trình truyền dữ liệu được đồng bộ bằng một đường xung clock riêng do thiết bị điều khiển tạo ra.

SPI thường có tốc độ truyền cao hơn UART và I2C, phù hợp với các thiết bị cần truyền dữ liệu nhanh.

## 2. Các tín hiệu cơ bản của SPI

Một giao tiếp SPI cơ bản thường sử dụng bốn đường tín hiệu chính:

| Ký hiệu | Tên đầy đủ                 | Chức năng                         |
| ------- | -------------------------- | --------------------------------- |
| SCLK    | Serial Clock               | Xung clock đồng bộ dữ liệu        |
| MOSI    | Master Out Slave In        | Dữ liệu từ Master gửi đến Slave   |
| MISO    | Master In Slave Out        | Dữ liệu từ Slave gửi về Master    |
| SS/CS   | Slave Select / Chip Select | Chọn thiết bị Slave cần giao tiếp |

Ngoài các dây tín hiệu trên, các thiết bị phải nối chung GND để có cùng mức tham chiếu điện áp.

Sơ đồ kết nối SPI cơ bản:

```text id="s4gk21"
Master                         Slave

SCLK  ---------------------->  SCLK
MOSI  ---------------------->  MOSI
MISO  <----------------------  MISO
CS    ---------------------->  CS
GND   -----------------------  GND
```

Trong đó, **Master** thường là vi điều khiển như Arduino, STM32, ESP32 hoặc Raspberry Pi. **Slave** là thiết bị ngoại vi như cảm biến, màn hình, bộ nhớ hoặc module truyền thông.

## 3. Mô hình Master và Slave

SPI hoạt động theo mô hình **Master - Slave**.

* **Master** là thiết bị điều khiển quá trình truyền thông, tạo xung clock trên đường SCLK và chọn thiết bị Slave thông qua chân CS.
* **Slave** là thiết bị ngoại vi được Master chọn để truyền hoặc nhận dữ liệu.

Một Master có thể giao tiếp với nhiều Slave. Tuy nhiên, mỗi Slave thường cần một chân CS riêng.

Ví dụ:

```text id="ka5cng"
STM32 / ESP32 / Arduino = Master
Màn hình TFT, thẻ SD, cảm biến, Flash = Slave
```

Khi Master muốn giao tiếp với Slave nào, Master sẽ kéo chân CS của Slave đó xuống mức kích hoạt, thường là mức logic 0.

## 4. Nguyên lý hoạt động của SPI

SPI là giao tiếp đồng bộ nên Master tạo xung clock trên đường SCLK. Mỗi nhịp clock sẽ đồng bộ việc truyền và nhận một bit dữ liệu.

SPI có hai đường dữ liệu riêng:

* MOSI dùng để Master gửi dữ liệu đến Slave.
* MISO dùng để Slave gửi dữ liệu về Master.

Nhờ có hai đường dữ liệu riêng, SPI có thể truyền và nhận dữ liệu đồng thời. Đây gọi là truyền **full-duplex**, nghĩa là song công hoàn toàn.

Một quá trình truyền SPI cơ bản gồm các bước:

1. Master chọn Slave bằng cách kích hoạt chân CS.
2. Master tạo xung clock trên đường SCLK.
3. Master gửi dữ liệu qua MOSI.
4. Slave đồng thời gửi dữ liệu về Master qua MISO nếu cần.
5. Sau khi truyền xong, Master ngừng clock và bỏ chọn Slave bằng chân CS.

Dạng truyền cơ bản:

```text id="8mrr6x"
CS active | Clock SCLK | Data on MOSI/MISO | CS inactive
```

SPI không dùng địa chỉ thiết bị như I2C. Việc chọn thiết bị cần giao tiếp được thực hiện bằng chân CS.

## 5. Các chế độ SPI

SPI có nhiều chế độ hoạt động dựa trên hai thông số:

| Thông số | Ý nghĩa                                          |
| -------- | ------------------------------------------------ |
| CPOL     | Clock Polarity - mức nghỉ của xung clock         |
| CPHA     | Clock Phase - cạnh clock dùng để lấy mẫu dữ liệu |

Từ CPOL và CPHA tạo ra 4 mode SPI:

| Mode   | CPOL | CPHA |
| ------ | ---- | ---- |
| Mode 0 | 0    | 0    |
| Mode 1 | 0    | 1    |
| Mode 2 | 1    | 0    |
| Mode 3 | 1    | 1    |

Khi giao tiếp SPI, Master và Slave phải được cấu hình cùng mode. Nếu sai mode, dữ liệu truyền nhận có thể bị sai hoặc không đọc được.

Mode 0 là chế độ thường gặp nhất, nhưng cần kiểm tra datasheet của từng thiết bị để chọn đúng mode.

## 6. Tốc độ truyền của SPI

SPI có tốc độ truyền cao, thường nhanh hơn I2C và UART.

Tốc độ SPI phụ thuộc vào:

* Vi điều khiển sử dụng.
* Thiết bị Slave.
* Chiều dài dây.
* Chất lượng mạch in.
* Nhiễu điện từ.
* Cấu hình clock SPI.

Một số tốc độ thường gặp:

| Tốc độ         | Ứng dụng                           |
| -------------- | ---------------------------------- |
| 1 MHz          | Giao tiếp cảm biến cơ bản          |
| 4 MHz          | Màn hình, bộ nhớ tốc độ trung bình |
| 8 MHz          | Thẻ nhớ, màn hình TFT              |
| 10 MHz trở lên | Bộ nhớ Flash, ADC tốc độ cao       |

Trong thực tế, nếu dây dài hoặc tín hiệu không ổn định, nên giảm tốc độ SPI để tránh lỗi dữ liệu.

## 7. Chế độ truyền dữ liệu của SPI

SPI thường hỗ trợ truyền **full-duplex**, tức là Master và Slave có thể gửi dữ liệu cho nhau cùng lúc.

Ngoài ra, trong nhiều ứng dụng thực tế, SPI cũng có thể dùng như:

* **Half-duplex**: chỉ truyền một chiều tại một thời điểm.
* **Simplex**: chỉ truyền một chiều, ví dụ Master chỉ gửi dữ liệu đến màn hình.

Tuy nhiên, về bản chất chuẩn SPI đầy đủ có hai đường MOSI và MISO riêng nên có khả năng truyền song công hoàn toàn.

## 8. Ưu điểm của SPI

SPI có các ưu điểm chính sau:

* Tốc độ truyền dữ liệu cao.
* Giao tiếp đồng bộ nên truyền dữ liệu ổn định ở tốc độ cao.
* Có thể truyền và nhận dữ liệu đồng thời.
* Cấu trúc giao thức đơn giản.
* Không cần địa chỉ thiết bị.
* Phù hợp với các thiết bị cần truyền dữ liệu nhanh như màn hình, thẻ nhớ, bộ nhớ Flash, ADC tốc độ cao.
* Được hỗ trợ rộng rãi trên Arduino, STM32, ESP32, Raspberry Pi và nhiều vi điều khiển khác.

## 9. Nhược điểm của SPI

SPI cũng có một số hạn chế:

* Cần nhiều dây hơn I2C và UART.
* Nếu có nhiều thiết bị Slave, mỗi thiết bị thường cần một chân CS riêng.
* Không có cơ chế địa chỉ thiết bị như I2C.
* Không có cơ chế xác nhận ACK giống I2C.
* Không phù hợp với khoảng cách truyền xa.
* Dễ bị nhiễu ở tốc độ cao nếu dây dài hoặc thiết kế mạch không tốt.
* Cần cấu hình đúng SPI mode, nếu sai CPOL/CPHA sẽ truyền nhận sai dữ liệu.

## 10. Ứng dụng của SPI

SPI được sử dụng phổ biến trong các hệ thống đo lường, điều khiển, vi điều khiển và máy tính nhúng.

Một số ứng dụng phổ biến:

* Giao tiếp với màn hình TFT, LCD, OLED tốc độ cao.
* Giao tiếp với thẻ nhớ SD.
* Giao tiếp với bộ nhớ Flash ngoài.
* Giao tiếp với ADC tốc độ cao để đọc tín hiệu analog.
* Giao tiếp với DAC để xuất tín hiệu analog.
* Giao tiếp với cảm biến tốc độ cao.
* Giao tiếp với module RF như nRF24L01.
* Giao tiếp giữa vi điều khiển và các IC ngoại vi cần tốc độ cao.

Ví dụ trong hệ thống robot:

```text id="gnlqjy"
STM32 / ESP32 / Raspberry Pi
        |
       SPI
        |
Màn hình TFT / ADC tốc độ cao / Flash / Module RF
```

Trong robot, SPI thường dùng cho các thiết bị cần tốc độ truyền nhanh hoặc dữ liệu nhiều hơn I2C, ví dụ màn hình hiển thị, bộ nhớ ngoài, ADC đọc tín hiệu nhanh, module truyền thông RF hoặc cảm biến yêu cầu tốc độ cao.

## 11. Các lỗi thường gặp khi dùng SPI

Một số lỗi thường gặp:

| Lỗi                                | Nguyên nhân thường gặp                        | Cách khắc phục                            |
| ---------------------------------- | --------------------------------------------- | ----------------------------------------- |
| Không nhận được dữ liệu            | Nối sai MOSI/MISO hoặc CS                     | Kiểm tra lại sơ đồ nối dây                |
| Dữ liệu đọc sai                    | Sai SPI mode CPOL/CPHA                        | Kiểm tra datasheet và cấu hình đúng mode  |
| Slave không phản hồi               | Chân CS chưa được kích hoạt đúng              | Kiểm tra mức logic chân CS                |
| Dữ liệu bị nhiễu                   | Tốc độ quá cao hoặc dây dài                   | Giảm tốc độ SPI, rút ngắn dây             |
| Chỉ gửi được nhưng không nhận được | Chưa nối MISO hoặc cấu hình sai chiều dữ liệu | Kiểm tra đường MISO                       |
| Nhiều Slave bị lỗi                 | Xung đột CS hoặc nhiều Slave cùng được chọn   | Đảm bảo mỗi lần chỉ kích hoạt một CS      |
| Giao tiếp không ổn định            | Không nối chung GND hoặc sai mức điện áp      | Nối GND chung, dùng level shifter nếu cần |

## 12. So sánh nhanh SPI với UART và I2C

| Tiêu chí         | SPI                                | I2C                         | UART                       |
| ---------------- | ---------------------------------- | --------------------------- | -------------------------- |
| Kiểu truyền      | Nối tiếp đồng bộ                   | Nối tiếp đồng bộ            | Nối tiếp không đồng bộ     |
| Dây chính        | SCLK, MOSI, MISO, CS               | SDA, SCL                    | TX, RX                     |
| Clock            | Có SCLK                            | Có SCL                      | Không có clock riêng       |
| Tốc độ           | Cao                                | Trung bình                  | Trung bình                 |
| Địa chỉ thiết bị | Không, dùng CS                     | Có địa chỉ                  | Không                      |
| Nhiều thiết bị   | Có, nhưng tốn chân CS              | Có, tiết kiệm dây           | Không thuận tiện           |
| Song công        | Full-duplex                        | Thường half-duplex          | Full-duplex                |
| Ứng dụng chính   | Màn hình, thẻ SD, Flash, ADC nhanh | Cảm biến, RTC, OLED, EEPROM | Debug, GPS, Bluetooth, GSM |

## 13. Kết luận

SPI là chuẩn giao tiếp truyền thông nối tiếp đồng bộ, sử dụng các đường SCLK, MOSI, MISO và CS để truyền dữ liệu giữa vi điều khiển và thiết bị ngoại vi. SPI hoạt động theo mô hình Master - Slave, trong đó Master tạo xung clock và chọn Slave bằng chân CS.

Ưu điểm lớn nhất của SPI là tốc độ truyền cao, giao thức đơn giản và có khả năng truyền nhận đồng thời. Vì vậy, SPI rất phù hợp với các thiết bị cần truyền dữ liệu nhanh như màn hình, thẻ nhớ, bộ nhớ Flash, ADC tốc độ cao và module RF.

Tuy nhiên, SPI cần nhiều dây hơn I2C, không có cơ chế địa chỉ thiết bị, không có ACK và không phù hợp truyền xa. Khi sử dụng SPI cần chú ý nối đúng MOSI/MISO, cấu hình đúng mode CPOL/CPHA, chọn đúng chân CS, nối chung GND và giảm tốc độ nếu tín hiệu bị nhiễu.

Trong kỹ thuật đo lường, điều khiển và robot, SPI là một chuẩn giao tiếp quan trọng, đặc biệt phù hợp cho các thiết bị ngoại vi cần tốc độ cao và độ trễ thấp.
