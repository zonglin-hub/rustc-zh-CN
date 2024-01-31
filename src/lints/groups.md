# Lint Groups

`rustc` 有个叫做 "lint group" 的概念， 你可以通过一个名称来切换其余几个 lint。

例如， `nonstandard-style` lint 一次便可设置 `non-camel-case-types`,
`non-snake-case`， 和 `non-upper-case-globals`。所以以下两条命令是等价的:

```bash
$ rustc -D nonstandard-style
$ rustc -D non-camel-case-types -D non-snake-case -D non-upper-case-globals
```

这儿有一个包含每个 lint group，和组成它们的 lint 的列表:

| Group | Description | Lints |
|-------|-------------|-------|
| warnings | 所有设置为发出问题警告的 lint | See [warn-by-default] for the default set of warnings |
| future-incompatible | 用来检测代码未来兼容性问题的 lint | [ambiguous-associated-items], [ambiguous-glob-imports], [byte-slice-in-packed-struct-with-derive], [cenum-impl-drop-cast], [coherence-leak-check], [coinductive-overlap-in-coherence], [conflicting-repr-hints], [const-evaluatable-unchecked], [const-patterns-without-partial-eq], [deprecated-cfg-attr-crate-type-name], [deref-into-dyn-supertrait], [elided-lifetimes-in-associated-constant], [forbidden-lint-groups], [ill-formed-attribute-input], [illegal-floating-point-literal-pattern], [implied-bounds-entailment], [indirect-structural-match], [invalid-alignment], [invalid-doc-attributes], [invalid-type-param-default], [late-bound-lifetime-arguments], [legacy-derive-helpers], [macro-expanded-macro-exports-accessed-by-absolute-paths], [missing-fragment-specifier], [nontrivial-structural-match], [order-dependent-trait-objects], [patterns-in-fns-without-body], [pointer-structural-match], [proc-macro-back-compat], [proc-macro-derive-resolution-fallback], [pub-use-of-private-extern-crate], [repr-transparent-external-private-fields], [semicolon-in-expressions-from-macros], [soft-unstable], [suspicious-auto-trait-impls], [uninhabited-static], [unstable-name-collisions], [unstable-syntax-pre-expansion], [unsupported-calling-conventions], [where-clauses-object-safety] |
| let-underscore | 检测可能无效的通配符 let 绑定。 | [let-underscore-drop], [let-underscore-lock] |
| nonstandard-style | 违反标准命令约定 | [non-camel-case-types], [non-snake-case], [non-upper-case-globals] |
| rust-2018-compatibility | 用来将代码从 Rust 2015 向 Rust 2018 转移的 lint | [absolute-paths-not-starting-with-crate], [anonymous-parameters], [keyword-idents], [tyvar-behind-raw-pointer] |
| rust-2018-idioms | 用来推动你适应Rust 2018 惯用features的 lint | [bare-trait-objects], [elided-lifetimes-in-paths], [ellipsis-inclusive-range-patterns], [explicit-outlives-requirements], [unused-extern-crates] |
| rust-2021-compatibility | 用于将代码从 2018 迁移到 2021 的 lint。 | [array-into-iter], [bare-trait-objects], [ellipsis-inclusive-range-patterns], [non-fmt-panics], [rust-2021-incompatible-closure-captures], [rust-2021-incompatible-or-patterns], [rust-2021-prefixes-incompatible-syntax], [rust-2021-prelude-collisions] |
| unused | 用来检测声明但未使用，或是语法冗余的 lint | [dead-code], [map-unit-fn], [path-statements], [redundant-semicolons], [unreachable-code], [unreachable-patterns], [unused-allocation], [unused-assignments], [unused-attributes], [unused-braces], [unused-doc-comments], [unused-extern-crates], [unused-features], [unused-imports], [unused-labels], [unused-macro-rules], [unused-macros], [unused-must-use], [unused-mut], [unused-parens], [unused-unsafe], [unused-variables] |

[warn-by-default]: listing/warn-by-default.md
[ambiguous-associated-items]: listing/deny-by-default.md#ambiguous-associated-items
[ambiguous-glob-imports]: listing/warn-by-default.md#ambiguous-glob-imports
[byte-slice-in-packed-struct-with-derive]: listing/warn-by-default.md#byte-slice-in-packed-struct-with-derive
[cenum-impl-drop-cast]: listing/deny-by-default.md#cenum-impl-drop-cast
[coherence-leak-check]: listing/warn-by-default.md#coherence-leak-check
[coinductive-overlap-in-coherence]: listing/deny-by-default.md#coinductive-overlap-in-coherence
[conflicting-repr-hints]: listing/deny-by-default.md#conflicting-repr-hints
[const-evaluatable-unchecked]: listing/warn-by-default.md#const-evaluatable-unchecked
[const-patterns-without-partial-eq]: listing/warn-by-default.md#const-patterns-without-partial-eq
[deprecated-cfg-attr-crate-type-name]: listing/deny-by-default.md#deprecated-cfg-attr-crate-type-name
[deref-into-dyn-supertrait]: listing/warn-by-default.md#deref-into-dyn-supertrait
[elided-lifetimes-in-associated-constant]: listing/warn-by-default.md#elided-lifetimes-in-associated-constant
[forbidden-lint-groups]: listing/warn-by-default.md#forbidden-lint-groups
[ill-formed-attribute-input]: listing/deny-by-default.md#ill-formed-attribute-input
[illegal-floating-point-literal-pattern]: listing/warn-by-default.md#illegal-floating-point-literal-pattern
[implied-bounds-entailment]: listing/deny-by-default.md#implied-bounds-entailment
[indirect-structural-match]: listing/warn-by-default.md#indirect-structural-match
[invalid-doc-attributes]: listing/warn-by-default.md#invalid-doc-attributes
[invalid-type-param-default]: listing/deny-by-default.md#invalid-type-param-default
[late-bound-lifetime-arguments]: listing/warn-by-default.md#late-bound-lifetime-arguments
[legacy-derive-helpers]: listing/warn-by-default.md#legacy-derive-helpers
[macro-expanded-macro-exports-accessed-by-absolute-paths]: listing/deny-by-default.md#macro-expanded-macro-exports-accessed-by-absolute-paths
[missing-fragment-specifier]: listing/deny-by-default.md#missing-fragment-specifier
[nontrivial-structural-match]: listing/warn-by-default.md#nontrivial-structural-match
[order-dependent-trait-objects]: listing/deny-by-default.md#order-dependent-trait-objects
[patterns-in-fns-without-body]: listing/deny-by-default.md#patterns-in-fns-without-body
[pointer-structural-match]: listing/allowed-by-default.md#pointer-structural-match
[proc-macro-back-compat]: listing/deny-by-default.md#proc-macro-back-compat
[proc-macro-derive-resolution-fallback]: listing/deny-by-default.md#proc-macro-derive-resolution-fallback
[pub-use-of-private-extern-crate]: listing/deny-by-default.md#pub-use-of-private-extern-crate
[repr-transparent-external-private-fields]: listing/warn-by-default.md#repr-transparent-external-private-fields
[semicolon-in-expressions-from-macros]: listing/warn-by-default.md#semicolon-in-expressions-from-macros
[soft-unstable]: listing/deny-by-default.md#soft-unstable
[suspicious-auto-trait-impls]: listing/warn-by-default.md#suspicious-auto-trait-impls
[uninhabited-static]: listing/warn-by-default.md#uninhabited-static
[unstable-name-collisions]: listing/warn-by-default.md#unstable-name-collisions
[unstable-syntax-pre-expansion]: listing/warn-by-default.md#unstable-syntax-pre-expansion
[unsupported-calling-conventions]: listing/warn-by-default.md#unsupported-calling-conventions
[where-clauses-object-safety]: listing/warn-by-default.md#where-clauses-object-safety
[let-underscore-drop]: listing/allowed-by-default.md#let-underscore-drop
[let-underscore-lock]: listing/deny-by-default.md#let-underscore-lock
[non-camel-case-types]: listing/warn-by-default.md#non-camel-case-types
[non-snake-case]: listing/warn-by-default.md#non-snake-case
[non-upper-case-globals]: listing/warn-by-default.md#non-upper-case-globals
[absolute-paths-not-starting-with-crate]: listing/allowed-by-default.md#absolute-paths-not-starting-with-crate
[anonymous-parameters]: listing/warn-by-default.md#anonymous-parameters
[keyword-idents]: listing/allowed-by-default.md#keyword-idents
[tyvar-behind-raw-pointer]: listing/warn-by-default.md#tyvar-behind-raw-pointer
[bare-trait-objects]: listing/warn-by-default.md#bare-trait-objects
[elided-lifetimes-in-paths]: listing/allowed-by-default.md#elided-lifetimes-in-paths
[ellipsis-inclusive-range-patterns]: listing/warn-by-default.md#ellipsis-inclusive-range-patterns
[explicit-outlives-requirements]: listing/allowed-by-default.md#explicit-outlives-requirements
[unused-extern-crates]: listing/allowed-by-default.md#unused-extern-crates
[array-into-iter]: listing/warn-by-default.md#array-into-iter
[bare-trait-objects]: listing/warn-by-default.md#bare-trait-objects
[ellipsis-inclusive-range-patterns]: listing/warn-by-default.md#ellipsis-inclusive-range-patterns
[non-fmt-panics]: listing/warn-by-default.md#non-fmt-panics
[rust-2021-incompatible-closure-captures]: listing/allowed-by-default.md#rust-2021-incompatible-closure-captures
[rust-2021-incompatible-or-patterns]: listing/allowed-by-default.md#rust-2021-incompatible-or-patterns
[rust-2021-prefixes-incompatible-syntax]: listing/allowed-by-default.md#rust-2021-prefixes-incompatible-syntax
[rust-2021-prelude-collisions]: listing/allowed-by-default.md#rust-2021-prelude-collisions
[dead-code]: listing/warn-by-default.md#dead-code
[map-unit-fn]: listing/warn-by-default.md#map-unit-fn
[path-statements]: listing/warn-by-default.md#path-statements
[redundant-semicolons]: listing/warn-by-default.md#redundant-semicolons
[unreachable-code]: listing/warn-by-default.md#unreachable-code
[unreachable-patterns]: listing/warn-by-default.md#unreachable-patterns
[unused-allocation]: listing/warn-by-default.md#unused-allocation
[unused-assignments]: listing/warn-by-default.md#unused-assignments
[unused-attributes]: listing/warn-by-default.md#unused-attributes
[unused-braces]: listing/warn-by-default.md#unused-braces
[unused-doc-comments]: listing/warn-by-default.md#unused-doc-comments
[unused-extern-crates]: listing/allowed-by-default.md#unused-extern-crates
[unused-features]: listing/warn-by-default.md#unused-features
[unused-imports]: listing/warn-by-default.md#unused-imports
[unused-labels]: listing/warn-by-default.md#unused-labels
[unused-macro-rules]: listing/allowed-by-default.md#unused-macro-rules
[unused-macros]: listing/warn-by-default.md#unused-macros
[unused-must-use]: listing/warn-by-default.md#unused-must-use
[unused-mut]: listing/warn-by-default.md#unused-mut
[unused-parens]: listing/warn-by-default.md#unused-parens
[unused-unsafe]: listing/warn-by-default.md#unused-unsafe
[unused-variables]: listing/warn-by-default.md#unused-variables

此外，有一个 `bad-style` lint 组是 `nonstandard-style` 的已弃用别名。

最后，您可以通过调用 `rustc -W help` 来查看上面的表格。这将为您安装的特定编译器提供确切的值。
