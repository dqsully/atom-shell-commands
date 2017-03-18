# DQ's Shell Commands package

Originally adapted from skywind3000's [atom-shell-commands](https://github.com/skywind3000/atom-shell-commands) package

Customize shell commands for atom. Similar to 'Run Commands' in NotePad++, 'User Tool' in EditPlus/UltraEdit, 'External Tool' in GEdit and 'Shell Commands' in TextMate.

Preface
-------
This package enables you to setup shell commands as atom commands. Shell output will be captured and displayed in a panel at the bottom of the screen. If you create a command named 'gcc-build', you will find 'dqs-shell-commands:gcc-build' in the atom command palette, of which you can define a keymap to setup a shortcut for it.

(From skywind3000): *"I was a window user and switched to Atom from EditPlus/Notepad++. Previously I wrote a script in EditPlus to compile-run my C/C++ programs and execute Makefiles with customizable compile flags and run options. Atom has numerous community packages and I have tried some. Most of them are really great in certain extent but still cannot fully satisfy my need. Therefore I decided to reference their packages and make my own.""*

Feature
-------

- Customize shell command to invoke compiler or other tools with active document/project.
- Customize arguments, working directory and system environment of the command.
- Automatically create atom command for each shell command.
- Keymap config allows you to make shortcut to each command.
- Shell output (stdout/stderr) can be captured in the bottom panel.
- Click the filename in the output panel will open it.
- Regular expression to match filename and line number in the error output.
- Hotkeys to navigate errors one by one just like quickfix in vim.
- Fast and lightweight, loading time is less than 2 milliseconds (TimeCop).
- Create different commands for each project if you like.

Installation
------------
    apm install dqs-shell-commands

Setup
-----

Configure commands on your config file (`File->Config...`): (**NOTE:** This is finicky and may not work properly)
```cson
"dqs-shell-commands":
  commands: [
    {
      name: "compile"
      command: "d:/dev/mingw/bin/gcc"
      arguments: [
        "{FileName}"
        "-o"
        "{FileNameNoExt}.exe"
      ]
      options:
        cwd: "{FileDir}"
        keymap: 'ctrl-2'
    }
  ]
```
Or also in your project root `.shell-commands.cson`:
```cson
commands: [
  {
    name: "compile"
    command: "d:/dev/mingw/bin/gcc"
    arguments: [
      "{FileName}"
      "-o"
      "{FileNameNoExt}.exe"
    ]
    options:
      cwd: "{FileDir}"
      keymap: 'ctrl-2'
  }
]
```


This will create the atom command "dqs-shell-commands:compile" that you can now launch from the command palette or use the binding keymap 'ctrl-2'. It also generates an entry in the `DQ's Shell Commands` menu under `Packages`. Command entries have the following properties:

| Field | Requred | Description |
|-------|----|---------|
| `name` | **[required]** | The name of the target. Viewed in the menu |
| `command` | **[required]** | The executable command (if not in terminal mode, can be a relative path to a file) |
| `aurguments` | *[optional]* | An array of arguments for the command |
| `selector` | *[optional]* | atom selector, default is 'atom-workspace' |
| `matchs` | *[optional]* | regular expression to match file name in output |
| `options` | *[optional]* | additional options to config dir, environment, keymap etc |

The `command`, `arguments` and `options` values accepts the variables below:

| Macro            | Description |
|------------------|-------------|
| {FilePath}       | File name of current document with full path. |
| {FileName}       | File name of current document without path. |
| {FileDir}        | Full path of current document without the file name |
| {FileExt}        | File extension of current document |
| {FileNameNoExt}  | File name of current document without path and extension |
| {ProjectDir}     | Current project directory |
| {ProjectRel}     | File name relativize to current project directory |
| {CurRow}         | Current row(line number) where the cursor is located |
| {CurCol}         | Current column index where the cursor is located |
| {CurSelected}    | Selected text |
| {CurLineText}    | Current line text under cursor |
| {CurWord}        | Current word under cursor |

You can setup as many commands as you wish, for example: build with your project makefile, compile a single source file directly, compile your latex, run grep in current directory, pass the current word under the cursor to man or a dictionary, call svn diff with current file and redirect the output to the bottom panel.


The `options` field can contain the following properties:

| Options | Required | Description |
|---------|------|-------|
| cwd | *[optional]* | Working directory of the command |
| save | *[optional]* | True or false(default) to save the current file before execute |
| silent | *[optional]* | True or false(default); true will not open the message panel when this command is run. |
| keymap | *[optional]* | A keymap string as defined by Atom. Pressing this key combination will trigger the target. Examples: ctrl-alt-k or cmd-shift-U. |
| env | *[optional]* | Key/value based system environment setting |
| sound | *[optional]* | File path for a wav/mp3/ogg file which will be played to remind you that the job is finished |
| file | *[optional]* | File path (relative or absolute) to copy the output of the command to |
| mode | *[optional]* | Mode for starting the process |
| context | *[optional]* | String of flags indicating which circumstances the command can be run under |

The `options.mode` field string can be any of the following:

| Type | Description |
| `silent` | Same as setting `silent` option to true |
| `terminal` | Opens the command in a separate window |
| `window` | Same as `terminal` |
| `open` | Same as `terminal` |

The `options.context` field string can contain any of the following flags:

| Flag | Description |
|------|-------------|
| `f` | Requires an open file to be focused by the user |
| `e` | Requires an editor to be focused by the user |
| `p` | Requires the focused editor to have a cursor |

*If no flags are specified, the command can always be called.*

Examples
--------

Compiling with command 'gcc'

![](https://github.com/skywind3000/dqs-shell-commands/blob/master/images/command_compile.png?raw=true)

Running with command 'execute'

![](https://github.com/skywind3000/dqs-shell-commands/blob/master/images/command_execute.png?raw=true)

command 'execute' config:
```cson
{
  name: "execute"
  command: "{FileNameNoExt}"
  options:
    cwd: "{FileDir}"
}
```

Running in a new window

![](https://github.com/skywind3000/dqs-shell-commands/blob/master/images/command_runinwindow.png?raw=true)

command 'runinwindow' config:
```cson
{
  name: "runinwindow"
  command: "cmd"
  arguments: [
    "/C"
    "start"
    "d:/software/atom/launch.cmd"
    "{FileNameNoExt}"
  ]
  options:
    cwd: "{FileDir}"
}
```

Lookup word in manual

Current word under cursor can be passed to the external executables or scripts, you can use it to look up current word in dictionary or manual with a single hotkey stroke:

```cson
{
  name: "man"
  command: "/usr/bin/man"
  arguments: [
      "{CurWord}"
  ]
  options:
    cwd: "{FileDir}"
    keymap: "ctrl-alt-m"
}
```

Error Matching
--------------

Error matching lets you specify one or more regular expressions which are matched the output of your build command, to open the matched file, row, and column of the reported error. For example:

```
ztest_hello.cpp: In function 'int main()':
ztest_hello.cpp:7:10: error: expected initializer before 'int'
```

Would be matched with the expression: `^(?<file>[\\/0-9a-zA-Z\\._\\\\:]+):(?<line>\\d+):(?<col>\\d+):`. After the build has failed, if you were to click on the error output line, the file would be opened and the cursor would be placed at row 7, column 10.

Note the syntax for match groups. This is from the [XRegExp](http://xregexp.com/) package
and has the syntax for named groups: `(?<name> RE )` where `name` would be the name of the group
matched by the regular expression `RE`.

The following named groups can be matched from the output:
  * `file` - **[required]** the file to open. May be relative `cwd` or absolute. `(?<file> RE)`.
  * `line` - *[optional]* the line the error resides on. `(?<line> RE)`.
  * `col` - *[optional]* the column the error resides on. `(?<col> RE)`.

Since the regular expressions are written in the user config file as .cson format, backslashes must be escaped.

The `file` should be an absolute path, or relative the `cwd` specified. If no `cwd` has been specified, default value '/' will be used.

Example user config file which is using error matching:

```cson
*:
  "dqs-shell-commands":
    commands: [
      {
        name: "compile"
        command: "d:/dev/mingw/bin/gcc"
        arguments: [
          "{FileName}"
          "-o"
          "{FileNameNoExt}.exe"
        ]
        matchs: [
          "^(?<file>[\\/0-9a-zA-Z\\._\\\\:]+):(?<line>\\d+):(?<col>\\d+):"
          "^(?<file>[\\/0-9a-zA-Z\\._\\\\:]+):(?<line>\\d+):"
          "^(?<file>[\\/0-9a-zA-Z\\._\\\\:]+)\\s*\\((?<line>\\d+)\\)\\s*:*:"
        ]
        options:
          cwd: "{FileDir}"
          keymap: 'ctrl-2'
          save: true
      }
    ]
```

This will match the `file`, `line` and `col` in both clang/gcc or msvc error output.

Quickfix
--------

dqs-shell-commands has a special design in the output panel to speedup the edit-compile-edit cycle.  This is inspired by the quickfix mode in vim. The idea is to save the error messages from the compiler and use builtin commands to jump to the errors one by one.  You can examine each problem and fix it, without having to remember all the error messages.

| Command | Description |
|---------|-------|
| dqs-shell-commands-config:error-first | go to the first error |
| dqs-shell-commands-config:error-last | go to the last error |
| dqs-shell-commands-config:error-next | go to the next error |
| dqs-shell-commands-config:error-prev | go to the previous error |

To avoid hotkey conflict to other packages, dqs-shell-commands has no predefined keymaps, but you can define your own in `File->Keymap...`. You can also trigger these actions from `Atom Shell Commands` menu under `Packages`, or from command palette directly.

An example user keymap file:
```cson
'atom-workspace':
    'F9' : 'dqs-shell-commands-config:error-next'
    'F10': 'dqs-shell-commands-config:error-prev'
```

Now you can have your F9/F10 to navigate errors without leaving your hand from keyboard to mouse/touch pad.

Misc
----
dqs-shell-commands has been tested in Windows, and previously in macOS and Ubuntu. You can use 'open' in macOS or '/usr/bin/gnome-terminal' in Ubuntu to open a new window and execute your command.

As executing program in a new terminal window correctly is a little tricky thing, I create a script to let you open a new terminal window to execute your program in both Windows, Linux (ubuntu), Cygwin and Mac OS X, you can try it from: https://github.com/skywind3000/terminal.

TO-DO
-----
- Commands could be setup in a new GUI window, not only in user config.



Reference
---------

- [https://github.com/DPISA/atom-user-commands](https://github.com/DPISA/atom-user-commands "atom-user-commands") (based on it).
- [https://github.com/joefitzgerald/go-plus](https://github.com/joefitzgerald/go-plus) (referenced).
- [https://github.com/noseglid/atom-build](https://github.com/noseglid/atom-build) (referenced).
- [https://github.com/ksxatompackages/cmd-exec](https://github.com/ksxatompackages/cmd-exec) (referenced).
