![Vale Linting](https://github.com/hepabolu/ttt/workflows/Linting/badge.svg)

# Creating the TTT publications

This repository contains all the content of the Taming the Terminal tutorial as well as the scripts that are needed to build the output.

## How it works

Download the HTML file of the episode and convert it to Markdown using the tttconvert code.
Then, convert the Markdown to Asciidoctor using Kramdoc.
Once you've set up all the necessary tools, you can simply build all three versions using the command

`bundle exec rake book:build`

To get there you need:

- Ruby as a separate install (because you don't want to mess up your System folder)
- various gems
- bundler
- rake
- asciidoctor
- asciidoctor-epub3
- asciidoctor-pdf
- clone the GitHub repository

## Prepare your environment

### Install NodeJS

Install NodeJS version 12.x.x or later. Follow the instructions on [nodejs.org](https://nodejs.org/en/).

### Install Ruby on macOS

Ruby is default part of macOS but every 'gem install <some package>' will lead to an attempt to update the system framework. Not a good idea.

Follow the instructions at: [GoRails.com](https://gorails.com/setup/osx/10.15-catalina)

just the part 'Installing Ruby'

### Install Kramdoc

Install Kramdoc using

```
gem install kramdown-asciidoc
```

more information at [Convert Markdown to AsciiDoc](https://matthewsetter.com/technical-documentation/asciidoc/convert-markdown-to-asciidoc-with-kramdoc/)

### Clone the git repository

Clone the repository

```
git clone https://github.com/hepabolu/ttt.git
```

### Install necessary Ruby gems

switch to the root directory of the git repository you just cloned and run

```
gem install
```

This installs all Ruby gems in the `Gemfile`.

## Compile

### Prepare the files

The original episodes are HTML pages on Bart's website. They need to be converted first to AsciiDoctor file before they can be processed further. This section explains how to do that for a new episode.

1. Download the HTML page from Bart's website, use Safari and download it as 'page source'. Save in the `sourcefiles` directory. This ensures the correct naming convention and the original links to the images and other assets.

2. Convert HTML to Markdown

```
cd tttconvert
./tttconvert.sh xx # xx is the number of episode, leave blank to convert all files
```

This app also downloads the images. Output is in `convert2` and `convert2/assets`.

3. Convert to Asciidoctor

```
cd ../convert2
kramdoc --format=GFM --output=tttXX.adoc tttXX.md
```

4. Copy the Asciidoctor files + assets to the book

```
cd ../../convert2
cp tttXX.adoc ../book  # XX is the file you want to copy
cp -r assets/tttXX ../book/assets
```

5. Make the QRcode

- Copy the link to the podcast to the file `publish/mp3_files
- run the script

```
cd ../scripts
./generate_qrcode.sh
```

6. Cleanup

- add the new tttXX.adoc file to `book/ttt-contents.adoc`
- if necessary, rename the QRcode file to match the TTT_XX.png naming convention
- open the `book/tttXX.adoc` file and fix the episode box, the reference to the QRcode and miscellaneous changes.

### Build the files

Test if your setup works by running

```
bundle exec rake book:build
```

or simply

```
bundle exec rake
```

because `book:build` is the default.

When it finishes, the output looks like this:

```
Generating contributors list

Converting to HTML...
 -- HTML output at output/ttt.html
Sync the assets

Converting to EPub...
 -- Epub output at output/ttt.epub

Converting to PDF A4... (this one takes a while)
 -- PDF output at output/ttt.pdf

Converting to PDF US... (this one takes a while)
 -- PDF output at output/ttt-us.pdf
```

and no other errors, there should be a PDF in A4-size, a PDF in letter-size, an HTML file and an ePub file in the `output` directory.

<s>**Note**: to have the screenshots visible in the HTML file, copy the `assets` folder from `book/assets` to `output/assets`</s>.
Assets are already synced in the build script.

## Book setup

Every episode is put in its own file in `book`. All images are in
`book/assets/ttt<nr of episode>`.

`ttt-spine.adoc`, `ttt-epub-spine.adoc` and `ttt-contents.adoc` hold the general information to pull the content together in one output file.

For now:

- <s>`colophon.adoc` holds some boilerplate text. It needs to be update to proper information</s>
- `index.asc` is empty, it's just there because the spine docs refer to it. Not sure if we need to fill it. Because the entries are only visible in the PDF, the entire section is commented out.

Note: language is British English!

## Bug fixes and workaround

This section contains some notes on bug fixes and workarounds that have been applied to get it working.

### Fake second paragraph

See: [Asciidoctor git repository](https://github.com/asciidoctor/asciidoctor/issues/2860)
Worked around by adding a second paragraph either by separating the last (few) sentence(s) or by adding an invisible second paragraph consisting of a single space.

```
+++&nbsp;+++
```

Books doesn't like this, so I had to surround it with `ifdef`s:

```
ifndef::backend-epub3[]
+++&nbsp;+++
endif::[]
```

**Update 2020-05-01**: Since the audio section was converted to a sidebar, that counts as a second paragraph, so all the fake second paragraphs are deleted.

### Backticks problems

Somehow there is a bug in `asciidoctor` that causes backticks to be passed through rather than marking the conversion to `monospace`.

### Color coding in ePub

`Rouge` is used as source code highlighter in PDF and HTML, but it doesn't work in ePub. Only `Pygments` is supported in ePub, but this has no good support for shell scripts. Somehow the color coding in Books actually looks to be just black & white.

**UPDATE**: it looks like the attribute for the styling in the spine is not honored. It should be added as an attribute on the command line like

```
-a pygments-style=manni
```

Source: [Prepare an asciidoc document](https://asciidoctor.org/docs/asciidoctor-epub3/#prepare-an-asciidoc-document)

**UPDATE 2020-07-16**: Looks like Rouge _is_ supported now in ePub, _AND_ it gives better colour coding, so all ePub is now also switched to Rouge.

### Highlights in source code

It is not possible to highlight specific parts of the source code, so all references to e.g. `<strong>` must be removed from the snippet or it will show up verbatim in the output file.

**UPDATE 2020-07-16**: highlighting is supported by adding an attribute to the codeblock indicating the lines to be highlighted and by adding appropriate CSS to the various themes. All code blocks that have highlighting in the original html are now marked for highlighting in the Asciidoctor files as well.

### Line numbering in source code

Although Rouge supports line numbering in source code blocks, the implementation in Asciidoctor is very simple. The code is placed in 2 table cells, one with the line numbers, one with the code. It doesn't take into account the extra space needed when long code lines wrap to the next line.
After numerous attempts to fix the problem I got stuck because my code adjustments in the Asciidoctor code broke the functionality to add annotations in the code and I haven't found a way to preserve that functionality.
I therefore decided to skip the line numberin in code blocks that have long lines of mostly output. The highlighting does work, therefore it's still possible to point out the important lines.

### Keyboard shortcuts

Asciidoctor supports the HTML5 keyboard shortcuts, so change any reference to keyboard shortcuts to the HTML5 keyboard counterparts.
Note, the command key (CMD) can be used as `{commandkey}`.
<s>For the shortcuts to show up properly in the VScode previewer, each document needs an `:experimental:` option in the header.</s>
I added a document with variables to each document so this is automatically taken care of.

<s>Rather than `Ctrl` write `Control`, because that's what it says on the key cap.</s>
I changed my mind, `Ctrl` is much more common than 'control'.

NB. with

```
&#8594; &rarr;  →      Right arrow
&#8592; &larr;  ←      Left arrow
&#8593; &uarr;  ↑      Up arrow
&#8595; &darr;  ↓      Down arrow
```

it's possible to create `kbd:[&larr;]` arrow keys. However, ePub doesn't support the second column definitions, PDF doesn't support the Unicode definitions for the up and down arrows. So I've decided to skip them entirely and just use the words 'up', 'down' etc.
