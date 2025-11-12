# Guide on running code assist on Gensyn
This Guide is for everyone who 's personally running the gensyn node before

---

# it's a Simple Guide for Installing & Running Gensyn CodeAssist on a VPS

This guide walks you through how to install and run **Gensyn CodeAssist** on your VPS — especially if you’re running both your **Gensyn Node** and **CodeAssist** on the same server.

It also explains how to connect to the CodeAssist web interface from your **local machine (WSL)** using **SSH port forwarding**.

---

## **Step-by-Step Setup**

### **Step 1.  Clone the CodeAssist Repository**

```bash
git clone https://github.com/gensyn-ai/codeassist
```

---

### **Step 2.  Install UV (Astral Runtime)**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

---

### **Step 3.  Reload Your Shell**

```bash
source ~/.bashrc
```

> If you’re using Zsh, run `source ~/.zshrc` instead.

---

### **Step 4.  Go Into the Project Directory**

```bash
cd codeassist
```

---

### Step 5.  Change the Port (this is for VPS USers Running Both Gensyn Node & CodeAssist)

If your VPS already has a **Gensyn Node** running on port `3000`, you’ll need to open an external port for CodeAssist to avoid conflicts.

#### **Edit the Docker Compose file**

```bash
nano compose.yml
```

Find this section:

```yaml
ports:
  - "3000:3000"
```

Change it to:

```yaml
ports:
  - "3001:3000"
```

Save and exit ( Ctrl + X, press y and Enter ).

---

#### **Edit the `run.py` File**

```bash
nano run.py
```

Find where the web UI port is defined and change `3000` to `3001`.

Example:

```python
# Before
uv.run("app:app", host="0.0.0.0", port=3000)

# After
uv.run("app:app", host="0.0.0.0", port=3001)
```

Save and exit the file.

---

### **Step 6.  Run CodeAssist**

```bash
uv run run.py
```

When you’re prompted for your **Hugging Face token**:

* Paste it with **Ctrl + V** (it won’t be visible on screen).
* Press **Enter** when done.

---

### **Step 7.  Forward Ports to Your Local Machine (WSL)**

Since your VPS doesn’t open `localhost` directly, you’ll need to forward ports to your WSL terminal.

Run this **on your local WSL**, replacing with your VPS username and IP:

```bash
ssh -L 3001:localhost:3001 \
    -L 8000:localhost:8000 \
    -L 8001:localhost:8001 \
    -L 8008:localhost:8008 \
    username@your_vps_ip
```

Here’s what each port is for:

* `3001` → CodeAssist Web UI
* `8000`, `8001`, `8008` → Optional (for other services like Gensyn Node)

If your VPS uses a public SSH key, you may just need to confirm with `yes` and enter your password.

---

## Optional: Use SSH Key Authentication (Recommended)

If you’d rather not type your password every time you connect, you can set up SSH key authentication.

---

### **Step 1.  Generate an SSH Key**

```bash
ssh-keygen -t ed25519 -C "rayyy"
```

> Replace `"rayyy"` with anything you want.
> When asked for a passphrase, just hit **Enter** three times (no need to set one).

---

### **Step 2.  Display Your Public Key**

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire line that appears — this is your **public SSH key**.

---

### **Step 3.  Add Your Key to the VPS**

If your VPS provider doesn’t allow you to upload SSH keys after setup, you can still add it manually using the VNC console.

On your VPS terminal, run:

```bash
nano ~/.ssh/authorized_keys
```

Paste your copied key inside the file, then save:

```
Ctrl + O → Enter → Ctrl + X
```

---

> ⚠️ For VPS hosts like **Serverica**, SSH keys can’t be added later from the dashboard. Use the **VNC console** method instead to add it manually — once done, your local system will connect without needing a password.

---

### **Step 4.  Connect Using Your Key**

Now you can reconnect easily using the same SSH port forwarding command:

```bash
ssh -L 3001:localhost:3001 \
    -L 8000:localhost:8000 \
    -L 8001:localhost:8001 \
    -L 8008:localhost:8008 \
    username@your_vps_ip
```

No password needed this time...it connects automatically

---

### **Step 8.  Access CodeAssist in Your Browser**

Once the SSH tunnel is open, head to:

```
http://localhost:3001
```

You’ll also see a clickable link in your terminal.
Log in with your email when prompted and you’re ready to use CodeAssist!

---

## Example Configuration

**Final `compose.yml` Example**

```yaml
services:
  codeassist:
    image: gensyn/codeassist:latest
    ports:
      - "3001:3000"
    environment:
      - HF_TOKEN=${HF_TOKEN}
```

**Example SSH Command**

```bash
ssh -L 3001:localhost:3001 -L 8000:localhost:8000 -L 8001:localhost:8001 -L 8008:localhost:8008 username@your_vps_ip
```

---

## Tips & Notes

* When pasting your Hugging Face token, it won’t display - that’s normal.
* Keep your SSH window open while using CodeAssist (it closes if you exit the terminal).
* If port `3001` is already taken, use another port (like `3002`) and update both files.
* If you’re not on a VPS, you can skip SSH forwarding and just open `http://localhost:3000`.
* For security, never expose your CodeAssist ports publicly - always use SSH tunnels.
* SSH keys are faster, safer, and save time for future logins.

---

**You’re all set!**
Once logged in, you can start using **Gensyn CodeAssist** to write, run, and debug code directly from your browser - powered by your own VPS setup.

---
