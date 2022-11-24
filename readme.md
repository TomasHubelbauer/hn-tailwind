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
