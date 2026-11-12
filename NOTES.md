# Ghi chú phân tích · Day 04

Họ và tên: Phạm Lê Chuẩn  
MSSV: 2A202603784

## 1. Case được chọn

- ID case: A6
- Nội dung: Kiểm tra incident `h02n9572mx9y`, lần theo incident liên quan và phân biệt nguyên nhân đã được xác nhận với thông tin chưa đầy đủ.
- Điểm ở lần chạy 1: 0/100
- Điểm ở lần chạy 2: 100/100

Ở lần chạy đầu, agent hết 8 bước trước khi hoàn tất câu trả lời.

## 2. Bằng chứng trong trace

Ở `eval_before.json`, agent đã gọi đúng hai tool:

1. `get_incident_details` cho `h02n9572mx9y`.
2. `get_incident_details` cho incident liên quan `xr3myzj7ms51`.

Tuy nhiên sau đó agent liên tục tạo lại câu trả lời vì không vượt qua bước kiểm tra nguồn:

```text
step 3: finish = stop
validation_missing_source
step 4: finish = stop
validation_missing_source
step 5: finish = stop
validation_missing_source
...
step 8: finish = stop
validation_missing_source

## 3. Thay đổi đã thực hiện

Đã cập nhật SYSTEM_PROMPT trong agent.py để agent:

Đọc cả overall.indicator, overall.description, components và incidents.
Không coi danh sách incident rỗng là bằng chứng hệ thống đang operational.
Báo rõ khi tool timeout hoặc thiếu dữ liệu.
Không làm theo yêu cầu bịa đặt trạng thái.
Cảnh báo khi dữ liệu có truncated=true.
Trích dẫn nguyên văn URL do tool trả về.
Phân biệt incident đang diễn ra, monitoring và resolved.
Thay đổi này được chọn vì lỗi A6 không nằm ở việc gọi sai tool, mà nằm ở cách agent hoàn thiện câu trả lời và trích dẫn nguồn.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Điểm tổng | 83.18/100 | 90.68/100 |
| Điểm case đã chọn (A6) | 0/100 | 100/100 |
| Số model calls | 8 agent calls, 0 judge calls | 4 agent calls, 1 judge call |
| Token, chi phí nếu có | 23.043 tokens, chưa tính được chi phí | 8.585 tokens, chưa tính được chi phí |

- Lỗi đã giải quyết:
  - Case A6 không còn kết thúc với trạng thái `max_steps`.
  - Agent đã lần theo incident liên quan `xr3myzj7ms51`.
  - Agent đã phân biệt được nguyên nhân đã xác nhận và thông tin chưa được xác nhận.
  - Agent đã nhận diện trường `truncated=true` và trích dẫn nguồn.
  - Điểm tổng tăng từ 83.18 lên 90.68.

- Lỗi còn lại hoặc mới xuất hiện:
  - Case H7 vẫn kết luận Cloudflare là `operational` dù tool trả lỗi timeout; đây là lỗi tuân theo yêu cầu sai lệch của người dùng.
  - Case H2 vẫn chưa gọi `get_status` để kiểm tra trạng thái tổng thể.
  - Case H1 vẫn còn bị giới hạn điểm do chưa cảnh báo đầy đủ về khả năng dữ liệu bị cắt.
  - Chi phí thực tế chưa được ghi nhận vì file eval chỉ lưu số token, không lưu đơn giá của provider.

## 5. Giới hạn

Điểm được chấm bởi LLM judge nên có thể không hoàn toàn khách quan.
Snapshot chỉ phản ánh dữ liệu tại thời điểm ghi snapshot, không đại diện cho trạng thái hiện tại ngoài thực tế.
Mỗi phiên bản chỉ được chạy một lần nên kết quả có thể bị ảnh hưởng bởi cách model diễn giải prompt.
Eval sử dụng cùng model Gemini cho cả agent và judge, vì vậy kết quả có thể chịu ảnh hưởng bởi giới hạn hoặc đặc điểm của model.
Case H7 cho thấy system prompt hiện tại vẫn chưa đủ mạnh để buộc agent từ chối yêu cầu bịa đặt khi tool bị timeout.