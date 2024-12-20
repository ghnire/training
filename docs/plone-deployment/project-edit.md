---
myst:
  html_meta:
    "description": "A comprehensive guide on customizing and enhancing your Plone project for deployment."
    "property=og:description": "Learn how to edit, customize, and enhance your Plone project for optimal deployment."
    "property=og:title": "Customizing Your Plone Project for Deployment"
    "keywords": "Edit, Plone, Project, Add-ons, Volto, OAuth, GitHub"
---

# Customize Your Project

Plone offers a wealth of features right out of the box. You can extend these capabilities using {term}`TTW` modifications, such as creating new content types, altering the default workflow, or configuring the top-level navigation. For additional functionalities not covered by Plone, you can either develop your own solutions or integrate existing add-ons.

## Integrating Add-ons

Both Plone Frontend and Backend in your project support add-on integration. Add-ons can be specific to either the Frontend or Backend, or they can be collaborative packages enhancing both components.

- **Plone Backend Add-ons**: These are Python packages available on PyPI. [Awesome Plone](https://github.com/collective/awesome-plone) offers a curated list of these add-ons.
- **Plone Frontend Add-ons**: Written in JavaScript or TypeScript, these are released as NPM packages. Check out [Awesome Volto](https://github.com/collective/awesome-volto) for a collection of Frontend add-ons.

### Adding a New Block to the Frontend

We'll illustrate the process of integrating a Frontend add-on, `@plonegovbr/volto-code-block`, which renders highlighted source code blocks. We'll centralize all modifications within our project's add-on, `volto-ploneconf2024`, to streamline future Volto version upgrades.

#### Incorporating a New Dependency

Edit {file}`frontend/packages/volto-ploneconf2024/package.json` and append `@plonegovbr/volto-code-block` to the `addons` and `dependencies` sections, as shown below:

```json
"addons": [
  "...more add-ons",
  "@plonegovbr/volto-code-block"
],
"dependencies": {
  "...more dependencies": "*",
  "@plonegovbr/volto-code-block": "*"
}
```

#### Reinstalling the Project

Execute `make frontend-install` from the repository root to install the new add-on and update {file}`frontend/pnpm-lock.yaml`.

#### Restarting the Project

Start your project with `make backend-start` and `make frontend-start` in different shells.
Navigate to http://localhost:3000.
After authentication, the new block becomes available on the content edit page.

#### Committing Changes

Format your codebase with `make check`, then commit and push the changes:

```shell
git add frontend/packages/volto-ploneconf2024/package.json frontend/pnpm-lock.yaml
git commit -m "Add @plonegovbr/volto-code-block"
git push
```

## Implementing OAuth Support with GitHub

We'll now add GitHub OAuth authentication, involving both Backend and Frontend add-ons and GitHub OAuth application creation.

### Creating a GitHub OAuth Application

Follow GitHub's guide on [Creating an OAuth App](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app), using the following configurations:

-   {guilabel}`Application Name`: `plone-conference-local`
-   {guilabel}`Homepage URL`: `http://ploneconf2024.localhost`
-   {guilabel}`Application Description`: `Plone Conference 2024`
-   {guilabel}`Authorization Callback URL`: `http://ploneconf2024.localhost/`

### Backend: Installing `pas.plugins.authomatic`

Modify {file}`backend/setup.py` to include `pas.plugins.authomatic` in `install_requires`. Also, update {file}`backend/src/ploneconf2024/dependencies.zcml` to load the package configuration during Plone Backend startup.

### Frontend: Installing `volto-authomatic`

Add `@plone-collective/volto-authomatic` to the `addons` and `dependencies` sections of {file}`frontend/packages/volto-ploneconf2024/package.json`. Run `make frontend-install` to update the dependencies.

### Activating and Configuring the Add-on

````{note}
To ensure the name `ploneconf24.localhost` points to the address `127.0.0.1`, we need to edit the file located at {file}`C:\Windows\System32\Drivers\etc\hosts`.

1.  Open the {guilabel}`Start Menu` and search for "Notepad".
    Right-click on it and choose {guilabel}`Run as administrator`.
    If Windows asks if you want the application to make changes to the system, click {guilabel}`Yes`.
    (If you don't open Notepad as an administrator, you won't be able to modify the `hosts` file.)

2.  In Notepad, click {menuselection}`File --> Open`.
    Then navigate to the folder {file}`C:\Windows\System32\Drivers\etc`.

3.  In the file dialog, change the filter from {guilabel}`Text Documents (*.txt)` to {guilabel}`All Files (*.*)`.
    You should now see the `hosts` file.
    Select it and click {guilabel}`Open`.

4.  At the end of the file, add the following line:

    ```text
    127.0.0.1 ploneconf24.localhost
    ```

5.  Save the file.

6.  Open PowerShell and confirm that the name resolution is working correctly by typing:

    ```shell
    ping ploneconf24.localhost
    ```
````

Start the Docker Compose stack with `make stack-start`. Navigate to http://ploneconf2024.localhost/ClassicUI/login.

Install `pas.plugins.authomatic` from the {guilabel}`Add-ons` control panel and configure it with the following JSON configuration, replacing `KEYHERE` and `SECRETHERE` with your GitHub OAuth application's client ID and secret.

```json
{
  "github": {
    "display": {
      "title": "Github",
      "cssclasses": {
        "button": "btn btn-default",
        "icon": "glypicon glyphicon-github"
      },
      "as_form": false
    },
    "propertymap": {
      "email": "email",
      "link": "home_page",
      "location": "location",
      "name": "fullname",
      "picture": "avatar_url"
    },
    "class_": "authomatic.providers.oauth2.GitHub",
    "consumer_key": "KEYHERE",
    "consumer_secret": "SECRETHERE",
    "access_headers": {
      "User-Agent": "Plone (pas.plugins.authomatic)"
    }
  }
}
```

### Authenticating with GitHub

Visit http://ploneconf2024.localhost/login and log in using GitHub.

### Configure the OAuth user permissions

By default, `pas.plugins.authomatic` will not assign any role or group to an user authenticated with GitHub, but you can, using an existing user such as `admin`, assign the OAuthed user to groups and grant permissions.

Back in the tab you are using to access the `ClassicUI`, visit the control panel {menuselection}`Users --> Users`.
For the {guilabel}`User name` row, check appropriate permissions.
For the purpose of this training, check {guilabel}`Manager`, then click {guilabel}`Apply changes`.

```{warning}
If you need to authenticate bypassing OAuth, there are fallback login forms:
-   Backend: Available at http://ploneconf2024.localhost/ClassicUI/failsafe_login.
-   Frontend: Available at http://ploneconf2024.localhost/fallback_login.

```
