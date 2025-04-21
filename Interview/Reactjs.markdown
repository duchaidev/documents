# Câu Hỏi Phỏng Vấn Frontend Junior/Middle

Dưới đây là tài liệu được tối ưu hóa và bổ sung thêm các câu hỏi phỏng vấn thường gặp cho vị trí Frontend Junior/Middle, với các giải thích chi tiết, ví dụ minh họa và nội dung mở rộng.

---

## 1. Sự Khác Nhau Giữa Thẻ `<div>` và `<span>`

- **`<div>` (Division)**:

  - **Mục đích**: Tạo khối chứa (block-level) cho các phần tử giao diện.
  - **Đặc tính**: Chiếm toàn bộ chiều rộng của dòng chứa, đẩy các phần tử khác xuống dòng mới.
  - **Ứng dụng**: Dùng cho các khung lớn như header, footer, sidebar, hoặc container bố cục.
  - **Ví dụ**:
    ```html
    <div class="container">
      <div class="header">Header</div>
      <div class="content">Main Content</div>
    </div>
    ```

- **`<span>`**:
  - **Mục đích**: Bao bọc nội dung nhỏ, thường là văn bản hoặc inline elements.
  - **Đặc tính**: Inline-level, chỉ chiếm không gian của nội dung, không xuống dòng.
  - **Ứng dụng**: Dùng để định dạng văn bản trong dòng (ví dụ: highlight, style nhỏ).
  - **Ví dụ**:
    ```html
    <p>Hãy <span class="highlight">chú ý</span> đến nội dung này.</p>
    ```

---

## 2. Sự Khác Nhau Giữa `localStorage`, `sessionStorage` và `cookie`

| Tính năng             | `localStorage`                        | `sessionStorage`             | `cookie`                           |
| --------------------- | ------------------------------------- | ---------------------------- | ---------------------------------- |
| **Dung lượng**        | ~5MB                                  | ~5MB                         | ~4KB                               |
| **Thời gian tồn tại** | Vĩnh viễn (cho đến khi xóa thủ công)  | Mất khi đóng tab trình duyệt | Tùy chỉnh thời gian hết hạn        |
| **Ứng dụng**          | Lưu cài đặt người dùng, cache dữ liệu | Lưu dữ liệu tạm trong phiên  | Lưu thông tin xác thực, tracking   |
| **Truy cập**          | Chỉ client (cùng origin)              | Chỉ client (cùng origin)     | Server và client (qua HTTP header) |
| **Ví dụ**             | Lưu theme người dùng                  | Lưu form tạm thời            | Lưu token JWT                      |

**Ví dụ sử dụng**:

```javascript
// localStorage
localStorage.setItem("theme", "dark");

// sessionStorage
sessionStorage.setItem("formData", JSON.stringify({ name: "John" }));

// Cookie
document.cookie = "token=abc123; expires=Fri, 31 Dec 2025 23:59:59 GMT";
```

---

## 3. Xử Lý Xác Thực Với JWT Trong ReactJS

### Cách Hoạt Động của JWT

- JWT (JSON Web Token) gồm 3 phần: **Header**, **Payload**, **Signature**, được mã hóa Base64 và phân cách bằng dấu chấm (`.`).
- Server tạo JWT khi người dùng đăng nhập thành công và gửi về client.
- Client lưu JWT (thường trong `localStorage` hoặc `cookie`) và gửi kèm trong header của các request để xác thực.

### Xử Lý Trong ReactJS

1. **Đăng nhập và lưu token**:

   ```javascript
   const handleLogin = async (username, password) => {
     const response = await fetch("/api/login", {
       method: "POST",
       body: JSON.stringify({ username, password }),
       headers: { "Content-Type": "application/json" },
     });
     const data = await response.json();
     if (data.token) {
       localStorage.setItem("jwtToken", data.token);
     }
   };
   ```

2. **Kiểm tra token**:

   ```javascript
   useEffect(() => {
     const token = localStorage.getItem("jwtToken");
     if (token) {
       const decoded = jwtDecode(token); // Giả sử dùng thư viện jwt-decode
       if (decoded.exp * 1000 < Date.now()) {
         refreshAuthToken(); // Gọi API refresh token
       }
     }
   }, []);
   ```

3. **Gửi token trong API call**:

   ```javascript
   const fetchData = async () => {
     const token = localStorage.getItem("jwtToken");
     const response = await fetch("/api/data", {
       headers: { Authorization: `Bearer ${token}` },
     });
     return response.json();
   };
   ```

4. **Refresh token**:
   ```javascript
   const refreshAuthToken = async () => {
     const response = await fetch("/api/refresh-token");
     const data = await response.json();
     if (data.newToken) {
       localStorage.setItem("jwtToken", data.newToken);
     }
   };
   ```

**Lưu ý**: Để bảo mật, nên lưu token trong `HttpOnly cookie` thay vì `localStorage` để tránh XSS attacks.

---

## 4. Tối Ưu Hiệu Năng Trang Web

1. **Lazy Loading**:

   - Chỉ tải tài nguyên (ảnh, video) khi chúng xuất hiện trong viewport.
   - **Ví dụ**:
     ```html
     <img loading="lazy" src="image.jpg" alt="Lazy loaded image" />
     ```

2. **Code Splitting**:

   - Tách ứng dụng thành các bundle nhỏ, chỉ tải khi cần.
   - **Ví dụ**:

     ```javascript
     import { lazy, Suspense } from "react";
     const LazyComponent = lazy(() => import("./Component"));

     function App() {
       return (
         <Suspense fallback={<div>Loading...</div>}>
           <LazyComponent />
         </Suspense>
       );
     }
     ```

3. **Caching**:

   - Sử dụng `localStorage`, `sessionStorage` hoặc service worker để lưu trữ tài nguyên tĩnh.
   - **Ví dụ**:
     ```javascript
     localStorage.setItem("cachedData", JSON.stringify(data));
     ```

4. **Memoization**:

   - Dùng `React.memo`, `useMemo`, `useCallback` để tránh render/tính toán không cần thiết.
   - **Ví dụ**:
     ```javascript
     const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
     ```

5. **Debounce/Throttle**:

   - Giới hạn tần suất thực thi các sự kiện như nhập liệu hoặc cuộn.
   - **Ví dụ**:
     ```javascript
     const debounce = (fn, delay) => {
       let timeout;
       return (...args) => {
         clearTimeout(timeout);
         timeout = setTimeout(() => fn(...args), delay);
       };
     };
     const handleInput = debounce((value) => console.log(value), 500);
     ```

6. **Pre-fetching Data**:
   - Tải trước dữ liệu khi dự đoán người dùng sẽ cần.
   - **Ví dụ**:
     ```javascript
     useEffect(() => {
       const prefetchData = async () => {
         const data = await fetch("/api/next-page");
         cache.set("nextPage", data);
       };
       prefetchData();
     }, []);
     ```

---

## 5. Sự Khác Nhau Giữa `useMemo` và `useCallback`

| Hook          | Công dụng                 | Khi nào sử dụng                      |
| ------------- | ------------------------- | ------------------------------------ |
| `useMemo`     | Memoize giá trị tính toán | Tối ưu tính toán phức tạp            |
| `useCallback` | Memoize hàm callback      | Tránh tạo lại hàm khi truyền vào con |

- **Ví dụ `useMemo`**:

  ```javascript
  const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  ```

- **Ví dụ `useCallback`**:
  ```javascript
  const handleClick = useCallback(() => doSomething(value), [value]);
  ```

---

## 6. Cleanup Function Trong `useEffect`

- **Cleanup Function**: Hàm trả về trong `useEffect`, chạy khi component unmount hoặc trước khi effect chạy lại.
- **Ứng dụng**: Hủy listeners, timers, hoặc API requests để tránh memory leak.
- **Ví dụ**:
  ```javascript
  useEffect(() => {
    const timer = setInterval(() => console.log("Tick"), 1000);
    return () => clearInterval(timer); // Cleanup
  }, []);
  ```

---

## 7. Lifecycle của Component và `useEffect`

- **Component Lifecycle**:

  - **Mounting**: Component được thêm vào DOM (`componentDidMount`).
  - **Updating**: State/props thay đổi (`componentDidUpdate`).
  - **Unmounting**: Component bị xóa khỏi DOM (`componentWillUnmount`).

- **`useEffect` Lifecycle**:
  - **No dependencies** (`useEffect(() => {...})`): Chạy sau mỗi render.
  - **Empty dependencies** (`useEffect(() => {...}, [])`): Chạy một lần khi mount.
  - **Specific dependencies** (`useEffect(() => {...}, [dep])`): Chạy khi `dep` thay đổi.
  - **Cleanup**: Chạy trước effect mới hoặc khi unmount.

---

## 8. WebSocket Trong ReactJS

- **Cách hoạt động**: WebSocket thiết lập kết nối hai chiều giữa client và server, cho phép trao đổi dữ liệu thời gian thực.
- **Ứng dụng**: Chat, thông báo, cập nhật dữ liệu trực tiếp.
- **Ví dụ với Socket.IO**:

  ```javascript
  import io from "socket.io-client";

  const socket = io("http://localhost:4000");

  function Chat() {
    useEffect(() => {
      socket.on("message", (data) => console.log("Received:", data));
      return () => socket.off("message"); // Cleanup
    }, []);

    const sendMessage = (msg) => socket.emit("send_message", msg);

    return <button onClick={() => sendMessage("Hello")}>Send</button>;
  }
  ```

---

## 9. Thuật Toán Sắp Xếp Mảng (Bubble Sort)

- **Tăng dần**:

  ```javascript
  function sortAscending(arr) {
    for (let i = 0; i < arr.length - 1; i++) {
      for (let j = 0; j < arr.length - 1 - i; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]; // Swap
        }
      }
    }
    return arr;
  }
  console.log(sortAscending([5, 2, 9, 1, 5, 6])); // [1, 2, 5, 5, 6, 9]
  ```

- **Giảm dần**:
  ```javascript
  function sortDescending(arr) {
    for (let i = 0; i < arr.length - 1; i++) {
      for (let j = 0; j < arr.length - 1 - i; j++) {
        if (arr[j] < arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
  console.log(sortDescending([5, 2, 9, 1, 5, 6])); // [9, 6, 5, 5, 2, 1]
  ```

---

## 10. `Promise` vs `async/await` vs `Promise.all`

- **Promise**: Đối tượng xử lý tác vụ bất đồng bộ, tránh callback hell.
- **async/await**: Cú pháp ngắn gọn, dễ đọc hơn cho Promise.
- **Promise.all**: Chạy nhiều Promise song song, trả về kết quả khi tất cả hoàn thành.
- **Ví dụ**:
  ```javascript
  async function fetchData() {
    const promises = [fetch("/api/1"), fetch("/api/2")];
    const results = await Promise.all(promises);
    return results.map((res) => res.json());
  }
  ```

---

## 11. `Arrow Function` vs `Regular Function`

| Đặc điểm        | Arrow Function            | Regular Function              |
| --------------- | ------------------------- | ----------------------------- |
| **Cú pháp**     | Ngắn gọn (`() => {}`)     | Dài hơn (`function() {}`)     |
| **This**        | Kế thừa từ scope cha      | Có `this` riêng, tùy ngữ cảnh |
| **Constructor** | Không dùng được           | Dùng được                     |
| **Ứng dụng**    | Callback, inline function | Phương thức, constructor      |

---

## 12. Closure Là Gì?

- **Closure**: Hàm bên trong ghi nhớ và truy cập biến của scope cha, ngay cả khi scope cha đã kết thúc.
- **Ví dụ**:
  ```javascript
  function createCounter() {
    let count = 0;
    return () => count++;
  }
  const counter = createCounter();
  console.log(counter()); // 1
  console.log(counter()); // 2
  ```

---

## 13. Virtual DOM Trong React

- **Virtual DOM**: Bản sao của DOM thật, giúp React tối ưu hóa cập nhật giao diện.
- **Cách hoạt động**:
  1. Khi state/props thay đổi, React tạo Virtual DOM mới.
  2. So sánh (diff) với Virtual DOM cũ.
  3. Chỉ cập nhật các thay đổi vào DOM thật (reconciliation).
- **Lợi ích**: Giảm thao tác trực tiếp với DOM, tăng hiệu năng.

---

## 14. Tối Ưu Hiển Thị 1000 Sản Phẩm

1. **Pagination**: Chia danh sách thành các trang (ví dụ: 20 sản phẩm/trang).
2. **Infinite Scroll**: Tải thêm sản phẩm khi cuộn.
3. **Virtualization** (thư viện `react-window`): Chỉ render sản phẩm trong viewport.

   ```javascript
   import { FixedSizeList } from "react-window";

   function ProductList({ products }) {
     const Row = ({ index, style }) => (
       <div style={style}>{products[index].name}</div>
     );

     return (
       <FixedSizeList
         height={400}
         width={300}
         itemCount={products.length}
         itemSize={35}
       >
         {Row}
       </FixedSizeList>
     );
   }
   ```

4. **Memoization**: Dùng `React.memo` để tránh render lại sản phẩm không thay đổi.

---

## 15. `useReducer` Là Gì?

- **Định nghĩa**: Hook quản lý state phức tạp bằng reducer, tương tự Redux.
- **Khi dùng**: Khi state có nhiều trường hoặc logic cập nhật phức tạp.
- **Ví dụ**:

  ```javascript
  const initialState = { count: 0 };

  function reducer(state, action) {
    switch (action.type) {
      case "increment":
        return { count: state.count + 1 };
      case "decrement":
        return { count: state.count - 1 };
      default:
        return state;
    }
  }

  function Counter() {
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
      <div>
        <p>Count: {state.count}</p>
        <button onClick={() => dispatch({ type: "increment" })}>+</button>
        <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      </div>
    );
  }
  ```

---

## 16. `display: none` vs `visibility: hidden` vs `opacity: 0`

| Thuộc tính           | Hiệu ứng                             | Không gian | Tương tác |
| -------------------- | ------------------------------------ | ---------- | --------- |
| `display: none`      | Ẩn hoàn toàn, không chiếm không gian | Không      | Không     |
| `visibility: hidden` | Ẩn nhưng giữ không gian              | Có         | Không     |
| `opacity: 0`         | Trong suốt, giữ không gian           | Có         | Có        |

---

## 17. Grid vs Flexbox

- **Flexbox**:
  - Layout một chiều (hàng hoặc cột).
  - Dùng cho bố cục đơn giản, nội dung động động.
- **Grid**:
  - Layout hai chiều (hàng và cột).
  - Dùng cho bố cục phức tạp, dạng lưới.
- **Khi dùng**:
  - Flex: Menu, card, danh sách ngang.
  - Grid: Dashboard, bố cục toàn trang.

---

## 18. Xử Lý 10 API Call Liên Tiếp

- **Tuần tự**:

  ```javascript
  async function callAPIsSequentially() {
    for (let i = 0; i < 10; i++) {
      await fetch(`/api/${i}`);
    }
  }
  ```

- **Song song**:
  ```javascript
  async function callAPIsConcurrently() {
    const promises = Array.from({ length: 10 }, (_, i) => fetch(`/api/${i}`));
    await Promise.all(promises);
  }
  ```

---

## 19. Unmount Component

- **Unmount**: Component bị xóa khỏi DOM, cleanup trong `useEffect` được gọi.
- **Ví dụ**:
  ```javascript
  useEffect(() => {
    console.log("Mounted");
    return () => console.log("Unmounted");
  }, []);
  ```

---

## 20. Reconciliation Trong React

- **Reconciliation**: Quy trình so sánh Virtual DOM với DOM thật để cập nhật tối ưu.
- **Lợi ích**: Giảm thao tác trực tiếp trên DOM, tăng hiệu năng.

---

## 21. `useEffect` vs `useLayoutEffect`

- **`useEffect`**: Chạy sau khi DOM cập nhật, không gây lag giao diện.
- **`useLayoutEffect`**: Chạy trước khi trình duyệt vẽ giao diện, dùng cho thay đổi layout.
- **Ví dụ**:
  ```javascript
  useLayoutEffect(() => {
    const element = document.getElementById("myElement");
    element.style.width = "100px";
  }, []);
  ```

---

## 22. React vs Next.js

| Đặc điểm      | React       | Next.js                   |
| ------------- | ----------- | ------------------------- |
| **Loại**      | Thư viện    | Framework                 |
| **Rendering** | Client-side | SSR, SSG, CSR             |
| **SEO**       | Kém         | Tốt (hỗ trợ SSR, SSG)     |
| **Ứng dụng**  | SPA         | Web tối ưu SEO, hiệu năng |

---

## 23. In Số Từ 10 Về 0 Mỗi Giây

```javascript
function countdown() {
  let count = 10;
  const interval = setInterval(() => {
    console.log(count);
    count--;
    if (count < 0) clearInterval(interval);
  }, 1000);
}
countdown();
```

---

## 24. Gitflow

### Các Nhánh Chính

1. **main**: Code ổn định, sẵn sàng cho production.
2. **develop**: Tích hợp tính năng mới, chuẩn bị phát hành.
3. **feature**: Phát triển tính năng, tạo từ `main`, merge về `main`.
4. **release**: Chuẩn bị phát hành, tạo từ `develop`, merge vào `main` và `develop`.
5. **hotfix**: Sửa lỗi khẩn cấp, tạo từ `main`, merge vào `main` và `develop`.

### Quy Trình Hotfix

```bash
git checkout main
git checkout -b hotfix/fix-issue
git add .
git commit -m "Hotfix: Fix issue"
git checkout main
git merge hotfix/fix-issue
git checkout develop
git merge hotfix/fix-issue
git push origin --delete hotfix/fix-issue
```

---

## 25. Dynamic Import

- **Dynamic Import**: Tải module theo yêu cầu, hỗ trợ code splitting.
- **Ví dụ**:
  ```javascript
  const loadModule = async () => {
    const module = await import("./myModule");
    module.default();
  };
  ```

---

## 26. Scope

- **Scope**: Phạm vi mà biến được định nghĩa và truy cập.
  - **Global Scope**: Biến toàn cục, truy cập khắp nơi.
  - **Function Scope**: Biến chỉ tồn tại trong hàm.
  - **Block Scope**: Biến trong khối `{}` (dùng `let`, `const`).

---

## 27. Tham Trị vs Tham Chiếu

- **Tham trị**: Sao chép giá trị (số, chuỗi, boolean).
  ```javascript
  let a = 5;
  let b = a;
  b = 10;
  console.log(a); // 5
  ```
- **Tham chiếu**: Tham chiếu đến cùng một đối tượng (object, array).
  ```javascript
  let arr = [1, 2];
  let arr2 = arr;
  arr2.push(3);
  console.log(arr); // [1, 2, 3]
  ```

---

## 28. Mock API

- **Mock API**: Tạo API giả để phát triển/test mà không cần backend thật.
- **Công cụ**: JSON Server, Mock Service Worker (MSW).
- **Ví dụ với JSON Server**:
  ```bash
  npm install -g json-server
  json-server --watch db.json
  ```
  ```json
  // db.json
  {
    "users": [{ "id": 1, "name": "John" }]
  }
  ```

---

## 29. Tìm Phần Tử Xuất Hiện Nhiều Nhất Trong Mảng

```javascript
function mostFrequentItem(arr) {
  const frequency = {};
  let maxCount = 0;
  let mostFrequent;

  for (const item of arr) {
    frequency[item] = (frequency[item] || 0) + 1;
    if (frequency[item] > maxCount) {
      maxCount = frequency[item];
      mostFrequent = item;
    }
  }

  return mostFrequent;
}

console.log(mostFrequentItem([1, 2, 2, 3, 2, 4])); // 2
```

---

## 30. Chuyển Lifecycle Class Sang Functional Component

- **componentDidMount**:

  ```javascript
  useEffect(() => {
    console.log("Mounted");
  }, []);
  ```

- **componentDidUpdate**:

  ```javascript
  useEffect(() => {
    console.log("Updated:", prop);
  }, [prop]);
  ```

- **componentWillUnmount**:
  ```javascript
  useEffect(() => {
    return () => console.log("Unmounted");
  }, []);
  ```

---

## 31. Invelop (Envelope)

- Có thể bạn muốn nói đến **"envelope"** trong lập trình, thường liên quan đến mẫu thiết kế (design pattern) hoặc cách đóng gói dữ liệu.
- Nếu ý bạn là **"envelope pattern"** trong giao tiếp API:
  - **Envelope**: Đóng gói dữ liệu trả về trong một cấu trúc cố định (ví dụ: `{ data: ..., error: ..., status: ... }`).
  - **Ví dụ**:
    ```json
    {
      "status": "success",
      "data": { "id": 1, "name": "John" },
      "error": null
    }
    ```

---

## 32. `useImperativeHandle`

- **Mục đích**: Tùy chỉnh giá trị ref của component khi dùng `forwardRef`.
- **Ví dụ**:

  ```javascript
  import { useRef, useImperativeHandle, forwardRef } from "react";

  const Input = forwardRef((props, ref) => {
    const inputRef = useRef();
    useImperativeHandle(ref, () => ({
      focus: () => inputRef.current.focus(),
    }));
    return <input ref={inputRef} />;
  });

  function App() {
    const inputRef = useRef();
    return (
      <div>
        <Input ref={inputRef} />
        <button onClick={() => inputRef.current.focus()}>Focus</button>
      </div>
    );
  }
  ```

---

## 33. `useState` Với Callback

- **Cách thực hiện**: Dùng `useEffect` để theo dõi state thay đổi.
- **Ví dụ**:

  ```javascript
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Count updated:", count);
  }, [count]);

  const increment = () => setCount((prev) => prev + 1);
  ```

---

## 34. Concurrent Mode và Suspense

- **Concurrent Mode**: Cho phép React xử lý nhiều tác vụ đồng thời, ưu tiên tác vụ quan trọng.
- **Suspense**: Trì hoãn render cho đến khi dữ liệu sẵn sàng.
- **Ví dụ**:

  ```javascript
  const LazyComponent = lazy(() => import("./Component"));

  function App() {
    return (
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    );
  }
  ```

---

## 35. Xử Lý Nhiều Suspense

- **Cách làm**: Sử dụng nhiều `Suspense` để quản lý các phần khác nhau của UI.
- **Ví dụ**:
  ```javascript
  function App() {
    return (
      <Suspense fallback={<div>Loading Header...</div>}>
        <Header />
        <Suspense fallback={<div>Loading Content...</div>}>
          <Content />
        </Suspense>
      </Suspense>
    );
  }
  ```

---

## 36. `useMemo` Có Cải Thiện Hiệu Suất?

- **Khi cần**: Tính toán phức tạp, dữ liệu lớn.
- **Khi không cần**: Tính toán đơn giản (ví dụ: `a + b`), vì `useMemo` cũng có overhead.

---

## 37. Tìm Phần Tử Xuất Hiện Nhiều Nhất (Bổ sung)

- **Cách khác** (dùng Map):

  ```javascript
  function mostFrequentItem(arr) {
    const map = new Map();
    let maxCount = 0;
    let mostFrequent;

    for (const item of arr) {
      const count = (map.get(item) || 0) + 1;
      map.set(item, count);
      if (count > maxCount) {
        maxCount = count;
        mostFrequent = item;
      }
    }

    return mostFrequent;
  }

  console.log(mostFrequentItem([1, 2, 2, 3, 2, 4])); // 2
  ```

---

Tài liệu này cung cấp các câu trả lời chi tiết, ví dụ minh họa và nội dung mở rộng, phù hợp cho phỏng vấn vị trí Frontend Junior/Middle với mức lương 20-30 triệu.
