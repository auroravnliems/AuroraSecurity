# AuroraSecurity

> Plugin bảo mật toàn diện cho máy chủ Minecraft (Spigot / Paper / Folia) và proxy (BungeeCord / Velocity).
> Chống chiếm OP, chống giả mạo IP/UUID (anti-spoof), xác thực mật khẩu, 2FA, GeoIP, cảnh báo Discord và nhiều hơn nữa.

**Phiên bản:** 3.7 · **API:** 1.21 · **Folia:** ✅ · **Tác giả:** Aurora_VN

---

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Tính năng nổi bật](#2-tính-năng-nổi-bật)
3. [Yêu cầu](#3-yêu-cầu)
4. [Kiến trúc & cách triển khai](#4-kiến-trúc--cách-triển-khai)
5. [Cài đặt nhanh](#5-cài-đặt-nhanh)
6. [Hướng dẫn theo từng mô hình](#6-hướng-dẫn-theo-từng-mô-hình)
7. [Lệnh & quyền](#7-lệnh--quyền)
8. [Chi tiết tính năng & cách dùng](#8-chi-tiết-tính-năng--cách-dùng)
9. [Cấu hình `config.yml`](#9-cấu-hình-configyml)
10. [Biến môi trường (bảo mật secret)](#10-biến-môi-trường-bảo-mật-secret)
11. [Khuyến nghị bảo mật tối đa](#11-khuyến-nghị-bảo-mật-tối-đa)
12. [Xử lý sự cố (FAQ)](#12-xử-lý-sự-cố-faq)
13. [Giấy phép & ghi nhận](#13-giấy-phép--ghi-nhận)

---

## 1. Giới thiệu

AuroraSecurity bảo vệ máy chủ khỏi các kiểu tấn công phổ biến nhất:

- **Chiếm tài khoản admin/OP** từ IP lạ.
- **Giả mạo IP/UUID/danh tính (spoofing)** khi kết nối thẳng vào backend bỏ qua proxy — kiểu tấn công khiến "server tự đưa OP cho kẻ lạ".
- **Đăng nhập trái phép** vào tài khoản staff (mật khẩu + 2FA).
- **Bot/spam/VPN** và đăng nhập từ quốc gia bất thường.

Plugin được thiết kế theo nguyên tắc **phòng thủ chiều sâu (defense in depth)** và **fail-closed** (khi nghi ngờ thì chặn, không thả).

---

## 2. Tính năng nổi bật

| Nhóm | Tính năng |
|------|-----------|
| 🛡️ **Chống giả mạo** | Anti-spoof handshake (STANDALONE/BUNGEE/VELOCITY, tự nhận diện AUTO), kiểm tra cấu hình & tự **LOCKDOWN** khi không an toàn, xác minh **BungeeGuard token** / **Velocity HMAC**, **tích hợp sẵn tiêm BungeeGuard token** (không cần plugin riêng), **allowlist IP proxy** (chặn kết nối trực tiếp tới backend) |
| 🔐 **Xác thực** | Mật khẩu admin (PBKDF2, khóa khi sai nhiều lần), **2FA TOTP** (Google Authenticator/Authy) |
| 👮 **Bảo vệ thời gian thực** | Kiểm tra OP / gamemode / quyền theo IP đã lưu, tự kick hoặc bắt xác thực |
| 🌍 **GeoIP & VPN** | Chặn/cảnh báo theo quốc gia, phát hiện VPN/proxy/datacenter |
| 📜 **Theo dõi & cảnh báo** | Lịch sử IP, cảnh báo IP mới, **nhật ký audit** theo ngày, chặn lệnh nguy hiểm khi chưa xác thực |
| 🤖 **Discord** | Bot quản lý IP qua slash command, **DM cảnh báo** riêng cho từng admin |
| 💾 **Lưu trữ** | SQLite / MySQL / YAML |
| 🌐 **Khác** | Đa ngôn ngữ (9 thứ tiếng), hỗ trợ Folia, kiểm tra cập nhật |

---

## 3. Yêu cầu

- **Backend:** Spigot / Paper / Folia **1.16+** (khuyến nghị Paper 1.20+).
- **Java 17+** (build dùng Java 21).
- **License key** (lấy tại Discord của Aurora — xem `config.yml`). Plugin cần Internet để xác thực license; có cache offline tối đa **7 ngày**.
- **Proxy (tùy chọn):** BungeeCord/Waterfall hoặc Velocity.
- **MySQL (tùy chọn):** nếu muốn dùng storage MySQL.
- **Discord Bot (tùy chọn):** nếu muốn dùng tính năng Discord.

---

## 4. Kiến trúc & cách triển khai

AuroraSecurity gồm các module:

| Module | Cài ở đâu | Vai trò |
|--------|-----------|---------|
| **AuroraSecurity** (backend) | **Mỗi server con** (Spigot/Paper) — **BẮT BUỘC** | Toàn bộ tính năng: xác thực, anti-spoof verify, GeoIP, Discord, lưu trữ… |
| **AuroraSecurity-Bungee** | Proxy BungeeCord (tùy chọn) | Chặn handshake dị dạng ở edge + **tự tiêm BungeeGuard token** |
| **AuroraSecurity-Velocity** | Proxy Velocity (tùy chọn) | Chặn handshake dị dạng ở edge + nhắc cấu hình forwarding |

> ⚠️ **Quan trọng:** Lớp bảo vệ thật nằm ở **backend**. Cài chỉ trên proxy thì gần như không bảo vệ được gì trước tấn công kết nối thẳng. **Luôn cài bản backend trên mọi server con.**

---

## 4b. Build & file universal (1 jar duy nhất)

Bạn có thể build **một file `.jar` chạy được trên CẢ Spigot/Paper/Folia + BungeeCord + Velocity** — nó tự nhận diện môi trường (mỗi nền tảng chỉ đọc file mô tả của nó: `plugin.yml` / `bungee.yml` / `velocity-plugin.json`).

```bash
./gradlew :universal:universalJar
# -> universal/build/libs/AuroraSecurity-Universal.jar
```

- Thả **cùng một file** đó vào `plugins/` của backend, proxy Bungee, hoặc Velocity — đều chạy.
- Trên proxy, phần backend (JDA…) chỉ là "hành lý" không được nạp, nên hơi nặng nhưng vô hại — đây là cách các plugin universal (LuckPerms, ViaVersion…) vẫn làm.
- Jar universal **chưa obfuscate**; nếu có Allatori bản đầy đủ, chạy obfuscate trên file này sau.
- Vẫn có thể build riêng từng nền tảng nếu muốn gọn nhẹ: `:loader:build` (backend), `:bungee:shadowJar`, `:velocity:shadowJar`.

> ⚠️ Hãy **test thực tế trên cả 3 nền tảng** trước khi phát hành — file đã được kiểm tra cấu trúc (đủ descriptor + class), nhưng chạy thử trên server thật là bước của bạn.

---

## 5. Cài đặt nhanh

1. Tải file `.jar` phù hợp và đặt vào thư mục `plugins/`.
2. Khởi động server một lần để sinh thư mục `plugins/AuroraSecurity/`.
3. Mở `plugins/AuroraSecurity/config.yml`, điền **`license-key`**.
4. (Tùy chọn) cấu hình storage, Discord, anti-spoof… (xem [mục 9](#9-cấu-hình-configyml)).
5. Khởi động lại. Xem console: license `OK`, các tính năng báo `enabled`.

---

## 6. Hướng dẫn theo từng mô hình

### 6.1. Server đơn (STANDALONE — không có proxy)

1. Cài bản **backend** AuroraSecurity.
2. `config.yml` → `general.anti-spoof.mode: AUTO` (tự nhận diện STANDALONE).
3. Để `online-mode=true` (server premium) hoặc `false` (server lậu) tùy nhu cầu.
4. Xong. Anti-spoof sẽ chặn mọi handshake nhồi payload giả mạo.

### 6.2. BungeeCord / Waterfall

**Trên proxy:**
1. Đặt `ip_forward: true` trong `config.yml` của BungeeCord.
2. Cài `AuroraSecurity-Bungee.jar`. Khởi động → console in **đường dẫn file token** (`plugins/AuroraSecurity/bungeeguard-token.txt`).
   - AuroraSecurity **tự tiêm token** → **không cần** cài plugin BungeeGuard riêng.
   - Nếu bạn vẫn cài BungeeGuard riêng, plugin tự nhường (không tiêm trùng).
3. Mở file token, **copy giá trị token**.

**Trên mỗi backend:**
1. Cài bản backend AuroraSecurity.
2. `spigot.yml` → `settings.bungeecord: true`, `online-mode=false` (trong `server.properties`).
3. `config.yml`:
   ```yaml
   general:
     anti-spoof:
       mode: "AUTO"          # hoặc BUNGEE
       bungeeguard-tokens:
         - "DÁN_TOKEN_VÀO_ĐÂY"
   ```
4. (Khuyến nghị) bật thêm [allowlist IP proxy](#84-anti-spoof) để chặn kết nối thẳng.

> ⚠️ Nếu `mode=BUNGEE` mà `bungeeguard-tokens` rỗng → plugin **tự LOCKDOWN** (chặn mọi đăng nhập) để tránh bị giả mạo. Bắt buộc phải có token.

### 6.3. Velocity

**Trên proxy:**
1. `velocity.toml` → `player-info-forwarding-mode = "modern"`, ghi nhớ `forwarding-secret`.
2. (Tùy chọn) cài `AuroraSecurity-Velocity.jar` để thêm lớp chặn edge.

**Trên mỗi backend:**
1. Cài bản backend AuroraSecurity.
2. Bật Velocity modern forwarding trong Paper (`config/paper-global.yml` → `proxies.velocity.enabled: true`) với **đúng `forwarding-secret`**; `online-mode=false`.
3. `config.yml` → `general.anti-spoof.mode: AUTO` (hoặc VELOCITY).
4. (Tùy chọn, phòng thủ chiều sâu) copy file secret của proxy vào thư mục plugin và đặt tên trùng `general.anti-spoof.velocity-secret-file` để bật lớp HMAC độc lập.

> ⚠️ Nếu `mode=VELOCITY` nhưng **chưa bật** modern forwarding trên backend → plugin coi đây là cấu hình **không an toàn** và **LOCKDOWN** (vì khi đó không có lớp xác minh forwarding nào).

---

## 7. Lệnh & quyền

Lệnh gốc: **`/aurorasec`** (alias **`/aursec`**). Cần quyền nền `aurorasecurity.admin` để dùng.

| Lệnh | Mô tả | Quyền |
|------|-------|-------|
| `/aurorasec help` | Danh sách lệnh | `aurorasecurity.admin` |
| `/aurorasec reload` | Tải lại messages/cấu hình | `aurorasecurity.reload` |
| `/aurorasec license` | Xem trạng thái license | `aurorasecurity.admin` |
| `/aurorasec password set <player> <password>` | Đặt mật khẩu cho người chơi | `aurorasecurity.password.set` |
| `/aurorasec password remove <player>` | Xóa mật khẩu | `aurorasecurity.password.remove` |
| `/aurorasec password check <player>` | Kiểm tra đã có mật khẩu chưa | `aurorasecurity.password.check` |
| `/aurorasec password logout` | Đăng xuất phiên hiện tại | `aurorasecurity.password.logout` |
| `/aurorasec 2fa setup` | Bắt đầu cài 2FA (nhận secret) | `aurorasecurity.2fa.setup` |
| `/aurorasec 2fa confirm <mã>` | Xác nhận & bật 2FA | `aurorasecurity.2fa.setup` |
| `/aurorasec 2fa disable <mã hiện tại>` | Tắt 2FA của chính mình (cần mã) | `aurorasecurity.2fa.disable` |
| `/aurorasec 2fa disable <player>` | Admin tắt 2FA hộ người khác | `aurorasecurity.2fa.admin` |
| `/aurorasec 2fa status [player]` | Xem trạng thái 2FA | `aurorasecurity.2fa.setup` |
| `/aurorasec discord link <discord_id>` | Liên kết Discord để nhận DM | `aurorasecurity.discord.link` |
| `/aurorasec discord unlink [player]` | Hủy liên kết | `aurorasecurity.discord.link` |
| `/aurorasec discord status [player]` | Xem trạng thái liên kết | `aurorasecurity.discord.link` |
| `/aurorasec discord test` | Gửi DM thử | `aurorasecurity.discord.link` |

**Quyền khác:**

| Quyền | Tác dụng |
|-------|----------|
| `aurorasecurity.discord.admin` | Quản lý liên kết Discord của người khác |
| `aurorasecurity.geoip.bypass` | Bỏ qua kiểm tra GeoIP |
| `aurorasecurity.antispoof.bypass` | Bỏ qua kiểm tra VPN/proxy |

> OP có sẵn mọi quyền. Với người chơi thường, cấp quyền qua LuckPerms (hoặc plugin quyền khác).

---

## 8. Chi tiết tính năng & cách dùng

### 8.1. Bảo vệ thời gian thực (Real-time Protection)

Định kỳ quét người chơi (mặc định mỗi 100 tick = 5 giây) và kiểm tra **những ai cần giám sát**:

- **OP** (`methods.op: true`)
- **Gamemode** Creative/Spectator (`methods.gamemode: true`)
- **Quyền cụ thể** (`methods.permissions: [...]`)

Nếu người bị giám sát có **IP khác với IP đã lưu** → tùy cấu hình sẽ:
- **Bắt nhập mật khẩu / 2FA** (nếu đã thiết lập), hoặc
- **Kick** + gửi cảnh báo Discord (kèm nút "Add IP" để admin duyệt IP mới ngay trên Discord).

> IP "đã lưu" được quản lý qua bot Discord (`set`/`remove`) hoặc nút "Add IP".

**Tự xóa OP của người chưa xác thực (`deop-unauthenticated`):**
Thay vì **kick** một OP chưa xác thực, bạn có thể chọn **gỡ OP** của họ sau một khoảng ân hạn (mặc định 5 phút):
- Khi bật, OP chưa xác thực được cho **ân hạn N phút**; trong thời gian đó họ **bị chặn mọi lệnh** (không lạm dụng được OP) và **được nhắc nhập mật khẩu/2FA** để giữ OP.
- Nếu **xác thực kịp** → giữ nguyên OP, đồng hồ ân hạn xóa.
- Nếu **hết giờ** → plugin chạy `setOp(false)`, người chơi **vẫn ở lại server** như người thường (admin cấp lại OP sau). Có ghi log audit (`OP_REMOVED`) + cảnh báo Discord.
- Khi bật, tính năng này **thay thế hành vi kick** cho OP chưa xác thực (kể cả lúc vào server và khi prompt hết giờ). Mặc định **TẮT**.
```yaml
general:
  protect:
    deop-unauthenticated:
      enable: true
      after-minutes: 5
```

### 8.2. Xác thực mật khẩu (Password Auth)

- Băm **PBKDF2-HMAC-SHA256, 100.000 vòng**, salt ngẫu nhiên 32 byte; so sánh **constant-time**.
- **Khóa khi sai:** 3 lần sai → khóa 5 phút; hết lượt → **kick** (fail-closed).
- **Phiên đăng nhập:** sau khi nhập đúng, không phải nhập lại trong `session-duration` phút (mặc định 60).
- Khi bị bắt nhập mật khẩu, người chơi bị **đóng băng** (không di chuyển/đặt-phá khối/dùng lệnh/đánh/mở rương) cho tới khi xác thực; **hết 30 giây không nhập → kick**.

**Dùng:**
```
/aurorasec password set Aurora MatKhauManh123
```
Người chơi đăng nhập sẽ được nhắc nhập mật khẩu trong chat. Đăng xuất chủ động: `/aurorasec password logout`.

> Bật tính năng: `general.password-auth.enable: true`.

### 8.3. Xác thực 2 lớp (2FA TOTP)

Tương thích Google Authenticator, Authy, Microsoft Authenticator…

**Cài đặt cho bản thân:**
```
/aurorasec 2fa setup            # nhận secret key
# (thêm secret vào app authenticator)
/aurorasec 2fa confirm 123456   # nhập mã 6 số để bật
```
Khi đăng nhập sẽ được nhắc nhập mã 6 số (60 giây, tối đa 5 lần sai → kick). Có **chống replay** (không dùng lại cùng một mã).

**Tắt 2FA:** cần nhập **mã hiện tại** (`/aurorasec 2fa disable <mã>`) — chống kẻ chiếm phiên gỡ 2FA. Admin có thể gỡ hộ bằng `/aurorasec 2fa disable <player>` (quyền `aurorasecurity.2fa.admin`).

> Bật tính năng: `general.2fa.enable: true`.

### 8.4. Anti-Spoof (chống giả mạo) — lõi bảo mật

Ngăn kẻ tấn công kết nối **thẳng** vào backend rồi giả IP/UUID để thành OP. Các lớp:

- **Chế độ (`mode`):** `AUTO` (khuyến nghị, tự nhận diện từ cấu hình server), hoặc ép `STANDALONE` / `BUNGEE` / `VELOCITY`.
- **Kiểm tra cấu hình & LOCKDOWN:** nếu phát hiện cấu hình spoof được (ví dụ STANDALONE nhưng bật bungeecord forwarding, hoặc BUNGEE mà không có token, hoặc VELOCITY mà chưa bật modern forwarding) → tùy `on-insecure-config`:
  - `LOCKDOWN` (mặc định): **chặn mọi đăng nhập** cho tới khi sửa cấu hình (server vẫn chạy để admin đọc console).
  - `WARN`: chỉ cảnh báo.
- **BungeeGuard token (BUNGEE):** backend chỉ chấp nhận kết nối kèm token hợp lệ; so sánh constant-time. Module proxy **tự tiêm token** (xem [6.2](#62-bungeecord--waterfall)).
- **Velocity HMAC (VELOCITY):** xác minh chữ ký modern forwarding.
- **Allowlist IP proxy (`proxy-ip-allowlist`):** chỉ chấp nhận kết nối tới backend **từ IP của proxy** (kiểm IP socket thật — không giả mạo được). Chặn mọi kết nối trực tiếp từ ngoài, **kể cả ping**. Loopback (proxy cùng máy) luôn được phép. Hỗ trợ IP đơn và dải CIDR. Chỉ áp dụng khi proxied. → Rất hữu ích trên **shared hosting** khi không có quyền firewall.
- **Username-Spoof Guard:** một tên "đã đăng ký" (có mật khẩu/2FA) đăng nhập từ **IP hoàn toàn mới** → `REQUIRE_AUTH` (bắt xác thực) hoặc `KICK`.
- **Account/IP Limiter:** giới hạn số tài khoản khác nhau từ một IP trong một khoảng thời gian (chống bot swarm).
- **VPN/Proxy Guard:** dùng cờ `proxy`/`hosting` của ip-api để `KICK`/`ALERT` (mặc định tắt, vì có thể chặn nhầm người dùng VPN thật).

### 8.5. GeoIP

Chặn hoặc cảnh báo khi **admin** đăng nhập từ quốc gia bất thường (dùng ip-api.com miễn phí, không cần key).

```yaml
general:
  geoip:
    enable: true
    mode: "WHITELIST"     # chỉ cho phép các nước trong danh sách
    countries: ["VN"]
    action: "KICK"        # hoặc ALERT (chỉ báo Discord)
```
Bỏ qua bằng quyền `aurorasecurity.geoip.bypass`.

### 8.6. Lịch sử IP & cảnh báo IP mới

Ghi lại các IP đăng nhập của admin. Khi xuất hiện **IP mới** → cảnh báo lên kênh Discord và (nếu đã liên kết) **DM riêng** cho admin đó kèm vị trí & ISP.

### 8.7. Discord Bot (SecuBot)

Bot quản lý IP tin cậy ngay trên Discord, có **phân quyền theo role** và **giới hạn kênh**:

- `/aurorasecurity set <player> <ip>` — đặt IP tin cậy.
- `/aurorasecurity remove <player>` — xóa.
- Khi có người bị kick vì sai IP, bot gửi embed kèm **nút "Add IP"** để duyệt nhanh.

Cấu hình ở mục `discord:` (token, guild, role, notification-channel).

### 8.8. Discord DM Alerts

Admin tự liên kết Discord để nhận **DM riêng** khi: đăng nhập từ IP mới, đăng nhập bị chặn, sai mật khẩu/2FA.

```
/aurorasec discord link 123456789012345678   # Discord User ID (bật Developer Mode để copy)
/aurorasec discord test
```
> Bật: `general.discord-dm-alerts.enable: true`.

### 8.9. Nhật ký Audit

Ghi mọi sự kiện bảo mật vào file theo ngày tại `plugins/AuroraSecurity/audit/audit-YYYY-MM-DD.log` (ghi bất đồng bộ, tự xóa log cũ theo `retention-days`). Dữ liệu đã được **lọc ký tự xuống dòng** để chống giả mạo dòng log.

### 8.10. Command Blocker

Khi người bị giám sát **chưa qua xác thực**, plugin **chặn toàn bộ lệnh** (không phải danh sách đen dễ né) → kẻ giả mạo không thể chạy `/op`, `/tp`, `/sudo`… trước khi bị kick.

### 8.11. Lưu trữ (Storage)

Chọn ở `storage.storage-type`: `SQLITE` (mặc định), `MYSQL`, hoặc `YAML`. Tất cả truy vấn dùng PreparedStatement (không SQL injection). MySQL **bật TLS + xác minh chứng chỉ** mặc định (chống MITM).

---

## 9. Cấu hình `config.yml`

```yaml
license-key: 'YOUR-LICENSE-KEY'     # BẮT BUỘC
config-version: 3
debug: false

storage:
  storage-type: "SQLITE"            # SQLITE | MYSQL | YAML
  mysql:
    host: "localhost"
    port: 3306
    database: "aurorasecurity"
    username: "root"
    password: "CHANGE_THIS_PASSWORD" # hoặc dùng biến môi trường (xem mục 10)
    ssl:
      enabled: true                  # bật TLS
      verify-certificate: true       # xác minh chứng chỉ (chống MITM) — chỉ tắt cho DB nội bộ/self-signed

discord:
  token: "YOUR_DISCORD_BOT_TOKEN"    # hoặc biến môi trường AURORA_DISCORD_TOKEN
  guild: "YOUR_GUILD_ID"
  role: "YOUR_ADMIN_ROLE_ID"
  notification-channel: "YOUR_CHANNEL_ID"

general:
  language: "vn"                     # en, vn, cn, jp, kr, es, fr, de, ru
  timezone: "Asia/Ho_Chi_Minh"
  date-format: "dd/MM/yyyy HH:mm:ss"
  shutdown-on-disable: false

  protect:
    enable: true
    interval: 100                    # tick (20 = 1 giây)
    methods:
      op: true
      gamemode: true
      permissions: []                # ví dụ: ["essentials.god"]
    deop-unauthenticated:            # gỡ OP (thay vì kick) nếu OP không xác thực trong N phút
      enable: false
      after-minutes: 5

  add-ip-button:
    enable: true
    text: "Add IP"
  send-valid-message: true

  password-auth:
    enable: false
    session-duration: 60             # phút

  audit-log:
    enable: true
    retention-days: 30

  command-blocker:
    enable: true
    blocked-commands: [ ... ]        # (giữ để tương thích; hiện chặn TẤT CẢ lệnh khi đang xác minh)

  ip-history:
    enable: true
    max-ips-per-player: 10

  2fa:
    enable: false
    issuer: "AuroraSecurity"
    window-size: 1                   # ±30s

  geoip:
    enable: false
    mode: "WHITELIST"                # WHITELIST | BLACKLIST
    countries: ["VN"]
    action: "KICK"                   # KICK | ALERT
    bypass-permission: "aurorasecurity.geoip.bypass"
    cache-minutes: 60

  discord-dm-alerts:
    enable: false

  anti-spoof:
    enable: true
    mode: "AUTO"                     # AUTO | STANDALONE | BUNGEE | VELOCITY
    on-insecure-config: "LOCKDOWN"   # LOCKDOWN | WARN
    velocity-secret-file: "forwarding.secret"
    bungeeguard-tokens: []           # (BUNGEE) token proxy tiêm vào
    proxy-ip-allowlist: []           # chỉ cho IP proxy kết nối tới backend (xem 8.4)
    username-spoof:
      enable: true
      action: "REQUIRE_AUTH"         # REQUIRE_AUTH | KICK
    account-ip-limit:
      enable: true
      max-accounts-per-ip: 3
      window-minutes: 60
    block-vpn:
      enable: false
      action: "ALERT"                # KICK | ALERT
```

---

## 10. Biến môi trường (bảo mật secret)

Để **không lưu secret trong file cấu hình**, dùng biến môi trường (plugin ưu tiên đọc biến môi trường nếu có):

| Biến | Thay cho |
|------|----------|
| `AURORA_DISCORD_TOKEN` | `discord.token` |
| `AURORA_MYSQL_PASSWORD` | `storage.mysql.password` |

```bash
# Linux/Mac
export AURORA_DISCORD_TOKEN="..."
export AURORA_MYSQL_PASSWORD="..."
```
```bat
:: Windows
set AURORA_DISCORD_TOKEN=...
set AURORA_MYSQL_PASSWORD=...
```

---

## 11. Khuyến nghị bảo mật tối đa

Xếp theo độ ưu tiên (kết hợp càng nhiều càng tốt):

1. **Anti-spoof: để `mode: AUTO` và `on-insecure-config: LOCKDOWN`.** Đây là lớp chống "tự đưa OP cho kẻ lạ".
2. **Backend không bao giờ mở ra Internet.** Nếu có quyền hệ thống → dùng **firewall/iptables/security group** chỉ cho IP proxy. Nếu không (shared hosting) → bật **`proxy-ip-allowlist`** trong plugin.
3. **Proxy dùng forwarding có chữ ký:** Velocity *modern* (tốt nhất) hoặc BungeeGuard token.
4. **Bật mật khẩu + 2FA cho admin/staff**, đặt `online-mode=false` đúng cách khi sau proxy.
5. **Đưa secret ra biến môi trường**, để MySQL `verify-certificate: true`.
6. **Bật audit-log** và (tùy chọn) **discord-dm-alerts** để phát hiện sớm.
7. Cân nhắc `username-spoof.action: KICK` và `block-vpn` nếu chấp nhận chặt hơn (có thể ảnh hưởng người dùng VPN hợp pháp).

> Triết lý: nhiều lớp độc lập. Lộ token vẫn còn allowlist IP; không có firewall vẫn còn plugin; sai cấu hình thì LOCKDOWN.

---

## 12. Xử lý sự cố (FAQ)

**Server bị LOCKDOWN, không ai vào được?**
→ Console đang in lý do (`[ANTI-SPOOF][INSECURE]`). Sửa cấu hình theo hướng dẫn (thường là thiếu BungeeGuard token, chưa bật velocity forwarding, hoặc mode sai). Sau khi sửa thì khởi động lại.

**Sau proxy mà người chơi bị kick "invalid token"/giả mạo?**
→ Token trên backend (`bungeeguard-tokens`) phải **trùng** token proxy in ra (`bungeeguard-token.txt`). Velocity: `forwarding-secret` của proxy phải trùng secret backend.

**Bật `proxy-ip-allowlist` xong không ai vào được?**
→ Bạn điền sai IP proxy. Xem log dòng `direct connection from non-proxy IP <ip>` → đó là IP cần thêm. Nếu proxy cùng máy thì để trống cũng được (loopback luôn cho qua).

**Quên mật khẩu admin?**
→ Console (OP): không cần — bạn vào console gỡ bằng cách xóa entry trong `passwords.yml` hoặc dùng `/aurorasec password remove <player>` từ một admin khác.

**License báo INVALID?**
→ Kiểm tra `license-key`, đảm bảo server có Internet. Mất mạng vẫn chạy được tối đa 7 ngày (cache offline).

**Dùng được trên hosting không?**
→ Có. Các tính năng đều chạy ở tầng plugin, không cần root. `proxy-ip-allowlist` đặc biệt hữu ích khi không có quyền firewall. Trên shared hosting dùng chung node, hãy coi token/HMAC là khóa chính.

---

## 13. Giấy phép & ghi nhận

- AuroraSecurity © Aurora_VN.
- Phần **tiêm BungeeGuard token** (lớp `SpoofedLoginResult` và bộ tiêm token trong module Bungee) được phỏng theo **BungeeGuard** (© lucko & contributors), giấy phép **MIT**. Toàn văn giấy phép đi kèm trong `THIRD-PARTY-NOTICES.txt` của jar Bungee.

---

> 💬 Hỗ trợ & license: xem link Discord trong `config.yml`.
