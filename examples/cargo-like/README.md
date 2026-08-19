# cargo-like

`cargo-like` is a deliberately serious example of a Kira CLI built with the
ArgumentParser library. It is a small build coordinator rather than a toy
echo command:

- `build` walks a real project tree and renders a cargo-style five-line status
  display;
- progress frames are rewritten in place with ANSI cursor movement, while
  `--plain` keeps every frame visible for logs and CI;
- `--profile`, `--target`, `--jobs`, repeated `--feature`, repeated `--unit`,
  `--locked`, `--frames`, and `--delay` are typed command arguments;
- `check` validates `package.kira` and reports source-file coverage;
- `metadata` emits a human-readable or JSON project summary.

The app uses the executable's real process arguments through
`commandLineArguments()`. Its manifest is `package.kira` and its dependency
points to the local ArgumentParser package.

Build and run it from the repository root:

```powershell
kira build --backend llvm examples/cargo-like
& .\examples\cargo-like\app\.kira-build\main.exe build . --profile release --target x86_64-pc-windows-msvc --jobs 4 --feature cli --feature progress --frames 6 --delay 120
& .\examples\cargo-like\app\.kira-build\main.exe build . --plain --frames 3 --delay 0
& .\examples\cargo-like\app\.kira-build\main.exe check . --strict
& .\examples\cargo-like\app\.kira-build\main.exe metadata . --format json
```

The default `build` command is intentionally observable: the executable
scans the project, waits between frames, and redraws the same five terminal
lines until the final status is reached.
