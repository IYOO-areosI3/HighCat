hcat – Syntax Highlighter & Light Linter

hcat is a terminal‑based syntax highlighter that turns your code into a colourful, easy‑to‑read output. It also includes a simple linter to catch common mistakes.

· Written in Python, powered by rich
· Supports C, C++, Python, JavaScript, Java, CSS
· Fully customisable via a TOML configuration file
· Command‑line flags for quick custom words and styles
· Built‑in linter with helpful messages

---

1. Installation

Requirements

· Python 3.7 or higher (Python 3.11+ recommended for built‑in TOML support)
· rich library (installed via pip)
· If your Python is older than 3.11 and you want to use the -config option, also install tomli

Installing Python and pip

On Linux (including Termux), install Python 3 and pip:

```bash
pkg install python       # Termux
# or on Debian/Ubuntu:
sudo apt install python3 python3-pip
```

Installing rich and tomli

```bash
pip install rich
pip install tomli   # only if Python < 3.11
```

Important: hcat will not work without rich. Make sure it’s installed.

---

Quick Installation on Termux (for Android)

If you’re using Termux, you can set up hcat in a few steps:

```bash
# 1. Install Python
pkg install python

# 2. Install required libraries
pip install rich
pip install tomli   # optional (for older Python)

# 3. Clone the repository
git clone https://github.com/IYOO-areosI3/HighCat.git

# 4. Enter the folder
cd HighCat

# 5. Make the script executable
chmod +x hcat

# 6. Run it
./hcat
# or
python hcat
```

If you encounter errors, check that the HighCat folder exists with ls. If not, try cloning again.

---

2. Quick Start

Highlight a file by providing the language extension and the filename (without the dot), or just the full filename:

```bash
hcat cpp main        # highlights main.cpp (C++)
hcat py my_script    # highlights my_script.py
hcat main.c          # auto‑detects C language
```

The terminal will show your code with syntax‑coloured keywords, strings, comments, numbers, and functions.

---

3. Command‑Line Usage

```
hcat [-config <file>] [-address-error] [-self-insert WORD ...] (ext) filename
```

File Arguments

You can specify the file in two ways:

· Two arguments – extension and name (without the dot)
    hcat cpp my_code → opens my_code.cpp, language = cpp
· One argument – filename with extension
    hcat my_code.js → opens my_code.js, language = js

If the extension is unknown, the highlighter will still try its best with generic keywords.

Flags

Flag Description
-config <file> Load a TOML configuration file (see Configuration).
-address-error Run the built‑in linter instead of highlighting (see Linting).
-self-insert <word> Add a custom word to highlight. Must be followed by at least one style option.
-color <name> Named colour (e.g. "blue", "red").
-color-hex <hex> Hex colour (e.g. "#ffaa00").
-color-rgb <r,g,b> RGB colour (e.g. "255,100,50").
-underline [true/false] Enable/disable underline (default true if no value given).
-overwrite [true/false] Override the built‑in colour for this keyword with your custom style.

Note: Each -self-insert block applies only to the word that came right before it. To add multiple custom words, repeat the whole -self-insert block.

---

Examples with Command‑Line Flags

```bash
# Highlight "TODO" in orange with underline
hcat -self-insert "TODO" -color "orange" -underline true cpp main

# Use a hex colour
hcat -self-insert "BUG" -color-hex "#ff0000" -underline false cpp main

# Use an RGB colour and overwrite any built‑in keyword
hcat -self-insert "class" -color-rgb "100,200,50" -overwrite true cpp main
```

If you don't provide any -self-insert flags, hcat will highlight print in red by default (just as a demo). You can override this behaviour by loading a config file or using your own custom words.

---

4. Configuration File (TOML)

With -config you can load all your custom words, scope overrides, and custom regex scopes from a single file.
The file must be a valid TOML document and must start with an [id] section to be recognised.

Basic structure

```toml
[id]
config = true   # required – must be true

# Your settings go here
```

a) config_keyword – custom words with style

Define words that you want highlighted, and how they should look.

```toml
[config_keyword]

[config_keyword.burger]          # normal word
enabled = true
color = "purple"
underline = true
overwrite = false                # if true, replaces built‑in keyword's colour

[config_keyword."#include"]      # word containing symbols (must be quoted)
enabled = true
color = "#ffaa00"                # hex colour
underline = false
overwrite = true                 # override the default preprocessor colour
```

Rules:

· The word can contain any character; if it has symbols, wrap it in double quotes.
· enabled must be true to activate the word.
· color can be a named colour (like "red"), a hex colour ("#ff0000"), or an RGB string ("rgb(255,0,0)").
· underline is optional (defaults to false).
· overwrite is optional (defaults to false). When true, the built‑in keyword with the same name will not be highlighted with its default colour; only your custom style will show.

b) config_scope – override built‑in scopes

You can change the colour and underline of the four built‑in scopes: string, comment, number, function.

```toml
[config_scope.string]
color = "#22dd88"
underline = false

[config_scope.comment]
color = "#888888"
underline = false

[config_scope.number]
color = "#b5cea8"

[config_scope.function]
color = "#dcdcaa"
underline = false   # turn off the default underline for functions
```

If you don’t set a scope, its default style will be used.

c) config_customScope – user‑defined regex patterns

Create your own highlighting patterns with a $any placeholder.
$any matches any sequence of characters (non‑greedy). The rest of the pattern is matched literally.

```toml
[config_customScope]
"<$any>" = { color = "cyan", underline = false }
"#define $any" = { color = "yellow" }
"ERROR: $any" = { color = "red", underline = true }
```

· <$any> will match <iostream>, <vector>, <any_text_here>.
· #define $any will match #define PI 3.14, #define MAX_SIZE, etc.
· ERROR: $any will match ERROR: something went wrong and underline it in red.

Custom scopes are applied after strings and comments, so they won’t interfere with those. They are also applied before your custom words, allowing custom words to override them if needed.

---

Full Example Configuration File

```toml
[id]
config = true

# ── Override scope colours ──
[config_scope.string]
color = "#ce9178"

[config_scope.comment]
color = "#6a9955"

[config_scope.function]
color = "#dcdcaa"
underline = false     # no underline for functions

# ── Custom keywords ──
[config_keyword]

[config_keyword."#include"]
enabled = true
color = "orange"
overwrite = true
underline = false

[config_keyword.TODO]
enabled = true
color = "red"
underline = true

# ── Custom regex scopes ──
[config_customScope]
"<$any>" = { color = "cyan" }
"FIXME: $any" = { color = "yellow", underline = true }
```

Save it as style.toml and run:

```bash
hcat -config style.toml cpp my_code
```

---

5. Supported Languages

Extension Language
cpp, cc, cxx C++
c C
py Python
js JavaScript
java Java
css CSS

The highlighter knows the keywords, comment syntax, string styles, and number formats for each language.

---

6. Linting (-address-error)

When you run hcat with the -address-error flag, it performs a simple style check on your code. It will report:

· Tab characters used (suggest spaces)
· Trailing whitespace at the end of lines
· Lines longer than 120 characters
· Possible use of = instead of == in comparisons
· Missing spaces around operators
· Language‑specific issues:
  · Python: bare except, camelCase variable names, leftover print() statements
  · C/C++: using namespace std; globally, use of NULL
  · JavaScript: var instead of let/const, missing semicolons, console.log statements
  · Java: System.out.print calls
  · CSS: duplicate properties, !important usage

Output example:

```
Code Quality Report – my_code.py
Language: py

╭─────────┬───────────────────────────────────────────┬───────────────────────────────────────╮
│  Line   │  Issue                                    │  How to fix                           │
├─────────┼───────────────────────────────────────────┼───────────────────────────────────────┤
│    5    │  Bare 'except:' – catches too much.      │  Specify exception type (e.g.,        │
│         │                                           │  except ValueError).                  │
│   12    │  camelCase variable name – prefer         │  Use snake_case naming.               │
│         │  snake_case.                              │                                       │
╰─────────┴───────────────────────────────────────────┴───────────────────────────────────────╯

bad  : 2 line(s) with issues
good : 38 clean line(s)
```

---

7. Troubleshooting

No module named 'rich'
Install it: pip install rich

Invalid TOML or Cannot overwrite a value
Make sure you don’t define a key both as a plain value and as a sub‑table. For custom keywords, always use the [config_keyword."word"] format, not word = true directly under [config_keyword].

Unknown scope '...'
Only string, comment, number, function are valid scope names. Check your spelling.

Custom scopes don’t highlight
They are not applied inside strings or comments. Also, check that your pattern doesn’t contain regex characters that need escaping – $any is the only part treated as a wildcard, the rest is escaped automatically.

Colour for a symbol word not showing
Words with symbols (like #include) must be quoted in the TOML file, e.g., [config_keyword."#include"]. Also consider setting overwrite = true if it’s a built‑in keyword.

tomllib missing
If you’re on Python older than 3.11, install tomli with pip install tomli.

File not found
Make sure you’re in the correct directory and the filename is correct. If you use the ext filename format, the script looks for filename.ext.

---

8. Enjoy!

If you find any issues or have suggestions, feel free to open an issue on the repository. Happy coding with colour!
