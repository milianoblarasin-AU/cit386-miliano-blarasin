# Connection Runbook: Windows to the CIT 386 Azure Server

This runbook takes a reader from a clean Windows computer to an SSH shell on the course Azure server. Follow the steps in order. Do not place key contents, fingerprints, or the server's live public IP in screenshots or a public repository.

## 1. Before you begin

You need:

1. A Windows 10 or Windows 11 computer with internet access.
2. **PuTTY 0.85 or later** installed. The standard 64-bit PuTTY installer includes both `putty.exe` and `puttygen.exe`.
3. The server connection note from the private course shell. It supplies the current public IP address and the login name `azureuser`.
4. These two course key files:
   - `VM01_key.pem` — the original **private** OpenSSH key.
   - `VM01_key.pub` — the matching **public** key.

In File Explorer, create `Documents\CIT386\keys` inside your Windows user profile. Put both key files there. For example, the folder will look like `C:\Users\<your-Windows-name>\Documents\CIT386\keys`.

Keep the private key on a trusted computer. Do not put either the `.pem` file or the converted `.ppk` file in this Git repository.

## 2. Convert the private key for PuTTY

PuTTY uses a `.ppk` private key. Convert the supplied `.pem` file once:

1. Open **PuTTYgen** from the Start menu.
2. Click **Load** under **Load an existing private key file**.
3. In the file picker, change **Files of type** to **All Files (`*.*`)** so the `.pem` file is visible.
4. Browse to `Documents\CIT386\keys`, select **`VM01_key.pem`**, and click **Open**.
5. PuTTYgen reports that it successfully imported a foreign key. Click **OK**. Do not copy the key text or fingerprint from the window.
6. Click **Save private key**. If the course key has no passphrase and PuTTYgen asks whether to save without one, click **Yes** only on this trusted course computer.
7. Save the converted file in the same folder as **`VM01_key.ppk`**. The output extension is **`.ppk`**.

After conversion, the folder should contain `VM01_key.pem`, `VM01_key.pub`, and `VM01_key.ppk`. The `.ppk` is the file PuTTY will use; the `.pub` is not selected in PuTTY.
