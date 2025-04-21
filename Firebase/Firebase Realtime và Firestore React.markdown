# Hướng Dẫn Sử Dụng Firebase Realtime và Firestore Trong Ứng Dụng React

Hướng dẫn này sẽ giúp bạn thiết lập Firebase, kích hoạt Firestore Database và Realtime Database, cấu hình chỉ số (indexes) và tích hợp Firestore vào ứng dụng React để xây dựng một ứng dụng chat đơn giản với tính năng realtime.

## Thiết Lập Dự Án Firebase

### 1. Tạo Dự Án Firebase

1. Truy cập [Firebase Console](https://console.firebase.google.com/) và nhấn **Add project**.
2. Đặt tên dự án và làm theo các bước để hoàn tất.

### 2. Kích Hoạt Firestore Database

1. Trong Firebase Console, vào **Build > Firestore Database**.
2. Nhấn **Create Database**:
   - Chọn chế độ **Test mode** (cho phép đọc/ghi không giới hạn trong 30 ngày) hoặc **Production mode**.
   - Chọn vị trí lưu trữ dữ liệu (ví dụ: `asia-southeast1`).
3. Mặc định, rules của Firestore là `false` (không cho phép truy cập). Để đơn giản, sửa rules thành `true` để cho phép đọc/ghi:

   ```javascript
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```

   **Lưu ý**: Rules này chỉ nên dùng trong giai đoạn phát triển. Trong môi trường production, cần cấu hình rules bảo mật hơn.
   ![alt text](<Untitled (1).webp>)

### 3. Kích Hoạt Realtime Database (Tùy Chọn)

1. Trong Firebase Console, vào **Build > Realtime Database**.
2. Nhấn **Create Database** và chọn vị trí lưu trữ.
3. Mặc định, rules của Realtime Database cũng là `false`. Sửa thành `true` để cho phép đọc/ghi:

   ```javascript
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```

   **Lưu ý**: Tương tự Firestore, chỉ dùng rules này trong giai đoạn phát triển.

### 4. Cấu Hình Indexes Cho Firestore

Để sử dụng truy vấn với `where` và `orderBy` trong Firestore, bạn cần tạo chỉ số (indexes) cho các trường được truy vấn:

1. Trong Firebase Console, vào **Firestore Database > Indexes**.
2. Nhấn **Add Index** và cấu hình:
   - **Collection ID**: Tên collection (ví dụ: `messages`).
   - **Field Path**: Trường cần truy vấn (ví dụ: `recipientId`, `timestamp`).
   - **Order**: Chọn `Ascending` hoặc `Descending` tùy theo `orderBy`.
3. Ví dụ cho ứng dụng chat:
   - Nếu truy vấn: `where('recipientId', '==', '1')` và `orderBy('timestamp')`, tạo index với:
     - **Collection**: `messages`
     - **Fields**:
       - `recipientId` (Ascending)
       - `timestamp` (Ascending)
4. Nhấn **Create** và đợi Firebase xử lý (có thể mất vài phút).

## Tích Hợp Firestore Trong Ứng Dụng React

Dưới đây là ví dụ mã React để xây dựng một ứng dụng chat đơn giản sử dụng Firestore với tính năng realtime.

### Mã Ví Dụ

```tsx
import React, { useState, useEffect } from "react";
import {
  collection,
  addDoc,
  onSnapshot,
  query,
  orderBy,
  where,
} from "firebase/firestore";
import { firestore } from "@/firebase-app/firebase-auth"; // Đường dẫn đến file cấu hình Firebase

const ChatApp = () => {
  const [messages, setMessages] = useState<any[]>([]);
  const [newMessage, setNewMessage] = useState("");

  // Lắng nghe dữ liệu realtime từ Firestore
  useEffect(() => {
    const q = query(
      collection(firestore, "messages"),
      where("recipientId", "==", "1"),
      orderBy("timestamp")
    );

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const messagesData: any[] = [];
      snapshot.forEach((doc) =>
        messagesData.push({ ...doc.data(), id: doc.id })
      );
      setMessages(messagesData);
    });

    // Hủy lắng nghe khi component unmount
    return () => unsubscribe();
  }, []);

  // Gửi tin nhắn mới
  const handleSendMessage = async () => {
    if (!newMessage.trim()) {
      alert("Vui lòng nhập nội dung tin nhắn!");
      return;
    }

    try {
      await addDoc(collection(firestore, "messages"), {
        text: newMessage,
        timestamp: new Date(),
        recipientId: "1",
        senderId: "2",
      });
      setNewMessage(""); // Xóa ô nhập liệu sau khi gửi
    } catch (error) {
      console.error("Lỗi khi gửi tin nhắn:", error);
      alert("Đã xảy ra lỗi khi gửi tin nhắn.");
    }
  };

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Ứng Dụng Chat Realtime</h2>
      <div className="border p-4 mb-4 h-64 overflow-y-auto">
        {messages.map((message) => (
          <div key={message.id} className="mb-2">
            {message.text}
          </div>
        ))}
      </div>
      <div className="flex">
        <input
          type="text"
          value={newMessage}
          onChange={(e) => setNewMessage(e.target.value)}
          className="border p-2 flex-1 mr-2"
          placeholder="Nhập tin nhắn..."
        />
        <button
          onClick={handleSendMessage}
          className="bg-blue-500 text-white p-2 rounded"
        >
          Gửi
        </button>
      </div>
    </div>
  );
};

export default ChatApp;
```

### Giải Thích Mã

- **State**:
  - `messages`: Lưu danh sách tin nhắn từ Firestore.
  - `newMessage`: Lưu nội dung tin nhắn người dùng nhập.
- **Realtime Listener** (`useEffect`):
  - Sử dụng `onSnapshot` để lắng nghe thay đổi trong collection `messages`.
  - Truy vấn với `where('recipientId', '==', '1')` để lọc tin nhắn và `orderBy('timestamp')` để sắp xếp theo thời gian.
  - Mỗi khi có thay đổi, cập nhật state `messages` với dữ liệu mới.
  - Hủy lắng nghe (`unsubscribe`) khi component unmount để tránh rò rỉ bộ nhớ.
- **Gửi Tin Nhắn** (`handleSendMessage`):
  - Kiểm tra tin nhắn không rỗng.
  - Thêm tài liệu mới vào collection `messages` với các trường `text`, `timestamp`, `recipientId`, `senderId`.
  - Xóa ô nhập liệu sau khi gửi thành công.
- **Giao Diện**:
  - Hiển thị danh sách tin nhắn trong một khu vực có thể cuộn.
  - Ô nhập liệu và nút gửi để thêm tin nhắn mới.

### Lưu Ý

- **Cấu hình Firebase**: Đảm bảo file `firebase-auth.ts` được cấu hình đúng với thông tin dự án Firebase (Firestore SDK).
- **Bảo mật rules**: Trong môi trường production, cần giới hạn quyền đọc/ghi dựa trên người dùng (ví dụ: chỉ cho phép người gửi đọc tin nhắn của họ).
- **Xử lý lỗi**: Thêm thông báo lỗi rõ ràng cho người dùng khi gửi tin nhắn thất bại.
- **Tối ưu hiệu suất**: Nếu collection `messages` lớn, cân nhắc phân trang hoặc giới hạn số lượng tin nhắn tải về.

### Mở Rộng

- **Hiển thị thời gian**: Thêm định dạng thời gian cho tin nhắn (`message.timestamp`).
- **Hỗ trợ nhiều người dùng**: Thay `recipientId` và `senderId` bằng ID người dùng thực (từ Firebase Authentication).
- **Thông báo qua Telegram**: Kết hợp với bot Telegram (như hướng dẫn trước) để gửi thông báo khi có tin nhắn mới.

Với hướng dẫn này, bạn có thể xây dựng một ứng dụng chat realtime sử dụng Firestore và tích hợp vào ứng dụng React một cách dễ dàng.
