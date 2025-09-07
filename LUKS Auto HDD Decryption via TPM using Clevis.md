# LUKS - Clevis - TPM
Prerequisites: already LUKS-encrypted root volume.

Choose the PCR IDs based on your needs and replace nvme0n1p3 with your root partition device.

`sha256` is the only hash supported on newer TPMs. For older TPMs, you may need to use `sha1`.

```
apt install clevis-tpm2 clevis-luks clevis-initramfs
sudo clevis luks bind -d /dev/nvme0n1p3 tpm2 '{"pcr_bank":"sha256", "pcr_ids":"0,2"}'
```

Your HDD will now be automatically decrypted during boot.

## Unbind
In case of problems (e.g. TPM PCR checksums changed due to BIOS settings or hardware changes), you need to unbind the TPM and execute the bind again.

```
sudo clevis luks unbind -d /dev/nvme0n1p3 -s 1
```
