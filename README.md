# Week 3: Password Cracking

Networkwalks Cybersecurity & Ethical Hacking Internship, Batch B083

The lab guide only asks you to crack `My Locked PDF1.pdf`, but the actual download includes two more locked PDFs (`My Locked PDF2.pdf` and `My Locked PDF3.pdf`). Since they were sitting right there, I went ahead and cracked all three using the same process, in both modules. Module 1 does it the "proper" way with John the Ripper and its GUI, Johnny. Module 2 does the exact same thing using two lightweight web tools Networkwalks built themselves, so you can see the same process without installing anything.

Screenshots and text output for both modules are in the module folders (layout is at the bottom).

## Modules

| Module | Topic | Tools |
|--------|-------|-------|
| W3-PM1 | Password cracking with JTR | John the Ripper (jumbo, 1.9.0), Johnny 2.2 GUI |
| W3-PM2 | Password cracking with Networkwalks tools | Networkwalks Hash Calculator, Networkwalks Password Cracker |

## The idea behind both modules

A locked PDF doesn't store your password in plain text — it stores a hash of it. To crack it, you first pull that hash out of the file (this is what `pdf2john` does under the hood), then run a cracking tool against it that hashes candidate words from a wordlist and checks for a match. Once it finds one, that's your password. Whether you do that with JTR on your own machine or with a browser tool doesn't change the underlying process — and it doesn't change whether you're doing it against one file or three, which is basically what this week ended up demonstrating.

---

## W3-PM1: John the Ripper + Johnny (GUI)

Done on Windows, since that's what the lab asked for (Kali has JTR preinstalled if you'd rather skip the download).

### Setup

1. Downloaded John the Ripper (jumbo, 1.9.0, 64-bit) from [openwall.com/john](https://www.openwall.com/john/)
2. Downloaded Johnny 2.2 from [openwall.info/wiki/john/johnny](https://openwall.info/wiki/john/johnny) and ran the installer
3. Opened Johnny, went to **Settings**, and pointed "John the Ripper executable" at `john.exe` inside the extracted `run` folder. Johnny picked it up as `John the Ripper 1.9.0-jumbo-1 OMP`.

### Extracting the hashes

JTR can't read a PDF directly, so the hash has to be pulled out first. I used [onlinehashcrack.com's PDF hash extractor](https://www.onlinehashcrack.com/tools-pdf-hash-extractor.php), which is really just running `pdf2john` for you in the browser — it says outright that it doesn't store uploaded files. Ran each of the three PDFs through it one at a time and got back three hashes, each starting with `$pdf$4*4*128*-1060*1*16*...`.

One gotcha the guide calls out: if you copy a hash and it comes out with a `b'` at the start (this happens if you copy from certain terminal outputs), strip that off before saving — the hash needs to start clean with `$pdf$`.

Pasted each hash into Notepad and saved them as `hash1.txt`, `hash2.txt` and `hash3.txt`.

### Cracking them

Same steps in Johnny, repeated for each hash file:

1. **Open password file** → selected the hash file. It showed up as one entry with format `PDF`.
2. **Start new attack**.
3. Waited for the Password column to fill in.

### Results

| File | Password | Notes |
|------|----------|-------|
| My Locked PDF1.pdf | `password1` | Matches the example in the guide |
| My Locked PDF2.pdf | `password1` | Same password as PDF1 |
| My Locked PDF3.pdf | `1qaz2wsx` | Keyboard-walk pattern (top row → next row), still a known weak password |

All three cracked almost instantly — nothing here is remotely close to a strong password. Opened each PDF in Acrobat with its password and each one unlocked to a "Congratulations, you captured a flag" page.

---

## W3-PM2: Networkwalks Hash Calculator + Password Cracker

Same three files, same results, but using two of Networkwalks' own browser tools instead of installing anything.

**Hash Calculator** — [networkwalks.com/hash-calculator](https://networkwalks.com/hash-calculator/)
Has three tabs (Text / File / PDF). Used the PDF tab and dropped each locked PDF in, one at a time. It flagged each file as encrypted and gave back the same kind of `$pdf$...` hash that `pdf2john` produces — the page says explicitly that everything runs locally in the browser via the Web Crypto API and nothing gets uploaded.

**Password Cracker** — [networkwalks.com/password-cracker](https://networkwalks.com/password-cracker/)
This one runs a dictionary attack against a hash you give it. It ships with a built-in list of 100 common passwords, or you can upload your own `.txt` wordlist. Pasted in each hash in turn, left the built-in list selected, and hit **Start Cracking**.

It streams its attempts on screen as it goes (`Trying: service`, `Trying: canada`, `Trying: hockey`...) until it hits a match and displays "Password Cracked Successfully" with a copy button.

### Results

| File | Password | Attempts (out of 100) |
|------|----------|------------------------|
| My Locked PDF1.pdf | `password1` | 91 |
| My Locked PDF2.pdf | `password1` | 91 |
| My Locked PDF3.pdf | `1qaz2wsx` | 35 |

Same last step as Module 1 for each: open the PDF, type in its password, done.

### Comparing the two

Both modules land on the exact same three passwords through the exact same two-stage process (extract hash → dictionary attack). JTR/Johnny is the industry-standard way to do this and scales to way more than a 100-word list; the Networkwalks tools are a nice way to see the mechanics without setting anything up, since they're doing the same job — extraction, then attack — just in a browser tab.

---

## What I took away from this week

- Encryption and hashing aren't the same thing. Encryption is reversible with the right key; a password hash isn't reversible at all, which is exactly why cracking it means guessing and checking rather than decrypting anything.
- PDF1 and PDF2 sharing the same password is a good reminder of why password reuse is such an easy win for an attacker — crack it once, and every account or file using it falls too.
- `1qaz2wsx` looks more "complex" than `password1` at a glance, but it's a well-known keyboard-walk pattern and it was still in a 100-word common-password list. Complexity that follows a predictable pattern isn't the same as actual randomness.
- The extraction step matters as much as the cracking step. Whether it's `pdf2john` on the command line or a browser tool doing the same thing under the hood, you can't crack a hash you haven't pulled out of the file first.

## Repo layout

```
.
├── module1-jtr-johnny/
│   ├── screenshots/
│   └── outputs/
│       ├── hash1.txt
│       ├── hash2.txt
│       └── hash3.txt
└── module2-networkwalks-tools/
    └── screenshots/
```

## Disclaimer

This is for learning only. The PDFs cracked here were practice files Networkwalks provided specifically for this lab, with deliberately weak passwords. Don't run any of this against a file or an account you don't own or don't have permission to test.
