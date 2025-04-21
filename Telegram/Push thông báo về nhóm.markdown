# Hướng Dẫn Gửi Tin Nhắn Telegram Vào Nhóm Bằng Bot

Hướng dẫn này sẽ giúp bạn tạo một bot Telegram và tích hợp nó vào ứng dụng React để gửi tin nhắn thông báo đến một nhóm Telegram.

## Các Bước Thiết Lập Bot Telegram

### 1. Tạo Bot Telegram và Lấy Token

Quy trình tạo bot và lấy token giống như khi gửi tin nhắn cá nhân:

1. Mở Telegram và tìm kiếm **@BotFather** (https://t.me/BotFather).

2. Gửi lệnh `/newbot` để tạo bot mới.

3. Nhập tên cho bot (ví dụ: `MyGroupNotificationBot`). @BotFather sẽ cung cấp một **token**. Ví dụ:

   ```
   7353515102:AAGR-7793375719:AAF71_UpCI9Vlv1HioyY
   ```

   ![alt text](image-1.png)
   **Lưu ý**: Token cần được giữ bí mật.

### 2. Thêm Bot Vào Nhóm và Lấy Chat ID

Để gửi tin nhắn đến một nhóm, bạn cần thêm bot vào nhóm và lấy **chat ID** của nhóm:

1. Thêm bot vào nhóm Telegram bằng cách mời bot (tìm bot theo tên, ví dụ: `@MyGroupNotificationBot`) và cấp quyền gửi tin nhắn.

2. Gửi một tin nhắn bất kỳ trong nhóm để bot ghi nhận (ví dụ: `/start`).

3. Gọi API Telegram để lấy thông tin cập nhật:

   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```

   Thay `<TOKEN>` bằng token của bot. Ví dụ:

   ```
   https://api.telegram.org/bot7353515102:AAGR-7793375719:AAF71_UpCI9Vlv1HioyY/getUpdates
   ```

4. Phản hồi từ API sẽ chứa thông tin về các tin nhắn, bao gồm **chat ID** của nhóm. Chat ID của nhóm thường có dạng số âm. Ví dụ:

   ```json
   {
     "ok": true,
     "result": [
       {
         "update_id": 123456,
         "message": {
           "chat": {
             "id": -1002212552051,
             "title": "haiproxxx",
             "type": "supergroup"
           }
         }
       }
     ]
   }
   ```

   Trong trường hợp này, **chat ID** của nhóm là `-1002212552051`.

### 3. Tích Hợp Gửi Tin Nhắn Vào Nhóm Trong Ứng Dụng React

Dưới đây là mã React để gửi tin nhắn đến nhóm Telegram. Mã này tương tự như khi gửi tin nhắn cá nhân, chỉ khác ở **chat ID** của nhóm.

```tsx
import React, { useState } from "react";
import axios from "axios";

const TelegramGroupNotification = () => {
  const [message, setMessage] = useState("");
  const TELEGRAM_BOT_TOKEN = "7353515102:AAGR-7793375719:AAF71_UpCI9Vlv1HioyY";
  const CHAT_ID = -1002212552051; // Chat ID của nhóm

  const handleSendMessage = async () => {
    if (!message.trim()) {
      alert("Vui lòng nhập nội dung tin nhắn!");
      return;
    }

    try {
      await axios.post(
        `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
        {
          chat_id: CHAT_ID,
          text: `New message: ${message}`,
        }
      );
      alert("Tin nhắn đã được gửi đến nhóm Telegram!");
      setMessage(""); // Xóa nội dung sau khi gửi
    } catch (error) {
      console.error("Lỗi khi gửi tin nhắn:", error);
      alert("Đã xảy ra lỗi khi gửi tin nhắn.");
    }
  };

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Gửi Tin Nhắn Đến Nhóm Telegram</h2>
      <input
        type="text"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        className="border p-2 mr-2"
        placeholder="Nhập tin nhắn..."
      />
      <button
        onClick={handleSendMessage}
        className="bg-blue-500 text-white p-2 rounded"
      >
        Gửi
      </button>
    </div>
  );
};

export default TelegramGroupNotification;
```

### Giải Thích Mã

- **State** `message`: Lưu nội dung tin nhắn người dùng nhập.
- **Biến** `TELEGRAM_BOT_TOKEN` **và** `CHAT_ID`: Lưu token của bot và chat ID của nhóm.
- **Hàm** `handleSendMessage`:
  - Kiểm tra xem tin nhắn có rỗng không.
  - Gửi yêu cầu POST đến API Telegram với nội dung tin nhắn.
  - Xử lý lỗi nếu có và thông báo kết quả.
- **Giao diện**:
  - Ô nhập liệu để người dùng nhập tin nhắn.
  - Nút "Gửi" để kích hoạt hàm gửi tin nhắn.

### Lưu Ý

- **Quyền của bot**: Đảm bảo bot có quyền gửi tin nhắn trong nhóm. Nếu bot bị hạn chế, vào cài đặt nhóm để cấp quyền.

- **Bảo mật token**: Lưu token và chat ID trong biến môi trường (ví dụ: `.env`) thay vì mã nguồn.

- **Tùy chỉnh tin nhắn**: Có thể định dạng tin nhắn bằng Markdown hoặc HTML bằng cách thêm tham số `parse_mode`. Ví dụ:

  ```javascript
  await axios.post(
    `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
    {
      chat_id: CHAT_ID,
      text: `*New message*: ${message}`,
      parse_mode: "Markdown",
    }
  );
  ```

- **Xử lý lỗi**: Thông báo lỗi rõ ràng để người dùng biết nếu gửi tin nhắn thất bại.

### Mở Rộng

- **Tích hợp với Firestore**: Kết hợp với Firestore để gửi tin nhắn tự động khi có dữ liệu mới.
- **Gửi nội dung đa dạng**: Sử dụng các API Telegram khác để gửi ảnh, tệp hoặc nút tương tác (inline buttons).
- **Quản lý nhiều nhóm**: Lưu danh sách chat ID của nhiều nhóm để gửi tin nhắn đến các nhóm khác nhau.

Với hướng dẫn này, bạn có thể dễ dàng tích hợp bot Telegram để gửi thông báo đến nhóm từ ứng dụng React.
