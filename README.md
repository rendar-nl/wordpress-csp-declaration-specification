# WordPress CSP Declaration Specification

**A proposal for declaring, compiling, and auditing Content Security Policy requirements in WordPress.**

This repository contains a draft specification for a WordPress-oriented CSP declaration system.

The goal is to define a common contract that can help WordPress projects:
- declare CSP-relevant behavior explicitly,
- compile a truthful policy per response context,
- support stricter and more maintainable CSP rollout,
- and improve tooling, review, and auditability.

## Why This Exists

WordPress has strong asset registration and rendering primitives, but it does not offer a common CSP declaration model.

As a result:
- CSP ownership is fragmented,
- plugins and themes can introduce hidden dependencies,
- inline behavior is difficult to reason about,
- and strict CSP rollout becomes harder than it should be.

This specification is intended as a proposal for a better WordPress ecosystem contract around CSP.

## What This Is

- A specification draft
- A declaration and compilation model proposal
- A WordPress-oriented design for stricter CSP support
- A document intended for discussion and review

## What This Is Not

- A finished WordPress standard
- An implementation or enforcement plugin
- A guarantee of WordPress core adoption
- A complete rollout guide for any specific project

## Current Structure

```text
spec/
  wordpress-csp-declaration-specification-0.2.md
```

## Current Focus

The current draft is centered on:
- hooks plus a registry as the preferred WordPress-oriented declaration surface,
- one compiled CSP per response context,
- optional `declared_by` metadata for audit and debugging,
- and a baseline aligned with current CSP best practices.

## WordPress CSP Context

Some existing WordPress and web security discussions that help frame this proposal:

- [WordPress Trac #39941: Add integrity and crossorigin to enqueued scripts](https://core.trac.wordpress.org/ticket/39941)
- [WordPress Trac #51407: Introduce support for Content Security Policy nonce](https://core.trac.wordpress.org/ticket/51407)
- [WordPress Trac #59446: Use `wp_get_inline_script_tag()` where possible to improve CSP compatibility](https://core.trac.wordpress.org/ticket/59446)
- [WordPress and CSP discussion on Stack Overflow](https://stackoverflow.com/questions/52553571/wordpress-and-csp)
- [Reddit discussion: Writing solid CSP headers for WordPress](https://www.reddit.com/r/Wordpress/comments/127q9ev/writing_solid_content_security_policy_csp_headers/)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [MDN Content Security Policy Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
- [web.dev: Mitigate XSS with a strict CSP](https://web.dev/articles/strict-csp)

## Feedback Wanted

Feedback is especially useful on:
- whether hooks + registry is the right center for WordPress,
- whether context handling is scoped correctly for v1,
- whether the declaration concepts are minimal but sufficient,
- and how closely this should align with current WordPress asset registration patterns.

## Status

- **Specification:** Draft v0.2
- **Stability:** Early review
- **Feedback:** Actively requested

## License

This specification is open and may be freely implemented, extended, or referenced.
