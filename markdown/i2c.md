# Câu 1: Trình bày những khái niệm, lý thuyết cơ bản và ứng dụng về chuẩn giao tiếp truyền thông nối tiếp I2C

## 1. Khái niệm I2C

I2C là viết tắt của **Inter-Integrated Circuit**, nghĩa là **chuẩn giao tiếp giữa các vi mạch tích hợp**.

I2C là một chuẩn giao tiếp truyền thông nối tiếp dùng để truyền và nhận dữ liệu giữa vi điều khiển với các thiết bị ngoại vi như cảm biến, IC thời gian thực, màn hình LCD/OLED, EEPROM, ADC, DAC hoặc các module mở rộng.

I2C là giao tiếp **nối tiếp** vì dữ liệu được truyền từng bit theo thời gian. I2C là giao tiếp **đồng bộ** vì có một đường xung clock riêng để đồng bộ quá trình truyền nhận dữ liệu giữa các thiết bị.

Điểm đặc biệt của I2C là chỉ cần **hai dây tín hiệu chính** nhưng có thể kết nối được nhiều thiết bị trên cùng một bus, nhờ mỗi thiết bị có một địa chỉ riêng.

## 2. Các tín hiệu cơ bản của I2C

I2C sử dụng hai dây tín hiệu chính:

| Ký hiệu | Tên đầy đủ   | Chức năng                     |
| ------- | ------------ | ----------------------------- |
| SDA     | Serial Data  | Đường truyền dữ liệu nối tiếp |
| SCL     | Serial Clock | Đường xung clock đồng bộ      |

Ngoài hai dây SDA và SCL, các thiết bị phải nối chung GND để có cùng mức tham chiếu điện áp.

Sơ đồ kết nối cơ bản:

```text id="f7s2am"
Vi điều khiển / Master
        |
        |---- SDA ---- Cảm biến 1
        |---- SCL ---- Cảm biến 1
        |
        |---- SDA ---- Cảm biến 2
        |---- SCL ---- Cảm biến 2
        |
       GND ----------- GND chung
```

Trong I2C, các thiết bị được nối song song trên cùng hai đường SDA và SCL.

## 3. Mô hình Master và Slave

I2C hoạt động theo mô hình **Master - Slave**.

* **Master** là thiết bị điều khiển bus, tạo xung clock trên đường SCL và khởi tạo quá trình truyền dữ liệu.
* **Slave** là thiết bị ngoại vi nhận lệnh hoặc gửi dữ liệu theo yêu cầu của Master.

Ví dụ:

```text id="l64bvd"
Arduino / STM32 / ESP32  = Master
Cảm biến nhiệt độ, OLED, EEPROM = Slave
```

Trong nhiều tài liệu mới, Master còn được gọi là **Controller**, Slave được gọi là **Peripheral**. Tuy nhiên, bản chất hoạt động vẫn là một thiết bị điều khiển truyền thông và các thiết bị còn lại phản hồi theo địa chỉ.

## 4. Nguyên lý hoạt động của I2C

Trên bus I2C, Master sẽ tạo xung clock trên đường SCL. Dữ liệu được truyền trên đường SDA và được đồng bộ theo xung clock này.

Một quá trình truyền dữ liệu I2C thường gồm các bước:

1. Master tạo điều kiện bắt đầu gọi là **Start condition**.
2. Master gửi địa chỉ của thiết bị Slave cần giao tiếp.
3. Master gửi bit chọn chiều truyền: đọc hoặc ghi.
4. Slave có địa chỉ phù hợp phản hồi bằng tín hiệu **ACK**.
5. Master và Slave truyền dữ liệu theo từng byte.
6. Sau mỗi byte, bên nhận gửi ACK để xác nhận đã nhận dữ liệu.
7. Master tạo điều kiện kết thúc gọi là **Stop condition**.

Dạng truyền cơ bản:

```text id="xztjva"
START | Address | R/W | ACK | Data | ACK | STOP
```

Trong đó:

* **START**: tín hiệu bắt đầu truyền.
* **Address**: địa chỉ thiết bị cần giao tiếp.
* **R/W**: bit xác định đọc hoặc ghi dữ liệu.
* **ACK**: tín hiệu xác nhận đã nhận đúng dữ liệu.
* **Data**: dữ liệu truyền đi.
* **STOP**: tín hiệu kết thúc truyền.

## 5. Địa chỉ thiết bị trong I2C

Mỗi thiết bị I2C có một địa chỉ riêng để Master nhận biết và giao tiếp đúng thiết bị.

I2C thường dùng địa chỉ:

| Loại địa chỉ | Đặc điểm                                    |
| ------------ | ------------------------------------------- |
| 7-bit        | Phổ biến nhất                               |
| 10-bit       | Ít dùng hơn, dùng khi cần nhiều địa chỉ hơn |

Ví dụ:

```text id="wpxxcg"
Cảm biến nhiệt độ: địa chỉ 0x48
Màn hình OLED: địa chỉ 0x3C
RTC DS3231: địa chỉ 0x68
```

Nhờ có địa chỉ, nhiều thiết bị có thể dùng chung hai đường SDA và SCL. Tuy nhiên, nếu hai thiết bị có cùng địa chỉ mà không đổi được địa chỉ, chúng có thể bị xung đột và không giao tiếp được đúng.

## 6. Điện trở kéo lên trong I2C

I2C cần có điện trở kéo lên, gọi là **pull-up resistor**, trên hai đường SDA và SCL.

Lý do là các chân I2C thường hoạt động theo kiểu **open-drain** hoặc **open-collector**, nghĩa là thiết bị chỉ kéo đường tín hiệu xuống mức 0, còn mức 1 được tạo nhờ điện trở kéo lên nguồn.

Sơ đồ đơn giản:

```text id="bvl0xs"
VCC
 |
R pull-up
 |
SDA / SCL
 |
Thiết bị I2C
```

Giá trị điện trở kéo lên thường dùng:

| Tốc độ / hệ thống           | Giá trị thường gặp               |
| --------------------------- | -------------------------------- |
| Tốc độ thấp, dây ngắn       | 4.7kΩ                            |
| Tốc độ cao hơn              | 2.2kΩ                            |
| Bus dài hoặc nhiều thiết bị | Cần tính toán theo điện dung bus |

Nếu điện trở pull-up quá lớn, tín hiệu lên mức 1 chậm, dễ lỗi ở tốc độ cao. Nếu điện trở quá nhỏ, dòng kéo xuống tăng, gây tốn dòng và có thể ảnh hưởng thiết bị.

## 7. Tốc độ truyền của I2C

Một số tốc độ truyền phổ biến của I2C:

| Chế độ          | Tốc độ     |
| --------------- | ---------- |
| Standard Mode   | 100 kbit/s |
| Fast Mode       | 400 kbit/s |
| Fast Mode Plus  | 1 Mbit/s   |
| High Speed Mode | 3.4 Mbit/s |

Trong thực tế với Arduino, STM32, ESP32 và các cảm biến thông dụng, tốc độ 100 kHz hoặc 400 kHz được dùng nhiều nhất.

Tốc độ càng cao thì yêu cầu dây ngắn hơn, điện trở pull-up phù hợp hơn và tín hiệu phải sạch hơn.

## 8. Ưu điểm của I2C

I2C có các ưu điểm chính sau:

* Chỉ cần hai dây tín hiệu SDA và SCL.
* Có thể kết nối nhiều thiết bị trên cùng một bus.
* Có cơ chế địa chỉ thiết bị.
* Phù hợp với cảm biến và IC ngoại vi tốc độ thấp hoặc trung bình.
* Dễ mở rộng hệ thống.
* Được hỗ trợ rộng rãi trên Arduino, STM32, ESP32, Raspberry Pi và nhiều vi điều khiển khác.
* Phù hợp cho các mạch nhỏ gọn vì tiết kiệm chân GPIO.

## 9. Nhược điểm của I2C

I2C cũng có một số hạn chế:

* Tốc độ không cao bằng SPI.
* Không phù hợp truyền dữ liệu dung lượng lớn.
* Dễ bị ảnh hưởng bởi điện dung đường dây khi dây dài.
* Cần điện trở kéo lên phù hợp.
* Nếu một thiết bị lỗi kéo SDA hoặc SCL xuống thấp, toàn bộ bus có thể bị treo.
* Có thể bị xung đột nếu nhiều thiết bị có cùng địa chỉ.
* Không phù hợp với môi trường nhiễu mạnh nếu không thiết kế dây và mạch cẩn thận.

## 10. Ứng dụng của I2C

I2C được sử dụng phổ biến trong các hệ thống đo lường, điều khiển và máy tính nhúng.

Một số ứng dụng phổ biến:

* Giao tiếp với cảm biến nhiệt độ, độ ẩm, áp suất.
* Giao tiếp với cảm biến gia tốc, con quay hồi chuyển, IMU.
* Giao tiếp với màn hình OLED hoặc LCD.
* Giao tiếp với IC thời gian thực RTC.
* Giao tiếp với EEPROM để lưu dữ liệu.
* Giao tiếp với ADC/DAC mở rộng.
* Mở rộng số chân vào/ra bằng IC I/O expander.
* Kết nối nhiều cảm biến với vi điều khiển trong robot hoặc IoT.

Ví dụ trong hệ thống robot:

```text id="uiqr40"
STM32 / ESP32 / Arduino
        |
       I2C
        |
IMU + Cảm biến khoảng cách + OLED + RTC
```

Trong robot, I2C thường dùng để đọc các cảm biến tốc độ thấp hoặc trung bình như IMU, cảm biến nhiệt độ, cảm biến áp suất, cảm biến khoảng cách gần và màn hình hiển thị nhỏ.

## 11. Các lỗi thường gặp khi dùng I2C

Một số lỗi thường gặp:

| Lỗi                             | Nguyên nhân thường gặp                                | Cách khắc phục                                                      |
| ------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------- |
| Không tìm thấy địa chỉ thiết bị | Sai địa chỉ, nối sai SDA/SCL, thiết bị chưa cấp nguồn | Quét địa chỉ bằng I2C scanner, kiểm tra dây                         |
| Bus bị treo                     | Một thiết bị kéo SDA hoặc SCL xuống thấp              | Reset thiết bị, kiểm tra module lỗi, tạo xung SCL để giải phóng bus |
| Dữ liệu đọc sai                 | Nhiễu tín hiệu, dây dài, pull-up không phù hợp        | Giảm tốc độ, rút ngắn dây, đổi điện trở pull-up                     |
| Xung đột địa chỉ                | Hai thiết bị có cùng địa chỉ                          | Đổi địa chỉ bằng chân cấu hình hoặc dùng I2C multiplexer            |
| Không giao tiếp được            | Không nối chung GND hoặc sai mức điện áp              | Nối GND chung, dùng level shifter nếu cần                           |
| Lỗi ở tốc độ cao                | Điện dung bus lớn, cạnh tín hiệu lên chậm             | Giảm tốc độ hoặc giảm giá trị điện trở pull-up phù hợp              |

## 12. So sánh nhanh I2C với UART và SPI

| Tiêu chí         | I2C                   | UART                       | SPI                              |
| ---------------- | --------------------- | -------------------------- | -------------------------------- |
| Kiểu truyền      | Nối tiếp đồng bộ      | Nối tiếp không đồng bộ     | Nối tiếp đồng bộ                 |
| Số dây chính     | 2 dây                 | 2 dây TX/RX                | 4 dây trở lên                    |
| Clock            | Có SCL                | Không có clock riêng       | Có SCLK                          |
| Địa chỉ thiết bị | Có                    | Không                      | Thường dùng chân CS              |
| Nhiều thiết bị   | Tốt                   | Không thuận tiện           | Tốt nhưng tốn chân CS            |
| Tốc độ           | Trung bình            | Trung bình                 | Cao                              |
| Ứng dụng chính   | Cảm biến, IC ngoại vi | Debug, module truyền thông | Màn hình, bộ nhớ, ADC tốc độ cao |

## 13. Kết luận

I2C là chuẩn giao tiếp truyền thông nối tiếp đồng bộ, sử dụng hai dây SDA và SCL để truyền dữ liệu giữa vi điều khiển và nhiều thiết bị ngoại vi. I2C hoạt động theo mô hình Master - Slave, trong đó Master tạo xung clock và gọi thiết bị Slave bằng địa chỉ riêng.

Ưu điểm của I2C là tiết kiệm dây, tiết kiệm chân vi điều khiển, hỗ trợ nhiều thiết bị trên cùng một bus và rất phù hợp với cảm biến, màn hình nhỏ, EEPROM, RTC và các IC ngoại vi. Tuy nhiên, I2C có tốc độ không quá cao, cần điện trở kéo lên, dễ bị ảnh hưởng khi dây dài và có thể bị lỗi nếu xung đột địa chỉ hoặc treo bus.

Trong kỹ thuật đo lường, điều khiển và robot, I2C là một chuẩn giao tiếp quan trọng, đặc biệt phù hợp để thu thập dữ liệu từ nhiều cảm biến và thiết bị ngoại vi trong cùng một hệ thống.
