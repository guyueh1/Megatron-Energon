<!--- Copyright (c) 2026, NVIDIA CORPORATION.
SPDX-License-Identifier: BSD-3-Clause -->

# Construction metadata snapshots

Applications may prepare a versioned metadata package for a frozen dataset
revision and serve it while constructing a loader:

```python
from megatron.energon.epathlib.metadata_snapshot import (
    MetadataSnapshot,
    use_metadata_snapshot,
)

snapshot = MetadataSnapshot.load(
    "/local/loader-metadata.json",
    sha256=expected_digest,
    recipe_identity=expected_recipe_identity,
)
with use_metadata_snapshot(snapshot):
    dataset = build_dataset()
# Start iteration and workers only after leaving the context.
```

Version 1 packages contain `format_version`, `recipe_identity`, base64-encoded
`contents`, and maps named `file_checks`, `sizes`, and `jsonl_sizes`. Keys are
logical EPath strings; `jsonl_sizes` contains terminal byte offsets. A producer
must capture these values from the intended immutable dataset revision. The
loader rejects a mismatched package digest or recipe identity.

Recorded reads receive independent streams. Unrecorded paths retain their normal
storage backend. Leaving the context restores ordinary access, including after
an exception. Keep media payloads outside the package and rebuild it after any
metadata revision. Digest validation identifies the selected package; it does not
check whether source files have changed. Applications own capture, distribution,
and snapshot selection; this API does not provide an automatic cache or launcher.
