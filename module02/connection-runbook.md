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

## 3. Configure and save the PuTTY session

1. Open **PuTTY**. The **Session** category appears first.
2. In **Host Name (or IP address)**, enter the current server public IP exactly as it appears in the private course-shell connection note. Do not include `https://`, a username, spaces, or punctuation.
3. Set **Port** to **22** and select **SSH** as the connection type.
4. In the left **Category** tree, expand **Connection → SSH → Auth**, then select **Credentials**.
5. Beside **Private key file for authentication**, click **Browse**. Select `C:\Users\<your-Windows-name>\Documents\CIT386\keys\VM01_key.ppk` and click **Open**. Select the `.ppk`, not the `.pem` or `.pub` file.
6. In the left tree, select **Connection → Data**.
7. In **Auto-login username**, enter **`azureuser`**. This keeps the username with the saved session; it is not entered in the host-name box.
8. Select **Session** at the top of the left tree.
9. Under **Saved Sessions**, type **`CIT386-VM01`** and click **Save**. The name should now appear in the saved-session list.
10. Confirm the page still shows the course server address, port 22, and SSH. Click **Open**.

Tomorrow, open PuTTY, select `CIT386-VM01`, click **Load**, confirm the host address is current, and click **Open**. If the public IP changes in the course shell, update the address and click **Save** again before connecting.

## 4. First connection and a successful result

The first connection displays a **PuTTY Security Alert** stating that **the host key is not cached for this server**. This is expected on a clean computer. The host key identifies the SSH server; caching it lets PuTTY warn you if a different server presents a different key later.

Before continuing, confirm that the public IP in PuTTY exactly matches the private course-shell connection note. If it matches and this is the expected first connection, click **Accept** to cache the server identity and continue. If the address is unexpected, or a later connection says the host key changed, click **Cancel** and contact the instructor. Do not put the fingerprint in this document or in a screenshot.

A successful connection opens a black terminal window, authenticates with the key without asking for the server account password, and displays a Linux shell prompt similar to:

```text
azureuser@<server-name>:~$
```

The exact server name may differ. Success is the prompt ending in `$` with the cursor waiting for a command. The window must not show a PuTTY Fatal Error, `Access denied`, or a password prompt.

## 5. Validation performed

On September 27, 2026, I converted `VM01_key.pem` to `VM01_key.ppk`, confirmed TCP port 22 was reachable, encountered the expected uncached-host-key condition, and authenticated with PuTTY's SSH engine using the configured username and `.ppk` file. A remote test command completed with exit code 0. No key contents, fingerprints, or live public IP were copied into this repository.

## 6. Troubleshooting by exact message

Start with the first check in the table. Change one item at a time, then reconnect.

| Exact message | What it means | First thing to check |
|---|---|---|
| **`Network error: Connection timed out`** | PuTTY sent connection attempts but received no response before the timeout. Common causes are an old public IP, a stopped VM, or a network-security rule that does not allow SSH from the current network. | Compare **Host Name (or IP address)** with the current value in the private course shell. Correct and resave it if it changed. |
| **`Network error: Connection refused`** | The address answered, but no service accepted the connection on the selected port. | Return to **Session** and verify **Port** is `22` and **SSH** is selected. If both are correct, the server's SSH service or Azure network rule needs instructor attention. |
| **`Host does not exist`** | PuTTY could not interpret or resolve the value in the host-name box. | Make sure **Host Name (or IP address)** contains only the supplied address—no `https://`, `azureuser@`, slash, comma, or trailing space. |
| **`Unable to use key file`** | PuTTY could not open the selected file or it is not in a usable PuTTY private-key format. | Under **Connection → SSH → Auth → Credentials**, confirm the selected filename ends in `.ppk`. If only the `.pem` exists, repeat Section 2. |
| **`No supported authentication methods available (server sent: publickey)`** | The server requires a public-key login, but PuTTY did not offer a usable matching private key. | Confirm **Private key file for authentication** points to `VM01_key.ppk`, not `VM01_key.pub` or `VM01_key.pem`. |
| **`Server refused our key`** | PuTTY offered a key, but the server did not accept its matching public key for this account. | Check **Connection → Data → Auto-login username** is exactly `azureuser`, then verify that the `.ppk` was converted from the key issued for this VM. |
| **`Access denied`** | Authentication failed after a login attempt. This usually means the username is wrong, the wrong key pair is selected, or a password was entered even though the server expects a key. | Load `CIT386-VM01`, recheck the username, and browse to the correct `VM01_key.ppk`; do not guess a password. |

If these first checks do not resolve the problem, stop changing settings and send the instructor the exact error message, the time of the attempt, and which first check was completed. Do not send any key file, key text, fingerprint, or screenshot containing them.

## 7. Clear-setup retest

To verify that the runbook is complete:

1. Close the SSH terminal and PuTTY.
2. Reopen PuTTY, select `CIT386-VM01` under **Saved Sessions**, and click **Delete**.
3. Confirm the session name disappears.
4. Begin again at Section 1 using only this document and the private course-shell connection note.
5. Recreate and save the session, connect, and confirm the Linux `$` prompt appears.
6. Add any missing click, filename, or field to this runbook immediately rather than relying on memory.
