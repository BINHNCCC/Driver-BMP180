# Driver-BMP180
Driver BMP 180 sensor for Linux Kernel space on Raspberry Pi 4B


//===========================================Đây là README hướng dẫn sử dụng Driver BMP 180============================


                                            File tài liệu bao gồm:
                                                1. Giới thiệu chung về cảm biến áp suất BMP 180
                                                2. Thông số kỹ thuật của BMP 180
                                                3. Cài đặt & Biên dịch
                                                4. Kiểm tra hoạt động của Driver
                                                5. Các hàm cần thiết 
                                                6. Tác giả và Demo


//=====================================================================================================================  
//                                     1. Giới thiệu chung về cảm biến áp suất BMP 180
//=====================================================================================================================


Cảm biến BMP180 là một cảm biến áp suất khí quyển và nhiệt độ do hãng Bosch Sensortec phát triển, thuộc dòng BMP Series 
(BMP085, BMP180, BMP280, BME280). Nó được sử dụng rộng rãi trong các ứng dụng IoT, dự án thời tiết, đo độ cao (altimeter)
và hệ thống nhúng.

#Đặc điểm nổi bật: 
✅ Đo áp suất khí quyển (300–1100 hPa) với độ chính xác ±0.12 hPa
✅ Đo nhiệt độ (−40°C đến +85°C) với độ chính xác ±0.5°C
✅ Giao tiếp I2C hoặc SPI (BMP180 chỉ hỗ trợ I2C)
✅ Tiêu thụ điện năng thấp (3.3V–5V, dòng tiêu thụ ~5µA ở chế độ sleep)
✅ Kích thước nhỏ gọn (3.6 × 3.8 × 0.93 mm)
#Nguyên lý hoạt động
Cảm biến áp suất: Một màng ngăn silicon biến dạng dưới tác dụng của áp suất khí quyển.
Cảm biến nhiệt độ: Đo nhiệt độ môi trường để bù sai số cho phép đo áp suất.
Bộ ADC 16-bit: Chuyển đổi tín hiệu analog sang digital.
Thuật toán hiệu chuẩn: Sử dụng 11 tham số hiệu chuẩn (AC1–AC6, B1–B2, MB, MC, MD) để tính toán chính xác giá trị áp suất và nhiệt độ.

//=====================================================================================================================  
//                                     2.Thông số kỹ thuật của BMP 180
//=====================================================================================================================

2.1 Thông số kỹ thuật chung
| Thông số                  | Giá trị                                                       |
| ------------------------- | ------------------------------------------------------------- |
| **Nguồn hoạt động (VDD)** | 1.8V đến 3.6V                                                 |
| **Nguồn logic (VDDIO)**   | 1.62V đến 3.6V                                                |
| **Giao tiếp**             | I2C (lên đến 3.4 MHz) hoặc SPI (3/4 dây)                      |
| **Dòng tiêu thụ**         | \~12 µA (trong chế độ hoạt động)                              |
| **Nhiệt độ hoạt động**    | -40°C đến +85°C                                               |
| **Áp suất hoạt động**     | 300 hPa đến 1100 hPa (tương đương độ cao: +9000 m đến -500 m) |

2.2 Độ phân giải và độ chính xác
| Thông số                   | Giá trị              |
| -------------------------- | -------------------- |
| **Độ phân giải áp suất**   | 0.01 hPa (1 Pa)      |
| **Độ chính xác tuyệt đối** | ±1.0 hPa (ở 0–65 °C) |
| **Độ chính xác nhiệt độ**  | ±1.0 °C              |
| **Độ phân giải nhiệt độ**  | 0.1 °C               |

2.3 Tốc độ đo (Oversampling Settings – OSS)
| OSS                       | Thời gian chuyển đổi | Độ nhiễu (RMS noise) | Độ phân giải |
| ------------------------- | -------------------- | -------------------- | ------------ |
| 0 (Ultra low power)       | \~4.5 ms             | 0.06 hPa (0.5 m)     | 16 bit       |
| 1 (Standard)              | \~7.5 ms             | 0.05 hPa (0.4 m)     | 17 bit       |
| 2 (High Resolution)       | \~13.5 ms            | 0.04 hPa (0.3 m)     | 18 bit       |
| 3 (Ultra High Resolution) | \~25.5 ms            | 0.03 hPa (0.25 m)    | 19 bit       |

2.4 Địa chỉ I2C và thanh ghi:
   ADDRESS_I2C_BMP180     0x77
  
   BMP180_REG_CONTROL     0xF4 //Start measuring pressure or temperature
  
   BMP180_REG_RESULT      0xF6

  //Calibration Coefficients
    //-Pressure
   BMP180_REG_CAL_AC1     0xAA
   BMP180_REG_CAL_AC2     0xAC
   BMP180_REG_CAL_AC3     0xAE
   BMP180_REG_CAL_AC4     0xB0  //unsigned
   BMP180_REG_CAL_B1      0xB6
   BMP180_REG_CAL_B2      0xB8
    //--Temperature
   BMP180_REG_CAL_AC5     0xB2 
   BMP180_REG_CAL_AC6     0xB4
   BMP180_REG_CAL_MB      0xBA
   BMP180_REG_CAL_MC      0xBC
   BMP180_REG_CAL_MD      0xBE

  //Control registers values for different internal oversampling_setting(oss)
  #define BMP180_REG_TEMPERATURE 0x2E
  #define BMP180_REG_PRESSURE    0x34  //oss= 0
    

//=====================================================================================================================  
//                                     3. Cài đặt & Biên dịch
//=====================================================================================================================

3.1 Thiết lập kết nối phần cứng

| BMP180 Pin | Raspberry Pi 4 Pin                   | Ghi chú                   |
| ---------- | ------------------------------------ | ------------------------- |
| **VIN**    | **Pin 1 (3.3V)**                     | Cảm biến chấp nhận 3.3    |
| **GND**    | **Pin 6 (GND)**                      | Nối đất                   |
| **SCL**    | **Pin 5 (GPIO3 / SCL)**              | Dữ liệu clock I2C         |
| **SDA**    | **Pin 3 (GPIO2 / SDA)**              | Dữ liệu I2C               |

3.2 Thiết lập phần mềm trên Raspberry Pi

Mở Terminal, chạy sudo raspi-config -> chọn Interface Options → I2C → Enable -> Sau đó reboot "sudo reboot"
Kiểm tra kết nối i2cdetect -y 1 -> Nếu hiện 0x77 thì đẫ kết nối thành công

3.3 Biên dịch Driver BMP 180

B1: Tải  file bmp180_driver.c bmp180.dts Makefile test.c tại trang Github https://github.com/ThanhTam1805/Driver-BMP180
vào cùng 1 thư mục trên Raspberry Pi
B2: Mở Terminal chuyển đến thư mục chứa các file đã tải chạy lệnh Make -> sau đó chạy lệnh sudo cp bmp180.dtbo /boot/overlays/ -> chạy lệnh sudo nano /boot/config.txt
  thêm dòng sau vào file config.txt thêm ở dòng cuối cùng chỗ [all] là: dtoverlay=bmp180 -> sau đó chạy lệnh sudo reboot
B3: Mở Terminal chạy lệnh make 

3.4 Cài đặt Driver BMP180 
B1: Mở Terminal chuyển đến thư mục chứa các file đã tải chạy lệnh sudo insmod bmp180_driver.ko
***Note: Để xóa Driver thì chạy lệnh sudo rmmod bmp180_driver.ko

//=====================================================================================================================  
//                                     4. Kiểm tra hoạt động Driver
//=====================================================================================================================


B1: Mở Terminal chuyển đến thư mục chứa các file đã tải chạy lệnh gcc -o bmp180_test test.c
B2: Chạy lệnh sudo ./bmp180_test

-> Kết quả nếu thành công 
    pi1@raspberrypi:~/Desktop/BMP $ sudo ./bmp180_test
    Temperature: 37.9 °C
    Pressure: 118267.00 Pa

//=====================================================================================================================  
//                                     5. Các hàm cần thiết
//=====================================================================================================================
      //-----Hàm test: giao tiếp userspace viuws kernel------////
  fd = open("/dev/bmp180", O_RDWR); // mở driver với chức năng read,write
  ioctl(fd, IOCTL_READ_TEMP, &temperature); // đọc nhiệt độ 
  //Đọc áp suất với mode oss= 0; 1; 2;3
  pressure_data.oss = 0; //vd: mode 0
  ioctl(fd, IOCTL_READ_PRESSURE, &pressure_data)
  
Đoạn mã sử dụng 3 hàm chính dùng để giao tiếp giữ lớp User pasce và Kernel space như:
*** Hàm open(): dùng để device file
- Cú pháp:
  int open(const char *pathname, int flags);
  int open(const char *pathname, int flags, mode_t mode);
- Các đối số của hàm:
  1. pathname
    Đường dẫn đến file bạn muốn mở.
    Ví dụ ở đây là file device ở dưới lớp Kernel: "/dev/bmp180" 
  2. flags — Cờ mở file
  Chỉ định hành vi mở file ở các dạng như (đọc, ghi, tạo mới, v.v.)
    Một số cờ phổ biến:
    Flag	Ý nghĩa
    O_RDONLY	Mở chỉ để đọc
    O_WRONLY	Mở chỉ để ghi
    O_RDWR	Mở để đọc và ghi
    O_CREAT	Tạo file nếu chưa tồn tại (cần thêm đối số mode)
    O_TRUNC	Xóa nội dung cũ (nếu mở để ghi)
    O_APPEND	Ghi nối vào cuối file
Ta có thể kết hợp nhiều cờ bằng toán tử |.
  Ví dụ: O_RDWR | O_CREAT
  3. mode (tùy chọn)
  Chỉ dùng nếu có O_CREAT.Xác định quyền truy cập file nếu nó được tạo mới

fd = open("/dev/bmp180", O_RDWR); // Đọc ghi giá trị hoặc ghi giá trị. Nếu có lỗi (driver chưa được tải hoặc không tồn 
tại) thì hàm này sẽ trả về giá trị < 0. 
*** Hàm ioctl(): dùng để gọi(gửi những lệnh cần thiết) từ lớp User space xuống lớp Kernel. Thường dùng để đọc/ghi các giá 
trị cần thiết
- Cú pháp
  int ioctl(int fd, unsigned long request, arg);
- Các đối số của hàm:
| Tham số          | Ý nghĩa                                                           |
| ---------------- | ----------------------------------------------------------------- |
| `fd`             | File descriptor đã mở bằng `open()` (ví dụ: `/dev/bmp180`)        |
| `request`        | Mã lệnh IOCTL, xác định hành động muốn thực hiện                  |
| `arg` (tùy chọn) | Địa chỉ đến vùng nhớ dữ liệu (có thể là struct, long, pointer...) |
- Các Macro thường dùng trong ioctl
| Macro                            | Ý nghĩa                                          |
| -------------------------------- | ------------------------------------------------ |
| `_IOR(type, number, data_type)`  | **Đọc** dữ liệu từ kernel-space về user-space    |
| `_IOW(type, number, data_type)`  | **Ghi** dữ liệu từ user-space xuống kernel-space |
| `_IOWR(type, number, data_type)` | **Ghi và đọc** dữ liệu (2 chiều)                 |
| `_IO(type, number)`              | Không truyền dữ liệu                             |
- Trong ví dụ về mã tôi đã sử lệnh _IOR cho việc đọc dữ liệu là nhiệt độ và lệnh _IOWR cho việc ghi thông số
oss (độ chính xác) và đọc áp giá trị áp suất cho lớp Kernel.
        ioctl(fd, IOCTL_READ_TEMP, &temperature);
        ioctl(fd, IOCTL_READ_PRESSURE, &pressure_data)
- Ngoài ra tôi còn sử dụng các hàm in thông báo perror() và hàm đống file close()
- Cú pháp:
            void perror(const char *msg);
            int close(int fd);
  - Lưu ý: Bạn có thể không sử dụng hàm perror() nhưng bắt buộc phải sử dụng hàm close() để đảm bảo đóng file khi
    sử dụng xong.
 ###NOTE: Các trị trả về như temperature và pressure từ hàm ioctl bạn có thể sử dụng để điều khiển thiết bị hoặc thông báo
           tùy ứng dụng mà bạn có thể phát triển thêm tùy theo dự án của mình!!!. Chúc bạn thành công


    
//=====================================================================================================================  
//                                     6. Tác giả 
//=====================================================================================================================

### Driver BMP180 version 1.0  được phát triển bởi các tác giả sau:
- Lê Nguyễn Thành Tâm      22146214
- Nguyễn Chí Bình          22146084
- Võ Phước Thắng 
### Driver BMP180 sử dụng tài liệu Bosch. Bạn có thể truy cập tại địa chỉ sau: https://cdn-shop.adafruit.com/datasheets/BST-BMP180-DS000-09.pdf
### Clip demo của Driver bạn có thể truy cập Youtube tại địa chỉ: https://youtu.be/ZMpUF-4gxYI

//=====================================================================================================================  
//                                              !!! NOTE NOTE NOTE !!!
//=====================================================================================================================

Trên một số dòng Raspberry Pi có thể chạy mặc định Driver bmp280_i2c khi vừa khởi động hệ thống làm cho driver không hoạt
động dù đã thực hiện tất các bước trên. Để khắc phục tình trạng này bạn cần thực hiện theo các bước sau:

B1: Chạy lệnh lsmod | grep bmp
    Nếu cửa sổ Terminal hiện thị như sau: 
    bmp280_i2c             16384  0
    bmp280                 28672  1 bmp280_i2c
    industrialio           90112  1 bmp280
    regmap_i2c             16384  1 bmp280_i2c
thực hiện bước 2
B2: Chạy lệnh 
    sudo rmmod bmp280_i2c  
    sudo rmmod bmp280      
B3: Tiến hành gỡ cài đặt Driver BMP180 (xem ở mục 3.4) và thực hiện lại các bước Mục 3.4





