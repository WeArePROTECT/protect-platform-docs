# Uploading and Downloading Data to the Arkin Lab Server (Niya)

**Beginner-Friendly Guide for PROTECT Team Members**

*Last updated: September 2025*

---

## 🔎 What You Need to Know First

- You must have access to the **Niya** server (confirmed by Spencer and Keith)
- You do **not** need to be on VPN
- You just need an internet connection and your MFA set up
- All uploads and downloads will happen within your lab's folder inside:

```
/usr2/people/protect/
  Zengler_Lab/
  Conrad_Lab/
  Nizet_Lab/
  Arkin_Lab/
```

---

## 🚀 Uploading Data to the Server

### ✅ Option 1: rsync (Recommended for large files or folders)

```bash
rsync -avzP /local/path/to/data/ your_username@niya.qb3.berkeley.edu:/usr2/people/protect/Your_Lab/
```

**What it does:**
- `-a` keeps permissions and timestamps
- `-v` shows what's happening
- `-z` compresses files to speed up transfer
- `-P` shows progress and allows resuming if interrupted

### ✅ Option 2: scp (Simple but less robust)

```bash
scp /local/path/to/file.txt your_username@niya.qb3.berkeley.edu:/usr2/people/protect/Your_Lab/
```

Good for one-off file uploads.

### ✅ Option 3: GUI Tools (Beginner-Friendly)

1. Install **FileZilla** or **Cyberduck**
2. Use these settings:
   - **Protocol:** SFTP
   - **Host:** niya.qb3.berkeley.edu
   - **Username:** your Arkin Lab username
   - **Password + OTP code:** enter when prompted
3. Drag and drop files into your folder under `/usr2/people/protect/Your_Lab/`

---

## 📂 Downloading Data from the Server

### ✅ Using rsync

```bash
rsync -avzP your_username@niya.qb3.berkeley.edu:/usr2/people/protect/Your_Lab/results/ ./local_results/
```

### ✅ Using scp

```bash
scp your_username@niya.qb3.berkeley.edu:/usr2/people/protect/Your_Lab/file.txt ./file.txt
```

### ✅ Using FileZilla or Cyberduck

Same method as uploading, just drag files from the server back to your local machine.

---

## 📁 Directory Structure & Best Practices

```
/usr2/people/protect/Your_Lab/
  raw_data/
  processed_data/
  metadata/
  README.md
```

### ✅ README.md should include:
- What each folder contains
- Who uploaded the files and when
- Brief description of how data was generated or processed

### ✅ Naming Conventions

Use clear, descriptive file names:
- `zengler_metaG_sample23_R1.fastq.gz`
- `conrad_mouse01_metadata.csv`

**Avoid:**
- Spaces or special characters
- Use `snake_case` or `kebab-case`

---

## 📆 Recommended Compression

To upload a full folder:

```bash
tar -czvf processed_data.tar.gz processed_data/
```

---

## ⚠️ Common Pitfalls to Avoid

- Uploading to the wrong lab folder
- Forgetting to include a README.md or metadata file
- Using inconsistent file names
- Overwriting existing files without checking with your team
- Forgetting to notify Spencer once data is uploaded

---

## 👍 You're All Set

If you're unsure about anything, reach out to Spencer. We're here to make the process smooth and painless!

**Let Spencer know once your data is uploaded so we can ensure proper backup and tracking.**
