# Chỉnh sửa file này nè, viết bằng tiếng anh

Trong đó folder đang để tượng trưng, cache weights và experiments, file selected-contexts và processed-contexts sẽ nằm trong gitignore nên m cũng phải mô tả đoạn này, mỗi folder có vai trò gì.

# Cách để chạy notebook áp dụng với máy mới, ko phải server của mình. M mà chạy trên server là lỗi.
## Chạy task 1

cd DSC2026-IntelliJQK

Tạo env riêng cho task 1.

hf download...(vì notebook đang tắt mạng, nếu bật lại thì notebook sẽ tự tải model trên hugging face)

pip install -r requirements_task1.txt

git clone https://github.com/anhduc1526/DSC-preprocessing.git

python DSC-preprocessing/main.py

cd notebook

jupyter nbconvert --to script "dsc2026_77.ipynb"

cd ..

nohup python -u notebook/dsc2026_77.py > logs/dsc2026_77.log 2>&1 & (ghi dsc2026_77.log để không ghi đè run_77.log có sẵn)

## Chạy task 2

cd DSC2026-IntelliJQK

Tạo env riêng cho task 2.

hf download...(vì notebook đang tắt mạng, nếu bật lại thì notebook sẽ tự tải model trên hugging face)

pip install -r requirements_task2.txt

cd notebook

jupyter nbconvert --to script "7task2_vileg17.ipynb"

cd ..

nohup python -u notebook/7task2_vileg17.py > logs/7task2_vileg17.log 2>&1 & (ghi 7task2_vileg17.log để không ghi đè run_vileg17.log có sẵn)

(Notebook task 2 có thể chạy training cho stage 2 hoặc không, tuy nhiên khuyến khích chạy train riêng ở task 1 rồi task 2 chỉ load model đã train)
