# Dịch vụ nhận diện cảm xúc bằng AI

Dịch vụ FastAPI nhận ảnh, chạy mô hình nhận diện cảm xúc khuôn mặt bằng PyTorch và trả về cảm xúc dự đoán cùng xác suất. Spring Boot phụ trách gọi dịch vụ AI và lưu lịch sử dự đoán vào MongoDB.

## Cài đặt và chạy

```powershell
cd backend-ai
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

API chạy tại `http://localhost:8000`. Có thể xem tài liệu Swagger tại `http://localhost:8000/docs`.

## Lựa chọn mô hình

- Dịch vụ sử dụng mô hình Hugging Face `trpakov/vit-face-expression` để nhận diện cảm xúc.
- Mô hình được tải ở lần dự đoán đầu tiên, sau đó được giữ lại để dùng cho các request tiếp theo. Mô hình chạy trên CPU.
- Cần kết nối Internet trong lần tải model đầu tiên.

Có thể đổi model Hugging Face bằng biến môi trường:

| Biến | Mặc định | Ý nghĩa |
| --- | --- | --- |
| `HF_EMOTION_MODEL_ID` | `trpakov/vit-face-expression` | ID model trên Hugging Face được dịch vụ sử dụng. |

Model phải cung cấp đủ bảy nhãn cảm xúc: `Angry`, `Disgust`, `Fear`, `Happy`, `Sad`, `Surprise`, `Neutral` (Giận dữ, Ghê tởm, Sợ hãi, Vui vẻ, Buồn, Ngạc nhiên, Bình thường). Dịch vụ kiểm tra nhãn khi tải model.

## API

| Phương thức | Endpoint | Mô tả |
| --- | --- | --- |
| `GET` | `/` | Kiểm tra trạng thái dịch vụ và thông tin mô hình. |
| `POST` | `/api/v1/predict` | Nhận ảnh tải lên dạng `multipart/form-data`, tên trường `file`. |
| `POST` | `/api/v1/predict-base64` | Nhận ảnh Base64 qua JSON; chấp nhận cả chuỗi Base64 thuần và Data URL. |

Ví dụ nội dung gửi tới `/api/v1/predict-base64`:

```json
{
	"image_base64": "<chuỗi Base64 hoặc Data URL của ảnh>"
}
```

Hai endpoint dự đoán trả về cùng cấu trúc dữ liệu:

```json
{
	"success": true,
	"emotion": "Happy",
	"emoji": "😊",
	"confidence": 0.98,
	"probabilities": {
		"Angry": 0.001,
		"Disgust": 0.001,
		"Fear": 0.002,
		"Happy": 0.98,
		"Sad": 0.005,
		"Surprise": 0.006,
		"Neutral": 0.005
	}
}
```

Các xác suất trong ví dụ chỉ để minh họa. Dịch vụ trả lỗi HTTP `400` khi ảnh hoặc dữ liệu Base64 không hợp lệ, `503` khi không tải được model và `500` khi quá trình dự đoán thất bại.