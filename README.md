# StudyMate

StudyMate là ứng dụng Android hỗ trợ quản lý lớp học và trao đổi giữa giáo viên, sinh viên. Project gồm ứng dụng Android viết bằng Java và REST API xây dựng bằng Spring Boot, sử dụng MySQL để lưu trữ dữ liệu.

## Chức năng chính

| Vai trò | Chức năng |
| --- | --- |
| Quản trị viên | Tạo, tìm kiếm, lọc tài khoản; cập nhật trạng thái và xóa tài khoản. |
| Giáo viên | Tạo, sửa, xóa lớp học; duyệt yêu cầu tham gia; quản lý sinh viên, điểm số; gửi thông báo và trao đổi phản hồi. |
| Sinh viên | Gửi yêu cầu tham gia, rời lớp; xem thông tin lớp, danh sách sinh viên, điểm và thông báo; trao đổi phản hồi. |

Hệ thống có đăng nhập, đăng xuất và phân quyền theo `ROLE_ADMIN`, `ROLE_TEACHER`, `ROLE_STUDENT`. Backend sử dụng Spring Security, JWT và BCrypt; ứng dụng Android duy trì cookie qua OkHttp.

## Công nghệ

| Thành phần | Công nghệ trong mã nguồn |
| --- | --- |
| Android | Java 17, XML Layout, ViewModel, LiveData, Navigation, Material Components |
| Kết nối API | Retrofit 2, Gson, OkHttp |
| Android SDK | minSdk 24, targetSdk 36, compileSdk 36 |
| Backend | Java 21, Spring Boot 3.5.7, Spring Web, Spring Security |
| Dữ liệu | Spring Data JPA, Hibernate, MySQL |
| Công cụ build | Gradle Wrapper cho Android, Maven Wrapper cho backend |

## Cấu trúc thư mục

```text
N10_StudyMate/
├── README.md
├── Androi_project-main/
│   └── Androi_project-main/          # Mở thư mục này bằng Android Studio
│       ├── app/src/main/
│       │   ├── java/com/example/studymate/
│       │   │   ├── data/            # API, repository và các lớp dữ liệu
│       │   │   └── ui/              # Giao diện và ViewModel
│       │   ├── res/                # Layout, navigation, tài nguyên
│       │   └── AndroidManifest.xml
│       └── gradlew.bat
└── AndroidBE-main/
    ├── src/main/java/com/example/StudyMate/
    │   ├── controller/             # REST API
    │   ├── model/                  # Entity
    │   ├── repository/             # Truy cập dữ liệu
    │   ├── service/                # Xử lý nghiệp vụ
    │   └── security/               # Xác thực và phân quyền
    ├── src/main/resources/application.properties
    ├── study_mate.sql              # Bản dump cơ sở dữ liệu
    ├── pom.xml
    └── mvnw.cmd
```

## Yêu cầu môi trường

- JDK 21 để chạy backend; Android đang cấu hình mức ngôn ngữ Java 17.
- Android Studio hỗ trợ Android Gradle Plugin 8.13.1, Android SDK Platform 36.
- Máy ảo hoặc thiết bị Android từ Android 7.0 (API 24).
- MySQL 8 trở lên. Nếu nhập bản dump, cần phiên bản tương thích với collation `utf8mb4_0900_ai_ci`.

Các lệnh dưới đây dùng PowerShell và được chạy từ thư mục gốc project, trừ khi có ghi chú khác.

## Chạy backend

### 1. Tạo cơ sở dữ liệu

Trong MySQL, tạo database trống:

```sql
CREATE DATABASE IF NOT EXISTS study_mate
CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```

Backend cấu hình `spring.jpa.hibernate.ddl-auto=update`, nên Hibernate sẽ tạo hoặc cập nhật các bảng khi khởi động.

Nếu cần dữ liệu từ bản dump, có thể nhập `AndroidBE-main/study_mate.sql` bằng MySQL Workbench. File này chứa lệnh `DROP TABLE`; chỉ nhập vào cơ sở dữ liệu dành cho phát triển mà bạn chấp nhận thay thế dữ liệu.

### 2. Cấu hình biến môi trường

Tạo file `.env` trong `AndroidBE-main/`:

```dotenv
DB_URL=jdbc:mysql://localhost:3306/study_mate
DB_USERNAME=your_mysql_username
DB_PASSWORD=your_mysql_password
JWT_SECRET=replace_with_generated_base64url_secret
JWT_EXPIRATION=86400000
```

`JWT_EXPIRATION` tính bằng mili giây; `86400000` tương ứng 24 giờ. `JWT_SECRET` phải là chuỗi Base64URL của khóa ngẫu nhiên có độ dài tối thiểu 32 byte. Có thể sinh bằng PowerShell:

```powershell
$jwtKeyBytes = New-Object byte[] 32
$jwtRng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$jwtRng.GetBytes($jwtKeyBytes)
[Convert]::ToBase64String($jwtKeyBytes).TrimEnd('=').Replace('+', '-').Replace('/', '_')
$jwtRng.Dispose()
```

Dán kết quả vào `JWT_SECRET`. File `.env` đã được backend đưa vào `.gitignore`; không đưa thông tin kết nối thật hoặc khóa JWT lên repository.

### 3. Khởi động

```powershell
cd AndroidBE-main
.\mvnw.cmd spring-boot:run
```

Backend chạy tại `http://localhost:8080`, các API nằm dưới `/api`.

### 4. Tài khoản mặc định

Khi bảng người dùng chưa có dữ liệu, backend tự tạo:

| Tên đăng nhập | Mật khẩu | Vai trò |
| --- | --- | --- |
| `admin` | `123456` | Quản trị viên |
| `teacher` | `123456` | Giáo viên |
| `student` | `123456` | Sinh viên |

Các tài khoản này chỉ được tạo khi `userRepository.count() == 0`. Nếu đã nhập bản dump hoặc có người dùng, bước tạo mặc định sẽ được bỏ qua. Đây là thông tin phục vụ chạy thử; cần thay đổi cơ chế khởi tạo và mật khẩu trước khi triển khai thực tế.

## Chạy ứng dụng Android

1. Mở `Androi_project-main/Androi_project-main/` bằng Android Studio.
2. Cài SDK Platform 36 nếu được yêu cầu, chờ Gradle Sync hoàn tất.
3. Mở file `app/src/main/java/com/example/studymate/data/network/RetrofitClient.java` và sửa `BASE_URL` theo môi trường:

   | Môi trường | URL ví dụ |
   | --- | --- |
   | Android Emulator chuẩn, backend trên cùng máy | `http://10.0.2.2:8080/` |
   | Điện thoại thật cùng mạng LAN với máy chạy backend | `http://<IP-LAN-cua-may>:8080/` |

   URL cần có dấu `/` ở cuối. Mã nguồn hiện đang dùng địa chỉ LAN cố định `http://192.168.100.185:8080/`; hãy thay bằng địa chỉ phù hợp với máy của bạn.

4. Khởi động backend, chọn máy ảo hoặc thiết bị thật và nhấn **Run**.
5. Đăng nhập bằng tài khoản phù hợp để sử dụng các màn hình theo vai trò.

Với thiết bị thật, máy chạy backend và điện thoại cần kết nối được với nhau; cho phép kết nối đến cổng 8080 trong firewall nếu cần.

## Các nhóm API chính

| Đường dẫn | Mục đích |
| --- | --- |
| `POST /api/auth/signin` | Đăng nhập |
| `POST /api/auth/logout` | Đăng xuất |
| `/api/admin/users` | Quản lý tài khoản |
| `/api/teacher/classes` | Quản lý lớp và yêu cầu tham gia |
| `POST /api/student/classes/join` | Yêu cầu tham gia lớp |
| `DELETE /api/student/classes/{classId}` | Rời lớp |
| `/api/teacher/grades` | Tạo, sửa, xóa điểm |
| `GET /api/student/classes/{classId}/grades` | Xem điểm trong lớp |
| `POST /api/teacher/classes/{classId}/notifications` | Tạo thông báo |
| `GET /api/user/classes/{classId}/notifications` | Danh sách thông báo |
| `POST /api/user/feedback/send` | Gửi phản hồi |
| `GET /api/user/classes/{classId}/feedback` | Xem trao đổi phản hồi |

Tham số và cấu trúc request/response được định nghĩa trong các controller, DTO của backend và `ApiService.java` của Android.

## Build và kiểm thử

Backend, chạy trong `AndroidBE-main/` với MySQL và các biến môi trường đã cấu hình:

```powershell
.\mvnw.cmd test
.\mvnw.cmd clean package
```

Android, chạy trong `Androi_project-main/Androi_project-main/`:

```powershell
.\gradlew.bat testDebugUnitTest
.\gradlew.bat assembleDebug
```

APK debug được tạo tại `app/build/outputs/apk/debug/app-debug.apk`. Các lệnh này là hướng dẫn thực thi; README không xác nhận rằng build hoặc kiểm thử đã chạy thành công trong môi trường hiện tại.

## Xử lý lỗi thường gặp

- **Backend không kết nối được MySQL:** kiểm tra dịch vụ MySQL, tên database, `DB_URL`, tài khoản và quyền truy cập.
- **Backend báo lỗi JWT:** kiểm tra `JWT_SECRET` là Base64URL hợp lệ, giải mã được ít nhất 32 byte, và `JWT_EXPIRATION` là số mili giây hợp lệ.
- **Android không gọi được API:** kiểm tra `BASE_URL`, backend đã chạy, cổng 8080 và kết nối mạng. Với Android Emulator, dùng `10.0.2.2` để truy cập máy chủ trên máy phát triển.
- **Đăng nhập mặc định thất bại:** kiểm tra database đã có người dùng hay chưa; tài khoản mẫu chỉ được khởi tạo khi bảng người dùng trống.
- **Gradle Sync thất bại:** kiểm tra JDK dùng cho Gradle, SDK 36 và khả năng tải các dependency.
