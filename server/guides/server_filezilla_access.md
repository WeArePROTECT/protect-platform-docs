# How to Connect to the Arkin Lab Server with FileZilla

This guide explains how to set up FileZilla to connect to the Arkin Lab server using SFTP with OTP (One-Time Password) authentication.

---

## Step 1 – Open Site Manager

1. Launch FileZilla.
2. Go to **File > Site Manager**.
3. Click **New Site** and give it a name (e.g., "Arkin Server").

---

## Step 2 – Configure Connection

In the **General** tab, enter the following:

- **Protocol:** SFTP – SSH File Transfer Protocol
- **Host:** `niya.qb3.berkeley.edu`
- **Logon Type:** Interactive
- **User:** `<yourusername>`

---

## Step 3 – Connect

1. Click **Connect**.
2. FileZilla will first prompt you for your password.
3. Then it will prompt you for your OTP code (just like in the terminal).

✅ **Thanks to Vishant Gandhi for discovering this solution — setting Logon Type to Interactive is the key to making OTP work in FileZilla.**

---

## Step 4 – Transfer Files

Once connected, you can drag and drop files between your local machine and the server.

- Use the **left panel** for your local computer
- Use the **right panel** for the server

---

## Troubleshooting

- If FileZilla does not prompt for OTP, double-check that **Logon Type** is set to **Interactive**.
- Make sure you are connecting with **SFTP** (not FTP).
- If issues persist, test your login in the terminal using:

```bash
sftp yourusername@niya.qb3.berkeley.edu
```

---

## ⚠️ Gotchas & Solutions

**Problem:** FileZilla repeatedly asks for your password or times out after a few file uploads (e.g., "Connection timed out after 20 seconds of inactivity").

**Cause:** FileZilla opens multiple SFTP sessions when uploading multiple files at once. Since OTP authentication cannot be reused or entered automatically, new sessions fail once the first one expires.

**Solution (thanks to Emma Rooholfada):**

1. Go to **Edit → Settings → Transfers**
2. Set **Maximum simultaneous transfers** to `1`
3. Go to **Edit → Settings → Connection**
4. Set **Timeout in seconds** to the maximum (`9999`)

This keeps a single persistent session alive and prevents repeated password/OTP prompts during uploads.
