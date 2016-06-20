## Release Automation
Note:  Releases for product where you own the server can be simple, recompile component, upload it to the server, done.
For an On Premise application you need to do a simultaneous release of all components, which is only as simple as your application.


#### Release Automation
![Build Infrastructure](images/BuildInfrastructure.png)
Note:  Build infrastructures are pretty simple you have a component build that pulls some dependencies from the component repository, performs the build, runs the quality checks and publishes the new artifact back to the component repository.  Then the product build can pull all of the components and their dependencies from the repository, create the installable product image and push that to the Product repository.
The logic is the same for both snapshot build and release builds.


#### Release Automation
![Example of Product complexity, large interwoven graph](images/ExampleApp.png)
Note:  The problem is we don't have a simple application.  This graph shows a simplified look at our components and how they depend on each other.
I actually had to simplify it significantly to get it to fit on the screen even when ignoring the readability of the text.


#### Release Automation
## On Premise Monolith
* ~38 Repositories
* ~101 Separate Builds
* 100+ individual code contributors
* Manual Release takes ~ 4 man weeks
Note:  We have literally man centuries of development in the product.  And while


#### Release Automation
## Release Process
1. Create Release Branch
1. Update versions on Master
1. Create new Build for Branch
1. Update Dependencies to Remove '-SNAPSHOT'
1. Run Release Build
1. Tag Repository
1. Goto #1 for next repo
Note:  So the release process takes so much time partialy because we can't just release off of the master branch and keep going,
we need to create a support branch for any potential bugs found after release.  So we need to branch all of the repositories,
update the versions on master for the next release, create new a new build for the new branch in our Continuous Integration server,
before we can even start the release process.


#### Release Automation
## Custom Gradle plugin
Note:  At this point it should be fairly obvious that we chose to use a custom gradle plugin to fix this issue.


#### Release Automation
## DSL:
    apply plugin: 'release-automation'

    buildEnvironment {
        ...
    }

    product {
        ...
        vcsRoots {
             ...
        }
        builds {
             ...
        }
    }
Note:  The release automation plugin's DSL is the basis of all automation.  You can't automate what you can't describe.
When you apply the plugin it creates two extensions to the gradle project, one small one that describes the current build environment,
and one large one that completely describes the product being automated.


#### Release Automation
## DSL continued:
    buildEnvironment {
        teamcityUrl = 'https://teamcity.example.com'
        baseTeamCityRestUrl = '/app/rest/8.1'
        stashUrl = 'https://git.example.com'
        mavenUrl = http://artifactory.example.com/repo/
    }
Note:  The build environment extension just declares the server names used to build the product, it doesn't contain any information about the product itself.


#### Release Automation
## DSL continued:
    product {
        name = 'Example Product'
        groupId = 'com.pros.example'
        jdkHome = '%teamcity.JAVA_HOMEx64_8%'
        vcsRoots {
             ...
        }
        builds {
             ...
        }
    }
Note:  The product extension contains all of the variables that are consistent across all components, all of the information that is needed to find the VCS roots and all of the information about the Component builds.


#### Release Automation
## DSL continued:
    vcsRoots {
        stash {
            projectKey = 'EXPRDT'
            branchName = 'master'
            repositoryNames = [
                'server-base-module',
                'html5-library',
                'flex-library',
                ...
                'html5-ui',
                'flex-ui',
                'servlet-app',
                'work-process-app',
            ]
        }
    }
Note:  The VCS Root definition DSL currently only supports the Stash repository which is a GIT server from Atlassian.  This additional layer of confiugration is just a little future proofing due to the fact that we have had three different VCS types in the last 10 years.  If we need to add a different VCS server with different configuration requirements we can just add a new type here.


#### Release Automation
## DSL continued:
    builds {
        gradle {
            ...
        }
        ant {
            ...
        }
        antSuite {
            ...
        }
    }
Note:  The builds extension is a little different than the rest of the extension.  Here the build types are method calls that take a configuration closure, where each method creates the build object and adds it to the list of all objects.  We currently support Gradle builds, Ant builds, and Ant Suite builds.  The Ant Suites are a series of builds that must all pass before a common artifact publish build passes, so when the Ant Suite method is called it actually creates multiple builds.


#### Release Automation
## DSL continued:
    gradle {
        name = 'Common UI Components'
        vcsRootName = 'html5-library'
        scheduleNightlyBuild = true
        triggerDependencies << 'Other build name'
        maxBuildTime = 5
        producedArtifacts = [
            [name: 'reactjs-common', ext: 'zip']
            [name: 'reactjs-common', classifier: 'debug', ext: 'zip']
        ]
    }
Note:  This DSL includes all of the information we need to configure a build and run a release.  We were able to shorten the amount of information required per component by making every build standarize on how their build works.  That includes using a certain set of task names for all CI builds, and controlling their dependencies on other parts of the product using certain known variables instead of hard coding the values.


#### Release Automation
## DSL continued:
    gradle {
        ...
        agentRequirement {
            exists 'system.db.sqlserver.major_version'
            exists 'system.db.oracle.major_version'
        }
        ...
    }
Note:  But the DSL also allows builds to override certain values when required, such as the agent requirements shown here.


##### Release Automation
## CI Server (TeamCity):
* Configuring Builds
* Running Builds
* Monitoring running Builds
* Running Builds in parallel if no dependency exists
* Restarting After Build Failure
Note:


#### Release Automation
## VCS (GIT / Stash)
* Creating a branch
* Updating dependencies on a branch
* Updating versions on a branch
Note:


#### Release Automation
## Success
* ~2 man hours per release
* ~12 hours build time
Note:  Release automation has 41 Java Classes, 613 Groovy classes, 14609 lines of code

