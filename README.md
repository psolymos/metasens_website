# Shiny app

This is a Shiny web application. You can run the application locally by clicking
the 'Run App' button after opening any of the following files in RStudio IDE:
`global.R`, `ui.R`, or `server.R`.

## Known issues

- Error in `evalues.RR`: Lower confidence limit should be less than upper confidence limit:
  probably need to `observe()` `input$lo.RR` and `input$hi.RR` or use `renderUI`.

## Deployment workflow

### Organizing the app for Docker based deployment

Revised the files:

- `startup.R` is now `global.R`,
- deleted `app.R`, no need for that when there is global/ui/server files
- made `server.R` to contain `server <- function() {...}`
- made `ui.R` to contain `ui <- navbarPage(...)`
- moved the app specific files into the `/app` folder so that nothing else gets copied into the Docker image
- added counter set at 50 seconds interval is added to `/app/server.R` (see end of the file) to prevent [time out after 55 seconds](https://devcenter.heroku.com/articles/limits#http-timeouts)
- added `Dockerfile`: see comments inside the file and prompts for where to edit.
- run `renv::init()` and/or `renv::snapshot()` to capture dependencies in the `renv.lock` file


### Heroku deployment examples

This workflow works with public and privare repositories.

1. Log into Heroku, in the dashboard, click on 'New' then select 'Create new App'.
Give a name (e.g. `shiny-example`, if available, this will create the app at https://shiny-example.herokuapp.com/) to the app and create the app.
2. In your Heroku dashboard, go to your personal settings, find your API key, click on reveal and copy it, you'll need it later.
3. Go to the Settings tab of the GitHub repo, scroll down to Secrets and add the
following new repository secrets:
  - `HEROKU_EMAIL`: your Heroku email that you use to log in
  - `HEROKU_APP_NAME`: you application name from above (shiny-example)
  - `HEROKU_API_KEY`: your Heroku api key, you can find it under your personal settings, click on reveal and copy
4. Trigger the GitHub action by a new commit to the repo.

See the `.github/workflows/deploy.yml` file for additional options
(`dockerfile_name`, `docker_options`, `dockerfile_directory`).

Note: the GitHub action responsible for building and deploying the app to Heroku is failing when the app is not in the root directory.
The work around is to copy the contents of the app folder into the root
when building the image for Heroku, see the Change directory step in the
`deploy.yml` file.

### Having multiple Shiny apps in the same GitHub repo

It is possible to copy the `deploy.yml` file (i.e. `deployApp2.yml`, etc.)
in the `.github/workflows` folder. This will run multiple jobs.
But this would lead to unnecessary re-deployments of apps that have not
changed and are functioning fine.

- Solution 1: add new apps as new git branches and make the action specific to that branch.
- Solution 2: add new apps as new GitHub repos (sharing a prefix in their names).

### Possible issues when applying this approach to other apps

System requirements need to be updated if a package installation fails.
E.g. I had to add `libxml2-dev` for the xml2 package.

Solutions:

- find the error log in GitHub action output: `Error: install of package 'xml2' failed`
- find system dependencies (often listed near the error): `Configuration failed because libxml-2.0 was not found. Try installing: deb: libxml2-dev (Debian, Ubuntu, etc)`
- if it is not listed, see https://github.com/rstudio/r-system-requirements or https://github.com/r-hub/sysreqsdb
