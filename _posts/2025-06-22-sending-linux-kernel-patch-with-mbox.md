---
title: Sending a Patch to the Linux Kernel from an .mbox File (and Solving SMTP Issues)
categories: [Open Source Software Development, Kernel Contributions, MAC0470]
tags: [linux, kernel, refactoring, patch, iio, open-source, kernel-linux, git-send-email, smtp]
render_with_liquid: false
---

Recently, I needed to send a patch to the Linux kernel for the IIO subsystem. The patch came as an `.mbox` file, and the task involved several steps: applying the patch, generating a clean commit, and sending it via email using `git send-email`.

Along the way, I encountered a common but frustrating problem: SMTP misconfiguration.

---

## The Context: Receiving the `.mbox` Patch

The patch did not arrive as a traditional `.diff` or `.patch` file. Instead, it came in `.mbox` format — a format used to store full emails, often used for mailing list workflows.

After attempting to apply the original patch, I encountered this error:

```
error: corrupt patch at line 76
Patch failed at 0001 iio: light : veml6030: Remove code duplication
```

David Tadokoro kindly sent me a correct `.mbox` version of the patch. Applying it was straightforward:

```bash
git am patch.mbox
```

---

## SMTP Problem: Sending from my IME-USP Email Failed

Initially, I tried sending the patch from my institutional email `gabriellimamoraes@ime.usp.br`.  

However, I soon realized that I didn't have SMTP properly configured for this account, and my attempts to send patches with `git send-email` were failing silently or being rejected by the mail server.

My `.gitconfig` had these settings at first:

```ini
[sendemail]
    smtpServer = smtp.ime.usp.br
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = gabriellimamoraes@ime.usp.br
    smtpPass = <password>
```

But sending did not work.

---

## Solution: Switching to Gmail SMTP

To quickly resolve the issue, I switched to my personal Gmail account.

I updated my `.gitconfig`:

```ini
[sendemail]
    smtpServer = smtp.gmail.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = gabriellimamoraes@gmail.com
    smtpPass = <Gmail App Password>
```

Then, I tested with:

```bash
git send-email --smtp-debug=1 --to="linux-iio@vger.kernel.org" --cc="gabriellimamoraes@ime.usp.br" HEAD~1
```

The log showed a `250 OK` response from the SMTP server, indicating successful delivery.

---

## Correct Mailing List Address: <linux-iio@vger.kernel.org>

Another issue: initially I was trying to send the patch to `iio@lists.linux.dev`, which returned this error:

```
550 5.1.1 : Recipient address rejected: User unknown in virtual alias table
```

After checking the official [vger.kernel.org mailing lists](https://vger.kernel.org/vger-lists.html), I confirmed that the correct address for the IIO subsystem is:

```
linux-iio@vger.kernel.org
```

---

## Final Send Command

```bash
git send-email --to="linux-iio@vger.kernel.org" --cc="gabriellimamoraes@ime.usp.br" HEAD~1
```

Final result:

- Email successfully sent
- I received a copy in my inbox (Cc)
- The patch appeared on [lore.kernel.org](https://lore.kernel.org/linux-iio/)

---

## Lessons Learned

- **Always double-check the mailing list address** on official pages like [vger.kernel.org](https://vger.kernel.org/vger-lists.html)
- **Configure SMTP with a reliable provider**: Gmail worked well for me
- **Use `--smtp-debug=1` for troubleshooting**
- **Cc yourself** if you want a copy of your patch submission
- **Confirm on lore.kernel.org** to ensure your patch was published

---

 Contributing to the Linux kernel can be challenging, but it's a rewarding learning experience

## Link to my patch on lore.kernel.org

You can view the final result of my patch submission here:

👉 <https://lore.kernel.org/all/20250623201539.16148-1-gabriellimamoraes@gmail.com/>
