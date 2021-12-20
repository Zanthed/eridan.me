---
title: "Hexavalent - The Desktop Vanadium"
date: 2021-12-06T00:00:00-00:00
draft: false
summary: "An upcoming desktop Windows, Linux, and macOS internet browser, focused on security and privacy."
---

**Browser security is hard**. This has been proven endlessly; no matter what browser you like, and no matter the operating system you're on. Sprawling codebases of C and C++, making memory management extremely difficult. Rust support is very niche and isn't complete or fully usable. There aren't real privacy-respecting browsers either. Firefox, Chrome, Edge, etc don't have sane defaults. LibreWolf, Ungoogle Chromium, and Brave have very questionable, dubious, or improperly researched patches or "privacy features", some of these browsers severely lacking in security too. Lots of those "privacy features" only make your privacy worse and make you stick out like a sore-thumb.

[Privacy Sandbox](https://privacysandbox.com/) intends to solve a fair chunk of privacy problems with the modern web while retaining the usefulness of the previous features to marketers and other forms of data collection in a less invasive and concerning way; however that's a [gradual process that will take years of research, testing and experimentation, and rolling out. It will be the beginning of 2023 by the time we are in the beginning stages of Privacy Sandbox adoption and transition](https://privacysandbox.com/timeline/).

Why isn't there a proper browser out there for desktop users? On Android if you're running GrapheneOS, Vanadium is a very security-focused and privacy-aware hardened Chromium fork, while sharing some privacy patches from [Bromite](https://github.com/bromite/bromite) that are carefully researched and have more saner defaults. Bromite has great fingerprinting protections, other privacy patches, and includes some security patches from Vanadium. Some good upstream security features such as [V8 Ubercage](https://twitter.com/5aelo/status/1450862485070827533) are highly experimental and cannot be yet mass-adopted due to performance regressions, crashes, and resource usage.

**[Hexavalent](https://hexavalent.org/) intends to change this**. Hexavalent is an upcoming hardened, security-focused and privacy-aware Chromium browser with similar to GrapheneOS's Vanadium browser, led by _[duck (qua3k)](https://github.com/qua3k)_ and _[flawedworld](https://github.com/flawedworld)_.

Hexavalent currently and will feature security hardening such as hardening Chromium's [PartitionAlloc](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/base/allocator/partition_allocator/PartitionAlloc.md) referencing GrapheneOS's [hardened_malloc](https://github.com/GrapheneOS/hardened_malloc), tightening Linux seccomp policies, enable usage of hardware-assisted shadow stacks like CET, forbid lots of dynamically generated code in as much areas as possible (e.g. renderer, browser, network, audio processes and more), implement a high-performance CSPRNG using ChaCha8 in places that require a CSPRNG and even replacing usages of an insecure RNG while still retaining performance, hardening [V8 engine](https://v8.dev/), using Control Flow Guard (ideally XFG in the future) for CFI, tightening the Chromium sandbox, and much more.

Hexavalent will also feature some privacy features with the goal of blending in with the Chrome and Edge crowd, similar to what Vanadium does. For example, Client Hints will be Chrome's, Hexavalent will not leak .onion domains to resolvers making it usable for Tor, do not send X-Client-Data, replace Google Geolocation APIs with a local GeoIP database, use a local database for SafeBrowsing instead of fetching from Google using APIs, using proper adblocking and content filtering with standard Chromium functionality like Bromite, freeze User Agent, HTTP Cache Partitioning, mark insecure (HTTP) origins as dangerous instead of insecure including mixed content and forms, disable hyperlink auditing, and more. Some critical things such as component updates will still be fetched by Google until they're mirrored by Hexavalent or possibly from another trusted source like Vanadium.

Hexavalent's goal is to take existing security measures, enhance, and improve upon them instead of re-inventing the wheel, enumerating badness, or turning exploit mitigations into attack surface themselves. No massive lists of bad things, no blocking things, no unnecessary ideologies, just meaningful security and privacy. Upstream contributions will be made too!

**Check out the repository here: https://github.com/Hexavalent-Browser/Hexavalent**

Check out qua3k's blog on Hexavalent: https://qua3k.github.io/hexavalent/

## Places to keep up with development and news:

- Matrix: [#hexavalent:matrix.org](https://app.element.io/#/room/%23hexavalent:matrix.org)
- Telegram: [t.me/Hexavalent](https://t.me/Hexavalent)
- Twitter: [@HexavalentWeb](https://twitter.com/HexavalentWeb)
