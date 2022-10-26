# Migrate existing CI/CD pipelines to GitHub and Improve Build/Test/Deployment workflows @ MetaCall, GSoC 2022

The CI/CD pipelines were earlier hosted on GitLab, using their built-in GitLab CI. This project aims to migrate the existing pipelines completely to GitHub Actions to centralize MetaCall's entire dev-ops infrastructure on GitHub, optimize, fix, and improve existing workflows, and create new pipelines to automate the development, building, testing, and deployment of the projects.

## What is MetaCall?

MetaCall is a polyglot runtime, which enables us to use interoperate code written in different programming languages. One possible use-case would be writing a function in Python and calling it in JavaScript. It is as seamless as simply importing the function inside NodeJS using the traditional CommonJS import syntax!

Additionally, MetaCall has a FaaS service, which can be used to deploy MetaCall-compatible functions on the cloud in the form of serverless function APIs.

## Migrating the Release Pipeline for the Linux Distributable

The first pipeline I migrated from GitLab CI to GitHub Actions was the one for MetaCall's Linux Distributable. The pipeline was supposed to build the distributable and release it to the GitHub repository of the project using [GHR](https://deeeet.com/ghr/), an open source command-line tool to manage GitHub releases.

Here's GitLab CI pipeline that was being used before the migration: https://github.com/metacall/distributable-linux/blob/v0.5.5/.gitlab-ci.yml

One of the biggest benefits to having the workflow on GitHub Actions was that we no longer needed to set any repository variables – It was already achieved using default variables and secrets available in GitHub Actions!
- `secrets.GITHUB_TOKEN` : A context variable that returns an access token.
- `GITHUB_REPOSITORY` : A default variable, containing a string of the form `username/repository-name`.

Persisting environment variables across steps in GitHub Actions is quite different from simply exporting them:
```bash
# --- Single-line values ---
export VARIABLE=value                # Traditional
echo "VARIABLE=value" >> $GITHUB_ENV # GitHub Actions

# --- Multi-line values ---

# Traditional
export VARIABLE="This
is
a
multiline
value
"

# GitHub Actions
echo "VARIABLE<<EOF" >> $GITHUB_ENV # Terminates at EOF
echo "This"          >> $GITHUB_ENV # Value begins
echo "is"            >> $GITHUB_ENV
echo "a"             >> $GITHUB_ENV
echo "multiline"     >> $GITHUB_ENV
echo "value"         >> $GITHUB_ENV # Value ends
echo "EOF"           >> $GITHUB_ENV # Terminator
```

The new pipeline builds the distributable, executes the tests, and publishes it to GitHub Releases, and was merged under [distributable-linux#9](https://github.com/metacall/distributable-linux/pull/9) and is now in "action": https://github.com/metacall/distributable-linux/blob/master/.github/workflows/ci.yml

## Migrating the Release Pipeline for the Core

The core ([metacall/core](https://github.com/metacall/core)) is the heart of MetaCall – This is where the open source runtime lives! It contains the source code for different runtime definitions and ports needed by other MetaCall projects (such as the CLI) to actually perform the interoperability. In simple words, if your application uses MetaCall, you are bound to land here before anywhere else.

For every tag pushed/created, there are supposed to be .DEB, .RPM, and .TAR.GZ releases for the following:
- The development build.
- The examples.
- The runtime.

The existing workflow used Travis CI: https://github.com/metacall/core/blob/v0.5.20/.travis.yml

Deployment to Bintray and the NPM Registry were not in effect, so the workflow was disabled. We needed to write one that works using GitHub Actions to recreate the Travis CI workflow. It should also generate a development log, similar to the one generated in the Linux Distributable, to be added to the release description.
```bash
# Outputs the Markdown-compatible log
git log --no-merges --format="- %s" ${PREVIOUS_TAG}..HEAD
```

Building was entirely encompassed in a script named `docker-compose.sh`, and once the artifacts are generated, I used GHR to release them to the repository, once again, without needing absolutely any variables to be added. The new pipeline was merged under [core#293](https://github.com/metacall/core/pull/293) and is now operating: https://github.com/metacall/core/blob/develop/.github/workflows/release.yml

## Writing a Version-Updating Hook for the TypeScript/JavaScript Projects

In projects that require versioning, it is always a hassle to synchronize the package version with the latest repository tag. For NodeJS-based projects, it is thrice the trouble.

"Thrice" is not a figurative expression here, because for every version update done using NPM, it updates the current package version in three different locations:
- `package.json` : Under the `version` property.
- `package-lock.json` : Under the `version` property.
- `package-lock.json` : Under the `version` property that resides under the `""` (an empty string) property of the `packages` sub-object.

In the core, we are handling this using the `CMakeLists.txt` file, by customizing the directory for the Git hooks, on a repository level: https://github.com/metacall/core/blob/develop/CMakeLists.txt#L327-L333

The version itself is stored in a `VERSION` file in the root of the repository: https://github.com/metacall/core/blob/develop/VERSION

Before working on this, my only belief was that all hooks must reside inside the `.git/hooks/` directory within the repository, and the fact that it cannot be included inside the repository to be stored remotely is why we have libraries like [Husky](https://typicode.github.io/husky/#/). I learnt that it is no longer needed to rely on external dependencies as such because we can now use a modern Git feature to customize the directory:
```bash
# This sets '/githooks' as the directory for the Git hooks in the current repository
git config --local core.hooksPath githooks
```

To be open to the possibility of having other pre-push hooks in the repositories, I generalized the hook to look like this:
```js
... // -- snip

const hooks = [
	'update-version',
	'other-hook-1',
	'other-hook-2',
	'other-hook-3',
	... // More hooks
];

... // -- snip

	for (const hook of hooks) {
		const filename = PREFIX + '-' + hook;
		debugLog('Executing hook:', filename);
		const { main } = require('./' + filename);
		main(process.argv.slice(2));
	}

... // -- snip
```

I proceeded to add a pre-push hook that updates the versioning 'triple-trouble' of the current package. I wrote the script in JavaScript: https://github.com/metacall/deploy/blob/master/githooks/pre-push-update-version.js

The hook flow goes like this:
1. Synchronize the tags from remote.
2. Fetch the latest tag.
3. Match the versions in the three places and update wherever needed.
4. Drop the tag.
5. Commit the changes.
6. Tag the latest commit.

After some testing, we finalized the hook and added it to the Deployment CLI ([metacall/deploy](https://github.com/metacall/deploy)) and the Protocol ([metacall/protocol](https://github.com/metacall/protocol)), both TypeScript-based. The following PRs contributed the hooks to the repositories:
- [protocol#25](https://github.com/metacall/protocol/pull/25) (closed, updated on `master`)
- [deploy#88](https://github.com/metacall/deploy/pull/88)
- [deploy#89](https://github.com/metacall/deploy/pull/89)

The hooks are now in effect. This is what the `githooks` directory looks like: https://github.com/metacall/deploy/tree/master/githooks

## Writing a Lint, Build, Test, and Release Workflow for the New FaaS

The new FaaS ([metacall/faas](https://github.com/metacall/faas)), currently in development, is made in TypeScript too. We have yet to reach the first milestone, but to ensure the development is smooth and bug-free using CI/CD technologies, we plan to add testcases that can help us verify the project works smoothly after the updates we make.

Adding a linting, building, testing, and releasing workflow to the new FaaS still required some refactoring to be done, because it was already failing to work. So, I made some small refactoring updates:
- Renamed the package from `metacall-faas` to `@metacall/faas`.
- Fixed the lockfile errors.
- Resolved TypeScript checks.
- Ran the linter.

The new pipeline has been added to the repository. There are unit tests in plan for future, once there's more development of the new FaaS. Here are the PRs:
- [faas#7](https://github.com/metacall/faas/pull/7) (closed, concatenated to #8)
- [faas#8](https://github.com/metacall/faas/pull/8)

## Writing the Linux Test Workflow for the Core

The tests, when run using the existing `docker-compose.sh` helper, were yielding errors, including potential false positives due to incomplete/invalid tests, which I reported under an issue and has been resolved at the time of writing: https://github.com/metacall/core/issues/285

The premise was:
- Understanding the Docker Compose behaviour within the project.
- Analyzing the following images and their respective tasks/purposes:
  - `deps` : For configuring the environment.
  - `dev` : For actually building the core, and then running the tests.
  ```yml
  deps: # Environment-creation
    image: metacall/core:deps
    build:
      args:
        METACALL_INSTALL_OPTIONS: base python ruby netcore5 nodejs typescript file rpc wasm java c cobol rust rapidjson funchook swig pack backtrace # clangformat v8rep51 coverage
  dev: # Configuration, building, and running the tests
    image: metacall/core:dev
    build:
      args:
        METACALL_BUILD_TYPE: debug
        METACALL_BUILD_OPTIONS: ${METACALL_BUILD_SANITIZER} python ruby netcore5 nodejs typescript file rpc wasm java c cobol rust examples tests scripts ports dynamic install pack benchmarks # v8 coverage
  ```
- Implement the environment-creation, configuration, and test/build steps in a single pipeline for Linux.

This was one of the most challenging and the second most engaging (the first being the Windows test workflow) task, and I worked in constant peer sessions with Vicente. A big realization that led us to was the fact that GitHub Actions uses `n` to manage Node versions by default, and it was definitely a conflicting environment for us to build the core in, so I resolved that by getting rid of `n` before any subsequent steps.

One the environment-creation and configuration steps were done, we worked on getting the build step to work. Yet, most tests were failing. Still, the fact that they were executing successfully was a moment of joy for us. Fixing the tests is secondary; Getting them to run successfully the goal. It involved the following two PRs:
- [core#308](https://github.com/metacall/core/pull/308)
- [core#309](https://github.com/metacall/core/pull/309)

## Migrating the Build and Push Pipeline for Guix Image

The Docker image being used for building MetaCall in all its CI/CD environments is located at [metacall/guix](https://github.com/metacall/guix). For the building and pushing to Docker Hub, we were previously relying on Travis CI.

Migrating the workflow wasn't a piece of cake, because Guix runs very slow, and oftentimes I had to wait for an hour or two to get to know the outcome (mostly a failure) of the workflow I wrote. After lots of debugging and trial-and-error sequences, we decided to turn off the certificate and version validation of downloads inside Guix for the CI to work. The work was done in the following PR: [guix#3](https://github.com/metacall/guix/pull/3)

## Writing the Windows Test Workflow for the Core

As we notice, the convention to centralize the environment-creation, configuration, and build/test steps in Linux was under the following Shell scripts:
- `tools/metacall-environment.sh` : To download and install all build and test dependencies/libraries.
- `tools/metacall-configure.sh` : To generate CMake configuration to recognize the system and locate the dependencies.
- `tools/metacall-build.sh` : To execute the build and execution steps.

Each of these scripts received a list of arguments upon execution that tells them exactly which modules or boxes to choose and operate on. For each module, we would have functions inside the scripts to serve the respective purpose. For doing the same on Windows, I had two choices: Using Batch (Command Prompt) scripts, or PowerShell scripts. I chose the latter because of the syntactical advantage, but it obviously came with problems of its own, because we lost the simplicity of scripting that Bash and Batch come with, and have to now deal with several unnecessary abstractions.

Following the pattern, I created PowerShell scripts for similar purposes that would work on Windows:
- `tools/metacall-environment.ps1`
- `tools/metacall-configure.ps1`
- `tools/metacall-build.ps1`

Recreating similar logic in a different syntax taught me a lot about PowerShell, scopes, variables, et alia. The following technologies were being built in the Windows Distributable ([metacall/distributable-windows](https://github.com/metacall/distributable-windows)):
- Python
  - Loaders
  - Ports
- Node
  - Loaders
  - Ports
- C#/.NET Loaders
- Ruby Loaders
- TypeScript Loaders

Following the supported libraries and technologies, I successfully implemented the environment-creation, configuration, and build steps for the following:
- Python
- Node
- Ruby
- .NET

Additionally, unlike the distributable, we also run the tests and build the `scripts` module here. For passing arguments, when PowerShell's capabilities gave up, we took the support of `cmd.exe` to pass Batch-like commands with arguments. It helped and we got the tests to work: https://github.com/metacall/core/blob/develop/.github/workflows/windows-test.yml

The whole job was done in constant co-ordination with Vicente and a lot of PRs were involved in merging stable changes:
- [core#329](https://github.com/metacall/core/pull/329)
- [core#336](https://github.com/metacall/core/pull/336)
- [core#339](https://github.com/metacall/core/pull/339)
- [core#344](https://github.com/metacall/core/pull/344)
- [core#346](https://github.com/metacall/core/pull/346)
- [core#349](https://github.com/metacall/core/pull/349)
- [core#350](https://github.com/metacall/core/pull/350)

During the process, we came across errors with respect to NodeJS, Python, Ruby, and TypeScript tests that required manual fix. For NodeJS and Python, I created the following issues, both of which have been resolved by Vicente at the time of writing:
- [core#328](https://github.com/metacall/core/pull/328)
- [core#333](https://github.com/metacall/core/pull/333)

## Pull Requests

Below is an sequential aggregation of all pull requests made for each of the above-mentioned tasks:

<table>
	<thead>
		<th>Task</th>
		<th>Pull Request(s)</th>
	</thead>
	<tbody>

<tr>
<td>
Migrating the Release Pipeline for the Linux Distributable
</td>
<td>

[distributable-linux#9](https://github.com/metacall/distributable-linux/pull/9)

</td>
</tr>

<tr>
<td>
Migrating the Release Pipeline for the Core
</td>
<td>

[core#293](https://github.com/metacall/core/pull/293)

</td>
</tr>

<tr>
<td>
Writing a Version-Updating Hook for the TypeScript/JavaScript Projects
</td>
<td>

[protocol#25](https://github.com/metacall/protocol/pull/25) (closed, updated on `master`)  
[deploy#88](https://github.com/metacall/deploy/pull/88)  
[deploy#89](https://github.com/metacall/deploy/pull/89)  

</td>
</tr>

<tr>
<td>
Writing a Lint, Build, Test, and Release Workflow for the New FaaS
</td>
<td>

[faas#7](https://github.com/metacall/faas/pull/7) (closed, concatenated to #8)  
[faas#8](https://github.com/metacall/faas/pull/8)  

</td>
</tr>

<tr>
<td>
Writing the Linux Test Workflow for the Core
</td>
<td>

[core#308](https://github.com/metacall/core/pull/308)  
[core#309](https://github.com/metacall/core/pull/309)  

</td>
</tr>

<tr>
<td>
Migrating the Build and Push Pipeline for Guix Image
</td>
<td>

[guix#3](https://github.com/metacall/guix/pull/3)

</td>

<tr>
<td>
Writing the Windows Test Workflow for the Core
</td>
<td>

[core#329](https://github.com/metacall/core/pull/329)  
[core#336](https://github.com/metacall/core/pull/336)  
[core#339](https://github.com/metacall/core/pull/339)  
[core#344](https://github.com/metacall/core/pull/344)  
[core#346](https://github.com/metacall/core/pull/346)  
[core#349](https://github.com/metacall/core/pull/349)  
[core#350](https://github.com/metacall/core/pull/350)  

</td>
</tr>

</tbody>
</table>

## Future Work

> Development never ceases; Maintenance is a lifetime process.

Following are the tasks I plan to work on after the end of my summer of code contributions:
- Participate in the development of tests and features for the new TypeScript-based FaaS platform.
- Add more modules to the Windows scripts, such as Java.
- Pair-programme and make the tests pass for the Windows and Linux test workflows in the core.
- Implement a Macintosh (macOS) test workflow for the core.
- Migrate to the use of the GitHub Container Registry instead of Docker Hub to store all Docker images.
- Use GitHub Packages for storing all development and production packages throughout projects. For example, the Git hooks, which are supposed to be consistent throughout all TypeScript/JavaScript projects.
- Use GitHub Artifacts to cache the previous builds, and use Ccache to speed up subsequent builds.
- Help out future contributors to MetaCall by assisting them with projects that I now have experience working with.

## Conclusion

This year's summer was one of my deepest dives into the development and maintenance of production-level open source software. Under the guidance of Vicente and Harsh, I developed the ability to work on new technology without letting my confidence down. With the constant availability of the mentors, I never felt alone or unguided in my struggles, for which I am very grateful. On behalf of MetaCall, I recently gave a talk with Raj, a fellow contributor, on [Hacktoberfest with MetaCall](https://youtu.be/rK6Eg5AHQds). The summer brought me technical, social, and emotional growth as an engineer with my contributions to MetaCall, and it would be an unforgettable experience for me.