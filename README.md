# pacquet libc scalar reproduction

Reproduces a crash in pacquet when `pnpm-lock.yaml` contains a scalar `libc` value.

## Environment

- Node.js: 24.16.0
- pnpm: 11.2.2
- @pnpm/pacquet: 0.2.5

## Steps to reproduce

```sh
pnpm install
```

## Error

https://github.com/kimulaco/pnpm-pacquet-string-libc/actions/runs/26334757175/job/77526512431

```
❯ pnpm i
Lockfile is up to date, resolution step is skipped
▶ Using pacquet for this install
  pacquet is pnpm's Rust install engine (preview); declared in configDependencies.
Error: pacquet_lockfile::parse_yaml
  × initialize the state
  ├─▶ Failed to parse lockfile content as YAML: error: line 317 column 11: unexpected event: expected sequence start
  │      --> <input>:317:11
  │       |
  │   315 |     cpu: [arm64]
  │   316 |     os: [linux]
  │   317 |     libc: glibc
  │       |           ^ unexpected event: expected sequence start
  │   318 |
  │   319 |   sass-embedded-linux-arm@1.100.0:
  │       |
  ├─▶ Failed to parse lockfile content as YAML: error: line 317 column 11: unexpected event: expected sequence start
  │      --> <input>:317:11
  │       |
  │   315 |     cpu: [arm64]
  │   316 |     os: [linux]
  │   317 |     libc: glibc
  │       |           ^ unexpected event: expected sequence start
  │   318 |
  │   319 |   sass-embedded-linux-arm@1.100.0:
  │       |
  ╰─▶ error: line 317 column 11: unexpected event: expected sequence start
         --> <input>:317:11
          |
      315 |     cpu: [arm64]
      316 |     os: [linux]
      317 |     libc: glibc
          |           ^ unexpected event: expected sequence start
      318 |
      319 |   sass-embedded-linux-arm@1.100.0:
          |
[ERR_PNPM_PACQUET_INSTALL_FAILED] pacquet exited with code 1
```

## Root cause

`sass-embedded-linux-*` packages publish `libc` as a string in their `package.json`:

```json
{ "libc": "glibc" }
```

pnpm writes this as a YAML scalar in the lockfile (`libc: glibc`). pacquet's lockfile parser deserializes `libc` as `Vec<String>` and fails on the scalar value.

`PackageMetadata.libc` in `crates/lockfile/src/package_metadata.rs` needs to accept both a scalar string and a sequence.
