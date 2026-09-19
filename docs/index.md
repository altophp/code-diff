# Alto Code Diff

Alto Code Diff compares text, renders structured differences, and parses,
emits, or applies unified patches. It works with strings and in-memory path
maps, so applications retain control of file and process access.

```php
use Alto\Code\Diff\Diff;
use Alto\Code\Diff\Renderer\UnifiedRenderer;

$result = Diff::build()->compare("old\n", "new\n");
$output = (new UnifiedRenderer('old.txt', 'new.txt'))->render($result);
```

## Documentation

- [Installation](installation.md): install the package and verify its requirements.
- [Getting started](getting-started.md): compare two strings and render a unified diff.
- [Diffing](diffing.md): configure comparisons, inspect results, and select an engine.
- [Rendering](rendering.md): produce unified text, HTML, JSON, or ANSI output.
- [Formats](formats.md): emit, parse, and apply single-file or multi-file patches.
- [Errors](errors.md): recover from rejected inputs and patches.

The package rejects binary input. It does not read or write files, invoke Git,
or resolve patch conflicts automatically.
