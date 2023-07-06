# clparser 

clparser is a simple, but flexible command line parser for NodeJS



## Get It

```
npm install -g @toptensoftware/clparser
```



## Usage

Require it:

```
// Import clparser
const clparser = require('@toptensoftware/clparser');
```

Create a parser with a command line specification and a set of options:

```
// Command line spec
const clparser_spec = [
    {
        name: "--help",
        help: "Show this help",
    },
    {
        name: "<filename>",
        help: "The file to process",
    },
    {
        name: "-c|--count:<n>",
        help: "The number of times to process the file",
    }
];

// Options
const clparser_options = {
    usagePrefix: "node myprogram.js",
    packageDir: __dirname,
};

// Create a parser
const parser = clparser.parser(clparser_spec, clparser_options);
```

Parse the command line

```
// Parse command line
let cl = parser.parse(process.argv.slice(2));
```

Optionally automatically handle `--help` and `--version` (you still need to specify them
in the spec above).  `handle_help` won't return if either option is specified by user.

```
// Handle --help and --version
parser.handle_help(cl);
```


Check that all required args were specified and nothing left unprocessed:

```
// Check args
parser.check(cl);
```

Use the parsed results.  The parse command returns an object with
keys for each of the options:

```
console.log(`Processing ${cl.filename} ${cl.count} times...`);
```



## Options

The options parameter supports the followin settings:

* `usagePrefix` - the name of your program as displayed in the usage string when showing help
* `packageDir` - a directory containing a package.json file from which name, version, license and 
  copyright information is extracted



## Command Line Specs

A "command line spec" is an array of objects, each specifying one command line option.

Options can be of the following types:

* positional - read from the command line based on position
* switch - an on/off switch eg: `--switch`
* named - a named value eg: `--name:value`
* command - a sub command eg: `push`

Each option supports the following settings:


### `name`

The name entry specifies the name and type of command line argument:

* `<name>` - a positional argument
* `-N` - a single letter switch
* `--name` - a named switch
* `-N:<value>` - a single letter named value
* `--name:<value>` - a named value
* `name` - a command name (see below for more on commands)

Multiple names for named options and switches can be specified by separating with a `|` delimiter. eg: `--name|-n`


The `<value>` part of the name is displayed to the user and can be anything however the following
special cases make setting up some options easier:

* `<n>` - an integer value
* `[a|b|c]` - an enumerated type with possible values `a`, `b` or `c`

To handle other values types, see `parse` below.


### `default`

The default value for an option if not specified by the user.


### `defaultWhenPresent`

A default value for a named option if supplied by the user without a value.


### `help`

A help string describing this option.

The help string can contain `\n` line breaks, but will also be automatically wrapped if too long.

Options without a help string aren't included in the
generated help or usage strings.


### `parse`

An value parsing function.  

The function is passed the value from the command line and should
either return a parsed value, or raise an exception.

Several built-in parsers are included:

* `clparser.parse_bool` - parses a boolean value from `yes`, `no`, *`true`, `false`, `1` or `0`
* `clparser.parse_integer(min, max)` - parses an integer value with optional min and max values
* `clparser.parse_enum(array)` - parses an enum value from and array of allowed values

For example, to only accept `.txt` files:

```
{
    name: "<file>",
    parse: function(arg) {
        if (!arg.endsWith('.txt'))
            throw new Error("Must be a text file");
        return arg;
    }
}
```

### `multiValue`

If true, multiple values can be supplied by the user and they will be returned as an array.

If the `default` entry is an array, `multiValue` is assumed true.


### `key`

The name of the option in the returned object.  If not specified, a default name is synthesized by
camelCasing the option's `name`.


### `terminal`

If true, upon processing this argument, stop processing additional items.  Unprocessed items will
be returned as an array in the `$tail` key of the returned object.



## Commands

A command is specified in the command spec as a name with no `-` or `--` prefix and no surrounding `<>` markers.

When an argument matching a command name is found, the parser returns immediately with the following
keys added to the returned object:

* `$command` - the name of the found command
* `$tail` - the rest of the command line after the command

The tail can then be passed to a different parser for that command.

eg:

```
// Command line spec
[
    { 
        name: "<file>",
    },
    {
        name: "push",       // pull command
    },
    {
        name: "pull",       // push command
    },
]
```

when passed these args:

```
myfile.txt push something else
```

would result in:

```
{
    file: "myfile.txt"
    $command: "push"
    $tail: [ "something", "else" ]
}
```



## Parser Behaviour

The following describes various behaviours of the parser


### Combined Short Names

Combined short names are automatically expanded.  

eg:

`-abc:xyz` 

is equivalent to:

`-a -b -c:xyz`


### Negative Switch Values

Switch values can be negated using `--switch:no`, `--switch:false` or `--switch:0`.


### Value Separators

Values can be specified delimited by either `:`, `=` or a space.

ie: these are all equivalent:

```
--option:value
--option=value
--option value
```

Note: the third variation isn't supported for options that have a `defaultIfPresent` option
as they automatically take that value if the user specified the option without a value.


## Case Sensitivity

Named options and command names are case sensitive.

The enum and bool value parsers are case insensitive.


### End Marker

The parser terminates if it sees `--` in the supplied args and anything after it is returned
as an array in the `$tail` field of the returned object.

