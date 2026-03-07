# BÀI TẬP TUẦN 5 MÔN HỆ ĐIỀU HÀNH NHÚNG

# BÀI 1: Viết 1 chương trình C/C++ có sử dụng thư viện cJSON
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
