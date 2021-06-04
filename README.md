<h1 align="center"> 🍰 </h1>

## What is Cake?
Cake is a *really* thin, drop-in replacement/wrapper around `make` that runs all
of your targets inside of a development Docker/Podman container.

### Vision
- Most projects should have a Makefile and a Dockerfile at their root for
  convenience and portability
  - The Makefile is the single source of truth for the build process
  - The Dockerfile is the single source of truth for the build environment
- A container runtime should not be a hard dependency to build the project.
- Choosing between containerized and "naked" builds should be as easy as typing
  `make` or `cake` interchangeably 
- CI/CD pipelines should be able to reuse the instructions from the Makefile in
  an ergonomic way without having to keep the build context in mind
    
## Why Cake?
Because I found myself constantly writing Makefiles that run their targets in a
container, then adding in add-hoc ways for people not to use the container
through environment variables, followed by a half-hearted attempt at
optimizations through bind-mounts and less frequent restarts, and some faulty
logic to avoid name and tag clashes. I figured it was time to extract this into
a script. Despite its simplicity, the script covers 99% of my use cases for
tools like [act](https://github.com/nektos/act) without being tied to a specific
forge.

### Why POSIX sh
Because additional dependencies are a problem, especially in corporate
environments. just `curl`/copy this script into a directory on your `$PATH` and
you're good to go.

## Completions
I might provide them for convenience later, but in principle all you need to do
is reuse existing make completions. In `zsh` that looks something like this:
```zsh
compdef _make cake
```

## How to Use
Just use `cake` instead of `make`. The defaults should fit most use cases. If
you really have to, you can specify additional `docker`/`podman` arguments using
`$CAKE_RUNTIME_ARGS`. I recommend placing these in your
[.envrc](https://direnv.net/) if you need them to stick around due to the
specific needs of your project.

## Tips

If I want to debug my development container, I like to add a `shell` target
to my `Makefile` like so:
``` makefile
shell:
    /bin/sh
```
It's more ergonomic then copying the container name.


The same goes for dealing with things like `./autogen.sh` and the `./configure`
script (often managed directly by the user). I tend to call those through a
`Makefile` as well. Take this snippet from the `GNUMakefile` in the Emacs source
tree as an example:

``` makefile
configure:
	@echo >&2 'There seems to be no "configure" file in this directory.'
	@echo >&2 Running ./autogen.sh ...
	./autogen.sh
	@echo >&2 '"configure" file built.'

Makefile: configure
	@echo >&2 'There seems to be no Makefile in this directory.'
	@echo >&2 'Running ./configure ...'
	./configure
	@echo >&2 'Makefile built.'

# 'make bootstrap' in a fresh checkout needn't run 'configure' twice.
bootstrap: Makefile
	$(MAKE) -f Makefile all
```

