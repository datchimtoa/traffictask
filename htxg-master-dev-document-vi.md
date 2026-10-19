# HTXG.PRO — Master Development Document
## Nền tảng nhiệm vụ – phần thưởng chủ quyền cho thị trường Việt Nam

Phiên bản: 2.0  
Ngày: 2026-10-19  
Trạng thái: Bản định hướng phát triển chính thức  
Đối tượng: Founder, Product Owner, Backend, Frontend, UI/UX, QA.

---

## 1. Định nghĩa sản phẩm

HTXG.PRO là một **nền tảng nhiệm vụ – phần thưởng hoàn toàn chủ quyền** dành cho người dùng Việt Nam.

Người dùng vào HTXG để:

- Tìm và hoàn thành nhiệm vụ: khảo sát, cài ứng dụng, đăng ký dịch vụ, xem nội dung, thử sản phẩm.
- Nhận điểm sau khi nhiệm vụ được xác minh.
- Theo dõi ví, lịch sử và trạng thái từng nhiệm vụ.
- Rút tiền về MoMo, ZaloPay hoặc tài khoản ngân hàng.

HTXG không phải là website nhúng PubCrypto.  
HTXG là **nền tảng offerwall riêng của HTXG**, nơi PubCrypto chỉ là một nguồn cung nhiệm vụ đầu tiên.

### 1.1 Vị trí của PubCrypto

PubCrypto được tích hợp như một **Supply Provider**, tức nguồn cung nhiệm vụ.

```text
PubCrypto        = Nhà cung cấp nhiệm vụ
HTXG.PRO         = Nền tảng sở hữu người dùng, ví, UI, anticheat, rút tiền
Người dùng       = Chỉ tương tác với HTXG
```

PubCrypto được dùng để:

- Lấy danh sách nhiệm vụ ban đầu.
- Tạo link/iframe nhiệm vụ.
- Nhận postback khi user hoàn thành.
- Tham khảo flow chuẩn: click → conversion → postback → credit/reversal.

Nhưng HTXG phải có khả năng thay thế hoặc bổ sung PubCrypto bằng provider khác mà không ảnh hưởng đến toàn bộ sản phẩm.

### 1.2 Nguyên tắc chủ quyền

| Thành phần | Chủ sở hữu |
|---|---|
| Thương hiệu, domain, UI/UX | HTXG |
| Tài khoản người dùng | HTXG |
| Ví điểm và lịch sử giao dịch | HTXG |
| Hệ thống anticheat | HTXG |
| Quy tắc quy đổi điểm và rút tiền | HTXG |
| Trang Earn, Wallet, Withdraw | HTXG |
| Nhiệm vụ từ PubCrypto | PubCrypto cung cấp, HTXG tích hợp |
| Nhiệm vụ từ provider khác | Từng provider cung cấp, HTXG tích hợp |

Người dùng không cần biết nhiệm vụ đến từ PubCrypto hay provider nào khác. Họ chỉ thấy HTXG.

---

## 2. Mục tiêu phiên bản đầu tiên

### MVP phải đạt được

1. Người dùng đăng ký, đăng nhập và có ví điểm riêng.
2. Người dùng xem danh sách nhiệm vụ trên trang Earn của HTXG.
3. Người dùng bắt đầu nhiệm vụ và được đưa đến provider.
4. HTXG nhận postback từ provider và tự động xác minh.
5. Điểm được cộng vào ví sau khi qua risk check và hold period.
6. Người dùng rút tiền qua MoMo, ZaloPay hoặc ngân hàng.
7. Hệ thống tự động chặn hoặc đánh dấu các hành vi gian lận phổ biến.
8. Admin chỉ xử lý trường hợp rủi ro cao, không duyệt từng nhiệm vụ.

### MVP không cần

- Không cần marketplace cho advertiser tự đăng campaign.
- Không cần cho webmaster khác nhúng offerwall của HTXG.
- Không cần machine learning phức tạp.
- Không cần tích hợp nhiều provider cùng lúc.
- Không cần duyệt thủ công toàn bộ nhiệm vụ.

---

## 3. Kiến trúc tổng thể

```text
Người dùng
   |
   v
HTXG Frontend
   |
   v
HTXG Backend API
   |
   +--> Auth & User Service
   +--> Device & Risk Profile Service
   +--> Offer Catalog Service
   +--> Task Session Service
   +--> Provider Adapter Layer
   +--> Postback Receiver
   +--> Risk Engine
   +--> Reward Ledger
   +--> Withdrawal Service
   +--> Notification Service
   +--> Admin Service
   |
   v
PostgreSQL + Redis + Queue + Object Storage
```

### 3.1 Provider Adapter Layer

Đây là lớp quan trọng nhất để HTXG chủ quyền.

```text
PubCrypto Adapter
Provider B Adapter
Provider C Adapter
        |
        v
Chuẩn hóa về định dạng HTXG
        |
        v
Offer Catalog / Postback Receiver / Risk Engine
```

Mỗi provider chỉ cần một adapter riêng cho:

- Lấy danh sách nhiệm vụ.
- Tạo link hoặc iframe.
- Xác minh postback.
- Chuẩn hóa `transaction_id`, `reward`, `status`.
- Xử lý reversal/chargeback.

Core HTXG không được phụ thuộc cứng vào PubCrypto.

### 3.2 Công nghệ đề xuất

| Hạng mục | Đề xuất |
|---|---|
| Frontend | Next.js + TypeScript + Tailwind CSS |
| Backend | NestJS + TypeScript, hoặc Go, hoặc Laravel |
| Database | PostgreSQL |
| Cache/counter | Redis |
| Queue | BullMQ hoặc RabbitMQ |
| Storage | S3-compatible |
| Monitoring | Sentry + Grafana/Prometheus |
| Secret | Environment variable hoặc secret manager |

---

## 4. Luồng sản phẩm chính

### 4.1 Hành trình người dùng

1. User truy cập HTXG.PRO.
2. Đăng ký/đăng nhập.
3. Vào trang Earn.
4. Chọn nhiệm vụ phù hợp.
5. Xem điều kiện, phần thưởng dự kiến và thời gian.
6. Bấm “Bắt đầu nhiệm vụ”.
7. HTXG tạo task session và chuyển user đến provider.
8. User hoàn thành nhiệm vụ.
9. Provider gửi postback về HTXG.
10. HTXG xác minh, chấm risk và cộng điểm vào pending.
11. Sau hold period, điểm chuyển sang available.
12. User rút tiền khi đủ điều kiện.

### 4.2 Trạng thái nhiệm vụ

| Trạng thái | Ý nghĩa |
|---|---|
| `started` | User đã bắt đầu nhiệm vụ |
| `awaiting_completion` | Đang chờ provider gửi kết quả |
| `verifying` | Postback đã đến, đang xác minh |
| `pending` | Hợp lệ, đang chờ hold period |
| `available` | Điểm đã có thể dùng/rút |
| `locked` | Đang review do risk |
| `rejected` | Không đủ điều kiện nhận thưởng |
| `reversed` | Provider hoàn trả/chargeback |

---

## 5. Database Design

### 5.1 users

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email CITEXT UNIQUE,
  phone VARCHAR(32) UNIQUE,
  password_hash TEXT NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'active',
  trust_score INT NOT NULL DEFAULT 50,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 5.2 devices

```sql
CREATE TABLE devices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  fingerprint_hash CHAR(64) NOT NULL,
  user_agent TEXT,
  os VARCHAR(50),
  browser VARCHAR(50),
  device_model VARCHAR(100),
  screen_resolution VARCHAR(30),
  timezone VARCHAR(64),
  language VARCHAR(20),
  ip INET,
  asn INT,
  is_vpn BOOLEAN DEFAULT false,
  is_datacenter BOOLEAN DEFAULT false,
  first_seen_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_seen_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (user_id, fingerprint_hash)
);
```

### 5.3 providers

```sql
CREATE TABLE providers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'active',
  config JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Provider đầu tiên: `pubcrypto`.

### 5.4 offers

```sql
CREATE TABLE offers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID NOT NULL REFERENCES providers(id),
  external_offer_id VARCHAR(191),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  category VARCHAR(50),
  payout_currency VARCHAR(10) NOT NULL DEFAULT 'USDT',
  provider_payout NUMERIC(18,8),
  user_reward_points INT NOT NULL,
  countries TEXT[] DEFAULT '{}',
  min_delay_seconds INT NOT NULL DEFAULT 0,
  daily_cap INT,
  weekly_cap INT,
  status VARCHAR(20) NOT NULL DEFAULT 'active',
  provider_metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (provider_id, external_offer_id)
);
```

### 5.5 task_sessions

```sql
CREATE TABLE task_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  device_id UUID NOT NULL REFERENCES devices(id),
  offer_id UUID NOT NULL REFERENCES offers(id),
  provider_id UUID NOT NULL REFERENCES providers(id),
  click_id VARCHAR(191) UNIQUE,
  session_token_hash CHAR(64) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'opened',
  issued_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at TIMESTAMPTZ NOT NULL,
  completed_at TIMESTAMPTZ,
  ip INET,
  user_agent TEXT
);
```

### 5.6 conversions

```sql
CREATE TABLE conversions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID NOT NULL REFERENCES providers(id),
  external_transaction_id VARCHAR(191) NOT NULL,
  task_session_id UUID REFERENCES task_sessions(id),
  user_id UUID NOT NULL REFERENCES users(id),
  offer_id UUID REFERENCES offers(id),
  provider_reward NUMERIC(18,8),
  user_reward_points INT,
  status VARCHAR(20) NOT NULL,
  risk_score INT NOT NULL DEFAULT 0,
  decision VARCHAR(20) NOT NULL,
  decision_reasons JSONB NOT NULL DEFAULT '[]',
  raw_payload JSONB,
  received_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  processed_at TIMESTAMPTZ,
  UNIQUE (provider_id, external_transaction_id)
);
```

### 5.7 ledger_entries

```sql
CREATE TABLE ledger_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  conversion_id UUID REFERENCES conversions(id),
  withdrawal_id UUID,
  entry_type VARCHAR(30) NOT NULL,
  amount_points INT NOT NULL,
  pending_after INT NOT NULL,
  available_after INT NOT NULL,
  locked_after INT NOT NULL,
  description TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 5.8 withdrawals

```sql
CREATE TABLE withdrawals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  amount_points INT NOT NULL,
  amount_vnd NUMERIC(18,2) NOT NULL,
  method VARCHAR(20) NOT NULL,
  payout_destination_hash CHAR(64) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'requested',
  risk_score INT NOT NULL DEFAULT 0,
  decision VARCHAR(20) NOT NULL,
  decision_reasons JSONB NOT NULL DEFAULT '[]',
  requested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  processed_at TIMESTAMPTZ
);
```

### 5.9 payout_destinations

```sql
CREATE TABLE payout_destinations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  method VARCHAR(20) NOT NULL,
  destination_hash CHAR(64) NOT NULL,
  destination_masked VARCHAR(100) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'active',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (method, destination_hash)
);
```

### 5.10 risk_events

```sql
CREATE TABLE risk_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  device_id UUID REFERENCES devices(id),
  task_session_id UUID REFERENCES task_sessions(id),
  conversion_id UUID REFERENCES conversions(id),
  withdrawal_id UUID REFERENCES withdrawals(id),
  event_type VARCHAR(50) NOT NULL,
  severity VARCHAR(20) NOT NULL,
  risk_score_delta INT NOT NULL,
  details JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## 6. Hệ thống Anticheat trung tâm

Đây là hệ thống **duy nhất**, dùng chung cho mọi provider.

### 6.1 Task Session

Mỗi lần bắt đầu nhiệm vụ:

- Backend tạo `session_id`.
- Sinh token một lần.
- Gắn `user_id`, `device_id`, `offer_id`, `provider_id`, `click_id`.
- Hết hạn sau 5–15 phút.
- Postback phải map về session hợp lệ.

### 6.2 Postback Receiver

Mỗi provider một endpoint:

```text
POST /postbacks/pubcrypto
POST /postbacks/provider-b
POST /postbacks/provider-c
```

Quy trình chung:

1. Lưu raw payload.
2. Verify signature theo adapter của provider.
3. Verify IP nếu provider công bố.
4. Verify timestamp.
5. Verify `transaction_id` chưa tồn tại.
6. Verify task session hợp lệ.
7. Chuẩn hóa dữ liệu.
8. Đưa vào queue xử lý risk.
9. Trả response theo yêu cầu provider.

### 6.3 Duplicate Protection

Dùng unique constraint:

```sql
UNIQUE (provider_id, external_transaction_id)
```

Không được chỉ check “đã tồn tại chưa” ở application layer.

### 6.4 Risk Signals

| Tín hiệu | Điểm rủi ro | Hành động |
|---|---:|---|
| Sai signature | 100 | Reject |
| Trùng `transaction_id` | 100 | Bỏ qua |
| Không có session hợp lệ | 100 | Reject |
| Conversion quá nhanh | 60 | Reject/review |
| IP datacenter/VPN/Tor | 40 | Review |
| Device dùng bởi 3+ user | 45 | Review |
| IP dùng bởi 5+ user | 40 | Review |
| Quốc gia/ngôn ngữ không khớp | 25 | Review |
| Vượt daily cap | 50 | Block |
| Spike từ một ASN/device | 35 | Review |

### 6.5 Risk Score Decision

| Điểm | Quyết định |
|---:|---|
| 0–39 | Auto approve vào pending |
| 40–69 | Credit nhưng locked/hold dài hơn |
| 70–99 | Reject hoặc review |
| 100 | Reject ngay |

### 6.6 Withdrawal Risk Gate

Trước khi cho rút:

- Kiểm tra available balance.
- Kiểm tra minimum withdrawal.
- Kiểm tra balance âm.
- Kiểm tra payout destination duy nhất.
- Kiểm tra hold period.
- Kiểm tra withdrawal velocity.
- Kiểm tra conversion đang review.

---

## 7. API Design

### 7.1 Auth

```http
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
```

### 7.2 Offers

```http
GET /api/v1/offers
GET /api/v1/offers/:id
POST /api/v1/offers/:id/start
GET /api/v1/tasks/:sessionId
GET /api/v1/tasks/history
```

### 7.3 Wallet

```http
GET /api/v1/wallet
GET /api/v1/wallet/transactions
POST /api/v1/wallet/withdraw
GET /api/v1/withdrawals
```

### 7.4 Postback

```http
POST /postbacks/:providerCode
```

### 7.5 Admin

```http
GET /api/v1/admin/users
GET /api/v1/admin/conversions
GET /api/v1/admin/risk-queue
GET /api/v1/admin/withdrawals
GET /api/v1/admin/providers
```

---

## 8. UI/UX Specification

### 8.1 Art Direction

Hướng design: dark analytics dashboard, chuyên nghiệp, đáng tin, không generic AI.

| Token | Giá trị |
|---|---|
| Background | `#0B0D10` |
| Surface | `#12151A` |
| Border | `#1E242C` |
| Primary | `#0F2BF0` |
| Success | `#22C55E` |
| Warning | `#F59E0B` |
| Danger | `#EF4444` |
| Text primary | `#F5F7FA` |
| Text secondary | `#9AA4B2` |
| Font UI | Inter hoặc Geist |
| Font mono | JetBrains Mono |

### 8.2 Information Architecture

```text
Trang chủ
Earn
Wallet
  - Balance
  - Pending
  - History
  - Withdraw
Tài khoản
  - Profile
  - Devices
  - Security
  - Payout methods
Hỗ trợ
  - FAQ
  - Appeal
Admin
  - Dashboard
  - Users
  - Conversions
  - Risk queue
  - Withdrawals
  - Providers
```

### 8.3 Trang Earn

Layout:

```text
+------------------------------------------------------+
| Sidebar | Offer Grid / List          | Activity Panel |
+------------------------------------------------------+
```

Sidebar:

- Logo HTXG.
- Navigation.
- Available balance.
- Pending balance.
- Daily progress.

Main:

- Filter nhiệm vụ.
- Search.
- Sort.
- Offer cards.

Offer card:

- Tên nhiệm vụ.
- Điểm thưởng.
- Thời gian ước tính.
- Quốc gia.
- Trạng thái.
- CTA: “Bắt đầu”.

### 8.4 Wallet

Hiển thị 4 số dư riêng:

- Pending.
- Available.
- Locked.
- Withdrawn.

Không gộp pending và available.

### 8.5 Withdraw

Hiển thị:

- Available balance.
- Số điểm muốn rút.
- Phương thức: MoMo, ZaloPay, Bank.
- Thông tin nhận tiền.
- Thời gian xử lý.
- Cảnh báo xác minh bổ sung nếu rủi ro cao.

### 8.6 Task Status

Timeline:

```text
Đã bắt đầu
   ↓
Đang chờ hoàn thành
   ↓
Đã nhận kết quả
   ↓
Đang xác minh
   ↓
Thưởng khả dụng
```

---

## 9. Admin

Admin không duyệt mọi nhiệm vụ. Admin chỉ xử lý exception.

### 9.1 Dashboard

Metrics:

- Active users.
- Task starts.
- Conversions.
- Approval rate.
- Rejection rate.
- Reversal rate.
- Pending liability.
- Withdrawal volume.
- Fraud-blocked events.

### 9.2 Risk Queue

Chỉ hiển thị case risk cao:

- User.
- Device.
- IP/ASN.
- Offer.
- Provider.
- Reward.
- Risk score.
- Reason codes.
- Raw postback.
- Action: approve, reject, request info.

### 9.3 Provider Health

Theo dõi:

- Postback volume.
- Success rate.
- Duplicate rate.
- Reversal rate.
- Latency.
- API error.
- Payout reconciliation.

---

## 10. Security

- HTTPS bắt buộc.
- Secret nằm ở secret manager.
- Không có secret trong frontend.
- Argon2id/bcrypt cho password.
- Rate limit auth, task start, postback, withdrawal.
- 2FA admin bắt buộc.
- Audit log append-only.
- Backup database.
- Tách dev/staging/production.

---

## 11. Testing

### 11.1 Unit test

- Signature verification.
- Reward calculation.
- Ledger balance.
- Risk score.
- Cooldown.
- Withdrawal eligibility.

### 11.2 Integration test

- Task start.
- Valid postback.
- Invalid signature.
- Duplicate postback.
- Reversal postback.
- Expired session.
- Withdrawal success/block.

### 11.3 Fraud test

- Fake postback.
- Replay postback.
- Multi-account same device.
- Multi-account same IP.
- VPN/datacenter IP.
- Instant conversion.
- Shared payout destination.
- Withdraw ngay sau khi được cộng điểm.

---

## 12. Launch Plan

### Phase 1: Foundation — 1–2 tuần

- Auth.
- User/device.
- Offer catalog.
- Task session.
- Reward ledger.
- Wallet UI.

### Phase 2: PubCrypto Integration — 1–2 tuần

- PubCrypto adapter.
- Signed iframe.
- Postback receiver.
- Signature verification.
- Dedupe.
- Reversal handling.

### Phase 3: Anti-Fraud — 2 tuần

- Velocity rules.
- Device/IP clustering.
- Risk score.
- Hold period.
- Withdrawal gate.
- Admin risk queue.

### Phase 4: Payment — 1 tuần

- MoMo/ZaloPay/bank.
- Withdrawal queue.
- Notification.
- Monitoring.
- Beta launch.

### Phase 5: Scale

- Thêm provider thứ hai.
- Provider health dashboard.
- Reconciliation report.
- Appeal workflow.
- Behavioral scoring.

---

## 13. Acceptance Criteria

Hệ thống sẵn sàng production khi:

- Không thể cộng điểm nếu không có task session hợp lệ.
- Không thể cộng trùng điểm khi postback retry.
- Postback sai signature bị reject và log.
- Reversal làm giảm đúng số dư.
- User không rút được pending/locked points.
- Withdrawal rủi ro cao bị hold tự động.
- Mọi conversion và withdrawal có audit trail.
- Admin có thể điều tra user đáng ngờ mà không duyệt toàn bộ nhiệm vụ.
- UI phân biệt rõ pending, available, locked, rejected, reversed.
- Provider secret không xuất hiện trong frontend.

---

## 14. Cảnh báo quan trọng

1. Đừng cộng điểm từ frontend.
2. Đừng tin `user_id`, `reward`, `timestamp` do client gửi.
3. Đừng cho rút ngay sau conversion.
4. Đừng cho nhiều user dùng chung payout destination.
5. Đừng để provider secret xuất hiện trong frontend.
6. Đừng coi MD5 signature là đủ chống gian lận.
7. Đừng ẩn reversal khỏi user.
8. Đừng mở unlimited withdrawal cho account mới.
9. Đừng tích hợp nhiều provider trước khi provider đầu ổn định.
10. Đừng dùng duyệt tay làm biện pháp chính.
