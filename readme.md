# Hacker News Tailwind

> Note: This readme is not a structured document, it is a chronological record
of the work presented herein and is more or less append-only. Be aware.

I want to learn Tailwind and in order to do it in a practical way, I am going to
clone Hacker News and see how it goes.
I might clone more websites next.

I am using the official Tailwind site as a resource:
https://tailwindcss.com/

I am not using any other framework in combination with Tailwind now so I'll use
the CLI to compile Tailwind to the final CSS.
Other ways to use Tailwind are:

- PostCSS - a plugin which gets invoked by PostCSS which itself is ran by a
  framework it is integrated in
- Framework-specific integration - when using a framework but not PostCSS
  - E.g.: Next https://tailwindcss.com/docs/guides/nextjs
- PlayCDN - use a `script` tag and try out Tailwind in a development setting

Since I am sticking with the CLI, the first step is to install it:

```sh
npm install --global tailwindcss
```

On macOS, with Node (and NPM) installed using the Node installer (my preferred
method) as opposed to NVM or some other tooling manager (e.g.: Turbo), file
permissions are set up in a way that will cause an error unless changed.

This behavior is IMO stupid and I'd argue things should work out of the box,
but nonetheless, that's where we are.
To fix the permissions, run:

```sh
sudo chown -R tomashubelbauer: /usr/local/lib/node_modules
```

In many cases, you'll want to install Tailwind as a development dependency or
use NPX to call it.
I don't use it for any other projects checked out on my machine and I want this
project to always use the latest version so I will install it globally and I'll
keep the globally installed version current.

Tailwind CLI even when installed globally won't export a binary executable to
call by name right away, it still needs to be called using NPX, it just won't
be taken from the repository scope, but instead from the global scope.

To verify that works, the `--help` option can be used.
Note that the Tailwind CLI doesn't provide a `version` command or `--version`
option.

```sh
npx tailwindcss --help
```

I am using Tailwind 3.2.4 at the time of writing.
My versions of Node and NPM are 19.1.0 and 8.19.3, respectively.

To actually install and configure Tailwind to this repository, the `init` CLI
command is used:

```sh
npx tailwindcss init
```

This will drop a configuration file named `tailwind.config.js` to the directory
and this file will be picked up and used when the CLI determines what files to
check for Tailwind classes and where to put the output, stuff like that.

It looks like this:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

So far, I am not loving the fact that the Tailwind CLI is named `tailwindcss`
and not just `tailwind`.
It is too much to type!
Also, not a fan of the old-school module approach in the configuration file.
It is almost 2023 and the configuration file really should be using ESM IMO.

The configuration file should also come with a default `content` value that
picks up `index.html`.
That would be a very weekly opinionated default, better than an empty value.
I tried to see if there is an undocumented implicit default by running just
`npx tailwindcss` but I got yelled at for not specifying `--input`.

Another thing, and this is not necessarily something I mind, but it is curious
how often you see "configuration" files which are just code files.
Of course, this is nothing new, WebPack does it, too, and Pulumi is basically
built on top of this idea.
It doesn't feel obviously good to me, but I don't think it is bad either.

Anyway, using `npx tailwindcss --input index.html` goes through but since I did
not specify `--output`, I get no output.
Not to mention that as of now, `index.html` is empty.

At this moment I am confused as to why I need to use `--input` when I have the
configuration file.

Curiously, using `npx tailwindcss --watch` and I do not need to specify the
`--input` option.
I think there are multiple entrypoints to the CLI but its interface, the help
output or the quickstart documentation don't make this obvious or discoverable.

In general, the installation documentation seems to end mid-way.
https://tailwindcss.com/docs/installation

I would have expected it to show how to set up the configuration file so that
the awkward CLI command can be reduced to something reasonable.

I'll try to put this together myself.
With the default configuration and the `--watch` option, I get no transpilation
as there is no input to speak of to transpile, and I get the result at the CLI
output.

A good first step then seems to figure out how to use the configuration to
specify the output.
Let's take a look at the configuration documentation.
https://tailwindcss.com/docs/configuration

Amazingly, this does not seem to be possible at the moment!
https://github.com/tailwindlabs/tailwindcss/discussions/5033

Between specifying `--output index.css` and `> index.css` I think I will prefer
the latter as it is shorter.

Tailwind really seems gearer towards integration with other pieces of what many
call the modern web development stack which means the DX of using it standalone
suffers as a result of less attention to the use-case.

I will factor out the CLI command to its own shell file and run it instead of
typing it out each time, but I really would like to be able to just run
`tailwindcss` (or better yet, `tailwind`), and have the right thing happen.

```sh
echo "npx tailwindcss > index.css" > run.sh
chmod +x run.sh
./run.sh
```

I will switch to using the `--watch` option later (probably will add it to this
shell script), but for now I want to be deliberate with when I run the compile
step.

The installation documentation page doesn't mention this explicitly, but it
nudges you to name your CSS entry-point `input.css`.
I will treat this as a Tailwind best practice and adopt this standard.

The next step is to import the Tailwind directives in the main CSS file:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

I am curious as to why the non-CSS keyword `@tailwind` was chosen over something
like `@import 'tailwind:…'` which would be as unique and recognizable to the TW
compiler as `@tailwind` is, but wouldn't cause the default VS Code CSS parser to
report a warning at this unknown directive.

With the input and output configured, I can start authoring my Tailwind-enhanced
HTML.
The output file will be `index.css` so let's reference it in the HTML template:
And to make sure it really works, let's adorn the HTML `body` with the Tailwind
`font-bold` utility class.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hacker News</title>
    <link href="index.css" rel="stylesheet" />
  </head>
  <body class="font-bold">
    Hello, world!
  </body>
</html>
```

After running `./run.sh` and opening `index.html` in the browser (e.g. by
running `open index.html`), we see that nothing actually happens!
Or rather, it is apparent that Tailwind is working somewhat on the page, as the
default browser margin is reset.
But our text is not bold and looking through the compilation target `index.css`
there is no hint of `font-bold` at all.

Is `font-bold` not in `@tailwind base;`?
The problem lies elsewhere, as is helpfully reported by the Tailwind CLI.
Despite our use of the `font-bold` helper class, it still reports that it did
not find any Tailwind utility classes in the input.

And indeed I never set this to anything, I only complained it doesn't seem to
default to anything, even implicitly.
That's confirmed now, so let's set it to `index.htm` explicitly and try again.

This time, `./run.sh` went through without that warning and we indeed do see a
bold span in the page as rendered by the browser.
At least HTML inputs are supported!
The documentation provides an example where a CSS input is used instead.

This should be enough to be able to put together a simple development loop of
edit, compile, test.

Let's start building Hacker News, then!
First, let's block out the major landmarks of the site.
We have the centered column containing the whole of the content.
We have the top menu row with the link navigation and optionally, the logged in
user controls.
We have the list of links and their associated controls.
And lastly, we have the footer with more links and the search field.

In terms of the main body wrapper, I want to center it relative to the viewport,
and one of the ways to do this is to make the `body` element a flex container.
Then, the content container can be centered using `justify-content: center`.
A quick helping of `<body class="flex justify-center">` will make this happen
for us.

To give the main container a size relative to the viewport, it seems one of the
container classes in Tailwind will be a way to go.
Let's go with `lg`, the 1024px breakpoint.
It seems it needs to be used in conjunction with `container`, so:
`<div class="container lg">`.

Resizing the browser window, it seems we do get some built-in breakpoints and
the container resizes and accomodates to fit the viewport width.

I think it is time to turn on `--watch` now as I've already forgotten to run
`./run.sh` a couple of times and then was bamboozled as to why my changes were
not working.

Funnily enough, `lg` matches the width of the Hacker News main body almost
exactly at my breakpoint when the browser is maximized.
I will decide later on whether I want to go for a pixel-perfect copy or just an
approximation which won't look exactly like Hacker News but will be nice in
terms of the Tailwind classes and how I use them.
I am already leaning towards the latter.

Let's block out the main menu, the link list and the footer now.
I have only added the container divs and a few links for now.
The link list is an `ol` and I found that `list-decimal` is the way to make it
show the numbers.

By default even unordered lists do not show the list points in Tailwind.

`amber-50` was the closest color from the Tailwind built-in colors to
approximate the Hackew News background.
Funnily enough the site is mockingly called "the Orange Site" yet `orange-50`
wasn't the right fit here.
I am sure it will redeem itself when I go coloring the main menu.
Since this was for a background color, I needed to prefix the color with `bg-`.

To get the body margin (which Tailwind zeroes by default) back, I used `m-2` on
the `body`.
It seems to match HN's pretty closely.

Now for the header background color.
I found that `orange-500` matches the Hacker News color almost perfectly so I
guess the site did redeem itself, indeed!

My menu is two-rows now because I don't have the right-float items figured out
yet.
A "float" is an old-school way of saying I need to make it a flex container and
put an automatic margin on the last item.

At this point I am slowly but surely starting to get why people like Tailwind.
Prototyping this site is _fast_ so far.
Granted, Hacker News is a simple site and it wouldn't be all that slower without
Tailwind, but I do feel the power of fast prototyping on my fingerprints using
it and that's a good sign!

The first menu link (the Hacker News "logo" link) needs to be bold and a bit
larger.
Since the menu is a flex container now, I can also introduce gaps between the
items so that aren't as crowded as they are without.

Initially I thought I would set the logo graphic as a background image of the
Hacker News logo link, but real HN uses an image for it and it seemed easier to
do the same thing to not overcomplicate the utility classes.

I downloaded the image off HN to the repository because it seems to load super
slowly or maybe it was just Firefox being weird on me.

Turns out to do a 1-pixel wide border, `border` is the way to go, not `border-1`
which surprised me but I think it makes sense.
Maybe `border-1` should print a warning?
One would think this would be a common mistake.

I put the image into the logo link so it is clickable as well.
Unfortunately, this made the logo link break into two rows.
I messed around with `whitespace-pre` but that blew up the logo because there is
whitespace before the `img` (I think).
I settled on using `flex` with a nested `gap`.

I put a padding on the menu row to match the HN spacing.
`0.5` looked best but I was disappointed to learn I can't do `p-.5`, have to do
`p-0.5`.

With the logo being a flex container the inline links in the rest of the menu
look off vertically now so I need to align the menu row flex items centerwise as
well.

The link text font size was matched using `text-sm`.
That looked good.

The "logout" item is weird.
It seems to stick to the end of the row, not respecting the padding.
And it should be padded more according to the HN template, so I tried using
`pr-2` on the menu row to no avail.

I still need to figure this out.

The rough next steps:

- [ ] Use the logo for the tab icon as well
- [ ] Figure out the "logout" item right padding from parent not being respected
- [ ] Mark the "news" item as active using the light color
  - [ ] Consider expanding this to `:target` and pure-CSS tabs
- [ ] Look into VS Code Tailwind extension for utility class auto-complete
- [ ] Figure out the list item numbers being outside of the area by default
- [ ] Expand the list items to have the control links and other details
  - [ ] Consider generating these dynamically
    This might make the previewing/testing experience a bit slower so be aware.
- [ ] Figure out why Firefox is laggy on this file - is it the watcher?
  - [ ] Try working with the watcher off to see if it is access race
    Maybe my browser refresh comes too early while the file is still being
    compiled or written by the watcher.
  - [ ] Consider using VS Code extension to have the browser page side by side
    I could side-step this whole issue by trying this out.
    There should be non-localhost based preview extensions hopefully.
  - [ ] Consider figuring out Tailwind hot reload or generic hot reload
    This would solve potential file access race issue if it is that.
- [ ] Design the list items
- [ ] Design the footer items
- [ ] Consider also designing the discussion tree for practice/experience
- [ ] Consider hooking this up to the Hacker News API for live data
  This would make the list load with a delay so might interfere with iteration
  speed while testing manually.
