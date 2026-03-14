# WordPress CSP Declaration Specification

**Version:** 0.2
**Status:** Draft
**Maintained by:** Rendar
**Scope:** WordPress plugins and themes

## 1. Purpose

The WordPress CSP Declaration Specification proposes a standardized WordPress system for declaring, compiling, and auditing Content Security Policy requirements.

The specification:

* describes a common CSP declaration model for WordPress,
* enables tooling, audits, and AI agents to understand CSP requirements,
* improves least-privilege policy design,
* makes it possible to generate a more truthful final CSP,
* and serves as a proposal for a better WordPress ecosystem contract around CSP.

## 2. Design Principles

1. **Declarative, not imperative**
   The specification describes *what must be represented in the CSP system*.

2. **Truthful, not permissive**
   The resulting policy should reflect actual requirements, not broad convenience allowlists.

3. **Context first**
   CSP requirements may differ across frontend, admin, login, REST, editor, and embed contexts.

4. **Single compiled policy per context**
   One final policy should be compiled per response context rather than being fragmented across unrelated emitters.

5. **Source responsibility**
   The code that introduces a resource dependency is responsible for making that dependency visible to the CSP system.

6. **Optional and additive**
   Declaration sources may partially declare their needs. A policy compiler may merge multiple declarations.

7. **CSP best practices first**
   The desired baseline is current CSP best practices, with the narrowest truthful policy per context.

8. **Tool-friendly**
   The model should be readable by humans, statically analyzable, and usable by automated tooling.

## 3. Problem Statement

WordPress currently has strong asset-loading and rendering primitives, but no unified CSP declaration model.

As a result:
- CSP ownership is fragmented,
- plugins and themes can introduce hidden dependencies,
- inline behavior is difficult to reason about,
- and strict CSP rollout becomes unnecessarily difficult.

This proposal defines the contract that a WordPress-native CSP system should expose.

## 4. Scope

The proposal covers any WordPress behavior that affects the final CSP.

This includes, but is not limited to:
- external scripts,
- external styles,
- external frames and embeds,
- remote API calls,
- fonts, images, media, and workers,
- inline script blocks,
- inline style blocks,
- style attributes,
- event-handler attributes,
- and CSP-relevant behavior introduced by legacy or third-party code.

## 5. System Model

The proposed system has five conceptual parts:

### 5.1 Declaration Sources
Declaration sources are the places where CSP-relevant behavior originates.

Examples:
- plugins,
- themes,
- blocks,
- editor integrations,
- site configuration,
- and runtime registration code.

### 5.2 Declaration Surface
The declaration surface is the mechanism through which CSP-relevant behavior becomes visible to the system.

The recommended WordPress-oriented model is:
- hooks for registration,
- plus a central registry as the canonical in-memory declaration surface.

Generated artifacts, metadata attached to asset registration, and other approaches may still feed that registry, but the registry is the clearer integration point for WordPress code.

### 5.3 Policy Compiler
The compiler resolves all declarations into one final policy per response context.

The compiled policy should be refreshed when the declaration landscape changes, including:
- plugin activation,
- plugin deactivation,
- plugin update,
- theme switch,
- theme update,
- and relevant settings changes.

### 5.4 Policy Output
The compiled policy may be emitted as:
- enforce mode,
- report-only mode,
- or both during rollout.

### 5.5 Audit Layer
The system should support comparing:
- declared behavior,
- compiled policy,
- and observed runtime behavior.

## 6. Required Declaration Concepts

Any compliant implementation should be able to represent at least these concepts:
- affected context,
- CSP directive,
- source expression or origin,
- declaration category,
- hash or nonce requirement,
- optional `declared_by` metadata,
- and any compilation-relevant declaration attributes.

## 7. Hooks And Registry Model

The recommended implementation shape is a registration hook backed by a central registry.

Illustrative example:

```php
add_action('wp_csp_register', function ($registry): void {
    $registry->add_source('connect-src', 'https://api.example.com', [
        'contexts' => ['frontend'],
        'declared_by' => [
            'name' => 'Example Plugin',
            'slug' => 'example-plugin',
            'kind' => 'plugin',
        ],
    ]);
});
```

The hook name and registry API are illustrative. The specification does not require these exact identifiers.

## 8. Declaration Records

Each declaration record should be able to capture:
- optional `declared_by` metadata,
- affected contexts,
- directive,
- source or hash details,
- nonce requirement where relevant,
- and any other implementation-specific metadata needed for compilation or audit.

Illustrative `declared_by` metadata:

```php
[
    'name' => 'Example Plugin',
    'slug' => 'example-plugin',
    'kind' => 'plugin',
    'version' => '1.0.0',
]
```

`declared_by` is optional metadata for ownership, debugging, and audit. It should not be required for policy compilation.

## 9. Contexts

Contexts represent request surfaces that may need different directives.

### 9.1 Context Categories

```php
[
    'frontend',
    'admin',
    'login',
    'rest',
    'editor',
    'embed',
]
```

Each context is optional.

### 9.2 Common Context Fields

An implementation may additionally support:
- inherited contexts,
- admin/editor sub-contexts,
- or environment-specific overlays.

## 10. Sources

Declares source expressions, origins, or CSP-relevant external requirements.

```php
add_action('wp_csp_register', function ($registry): void {
    $registry->add_source('frame-src', 'https://js.stripe.com', [
        'contexts' => ['frontend'],
        'declared_by' => ['name' => 'Payments', 'kind' => 'plugin'],
    ]);
});
```

Source declarations should support:
- one directive,
- one source expression or origin,
- one or more contexts,
- optional `declared_by` metadata,
- and any declaration attributes needed by the compiler.

## 11. Hashes And Nonces

Declares hashes or nonce-related requirements for unavoidable inline code.

```php
add_action('wp_csp_register', function ($registry): void {
    $registry->add_hash('style-src', 'sha256', 'base64-hash', [
        'contexts' => ['admin'],
        'declared_by' => ['name' => 'Legacy Admin UI', 'kind' => 'plugin'],
    ]);
});
```

An implementation MAY choose nonces, hashes, or both depending on the directive and runtime behavior involved.

Script handling should strongly prefer nonce/hash-based strict CSP patterns.

Style handling should support hashes and other declaration metadata, but style-related inline behavior should remain highly visible as debt rather than being normalized as best practice.

## 12. Compilation Model

The proposal assumes:
- declaration sources expose their requirements,
- one implementation compiles the final policy per response context,
- and the resulting policy can be emitted as enforce or report-only depending on rollout stage.

## 13. Semantics & Interpretation

* Sources and hashes represent declared CSP behavior
* A compiler SHOULD produce one final policy per response context
* Missing declarations imply **unknown or undeclared resource requirements**
* Implementations SHOULD support validation against observed runtime behavior where possible

## 14. Validation Rules

* Declarations MAY preserve `declared_by` metadata
* Directive names SHOULD map to real CSP directives
* Duplicate entries SHOULD be normalized by consuming tools
* Invalid declarations MUST be ignored gracefully
* Declarations SHOULD be auditable against the actual emitted resources

## 15. Security Considerations

* Specification data MUST NOT be blindly trusted as a secure final policy
* Implementations SHOULD align with current CSP best practices
* Inline CSS and JS SHOULD be removed where possible instead of normalized into permanent hashes
* Inline style attributes SHOULD be treated as a problem to eliminate, not a comfortable steady state
* Consuming tools SHOULD report discrepancies between declared and observed behavior

## 16. WordPress-Oriented Implementation Model (Non-Normative)

The recommended WordPress-oriented implementation model is:
- declaration registration through hooks,
- a central registry as the canonical declaration surface,
- compiler support for report-only mode,
- and declared-versus-observed drift reporting.

Additional tooling may build on top of that model through:
- generated machine-readable artifacts for CI and audits,
- CSP metadata attached to asset registration,
- per-handle asset declarations,
- and integration with deployment or server configuration tooling.

## 17. Future Extensions (Non-Normative)

* Per-handle WordPress asset mapping
* Automatic hash generation
* Hook and filter contribution registries
* Report-only compilation and violation collection
* Declared-versus-observed drift reporting
* Environment-specific overlays
* Integration with server configuration generators

## 18. License

This specification is open and may be freely implemented.
