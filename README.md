# R Package Development Environment (.devcontainer)

This directory contains the configuration files for a **Dev Container**, providing a consistent, reproducible environment for R package development using RStudio Server, `renv`, and AI-assisted tooling.

## 1. System Setup

### 1.1. Install Docker, npm & devcontainer CLI

Install Docker via the [Docker Instructions](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

Install npm via the [npm Instructions](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-22-04#option-1-installing-node-js-with-apt-from-the-default-repositories)

Install devcontainer CLI via the [devcontainer CLI Instructions](https://code.visualstudio.com/docs/devcontainers/devcontainer-cli#_npm-install)

### 1.2. ANTHROPIC_API_KEY

Sign up to [Claude](https://claude.ai/)

Sign up to [Anthropic](https://console.anthropic.com)

### 1.3. .Renviron for Storing Credentials

I use .Renviron for storing credentials. Make sure .Renviron is added to your .gitignore file.

```R
ANTHROPIC_API_KEY=sk-ant-...
GITHUB_PAT=ghp_...
```

## 2. Setup and Launch

Use the `devcontainer` CLI to build and launch the environment.

### 2.1. Command to Build and Run

Run this command from the root of your project directory to build the container, apply your dotfiles, and start the RStudio Server process.

```Bash
devcontainer up --workspace-folder . --remove-existing-container --dotfiles-repository "https://github.com/MichaelSandilands/dotfiles" --dotfiles-install-command "install.sh"
```

You can see [My Dotfiles](https://github.com/MichaelSandilands/dotfiles) for an example of dotfiles use. 

### 2.2. Access RStudio Server

Once the container is running and the RStudio Server process has started (`postStartCommand`), access the IDE by navigating to the following URL in your web browser:

```
http://localhost:8787/
```

## 3. Package & Tooling Configuration

The Dev Container environment is configured to prioritize a tidy package development workflow and uses `renv` to manage dependencies.

### 3.1. Initializing a New Tidy Package

If you are starting a **brand new R package project** in the current directory (which is recommended to be empty first), you can use a series of powerful `usethis` commands to scaffold a complete, modern package with Git, GitHub, and Continuous Integration (CI) workflows.

Run these commands sequentially in your RStudio console:

```R
renv::install("devtools") # Ensure devtools is installed in the renv project

usethis::create_tidy_package(getwd()) # Creates the basic package structure in the current directory
usethis::use_git() # Initializes Git repository
gitcreds::gitcreds_set() # Enter your github PAT https://usethis.r-lib.org/articles/git-credentials.html
usethis::use_github() # Links local repo to a new/existing GitHub repository
usethis::use_tidy_github() # Sets up Tidyverse-style GitHub configuration
usethis::use_tidy_github_actions() # Adds GitHub Actions workflows (e.g., R-CMD-check)
usethis::use_tidy_github_labels() # Adds standard GitHub issue labels
usethis::use_pkgdown_github_pages() # Sets up pkgdown for documentation website deployment
```

### 3.2. Enabling Development & AI Tools (Suggests)

To use powerful developer packages like `devtools` and the AI code assistants (`gander`, `chores`, `ellmer`) without making them mandatory dependencies for end-users, they should be added to the `Suggests` field of your `DESCRIPTION` file.

Run the following R code to formally add the packages to your project's metadata:

```R
usethis::use_package("devtools", "Suggests")
usethis::use_package("chores", "Suggests")
usethis::use_package("gander", "Suggests")
usethis::use_package("ellmer", "Suggests")
```

Restart the R session and add shorcuts for the AI tools:

`Tools > Modify Keyboard Shortcuts > Search "Chores"` Set this to `Ctrl+Alt+c`

`Tools > Modify Keyboard Shortcuts > Search "gander"` Set this to `Ctrl+Alt+g`

### 3.3. Add Test Coverage (covr)

Create a [Code Coverage](https://about.codecov.io/) account and link it to your Github account. 

In the root of your R package create a `.travis.yml` file and populate it with the following:

```
language: r

r_packages:
  - covr

after_success:
  - Rscript -e 'library(covr); codecov()'
```

Add `covr` to your project's metadata with this R code:

```R
usethis::use_package("covr", "Suggests")
```

Add test coverage to your project with the following R code:

```R
usethis::use_coverage()
```

### 3.4. Automated Startup (.Rprofile Content)

The following code should be placed in your project's `.Rprofile` to ensure `devtools` is loaded and the AI tools are connected automatically.

```R
# Activate renv
source("renv/activate.R")

# --- AI Tool Setup ---
# Check if {chores}, {gander} and {ellmer} are installed and an API key is present
if (requireNamespace("chores", quietly = TRUE) &&
    requireNamespace("gander", quietly = TRUE) &&
    requireNamespace("ellmer", quietly = TRUE) &&
    Sys.getenv("ANTHROPIC_API_KEY") != "") {
  
  # Set the {chores} chat option to use the {ellmer} OpenAI chat model
  options(.chores_chat = ellmer::chat_anthropic())
  options(.gander_chat = ellmer::chat_anthropic())
}

# Load {devtools} if it is installed for development functions (e.g., check, document, load_all)
if (requireNamespace("devtools", quietly = TRUE)) { 
  library(devtools) 
}
```

Again, **make sure you add .Renviron to .gitignore.**
