# vodafon/jsonc

- Lightweight [JSON5](https://json5.org) pre-processor library for Go.
- Parses JSON5 input to JSON that Go understands. (Think of it as a superset to JSON.)
- Makes possible to have comment in any form of JSON data.
- Supported comments: single line `// comment` or multi line `/* comment */`.
- Supports trailing comma at the end of array or object, eg:
    - `[1,2,,]` => `[1,2]`
    - `{"x":1,,}` => `{"x":1}`
- Supports single quoted string.
- Supports object keys without quotes.
- Handles literal LF (linefeed) in string for splitting long lines.
- Supports explicit positive and hex number. `{"change": +10, "hex": 0xffff}`
- Supports decimal numbers with leading or trailing period. `{"leading": .5, "trailing": 2.}`
- Supports JSON string inside JSON string.
- Zero dependency (no vendor bloat).

---
### Example

This is [example](./examples/test.json5) of the JSON that you can parse with `vodafon/jsonc`:

```json5
/*start*/
//..
{
    // this is line comment
    a: [ // unquoted key
        'bb', // single quoted string
        "cc", // double quoted string
    /* multi line
     * comment
     */
        123, // number
        +10, // +ve number, equivalent to 10
        -20, // -ve number
        .25, // floating number, equivalent to 0.25
        5.,  // floating number, equivalent to 5.0
        0xabcDEF, // hex base16 number, equivalent to base10 counterpart: 11259375
        {
            123: 0xf, // even number as key?
            xx: [1, .1, 'xyz',], y: '2', // array inside object, inside array
        },
        "// not a comment",
        "/* also not a comment */",
        ['', "", true, false, null, 1, .5, 2., 0xf, // all sort of data types
            {key:'val'/*comment*/,}], // object inside array, inside array
        'single quoted',
    ],
    /*aa*/aa: ['AA', {in: ['a', "b", ],},],
    'd': { // single quoted key
        t: /*creepy comment*/true, 'f': false,
        a_b: 1, _1_z: 2, Ḁẟḕỻ: 'ɷɻɐỨẞṏḉ', // weird keys?
        "n": null /*multiple trailing commas?*/,,,
        /* 1 */
        /* 2 */
    },
    "e": 'it\'s "good", eh?', // double quoted key, single quoted value with escaped quote
    // json with comment inside json with comment, read that again:
    "g": "/*comment*/{\"i\" : 1, \"url\" : \"http://foo.bar\" }//comment",
    "h": "a new line after word 'literal'
this text is in a new line as there is literal EOL just above. \
but this one is continued in same line due to backslash escape",
    // 1.
    // 2.
}
//..
/*end*/
```

## Installation

```sh
go get -u github.com/vodafon/jsonc
```

## Usecase

You would ideally use this for organizing JSON configurations for humans to read and manage.
The JSON5 input is processed down into JSON which can be Unmarshal'ed by `encoding/json`.

For performance reasons you may also use [cached decoder](#cached-decoder) to have a cached copy of processed JSON output.

## Usage

Import and init library:
```go
import (
	"fmt"
	"github.com/vodafon/jsonc"
)

j := jsonc.New()
```

Strip and parse:
```go
json := []byte(`{
	// single line comment
	"a'b": "apple'ball",
	/* multi line
	   comment */
	"cat": [
		"dog",
		"// not a comment",
		"/* also not a comment */",
	],
	"longtext": "long text in
	multple lines",
}`)

var out map[string]interface{}

j.Unmarshal(json, &out)
fmt.Printf("%+v\n", out)
```

Strip comments/commas only:
```go
json := []byte(`{"some":"json",}`)
json = j.Strip(json)
```

Using strings instead of byte array:
```go
json := `{"json": "some
	text",// comment
	"array": ["a",]
}`
json = j.StripS(json)
```

Parsing from JSON file directly:
```go
var out map[string]interface{}

j.UnmarshalFile("./examples/test.json5", &out)
fmt.Printf("%+v\n", out)
```

### Cached Decoder

If you are weary of parsing same JSON5 source file over and over again, you can use cached decoder.
The source file is preprocessed and cached into output file with extension `.cached.json`.
It syncs the file `mtime` (aka modified time) from JSON5 source file to the cached JSON file to detect change.

The output file can then be consumed readily by `encoding/json`.
Leave that cached output untouched for machine and deal with source file only.
> (You can add `*.cached.json` to `.gitignore` if you so wish.)

As an example [examples/test.json5](./examples/test.json5) will be processed and cached into `examples/test.cached.json`.

Every change in source file `examples/test.json5` is reflected to the cached output on next call to `Decode()`
thus always maintaining the sync.

```go
import (
    "fmt"
    "github.com/vodafon/jsonc"
)

var dest map[string]interface{}
err := jsonc.NewCachedDecoder().Decode("./examples/test.json5", &dest);
if err != nil {
    fmt.Printf("%+v", err)
} else {
    fmt.Printf("%+v", dest)
}
```

> Run working [examples](./examples/main.go) with `go run examples/main.go`.

---
## License

> &copy; [MIT](./LICENSE) | 2022-2099, Jitendra Adhikari

---
### Credits

Inspired by and forked from [adhocore/jsonc](https://github.com/adhocore/jsonc).
