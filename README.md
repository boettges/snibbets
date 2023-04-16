

[![RubyGems.org](https://img.shields.io/gem/v/snibbets)](https://rubygems.org/gems/snibbets)
<!-- [![GitHub Actions](https://github.com/ttscoff/snibbets/actions/workflows/check.yml/badge.svg)](https://github.com/ttscoff/snibbets/actions/workflows/check.yml) -->

A tool for accessing code snippets contained in a folder of plain text Markdown files.

Snibbets allows me to keep code snippets in raw files, not relying on a dedicated code snippet app. I can collect and edit my snippets using a text editor, nvALT (nvUltra), or simply by saving snippets from my clipboard to a text file using *NIX redirection on the command line. I can add descriptive names and extended descriptions/notes to code snippets using standard Markdown.

What Snibbets does is simply allow a quick search for a specific snippet that I can either output to the terminal, pipe to my clipboard, or access via LaunchBar (via the included LaunchBar Action). It's basically a wrapper for `find` and `grep` with the ability to separate code blocks from other text in my Markdown files.


![Snibbets in action](https://cdn3.brettterpstra.com/uploads/2023/04/snibbets2.gif)



## Collecting Snippets

Snibbets is designed to work with a folder containing Markdown files. Each Markdown file can have somewhat flexible formatting, as long as there's at least one code block (either indented by 4 spaces/1 tab or fenced with backticks).

I recommend using filenames with multiple extensions (ending with your markdown extension), primarily to define the syntax for a snippet. For example, a css snippet would be `*.css.md`, a ruby snippet would be `*.rb.md`. This can aid in searching and makes it easy to script things like adding language tags automatically.

The name of the file should be the description of the snippet, at least in the case where there's only one snippet in the document. Call it 'javascript url parser.js.md' or similar. If I got the snippet from StackOverflow, I give it a name based on the question I searched to find it. Be descriptive.

You can combine multiple snippets in a file, though. For example, I have a file called 'Ruby hash snippets.rb.md'. That file contains an array of useful snippets, and each one has a descriptive title in an h3 header above it. Those (ATX) headers are used to split the file, and when you search from the command line, you'll get a menu of all of the snippets in the selected file. (And if you have [fzf](https://github.com/junegunn/fzf) or [gum](https://github.com/charmbracelet/gum) installed, you can quickly filter through with fuzzy searching and find exactly what you need.)

If a file contains multiple snippets, they should be separated by ATX-style headers (one or more `#`) describing the snippets. Additional  descriptions can be included outside of the code block. For example:

A file titled `unix find.bash.md`:

    title: Unix find commands
    tags: bash, shell, unix

    ### Find by name and execute command

        find . -name "rc.conf" -exec chmod o r '{}' \;

    ### Find by name and grep contents

        find . -name "*.php" -exec grep -H googleapis '{}' \;

    ### Find by age range to CSV

    Finds files older than 18months and newer than 24 months, cats the output to a CSV in the format `/some/path/somewhere, size in bytes, Access Time, Modified Time`

        find /dir/dir -type f -mtime +540 -mtime -720 -printf \"%p\",\"%s\",\"%AD\",|"%TD\"\\n > /dir/dir/output.csv

You can include MultiMarkdown metadata in your snippets, either in a YAML block or just at the top of the file with raw key/value pairs. I mostly use this for adding tags, which are then synced to macOS tags when I save. It makes it easy to search for snippets in [nvUltra](https://nvultra.com/), and also allows you to do searches like `snibbets tag:javascript url parser` in Snibbets.

## CLI

### Dependencies

Snibbets requires Ruby 3.0+. On recent versions of macOS, this is not included by default. You can install it via the Command Line Tools from Apple. On macOS and most other systems, you can use something like [Homebrew], [rbenv], [rvm], or [asdf] to install Ruby 3.

If available, menus are generated by [fzf] or [gum]. If neither are available, a basic Readline menu system will be displayed, so neither are required, just nice to have as they provide fuzzy filtering, scrolling, and type-ahead completion.

[homebrew]: https://brew.sh/ "Homebrew???The Missing Package Manager for macOS (or Linux)"
[rbenv]: https://github.com/rbenv/rbenv "rbenv/rbenv:Manage your app's Ruby environment"
[rvm]: https://rvm.io/ "Ruby Version Manager"
[asdf]: https://asdf-vm.com/ "ASDF environment manager"
[fzf]: https://github.com/junegunn/fzf "junegunn/fzf:A command-line fuzzy finder"
[gum]: https://github.com/charmbracelet/gum "charmbracelet/gum:A tool for glamorous shell scripts ????"

### Installation

To install Snibbets:

    gem install snibbets

If you're using the system Ruby, you may need to use `sudo gem install snibbets`.

### Configuration

When you run it the first time, Snibbets will write a configuration file to `~/.config/snibbets/snibbets.yml`. You can edit that file to set things like your snippets directory, your preferred file extension, and a few other options. Options specified in the config file can always be overriden on the command line with flags.

Once an editor is defined, you can use `snibbets --configure` to open it automatically. Running that commmand before configuring an editor will use whatever app/utility your system defaults to for YAML files.

Default config:

```yaml
---
all: false
copy: false
editor: 
extension: md
highlight: false
include_blockquotes: false
interactive: true
launchbar: false
menus: 
name_only: false
output: raw
source: "~/Dropbox/Snippets"
```

#### Snippet Location

Set the `source` key to the folder where you keep your Markdown snippets. Optionally adjust the `extension` setting if you use an extension other than `md` (e.g. `markdown` or `txt`).

#### Other Options

The `all` setting determines how Snibbets handles files containing multiple snippets. If `all` is true, then it will always display every snippet in the selected file. If false, it will offer a menu and let you choose which snippet to display. You can use `--all` on the command line to just enable this once.

The `copy` setting determines whether the output is copied to the clipboard in addition to being displayed on STDOUT. This is the equivalent of running `snibbets QUERY | pbcopy` (macOS) or `snibbets QUERY | xclip` (Linux). This can be enabled for just one run with `--copy` on the command line. Setting it to true in the config will copy to the clipboard every time a snippet is displayed. On Mac this will work automatically, on Windows/Linux you may need to [install `xclip` or `xsel`][xclip].

[xclip]: https://ostechnix.com/access-clipboard-contents-using-xclip-and-xsel-in-linux/

The `editor` setting is used to open the config file, and to open snippets for editing when using the `--edit` flag. This setting can be any command line utility (`code`, `subl`, `vim`, `nano`, etc.), or on macOS it can be an application name (`BBEdit`, `VS Code`, etc.) or a bundle identifier (`com.sublimetext.4`, `com.microsoft.VSCode`, etc.). If no editor is set, then the file will be opened by whatever the system default is (using `open` on macOS, `start` on Windows, or `xdg-open`on Linux).

The `highlight` key turns on syntax highlighting. This requires that either `pygmentize` or `skyligting` is available on your system (both available via package managers like Homebrew). This feature is still in development and results may be mixed.

The `include_blockquotes` setting determines whether blockquotes are included in the output. By default, Snibbets removes everything other than code blocks (indented or fenced) from the output it displays. But if you want to include a note that you'll see on the command line, you can put it in a block quote by preceding each line you want to preserve with a right angle bracket (`>`).

The `interactive` setting determines whether menus will be displayed. This should generally be true, but if you want silent operation that just displays the best match automatically, set it to false. 

The `menus` setting will determine what method is used for displaying interactive menus. If this is not set, it will be automatically determined in the order of `fzf`, `gum`, and `console`. You can manually choose to use one of these options over another by making it the `menus` setting.

The `name_only` key will permanently set Snibbets to only search for snippets by their filename rather than examining their contents. You can enable this at runtime using `--name-only` in the command.

### Usage

```
Snibbets v2.0.14

Usage: snibbets [options] query
    -a, --all                        If a file contains multiple snippets, output all of them (no menu)
    -c, --[no-]copy                  Copy the output to the clibpoard (also displays on STDOUT)
    -e, --edit                       Open the selected snippet in your configured editor
    -n, --[no-]name-only             Only search file names, not content
    -o, --output FORMAT              Output format (json|launchbar|*raw)
    -p, --paste, --new               Interactively create a new snippet from clipboard contents (Mac only)
    -q, --quiet                      Skip menus and display first match
    -s, --source FOLDER              Snippets folder to search
        --configure                  Open the configuration file in your default editor
        --[no-]blockquotes           Include block quotes in output
        --highlight                  Use pygments or skylighting to syntax highlight (if installed)
        --save                       Save the current command line options to the YAML configuration
    -h, --help                       Display this screen
    -v, --version                    Display version information
```

If your Snippets folder is set in the config, simply running `snibbets [search query]` will perform the search and output the code blocks, presenting a menu if more than one match is found or the target file contains more than one snippet. Selected contents are output raw to STDOUT.

> If you have fzf or gum installed, snibbets will use those for menus, providing fuzzy filtering of options.

#### JSON output

An undocumented output option is `-o json`, which will output all of the matches and their code blocks as a JSON string that can be incorporated into other scripts. It's similar to the `-o launchbar` option, but doesn't contain the extra keys required for the LaunchBar action.

#### Open snippets in your editor

Use the `--edit` flag on any search to open the found snippet file in your editor. Configure your default editor in the config file. `snibbets configure` will open that, but if you don't have an editor set, it might have strange results. To edit manually, open `~/.config/snibbets/snibbets.yml` in your text editor of choice.

#### Creating new snippets

I do most of my snippet editing in [nvUltra], but sometimes I have a function in my clipboard that just needs quick saving and there are so few moving parts to creating a snippet that it just feels like they could be automated/simplified. That's why I added the `--paste` flag. If you have a code snippet in your clipboard, you can just run `snibbets --paste` (or just `-p`) and you'll get a prompt asking you to describe the snippet (used for filename) and one asking what language(s) are represented.

You can input the languages as names, e.g. `rust`, `typescript`, or `scala`, or you can just add file extensions that represent the language. If I say `ts` to that prompt, it will generate an extension of `.ts.md` and then add a metadata tag of `typescript` to the file. The code from the clipboard goes into a fenced code block in the document. You can always go add notes to it later, but it's a great way to save snippets as you come across them (or solutions you figure out after a week of banging your head).

This command requires that a clipboard utility be available. On macOS, you have `pbpaste` by default and don't need to do anything. On Windows and Linux, you'll need to [install either `xclip` or `xsel`][xclip].

[nvUltra]: https://nvultra.com "nvUltra for Mac"


#### Saving Settings When Running

Any time you specify things like a source folder with the `--source` flag, or turn on highlighting or name-only search, you can add the flag `--save` to write those to your config and make them the default options.

## LaunchBar Action

_I'm currently reworking the LaunchBar action, and it doesn't function very well at this time. I'll update when I have a chance._

### Installation

The LaunchBar action can be installed simply by double clicking the `.lbaction` file in Finder. The CLI is not required for the LaunchBar action to function. 

Once installed, run the action (type `snib` and hit return on the result) to select your Snippets folder.

### Usage

Type `snib` to bring the Action up, then hit Space to enter your query text. Matching files will be presented. If the selected file contains more than one snippet, a list of snippets (based on ATX headers in the file) will be presented as a child menu. Selecting a snippet and hitting return will copy the associated code block to the clipboard.
