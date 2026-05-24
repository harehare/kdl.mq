<h1 align="center">kdl.mq</h1>

A [KDL](https://kdl.dev/) parser implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- Nodes with positional arguments, key=value properties, and nested children blocks
- Double-quoted strings with escape sequences (`\n`, `\r`, `\t`, `\b`, `\f`, `\\`, `\"`, `\u{XXXX}`)
- Raw strings (`r"..."`, `r#"..."#`, `r##"..."##`, …)
- Decimal, hexadecimal (`0x`), octal (`0o`), and binary (`0b`) numbers
- Underscore separators in numbers (`1_000_000`, `0xFF_FF`)
- Boolean literals (`true`, `false`) and null (`null`)
- Type annotations on nodes (`(u8)node`, `(http)request`)
- Line comments (`//`) and block comments (`/* */`)
- Slashdash comments (`/-`) to comment out a node, value, property, or children block
- Line continuation with `\` before a newline

## Installation

Copy `kdl.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp kdl.mq ~/.local/mq/config/
```

## Usage

```sh
mq -L /path/to/modules -I raw \
  'import "kdl" | kdl::kdl_parse(.)' input.kdl
```

If you copied it to the mq built-in module directory:

```sh
mq -I raw 'import "kdl" | kdl::kdl_parse(.)' input.kdl
```

## API

### `kdl_parse(input)`

Parses a KDL string and returns a `{key: value}` dict.

| Input type | Output |
|---|---|
| String | Dict |

Conversion rules per node:

| Node shape | Value |
|---|---|
| Has children | Recursively converted dict |
| One arg, no props | The arg value directly |
| Multiple args, no props | Array of args |
| Props only | Props dict |
| Args + props | Props dict with args stored under `"args"` |
| Neither | `null` |

When multiple nodes share the same name the later one wins.

Raises an error if the input is not valid KDL.

## Example

Given `config.kdl`:

```kdl
// Server configuration
server {
    host "localhost"
    port 8080
    debug true

    limits {
        max-connections 0xFF   // hex literal → 255
        timeout 1_000          // underscore separator → 1000
    }

    tags "web" "api"           // multiple args → array
    /-disabled-feature true    // slashdash: ignored
}
```

```sh
mq -L . -I raw 'import "kdl" | kdl::kdl_parse(.)' config.kdl
# => {"server": {"host": "localhost", "port": 8080, "debug": true,
#      "limits": {"max-connections": 255, "timeout": 1000},
#      "tags": ["web", "api"]}}
```

```sh
mq -L . -I raw 'import "kdl" | kdl::kdl_parse() | get("server") | get("host")' config.kdl
# => "localhost"

mq -L . -I raw 'import "kdl" | kdl::kdl_parse() | get("server") | get("limits") | get("max-connections")' config.kdl
# => 255

mq -L . -I raw 'import "kdl" | kdl::kdl_parse() | get("server") | get("tags")' config.kdl
# => ["web", "api"]
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.5 or later.

## License

MIT
