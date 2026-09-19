# Formats

Alto Code Diff exchanges changes as standard unified diffs and understands
common Git headers. A `DiffResult` represents one comparison; a `DiffBundle`
groups path-labelled results for multi-file patches. The package operates on
strings and associative arrays, while the application owns file-system access.

## Emit a patch

`UnifiedEmitter::emit()` accepts a `DiffResult` or a multi-file `DiffBundle`.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\Code\Diff\Diff;
use Alto\Code\Diff\Patch\UnifiedEmitter;

$result = Diff::build()->compare(
    "Line one\nLine two\n",
    "Line one\nLine two changed\n",
);

$patch = (new UnifiedEmitter())->emit($result);
echo $patch;
```

A bare result uses `a` and `b` as labels. To control paths or represent multiple files, construct `DiffFile` objects and place them in a `DiffBundle`.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\Code\Diff\Diff;
use Alto\Code\Diff\Model\DiffBundle;
use Alto\Code\Diff\Model\DiffFile;
use Alto\Code\Diff\Patch\UnifiedEmitter;

$result = Diff::build()->compare("old\n", "new\n");
$file = new DiffFile(
    oldPath: 'src/example.txt',
    newPath: 'src/example.txt',
    result: $result,
    headers: [
        'diff' => 'diff --git a/src/example.txt b/src/example.txt',
        'index' => 'index 3367afd..3e75765 100644',
    ],
);

echo (new UnifiedEmitter())->emit(new DiffBundle([$file]));
```

Supported metadata keys are `diff`, `index`, `old_mode`, `new_mode`, `new_file_mode`, `deleted_file_mode`, `similarity_index`, `rename_from`, `rename_to`, `copy_from`, and `copy_to`.

## Parse a patch

`UnifiedParser::parse(string $patch): DiffBundle` validates hunk lengths and returns files, paths, headers, results, hunks, and edits. Leading `a/` and `b/` path prefixes are removed.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\Code\Diff\Patch\UnifiedParser;

$patch = <<<'PATCH'
diff --git a/example.txt b/example.txt
index 3367afd..3e75765 100644
--- a/example.txt
+++ b/example.txt
@@ -1 +1 @@
-old
+new
PATCH;

$bundle = (new UnifiedParser())->parse($patch);
$file = $bundle->files()[0];

printf("%s -> %s\n", $file->oldPath, $file->newPath);
```

The parser recognizes file modes, creation, deletion, rename, copy, similarity,
and index headers. It preserves no-trailing-newline markers. See
[Errors](errors.md) for malformed hunks and binary patch markers.

## Apply a single-file patch

`PatchApplier::apply(string $original, string $unifiedPatch): string` accepts exactly one patched file. An empty patch returns the original string.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\Code\Diff\Patch\PatchApplier;

$original = "Line one\nLine two\n";
$patch = <<<'PATCH'
--- a/example.txt
+++ b/example.txt
@@ -1,2 +1,2 @@
 Line one
-Line two
+Line two changed
PATCH;

$updated = (new PatchApplier())->apply($original, $patch);
echo $updated;
```

The constructor accepts `fuzz` and `maxBytes`, both defaulting to `0` and
`5_000_000`. Fuzz searches that many lines before and after a hunk's expected
position. It does not resolve conflicts or accept different source text.

## Apply a bundle

Use `applyBundle(array $files, DiffBundle $bundle): array` for multiple files. The input and result use `path => content` maps.

The method handles modifications, renames, creations from `/dev/null`, and
deletions to `/dev/null`. The library returns updated content but never writes
it to disk. Missing paths and unmatched hunks are covered in [Errors](errors.md).

## Round trip

This example emits, parses, and applies a patch while keeping both files in
memory. The path keys are data; the package does not open them.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\Code\Diff\Diff;
use Alto\Code\Diff\Model\DiffBundle;
use Alto\Code\Diff\Model\DiffFile;
use Alto\Code\Diff\Patch\PatchApplier;
use Alto\Code\Diff\Patch\UnifiedEmitter;
use Alto\Code\Diff\Patch\UnifiedParser;

$files = ['a.txt' => "old\n", 'b.txt' => "keep\n"];
$change = new DiffFile(
    'a.txt',
    'a.txt',
    Diff::build()->compare($files['a.txt'], "new\n"),
);
$patch = (new UnifiedEmitter())->emit(new DiffBundle([$change]));
$bundle = (new UnifiedParser())->parse($patch);
$updated = (new PatchApplier())->applyBundle($files, $bundle);

echo json_encode($updated, JSON_THROW_ON_ERROR), "\n";
```

The output is:

```text
{"a.txt":"new\n","b.txt":"keep\n"}
```
