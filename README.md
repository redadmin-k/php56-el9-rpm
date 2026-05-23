# php56-el9-rpm

Experimental PHP 5.6 RPM build for AlmaLinux 9.

This repository contains a modified RPM build of PHP 5.6.40 for AlmaLinux 9.
It is based on Remi's PHP RPM packaging, with local modifications to make PHP 5.6 build on EL9.

## Important Notice

This is not an official Remi package.

This is not an official AlmaLinux package.

This package is provided for experimental, compatibility, and legacy migration purposes only.

Use it entirely at your own risk.

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

Do not use this build for new applications.

Do not expose it to the Internet unless you fully understand the security risks.

You are responsible for all security, compatibility, and operational risks.

## Build Requirements

This package was built with:

- AlmaLinux 9
- EPEL 9
- Remi repository for EL9
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
"""
```

## Build

Place the OpenSSL 1.0.2u source archive under `~/rpmbuild/SOURCES/`:

```bash
cd ~/rpmbuild/SOURCES
curl -LO https://www.openssl.org/source/old/1.0.2/openssl-1.0.2u.tar.gz
```

Build the source RPM:

```bash
rpmbuild -bs ~/rpmbuild/SPECS/php.spec
```

Build with mock:

```bash
mock -r alma+epel-9-x86_64 --scrub=all
mock -r alma+epel-9-x86_64 ~/rpmbuild/SRPMS/php-5.6.40-46.el9.src.rpm
```

Built RPMs will be available under:

```text
/var/lib/mock/alma+epel-9-x86_64/result/
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

## License

This repository only contains RPM packaging modifications.

PHP itself is licensed under the PHP License.

OpenSSL is licensed under the OpenSSL license.

The original RPM packaging work is based on Remi's PHP RPM packaging.

## Disclaimer

No warranty is provided.

Use this package at your own risk.

You are fully responsible for testing, security review, deployment, operation, and maintenance.
