# Errors

All package exceptions implement `CodeDiffExceptionInterface`. Catch a precise
exception when the application has a recovery path, or use the shared interface
at a request, job, or command boundary.

| Exception | Cause | Recovery |
| --- | --- | --- |
| `SizeLimitException` | Comparison or patch input exceeds `maxBytes` | Compare smaller meaningful units, or deliberately raise the limit after reviewing memory and execution time |
| `BinaryInputException` | Text input contains binary indicators, or a patch contains binary file markers | Decode the content to an appropriate textual representation, or use a binary comparison tool |
| `ParseException` | A unified patch has an invalid header, hunk count, edit line, or newline marker | Obtain the complete patch and preserve its headers and hunk counts rather than guessing missing lines |
| `PatchApplyException` | A patch contains several files for `apply()`, a path is missing, or a hunk does not match | Use `applyBundle()` for several files, verify the source revision and path map, then inspect `hunkIndex` |

## Comparison limits

`Diff::compare()` validates both strings before an engine computes changes.
Binary detection checks null bytes and excessive control characters near the
beginning of the input. Retrying the same bytes with a larger size limit does
not make binary content valid text.

`contextLines()` rejects negative values and `Diff::maxBytes()` rejects
non-positive values. These configuration errors raise `InvalidArgumentException`
and should be fixed before processing user input.

## Patch recovery

`PatchApplyException::hunkIndex` identifies the zero-based hunk that failed.
Fuzz searches nearby line positions; it does not ignore changed content and is
not conflict resolution. If a patch targets another revision, regenerate it
against the intended base or let the application present a conflict workflow.

For multi-file patches, every required old path must exist in the input
`path => content` map. Keep the returned map separate until the application has
decided how to persist it. Alto Code Diff never writes files or invokes Git.
