# iBrowser platform development

This is a public Safe Exam Browser fork. Read the upstream README and your assigned task/contract before editing. Keep the upstream layout and licence notices.

- Work on a named task branch from the reviewed ibrowser commit in an isolated clone. One owner per writable path; claim the coordinating issue before starting. Never switch another agent’s checkout.
- Preserve .seb formats/settings, seb:// and sebs:// launch protocols, standard X-SafeExamBrowser-RequestHash and X-SafeExamBrowser-ConfigKeyHash, and SafeExamBrowser.security.*.
- iBrowser branding is additive. Native X-iBrowser-RequestHash and X-iBrowser-ConfigKeyHash must reuse the corresponding standard proof and recipient scope. iBrowser.security.* must share the native API, functions and live update lifecycle; never snapshot proof values.
- Never rename upstream namespaces globally, weaken lockdown, TLS/certificate checks, integrity checks, browser sandboxing or safe exit to make a build work.
- Never include private backend source, student/exam data, production keys, credentials, signing material or restricted proprietary binaries. Use only synthetic fixture bundles explicitly supplied for public use; do not copy a private coordinating repository into this fork.
- Inspect all callers/tests/config/exports affected by a change. Use existing build logic and the smallest complete change. Nontrivial logic requires meaningful tests; no invented build recipe or success claim.
- Use native toolchains/runners for platform builds and real devices for lockdown acceptance. Compilation does not establish signing, component rights, security parity or final-build approval.
- Source updates and documentation alone do not authorize production changes, purchases, external messages or student releases. CI signing/cloud credentials must never be available to untrusted PR code.
- Submit a PR to this fork’s ibrowser branch, preserving upstream branches/tags. Do not push to the official upstream. The coordinator updates a parent pin only after review and merge; task branches are not release identity.
- Handoff includes task ID, full commit/PR, changed behavior/paths, exact commands and results, toolchain, source/notice implications, skipped checks, risks and next owner/dependency. Keep diagnostics redacted.

Windows baseline candidate is v3.10.3. Read solution/AppVeyor inputs before selecting Visual Studio/MSBuild/SDK versions; .NET Framework runtime requirements are not a complete build recipe. P03 must reproduce the unmodified baseline before wrappers or branding.
