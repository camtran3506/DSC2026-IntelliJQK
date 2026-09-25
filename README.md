# Chỉnh sửa file này nè, viết bằng tiếng anh

Trong đó folder đang để tượng trưng, cache weights và experiments, file selected-contexts và processed-contexts sẽ nằm trong gitignore nên m cũng phải mô tả đoạn này, mỗi folder có vai trò gì.

## Cách để chạy notebook:

cd DSC2026-IntelliJQK

pip install -r requirements_task1.txt

git clone https://github.com/anhduc1526/DSC-preprocessing.git

cd DSC-preprocessing

python main.py (Nếu bước này bị lỗi đường dẫn báo t)

cd notebook

jupyter nbconvert --to script "dsc2026_77.ipynb"

cd ..

nohup python -u notebook/dsc2026_77.py > logs/dsc2026_77.log 2>&1 &