## License Compliance
![](images/GplFail.png)
Note:  As we evolve the product to use new techniques there is a constant churn of new libraries being added to the product.
With the number of committers there is always a chance of someone making a mistake and adding a libraries that has an
incompatible license.


#### License Compliance
## Commercial Offerings
# $$$
Note: There are commercial offerings that claim to be able to audit your licenses for you, but they are rather expensive.


#### License Compliance
## Violating Terms
# $$$$$
Note:  Of course the cost adding a library with an incompatible license would be even higher.


#### License Compliance
## Artifact Repository
Note:  The first way we can ensure no violations it to use an in house Artifact Repository.  No product can ship
or be deployed that pulls from Maven Central or any other public repository.  But there is still the problem that our
in house Repository contains libraries with incompatible licenses.  We are allowed and encouraged to use GPL code internally
for development and testing, we just can't use it in shippable code.


#### License Compliance
## Artifact Classifiers
    'org.example : service : 1.0 : license'
Note:  So we just add the full license for the artifact to the same coordinates with a classifier of "license".  We can then check all dependencies that ship to customers, if this artifact exists is can ship, if not fail the build.  Plus we can download the "license" object to ship with the product.


#### License Compliance
## Artifact Classifiers
    'org.example : service : 1.0 : license-nodist'
Note:  If we add a library that shouldn't ship we still upload the license text, but change the classifier to include nodist.


#### License Compliance
## Example Task
    task checkJavaLicenses(type: com.pros.gradle.LicenseCheckTask) {
        licenseDir = file("build/config/copyright")
        repositoryURL = 'http://maven.example.com/repo'
        configuration = project.configurations.java
    }
Note:
        // The directory that the licenses will be downloaded into
        // optional: defaults to "${project.buildDir}/resources/main")
        // The base url for the repository containing the licenses
        // The configuration to check that every artifact is licensed for distribution


#### License Compliance
## Task Implementation
    List<String> missingLicenses = [];
    configuration.resolvedConfiguration.resolvedArtifacts
        .collect { dependency -> dependency.moduleVersion.id }
        .each { id ->
            String url =  getLicenseURL(id.group, id.name, id.version);
            try {
                ... Copy URL to file
            } catch (FileNotFoundException e) {
                missingLicenses.add(id.toString());
            }
        }
    assert missingLicenses.isEmpty() : ... Print nice message here
Note:

