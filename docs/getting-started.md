# Getting started

Compare two strings, then choose a renderer for the destination of the result.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\Code\Diff\Diff;
use Alto\Code\Diff\Renderer\UnifiedRenderer;

$old = "line one\nline two\n";
$new = "line one\nline two changed\n";

$result = Diff::build()->compare($old, $new);

echo (new UnifiedRenderer('old.txt', 'new.txt'))->render($result);
```

The output is a standard unified diff:

```diff
--- old.txt
+++ new.txt
@@ -1,2 +1,2 @@
 line one
-line two
+line two changed
```

The returned string has no final line break. Append `PHP_EOL` when the output
destination requires one. Source newline-at-EOF metadata remains available in
the diff result.

`Diff::build()` creates an immutable builder. Each configuration method returns a new instance, so a configured builder can be reused safely.

Next, configure [diffing](diffing.md), select another [renderer](rendering.md),
exchange unified [formats](formats.md), or handle [errors](errors.md).
