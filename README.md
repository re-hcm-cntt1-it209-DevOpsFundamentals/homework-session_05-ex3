Bài 3: Xử lý xung đột phức tạp trong quá trình Rebase

1. Mục tiêu

Thực hành xử lý xung đột khi sử dụng Rebase trong Git.

Các mục tiêu chính:

Hiểu sự khác biệt giữa giải quyết xung đột khi Merge và khi Rebase.

Thực hiện quy trình giải quyết conflict từng bước trong quá trình Rebase.

Đưa lịch sử của nhánh feature-api lên phía trên nhánh main.

Tạo lịch sử Git thẳng, không phát sinh merge commit.

2. Bối cảnh bài tập

Ban đầu, nhánh main và nhánh feature-api cùng bắt đầu từ một commit.

Sau đó:

Nhánh feature-api có 2 commit:

feat: change port

feat: enable debug

Nhánh main có 2 commit:

update port on main

add env config

Hai nhánh cùng sửa file config.json, trong đó cả hai bên đều thay đổi giá trị port.

Vì vậy, khi thực hiện:

git rebase main

Git phải áp dụng lần lượt các commit của feature-api lên lịch sử mới nhất của main, dẫn đến conflict.

3. Các commit ban đầu

Commit đầu tiên trên main

Tạo file config.json:

{
  "port": 8080,
  "debug": false
}

Commit:

git add config.json
git commit -m "init config"

4. Tạo nhánh feature-api

Tạo nhánh mới từ main:

git checkout -b feature-api

Commit 1 trên feature-api

Thay đổi port từ 8080 thành 9000:

{
  "port": 9000,
  "debug": false
}

Commit:

git add config.json
git commit -m "feat: change port"

Commit 2 trên feature-api

Tiếp tục bật debug:

{
  "port": 9000,
  "debug": true
}

Commit:

git add config.json
git commit -m "feat: enable debug"

Kiểm tra lịch sử:

git log --oneline

Lịch sử trên feature-api lúc này có dạng:

<hash> feat: enable debug
<hash> feat: change port
<hash> init config

Hash thực tế phụ thuộc vào repository trên máy.

5. Tạo thay đổi mới trên main

Chuyển về main:

git checkout main

Commit 1 trên main

Thay đổi port thành 8081:

{
  "port": 8081,
  "debug": false
}

Commit:

git add config.json
git commit -m "update port on main"

Commit 2 trên main

Thêm cấu hình môi trường:

{
  "port": 8081,
  "debug": false,
  "env": "production"
}

Commit:

git add config.json
git commit -m "add env config"

Kiểm tra lịch sử:

git log --oneline

Lịch sử trên main lúc này có dạng:

<hash> add env config
<hash> update port on main
<hash> init config

6. Trạng thái branch trước khi Rebase

Chuyển lại feature-api:

git checkout feature-api

Kiểm tra toàn bộ lịch sử:

git log --graph --oneline --all

Lúc này lịch sử có dạng:

* <hash> feat: enable debug
* <hash> feat: change port
| * <hash> add env config
| * <hash> update port on main
|/
* <hash> init config

Hai nhánh đã phát triển theo hai hướng khác nhau.

7. Thực hiện Rebase

Đứng trên nhánh feature-api và chạy:

git rebase main

Git sẽ lần lượt lấy các commit của feature-api và đặt chúng lên trên commit mới nhất của main.

Trong quá trình này, conflict có thể xảy ra khi Git áp dụng commit:

feat: change port

Git có thể hiển thị:

CONFLICT (content): Merge conflict in config.json
error: could not apply <hash>... feat: change port

Kiểm tra trạng thái:

git status

8. Xử lý Conflict lần 1

Mở file config.json:

cat config.json

Git sẽ chèn các conflict marker tương tự:

<<<<<<< HEAD
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
=======
{
  "port": 9000,
  "debug": false
}
>>>>>>> <hash>

Phân tích conflict

Thay đổi của main:

{
  "port": 8081,
  "debug": false,
  "env": "production"
}

Thay đổi của feature-api:

{
  "port": 9000,
  "debug": false
}

Trong bài thực hành này, kết quả mong muốn là:

{
  "port": 9000,
  "debug": false,
  "env": "production"
}

Điều này có nghĩa:

Giữ port: 9000 từ feature-api.

Giữ debug: false tại chặng này.

Giữ env: production từ main.

Loại bỏ toàn bộ conflict marker.

Có thể sửa bằng:

cat > config.json <<'EOF'
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
EOF

Kiểm tra lại:

cat config.json

Đánh dấu conflict đã được giải quyết:

git add config.json

Tiếp tục Rebase:

git rebase --continue

Nếu Git mở trình soạn thảo để xác nhận commit message, giữ nguyên:

feat: change port

Sau đó lưu và thoát.

9. Xử lý Conflict lần 2

Git tiếp tục áp dụng commit tiếp theo:

feat: enable debug

Trường hợp Git phát sinh conflict

Nếu Git báo:

CONFLICT (content): Merge conflict in config.json

Kiểm tra:

git status

Mở file:

cat config.json

Sau khi giải quyết conflict, kết quả mong muốn là:

{
  "port": 9000,
  "debug": true,
  "env": "production"
}

Sửa file:

cat > config.json <<'EOF'
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
EOF

Kiểm tra:

cat config.json

Đánh dấu đã giải quyết:

git add config.json

Tiếp tục:

git rebase --continue

Nếu Git mở trình soạn thảo để xác nhận commit message, giữ:

feat: enable debug

10. Trường hợp không phát sinh Conflict lần 2

Việc conflict có xảy ra ở commit thứ hai hay không phụ thuộc vào cách Git áp dụng patch của commit đó.

Trong bài này:

Commit feat: change port chắc chắn có khả năng conflict vì main cũng thay đổi port.

Commit feat: enable debug chỉ thay đổi debug, trong khi thay đổi mới nhất của main thêm env.

Vì vậy, sau khi giải quyết conflict lần 1, Git có thể áp dụng commit thứ hai mà không cần conflict lần nữa.

Nếu Git báo:

Successfully rebased and updated refs/heads/feature-api.

thì Rebase đã hoàn tất.

Không cần cố tạo thêm conflict giả.

11. Kiểm tra sau khi Rebase

Kiểm tra trạng thái

git status

Kết quả mong đợi:

On branch feature-api
nothing to commit, working tree clean

Kiểm tra file config.json

cat config.json

Kết quả cuối cùng:

{
  "port": 9000,
  "debug": true,
  "env": "production"
}

12. Kiểm tra lịch sử sau Rebase

Chạy:

git log --graph --oneline

Lịch sử mong đợi có dạng:

* <new-hash> feat: enable debug
* <new-hash> feat: change port
* <hash> add env config
* <hash> update port on main
* <hash> init config

Điểm quan trọng:

Các commit của feature-api nằm phía trên các commit mới nhất của main.

Lịch sử được nối thẳng.

Không có merge commit.

Sơ đồ:

A -- B -- C -- D -- E' -- F'

Trong đó:

A -- B: lịch sử chung ban đầu.

C -- D: các commit mới của main.

E' -- F': các commit của feature-api sau khi được Rebase.

13. Kiểm tra toàn bộ branch

Kiểm tra branch:

git branch

Kết quả:

* feature-api
  main

Kiểm tra toàn bộ lịch sử:

git log --graph --oneline --all

Sau Rebase, feature-api nằm trên phần lịch sử mới nhất của main.

14. So sánh Merge và Rebase

Merge

Merge kết hợp hai nhánh và thường tạo thêm một merge commit:

A -- B -- C -------- M
     \              /
      D -- E -------

Rebase

Rebase đưa các commit của nhánh hiện tại lên trên nhánh đích:

A -- B -- C -- D -- E'

Vì vậy, Rebase giúp lịch sử Git thẳng và dễ đọc hơn.

15. Kết quả đạt được

Sau khi hoàn thành bài:

Tạo được nhánh feature-api từ main.

Tạo các commit thay đổi config.json trên feature-api.

Tạo các commit thay đổi config.json trên main.

Thực hiện git rebase main.

Xử lý conflict thủ công trong config.json.

Sử dụng git add để đánh dấu conflict đã được giải quyết.

Sử dụng git rebase --continue để tiếp tục quá trình Rebase.

Giữ được các thay đổi cần thiết từ cả main và feature-api.

Tạo được lịch sử commit thẳng, không có merge commit.

16. Các lệnh Git đã sử dụng

git init
git branch -M main
git checkout -b feature-api
git checkout main
git checkout feature-api

git add config.json
git commit -m "init config"
git commit -m "feat: change port"
git commit -m "feat: enable debug"
git commit -m "update port on main"
git commit -m "add env config"

git log --oneline
git log --graph --oneline --all

git rebase main
git status
git add config.json
git rebase --continue

17. Screenshot minh họa

Conflict lần 1

Chèn ảnh chụp màn hình khi Git báo conflict:

![Conflict lần 1](./images/conflict-1.png)

Conflict lần 2

Nếu thực tế có phát sinh conflict lần 2:

![Conflict lần 2](./images/conflict-2.png)

Lịch sử sau Rebase

Chèn ảnh chụp kết quả:

git log --graph --oneline

![Git log sau Rebase](./images/git-log-rebase.png)

18. Cấu trúc thư mục

Sau khi hoàn thành:

bai3/
├── .git/
├── README.md
├── config.json
└── images/
    ├── conflict-1.png
    ├── conflict-2.png
    └── git-log-rebase.png

Nếu không có conflict lần 2 thì không cần ảnh conflict-2.png.

19. Kết luận

Qua bài tập này, có thể thấy Rebase không đơn giản chỉ là đưa một nhánh lên trên nhánh khác. Khi lịch sử hai nhánh cùng thay đổi một phần của cùng một file, Git có thể dừng lại để yêu cầu xử lý conflict.

Quy trình cơ bản khi Rebase gặp conflict là:

git rebase main
        ↓
   conflict
        ↓
sửa file thủ công
        ↓
git add <file>
        ↓
git rebase --continue
        ↓
tiếp tục cho đến khi hoàn tất

Kết quả cuối cùng là nhánh feature-api được đặt lên trên lịch sử mới nhất của main, đồng thời lịch sử Git vẫn thẳng và không tạo merge commit.