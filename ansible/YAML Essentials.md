# YAML Essentials
> [!NOTE]
>YAML is a human-readable data-serialisation language that uses mappings, sequences, scalars, and indentation to represent portable structured data for configuration, automation, and exchange.
## Purpose and design
YAML is a Unicode-based data serialisation language for representing structured data. Its recursive name means "YAML Ain't Markup Language". Clark Evans, Ingy dot Net, and Oren Ben-Kiki designed it around data structures common to dynamic programming languages such as JavaScript, Perl, PHP, Python, and Ruby.

YAML prioritises human readability, portability between programming languages, compatibility with native data structures, a consistent data model, one-pass processing, extensibility, and ease of implementation. These qualities suit configuration files, data exchange, object persistence, logs, and infrastructure definitions. YAML commonly appears in tools such as Ansible, Kubernetes, and Salt.

YAML has three node kinds:
- mappings, which associate keys with values
- sequences, which hold ordered entries
- scalars, which hold single values

A YAML processor constructs these nodes according to a selected schema. The schema determines whether an unquoted scalar represents a string, integer, floating-point number, Boolean, null value, or another supported type.

The serialisation process converts an application's data graph into a YAML character stream. Loading reverses that process. Presentation choices such as indentation, comments, anchors, and scalar style can change the source text without changing the represented values. This separation explains why parsers preserve data more reliably than formatting or comments.
## Syntax and layout
Indentation defines block structure. Indentation must use spaces, not tabs, but YAML does not require two spaces. Writers may choose any consistent number of spaces for each level. Tabs remain valid in some non-indentation contexts.

A colon followed by separation whitespace introduces a block mapping value. A dash followed by separation whitespace introduces a block sequence entry. Keys in the same mapping must be unique.

Sibling nodes align at the same indentation level, while child nodes sit farther to the right. The exact indentation width may change between nested levels if the structure remains valid, though a consistent house style reduces errors. Indicators such as `:`, `-`, `#`, `[`, `]`, `{`, `}`, `&`, `*`, `!`, `|`, and `>` have context-dependent roles. Quotation marks or block scalar forms protect content that could otherwise be read as syntax.

```yaml
host: phl-42
datacentre:
  location: Philadelphia
  cabinet: "13"
  cabinet_unit: "3"
roles:
  - webserver
  - wordpress_database
```

The `.yaml` extension is preferred, although `.yml` is also common. Applications may use other extensions. A code-aware editor helps preserve indentation and can validate syntax against an application-specific schema.

YAML character streams may use UTF-8, UTF-16, or UTF-32, subject to restrictions on control characters and invalid Unicode code points. Conforming processors must support UTF-8 and UTF-16, while UTF-32 support is optional. UTF-8 provides the safest interchange format. JSON exchanged outside a closed ecosystem must use UTF-8, not UTF-32.
## Block and flow styles
Block style uses indentation and line breaks. It usually offers the clearest form for people editing configuration. Flow style encloses mappings in braces and sequences in brackets, with commas between entries. Flow style resembles JSON and can appear inside block-style documents.

```yaml
datacentre: {location: Philadelphia, cabinet: "13"}
roles: [webserver, wordpress_database]
```

YAML 1.2 aims to accept JSON syntax, but flow style is not an extension added for machine efficiency. Style changes presentation rather than the represented data.
## Mappings, sequences, and collections
A mapping associates key nodes with value nodes. A sequence orders zero or more nodes. Both are collections, and either can contain mappings, sequences, or scalars. Indentation shows which nodes belong to a parent collection.

Block mappings normally place each key-value pair on its own line. Nested mappings begin after a key with no value on that line. Block sequences place `-` before each entry, and a sequence entry may begin a mapping. Flow collections use comma-separated entries and permit nested flow or block-compatible nodes where the grammar allows them. These combinations represent records, tables, inventories, and configuration trees without introducing new node kinds.

Empty collections are valid as `[]` and `{}`. An empty block sequence entry is also valid and represents an empty node, which a schema commonly resolves as null. Applications may impose stricter rules through schemas even when the YAML syntax itself is valid.

```yaml
servers:
  - host: phl-42
    roles: [web, database]
  - host: phl-43
    roles: []
```
## Scalars
Plain scalars do not require quotation marks when their characters and context are unambiguous. Quoting prevents unintended type resolution and protects characters with syntactic roles. Single-quoted scalars treat most content literally. Double-quoted scalars support escape sequences such as `\n`.

Literal block scalars use `|` and preserve line breaks. Folded block scalars use `>` and usually convert ordinary line breaks to spaces. Indentation, blank lines, more-indented lines, and optional chomping indicators affect the final value.

Block scalar indentation is inferred from the first non-empty content line unless an explicit indentation indicator supplies it. The default clip behaviour keeps one final line break. The strip indicator `-` removes trailing line breaks, while the keep indicator `+` retains them. These controls distinguish prose that should fold into one paragraph from scripts, certificates, or messages whose physical line structure carries meaning.

```yaml
downtime: |
  2026-10-31: kernel upgrade
  2027-02-02: security update
comment: >
  High I/O remains under
  investigation.
```
## Documents and comments
A YAML stream may contain one or more documents. `---` marks an explicit document start, while `...` marks an explicit document end. A stream can begin with one implicit document when no directives are present, but later documents require explicit starts. The markers are presentation syntax, not directives. Directives begin with `%`, apply to the following document, and must precede its `---` marker.

The `%YAML 1.2` directive declares a YAML version for the next document. `%TAG` defines a tag shorthand. Each directive ends at the document start marker, and directives do not persist into later documents. Applications such as Kubernetes often store several resource objects as separate YAML documents in one stream, with `---` separating the objects.

Comments begin with `#` outside quoted scalars and continue to the end of the line. An inline comment requires whitespace before `#`. A comment need not contain whitespace after `#`, although a space improves readability. A `#` inside a quoted scalar remains part of the value. Blank lines are separate from comments and carry no content.
## Tags
Every node has a tag that identifies its type. A processor may resolve a plain scalar implicitly, while an explicit tag can force or declare a type. The shorthand `!!str` identifies the standard string tag.

```yaml
cabinet: !!str 13
```

The `%TAG` directive establishes a handle for a tag prefix. The handle and suffix combine into a complete tag, which is a URI. Local tags beginning with `!` have application-specific meaning and need not use a global URI. Applications must recognise custom tags before they can construct corresponding native values safely.
## Anchors and aliases
An anchor labels a node with `&name`. A later alias such as `*name` refers to that node, allowing a representation graph to reuse data without repeating it. An alias does not copy presentation details such as comments. Anchor names may be reused, and each alias refers to the most recent preceding anchor with that name.

Anchors and aliases are scoped to one document. An alias cannot refer to an anchor in another document within the same stream. Reusable data must therefore remain in the same document or be assembled by application-level tooling.

```yaml
wordpress_roles: &wordpress_roles
  - webserver
  - wordpress_database
hosts:
  - name: phl-42
    roles: *wordpress_roles
  - name: phl-43
    roles: *wordpress_roles
```
## Reliable practice
Readable YAML uses consistent indentation, descriptive keys, restrained comments, and quotes where schema resolution could surprise readers. Validation should cover both YAML syntax and the consuming application's schema. Parser versions and supported schemas also require attention because YAML 1.1 and YAML 1.2 can resolve some plain scalars differently.

Applications should use maintained parsers rather than ad hoc text processing. Loaders should restrict application-specific tags when input is untrusted because object construction can invoke unsafe behaviour in some libraries. Alias expansion also warrants resource limits to prevent excessive memory or processing. Round-trip tests should confirm that dumping and loading preserve intended values, while schema validation should detect missing keys, unexpected properties, invalid types, and unsupported values.