# Nefertiti

## Challenge Description

We are given:

- `secret.enc`
- `nefertiti.png`

Challenge hint:
> Her name may still unlock secrets long forgotten.

The goal is to recover the hidden data from both files and reconstruct the full flag.

---

## Step 1: Analyze `secret.enc`

The file `secret.enc` is encrypted, but this time there is a very strong clue in its header.

If we inspect the beginning of the file with a hex viewer we see the ASCII string `Salted__` is a well-known marker added by **OpenSSL** when a file is encrypted using `openssl enc` with a password.

So from that header, we can immediately conclude that:

- the file was encrypted with **OpenSSL**
- it uses a password-based key derivation process
- we should try OpenSSL decryption directly

---

## Step 2: Find the correct OpenSSL decryption method

Since the file is clearly OpenSSL-encrypted, the next step is to test the common OpenSSL cipher options.

A standard modern combination is:

- `aes-256-cbc`
- `pbkdf2`

And the password is clearly "nefertiti" based on the challenge description (Her name may still unlock secrets long forgotten)

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out decrypted.bin -pass pass:nefertiti
```

## Step 3: Identify the decrypted file type

Opening `decrypted.bin` in a hex editor shows that it resembles a **corrupted PNG**.

A valid PNG file should begin with this signature:

```text
89 50 4E 47 0D 0A 1A 0A
```

Comparing the decrypted data with a normal PNG makes it clear that the file is a PNG with a damaged header.

That means the decryption worked; the result just needs minor repair.

---

## Step 4: Repair the PNG header

We manually fix the first bytes in a hex editor so the file starts with the correct PNG signature:

```text
89 50 4E 47 0D 0A 1A 0A
```

After repairing the header, the file opens correctly as an image.

---

![QR](screenshots/decrypted.png)

## Step 5: Decode the recovered image

The repaired image is a **QR code**.

Decoding it gives:

```text
Pioneers25{qu33n_h1dd3n_
```

This is the first part of the flag.

---

## Step 6: Analyze `nefertiti.png`

Since nothing is visibly hidden in the picture, the next logical step is to test for steganography.

---

## Step 7: Extract hidden data with OpenStego

We start by trying any stegano command but we get nothing.
So we should think about image stegano tool like **OpenStego**.

We open the image with **OpenStego** and use the same password:

```text
nefertiti
```

The extraction succeeds and reveals:

```text
flag.txt
```

Its content is:

```text
und3rn34th_th3_du5t}
```

---

## Step 8: Combine both parts

From the repaired QR code:

```text
Pioneers25{qu33n_h1dd3n_
```

From `flag.txt` extracted from `nefertiti.png`:

```text
und3rn34th_th3_du5t}
```

Final flag:

```text
Pioneers25{qu33n_h1dd3n_und3rn34th_th3_du5t}
```

---
