Bài nâng cao 1: NC3 – Mở lại ứng dụng khi nhấn vào thông báoMô tả & Nguyên lý hoạt độngMặc định thông báo chỉ hiển thị văn bản tĩnh. Để tăng tính tương tác và điều hướng người dùng, ứng dụng triển khai PendingIntent ủy quyền cho hệ thống Android khởi chạy lại MainActivity ngay khi người dùng chạm vào thông báo.Điểm kỹ thuật chínhBọc Intent trỏ về MainActivity.class bên trong PendingIntent.getActivity().Đặt cờ Intent.FLAG_ACTIVITY_CLEAR_TOP | Intent.FLAG_ACTIVITY_SINGLE_TOP để tránh tạo chồng chéo nhiều instance của Activity nếu ứng dụng đang ở chế độ nền.Tương thích Android 12+ (API 31+): Bắt buộc sử dụng cờ PendingIntent.FLAG_IMMUTABLE (hoặc kết hợp FLAG_UPDATE_CURRENT) để đảm bảo an toàn bảo mật.Bật .setAutoCancel(true) trên NotificationCompat.Builder để thông báo tự đóng sau khi click.Đoạn mã nguồn triển khaiIntent intent = new Intent(this, MainActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP | Intent.FLAG_ACTIVITY_SINGLE_TOP);

PendingIntent pendingIntent = PendingIntent.getActivity(
        this,
        0,
        intent,
        PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
);

NotificationCompat.Builder b = new NotificationCompat.Builder(this, CHANNEL_ID)
        .setSmallIcon(android.R.drawable.ic_dialog_info)
        .setContentTitle(getString(R.string.noti_title))
        .setContentText(getString(R.string.noti_text))
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)
        .setContentIntent(pendingIntent)
        .setAutoCancel(true);
Bài nâng cao 2: NC4 – Sơ đồ luồng xử lý quyền (Flowchart)Mô tả luồng xử lýSơ đồ mô hình hóa toàn bộ chu trình kiểm tra và xử lý quyền Camera theo mô hình 3 nhánh chuẩn của Google, phân biệt rõ giữa việc từ chối tạm thời và bị hệ thống tự động chặn vĩnh viễn:Nhánh 1: Kiểm tra ContextCompat.checkSelfPermission(...). Nếu bằng PERMISSION_GRANTED, trực tiếp gọi contract TakePicturePreview.Nhánh 2: Khi chưa có quyền, kiểm tra shouldShowRequestPermissionRationale(...). Nếu trả về true, kích hoạt dialog Rationale giải thích lý do cần quyền.Nhánh 3: Nếu cả hai điều kiện trên sai (lần đầu hoặc đã bị chặn vĩnh viễn), gọi launcher.launch(...). Tại callback:Nếu được cấp: Khởi chạy camera.Nếu bị từ chối: Kiểm tra lại shouldShowRequestPermissionRationale. Nếu trả về false, xác định quyền đã bị khóa $\rightarrow$ Hiển thị dialog dẫn hướng sang Settings.ACTION_APPLICATION_DETAILS_SETTINGS.Sơ đồ Mermaid Flowchartflowchart TD
    A([Người dùng nhấn nút 'Chụp ảnh']) --> B{checkSelfPermission == GRANTED?}
    
    B -- ĐÃ CẤP --> C[moCamera: Mở camera bằng TakePicturePreview]
    
    B -- CHƯA CẤP --> D{shouldShowRequestPermissionRationale?}
    
    D -- TRUE: Đã từ chối trước đó --> E[hienGiaiThich: Hiện AlertDialog giải thích lý do]
    E --> F{Lựa chọn của người dùng}
    F -- Đồng ý --> G[xinQuyenCamera.launch]
    F -- Để sau --> H([Dừng lại / Giữ nguyên trạng thái])
    
    D -- FALSE: Lần đầu hoặc Bị chặn --> G
    
    G --> I{Kết quả Callback duocCap}
    I -- TRUE (Cho phép) --> C
    I -- FALSE (Từ chối) --> J{shouldShowRequestPermissionRationale?}
    
    J -- TRUE --> E
    J -- FALSE: Bị chặn vĩnh viễn --> K[hienMoCaiDat: Hiện AlertDialog mời vào Cài đặt]
    K --> L[Mở ACTION_APPLICATION_DETAILS_SETTINGS]
    
    C --> M[Chụp ảnh xem trước Bitmap]
    M --> N[Cập nhật ImageView và kích thước ảnh lên TextView]
    N --> O([Kết thúc luồng])
