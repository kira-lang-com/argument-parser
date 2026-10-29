# ArgumentParser for Kira

`ArgumentParser` is a real command-line parsing library for Kira, modeled on
Swift ArgumentParser's command-first design. Commands declare a schema with
`argument`, `option` and `flag` and their modifiers, `@Derive(Parsable)`
projects validated tokens into typed Kira values, and `runCommand` or
`runCommandRouter` owns help, version, errors, and dispatch.

A program depends on it by path:

```kira
let dependencies: [Dependency] = [Dependency { name: "ArgumentParser", path: "../argument-parser" }]
```

## Swift-shaped Kira

Swift:

```swift
import ArgumentParser

@main
struct Repeat: ParsableCommand {
    @Flag(help: "Include a counter with each repetition.")
    var includeCounter = false

    @Option(name: .shortAndLong, help: "How many times to repeat 'phrase'.")
    var count: Int? = nil

    @Argument(help: "The phrase to repeat.")
    var phrase: String

    mutating func run() throws { ... }
}
```

The Kira equivalent keeps the same intent while making the schema explicit:

```kira
import ArgumentParser

@Derive(Parsable)
struct RepeatArguments {
    let includeCounter: Bool
    let count: Number
    let phrase: String
}

ParsableCommand RepeatCommand {
    let configuration: CommandConfiguration {
        return CommandConfiguration { name: "repeat", abstract: "Repeat a phrase" }
    }

    let arguments: [ArgumentSpec] {
        return [
            flag(name: "includeCounter", help: "Include a counter").short("i"),
            option(name: "count", help: "Repetitions").short("c").whole().defaulting("2"),
            argument(name: "phrase", help: "The phrase to repeat"),
        ]
    }

    function run(arguments: borrow ParsedArguments) -> Number {
        let parsed = parse_RepeatArguments(arguments)
        for index in 1..parsed.count + 1 {
            print(parsed.includeCounter ? "\(index): \(parsed.phrase)" : parsed.phrase)
        }
        return 0
    }
}

@Main
function main() -> Number {
    return runCommand(RepeatCommand(), commandLineArguments())
}
```

`commandLineArguments()` reads the compiled program's actual user arguments;
the executable path is omitted. Tests and embedding hosts can still pass a
literal `[String]` directly to `Parser.parse`, which keeps parser behavior
deterministic. A program that parses on its own reads values by name from
`ParsedArguments`: `text(name)` and `number(name)` answer an optional,
`flag(name)` a `Bool`, and `texts(name)` and `numbers(name)` every value of a
repeating option. `CommandRouter.explain` prints what a failed parse calls
for, and `ArgumentError.isCleanExit` says whether it was `--help` or
`--version`.

## What the library provides

- required and optional positional arguments;
- long options, short options, aliases, `--name=value`, and short clusters;
- whole and decimal `Number` values, boolean flags, defaults, choices, and
  repeated options;
- `--` terminators and optional remainder collection, which runs from the
  first operand no argument takes to the end, as getopt's `+` does;
- independent subcommand schemas through `CommandRouter`;
- generated usage, help, version output, and structured `ArgumentError` values;
- `ParsableCommand` construct families for heterogeneous command routers;
- `@Derive(Parsable)` support for `String`, `Number`, `Int`, `Bool`, the
  optional `String?` and `Number?`, and repeated `[String]` or `[Number]`
  fields.

## Serious examples

The repository keeps examples substantial and runnable rather than filling the
tree with toy programs. [`examples/kira-audit`](examples/kira-audit/) is a
file-backed workspace inspector with `scan`, `check`, and `explain` commands.
[`examples/cargo-like`](examples/cargo-like/) is a cargo-shaped build
coordinator with `build`, `check`, and `metadata` commands, repeated features
and units, and a live five-line ANSI progress display.

Build and invoke the actual program:

```powershell
kira build --backend llvm examples/kira-audit
& .\examples\kira-audit\app\.kira-build\main.exe scan . --max-depth 1 --format json
& .\examples\kira-audit\app\.kira-build\main.exe check . --strict
& .\examples\kira-audit\app\.kira-build\main.exe explain package.kira --lines 4

kira build --backend llvm examples/cargo-like
& .\examples\cargo-like\app\.kira-build\main.exe build . --profile release --feature cli --feature progress --frames 6 --delay 120
& .\examples\cargo-like\app\.kira-build\main.exe metadata . --format json
```

Each example's `package.kira` points at this library with a local dependency,
so both are runnable directly from the repository without a registry package.
