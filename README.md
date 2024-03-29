<p align="center">
  <img src="https://www.snaplogic.com/wp-content/themes/snaplogic/public/img/snaplogic-logo-white.svg" alt="SnapLogic Logo" width="226" style="vertical-align:middle;margin:10px 10px">
</p>

<p align="center">The <a href="http://developer.snaplogic.com">Snap Development Documentation</a> guides a developer creating Snaps for the <a href="https://www.snaplogic.com/">SnapLogic Intelligent Integration Platform</a>.</p>

<p align="center"><img src="https://i.imgur.com/Ile65Kr.png" width=700 alt="Screenshot of Snap Development Documentation"></p>

# Highlights

* **Open-Source** - The Snap Development Docs are a fork of [Slate](https://github.com/lord/slate), an open-source API documentation generator. The SnapLogic Community can [contribute back](https://gist.github.com/Chaser324/ce0505fbed06b947d962), [give feedback, open issues](https://github.com/SnapLogic/developer.snaplogic.com/issues) etc. 

* **Bootstrapped** - The Maven archetype (project building tool) that partners the documentation provides eight sample Snaps to learn from and to build upon. A SnapLogic customer can be up-and-running and deploying custom Snaps to their organization within minutes.

* **Detailed** - In-depth guides to setting up a Snaplex, and developing and deploying your own Snaps, with explanations, screenshots, and code samples side-by-side, including:

	* Creating a Snap
	* Reading and writing documents and binary data
	* Accepting and validating user input
	* Expression-enabled Snap properites
	* Suggesting property values
	* Understanding Input/Output View Schemas
	* Exceptions and Error Views
	* Authenticating with Accounts
	* Deploying Snap Packs
	* Testing Snaps

* **Fast, serverless** - Single-page static website.

* **Responsive** - Design adapts to screen size (phone, tablet etc.).

* **Searchable** - Basic search supported.

* **Portable** - Uses Markdown to decouple content from presentation; multiple themes possible.

* **Coherent** - Table of Contents dynamically highlights what content you are currently reading.

# Contributing Back? Need Help? Found a bug?

Contributing to this documentation is straightforward. Simply fork or clone (SnapLogic employees only) this repository and follow the instructions below.

[Submit an issue](https://github.com/SnapLogic/developer.snaplogic.com/issues) to the developer.snaplogic.com Github repository if you need any help. And, of course, feel free to [submit pull requests](https://gist.github.com/Chaser324/ce0505fbed06b947d962) with bug fixes or changes.

# Running Locally

### Prerequisites

You're going to need:

 - **Linux or macOS** — Windows may work, but is unsupported.
 - **Ruby, version 2.5 or newer**
 - **Bundler** — If Ruby is already installed, but the `bundle` command doesn't work, just run `gem install bundler` in a terminal.

### Getting Set Up

1. Fork this repository on Github. (SnapLogic employees can skip this step.)
2. Clone *your forked repository* (not our original one) to your hard drive with `git clone https://github.com/YOURUSERNAME/developer.snaplogic.com.git`
3. `cd developer.snaplogic.com`
4. Initialize and start the Slate server. You can either do this locally, or with [Vagrant](https://www.vagrantup.com/):

```shell
# either run this to run locally
bundle install
bundle exec middleman server

# OR run this to run with vagrant
vagrant up

# OR with docker, you can serve the live documents with the following (from the root of this repository)
docker run --rm --name slate -p 4567:4567 -v $(pwd)/source:/srv/slate/source slatedocs/slate:latest serve

# With docker, to build for deploying, run the following (from the root of this repository)
docker run --rm --name slate -v $(pwd)/build:/srv/slate/build -v $(pwd)/source:/srv/slate/source slatedocs/slate:latest build
```

You can now see the docs at http://localhost:4567. Whoa! That was fast!

### Deploying

* `git commit -a -m "Commit message"`
* `git push`
* `./deploy.sh`

If running via docker locally, follow these steps:
* `git commit -a -m "Commit message"`
* `git push`
* `docker run --rm --name slate -v $(pwd)/build:/srv/slate/build -v $(pwd)/source:/srv/slate/source slatedocs/slate:latest build`
* `./deploy.sh --push-only`

Learn more about [editing Slate markdown](https://github.com/lord/slate/wiki/Markdown-Syntax).

If you'd prefer to use Docker, instructions are available [in the Slate wiki](https://github.com/lord/slate/wiki/Docker).
