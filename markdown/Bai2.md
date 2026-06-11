# Câu 2: Thiết kế và xây dựng hệ thống đo giám sát – điều khiển động cơ điện xoay chiều 3 pha

---

# I. Đề bài

Cho một tải là một động cơ điện xoay chiều 3 pha không đồng bộ roto lồng sóc có thông số kỹ thuật như bảng sau:

| Công suất P(kW) | Điện áp Uđm(V) | Dòng điện Iđm(A) | Cách đấu | Tần số f(Hz) | Tốc độ vòng/phút | Hiệu suất η% | Cosφ |
| --------------- | -------------- | ---------------- | -------- | ------------ | ---------------- | ------------ | ---- |
| 1160            | 6000/3500      | 132/228          | Y/Δ      | 50           | 1500             | 95           | 0,9  |

Thiết kế và xây dựng các hệ thống đo giám sát – điều khiển với các yêu cầu sau:

## 1. Động cơ làm việc ở chế độ đấu hình Y

## 2. Vẽ sơ đồ nguyên lý và thuyết minh thuật toán đo 2 đại lượng sau khi thu thập dữ liệu đo về PLC hoặc VĐK hoặc PC

### 2.1. Công suất tác dụng động cơ

### 2.2. Nhiệt độ làm việc trên Stato của động cơ

## 3. Tính chọn thiết bị và thông số kỹ thuật các thiết bị trên sơ đồ gồm

Thiết bị biến đổi đo lường, sensor, transducer, transmitter...

## 4. Xây dựng các đường đặc tính chuyển đổi và các phương trình cho các thiết bị biến đổi, sensor, transducer, transmitter, bộ đếm cổng vào AI của PLC, VĐK hoặc PC

## 5. Lập trình thu thập dữ liệu đo trên PLC hoặc VĐK hoặc PC

## 6. Vẽ lưu đồ thuật toán điều khiển để dừng sự làm việc động cơ cho các sự cố sau

### 6.1. Khi Pđộng cơ ≥ 130%.Pđm trong thời gian 2 phút

### 6.2. Khi nhiệt độ Stato thực tế ≥ 90°C trong thời gian 1 phút

---

# II. Bài làm

---

# Ý 1. Động cơ làm việc ở chế độ đấu hình Y

Theo đề bài, động cơ có thông số:

```text
Điện áp định mức: 6000/3500 V
Dòng điện định mức: 132/228 A
Cách đấu: Y/Δ
```

Đề yêu cầu động cơ làm việc ở chế độ đấu hình Y, vì vậy ta chọn thông số làm việc tương ứng với chế độ Y:

```text
Uđm = 6000 V
Iđm = 132 A
Pđm = 1160 kW
f = 50 Hz
n = 1500 vòng/phút
η = 95%
cosφ = 0,9
```

Đây là động cơ điện áp cao, công suất lớn, nên các tín hiệu điện áp và dòng điện không thể đưa trực tiếp vào PLC, vi điều khiển hoặc PC. Vì vậy hệ thống cần sử dụng các thiết bị biến đổi đo lường như biến dòng CT, biến điện áp PT, bộ chuyển đổi công suất và bộ chuyển đổi nhiệt độ.

---

# Ý 2. Vẽ sơ đồ nguyên lý và thuyết minh thuật toán đo

## 2.1. Sơ đồ nguyên lý tổng quát của hệ thống

```text
Nguồn 3 pha 6kV
      |
      |
Máy cắt / Contactor / Khởi động từ
      |
      |
Động cơ 3 pha không đồng bộ roto lồng sóc
      |
      |-------------------------------|
      |                               |
Đo công suất tác dụng            Đo nhiệt độ Stato
      |                               |
CT + PT/VT                       Cảm biến PT100
      |                               |
Power Transducer                 Temperature Transmitter
      |                               |
Tín hiệu 4–20mA                  Tín hiệu 4–20mA
      |                               |
      |---------- PLC / VĐK / PC ------|
                       |
                       |
              Xử lý, hiển thị, giám sát
                       |
                       |
              Tín hiệu điều khiển dừng
                       |
                       |
             Relay / Contactor / Máy cắt
                       |
                       |
                  Dừng động cơ
```

Hệ thống gồm hai nhánh đo chính:

```text
Nhánh 1: Đo công suất tác dụng của động cơ.
Nhánh 2: Đo nhiệt độ làm việc trên Stato động cơ.
```

PLC hoặc vi điều khiển sẽ đọc hai tín hiệu đo, quy đổi thành giá trị thực, sau đó so sánh với ngưỡng sự cố để quyết định có dừng động cơ hay không.

---

## 2.2. Thuyết minh thuật toán đo công suất tác dụng động cơ

Công suất tác dụng của động cơ 3 pha có thể xác định theo công thức:

```text
P = √3 × U × I × cosφ
```

Trong đó:

```text
P     : Công suất tác dụng, đơn vị W
U     : Điện áp dây, đơn vị V
I     : Dòng điện dây, đơn vị A
cosφ  : Hệ số công suất
```

Tuy nhiên, do động cơ làm việc ở điện áp 6000 V và dòng điện 132 A nên không thể đưa trực tiếp tín hiệu vào PLC. Vì vậy ta sử dụng:

```text
CT: Biến dòng, hạ dòng điện lớn xuống dòng tiêu chuẩn.
PT/VT: Biến điện áp, hạ điện áp cao xuống điện áp tiêu chuẩn.
Power Transducer: Bộ chuyển đổi công suất 3 pha.
PLC AI: Cổng vào analog của PLC.
```

Quy trình đo công suất:

```text
Bước 1: CT đo dòng điện của động cơ.
Bước 2: PT đo điện áp cấp cho động cơ.
Bước 3: Tín hiệu từ CT và PT đưa vào Power Transducer.
Bước 4: Power Transducer tính công suất tác dụng 3 pha.
Bước 5: Power Transducer xuất tín hiệu chuẩn 4–20mA.
Bước 6: PLC đọc tín hiệu 4–20mA.
Bước 7: PLC quy đổi tín hiệu analog thành công suất thực tế kW.
Bước 8: PLC hiển thị và giám sát công suất động cơ.
```

Sơ đồ đo công suất:

```text
Điện áp 3 pha 6kV + Dòng điện động cơ
              |
             CT + PT
              |
       Power Transducer
              |
        Tín hiệu 4–20mA
              |
        Analog Input PLC
              |
       Giá trị công suất P
```

---

## 2.3. Thuyết minh thuật toán đo nhiệt độ làm việc trên Stato

Nhiệt độ Stato được đo bằng cảm biến nhiệt độ gắn tại vùng Stato của động cơ. Trong bài này chọn cảm biến PT100.

PT100 là cảm biến nhiệt điện trở. Khi nhiệt độ thay đổi, điện trở của PT100 thay đổi theo. Tín hiệu điện trở này được đưa vào bộ chuyển đổi nhiệt độ để biến đổi thành tín hiệu chuẩn 4–20mA đưa về PLC.

Quy trình đo nhiệt độ:

```text
Bước 1: PT100 cảm nhận nhiệt độ tại Stato động cơ.
Bước 2: Điện trở của PT100 thay đổi theo nhiệt độ.
Bước 3: Temperature Transmitter nhận tín hiệu từ PT100.
Bước 4: Transmitter chuyển tín hiệu PT100 thành dòng 4–20mA.
Bước 5: PLC đọc tín hiệu 4–20mA qua cổng analog input.
Bước 6: PLC quy đổi tín hiệu analog thành nhiệt độ thực tế °C.
Bước 7: PLC hiển thị và giám sát nhiệt độ Stato.
```

Sơ đồ đo nhiệt độ Stato:

```text
Nhiệt độ Stato động cơ
              |
            PT100
              |
 Temperature Transmitter
              |
        Tín hiệu 4–20mA
              |
        Analog Input PLC
              |
      Giá trị nhiệt độ T
```

---

# Ý 3. Tính chọn thiết bị và thông số kỹ thuật các thiết bị trên sơ đồ

## 3.1. Chọn biến dòng CT

Dòng điện định mức của động cơ khi đấu Y là:

```text
Iđm = 132 A
```

Chọn biến dòng CT có dòng sơ cấp lớn hơn dòng định mức động cơ.

Chọn:

```text
CT = 150/5 A
```

Thông số:

| Thiết bị        | Thông số chọn                                       |
| --------------- | --------------------------------------------------- |
| Loại            | Biến dòng CT                                        |
| Dòng sơ cấp     | 150 A                                               |
| Dòng thứ cấp    | 5 A                                                 |
| Tỷ số biến dòng | 150/5 A                                             |
| Chức năng       | Hạ dòng điện động cơ về mức an toàn cho thiết bị đo |

Lý do chọn CT 150/5 A:

```text
Dòng định mức động cơ là 132 A.
Chọn CT 150/5 A để có dự phòng và phù hợp với dòng làm việc của động cơ.
```

---

## 3.2. Chọn biến điện áp PT/VT

Điện áp định mức của động cơ khi đấu Y là:

```text
Uđm = 6000 V
```

Chọn biến điện áp:

```text
PT = 6000/100 V
```

Thông số:

| Thiết bị        | Thông số chọn                                    |
| --------------- | ------------------------------------------------ |
| Loại            | Biến điện áp PT/VT                               |
| Điện áp sơ cấp  | 6000 V                                           |
| Điện áp thứ cấp | 100 V                                            |
| Tỷ số biến áp   | 6000/100 V                                       |
| Chức năng       | Hạ điện áp cao về mức tiêu chuẩn cho thiết bị đo |

---

## 3.3. Chọn bộ chuyển đổi công suất Power Transducer

Bộ chuyển đổi công suất nhận tín hiệu từ CT và PT, sau đó chuyển thành tín hiệu analog chuẩn đưa về PLC.

Chọn:

```text
Power Transducer 3 pha
Input: 3 pha 100 V, 5 A
Output: 4–20mA
Dải đo: 0–1600 kW
Nguồn nuôi: 24VDC hoặc 220VAC
```

Lý do chọn dải 0–1600 kW:

```text
Công suất định mức Pđm = 1160 kW.
Ngưỡng bảo vệ quá công suất là 130%Pđm = 1508 kW.
Dải 0–1600 kW bao phủ được cả vùng làm việc định mức và vùng sự cố.
```

---

## 3.4. Chọn cảm biến nhiệt độ PT100

Chọn cảm biến nhiệt độ:

```text
Loại: PT100
Kiểu đấu dây: 3 dây
Dải đo: 0–150°C
Vị trí lắp: Stato động cơ
```

Lý do chọn PT100:

```text
PT100 có độ chính xác cao, ổn định, phù hợp đo nhiệt độ động cơ trong công nghiệp.
```

---

## 3.5. Chọn bộ chuyển đổi nhiệt độ Temperature Transmitter

Chọn:

```text
Input: PT100 3 dây
Dải đo: 0–150°C
Output: 4–20mA
Nguồn nuôi: 24VDC
```

Lý do chọn dải đo 0–150°C:

```text
Ngưỡng bảo vệ theo đề bài là 90°C.
Dải 0–150°C đủ để đo, giám sát và bảo vệ nhiệt độ Stato.
```

---

## 3.6. Chọn PLC hoặc bộ điều khiển

PLC cần có:

```text
Ít nhất 2 ngõ vào analog AI nhận tín hiệu 4–20mA.
Ít nhất 1 ngõ ra số DO để điều khiển dừng động cơ.
Có bộ định thời Timer.
Có khả năng lập trình, xử lý và hiển thị dữ liệu.
```

Các kênh sử dụng:

```text
AI1: Đọc tín hiệu công suất từ Power Transducer.
AI2: Đọc tín hiệu nhiệt độ từ Temperature Transmitter.
DO1: Xuất tín hiệu dừng động cơ.
```

---

# Ý 4. Xây dựng đường đặc tính chuyển đổi và phương trình

## 4.1. Công thức tính ngưỡng công suất bảo vệ

Theo đề bài:

```text
Pđộng cơ ≥ 130% × Pđm trong thời gian 2 phút
```

Ta có:

```text
Pđm = 1160 kW
```

Ngưỡng bảo vệ công suất:

```text
Pngưỡng = 130% × Pđm
        = 1,3 × 1160
        = 1508 kW
```

Vậy:

```text
Nếu P ≥ 1508 kW liên tục trong 120 giây thì dừng động cơ.
```

---

## 4.2. Đặc tính chuyển đổi của bộ đo công suất

Chọn Power Transducer có dải đo:

```text
0–1600 kW tương ứng 4–20mA
```

Đường đặc tính:

| Dòng ngõ ra | Công suất |
| ----------- | --------- |
| 4mA         | 0 kW      |
| 8mA         | 400 kW    |
| 12mA        | 800 kW    |
| 16mA        | 1200 kW   |
| 19,08mA     | 1508 kW   |
| 20mA        | 1600 kW   |

Phương trình chuyển đổi:

```text
P = (Iout - 4) × 1600 / 16
```

Rút gọn:

```text
P = 100 × (Iout - 4)
```

Trong đó:

```text
P    : Công suất động cơ, đơn vị kW
Iout : Dòng tín hiệu từ Power Transducer, đơn vị mA
```

Tín hiệu dòng tương ứng với ngưỡng 1508 kW:

```text
Iout = P / 100 + 4
     = 1508 / 100 + 4
     = 19,08 mA
```

---

## 4.3. Đặc tính chuyển đổi của bộ đo nhiệt độ

Chọn dải đo nhiệt độ:

```text
0–150°C tương ứng 4–20mA
```

Đường đặc tính:

| Dòng ngõ ra | Nhiệt độ |
| ----------- | -------- |
| 4mA         | 0°C      |
| 8mA         | 37,5°C   |
| 12mA        | 75°C     |
| 13,6mA      | 90°C     |
| 16mA        | 112,5°C  |
| 20mA        | 150°C    |

Phương trình chuyển đổi:

```text
T = (Iout - 4) × 150 / 16
```

Rút gọn:

```text
T = 9,375 × (Iout - 4)
```

Trong đó:

```text
T    : Nhiệt độ Stato, đơn vị °C
Iout : Dòng tín hiệu từ Temperature Transmitter, đơn vị mA
```

Tín hiệu dòng tương ứng với ngưỡng 90°C:

```text
Iout = T / 9,375 + 4
     = 90 / 9,375 + 4
     = 13,6 mA
```

---

## 4.4. Phương trình quy đổi giá trị analog input của PLC

Giả sử module analog input của PLC có giá trị số hóa:

```text
0–27648 tương ứng với 4–20mA
```

Với kênh đo công suất:

```text
P = AI_P × 1600 / 27648
```

Trong đó:

```text
P    : Công suất thực tế, đơn vị kW
AI_P : Giá trị số đọc từ kênh analog đo công suất
```

Với kênh đo nhiệt độ:

```text
T = AI_T × 150 / 27648
```

Trong đó:

```text
T    : Nhiệt độ thực tế, đơn vị °C
AI_T : Giá trị số đọc từ kênh analog đo nhiệt độ
```

---

# Ý 5. Lập trình thu thập dữ liệu đo trên PLC hoặc VĐK hoặc PC

## 5.1. Các biến sử dụng trong chương trình

```text
AI_P      : Giá trị analog đo công suất
AI_T      : Giá trị analog đo nhiệt độ
P         : Công suất thực tế của động cơ, kW
T         : Nhiệt độ thực tế của Stato, °C
P_LIMIT   : Ngưỡng công suất bảo vệ
T_LIMIT   : Ngưỡng nhiệt độ bảo vệ
Timer_P   : Bộ định thời quá công suất
Timer_T   : Bộ định thời quá nhiệt
Trip      : Tín hiệu sự cố
Motor_Run : Tín hiệu cho phép động cơ chạy
```

Gán giá trị:

```text
P_LIMIT = 1508 kW
T_LIMIT = 90°C
Timer_P = 120 giây
Timer_T = 60 giây
```

---

## 5.2. Chương trình dạng giả mã

```text
Bắt đầu chương trình

Đọc AI_P từ ngõ vào analog đo công suất
Đọc AI_T từ ngõ vào analog đo nhiệt độ

Quy đổi giá trị đo:

P = AI_P × 1600 / 27648
T = AI_T × 150 / 27648

Hiển thị P và T lên màn hình giám sát

Kiểm tra quá công suất:

Nếu P >= 1508 thì
    Bắt đầu đếm Timer_P
Ngược lại
    Reset Timer_P

Nếu Timer_P >= 120 giây thì
    Trip = 1

Kiểm tra quá nhiệt:

Nếu T >= 90 thì
    Bắt đầu đếm Timer_T
Ngược lại
    Reset Timer_T

Nếu Timer_T >= 60 giây thì
    Trip = 1

Nếu Trip = 1 thì
    Ngắt tín hiệu Motor_Run
    Xuất tín hiệu DO để dừng động cơ
    Hiển thị cảnh báo sự cố

Lặp lại chương trình
```

---

# Ý 6. Vẽ lưu đồ thuật toán điều khiển dừng động cơ

## 6.1. Điều kiện dừng khi quá công suất

Theo đề bài:

```text
Pđộng cơ ≥ 130%.Pđm trong thời gian 2 phút
```

Đã tính được:

```text
130%.Pđm = 1508 kW
```

Vì vậy điều kiện dừng là:

```text
Nếu P ≥ 1508 kW liên tục trong 120 giây thì dừng động cơ.
```

---

## 6.2. Điều kiện dừng khi quá nhiệt Stato

Theo đề bài:

```text
Nhiệt độ Stato thực tế ≥ 90°C trong thời gian 1 phút
```

Vì vậy điều kiện dừng là:

```text
Nếu T ≥ 90°C liên tục trong 60 giây thì dừng động cơ.
```

---

## 6.3. Lưu đồ thuật toán dạng text

```text
                         BẮT ĐẦU
                            |
                            v
                 Đọc tín hiệu AI_P, AI_T
                            |
                            v
           Quy đổi AI_P thành công suất P
           Quy đổi AI_T thành nhiệt độ T
                            |
                            v
                   Hiển thị P và T
                            |
                            v
              +--------------------------+
              | P >= 1508 kW ?           |
              +--------------------------+
                  | Có             | Không
                  v                v
          Bắt đầu Timer_P     Reset Timer_P
                  |
                  v
              +--------------------------+
              | Timer_P >= 120 giây ?    |
              +--------------------------+
                  | Có             | Không
                  v                v
              Trip = 1       Tiếp tục kiểm tra
                  |
                  v
              +--------------------------+
              | T >= 90°C ?              |
              +--------------------------+
                  | Có             | Không
                  v                v
          Bắt đầu Timer_T     Reset Timer_T
                  |
                  v
              +--------------------------+
              | Timer_T >= 60 giây ?     |
              +--------------------------+
                  | Có             | Không
                  v                v
              Trip = 1       Tiếp tục chạy
                  |
                  v
              +--------------------------+
              | Trip = 1 ?               |
              +--------------------------+
                  | Có             | Không
                  v                v
           Dừng động cơ      Động cơ tiếp tục chạy
                  |
                  v
            Hiển thị cảnh báo
                  |
                  v
                 KẾT THÚC
```

---

# III. Kết luận

Hệ thống đo giám sát – điều khiển cho động cơ điện xoay chiều 3 pha không đồng bộ roto lồng sóc được thiết kế dựa trên hai đại lượng cần giám sát là công suất tác dụng và nhiệt độ Stato.

Do động cơ làm việc ở chế độ đấu Y với điện áp 6000 V, dòng điện 132 A và công suất định mức 1160 kW, cần sử dụng các thiết bị đo trung gian như CT, PT, Power Transducer, cảm biến PT100 và Temperature Transmitter để đưa tín hiệu đo về mức chuẩn 4–20mA cho PLC.

PLC thực hiện nhiệm vụ đọc tín hiệu analog, quy đổi thành giá trị thực, hiển thị thông số và điều khiển bảo vệ. Khi công suất động cơ lớn hơn hoặc bằng 1508 kW liên tục trong 2 phút hoặc nhiệt độ Stato lớn hơn hoặc bằng 90°C liên tục trong 1 phút, PLC sẽ xuất tín hiệu dừng động cơ để bảo vệ hệ thống.

Bài làm đã đáp ứng đầy đủ các yêu cầu của đề bài gồm: xác định chế độ đấu Y, vẽ sơ đồ nguyên lý, thuyết minh thuật toán đo, tính chọn thiết bị, xây dựng phương trình chuyển đổi, lập trình thu thập dữ liệu và xây dựng lưu đồ điều khiển dừng động cơ khi có sự cố.
