+++
title = "A look at (r)age for modern encryption"
date = 2022-05-06T00:00:00+00:00
description = "A dive into age, the tool offering simple, modern, and secure encryption. How it works, tooling, and how Rust developers can commit secrets publicly, safely, and confidently."

[taxonomies]
categories = ["Rust"]
tags = ["rust", "cryptography"]
+++

Modern encryption has a lot of options for data - AES, RSA, PGP, etc. But what expectations are there for developers, who otherwise do not have a background in cyber security, to understand about cryptography? Well, aside from GitHub's requirement of SSH keys, probably little.

This blog will be about *age*, a new file encryption tool which is stupidly simple and delivers fearless, modern encryption. Age is ready today, and is starting to see acceptance fast[^acceptance].

# What is age?
The author of age uses this description:
> A simple, modern and secure encryption tool with small explicit keys, no config options, and UNIX-style composability.
> — <cite>Filippo Valsorda[^age]</cite>

How does age measure up to this definition? Well, age uses a native backend of X25519 asymmetric encryption which is fast, secure, and particularly resistant to timing attacks.

<details>
    <summary>X25519? That sounds familiar...</summary>
    <p>You might notice a slight resemblance with Ed25519, universally used for signage (by GitHub) and SSH keys. The security relies on the same curve for cryptography, Curve25519, however Ed25519 is a signature algorithm, which is why it is used as an SSH key.</p>
</details>

Age emits terse 63 character public keys with support for multiple pluggable recipients and passphrases. This means multiple keys can decrypt a file. Age also provides excellent interoperability with existing standards, such as supporting SSH public keys as recipient types. It does this by injecting a header on the encrypted file with recipient keys and types[^age_format]. These tags are, however, embedded in the file. Thankfully, the native X25519 backend is able to obfuscate this information. However with SSH keys, it is possible to track files encrypted to a specific public key. I advise using SSH keys only as a convenience.

# Age in Rust
[![crates.io](https://img.shields.io/crates/v/age?style=flat-square)](https://crates.io/crates/age)
[![docs.rs](https://img.shields.io/docsrs/age/latest?style=flat-square)](https://docs.rs/age/latest/age)

Age was originally a Go library, but it was quickly ported to Rust as an interoperable equivalent, called *rage*. In other words, the binary executable rage can replace the original executable simply with `cargo install rage` and `alias age=rage` ([other platforms](https://github.com/str4d/rage#installation)).

Developers in Rust wanting to work with age are provided the [*age*](https://crates.io/crates/age) library crate, which nice features like async support. Those interested in WASM support can look at the WASM example, [*wage*](https://github.com/str4d/wage).

The examples on docs are pretty easy to follow.

```rust
let key = age::x25519::Identity::generate();
let pubkey = key.to_public();

let plaintext = b"Hello world!";

// Encrypt the plaintext to a ciphertext...
let encrypted = {
    let encryptor = age::Encryptor::with_recipients(vec![Box::new(pubkey)]);

    let mut encrypted = vec![];
    let mut writer = encryptor.wrap_output(&mut encrypted)?;
    writer.write_all(plaintext)?;
    writer.finish()?;

    encrypted
};
```

## SOPS
There's also some great tooling for preserving the original file structure. Mozilla's [Secret OPerationS (*SOPS*)](https://github.com/mozilla/sops) product encrypts the values of certain files, but not the keys. SOPS supports favorite formats like YAML, JSON, ENV, and INI but also works with BINARY formats.

To get the point across, here is what an age encrypted file looks like:
```text
age-encryption.org/v1                                   <-- Age version
-> X25519 3tu9gP/qdJ+RjTFe0HS0M9ROyYrKice+V+6p6Ugr1mY   <-- Recipient Stanza
9UE+yki68cqCc7GWDJ2sQxPeQZtg+eh3g9HhL7a7qAY
--- YG8EY5Re2TWV1Y1L66bd2i1zv68lJn4sjqRKpcC7N9Q         <-- Payload
```

With the magic of SOPS (`sops -e -i my.yaml`), a YAML file can turn still maintain its key structure, while encrypting the values.

<table style='table-layout: fixed;'>
<tr>
<th style='width: 50%;'> Plain-text </th>
<th style='width: 50%;'> SOPS + age </th>
</tr>
<tr>
<td>

```yaml
key1: value1
key2:
- value2
- value3
...
```

</td>
<td>

```yaml
key1: ENC[AES256_GCM,data:gZm+cIsS,iv:k2M+w7H2B3kfiBgIm5y9pW5UupxwUJilkdJpW+GJ9S0=,tag:R9gD6jHmsRuNtp2LwhgyVA==,type:str]
key2:
    - ENC[AES256_GCM,data:+bjyJkk6,iv:2d1mwd0zJqmgBE4fAUj+OrsxnUVLU1nQVSkx/h7xT/I=,tag:YhjhkyfDDvyrldxUxPM3vQ==,type:str]
    - ENC[AES256_GCM,data:umIMgauY,iv:1Y+ZR8mrzxx3l1tDVwtLHa0skhVyasxhmXeZLPZPfcs=,tag:DlL9rNrVyBiWIHXfypAtXw==,type:str]
...
```

</td>
</tr>
</table>

This is a gain for open-source initiatives, because config files and private configurations can be shared publicly with safety and better direction. For example, [my home kubernetes configuration](https://github.com/simbleau/home-ops) matter is completely public and open source, with [secrets encrypted with SOPS](https://github.com/simbleau/home-ops/tree/main/vpn). I also recommend [pre-commit](https://pre-commit.com/) with a hook to [forbid secrets](https://github.com/zricethezav/gitleaks) accidentally being committed.

## SOPS + Visual Studio (VS) Code
SOPS has the benefit of integrating your favorite `$EDITOR` (default is `vim`), but many, including Rustaceans, use *VSCode*. Thankfully, there's a plugin to support realtime editing of SOPS encrypted files: [signageos.signageos-vscode-sops](https://marketplace.visualstudio.com/items?itemName=signageos.signageos-vscode-sops). You'll need to create a `.sops.yaml` file in your repository base which declares the regex pattern of your SOPS files ([example](https://github.com/simbleau/home-ops/blob/main/.sops.yml)).

A toy example to match files with `.sops.` anywhere in the filename, e.g. `example.sops.yml`
```yaml
---
# Configuration for VSCode Extension: `signageos.signageos-vscode-sops`
# see also:
#   SOPS: https://github.com/mozilla/sops#readme

creation_rules:
  - path_regex: .*\.sops\..*
    age: >-
      age1vyn7adkeklaztjhgcuh3sflwzlk6mcuy52ja0wr5kn3ghf5u8e2sa2esn4
```

I recommend if you go this route, you also add a `$REPO/.vscode/extensions.json` file which recommends upstream VSCode users to download the extension ([example](https://github.com/simbleau/home-ops/blob/main/.vscode/extensions.json)).

```json
{
    "recommendations": [
        "signageos.signageos-vscode-sops",
    ]
}
```

---
# Exercise
Write me a message using [my public keys](https://github.com/simbleau.keys) as recipients of a message, and paste your message in the merge request for this blog. Only I will be able to decrypt it, and for the first three individuals, I will respond with a payload only you can decrypt, containing an access code to a technical community of likeminded individuals, assuming you want to join.

<details>
    <summary>Hint 1</summary>

    curl https://github.com/simbleau.keys | age -R
</details>

<details>
    <summary>Hint 2</summary>
    <p>You will need armor to post an age contents on GitHub.</p>
</details>

*Farvel! Until next time.*

---
<!-- Note: There must be a blank line between every two lines of the footnote difinition.  -->
[^acceptance]: Age was vetted and admited in the officially curated [Ubuntu](https://packages.ubuntu.com/impish/age) and [Debian](https://packages.debian.org/bullseye/age) repositories, among others.

[^age]: Age, [described on GitHub](https://github.com/FiloSottile/age)

[^age_format]: The age file [encrypted file format](https://github.com/C2SP/C2SP/blob/main/age.md#encrypted-file-format) decomposition specifics and RFCs