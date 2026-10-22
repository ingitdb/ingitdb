# Cache-Control conformance vectors

Normative vectors for the `cache_control` setting (see the Feature
`cache-control` in [ingitdb-go](https://github.com/ingitdb/ingitdb-go/tree/main/spec/features/cache-control)).
An implementation conforms if it matches every row of [`vectors.yaml`](./vectors.yaml).

- `parse` rows: the validator's verdict (`valid`) and the reader's view of the
  directives. Validators reject; readers drop the directive and keep going.
- `freshness` rows: the decision table for how long a fetched file may be reused.

Each implementation vendors this file byte-for-byte (Go: `ingitdb/testdata/cache_control_vectors.yaml`;
TypeScript: `packages/client/src/cache/__fixtures__/cache-control-vectors.yaml`) and runs it as a test.
Change the vectors here first, then re-vendor.
