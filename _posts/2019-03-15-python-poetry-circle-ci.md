---
layout: post
title: "A Recipe for Poetry and CircleCI"
date: 2019-03-15 01:00:00
description: "Making Poetry and CircleCI really good friends ❤️"
image:
---

Last week I wrote a Python library using [Poetry](https://github.com/sdispater/poetry) to manage its dependencies and for publishing it to [PyPI](https://pypi.org/). So far, the experience has been fantastic.

# Poetry
> Poetry helps you declare, manage and install dependencies of Python projects, ensuring you have the right stack everywhere.

Poetry is so cool! It does a lot of things, including:
- Manages Python dependencies really well, using the new [standardized pyproject.toml](https://www.python.org/dev/peps/pep-0518/).
- Separates dependencies and _dev_ dependencies.
- Deals with the virtual environment for you.
- Has one (_one!_) command to build _and_ publish the package to PyPI.

After Poetry was up and running, I needed to hook it up with a Continuous Integration & Continuos Delivery tool to run our builds and publish the lib to PyPI. Enters CircleCI.

# CircleCI
[CircleCI](https://circleci.com) is a CI/CD platform. It runs certain workflows in a certain environment when a certain event happens. Very specific, right?

Let me give you two examples:
- Whenever I open a PR at GitHub, I want to checkout the new code and run the tests.
- Whenever I create a new release at GitHub, I'd like to run the step defined before _and_ publish it to a package repository.

This is what CircleCI can do for you! On top of that, the tool is free for open source projects ✨

All the nitty gritty from the examples above are configured in a `.circleci/config.yml` file. There, you can specify which kind of environment you want to have (Python?, Node?), what commands you want to execute (install dependencies? run tests?) and when they should be execute (new PR? new release?).

The catch of my adventure was to figure it out how Poetry and CircleCI could be good friends ❤️

# The Recipe
First of all, no change was needed in the Poetry configuration file – just make sure you have the libraries used in the CircleCI jobs: `flake8` and `coveralls`. The file below is the `.circleci/config.yml`.

Second, it took me a while to figure out the whole cache thing, but everything seems to be working nicely now 😌

Finally, here is how you can make this friendship thrive!

<script src="https://gist.github.com/jonatasbaldin/b0eba2ac8ee887ca27ee811697d4d73b.js"></script>

---

I hope this article helped you to get into Poetry with CircleCI! If you have any feedback, please leave them in the comments 🚀
