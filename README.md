# SSO Plugin Setup Guide

A comprehensive guide to configuring Single Sign-On (SSO) providers for Botble CMS.

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Google OAuth Setup](#2-google-oauth-setup)
3. [GitHub OAuth Setup](#3-github-oauth-setup)
4. [Microsoft Azure AD Setup](#4-microsoft-azure-ad-setup)
5. [Authentik Setup](#5-authentik-setup)
6. [Keycloak Setup](#6-keycloak-setup)
7. [Configuring Providers in Botble Admin](#7-configuring-providers-in-botble-admin)
8. [Testing Your Configuration](#8-testing-your-configuration)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Prerequisites

### 1.1. Install and Activate the Plugin

Ensure the SSO plugin is installed and migrations are run:

```bash
cd /path/to/your/botble
php artisan migrate
```

### 1.2. Understanding Callback URLs

The callback URL is where the OAuth provider redirects users after authentication. The format is:

```
https://your-domain.com/sso/{provider-slug}/callback
```

**Examples:**
| Provider | Callback URL |
|----------|-------------|
| Google | `https://your-domain.com/sso/google/callback` |
| GitHub | `https://your-domain.com/sso/github/callback` |
| Microsoft | `https://your-domain.com/sso/microsoft/callback` |
| Authentik | `https://your-domain.com/sso/authentik/callback` |

> ⚠️ **Important for Local Development:**
> - Use `http://localhost:8000/sso/{slug}/callback` with `php artisan serve`
> - Or use `http://127.0.0.1:8000/sso/{slug}/callback`
> - **DO NOT** use `.test` or `.local` domains - Google and Microsoft reject these

---

## 2. Google OAuth Setup

### Step 1: Access Google Cloud Console

1. Navigate to [Google Cloud Console](https://console.cloud.google.com/)
2. Sign in with your Google account

### Step 2: Create a New Project

1. Click the project dropdown at the top left
2. Click **"New Project"**
3. Enter a project name (e.g., `My Website SSO`)
4. Click **"Create"**
5. Wait for the project to be created, then select it

### Step 3: Configure OAuth Consent Screen

1. In the left sidebar, go to **APIs & Services** → **OAuth consent screen**
2. Select **"External"** user type → Click **"Create"**
3. Fill in the required fields:

| Field | Value |
|-------|-------|
| App name | Your website name |
| User support email | Your email address |
| App logo | (Optional) Your logo |
| App domain | Your website URL |
| Developer contact information | Your email address |

4. Click **"Save and Continue"**

5. **Scopes Configuration:**
   - Click **"Add or Remove Scopes"**
   - Select the following scopes:
     - `../auth/userinfo.email`
     - `../auth/userinfo.profile`
     - `openid`
   - Click **"Update"** → **"Save and Continue"**

6. **Test Users** (for apps in testing mode):
   - Add email addresses that can test the login
   - Click **"Save and Continue"**

7. Review summary and click **"Back to Dashboard"**

### Step 4: Create OAuth 2.0 Credentials

1. Go to **APIs & Services** → **Credentials**
2. Click **"+ Create Credentials"** → **"OAuth client ID"**
3. Configure the client:

| Field | Value |
|-------|-------|
| Application type | Web application |
| Name | `Botble SSO Client` |

4. Under **Authorized redirect URIs**, click **"+ Add URI"** and add:
   ```
   https://your-domain.com/sso/google/callback
   ```
   
   For local development, also add:
   ```
   http://localhost:8000/sso/google/callback
   ```

5. Click **"Create"**

### Step 5: Save Your Credentials

A popup will display your credentials:
- **Client ID**: `123456789-xxxxx.apps.googleusercontent.com`
- **Client Secret**: `GOCSPX-xxxxxxxxxx`

> ⛔ **CRITICAL**: Save these immediately! The Client Secret is only shown once.

### Google Configuration Summary

| Field | Value |
|-------|-------|
| Type | OIDC |
| Slug | `google` |
| Authorization URL | `https://accounts.google.com/o/oauth2/v2/auth` |
| Token URL | `https://oauth2.googleapis.com/token` |
| User Info URL | `https://openidconnect.googleapis.com/v1/userinfo` |
| Scopes | `openid email profile` |

---

## 3. GitHub OAuth Setup

### Step 1: Access GitHub Developer Settings

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Sign in to your GitHub account

### Step 2: Create an OAuth App

1. Click **"OAuth Apps"** in the left sidebar
2. Click **"New OAuth App"**
3. Fill in the application details:

| Field | Value |
|-------|-------|
| Application name | `My Website SSO` |
| Homepage URL | `https://your-domain.com` |
| Application description | (Optional) Description of your app |
| Authorization callback URL | `https://your-domain.com/sso/github/callback` |

4. Click **"Register application"**

### Step 3: Generate Client Secret

1. After registration, you'll see your **Client ID**
2. Click **"Generate a new client secret"**
3. Copy the secret immediately

> ⛔ **CRITICAL**: The secret is only displayed once. Save it immediately!

### Step 4: (Optional) Upload a Logo

Upload an application logo to make the login screen more professional.

### GitHub Configuration Summary

| Field | Value |
|-------|-------|
| Type | OAuth2 |
| Slug | `github` |
| Authorization URL | `https://github.com/login/oauth/authorize` |
| Token URL | `https://github.com/login/oauth/access_token` |
| User Info URL | `https://api.github.com/user` |
| Scopes | `read:user user:email` |

### GitHub Claim Mapping

GitHub returns user data in a different format. Use this claim mapping:

```json
{
    "email": "email",
    "name": "name|login",
    "avatar": "avatar_url"
}
```

> 📝 **Note**: The `name|login` syntax means "use `name` if available, otherwise fall back to `login` (username)".

---

## 4. Microsoft Azure AD Setup

### Step 1: Access Azure Portal

1. Go to [Azure Portal](https://portal.azure.com/)
2. Sign in with your Microsoft account

### Step 2: Register a New Application

1. Search for **"App registrations"** in the search bar
2. Click on **App registrations**
3. Click **"+ New registration"**
4. Configure the application:

| Field | Value |
|-------|-------|
| Name | `Botble CMS SSO` |
| Supported account types | Accounts in any organizational directory and personal Microsoft accounts |
| Redirect URI | Web - `https://your-domain.com/sso/microsoft/callback` |

5. Click **"Register"**

### Step 3: Get Application (Client) ID

1. After registration, you'll be on the app's **Overview** page
2. Copy the **Application (client) ID** - this is your Client ID

### Step 4: Create a Client Secret

1. In the left sidebar, click **"Certificates & secrets"**
2. Click **"+ New client secret"**
3. Configure:

| Field | Value |
|-------|-------|
| Description | `Botble SSO Secret` |
| Expires | 24 months (recommended) |

4. Click **"Add"**
5. **Immediately copy the "Value"** - this is your Client Secret

> ⛔ **CRITICAL**: The secret value is only shown once! Copy it immediately.

### Step 5: Configure API Permissions

1. In the left sidebar, click **"API permissions"**
2. Click **"+ Add a permission"**
3. Select **"Microsoft Graph"**
4. Choose **"Delegated permissions"**
5. Search for and select:
   - `openid`
   - `email`
   - `profile`
   - `User.Read`
6. Click **"Add permissions"**
7. (Optional) Click **"Grant admin consent"** if you're an admin

### Microsoft Configuration Summary

| Field | Value |
|-------|-------|
| Type | OIDC |
| Slug | `microsoft` |
| Authorization URL | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Token URL | `https://login.microsoftonline.com/common/oauth2/v2.0/token` |
| User Info URL | `https://graph.microsoft.com/v1.0/me` |
| Scopes | `openid email profile User.Read` |

### Microsoft Claim Mapping

Microsoft Graph returns data in a specific format. Use this mapping:

```json
{
    "email": "mail|userPrincipalName",
    "name": "displayName",
    "first_name": "givenName",
    "last_name": "surname"
}
```

> 📝 **Note**: Some Microsoft accounts don't have a `mail` field, so we fall back to `userPrincipalName`.

---

## 5. Authentik Setup

[Authentik](https://goauthentik.io/) is a popular open-source identity provider.

### Step 1: Access Authentik Admin Interface

1. Navigate to your Authentik admin: `https://your-authentik-domain/if/admin/`
2. Log in with an administrator account

### Step 2: Create an OAuth2/OpenID Provider

1. Go to **Applications** → **Providers** in the left sidebar
2. Click **"Create"**
3. Select **"OAuth2/OpenID Provider"**
4. Configure the provider:

| Field | Value |
|-------|-------|
| Name | `Botble CMS` |
| Authentication flow | `default-authentication-flow` |
| Authorization flow | `default-provider-authorization-explicit-consent` |
| Client type | Confidential |
| Client ID | Auto-generated or custom |
| Client Secret | Auto-generated or custom |
| Redirect URIs/Origins | `https://your-domain.com/sso/authentik/callback` |
| Signing Key | Select your signing key |

5. Under **Advanced protocol settings**:
   - Scopes: Select `email`, `openid`, `profile`
   
6. Click **"Create"**

### Step 3: Create an Application

1. Go to **Applications** → **Applications**
2. Click **"Create"**
3. Configure:

| Field | Value |
|-------|-------|
| Name | `Botble CMS` |
| Slug | `botble-cms` |
| Provider | Select the provider you just created |
| Launch URL | `https://your-domain.com` |

4. Click **"Create"**

### Step 4: Get Provider URLs

You can find the URLs in two ways:

**Option A**: Check the Provider Details
1. Go to **Applications** → **Providers**
2. Click on your provider
3. Find the URLs under **OpenID Configuration**

**Option B**: Access the Well-Known Endpoint
```
https://your-authentik-domain/application/o/{application-slug}/.well-known/openid-configuration
```

### Authentik Configuration Summary

| Field | Value |
|-------|-------|
| Type | OIDC |
| Slug | `authentik` |
| Authorization URL | `https://your-authentik-domain/application/o/authorize/` |
| Token URL | `https://your-authentik-domain/application/o/token/` |
| User Info URL | `https://your-authentik-domain/application/o/userinfo/` |
| Scopes | `openid email profile` |

---

## 6. Keycloak Setup

[Keycloak](https://www.keycloak.org/) is an open-source identity and access management solution.

### Step 1: Access Keycloak Admin Console

1. Go to your Keycloak admin: `https://your-keycloak-domain/admin/`
2. Log in with an administrator account

### Step 2: Create or Select a Realm

1. Hover over the realm dropdown (top left)
2. Either select an existing realm or click **"Create realm"**
3. If creating new: Enter a realm name and click **"Create"**

### Step 3: Create a Client

1. Go to **Clients** in the left sidebar
2. Click **"Create client"**
3. Configure General Settings:

| Field | Value |
|-------|-------|
| Client type | OpenID Connect |
| Client ID | `botble-cms` |

4. Click **"Next"**

5. Configure Capability Config:
   - Client authentication: **ON**
   - Authorization: OFF
   - Standard flow: ✓ Checked
   - Direct access grants: ✓ Checked

6. Click **"Next"**

7. Configure Login Settings:

| Field | Value |
|-------|-------|
| Root URL | `https://your-domain.com` |
| Valid redirect URIs | `https://your-domain.com/sso/keycloak/callback` |
| Web origins | `https://your-domain.com` |

8. Click **"Save"**

### Step 4: Get Client Secret

1. After saving, go to the **"Credentials"** tab
2. Copy the **Client secret**

### Keycloak Configuration Summary

| Field | Value |
|-------|-------|
| Type | OIDC |
| Slug | `keycloak` |
| Authorization URL | `https://your-keycloak-domain/realms/{realm}/protocol/openid-connect/auth` |
| Token URL | `https://your-keycloak-domain/realms/{realm}/protocol/openid-connect/token` |
| User Info URL | `https://your-keycloak-domain/realms/{realm}/protocol/openid-connect/userinfo` |
| Scopes | `openid email profile` |

Replace `{realm}` with your actual realm name (e.g., `master` or `myrealm`).

---

## 7. Configuring Providers in Botble Admin

### Step 1: Access SSO Provider Management

1. Log in to your Botble admin panel
2. Navigate to **SSO Providers** in the left menu
3. Click **"Create"** to add a new provider

### Step 2: Fill in Provider Details

#### Basic Information

| Field | Description | Example |
|-------|-------------|---------|
| Name | Display name for the provider | `Google` |
| Slug | URL-safe identifier (lowercase, no spaces) | `google` |
| Type | Protocol type | `OIDC` or `OAuth2` |
| Status | Enable/disable this provider | `Enabled` |
| Button Text | Custom text for login button | `Sign in with Google` |

#### OAuth URLs

| Field | Description |
|-------|-------------|
| Authorization URL | URL where users are redirected to authenticate |
| Token URL | URL to exchange authorization code for tokens |
| User Info URL | URL to fetch user profile information |

#### Credentials

| Field | Description |
|-------|-------------|
| Client ID | Your application's client ID from the provider |
| Client Secret | Your application's secret (stored encrypted) |
| Scopes | Space-separated list of permissions |

#### User Type Settings

| Field | Description |
|-------|-------------|
| Allow Admin | Enable SSO for admin users |
| Admin Scopes | Custom scopes for admin login (optional) |
| Allow Member | Enable SSO for member/frontend users |
| Member Scopes | Custom scopes for member login (optional) |

#### Advanced Settings (Optional)

| Field | Description |
|-------|-------------|
| Claim Mapping | JSON mapping for user attributes |
| Extra Config | Additional configuration in JSON format |

### Step 3: Save and Test

1. Click **"Save"** to create the provider
2. Visit your login page to see the new SSO button
3. Test the login flow

---

## 8. Testing Your Configuration

### Verify Provider is Active

Run this command to check your providers:

```bash
php artisan tinker --execute="echo 'Total: ' . \Botble\Sso\Models\SsoProvider::count() . '\n';"
```

### Test Login URLs

| User Type | URL |
|-----------|-----|
| Admin | `/sso/{slug}/redirect?guard=admin` |
| Member | `/sso/{slug}/redirect?guard=member` |

### Check SSO Buttons

1. Go to your admin login page: `/admin/login`
2. Go to your member login page: `/login`
3. You should see SSO buttons for each enabled provider

---

## 9. Troubleshooting

### Error: "redirect_uri_mismatch"

**Cause**: The callback URL configured in Botble doesn't match the one registered with the provider.

**Solution**:
1. Verify the exact callback URL in your provider settings
2. Ensure it matches: `https://your-domain.com/sso/{slug}/callback`
3. Check for:
   - Correct protocol (http vs https)
   - Trailing slashes
   - Exact domain match

### Error: "invalid_client"

**Cause**: Client ID or Client Secret is incorrect.

**Solution**:
1. Double-check the Client ID from your provider
2. Regenerate the Client Secret if necessary
3. Ensure no extra spaces when pasting

### Error: "access_denied"

**Cause**: User declined permissions or app is not approved.

**Solution (for Google)**:
1. Go to Google Cloud Console → OAuth consent screen
2. If in "Testing" status, add the user's email to Test Users
3. Or publish the app for production use

### Error: "invalid_state" or "Could not verify state"

**Cause**: Session expired or CSRF protection triggered.

**Solution**:
1. Clear browser cookies and try again
2. Check Laravel session configuration
3. Ensure `SESSION_DRIVER` is properly configured in `.env`

### Error: "User not found" or Login Fails Silently

**Cause**: User info endpoint returned unexpected data.

**Solution**:
1. Check Laravel logs: `tail -f storage/logs/laravel.log`
2. Verify claim mapping is correct
3. Ensure required scopes are granted

### SSO Buttons Not Appearing

**Cause**: Provider not enabled or no providers configured.

**Solution**:
1. Verify provider has `status = enabled`
2. Check `allow_admin` or `allow_member` is true
3. Clear cache: `php artisan cache:clear`

---

## Quick Reference Card

### Google
```
Type:      OIDC
Auth URL:  https://accounts.google.com/o/oauth2/v2/auth
Token URL: https://oauth2.googleapis.com/token
User URL:  https://openidconnect.googleapis.com/v1/userinfo
Scopes:    openid email profile
```

### GitHub
```
Type:      OAuth2
Auth URL:  https://github.com/login/oauth/authorize
Token URL: https://github.com/login/oauth/access_token
User URL:  https://api.github.com/user
Scopes:    read:user user:email
```

### Microsoft
```
Type:      OIDC
Auth URL:  https://login.microsoftonline.com/common/oauth2/v2.0/authorize
Token URL: https://login.microsoftonline.com/common/oauth2/v2.0/token
User URL:  https://graph.microsoft.com/v1.0/me
Scopes:    openid email profile User.Read
```

### Authentik
```
Type:      OIDC
Auth URL:  https://{domain}/application/o/authorize/
Token URL: https://{domain}/application/o/token/
User URL:  https://{domain}/application/o/userinfo/
Scopes:    openid email profile
```

### Keycloak
```
Type:      OIDC
Auth URL:  https://{domain}/realms/{realm}/protocol/openid-connect/auth
Token URL: https://{domain}/realms/{realm}/protocol/openid-connect/token
User URL:  https://{domain}/realms/{realm}/protocol/openid-connect/userinfo
Scopes:    openid email profile
```

---

## Debugging Tips

### Enable Debug Mode

In your `.env` file:
```
APP_DEBUG=true
```

### Check Laravel Logs

```bash
tail -f storage/logs/laravel.log
```

### Verify Routes Are Registered

```bash
php artisan route:list | grep sso
```

Expected output:
```
GET|HEAD  sso/{slug}/callback     sso.callback
GET|HEAD  sso/{slug}/redirect     sso.redirect
```

### Test Provider Connectivity

```bash
php artisan tinker
>>> $provider = \Botble\Sso\Models\SsoProvider::first();
>>> echo $provider->authorization_url;
>>> echo $provider->client_id;
```
