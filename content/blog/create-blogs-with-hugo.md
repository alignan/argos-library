---
title: "Create Blogs With Hugo"
date: 2017-09-30T23:41:11+02:00
draft: false
---

# Create a blog using Hugo

Hugo is an open source static site generator based on [Go](https://golang.org)

This is part of my [Challenge to make 26 years before 2017 ends](https://github.com/alignan/things-to-do/blob/master/README.md).  Proudly to say, this is the begining of my journey.

## Quick-start

Taken from [Hugo's documentation](https://gohugo.io/getting-started/quick-start/)

### Install Hugo and other dependencies

Heads-up: you will need [Homebrew](https://brew.sh).

````bash
brew install hugo
````

Check the version:

````bash
$ hugo version
Hugo Static Site Generator v0.29 darwin/amd64 BuildDate: 2017-09-30T22:45:47+02:00
````

Seems we are good to go!

### Create a new site and add a theme

Let's create a new site called `argos-library`

_Argos will be a recurrent theme across my development work and digital life_

````bash
$ hugo new site argos-library

Congratulations! Your new Hugo site is created in /Users/antonio.lignan/Desktop/Sandbox/static-blog/argos-library.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
````

The following structure is created:

````bash
.
├── archetypes  : preconfigured front matter fields
├── config.toml : configuration file in TOML format
├── content     : holds the sections (content/blog, etc.) and all content of the website
├── data        : store configuration files used by Hugo
├── layouts     : .html templates which defines how the site is rendered
├── static      : static content of the site (CSS, JavaScript, images, etc.)
└── themes      : holds the pre-defined themes
````

Initialize a git repository inside

````bash
cd argos-library
git init
````

As suggested above, the next step is to include a `theme`.  Hugo has [an extensive list of available themes](https://themes.gohugo.io/).


I chose [Hyde](https://themes.gohugo.io/hyde/) theme to start with.  I like clean and minimal designs, but I like having a static sidebar as navigation reference, which I will probably end up using for random messages... whatever...

Initialize the theme repository in the `themes` folder as a submodule:

````bash
git submodule add https://github.com/spf13/hyde.git themes/hyde
````

And enable `hyde` as the default theme:

````bash
echo 'theme = "hyde"' >> config.toml
````

The `config.toml` should look like this:

````bash
baseURL = "http://example.org/"
languageCode = "en-us"
title = "Antonio Lignan's blog"
theme = "hyde"
````

I changed the `title` obviously, no further modifications at this stage.

## Adding a new post

Creating a new page is surprisingly easy, just run the following:

````bash
hugo new blog/create-blogs-with-hugo.md
````

Which as expected is this very own page.

The file was created at `content/blog/create-blogs-with-hugo.md`.

The newly created file had the following content:

````
---
title: "Create Blogs With Hugo"
date: 2017-09-30T23:41:11+02:00
draft: true
---
````

As here is the first mention of `draft` (self-explanatory), let's take a step into `front matter`.  This is a concept from `Jekyll` (I think...), basically any file with a `YAML` front matter block is processed as a special file.  The block shown above must be placed in the beginning of the file and keep the triple-dashed lines.

Inside the `YAM` block variables (custom or predefined) can be set.  Hugo accepts `TOML`, `YAML` and `JSON` as front matter formats, each with its own identifying tokens.

Some of the basics predefined front matter variables are:

* `draft`: if `true`, the content will not be rendered (unless `--buildDrafts` is used)
* `title`: self-explanatory
* `date`: self-explanatory and auto-populated
* `keywords`: content meta keyword

## Running the blog locally

To start the server just run:

````bash
$ hugo server -D

Started building sites ...

Built site for language en:
1 of 1 draft rendered
0 future content
0 expired content
1 regular pages created
8 other pages created
0 non-page files copied
0 paginator pages created
0 tags created
0 categories created
total in 4 ms
Watching for changes in /Users/antonio.lignan/Desktop/Sandbox/static-blog/argos-library/{data,content,layouts,static,themes}
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
````

The `-D` is the equivalent of `--buildDrafts`.  The page should be accessible over the [http://localhost:1313](http://localhost:1313/) address.

Images should be put inside the `/static` folder in any sub-folder as preferred.  I opted to adopt a naming convention based on the title of the post, and images are names as an increasing number.  My first image was `00.png` stored in `static/img/create-blogs-with-hugo`, just showing how the blog was rendered locally:

[![](/img/create-blogs-with-hugo/00.png)](/img/create-blogs-with-hugo/00.png)

Note the way to add images to the pages is to use `Markdown` tags as usual.  As what link to use, regardless the images are added to the `/static` folder, by default the content of this folder is dumped at root.  My image above was linked to `/img/create-blogs-with-hugo/00.png`, this is the complete path to the image once rendered:

````bash
http://localhost:1313/img/create-blogs-with-hugo/00.png
````

***Update:*** I changed the theme and used [Ghostwriter](https://themes.gohugo.io/ghostwriter/)

## Minor tweaks

Before committing the first version, first add a `gitignore` file:

````bash
# Hugo default output directory
/public

# Other files and folders not wanted
.DS_Store
````

The blog sources are in [Github](https://github.com/alignan/argos-library)

## Next steps

* Deploy the blog in Github pages (probably)
* Other tweaks and improvements to the template and formatting
