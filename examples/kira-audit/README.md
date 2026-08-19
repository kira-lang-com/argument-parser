# kira-audit

`kira-audit` is the repository's single complete example for
`ArgumentParser`. It is a small workspace utility with three real commands:

- `scan` walks a directory or file and reports files, directories, bytes, and
  Kira sources;
- `check` validates a `package.kira` manifest;
- `explain` reads a source file and prints metadata plus a line preview.

Build it with the repository's Kira compiler:

```powershell
kira build --backend llvm examples/kira-audit
```

Then invoke the generated program directly:

```powershell
& .\examples\kira-audit\app\.kira-build\main.exe scan . --max-depth 1 --format json
& .\examples\kira-audit\app\.kira-build\main.exe check . --strict
& .\examples\kira-audit\app\.kira-build\main.exe explain package.kira --lines 4
```

The `@Main` function calls `commandLineArguments()`, so these values come from
the operating system process rather than from a hard-coded token array.
