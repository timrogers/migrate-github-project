# Migrate GitHub Project

Migrate organization-owned [GitHub Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects) between GitHub products (e.g. GitHub Enterprise Server to GitHub.com) and organizations (e.g. classic GitHub.com organization to [Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/identity-and-access-management/using-enterprise-managed-users-for-iam/about-enterprise-managed-users) organization).

## Requirements

To use `migrate-github-project`, you must be running a [supported release of Node.js](https://github.com/nodejs/release#release-schedule). At the time of writing, v16, v18 and v20 are supported.

## Limitations

The following data is not migrated and will be skipped:

* Views
* The order of project items displayed in your views
* Workflows
* Iteration custom fields
* Draft issues' assignees

There are a few other limitations you should be aware of:

* `migrate-github-project` can only migrate to and from organization-owned projects. It is not compatible with user-owned projects.
* Migrated draft issues will show as being created by the person who ran the migration at the time they ran the migration. A note will be prepended to the body with original author login and timestamp.

## Instructions

### Step 1. Migrate your issues and pull requests

Items in GitHub Projects are linked to issues and pull requests. Before you can migrate a project with `migrate-github-project`, you need to migrate the relevant issues and pull requests.

If you're migrating from GitHub Enterprise Server to GitHub.com or between organizations on GitHub.com, you can migrate your issues and pull requests using [GitHub Enterprise Importer](https://docs.github.com/en/migrations/using-github-enterprise-importer).

If you're migrating to GitHub Enterprise Server, you can migrate your issues and pull requests using [`ghe-migrator`](https://docs.github.com/en/enterprise-cloud@latest/migrations/using-ghe-migrator/about-ghe-migrator).

When you run your migrations, make a note of the new organization name and repo name for each repository.

### Step 2. Install `migrate-github-project`

Install the Node.js package from `npm` by running `npm install -g migrate-github-project`.

### Step 3: Export your project from migration source

`migrate-github-project` works in two distinct phases: export and import. The first step is to export your project and its items from your migration source.

You can export your project and its items using the `migrate-github-project export` command:

```bash
migrate-github-project export \
    # A GitHub access token with permissions to read the project and any relevant issues and pull requests.
    # This can also be configured using the `GITHUB_TOKEN` environment variable.
    --access-token GITHUB_TOKEN
    # The name of the organization that owns the project you want to export
    --project-owner monalisa
    # The number of the project you want to export
    --project-number 1337
    # OPTIONAL: The base URL of the GitHub API, if you're migrating from a migration source other than GitHub.com.
    --base-url https://github.acme.inc/api/v3
```

When the export finishes, you'll have two files written to your current directory:

* `project.json`: The raw data of your project and all of its project items
* `repository-mappings.csv`: A repository mappings CSV template that you need to fill out with the names of your repositories in the migration target

### Step 4. Complete the repository mappings template

You'll need to complete the `repository-mappings.csv` file outputted from the `export` command with repository mappings, so the tool knows how to match repositories in your migration source to repositories in your migration target.

The CSV will look like this:

```
source_repository,target_repository
corp/widgets,
corp/website,
```

Imagine that you're in the process of migrating from GitHub Enterprise Server to GitHub.com, and you've already moved your repositories into a new GitHub.com organization called `monalisa-emu`. You'd fill out the CSV like this:

```
source_repository,target_repository
corp/widgets,monalisa-emu/widgets
corp/website,monalisa-emu/website
```

If you don't want to map a repository - for example because it hasn't been migrated - just delete the line from the CSV or leave the `target_repository` blank. If a repository hasn't been mapped, project items related to that repository will be skipped during the import.

### Step 5. Import your project into your migration target

You've exported your data and filled out the repository mappings template. You can now import your project into your migration target.

You can import your project using the `migrate-github-project import` command:

```bash
migrate-github-project import \
    # A GitHub access token with read-write Projects permissions in your destination organization
    # This can also be configured using the `GITHUB_TOKEN` environment variable.
    --access-token GITHUB_TOKEN
    # The name of the organization that will own the newly-imported project
    --project-owner monalisa
    # The path of the project data generated by the `export` command
    --input-path project.json
    # The path of the repository mappings file generated by the `export` command and completed by you
    --repository-mappings-path repository-mappings.csv
    # OPTIONAL: The base URL of the GitHub API, if you're migrating to a migration target other than GitHub.com.
    --base-url https://github.acme.inc/api/v3
```

Near the start of the import, the tool will ask you to manually set up your options for the "Status" field. It will explain exactly what to do, and will validate that you've correctly copied the options from your migration source.

Once you've set up the "Status" field, your project will be imported. Watch out for `warn` lines in the logs, which will let you know about data which hasn't been imported.


