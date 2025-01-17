# Tapey Tape

Tapey Tape is an alternative to
[Plover](https://www.openstenoproject.org/plover/)’s built-in paper tape.
It provides a side-by-side view of strokes and translations as well as
some extra information such as bars showing hesitation time and
[clippy](https://github.com/tckmn/plover_clippy)-style
suggestions showing opportunities to save strokes.

```
      |   KP   A              | {-|}
   ++ |      H AO E  R        | here
    + |        A  E        S  | *here's  >HAO*ERS
    + |     WH A              | what
      |  T                    | it
 ++++ |      HRAO      B G   Z| look {^s}
   ++ |      HRAO EU   B G    | like
    + |  T P H                | in
    + |    P  RA       B G S  | practice
    + |  T P          P L     | {.}
```

## Viewing the paper tape

Tapey Tape does not have a graphical interface yet. Currently, it just
writes the paper tape to a text file named `tapey_tape.txt` in Plover’s
configuration directory:

- Windows: `C:\Users\<your username>\AppData\Local\plover\plover`
- macOS: `~/Library/Application Support/plover`
- Linux: `~/.config/plover`

To view the paper tape, simply open the file with a text editor.

To get a realtime feed, you can use the `tail -f` command in a terminal
if you’re on macOS or Linux:

- `tail -f ~/Library/Application\ Support/plover/tapey_tape.txt` (macOS)
- `tail -f ~/.config/plover/tapey_tape.txt` (Linux)

Alternatively, you can use `less +F`, which has more features.
For example, you can press `Ctrl+C` to pause the realtime feed
and use the arrow keys to scroll up and down, press `/` to search, etc.
You can then press `F` to resume the realtime feed.

If you’re on Windows, you can use this PowerShell command:

```powershell
Get-Content $env:LOCALAPPDATA\plover\plover\tapey_tape.txt -Wait
```

## Suggestions

Tapey Tape shows a suggestion whenever you write something using more
strokes than necessary:

```
|    P      E  R        | per
|  T P    O    R        | for
|    P H A      PB   S  | *performance  >PO*RPLS
```

When multiple translations are referred to, the number of translations
is shown before `>`:

```
|    P H A  E  R    T   | matter
|             F         | of
|  T P   A       B GT   | fact  3>PHAERBGT
```

Suggestions will be shown if you use the attach operator `{^}` to
suppress space when you could have used a prefix, suffix, or infix:

```
|  T P H  O E       T   | note
|  TK             L  S  | {^}
|    PW  AO      B G    | book  2>PWAO*BG
```

Suggestions will also be shown for stroke-inefficient capitalization:

```
|   KP   A              | {-|}
|        A   U  PB  T   | aunt  2>A*UPBT
|  TKPW RA  EU       S  | grace
|   KP   A            D | {*-|}  2>TKPWRA*EUS
```

Here “Aunt” is capitalized with the “capitalize next word” command
`{-|}`, and “Grace” with the “capitalize last word” command `{*-|}`.
In both cases you could have used the starred outline for the
capitalized version of the word.

Contiguous fingerspelling strokes are treated as a group. For example,
[fingerspelling “kvetch”](https://www.youtube.com/watch?v=DIfjztBuBc8)
will only trigger suggestions for “kvetch”, not sub-words like “vet”,
“vetch”, “et”, “etc”, and “etch”.

```
|      H    E           | he
|             F      S  | was  2>HEFS EFS
|   K      *            | {>}{&k}
| S     R  *            | {>}{&v}
|          *E           | {>}{&e}
|  T       *            | {>}{&t}
|   K   R  *            | {>}{&c}
|      H   *            | {>}{&h}  >KW*EFP
|                  G    | {^ing}
|    PW                 | about
|                   T   | the  2>PW-T
|    P  RAO EU       S  | price
|  T P          P L     | {.}
```

## Installation

To install this plugin, right click the Plover icon, go to Tools →
Plugins Manager, find `plover-tapey-tape`, and click “Install/Update”.
When it finishes installing, restart Plover, go to Configure → Plugins,
and check the box next to `plover-tapey-tape` to activate the plugin.

(If you don’t see the plugins manager, it may not be installed
because you’re using an older version of Plover. Please see the
[Plugins](https://github.com/openstenoproject/plover/wiki/Plugins)
page on the Plover Wiki for instructions.)

### Fork Installation

```sh
git clone git@github.com:paulfioravanti/plover-tapey-tape.git
cd plover-tapey-tape
plover -s plover_plugins install .
```

`plover` should be a reference/symbolic link to your installed version of
Plover. See the
[Command Line Reference](https://plover.readthedocs.io/en/latest/cli_reference.html)
for details.

This fork enables changes in Tapey Tape config (`tapey_tape.json`) to be re-read
in by just pressing the Plover UI reconnect button, rather than restarting
Plover itself.

It uses some of the steno engine's stated private APIs to achieve the use case,
since there seems to currently be no other way to have plugin extension config
files be re-read in without restarting Plover.

Looking at the steno engine code for
[starting/stopping extensions](https://github.com/openstenoproject/plover/blob/master/plover/engine.py#L287),
it doesn't *seem* like there would be any issues using these private APIs, but
since that is currently unclear, this code should just be considered
experimental for now.

If Plover removes or changes the private APIs that this fork relies on, it
should just degrade to Tapey Tape's default behavior.

## Configuration

You can customize the behaviour of this plugin by creating a
[JSON](https://www.json.org/json-en.html) configuration file named
`tapey_tape.json` in Plover’s configuration directory (see above).
(If you don’t create the file or don’t specify certain options,
the default values will be used.) The available options are:

- `"output_file"`: an absolute filepath specifying the file to
  output to. `~` is expanded to the home directory. Defaults to
  `tapey_tape.txt` in Plover’s configuration directory.
- `"bar_character"`: the character used to draw the hesitation bar.
  Defaults to `"+"`.
- `"bar_max_width"`: the maximum number of characters drawn.
  Defaults to `5`.
- `"bar_time_unit"`: the number of seconds each character represents.
  Defaults to `0.2`.
- `"bar_threshold"`: a constant number of seconds to subtract from the
  hesitation time of each stroke. (In other words, how long to wait
  before the clock starts ticking.) This can be used to hide the bars
  for fast strokes so that the bars for the slow strokes stand out more
  visually. Defaults to `0`.
- `"bar_alignment"`: either `"left"` or `"right"` indicating whether the
  bar should be left-aligned or right-aligned. Defaults to `"right"`.
- `"line_format"`: a string template specifying how each line in the
  output should be formatted. Special codes beginning with `%` are
  transformed into different items:

| Code | Item           | Example                                 |
|:-----|:---------------|:----------------------------------------|
| `%t` | date and time  | `2020-02-02 12:34:56.789`               |
| `%b` | hesitation bar | `+++++`                                 |
| `%S` | steno          | `...K.W.R....U.RPB......` (`.` = space) |
| `%r` | raw steno      | `KWRURPB`                               |
| `%D` | definition     | `yes{,}your Honor`                      |
| `%T` | translation    | `Yes, your Honor`                       |
| `%s` | suggestions    | `2>KWRURPB`                             |
| `%%` | an actual `%`  | `%`                                     |

The default format is `%b |%S| %D  %s`:

```
    %b |          %S           | %D      %s

  ++++ | ST        E   PB      | sten
    ++ | S K W R O             | *steno  >STOEUPB STO*EUPB
```

Here’s a comparison between `%D` and `%T`:

```
                          %D        %T

|    P H       R        | Mr.{-|}   Mr.
|    PW R O  U  PB      | brown     Brown
|      H  O EU    L   DZ| /         HOEULDZ
|          *            | *         *
|      H  O E     L   DZ| holds     holds
|        A  EU          | a         a
|    P     *    P       | {&P}      P
|      H   *            | {>}{&h}   h
|  TK      *    P       | {&D}      D
|  TK       E      G    | degree    degree
|  T P H                | in        in
|  T P H AO* U R        | {neuro^}  neuro
| S K   RAO EU  PB   S  | science   science
|  T P          P L     | {.}       .
```

In short, `%D` is the *definition* in your dictionary, including
commands like `{-|}` and `{.}`. `%T` is the *translation* by Plover,
or the actual characters that are output. (If a stroke is undefined,
`%D` is displayed as `/`.)

You can additionally specify the *minimum width* of an item by inserting
a number between the `%` and the letter code. For example, `%10r` makes
the raw steno at least 10 characters wide by padding it with spaces.
This can be used to generate a tabular output:

```json
{
    "line_format": "%10r -> %T"
}
```

```
-T         -> The
KWEUBG     -> quick
PWROUPB    -> brown
TPOBGS     -> fox
SKWRUFRPZ  -> jumps
OEFR       -> over
-T         -> the
HRAEZ      -> lazy
TKOG       -> dog
TP-PL      -> .
```

Here’s an example configuration that stretches the hesitation bar to
twice its default width:

```json
{
    "bar_max_width": 10,
    "bar_time_unit": 0.1
}
```

Note that if you edit `tapey_tape.json` while Plover is running, you’ll
have to restart Plover for the new settings to take effect.

## Changelog

- v0.0.6: Improved suggestions.
- v0.0.5: Fixed an encoding issue.
- v0.0.4: Added the `output_file` and `bar_threshold` options.
- v0.0.3: Made the output formatting configurable.
- v0.0.2: Added different styles of displaying translations.
- v0.0.1: Initial version.

## Related projects

- [plover-tapey-tape.nvim](https://github.com/derekthecool/plover-tapey-tape.nvim):
  a Neovim plugin that features a toggle view of Tapey Tape,
  suggestion notifications, and status line integration.

## Acknowledgements

The name of this plugin is a tribute to
[Typey Type](https://didoesdigital.com/typey-type/),
a free resource for learning steno.

This plugin is heavily inspired by
[plover-clippy](https://github.com/tckmn/plover_clippy).
