# rustdesk-yunisky build notes

## Goal
Embed Yunisky private RustDesk server defaults so a freshly installed client comes up preconfigured without manual server entry.

## Embedded defaults
- ID / Rendezvous server: `84.8.135.19`
- Relay server: inferred automatically as `84.8.135.19:21117` by RustDesk when relay is not explicitly set
- API server: `http://84.8.135.19:21114`
- Server public key: `AoPZj0gUgJbSvkzayqo/MkGzST773COTSv2+aLV3Zq8=`

## Why these code changes
RustDesk already supports compile-time embedding of server defaults via `RENDEZVOUS_SERVER`, `RS_PUB_KEY`, and `API_SERVER`. In upstream code, the fallback constants still point to public infrastructure when build-time env vars are absent.

This fork changes the source-level fallback defaults so local builds automatically target the Yunisky infrastructure even if no CI secrets or shell env vars are set.

## Files changed
- `libs/hbb_common/src/config.rs`
  - sets source fallback rendezvous server to `84.8.135.19`
  - sets source fallback public key to the Yunisky server key
  - adds `DEFAULT_API_SERVER` with fallback `http://84.8.135.19:21114`
- `src/common.rs`
  - uses `config::DEFAULT_API_SERVER` instead of only `option_env!("API_SERVER")`

## Behavior
- Fresh install defaults to Yunisky private infrastructure.
- User can still open settings and modify ID / relay / API server manually.
- Relay remains editable; if left blank, RustDesk derives relay from the rendezvous host using its normal port logic.

## Local build attempt on macOS
Recommended command path from upstream:
- `python3 build.py --flutter`

If Flutter desktop toolchain is unavailable, a partial Rust-only verification command is:
- `cargo check --features flutter`

For CI/release builds, environment variables can still override source defaults:
- `RENDEZVOUS_SERVER`
- `RS_PUB_KEY`
- `API_SERVER`
