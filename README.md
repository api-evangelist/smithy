# Smithy

Smithy is an open source, protocol-agnostic interface definition language (IDL) and toolchain developed at AWS for defining, validating, and generating API clients, servers, and documentation for any programming language. It powers the AWS SDK code generation pipeline and supports protocol-agnostic API modeling with shapes, traits, validators, and code generators. Smithy IDL 2.0 is the current stable version.

## Overview

Smithy models consist of **shapes** (the type system) and **traits** (annotations). Models can be written in the Smithy IDL syntax or represented as JSON AST. The Smithy CLI can build, validate, diff, and transform models into generated artifacts.

## Supported Protocols

- **AWS Protocols**: restJson1, restXml, json1.0, json1.1, query, ec2Query
- **Smithy Protocols**: rpcv2Cbor
- **Event Streaming**: MQTT bindings

## Artifacts

### JSON Schema
- [smithy-shape-schema.json](json-schema/smithy-shape-schema.json) — Schema for a Smithy shape (all types)
- [smithy-model-schema.json](json-schema/smithy-model-schema.json) — Schema for a Smithy model in JSON AST format

### JSON Structure
- [smithy-model-structure.json](json-structure/smithy-model-structure.json) — Smithy model JSON AST structure documentation

### JSON-LD
- [smithy-context.jsonld](json-ld/smithy-context.jsonld) — Linked data context for Smithy model concepts

### Examples
- [smithy-simple-service-example.json](examples/smithy-simple-service-example.json) — A simple Smithy 2.0 service model in JSON AST

### Rules
- [smithy-rules.yml](rules/smithy-rules.yml) — Spectral ruleset for Smithy-generated OpenAPI conventions

### Vocabulary
- [smithy-vocabulary.yml](vocabulary/smithy-vocabulary.yml) — Normative vocabulary: shapes, traits, protocols, CLI commands

## Code Generators

| Language | Repository |
|----------|------------|
| Java | [smithy-lang/smithy](https://github.com/smithy-lang/smithy) |
| Rust | [smithy-lang/smithy-rs](https://github.com/smithy-lang/smithy-rs) |
| Python | [smithy-lang/smithy-python](https://github.com/smithy-lang/smithy-python) |
| TypeScript | [awslabs/smithy-typescript](https://github.com/awslabs/smithy-typescript) |
| Kotlin | [smithy-lang/smithy-kotlin](https://github.com/smithy-lang/smithy-kotlin) |
| Go | [aws/smithy-go](https://github.com/aws/smithy-go) |

## Developer Resources

- [Smithy 2.0 Documentation](https://smithy.io/2.0/)
- [Specification](https://smithy.io/2.0/spec/)
- [Quick Start](https://smithy.io/2.0/quickstart.html)
- [CLI Guide](https://smithy.io/2.0/guides/smithy-cli/index.html)
- [GitHub Organization](https://github.com/smithy-lang)
- [Examples](https://github.com/smithy-lang/smithy-examples)
- [Awesome Smithy](https://github.com/smithy-lang/awesome-smithy)
- [AWS API Models](https://github.com/aws/api-models-aws)
