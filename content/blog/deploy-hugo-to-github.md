---
title: "Deploy Hugo to Github pages"
date: 2017-10-02T22:46:12+02:00
draft: false
---

# Deploy a Hugo website to Github pages

> GitHub provides free and fast static hosting over SSL for personal, organization, or project pages directly from a GitHub repository via its [GitHub Pages service](https://help.github.com/articles/what-is-github-pages/).

I know there are other options, such as [Netlify](https://www.netlify.com/blog/2016/10/27/a-step-by-step-guide-deploying-a-static-site-or-single-page-app/), but I was not too keen to yet depend on another framework, specially one using `node.js`... famous last words...

## What didn't work... or at least I gave up in 5 minutes

From the [Hugo's tutorial](https://gohugo.io/hosting-and-deployment/hosting-on-github/):

````bash
git checkout --orphan gh-pages
git reset --hard
git commit --allow-empty -m "Initializing gh-pages branch"
git push origin gh-pages
git checkout master
rm -rf public
git worktree add -B gh-pages public origin/gh-pages
````

Create the following `commit-gh-pages-files.sh`:

````bash
hugo -D
cd public && git add --all && git commit -m "Publishing to gh-pages" && cd ..
````

And run with exec mode:

````bash
chmod +x commit-gh-pages-files.sh
./commit-gh-pages-files.sh
````

Push your changes:

````bash
git push origin gh-pages
````

The next step is to point `Github` from where do we want to publish our page, by default it should use the same `gh-pages` branch we just created, but double-checking does no harm.  On the repository page go to the `Option` tab (not the same as the profile one), and scroll down until you reach to the Github pages.

The page was published at [https://alignan.github.io/argos-library/](https://alignan.github.io/argos-library).

I followed all steps above but the result was not as expected:

[![](/img/deploy-hugo-to-github/00.png)](/img/deploy-hugo-to-github/00.png)

Hardly the result I was expecting!

The pase was not rendering functional (i.e. the links were broken and pointing to a default `404` page)... what!?  I even changed the `baseURL = "/"` as pointed in some documentation.  The theme was not rendering properly...

## Next thing I tried...

From [haruair](https://haruair.github.io/post/setup-hugo-blog-on-github-pages-with-travis-ci/) site I took the following instructions, which is great because I wanted to have a [Travis](https://travis-ci.org) integration as well.

First step is clean up the mess from the section above:

````bash
rm -rf public
git worktree prune
git branch -D gh-pages
git push origin :gh-pages
````

However the setup instructions didn't convince me... I also checked instructions at [Jente's blog](https://hjdskes.github.io/blog/update-deploying-hugo-on-personal-gh-pages/), but I wanted to keep my master branch for sources, and I felt I was going the wrong direction...

## The one I finally implemented

Then I stumbled upon [this snippet](https://github.com/whipperstacker/blog/blob/master/content/post/deploying-a-hugo-site-to-github-pages.md) and the explanation about why using `gh-pages` is not recommended made sort of sense: from a practical perspective, I'm creating a personal page (user page), whereas I might create pages for my specific projects (gh-pages) - this semantic differentiation was logical.

The next step is to create a new repository to host the ***rendered site from Hugo*** and add as submodule, I created mine as `alignan.github.io` as expected [by the User pages](https://help.github.com/articles/user-organization-and-project-pages/) (see the link for more information).

***Pro-tip:*** remember to change the `draft: false` in your posts!, else it will not be rendered by Hugo (unless you are enabling the option to render drafts).

````bash
rm -rf public
git submodule add -f https://github.com/alignan/alignan.github.io.git public
````

Now render the page, and publish to the `alignan.github.io` repository.

````bash
hugo
cd public & git add .
git commit -m "Rendered website"
git push -f origin master
````

Now back to the main source code repo (in the step above remember we are in the `/public` folder):

````bash
cd ..
git add .
git commit -m "Added a repository to render the site"
git push -u origin master
````

If everything was done correctly, you should see both `submodules`:

````bash
$ cat .gitmodules 
[submodule "themes/ghostwriter"]
	path = themes/ghostwriter
	url = https://github.com/jbub/ghostwriter
[submodule "public"]
	path = public
	url = git@github.com:alignan/alignan.github.io.git
````



