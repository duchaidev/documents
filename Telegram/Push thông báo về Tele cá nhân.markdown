# Hướng Dẫn Bot Telegram gửi thông báo

Hướng dẫn này sẽ giúp bạn tạo một bot Telegram và tích hợp nó vào ứng dụng React để gửi tin nhắn thông báo đến Telegram cá nhân.

## Các Bước Thiết Lập Bot Telegram

### 1. Tạo Bot Telegram

1. Mở Telegram và tìm kiếm **@BotFather** (https://t.me/BotFather).

2. Gửi lệnh `/newbot` để bắt đầu tạo bot.

3. Nhập tên cho bot (ví dụ: `MyNotificationBot`). @BotFather sẽ cung cấp một **token** để truy cập bot. Ví dụ:

   ```
   7793375719:AAF71_UpCI9Vlv1HioyY1ga0F
   ```

   Lưu ý: Token này cần được giữ bí mật.
   ![alt text](image.png)

### 2. Lấy Chat ID

Để gửi tin nhắn đến một người dùng, bạn cần biết **chat ID**. Thực hiện như sau:

1. Gửi một tin nhắn bất kỳ đến bot vừa tạo (ví dụ: `/start`).

2. Gọi API Telegram để lấy thông tin cập nhật từ bot:

   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```

   Thay `<TOKEN>` bằng token của bot. Ví dụ:

   ```
   https://api.telegram.org/bot7793375719:AAF71_UpCI9Vlv1HioyY1ga0F/getUpdates
   ```

3. Phản hồi từ API sẽ chứa thông tin về tin nhắn, bao gồm **chat ID**. Ví dụ:

   ```json
   {
     "ok": true,
     "result": [
       {
         "update_id": 123456,
         "message": {
           "chat": {
             "id": 5760012591,
             "type": "private"
           }
         }
       }
     ]
   }
   ```

   Trong trường hợp này, **chat ID** là `5760012591`.

### 3. Tích Hợp Gửi Tin Nhắn Trong Ứng Dụng React

Dưới đây là ví dụ mã React để gửi tin nhắn đến Telegram khi người dùng nhập nội dung và nhấn nút gửi. Mã này sử dụng thư viện `axios` để gọi API Telegram.

```tsx
import React, { useState } from "react";
import axios from "axios";

const TelegramNotification = () => {
  const [message, setMessage] = useState("");
  const TELEGRAM_BOT_TOKEN = "7793375719:AAF71_UpCI9Vlv1HioyY1ga0F";
  const CHAT_ID = 5760012591;

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
      alert("Tin nhắn đã được gửi đến Telegram!");
      setMessage(""); // Xóa nội dung sau khi gửi
    } catch (error) {
      console.error("Lỗi khi gửi tin nhắn:", error);
      alert("Đã xảy ra lỗi khi gửi tin nhắn.");
    }
  };

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Gửi Tin Nhắn Telegram</h2>
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

export default TelegramNotification;
```

### Giải Thích Mã

- **State** `message`: Lưu nội dung tin nhắn người dùng nhập.
- **Biến** `TELEGRAM_BOT_TOKEN` **và** `CHAT_ID`: Lưu token của bot và chat ID của người nhận.
- **Hàm** `handleSendMessage`:
  - Kiểm tra xem tin nhắn có rỗng không.
  - Gửi yêu cầu POST đến API Telegram với nội dung tin nhắn.
  - Xử lý lỗi nếu có và thông báo kết quả cho người dùng.
- **Giao diện**:
  - Một ô nhập liệu để người dùng nhập tin nhắn.
  - Một nút "Gửi" để kích hoạt hàm `handleSendMessage`.

### Lưu Ý

- **Bảo mật token**: Không lưu token trực tiếp trong mã nguồn. Sử dụng biến môi trường (ví dụ: `.env`) để lưu trữ token và chat ID.
- **Xử lý lỗi**: Đảm bảo thông báo lỗi rõ ràng để người dùng biết khi có sự cố.
- **Tùy chỉnh tin nhắn**: Bạn có thể thay đổi định dạng tin nhắn trong tham số `text` của API để phù hợp với nhu cầu (ví dụ: thêm thời gian, thông tin người gửi).

### Mở Rộng

- **Gửi tin nhắn tự động**: Kết hợp với các sự kiện trong ứng dụng (ví dụ: khi có dữ liệu mới trong Firestore) để gửi thông báo tự động.
- **Hỗ trợ định dạng**: API Telegram hỗ trợ gửi tin nhắn với định dạng Markdown hoặc HTML bằng cách thêm tham số `parse_mode`.

Ví dụ gửi tin nhắn với Markdown:

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

Với hướng dẫn này, bạn có thể dễ dàng tích hợp bot Telegram để gửi thông báo từ ứng dụng React của mình.
