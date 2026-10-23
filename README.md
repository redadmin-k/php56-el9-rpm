# php56-el9-rpm

Experimental PHP 5.6 RPM build for AlmaLinux 9.

This repository contains a modified RPM build of PHP 5.6.40 for AlmaLinux 9.
It is based on Remi's PHP RPM packaging, with local modifications to make PHP 5.6 build on EL9.

## Important Notice

This is not an official Remi package.

This is not an official AlmaLinux package.

This package is provided for experimental, compatibility, and legacy migration purposes only.

Use it entirely at your own risk.

## Security Backport: php2shell (5.6.40-47)

Release **5.6.40-47.el9** integrates a **community-provided php2shell backport** into the Remi-based PHP 5.6.40 RPM spec file.

- Targeted vulnerabilities: **CVE-2026-109618** and **CVE-2026-109619**.
- Patch file: `php-5.6-enhanced-remicollet.patch`.
- Patch source: [Anton-Barinov/php2shell-patches](https://github.com/Anton-Barinov/php2shell-patches).
- RPM spec entry: `Patch280` (applied in `%prep` after `Patch279`).
- RPM release changed from `5.6.40-46` to `5.6.40-47`.

The patch was applied during a Mock build with `--fuzz=0`, and the build of `php-5.6.40-47.el9.src.rpm` completed successfully on AlmaLinux 9.

**Validation status:** Successful patch application and RPM compilation do **not** demonstrate that both vulnerabilities are fully fixed. Independent exploit/regression testing has not been completed. The PHP test run reported failures, including OpenSSL certificate verification tests and some date/language tests; those failures have not been attributed to this patch. Do not treat this build as a fully security-validated release.

To retrieve the community patch for an SRPM build, place it in `~/rpmbuild/SOURCES/`:

```bash
curl -fL \
  https://raw.githubusercontent.com/Anton-Barinov/php2shell-patches/main/patches/php-5.6-enhanced-remicollet.patch \
  -o ~/rpmbuild/SOURCES/php-5.6-enhanced-remicollet.patch
```

The spec file must contain the corresponding patch declaration and application:

```spec
Patch280: php-5.6-enhanced-remicollet.patch

# In %prep, after Patch279:
%patch -P280 -p1 -b .php2shell
```

## OpenSSL

This build uses a private OpenSSL 1.0.2u build for PHP's OpenSSL extension.

The system OpenSSL provided by AlmaLinux 9 is not used for the PHP OpenSSL extension, because PHP 5.6 is not compatible with OpenSSL 3 without additional workarounds.

Verified example:

```bash
php -n -r 'echo OPENSSL_VERSION_TEXT, PHP_EOL;'
```

Expected output:

```text
OpenSSL 1.0.2u  20 Dec 2019
```

## Warning

PHP 5.6 is obsolete and no longer supported by upstream PHP.

OpenSSL 1.0.2u is also obsolete.

The community php2shell patch does not address all other vulnerabilities in these obsolete components.

Do not use this build for new applications.

Do not expose it to the Internet unless you fully understand the security risks.

You are responsible for all security, compatibility, and operational risks.

## Build Requirements

This package was built with:

- AlmaLinux 9
- EPEL 9
- Remi repositories for EL9 (including the Safe repository for dependencies)
- mock
- private OpenSSL 1.0.2u

## mock Configuration Example

Example mock configuration:

```python
include('almalinux-9-x86_64.cfg')
include('templates/epel-9.tpl')

config_opts['root'] = "alma+epel-9-{{ target_arch }}"
config_opts['description'] = 'AlmaLinux 9 + EPEL + Remi'

config_opts['yum.conf'] += """

[remi]
name=Remi EL9
baseurl=https://rpms.remirepo.net/enterprise/9/remi/$basearch/
enabled=1
gpgcheck=0

[remi-safe]
name=Remi Safe EL9
baseurl=https://rpms.remirepo.net/enterprise/9/safe/$basearch/
enabled=1
gpgcheck=0
"""
```

This example disables RPM signature verification for the additional repositories; enable GPG checking and configure the appropriate trusted keys for a production build pipeline.

## Build

Place the OpenSSL 1.0.2u source archive under `~/rpmbuild/SOURCES/`:

```bash
cd ~/rpmbuild/SOURCES
curl -LO https://www.openssl.org/source/old/1.0.2/openssl-1.0.2u.tar.gz
```

Also place `php-5.6-enhanced-remicollet.patch` under `~/rpmbuild/SOURCES/` as described above.

Build the source RPM:

```bash
rpmbuild -bs ~/rpmbuild/SPECS/php.spec
```

Build with mock using the example configuration above:

```bash
mock -r alma+epel-9-x86_64 --scrub=all
mock -r alma+epel-9-x86_64 --rebuild ~/rpmbuild/SRPMS/php-5.6.40-47.el9.src.rpm
```

Alternatively, when Remi repositories are not defined in the Mock configuration, supply them explicitly:

```bash
mock -r alma+epel-9-x86_64 \
  --addrepo=https://rpms.remirepo.net/enterprise/9/remi/x86_64/ \
  --addrepo=https://rpms.remirepo.net/enterprise/9/safe/x86_64/ \
  --rebuild ~/rpmbuild/SRPMS/php-5.6.40-47.el9.src.rpm
```

Built RPMs and logs will be available under:

```text
/var/lib/mock/alma+epel-9-x86_64/result/
```

To inspect patch application in the build log:

```bash
grep -E 'Patch #280|php2shell|php-5.6-enhanced-remicollet' \
  /var/lib/mock/alma+epel-9-x86_64/result/build.log
```

## Install

Install the generated RPMs locally:

```bash
dnf install ./php-*.rpm
```

For a minimal CLI test:

```bash
php -n -v
php -n -m | grep openssl
php -n -r 'echo OPENSSL_VERSION_TEXT, PHP_EOL;'
```

## Verification

You can check the direct shared library dependencies with:

```bash
readelf -d /usr/bin/php | grep -E 'RPATH|RUNPATH|NEEDED'
```

The PHP binary itself should not directly require `libcrypto.so.3`.

If `ldd` shows `libcrypto.so.3`, it may be pulled indirectly by other system libraries such as Kerberos-related libraries.

For a security release, additionally run focused tests for the php2shell vulnerabilities and compare the regression results with the previous `5.6.40-46` build.

## License

This repository only contains RPM packaging modifications.

PHP itself is licensed under the PHP License.

OpenSSL is licensed under the OpenSSL license.

The original RPM packaging work is based on Remi's PHP RPM packaging.

The community php2shell patch is provided by its respective author; review its upstream license and terms separately.

## Disclaimer

No warranty is provided.

Use this package at your own risk.

You are fully responsible for testing, security review, deployment, operation, and maintenance.
