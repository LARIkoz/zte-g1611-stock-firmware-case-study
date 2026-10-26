# Proposed upstream README addition: decoding type 6

Prepared documentation proposal; not submitted or accepted. The inspected upstream README contains other payload examples but no dedicated type-6 usage section. The existing implementation already supports this path; this proposal documents it and claims no new codec or G1611 support.

## Suggested README text

### Decode a type-6 configuration when its key and IV seed strings are known

The type-6 decoder requires both `--key` and `--iv`:

```sh
python3 examples/decode.py "$INPUT_CFG" "$OUTPUT_XML" --key "$KEY_SEED" --iv "$IV_SEED"
```

Set these variables locally to the input/output paths and the actual seed strings required by the configuration. No device-specific seed strings are supplied by this example.

In this implementation, `CBCXcryptor.set_key` derives the AES key as SHA-256 of the key string, and the CBC IV as the first 16 bytes of SHA-256 of the IV string. Hexadecimal text supplied on the CLI is still treated as text; it is not automatically interpreted as raw AES bytes.

The decoder validates the inner payload header before decompression. A native device's label "type 6" does not by itself establish that its complete file/container is accepted by this tool. This example does not add encoder compatibility or qualify an untested router model.

The decoder can print the successful key input, and decrypted XML may contain credentials. Keep actual parameters and output private; sanitize logs before attaching them to an issue.

## Proposal basis

- Inspected commit: `e3a1a7f56c2df26a02f52f72b16173d0702d9ad0`.
- Existing [type-6 decoder](https://github.com/mkst/zte-config-utility/blob/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0/examples/decode.py).
- Existing [CBC parameter derivation](https://github.com/mkst/zte-config-utility/blob/e3a1a7f56c2df26a02f52f72b16173d0702d9ad0/zcu/xcryptors.py).
- Existing feature credited to [PR #204](https://github.com/mkst/zte-config-utility/pull/204).

This is a small documentation contribution candidate grounded in existing code. It can be proposed independently of the unresolved G1611 compatibility investigation. A code/test contribution would need its own patch and executed tests; neither is claimed by this document.
