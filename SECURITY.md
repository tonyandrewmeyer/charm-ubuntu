# Security policy

## Product architecture

charm-ubuntu is a packaging-layer charm that deploys a pristine Ubuntu
Server image and applies a small amount of configuration (hostname, mounted
storage). It runs no daemons of its own and exposes no network surface
beyond what the base Ubuntu image and the operator's deployment add. The
trust boundaries it sits between are:

- **Juju ↔ charm code:** the charm receives configuration and lifecycle
  events from Juju over the standard Juju agent channel; no network
  listener is opened by charm-ubuntu itself.
- **Charm ↔ Ubuntu base:** the charm applies hostname and storage
  configuration to the underlying Ubuntu base, which is the security
  boundary for everything else (kernel, package set, default services).
- **Ubuntu base ↔ network:** any network-exposed services are part of the
  Ubuntu base or layered on by the operator, not by this charm.

## Secure by Design

charm-ubuntu is intentionally minimal:

- No cryptography is implemented or configured by the charm.
- No long-running processes are started by the charm.
- No secrets are generated, stored, or handled by the charm; Juju secrets
  are not used.
- The charm relies on the Ubuntu base's security posture (apt/snapd update
  channels, AppArmor profiles, unattended-upgrades when enabled) rather
  than re-implementing any of it.

Operational risk reduction is therefore concentrated in the Ubuntu base
selection: deploy with the most recent supported LTS where possible.

## Cryptography

charm-ubuntu does not implement, configure, or expose any cryptographic
technology. All cryptographic functionality — TLS, package signing,
storage encryption, kernel-level crypto — is provided by the Ubuntu base
and the [Ubuntu cryptographic
packages](https://ubuntu.com/security/certifications/docs/cryptographic-packages),
which the charm does not override.

There is no encryption at rest applied by the charm. Any disk encryption
must be configured by the operator at the Ubuntu base level (LUKS at
install time, or filesystem-level encryption on the storage volumes the
charm attaches).

## Hardening guidelines

Hardening is delegated to the Ubuntu base. Recommended references:

- [Ubuntu server security
  documentation](https://documentation.ubuntu.com/server/explanation/intro-to/security/)
  — kernel hardening, AppArmor, auditd, ufw.
- [CIS benchmarks for
  Ubuntu](https://ubuntu.com/security/certifications/docs/disa-stig-cis) —
  CIS Level 1 / Level 2 hardening; Ubuntu Pro provides
  `ubuntu-security-guide` for automated application.
- [Juju
  hardening](https://documentation.ubuntu.com/juju/3.6/howto/manage-controllers/#harden-a-controller)
  — controller-side hardening.

charm-ubuntu itself has no additional hardening surface; once the base is
hardened, the charm adds nothing further to harden.

## Logging and monitoring

charm-ubuntu emits only standard charm logs via `juju-log` (visible with
`juju debug-log`). It does not implement OWASP Application Logging
Vocabulary events under SEC0045 — there is no authentication, authorization,
admin, or user-creation surface in the charm to instrument. Operational
monitoring of the underlying Ubuntu base (system logs, journal, audit) is
the operator's responsibility and uses the standard Ubuntu mechanisms.

## Secure decommissioning

To remove a charm-ubuntu deployment:

1. `juju remove-application ubuntu` — Juju tears down the unit; the
   underlying machine, its storage volumes, and any attached block devices
   are released to the cloud provider.
2. Confirm any attached filesystem-storage volumes that contained
   sensitive data are wiped or destroyed at the cloud-provider level —
   `juju remove-application` releases the volume but does not securely
   erase data already written to it.
3. The Ubuntu base image itself contains no persistent charm state once
   the unit is removed.

## Security lifecycle

The security support lifetime for a charm-ubuntu deployment mirrors the
[Ubuntu release lifecycle](https://ubuntu.com/about/release-cycle) of the
base being deployed. LTS releases receive standard security maintenance
for 5 years; interim releases receive security maintenance for 9 months.
Once a given Ubuntu base reaches end-of-life, that base is no longer
security-maintained in this project.

Security updates are delivered through:

- **Ubuntu base updates** — `apt` from the Ubuntu archive on each unit;
  `unattended-upgrades` (if enabled by the operator) applies them
  automatically. This is the channel for kernel, library, and system
  package CVEs.
- **Charm refreshes** — published to
  [Charmhub](https://charmhub.io/ubuntu); operators receive updates by
  running `juju refresh ubuntu`. Charm-level security fixes (e.g. to the
  charm code itself) are delivered this way.

To verify the deployed charm revision, run `juju status` and check the
`Charm rev` column for the `ubuntu` application against the revision
listed on [Charmhub](https://charmhub.io/ubuntu).

## Reporting a vulnerability

Please provide a description of the issue, the steps you took to
create the issue, affected versions, and, if known, mitigations for
the issue.

The easiest way to report a security issue is through
[GitHub's security advisory for this project](https://github.com/canonical/charm-ubuntu/security/advisories/new). See
[Privately reporting a security
vulnerability](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing/privately-reporting-a-security-vulnerability)
for instructions on reporting using GitHub's security advisory feature.

The charm-ubuntu GitHub admins will be notified of the issue and will work with you
to determine whether the issue qualifies as a security issue and, if so, in
which component. We will then figure out a fix, get a CVE
assigned, and coordinate the release of the fix.

You may also send email to security@ubuntu.com. If you want to encrypt your email,
follow [Canonical's reporting instructions](https://ubuntu.com/security/disclosure-policy#contact-us).

If you have a deadline for public disclosure, please let us know.
Our vulnerability management team intends to respond within 3 working
days of your report. This project aims to resolve all vulnerabilities
within 90 days.

The [Ubuntu Security disclosure and embargo
policy](https://ubuntu.com/security/disclosure-policy) contains more
information about what you can expect when you contact us, and what we
expect from you.
