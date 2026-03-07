# BÀI TẬP TUẦN 5 MÔN HỆ ĐIỀU HÀNH NHÚNG

# A BÀI 1: Viết 1 chương trình C/C++ có sử dụng thư viện cJSON
## MỤC TIÊU CỦA BÀI
- Build Buildroot có tích hợp thư viện cJSON
- Viết chương trình HelloJSON.c
- Cross compile bằng toolchain Buildroot
- Copy sang BBB
- Chạy thành công
## CÁC BƯỚC TIẾN HÀNH LÀM BÀI
### PHẦN 1: BẬT cJSON TRONG BUILDROOT
### Bước 1: Vào menu cấu hình
```bash
cd buildroot
make menuconfig
```
### Bước 2: Bật thư viện cJSON
- Vào:
```bash
Target packages
    → Libraries
        → JSON/XML
            → [*] cjson
```
- Chọn [*] cjson
- Thoát và Save

### Bước 3: Build lại hệ thống
- Dùng lệnh "make" hoặc "make -j2"
### Bước 4: Kiểm tra đã có thư viện chưa
```bash
ls output/target/usr/lib | grep cjson
```
- Phải thấy:
```bash
libcjson.so
libcjson.so.1
```
### PHẦN 2: VIẾT CHƯƠNG TRÌNH HELLOJSON.c
- Tạo file:
```bash
nano HelloJSON.c
```
- Nội dung:
```bash
#include <stdio.h>
#include <cjson/cJSON.h>

int main() {
    // Chuỗi JSON mẫu
    const char *chuoi_json = "{ \"ten\": \"Quan\", \"tuoi\": 21, \"truong\": \"PTIT\" }";

    // Parse JSON
    cJSON *json = cJSON_Parse(chuoi_json);
    if (json == NULL) {
        printf("Loi parse JSON\n");
        return 1;
    }

    // Lấy từng trường
    cJSON *ten = cJSON_GetObjectItem(json, "ten");
    cJSON *tuoi = cJSON_GetObjectItem(json, "tuoi");
    cJSON *truong = cJSON_GetObjectItem(json, "truong");

    // In ra màn hình
    printf("Ten: %s\n", ten->valuestring);
    printf("Tuoi: %d\n", tuoi->valueint);
    printf("Truong: %s\n", truong->valuestring);

    // Giải phóng bộ nhớ
    cJSON_Delete(json);

    return 0;
}
```
### PHẦN 3: Biên dịch chương trình HelloJSON bằng toolchain Buildroot
- Quay lại thư mục Buildroot.

- Thêm toolchain vào PATH:
```bash
export PATH=$PATH:$(pwd)/output/host/bin
```
- Kiểm tra compiler:
```bash
ls output/host/bin | grep gcc
```
- Biên dịch chương trình:
```bash
arm-buildroot-linux-gnueabihf-gcc HelloJSON.c -o HelloJSON -lcjson
```
- Kiểm tra file:
```bash
file HelloJSON
```
- Kết quả:
```bash
ELF 32-bit LSB executable, ARM
```

### PHẦN 4: Chép chương trình vào rootfs
- Giải nén rootfs:
```bash
mkdir rootfs
cd rootfs
zcat ../output/images/rootfs.cpio.gz | cpio -idmv
```
- Copy chương trình:
```bash
cp ../HelloJSON usr/bin/
```
### PHẦN 5: CHẠY CHƯƠNG TRÌNH TRONG QEMU (Vì lúc em làm thì bạn trong nhóm em cầm BBB nên em thử làm trên QEMU)
- Sau khi boot xong chạy chương trình:
```bash
HelloJSON
```
- Kết quả hiển thị:
```bash
Ten: Quan
Tuoi: 21
Truong: PTIT
```

<img width="1927" height="2560" alt="image" src="https://github.com/user-attachments/assets/de4965f9-8345-4582-aa25-a4773f0bea30" />

# B Bài 2: Tự tạo thư viện cá nhân
## MỤC TIÊU CỦA BÀI
- Hiểu được cách tạo thư viện trong C/C++
- Biết được cách biên dịch Static Library (.a) và Shared Library (.so)
- Sử dụng thư viện đã tạo trong một chương trình C
- Biết cách biên dịch chương trình sử dụng 2 loại thư viện khác nhau
- So sánh kích thước chương trình và dependency giữa static và shared library
## CÁC BƯỚC THỰC HIỆN
### Bước 1: Tạo thư mục làm việc 
```bash
cd ~
mkdir -p buildroot/mylib
cd buildroot/mylib
mkdir include
mkdir src
mkdir build
```
### Bước 2: Tạo file header của thư viện
- Tạo file header:
```bash
nano include/mylib.h
```
- Nội dung:
```bash
#ifndef MYLIB_H
#define MYLIB_H

int add(int a, int b);
int sub(int a, int b);

#endif
```
### Bước 3: Tạo file source của thư viện
- Tạo file:
```bash
nano src/mylib.c
```
- Nội dung:
```bash
#include "mylib.h"

int add(int a, int b)
{
    return a + b;
}

int sub(int a, int b)
{
    return a - b;
}
```
### Bước 4: Biên dịch source thành objcet file
```bash
gcc -c src/mylib.c -Iinclude -fPIC -o build/mylib.o
```
### Bước 5: Tạo Static Library (.a)
```bash
ar rcs build/libmylib.a build/mylib.o
```
- Kiểm tra:
```bash
ls build
```
- Kết quả:
```bash
libmylib.a
mylib.o
```
### Bước 6: Tạo Shared Library (.so)
```bash
gcc -shared -o build/libmylib.so build/mylib.o
```
- Kiểm tra:
```bash
ls build
```
- Kết quả:
```bash
libmylib.a
libmylib.so
mylib.o
```
<img width="567" height="301" alt="image" src="https://github.com/user-attachments/assets/91042450-2fb1-4a90-b658-97043c5936a2" />

### Bước 7: Viết chương trình sử dụng thư viện
- Tạo file:
```bash
nano main.c
```
- Nội dung:
```bash
#include <stdio.h>
#include "mylib.h"

int main()
{
    int a = 10;
    int b = 5;

    printf("Xin chao tu mylib!\n");
    printf("Tong: %d\n", add(a,b));
    printf("Hieu: %d\n", sub(a,b));

    return 0;
}
```
### Bước 8: Biên dịch chương trình với Static Library
```bash
${CROSS}gcc main.c -Iinclude build/libmylib.a -o test_static_arm
```
### Bước 9: Biên dịch chương trình với Shared Library
```bash
${CROSS}gcc main.c -Iinclude -Lbuild -lmylib -o test_shared_arm
```
- Kết quả:
<img width="1080" height="1440" alt="image" src="https://github.com/user-attachments/assets/bb690df2-4143-43c1-9271-1720936ad71f" />
### Bước 10: So sánh kích thước chương trình
```bash
ls -lh test_static_arm
ls -lh test_shared_arm
```
- Kết quả:
<img width="1927" height="2560" alt="image" src="https://github.com/user-attachments/assets/7e2b83e2-50d1-40ab-acf9-4a656aac595e" />

### Bước 11: Kiểm tra dependency của chương trình
```bash
readelf -d test_static_arm
readelf -d test_shared_arm
```
- Trong test_shared_arm sẽ thấy:
```bash
NEEDED libmylib.so
- Điều này cho thấy chương trình phụ thuộc vào shared library.

