# Hướng Dẫn Cài Đặt SSO Plugin Chi Tiết

## Mục Lục

1. [Chuẩn Bị](#1-chuẩn-bị)
2. [Cài Đặt Google OAuth](#2-cài-đặt-google-oauth)
3. [Cài Đặt GitHub OAuth](#3-cài-đặt-github-oauth)
4. [Cài Đặt Microsoft OAuth](#4-cài-đặt-microsoft-oauth)
5. [Cài Đặt Authentik](#5-cài-đặt-authentik)
6. [Cấu Hình Trong Botble Admin](#6-cấu-hình-trong-botble-admin)
7. [Xử Lý Lỗi Thường Gặp](#7-xử-lý-lỗi-thường-gặp)

---

## 1. Chuẩn Bị

### 1.1. Kích Hoạt Plugin

```bash
# Chạy migration
php artisan migrate
```

### 1.2. Xác Định Callback URL

Callback URL có dạng:
```
https://your-domain.com/sso/{provider-slug}/callback
```

**Ví dụ:**
- Google: `https://your-domain.com/sso/google/callback`
- GitHub: `https://your-domain.com/sso/github/callback`
- Microsoft: `https://your-domain.com/sso/microsoft/callback`
- Authentik: `https://your-domain.com/sso/authentik/callback`

> ⚠️ **Lưu ý cho môi trường local development:**
> - Sử dụng `http://localhost:8000/sso/{slug}/callback` (nếu dùng `php artisan serve`)
> - Hoặc `http://127.0.0.1:8000/sso/{slug}/callback`
> - **KHÔNG** sử dụng `.test` domain vì Google và Microsoft không chấp nhận.

---

## 2. Cài Đặt Google OAuth

### Bước 1: Truy cập Google Cloud Console

1. Đi đến: https://console.cloud.google.com/
2. Đăng nhập bằng tài khoản Google

### Bước 2: Tạo Project (nếu chưa có)

1. Click **Select a project** ở góc trái trên
2. Click **NEW PROJECT**
3. Nhập tên project: `Botble SSO` (hoặc tên bất kỳ)
4. Click **CREATE**

### Bước 3: Cấu hình OAuth Consent Screen

1. Menu trái → **APIs & Services** → **OAuth consent screen**
2. Chọn **External** → Click **CREATE**
3. Điền thông tin:
   - **App name**: Tên website của bạn
   - **User support email**: Email của bạn
   - **Developer contact information**: Email của bạn
4. Click **SAVE AND CONTINUE**
5. **Scopes**: Click **ADD OR REMOVE SCOPES**
   - Chọn: `email`, `profile`, `openid`
   - Click **UPDATE** → **SAVE AND CONTINUE**
6. **Test users**: Bỏ qua → **SAVE AND CONTINUE**
7. **Summary**: Click **BACK TO DASHBOARD**

### Bước 4: Tạo OAuth Credentials

1. Menu trái → **Credentials**
2. Click **+ CREATE CREDENTIALS** → **OAuth client ID**
3. **Application type**: Web application
4. **Name**: `Botble SSO`
5. **Authorized redirect URIs**: Click **+ ADD URI**
   
   ```
   Thêm các URI sau:
   - https://your-domain.com/sso/google/callback
   - http://localhost:8000/sso/google/callback (cho dev)
   ```

6. Click **CREATE**

### Bước 5: Lưu Credentials

Sau khi tạo, bạn sẽ thấy popup với:
- **Client ID**: `xxxxxxxxxxxxx.apps.googleusercontent.com`
- **Client Secret**: `GOCSPX-xxxxxxxxxx`

> ⛔ **QUAN TRỌNG**: Lưu lại ngay! Client Secret chỉ hiển thị một lần.

### Thông tin cấu hình Google:

| Field | Value |
|-------|-------|
| Type | OIDC |
| Authorization URL | `https://accounts.google.com/o/oauth2/v2/auth` |
| Token URL | `https://oauth2.googleapis.com/token` |
| User Info URL | `https://openidconnect.googleapis.com/v1/userinfo` |
| Scopes | `openid email profile` |

---

## 3. Cài Đặt GitHub OAuth

### Bước 1: Truy cập GitHub Developer Settings

1. Đi đến: https://github.com/settings/developers
2. Đăng nhập GitHub

### Bước 2: Tạo OAuth App

1. Click **OAuth Apps** ở menu trái
2. Click **New OAuth App**
3. Điền thông tin:

| Field | Value |
|-------|-------|
| Application name | `Botble SSO` |
| Homepage URL | `https://your-domain.com` |
| Application description | (tùy chọn) |
| Authorization callback URL | `https://your-domain.com/sso/github/callback` |

4. Click **Register application**

### Bước 3: Lấy Credentials

1. Sau khi tạo, bạn sẽ thấy **Client ID**
2. Click **Generate a new client secret**
3. Copy **Client Secret** ngay

> ⛔ Client Secret chỉ hiển thị một lần. Lưu lại ngay!

### Thông tin cấu hình GitHub:

| Field | Value |
|-------|-------|
| Type | OAuth2 |
| Authorization URL | `https://github.com/login/oauth/authorize` |
| Token URL | `https://github.com/login/oauth/access_token` |
| User Info URL | `https://api.github.com/user` |
| Scopes | `read:user user:email` |

### Claim Mapping cho GitHub:

```json
{
    "email": "email",
    "name": "name|login",
    "avatar": "avatar_url"
}
```

> 📝 GitHub trả về `login` (username) thay vì `name` nếu user không set tên. Plugin tự động xử lý điều này.

---

## 4. Cài Đặt Microsoft OAuth

### Bước 1: Truy cập Azure Portal

1. Đi đến: https://portal.azure.com/
2. Đăng nhập bằng tài khoản Microsoft

### Bước 2: Đăng ký Application

1. Tìm kiếm **App registrations** → Click vào
2. Click **+ New registration**
3. Điền thông tin:

| Field | Value |
|-------|-------|
| Name | `Botble SSO` |
| Supported account types | **Accounts in any organizational directory and personal Microsoft accounts** |
| Redirect URI | Web - `https://your-domain.com/sso/microsoft/callback` |

4. Click **Register**

### Bước 3: Lấy Client ID

1. Sau khi tạo, vào **Overview**
2. Copy **Application (client) ID** - đây là Client ID

### Bước 4: Tạo Client Secret

1. Menu trái → **Certificates & secrets**
2. Click **+ New client secret**
3. **Description**: `Botble SSO`
4. **Expires**: Chọn thời hạn (khuyến nghị 24 months)
5. Click **Add**
6. Copy **Value** ngay - đây là Client Secret

> ⛔ Secret Value chỉ hiển thị một lần. Lưu lại ngay!

### Bước 5: Cấu hình API Permissions

1. Menu trái → **API permissions**
2. Click **+ Add a permission**
3. Chọn **Microsoft Graph**
4. Chọn **Delegated permissions**
5. Tìm và chọn:
   - `openid`
   - `email`
   - `profile`
   - `User.Read`
6. Click **Add permissions**

### Thông tin cấu hình Microsoft:

| Field | Value |
|-------|-------|
| Type | OIDC |
| Authorization URL | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Token URL | `https://login.microsoftonline.com/common/oauth2/v2.0/token` |
| User Info URL | `https://graph.microsoft.com/v1.0/me` |
| Scopes | `openid email profile User.Read` |

### Claim Mapping cho Microsoft:

```json
{
    "email": "mail|userPrincipalName",
    "name": "displayName",
    "first_name": "givenName",
    "last_name": "surname"
}
```

---

## 5. Cài Đặt Authentik

### Bước 1: Truy cập Authentik Admin

1. Đi đến Authentik admin: `https://your-authentik-domain/if/admin/`
2. Đăng nhập với tài khoản admin

### Bước 2: Tạo Provider

1. Menu trái → **Applications** → **Providers**
2. Click **Create**
3. Chọn **OAuth2/OpenID Provider**
4. Điền thông tin:

| Field | Value |
|-------|-------|
| Name | `Botble SSO` |
| Authorization flow | `default-provider-authorization-explicit-consent` |
| Client type | Confidential |
| Client ID | (tự động tạo hoặc nhập tùy chỉnh) |
| Client Secret | (tự động tạo hoặc nhập tùy chỉnh) |
| Redirect URIs/Origins | `https://your-domain.com/sso/authentik/callback` |
| Scopes | `openid email profile` |

5. Click **Create**

### Bước 3: Tạo Application

1. Menu trái → **Applications** → **Applications**
2. Click **Create**
3. Điền thông tin:

| Field | Value |
|-------|-------|
| Name | `Botble CMS` |
| Slug | `botble-cms` |
| Provider | Chọn `Botble SSO` (vừa tạo) |

4. Click **Create**

### Bước 4: Lấy URLs

1. Vào Provider vừa tạo
2. Lấy các URL từ phần **OpenID Configuration**

Hoặc truy cập: `https://your-authentik-domain/application/o/{application-slug}/.well-known/openid-configuration`

### Thông tin cấu hình Authentik:

| Field | Value |
|-------|-------|
| Type | OIDC |
| Authorization URL | `https://your-authentik-domain/application/o/authorize/` |
| Token URL | `https://your-authentik-domain/application/o/token/` |
| User Info URL | `https://your-authentik-domain/application/o/userinfo/` |
| Scopes | `openid email profile` |

---

## 6. Cấu Hình Trong Botble Admin

### Bước 1: Truy cập SSO Management

1. Đăng nhập Botble Admin
2. Menu trái → **Settings** → **SSO Providers**
3. Click **Create**

### Bước 2: Điền Thông Tin Provider

#### Tab: Basic Info

| Field | Mô tả |
|-------|-------|
| Name | Tên hiển thị (VD: "Google", "GitHub") |
| Slug | URL-friendly identifier (VD: "google", "github") |
| Type | OIDC hoặc OAuth2 |
| Status | Enabled/Disabled |
| Button Text | Text hiển thị trên nút login (VD: "Đăng nhập với Google") |

#### Tab: URLs

| Field | Mô tả |
|-------|-------|
| Authorization URL | URL để redirect user đến provider |
| Token URL | URL để đổi code lấy access token |
| User Info URL | URL để lấy thông tin user |

#### Tab: Credentials

| Field | Mô tả |
|-------|-------|
| Client ID | ID từ provider |
| Client Secret | Secret từ provider (sẽ được mã hóa) |
| Scopes | Permissions yêu cầu (VD: "openid email profile") |

#### Tab: User Types

| Field | Mô tả |
|-------|-------|
| Admin Enabled | Cho phép Admin login qua SSO |
| Admin Scopes | Scopes riêng cho Admin (optional) |
| Member Enabled | Cho phép Member login qua SSO |
| Member Scopes | Scopes riêng cho Member (optional) |

#### Tab: Advanced (Optional)

| Field | Mô tả |
|-------|-------|
| Claim Mapping | JSON mapping cho user attributes |
| Extra | Cấu hình bổ sung dạng JSON |

### Bước 3: Ví Dụ Cấu Hình Google

```
Name: Google
Slug: google
Type: OIDC
Authorization URL: https://accounts.google.com/o/oauth2/v2/auth
Token URL: https://oauth2.googleapis.com/token
User Info URL: https://openidconnect.googleapis.com/v1/userinfo
Client ID: [your-client-id].apps.googleusercontent.com
Client Secret: [your-client-secret]
Scopes: openid email profile
Admin Enabled: ✓
Member Enabled: ✓
Button Text: Đăng nhập với Google
```

### Bước 4: Lưu và Test

1. Click **Save**
2. Đi đến trang login Admin hoặc Member
3. Bạn sẽ thấy nút "Đăng nhập với Google" (hoặc tên provider)
4. Click để test

---

## 7. Xử Lý Lỗi Thường Gặp

### Lỗi: "redirect_uri_mismatch"

**Nguyên nhân**: Callback URL trong Botble không khớp với URL đã đăng ký ở provider.

**Giải pháp**:
1. Kiểm tra URL trong provider settings
2. Đảm bảo URL chính xác, bao gồm:
   - Protocol (http vs https)
   - Domain
   - Path (`/sso/{slug}/callback`)

### Lỗi: "invalid_client"

**Nguyên nhân**: Client ID hoặc Client Secret sai.

**Giải pháp**:
1. Copy lại Client ID và Secret từ provider
2. Paste cẩn thận, không có space thừa
3. Lưu lại trong Botble admin

### Lỗi: "access_denied"

**Nguyên nhân**: User từ chối quyền hoặc app chưa được approve.

**Giải pháp** (cho Google):
1. Vào Google Cloud Console → OAuth consent screen
2. Nếu app đang ở "Testing", thêm email user vào Test users
3. Hoặc submit app để Google review (cho production)

### Lỗi: "Email đã tồn tại"

**Nguyên nhân**: Đã có user với email này trong hệ thống.

**Giải pháp hiện tại**:
- SSO sẽ tự động login user hiện có (email match)
- Không tạo account mới

### Lỗi: "Could not verify state"

**Nguyên nhân**: Session hết hạn hoặc CSRF token không khớp.

**Giải pháp**:
1. Thử login lại từ đầu
2. Kiểm tra session config trong Laravel
3. Đảm bảo cookie hoạt động đúng

---

## Quick Reference Card

### Google OAuth
```
Auth URL:  https://accounts.google.com/o/oauth2/v2/auth
Token URL: https://oauth2.googleapis.com/token
User URL:  https://openidconnect.googleapis.com/v1/userinfo
Scopes:    openid email profile
```

### GitHub OAuth
```
Auth URL:  https://github.com/login/oauth/authorize
Token URL: https://github.com/login/oauth/access_token
User URL:  https://api.github.com/user
Scopes:    read:user user:email
```

### Microsoft OAuth
```
Auth URL:  https://login.microsoftonline.com/common/oauth2/v2.0/authorize
Token URL: https://login.microsoftonline.com/common/oauth2/v2.0/token
User URL:  https://graph.microsoft.com/v1.0/me
Scopes:    openid email profile User.Read
```

### Authentik
```
Auth URL:  https://{domain}/application/o/authorize/
Token URL: https://{domain}/application/o/token/
User URL:  https://{domain}/application/o/userinfo/
Scopes:    openid email profile
```

---

## Debug Tips

Nếu gặp lỗi, kiểm tra Laravel log:
```bash
tail -f storage/logs/laravel.log
```

Hoặc bật debug mode trong `.env`:
```
APP_DEBUG=true
```
