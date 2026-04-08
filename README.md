# Blind SPRINT trên blindsprint

Tác giả bản hiện tại: **Lê Đình Hải**  
Mục tiêu: triển khai và kiểm thử luồng **Blind SPRINT** (blind extension của threshold Schnorr theo kiến trúc preprocess + online signing).

## 1) Tổng quan
Dự án mở rộng từ code ECFFT/threshold trước đó để bổ sung cơ chế ký mù theo pipeline:

1. `setup`: tạo khóa công khai và chia sẻ bí mật dài hạn.
2. `preprocess`: tạo batch slot nonce đã chứng nhận (`R_bar`) và trạng thái slot.
3. `blind_request`: requester làm mù slot bằng `alpha`, tạo `R*`, challenge `e`, proof biết `alpha`.
4. `threshold_blind_response`: committee trả response share `pi_i = rho_i + e*sigma_i`.
5. `aggregate_and_unblind`: nội suy `z'`, giải mù thành `z`.
6. `verify`: kiểm tra Schnorr chuẩn `zG = R* + eS`.

Đầu ra cuối vẫn là chữ ký Schnorr chuẩn `(R*, z)` nên giữ tương thích verifier thông thường.

## 2) Cấu trúc file liên quan
- `src/blind_sprint.rs`: triển khai lõi Blind SPRINT (có comment tiếng Việt chi tiết).
- `src/threshold_schnorr.rs`: re-export để tương thích code cũ.
- `src/bin/testblind.rs`: chương trình demo + benchmark theo bước.
- `src/my_group.rs`: trait nhóm + canonical serialization dùng cho transcript hash.

## 3) Cách chạy
```powershell
cargo check
cargo run --bin testblind
cargo test blind_sprint -- --nocapture
```

## 4) Tham số thử nghiệm trong `testblind`
Mặc định benchmark hiện tại dùng:
- `n = 32`
- `t = 10`
- `a = 4`
- `b = 8`
- `d = t + a - 1 = 13`
- số signer cần để nội suy: `d + 1 = 14`
- số slot mỗi lần preprocess: `a * b = 32`

## 5) Nội dung in ra khi chạy `testblind`
Chương trình in rõ theo từng bước:
- Step 4: thông tin response share từng signer + kiểm tra hợp lệ share.
- Step 5: giá trị `z'` sau aggregate và `z` sau unblind.
- Step 6: kết quả verify cuối (`OK/FAIL`).
- Signature size: độ dài `R*`, `z`, tổng chữ ký.
- Timing: thời gian từng công đoạn.
- Online benchmark summary: trung bình microseconds cho nhiều vòng.

## 6) Kết quả thực nghiệm mẫu (bạn cung cấp)
```text
cargo run --bin testblind
   Compiling blindsprint v0.1.0 (C:\Users\ledin\Downloads\blindsprint (1)\blindsprint)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 1.60s
     Running `target\debug\testblind.exe`
--- Step 4: Signer response shares ---
party #1: share_valid=true, pi_i=BigInt([16096284758013177862, 10503037090147386795, 12009877204198010797, 131529772324503234]), P_i_len=32 bytes
party #2: share_valid=true, pi_i=BigInt([12470985548687967218, 770482645116206169, 14182681790219486909, 1139396817574811973]), P_i_len=32 bytes
party #3: share_valid=true, pi_i=BigInt([7723732180530754095, 14732265865671500470, 16331273870053405071, 695708749100132306]), P_i_len=32 bytes
party #4: share_valid=true, pi_i=BigInt([18006865444360763443, 12607688689363272954, 5953001292763292299, 160251235783579124]), P_i_len=32 bytes
party #5: share_valid=true, pi_i=BigInt([8190481637083416321, 9828095171016532533, 753336366680635774, 802888598401102354]), P_i_len=32 bytes
party #6: share_valid=true, pi_i=BigInt([16342510298506057445, 7466624965794371498, 3582839098825815898, 594566360104287406]), P_i_len=32 bytes
party #7: share_valid=true, pi_i=BigInt([18187373136926208984, 18106306481864371542, 1751357255856569820, 251457016703709526]), P_i_len=32 bytes
party #8: share_valid=true, pi_i=BigInt([6523887147189807413, 1522293756829824009, 7860246546490439533, 172783957666181749]), P_i_len=32 bytes
party #9: share_valid=true, pi_i=BigInt([5133199256789866062, 12532608256952008761, 15994305665529538322, 840134754460943146]), P_i_len=32 bytes
party #10: share_valid=true, pi_i=BigInt([12839439553589842940, 13216620356598578927, 17142875037545327817, 765411269859580929]), P_i_len=32 bytes
party #11: share_valid=true, pi_i=BigInt([4250829213667560609, 12104058661019040940, 11405052427987591414, 33804620651031444]), P_i_len=32 bytes
party #12: share_valid=true, pi_i=BigInt([9366369861298835157, 15906420031750374764, 646889110631717529, 540191420588067283]), P_i_len=32 bytes
party #13: share_valid=true, pi_i=BigInt([16845357696535398563, 7596620837615922333, 11142965113413321233, 864213703403828870]), P_i_len=32 bytes
party #14: share_valid=true, pi_i=BigInt([916041083526157183, 4768542173676838223, 18445752358215337658, 870004186699942327]), P_i_len=32 bytes
=== Blind SPRINT demo ===
n=32, t=10, a=4, b=8
threshold degree d=t+a-1=13, signers needed=14
slots per preprocess = a*b = 32
slot: (u=0, v=0)
response shares: 14
--- Step 5: Aggregate & Unblind ---
z' (aggregate) = BigInt([9885717927582687983, 9162423282125674862, 435555186682676696, 310076550505343857])
alpha (blind)  = BigInt([12157397185861278389, 12840965921820838164, 1843122195407902296, 1020393977466900614])
z (unblind)    = BigInt([15696871323645602231, 2052731070036444588, 2278677382090578993, 177549023365397495])
--- Step 6: Verify ---
verify: OK
--- Signature size ---
|R*| = 32 bytes
|z|  = 32 bytes
|sig|= 64 bytes (Schnorr standard format)
--- Timing (ms) ---
setup: 33
preprocess: 43
blind_request: 2
threshold_response: 43
aggregate_only: 10
unblind_only: 0
aggregate_unblind_api: 10
verify: 2
--- Online benchmark summary ---
rounds: 20, ok: 20/20
avg blind_request: 2463 us
avg threshold_response: 43871 us
avg aggregate_unblind: 10163 us
avg verify: 2161 us
```

## 7) Nhận xét nhanh từ kết quả
- Tính đúng: `verify: OK` và benchmark `20/20` thành công.
- Định dạng chữ ký: giữ chuẩn Schnorr, tổng `64 bytes`.
- Chi phí online chủ yếu nằm ở `threshold_response` và `aggregate_unblind`.

## 8) Ghi chú môi trường
- Khuyến nghị chạy trong môi trường có linker MSVC đầy đủ trên Windows.
- Nếu thay đổi tham số (`n,t,a,b`), cần đảm bảo số signer >= `t+a` để nội suy đúng theo bậc `d=t+a-1`.

