---
title: "My Daughter's Book"
date: 2017-11-26T01:53:16+01:00
draft: false
tags: [ "Challenge 2017", "Personal", "Erin", "Book" ]
---

# My Daughter's book

This is part of my [Challenge to make 26 years before 2017 ends](https://github.com/alignan/things-to-do/blob/master/README.md).

I had an overdue and on-going task in my list: write a book for Erin, my daughter.  I began writing a fantastic tale about Erin (our fearless heroin) and her surprising companion back in June 5th 2017, and now, November 25th it is finally done and sent to the printers.

Literally.

[![](/img/my-daughters-book/00.png)](img/my-daughters-book/00.png)

In Spanish.

I self-published in [lulu](http://www.lulu.com/home).  It is relatively easy to publish a book, and the prices are not bad.  I selected a hard cover and full color printing (15,24 x 22,86 cm), for overall 16.17€.  The wizard to import from PDF and create the cover and back-cover is painful to use, and shipping is quite expensive (~12€).  However, I wanted something fast to use, and the cost/time relationship was acceptable.

## The development

The book was written in `Markdown`, using `Pandoc` and `xelatex` engine to compile to PDF (it is also possible to compile for `EPUB` and `HTML` as well).  The sources are available in the following repository:

````
https://github.com/alignan/cuento-erin-y-poki
````

I wanted to write the book not only in a portable way, but also thinking on automating the building process.  As I commented earlier in the Project page, I later opted-out of this option as it would involve either making the repository private, or having to upload family photos on the open.  I decided to compile locally, thus the photos used in the book are kept in my local working copy (synced to `Dropbox` as well).

The `Makefile` is interesting (and easy to automate using `Travis CI` or similar):

````bash
BUILD = build
BOOKNAME = erin-y-poki
TITLE = titulo.txt
METADATA = metadata.yaml
CHAPTERS =  ch/00.md ch/01.md ch/02.md ch/03.md ch/04.md ch/05.md ch/06.md \
            ch/07.md ch/08.md ch/09.md ch/10.md ch/11.md ch/12.md
COVER_IMAGE = img/cover.jpg
LATEX_CLASS = book
CSS_STYLE = stylesheet.css
TEMPLATE_PDF = templates/mybook.latex
FORCE_DATA_DIR = --data-dir=$(CURDIR)

PANDOC_LANG = -V lang=es-ES
PANDOC_PDF_SIZE = -V geometry:paperwidth=6in -V geometry:paperheight=9in -V geometry:margin=.5in

# https://alvinalexander.com/macos/list-xetex-xelatex-fonts-available-mactex
PANDOC_FONT = -V mainfont=Monaco

all: book

book: epub html pdf

clean:
	rm -rf $(BUILD)

epub: $(BUILD)/epub/$(BOOKNAME).epub
html: $(BUILD)/html/$(BOOKNAME).html
pdf:  $(BUILD)/pdf/$(BOOKNAME).pdf 

$(BUILD)/epub/$(BOOKNAME).epub: $(TITLE) $(CHAPTERS)
	mkdir -p $(BUILD)/epub
	pandoc $(PANDOC_LANG) $(PANDOC_FONT) -S --epub-metadata=$(METADATA) --epub-cover-image=$(COVER_IMAGE) -o $@ $^

$(BUILD)/html/$(BOOKNAME).html: $(TITLE) $(CHAPTERS)
	mkdir -p $(BUILD)/html
	pandoc $(PANDOC_LANG) $(PANDOC_FONT) --standalone --to=html5 -o $@ $^

$(BUILD)/pdf/$(BOOKNAME).pdf: $(TITLE) $(CHAPTERS)
	mkdir -p $(BUILD)/pdf
	pandoc $(PANDOC_LANG) $(PANDOC_PDF_SIZE) $(PANDOC_FONT) -fmarkdown-implicit_figures --latex-engine=xelatex -V documentclass=$(LATEX_CLASS) $(FORCE_DATA_DIR) --template=$(TEMPLATE_PDF) -o $@ $^

.PHONY: all book clean epub html pdf
````

The only thing required is to run `make`, easy eh?

I had to fix quirks and small annoyances manually, such as code in blocks not being wrapped-around, sizes and fonts, removing auto-captioning for images, etc.

The most problematic part was having to use absolute paths (instead of relatives) for the images, as I wasn't able to expand from the `Makefile` directly.  I could just used a script to replace a given text occurrence with the `$(CURDIR)` output before compiling with `pandoc`, but I chose to move forward with the printing itself, as my goal was to have this book printed before December 24th, so it could be a Christmas present for my daughter.

I created an [issue about it](https://github.com/alignan/cuento-erin-y-poki/issues/1) as a reminder to pay out my technical debt.

I modified the `latex` template as well, the one used for the book is in [templates/mybook.latex](https://github.com/alignan/cuento-erin-y-poki/blob/master/templates/mybook.latex).  People more familiar with `latex` than me should be able to create a more refined content.

I still need to improve the templates for `EPUB` and `HTML` - I just defaulted to whatever `pandoc` prefers to use, but there is a `stylesheet.css` file which could be used to further style the look and feel.

## The result

I released the first printable version (the one sent to print to Lulu).  Details are available in the `v1.0` release:

[![](/img/my-daughters-book/01.png)](img/my-daughters-book/01.png)

I will need to wait until December 24th to see the actual physical result.

It has been quite a ride - writing a children's book while fighting with a `Makefile`, not always easy to keep the proper focus and inspiration at the same time.

Erin, if you ever read this post, please note how much I love You, specially through the commit history.
