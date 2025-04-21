# Hướng Dẫn Sử Dụng Git và Quy Trình Gitflow Tùy Chỉnh

Hướng dẫn này cung cấp danh sách các lệnh Git phổ biến, bao gồm các lệnh nâng cao như `cherry-pick`, mô tả chi tiết cách sử dụng và giải thích quy trình **Gitflow** tùy chỉnh, trong đó các nhánh mới được tạo từ nhánh `production` thay vì `develop`.

## Các Lệnh Git Phổ Biến và Nâng Cao

### 1. Khởi Tạo Repository

- **Lệnh**: `git init`

- **Mô tả**: Khởi tạo một repository Git mới, tạo thư mục `.git` để theo dõi lịch sử thay đổi.

- **Ví dụ**:

  ```bash
  git init
  ```

### 2. Clone Dự Án Từ Repository

- **Lệnh**: `git clone <repository-url>`

- **Mô tả**: Tải một bản sao của repository từ remote về máy cục bộ.

- **Ví dụ**:

  ```bash
  git clone https://gitlab.com/duchaidev/clientcms.git
  ```

### 3. Thêm Tệp Để Chuẩn Bị Commit

- **Lệnh**: `git add <file-name>` hoặc `git add .`

- **Mô tả**:

  - `git add <file-name>`: Thêm tệp cụ thể vào staging area.
  - `git add .`: Thêm tất cả các tệp đã thay đổi.

- **Ví dụ**:

  ```bash
  git add src/index.js
  git add .
  ```

### 4. Tạo Commit Với Message

- **Lệnh**: `git commit -m "Commit message"`

- **Mô tả**: Lưu các thay đổi trong staging area vào lịch sử commit với thông điệp rõ ràng.

- **Ví dụ**:

  ```bash
  git commit -m "FEATURE: permission SSO"
  ```

### 5. Đẩy Code Lên Remote Repository

- **Lệnh**: `git push origin <branch-name>`

- **Mô tả**: Đẩy các commit từ nhánh cục bộ lên nhánh remote.

- **Ví dụ**:

  ```bash
  git push origin duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 6. Tạo Nhánh Mới

- **Lệnh**: `git branch <branch-name>`

- **Mô tả**: Tạo một nhánh mới.

- **Ví dụ**:

  ```bash
  git branch duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 7. Chuyển Sang Nhánh Khác

- **Lệnh**: `git checkout <branch-name>`

- **Mô tả**: Chuyển sang nhánh được chỉ định.

- **Ví dụ**:

  ```bash
  git checkout duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 8. Tạo Và Chuyển Sang Nhánh Mới

- **Lệnh**: `git checkout -b <branch-name>`

- **Mô tả**: Tạo và chuyển sang nhánh mới.

- **Ví dụ**:

  ```bash
  git checkout -b duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 9. Xóa Nhánh Cục Bộ

- **Lệnh**: `git branch -d <branch-name>`

- **Mô tả**: Xóa nhánh cục bộ sau khi đã merge.

- **Ví dụ**:

  ```bash
  git branch -d duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 10. Merge Nhánh

- **Lệnh**: `git merge <branch-name>`

- **Mô tả**: Hợp nhất code từ nhánh được chỉ định vào nhánh hiện tại.

- **Ví dụ**:

  ```bash
  git checkout develop
  git merge duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 11. Lấy Code Từ Remote Repository

- **Lệnh**: `git pull`

- **Mô tả**: Kéo các thay đổi mới nhất từ nhánh remote về máy cục bộ.

- **Ví dụ**:

  ```bash
  git pull origin develop
  ```

### 12. Kiểm Tra Trạng Thái Repository

- **Lệnh**: `git status`

- **Mô tả**: Hiển thị trạng thái hiện tại của repository.

- **Ví dụ**:

  ```bash
  git status
  ```

### 13. Xem Lịch Sử Commit

- **Lệnh**: `git log`

- **Mô tả**: Hiển thị lịch sử commit.

- **Ví dụ**:

  ```bash
  git log --oneline
  ```

### 14. Hủy Thay Đổi Chưa Commit

- **Lệnh**:

  - `git restore <file-name>`: Hủy thay đổi chưa add.
  - `git restore --staged <file-name>`: Xóa tệp khỏi staging area.

- **Ví dụ**:

  ```bash
  git restore src/index.js
  ```

### 15. So Sánh Sự Khác Biệt

- **Lệnh**: `git diff`

- **Mô tả**: Hiển thị sự khác biệt giữa các thay đổi chưa commit.

- **Ví dụ**:

  ```bash
  git diff
  ```

### 16. Lưu Tạm Thay Đổi (Stash)

- **Lệnh**:

  - `git stash`: Lưu tạm thay đổi chưa commit.
  - `git stash pop`: Khôi phục thay đổi đã stash.

- **Ví dụ**:

  ```bash
  git stash
  git stash pop
  ```

### 17. Xóa Nhánh Trên Remote

- **Lệnh**: `git push origin --delete <branch-name>`

- **Mô tả**: Xóa nhánh trên remote repository.

- **Ví dụ**:

  ```bash
  git push origin --delete duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  ```

### 18. Cherry-pick Commit

- **Lệnh**: `git cherry-pick <commit-hash>`

- **Mô tả**: Áp dụng một commit cụ thể từ nhánh khác vào nhánh hiện tại.

- **Ví dụ**:

  ```bash
  git checkout develop
  git cherry-pick a1b2c3d
  ```

- **Trường hợp sử dụng**: Khi cần áp dụng một sửa lỗi hoặc tính năng cụ thể mà không merge toàn bộ nhánh.

### 19. Rebase Nhánh

- **Lệnh**: `git rebase <branch-name>`

- **Mô tả**: Chuyển các commit của nhánh hiện tại lên trên cùng của nhánh được chỉ định.

- **Ví dụ**:

  ```bash
  git checkout duchaidev/feature-kaizen-dashboard-lap-ke-hoach
  git rebase develop
  ```

- **Lưu ý**: Giải quyết xung đột nếu có và chạy `git rebase --continue`.

### 20. Reset Commit

- **Lệnh**:

  - `git reset --soft <commit-hash>`: Hủy commit nhưng giữ thay đổi.
  - `git reset --hard <commit-hash>`: Hủy commit và xóa thay đổi.

- **Ví dụ**:

  ```bash
  git reset --soft HEAD~1
  ```

## Quy Trình Gitflow Tùy Chỉnh

Quy trình Gitflow này được tùy chỉnh để tất cả các nhánh mới (tính năng, hotfix) được tạo từ nhánh `production`, thay vì `develop` như Gitflow truyền thống. Quy trình này phù hợp với các dự án ưu tiên code ổn định từ production và tích hợp dần vào develop trước khi phát hành.

### Các Nhánh Chính

1. **production**:

   - Chứa code ổn định, đang chạy trên môi trường production.
   - Là nhánh gốc để tạo tất cả các nhánh mới (`feature`, `hotfix`).
   - Nhận merge từ nhánh `develop` hoặc `hotfix` khi phát hành hoặc sửa lỗi.

2. **develop**:

   - Chứa code tích hợp các tính năng mới hoặc sửa lỗi từ các nhánh `feature` và `hotfix`.
   - Được sử dụng để kiểm tra và chuẩn bị trước khi merge vào `production`.

3. **feature**:

   - Tên nhánh: `<tên-người-thực-hiện>/feature-<tên-tính-năng>` (ví dụ: `duchaidev/feature-kaizen-dashboard-lap-ke-hoach`).
   - Dùng để phát triển tính năng mới.
   - Tạo từ `production`, rebase với `develop` trước khi đẩy code, và merge vào `develop` sau khi hoàn thành.

4. **hotfix**:

   - Tên nhánh: `<tên-người-thực-hiện>/hotfix-<mô-tả-lỗi>` (ví dụ: `duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard`).
   - Dùng để sửa lỗi khẩn cấp trên production.
   - Tạo từ `production`, rebase với `production` trước khi đẩy code, và merge vào cả `production` và `develop`.

### Quy Trình Làm Việc

1. **Tạo Nhánh Mới**:

   - Tất cả nhánh (`feature`, `hotfix`) được tạo từ nhánh `production`.

   - Ví dụ tạo nhánh tính năng:

     ```bash
     git checkout production
     git checkout -b duchaidev/feature-kaizen-dashboard-lap-ke-hoach
     ```

2. **Rebase Trước Khi Đẩy Code**:

   - **Nhánh** `feature`:

     - Trước khi đẩy code, rebase với nhánh `develop` để đảm bảo nhánh tính năng đồng bộ với các thay đổi mới nhất:

       ```bash
       git checkout duchaidev/feature-kaizen-dashboard-lap-ke-hoach
       git rebase develop
       ```

     - Nếu có xung đột, giải quyết và tiếp tục:

       ```bash
       git rebase --continue
       ```

     - Đẩy code:

       ```bash
       git push origin duchaidev/feature-kaizen-dashboard-lap-ke-hoach --force
       ```

   - **Nhánh** `hotfix`:

     - Rebase với nhánh `production`:

       ```bash
       git checkout duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard
       git rebase production
       ```

     - Đẩy code:

       ```bash
       git push origin duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard --force
       ```

3. **Quy Tắc Đặt Tên Nhánh**:

   - Format: `<tên-người-thực-hiện>/<loại-công-việc>-<tên-tính-năng-hoặc-mô-tả-lỗi>`
   - Loại công việc:
     - `feature`: Phát triển tính năng mới.
     - `hotfix`: Sửa lỗi trên production.
   - Ví dụ:
     - Tính năng mới: `duchaidev/feature-kaizen-dashboard-lap-ke-hoach`
     - Sửa lỗi: `duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard`

4. **Quy Định Fix Bug**:

   - **Bug trên môi trường test**: Tiếp tục sử dụng nhánh `feature` gốc (ví dụ: `duchaidev/feature-kaizen-dashboard-lap-ke-hoach`) để sửa bug liên quan đến tính năng mới. Chỉ xóa nhánh khi code đã được merge vào `production`.
   - **Bug trên production**: Tạo nhánh `hotfix` từ `production` để sửa lỗi.

5. **Quy Tắc Đặt Tên Commit**:

   - Commit phải rõ ràng, nêu loại commit (FEATURE, FIX, v.v.), mô tả tính năng hoặc lỗi, và màn hình liên quan.

   - Tránh các commit chung chung như "fix", "update".

   - Ví dụ:

     ```bash
     git commit -m "FEATURE: permission SSO"
     git commit -m "FIX: selected date dashboard KLMT"
     ```

6. **Merge Code**:

   - **Nhánh** `feature`:

     - Sau khi hoàn thành, tạo merge request từ nhánh `feature` vào `develop`:

       ```bash
       git checkout develop
       git merge duchaidev/feature-kaizen-dashboard-lap-ke-hoach
       git push origin develop
       ```

   - **Nhánh** `hotfix`:

     - Merge vào cả `production` và `develop`:

       ```bash
       git checkout production
       git merge duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard
       git push origin production
       git checkout develop
       git merge duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard
       git push origin develop
       ```

   - Xóa nhánh sau khi merge:

     ```bash
     git push origin --delete duchaidev/feature-kaizen-dashboard-lap-ke-hoach
     ```

7. **Phát Hành**:

   - Khi `develop` đã tích hợp đủ tính năng và ổn định, merge vào `production`:

     ```bash
     git checkout production
     git merge develop
     git push origin production
     ```

   - Đánh tag phiên bản (nếu cần):

     ```bash
     git tag v1.0.0
     git push origin v1.0.0
     ```

### Ví Dụ Quy Trình

1. **Phát triển tính năng mới**:

   ```bash
   git checkout production
   git checkout -b duchaidev/feature-kaizen-dashboard-lap-ke-hoach
   git add .
   git commit -m "FEATURE: thêm giao diện lập kế hoạch dashboard"
   git rebase develop
   git push origin duchaidev/feature-kaizen-dashboard-lap-ke-hoach --force
   ```

   - Tạo merge request vào `develop`.

2. **Sửa lỗi trên production**:

   ```bash
   git checkout production
   git checkout -b duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard
   git add .
   git commit -m "FIX: lỗi hiển thị ngày tháng dashboard KLMT"
   git rebase production
   git push origin duchaidev/hotfix-loi-hien-thi-ngay-thang-kaizen-dashboard --force
   ```

   - Merge vào `production` và `develop`.

### Lợi Ích Của Gitflow Tùy Chỉnh

- **Đồng bộ từ production**: Tạo nhánh từ `production` đảm bảo code luôn dựa trên phiên bản ổn định nhất.
- **Kiểm soát chặt chẽ**: Rebase trước khi đẩy code giúp lịch sử commit sạch sẽ và đồng bộ.
- **Phân biệt rõ ràng**: Quy tắc đặt tên nhánh và commit giúp dễ dàng theo dõi công việc của từng thành viên.

### Lưu Ý

- **Rebase cẩn thận**: Sử dụng `--force` khi đẩy code sau rebase, nhưng chỉ trên nhánh cá nhân để tránh ảnh hưởng người khác.
- **Code review**: Tạo merge request và kiểm tra code kỹ lưỡng trước khi merge vào `develop` hoặc `production`.
- **Xóa nhánh sau merge**: Giữ repository gọn gàng bằng cách xóa nhánh đã hoàn thành.
- **Bảo mật commit**: Không commit thông tin nhạy cảm (mật khẩu, token) vào repository.

Với các lệnh Git và quy trình Gitflow tùy chỉnh này, bạn có thể quản lý mã nguồn hiệu quả, tích hợp các công cụ nâng cao như cherry-pick, rebase, và tổ chức dự án một cách chuyên nghiệp.
