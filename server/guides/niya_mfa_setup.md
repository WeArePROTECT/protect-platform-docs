# MFA Setup Guide for Arkin Lab Server Access

**Simplified Companion to Keith's Full Doc**  
*Last updated: September 2025*

---

## 🧰 What You'll Need

- Your Arkin server username and temporary password (provided by Keith via Spencer)
- An OTP (one-time password) app on your phone:
  - ✅ Google Authenticator
  - ✅ Aegis (Android)
  - ✅ Authy
  - ⚠️ **Duo Mobile** only works if you add a new Niya-specific token (Your existing CalNet or LBL Duo accounts will not work here)

---

## 🛠️ Step-by-Step Setup

### 1. Connect to the Server

Use a terminal (Mac/Linux) or Windows Power Shell/PuTTY (Windows) and connect to:

```bash
ssh your_username@niya.qb3.berkeley.edu
```

Replace `your_username` with the one Keith gave you.

### 2. Initial Login

- Say **yes** to the fingerprint prompt (first time only)
- Enter your temporary password
- You'll see a message about MFA — you can ignore this for now
- You should then see:
  - A QR code
  - A secret key
  - A prompt: `Enter code from app (-1 to skip):`

### 3. Set Up MFA on Your Phone

1. Open your OTP app
2. Scan the QR code (If scanning fails, manually enter the secret key using your app's "Enter key manually" option)
3. Wait for a fresh 6-digit code
4. Enter that code into the terminal prompt

### 4. Save Emergency Codes

- You'll be shown a set of emergency scratch codes
- ✅ **Save them in a secure place** in case you lose your phone
- Enter `y` to save your `.google_authenticator` file

### 5. Verify MFA Works

After setup, test your token by running:

```bash
oathtool --totp -b $(head -1 ~/.google_authenticator) XXXXXX
```

Replace `XXXXXX` with the current code from your app.

✅ If the output is `0`, your token is valid.  
📩 **Let Keith know in the email thread that it worked.**

### 6. Change Your Password

While still logged in, type:

```bash
passwd
```

- For LDAP password, use your temporary password
- Then set your new password (enter twice to confirm)

### 7. Final Login Test

Open a new terminal window and try logging in again:

```bash
ssh your_username@niya.qb3.berkeley.edu
```

✅ You should now be prompted for:
- Your new password
- Your 6-digit MFA code from the app

🎉 **If both work, you're successfully onboarded to the niya server.**

---

## 🛀 Quick Reminders

- Don't exceed 3 failed login attempts — you may get temporarily locked out
- Make sure you wait for a fresh OTP code before entering
- CalNet or LBL Duo tokens will not work — you need a niya-specific token
- We will only be using the niya virtual machine (no need to worry about other servers)

---

## ✅ You're Set

Once you confirm login success, you're ready to move on to uploading data.
