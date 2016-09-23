<p align="center">
  <img src="https://www.snaplogic.com/assets/application/logo-header-096e0919c2ce87a4ebd541deac1f5592.png" alt="SnapLogic Logo" width="226">
</p>

<p align="center">The <a href="http://developer.snaplogic.com">Snap Development Documentation</a> guides a developer creating Snaps for the <a href="https://www.snaplogic.com/">SnapLogic Elastic Integration Platform</a>.</p>

<p align="center"><img src="https://dl.dropboxusercontent.com/u/3519578/Screenshots/dmbo.png" width=700 alt="Screenshot of Snap Development Documentation"></p>

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

No need to fork the repository - 

# Running Locally

### Prerequisites

You're going to need:

 - **Linux or OS X** — Windows may work, but is unsupported.
 - **Ruby, version 2.0 or newer**
 - **Bundler** — If Ruby is already installed, but the `bundle` command doesn't work, just run `gem install bundler` in a terminal.

### Getting Set Up

1. Fork this repository on Github.
2. Clone *your forked repository* (not our original one) to your hard drive with `git clone https://github.com/YOURUSERNAME/developer.snaplogic.com.git`
3. `cd developer.snaplogic.com`
4. Initialize and start the Slate server. You can either do this locally, or with Vagrant:

```shell
# either run this to run locally
bundle install
bundle exec middleman server

# OR run this to run with vagrant
vagrant up
```

You can now see the docs at http://localhost:4567. Whoa! That was fast!

Learn more about [editing Slate markdown](https://github.com/lord/slate/wiki/Markdown-Syntax).

If you'd prefer to use Docker, instructions are available [in the Slate wiki](https://github.com/lord/slate/wiki/Docker).
