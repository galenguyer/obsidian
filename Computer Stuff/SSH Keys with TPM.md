---
tags: security
---

# SSH Keys with TPM
This guide is for a Thinkpad T470 running Arch Linux

## Hardware Support
First, ensure your TPM is recognized by the kernel.
```bash
root@avalon:~# dmesg | grep tpm
[    0.879747] tpm_tis MSFT0101:00: 2.0 TPM (device-id 0x0, rev-id 78)
```

## TPM Software
Install the [`tpm2-tools`](https://archlinux.org/packages/community/x86_64/tpm2-tools/) and [`tpm2-abrmd`](https://archlinux.org/packages/community/x86_64/tpm2-abrmd/) packages. `tpm2-tools` is a bundle of tools for interacting with the TPM based on `tpm2-tss`, and `tpm2-abrmd` is the Trusted Platform Module 2.0 Access Broker and Resource Management Daemon. We'll also need [`tpm2-pkcs11`](https://archlinux.org/packages/community/x86_64/tpm2-pkcs11/) for talking to the TPM like it's a PKCS#11 token.

## Generating a key
Set the following environment variable in your shell. 
```bash
export TPM2_PKCS11_TCTI=tabrmd:
```
Without it, `tpm2-pkcs11` will attempt to talk directly to the TPM device and only fall back to the access broker daemon if it fails. 

Create a directory for the keys. The SSH key will not actually be stored on the TPM but in the directory we create. It's important to not lose the contents of this folder else you'll lose your key, just like you would with a normal SSH key. However, the key is encrypted in such a way only the TPM can decrypt it.
```bash
chef@avalon:~$ mkdir ~/.tpm2_pkcs11
chef@avalon:~$ tpm2_ptool init
action: Created
id: 1
```

Create a PKCS#11 token.
```bash
chef@avalon:~$ tpm2_ptool addtoken --pid=[POOL_ID] --label=[LABEL] --sopin=[ADMIN_PIN] --userpin=[USER_PIN]
```

POOL_ID should be the id returned by the init command you just ran. The LABEL, ADMIN_PIN and USER_PIN values are chosen by you. LABEL must be unique across all tokens in the pool.

Now create the key. 
```bash
chef@avalon:~$ tpm2_ptool addkey --label=[LABEL] --userpin=[USER_PIN] --algorithm=ecc256
```

## Using the key
Derive the public key with the following command:
```bash
chef@avalon:~$ ssh-keygen -D /usr/lib/pkcs11/libtpm2_pkcs11.so
```

You can specify the key for an SSH session using the command line:
```bash
ssh -I /usr/lib/pkcs11/libtpm2_pkcs11.so example.com
```
You can also use the following lines in your `~/.ssh/config` file:
```txt
PKCS11Provider /usr/lib/pkcs11/libtpm2_pkcs11.so
IdentityAgent none
```