<div align="center">
    <h1><code>anysc-rustls</code></h1>
    <p>crate-level unification layer for any <code>{async crate}-rustls</code></p>
</div>

<br/>

## Usage

1. Add to `dependencies`:
  
  ```toml
  [dependencies]
  # ...
  anysc-rustls = { version = "0.1.0", features = ["(Go to 2.)"] }
  ```

2. **Select one of `io_*` feature flags** based on your case:
  
  - `io_tokio` ( `tokio::io` interface, used by:
    [`tokio`](https://github.com/tokio-rs/tokio),
    [`nio`](https://github.com/nurmohammed840/nio),
    etc. )
  - `io_futures` ( `futures-io` interface, used by:
    [`async-std`](https://github.com/async-rs/async-std),
    [`smol`](https://github.com/smol-rs/smol),
    [`glommio`](https://github.com/DataDog/glommio),
    etc. )

3. Depending on the use case, activate the inheritied features:
  
  - `aws-lc-rs`
  - `aws_lc_rs`
  - `early-data`
  - `fips`
  - `logging`
  - `ring`
  - `tls12`

4. Write your code with `anysc-rustls` as with `{tokio, futures}-rustls`.

<br/>

## What this does?

**Just reexporting** one of

- [`tokio-rustls`](https://github.com/rustls/tokio-rustls)
- [`futures-rustls`](https://github.com/rustls/futures-rustls)

based on the `io_*` feature flag selected.

The point is that **this is a crate**: it enables,
for some (maybe niche) crates that

* support multiple async runtimes over **different async IO interfaces** (`tokio::io`, `futures-io`)
* AND **optionally** provide [rustls](`https://github.com/rustls/rustls`)-powered TLS functionality
  behind a **feature flag** (like `tls`)

to switch `{async crate}-rustls` dependencies **without any needless dependencies**.

### Problem

That's impossible by other way: if simply having a submodule reexporting
`tokio-rustls` and `futures-rustls` conditionally with respective feature flags (like `tokio-io`, `futures-io`),
indeed it works, but the crate's `[dependencies]` will be like

```toml
[dependencies]
tokio-rustls = { optional = true, version = ... }
futures-rustls = { optional = true, version = ... }

[features]
tokio-io = ...
futures-io = ...
tls = ...
```

Here, how we setup the features?

1. `tokio-io = if "tls" ["dep:tokio-rustls"]` and `futures-io = if "tls" ["dep:futures-tls"]`

  impossible.

2. `tls = if "tokio-io" ["dep:tokio-rustls"] else if "futures-io" ["dep:futures-tls"]`

  impossible.

3. `tls = ["dep:tokio-rustls", "dep:futures-rustls"]`

  works, but one of them must be needless.

So it's **impossible to avoid undesired dependencies** in this way.

### Solution

However, it's enabled by a **crate-level unification layer** as this crate:

```toml
[dependencies]
anysc-rustls = { optional = true, version = "0.1.0" }

[features]
tls = ["dep:anysc-rustls"]
tokio-io = ["anysc-rustls?/io_tokio", ...]
futures-io = ["anysc-rustls?/io_futures", ...]
```

Yes, that's done by the `<crate>?/<feature>` syntax!
