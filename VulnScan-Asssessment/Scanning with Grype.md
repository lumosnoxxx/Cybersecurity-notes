# Vulnerability Scanning with Grype

[Grype](https://github.com/anchore/grype) is an open-source tool that checks software components for known vulnerabilities.

It supports directories, container images, and SBOMs, including components from Linux, Go, Python, Java, and JavaScript ecosystems.

## 1. Install on Ubuntu / Debian

```bash
sudo apt update
sudo apt install curl
mkdir -p ~/bin
curl -sSfL https://get.anchore.io/grype -o ~/install-grype.sh
sh ~/install-grype.sh -b ~/bin
~/bin/grype version
```

The `-b` option specifies where to install Grype.

## 2. Scan a Directory

Replace `my-folder` with the directory you want to analyze:

```bash
~/bin/grype dir:./my-folder -o table
```

To display and save the results:

```bash
set -o pipefail
~/bin/grype dir:./my-folder -o table 2>&1 | tee grype-results.txt
```

Grype updates its vulnerability database, identifies detectable components, and checks their versions. It does not execute the analyzed program or apply fixes.

## 3. Scan the Contents of an RPM

Place `package.rpm` in your current directory, then extract and scan its contents:

```bash
sudo apt install rpm2cpio cpio
mkdir rpm-analysis
cd rpm-analysis
rpm2cpio ../package.rpm | cpio -idmv --no-absolute-filenames
~/bin/grype dir:. -o table 2>&1 | tee ../grype-results.txt
```

Run the extraction without `sudo`, in a test VM. This extracts the files without installing the software.

## 4. Read the Results

| Column | Meaning |
|---|---|
| `NAME` | Identified component |
| `INSTALLED` | Detected version |
| `FIXED IN` | Reported fixed version, when available |
| `VULNERABILITY` | Vulnerability identifier |
| `SEVERITY` | Reported severity |

The table shows vulnerability matches, not every detected component.

A finding does not prove exploitability in your environment. No findings does not guarantee security: check scan coverage and warnings.

After updating the software, scan again and compare results. Keep the file hash, scanner version, and scan output for traceability.

## Optional: Pipeline Error Handling

In Bash scripts, enable:

```bash
set -o pipefail
```

This makes a pipeline report failure when an earlier command fails, even if `tee` succeeds. It does not change vulnerability detection or make Grype fail automatically when findings exist.

## Reference

- [Official Grype documentation](https://github.com/anchore/grype)
