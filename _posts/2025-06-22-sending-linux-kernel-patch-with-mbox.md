---
title: Sending a Patch to the Linux Kernel from an .mbox File
categories: [Open Source Software Development, Kernel Contribuitions, MAC0470]
tags: [linux, kernal, refactoring, patch, iio, open-source, kernel-linux]
render_with_liquid: false
---
Recently, I needed to send a patch to the Linux kernel using an `.mbox` file sent by a monitor from the Free Software course. Here, I share the step-by-step process I followed, the difficulties I encountered, and some tips that might help anyone in the same situation.

---

## The Context: Receiving the `.mbox` Patch

The patch did not come in the traditional diff or `.patch` file format but rather as an `.mbox` file — a format used to store emails, common in patch submissions via the kernel mailing list.

I received guidance from David Tadokoro, who warned me that the version of the patch I had was corrupted and did not apply correctly to the kernel tree, showing the error:

```
error: corrupt patch at line 76
Patch failed at 0001 iio: light : veml6030: Remove code duplication
```

He sent me version v1 of the patch in `.mbox` format that applied correctly using:

```bash
git am <path-to-mbox-file>
```

This command "rebuilds" the commit from the patch and makes local corrections easier with `git commit --amend`.

---

## Steps I Followed in the Terminal

Below is a summary of the commands I used to prepare the repository, apply the patch, and send it:

```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git checkout master
git pull

git am patch.mbox    # Apply the patch received in mbox format

git log -1           # Verify the applied commit
git format-patch -1 HEAD  # Generate patch for sending

git config user.email "gabriellimamoraes@ime.usp.br"
git config user.name "Gabriel Lima de Moraes"

git send-email --to="iio@lists.linux.dev" --cc="linux-iio@vger.kernel.org" HEAD~1
```

---

## Problems I Encountered

### 1. Corrupted Patch

As David mentioned, the original patch had corrupted lines causing `git am` to fail. Always verify patch integrity before trying to apply it.

### 2. Case Sensitivity on macOS

My local environment is a MacBook, whose filesystem is **case insensitive by default**. This caused some subtle problems with filenames and paths expected by Linux, which is case sensitive.

To avoid confusion, I recommend doing patch work in a native Linux environment or in a virtual machine/container, or at least paying close attention to patch filenames and paths.

### 3. Using `--dry-run` in `git send-email`

I did some tests with the `--dry-run` flag to simulate patch sending without actually sending the email. This is useful to check if the sending is configured correctly, but don’t forget to remove this flag when you want to actually send it.

---

## Lessons Learned

- Understanding the `.mbox` format and how to use `git am` is essential to contribute to the Linux kernel, as many patches are sent by email.
- Always ensure your local repository is clean and up to date before applying patches.
- Set the user configurations (`user.name` and `user.email`) so your commits have the correct authorship.
- Test sending with `--dry-run` to avoid accidental sends.
- Be especially careful with the environment where you apply the patch — filesystem differences can cause headaches.

---

## Patch details

```git
gabriellima@Gabriels-MacBook-Pro linux % git send-email --to="iio@lists.linux.dev" --cc="linux-iio@vger.kernel.org" HEAD~1

/var/folders/q8/5ns3_c0s02j19tb1zyv3868w0000gn/T/UPhdN3F8hQ/0001-iio-light-veml6030-Remove-code-duplication.patch
(mbox) Adding cc: Vitor Marques <vitor.marques@ime.usp.br> from line 'From: Vitor Marques <vitor.marques@ime.usp.br>'
(body) Adding cc: Vitor Marques <vitor.marques@ime.usp.br> from line 'Signed-off-by: Vitor Marques <vitor.marques@ime.usp.br>'
(body) Adding cc: Gabriel Lima <gabriellimamoraes@ime.usp.br> from line 'Co-developed-by: Gabriel Lima <gabriellimamoraes@ime.usp.br>'
(body) Adding cc: Gabriel José <gabrieljpe@ime.usp.br> from line 'Co-developed-by: Gabriel José <gabrieljpe@ime.usp.br>'

From: Gabriel Lima de Moraes <gabriellimamoraes@ime.usp.br>
To: iio@lists.linux.dev
Cc: linux-iio@vger.kernel.org,
	Vitor Marques <vitor.marques@ime.usp.br>,
	Gabriel Lima <gabriellimamoraes@ime.usp.br>,
	=?UTF-8?q?Gabriel=20Jos=C3=A9?= <gabrieljpe@ime.usp.br>
Subject: [PATCH] iio: light : veml6030 Remove code duplication
Date: Sun, 22 Jun 2025 20:17:34 -0300
Message-ID: <20250622231734.29684-1-gabriellimamoraes@ime.usp.br>
X-Mailer: git-send-email 2.43.0
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit

    The Cc list above has been expanded by additional
    addresses found in the patch commit message. By default
    send-email prompts before sending whenever this occurs.
    This behavior is controlled by the sendemail.confirm
    configuration setting.

    For additional information, run 'git send-email --help'.
    To retain the current behavior, but squelch this message,
    run 'git config --global sendemail.confirm auto'.

Send this email? ([y]es|[n]o|[e]dit|[q]uit|[a]ll): y
OK. Log says:
Sendmail: /usr/sbin/sendmail -i iio@lists.linux.dev linux-iio@vger.kernel.org vitor.marques@ime.usp.br gabriellimamoraes@ime.usp.br gabrieljpe@ime.usp.br
From: Gabriel Lima de Moraes <gabriellimamoraes@ime.usp.br>
To: iio@lists.linux.dev
Cc: linux-iio@vger.kernel.org,
	Vitor Marques <vitor.marques@ime.usp.br>,
	Gabriel Lima <gabriellimamoraes@ime.usp.br>,
	=?UTF-8?q?Gabriel=20Jos=C3=A9?= <gabrieljpe@ime.usp.br>
Subject: [PATCH] iio: light : veml6030 Remove code duplication
Date: Sun, 22 Jun 2025 20:17:34 -0300
Message-ID: <20250622231734.29684-1-gabriellimamoraes@ime.usp.br>
X-Mailer: git-send-email 2.43.0
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit

Result: OK
```

---



## Conclusion

Sending patches to the Linux kernel might seem intimidating, but with the right workflow and good practices, it becomes a smooth process. My experience involved technical challenges, communication with maintainers, and fine-tuning the environment, but it was a great learning opportunity.

If you are on this journey too, don’t hesitate to seek help from the community and validate your steps before sending.

---
