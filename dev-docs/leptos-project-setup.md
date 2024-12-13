# Leptos Project Setup

How to create a Laptos CSR project from scratch using Trunk, Tailwind CSS and daisyUI.

## Overview

**Leptos**

_Leptos makes it easy to build applications in the most-loved programming language, combining the best paradigms of modern web development with the power of Rust._

<https://leptos.dev/>

<https://book.leptos.dev/>

<https://github.com/leptos-rs/leptos>

<https://github.com/leptos-rs/start-trunk>

**Trunk**

_Trunk is a WASM web application bundler for Rust. Trunk uses a simple, optional-config pattern for building & bundling WASM, JS snippets & other assets (images, css, scss) via a source HTML file._

<https://trunkrs.dev/>

<https://trunkrs.dev/guide/>

**Tailwind CSS**

_Tailwind CSS is a utility-first CSS framework for rapidly building modern websites without ever leaving your HTML._

<https://tailwindcss.com/>

**daisyUI**

_daisyUI adds component class names to Tailwind CSS so you can make beautiful websites faster than ever._

<https://daisyui.com/>

## Instructions

### 1 - Basic Leptos Project

<https://book.leptos.dev/getting_started/index.html#hello-world-getting-set-up-for-leptos-csr-development>

```sh
# Ubuntu
# sudo apt install --no-install-recommends -y build-essential libssl-dev

cargo install cargo-edit cargo-update cargo-binstall

cargo binstall trunk

rustup target add wasm32-unknown-unknown

cargo init programmable-matter-leptos

cd programmable-matter-leptos

sed -i 's/0\.1\.0/0\.0\.1/' Cargo.toml

cargo add leptos --features=csr

echo '<!DOCTYPE html>
<html>
  <head></head>
  <body></body>
</html>' \
> index.html

echo 'use leptos::prelude::*;

fn main() {
    leptos::mount::mount_to_body(|| view! { <p>"Programmable Matter"</p> })
}' \
> src/main.rs

cargo check

trunk serve

firefox --private-window http://127.0.0.1:8080/
```

### 2 - Developer Experience (DX) Improvements

<https://book.leptos.dev/getting_started/leptos_dx.html>

<https://github.com/bram209/leptosfmt>

```sh
#
# Browser Console Log
#

cargo add log console_log console_error_panic_hook

echo 'use leptos::prelude::*;

fn main() {
    _ = console_log::init_with_level(log::Level::Debug);
    console_error_panic_hook::set_once();

    leptos::mount::mount_to_body(|| view! { <p>"Programmable Matter"</p> })
}' \
> src/main.rs

#
# view macro formatting
#

cargo binstall leptosfmt

echo 'edition = "2021"' > rustfmt.toml

echo '[rustfmt]
overrideCommand = ["leptosfmt", "--stdin", "--rustfmt"]' \
> rust-analyzer.toml

echo 'max_width = 100
tab_spaces = 4
indentation_style = "Spaces"
newline_style = "Unix"
attr_value_brace_style = "WhenRequired"
macro_names = [ "leptos::view", "view" ]
closing_tag_style = "Preserve"

[attr_values]
class = "Tailwind"' \
> leptosfmt.toml

leptosfmt ./**/*.rs
```

### 3 - GitHub Pages Deploy

<https://book.leptos.dev/deployment/csr.html#github-pages>

<https://github.com/diversable/deploy_leptos_csr_to_gh_pages>

<https://github.com/leptos-rs/start-trunk>

```sh
#
# CSR release files
#

echo '
[profile.release]
strip = true
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"' \
>> Cargo.toml

echo '/dist' >> .gitignore

trunk build --release

ls -ldh dist/* | tr -s ' ' | cut -d ' '  -f 5,9
# 835 dist/index.html
# 48K dist/programmable-matter-leptos-363ed68574b02c6b_bg.wasm
# 22K dist/programmable-matter-leptos-363ed68574b02c6b.js


#
# GitHub Action
#
# Go to your repo's settings, and click on "Pages". In the "Build and deployment" section of the page, change the "source" to "Github Actions".

mkdir -p .github/workflows

echo 'name: Publish

on:
    push:
        branches: [main]
    workflow_dispatch:

permissions:
    contents: read
    pages: write
    id-token: write

concurrency:
    group: "pages"
    cancel-in-progress: false

jobs:
    Publish:
        timeout-minutes: 10

        environment:
            name: github-pages
            url: ${{ steps.deployment.outputs.page_url }}

        runs-on: ubuntu-latest

        steps:
            - uses: actions/checkout@v4

            # Install Rust Nightly Toolchain, with Clippy & Rustfmt
            - name: Install Rust
              uses: dtolnay/rust-toolchain@stable
              with:
                  components: clippy, rustfmt

            - name: Setup
              run: |
                  rustup target add wasm32-unknown-unknown
                  curl -L --proto "=https" --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash
                  cargo binstall -y trunk leptosfmt

            - name: Test
              run: |
                  cargo test --all-targets --all-features
                  cargo clippy --all-targets --all-features -- -D warnings
                  cargo fmt --all

            - name: Build
              run: trunk build --release --public-url "/${GITHUB_REPOSITORY#*/}"

            # Deploy with Github Static Pages

            - name: Configure
              uses: actions/configure-pages@v5
              with:
                  enablement: true
                  # token:

            - name: Upload
              uses: actions/upload-pages-artifact@v3
              with:
                  # Upload dist dir
                  path: "./dist"

            - name: Deploy
              id: deployment
              uses: actions/deploy-pages@v4' \
> .github/workflows/gh-pages-deploy.yml

git add .
git commit -m "Basic Leptos Project"

git remote add origin git@github.com:cirocavani/programmable-matter-leptos.git
git push -u origin main

# Action execution
# https://github.com/cirocavani/programmable-matter-leptos/actions

firefox --private-window https://cirocavani.github.io/programmable-matter-leptos/
```
