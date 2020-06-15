# Migrating your GitHub Default Branch

Migrating your GitHub default branch is not difficult.

NOTE: For clarity, we assume a migration from `master` to `main`, but the advice and tool are useful for any value of `$previous_default_branch` and `$new_default_branch`

## Simple Repositories

For most repositories (no CI tooling and no backlog of open PRs):
1. [Create the new branch via the UI](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-and-deleting-branches-within-your-repository#creating-a-branch)
	1. On GitHub, navigate to the main page of the repository.
	2. Branches link on overview page
	3. Click the branch selector menu.
	4. Type the name of your new branch, then select Create branch.
	
	![Create New Branch in GitHub UI](https://user-images.githubusercontent.com/328073/84691723-9ad2cd80-af12-11ea-98bb-68dd98bdf55a.png)

1. Alternatively, you can use the command line

	```bash
	> git checkout -b main master
	> git push origin main
	```

1. [Change the default branch](https://help.github.com/en/github/administering-a-repository/setting-the-default-branch)
	1. Click `Settings` in the top repository menu (to the far right of code, issues, pull requests, etc)
	2. On the left hand menu, click `branches`
	3. Change the default branch in the dropdown

## Complex Repositories

You may need to be more thoughtful if your repository is slightly more complex:
- You have CI that runs off master
- You have existing PRs that need to be updated

### Strategy

1. Create `main` branch and have it mimic `master`. Whenver a change is merged into `master`,
`main` will be updated as well.
	1. This GitHub action will automatically create the `main` branch
	2. On every merge to `master`, this GitHub action will update `main` to the same commit.
2. Then, you can update any Continuous Integration/Continuous Deployment scripts to point to `main`.
	1. This is specific to your repository and setup
	2. This can happen at your own pace
	3. Don't manually merge anything into `main` during this time. All developers should continue to use `master`.
3. Once everything is updated to work with `main`, switch the default branch in the GitHub UI.
	1. See step 3 from Simple Repositories above.
	2. All developers should now use `main` instead of `master`
4. Update all open Pull Requests that are based on `master` to be based on `main`.
	1. This GitHub action will update all pull requests based on `master` to be based on `main` when:
		1. The default branch is correctly set to `main` in GitHub
		2. A commit is merged to `main`
5. Congratulations
	1. You can uninstall this action from your repo
	
## Usage
1. Create a file: `.github/workflows/default-branch-migration.yml`
2. Copy and paste the following:

```yaml
name: Default Branch Migration
on: [push]
# The script does nothing if the push isn't to one of the default branches
# However, it still runs before exiting. If you would like to not even start it,
# you can specify the two branches here, uncomment it, and remove the `on: [push]`
# on:
#   push:
#     branches:
#       # TODO: Change this to the default branch you want to migrate away from
#       - master
#       # TODO: Change this to the default branch you want to migrate to
#       - main

jobs:
  migrate_branch:
    name: Migrate Branch
    runs-on: ubuntu-latest
    steps:
      - name: Migrate
        uses: liyanchang/default-branch-migration@v1.0.0
        with:
          # GitHub will fill in this template with the correct token
          # You don't need to edit this line
          github_token: ${{ secrets.GITHUB_TOKEN }}
          # TODO: Change this to the default branch you want to migrate away from
          previous_default: master
          # TODO: Change this to the default branch you want to migrate away to
          new_default: main
```
