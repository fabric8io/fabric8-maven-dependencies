## Fabric8 Maven Dependencies

This project is used to keep track of common maven dependencies across all the quickstart projects in the [fabric8-quickstarts github organisation](https://github.com/fabric8-quickstarts).

The quickstarts are created as small independent projects which are easy to turn in to maven archetypes; or can be directly forked and used by users. 

To make the quickstarts more usable in different organisations we don't use a common paremt pom.xml as usually different organisations will want to use their own local parent pom.xml to define things like the distribution, common repositories and versions of things and so forth.

### How it works

This project contains a [pom.xml file](https://github.com/fabric8io/fabric8-maven-dependencies/blob/master/pom.xml#L18) which contains a number of [maven properties](https://github.com/fabric8io/fabric8-maven-dependencies/blob/master/pom.xml#L39) for maven dependency versions or maven plugin versions.

If you fork this project and submit a Pull Request to change any of these properties, the PR merge build will trigger a Jenkins pipeline based on the [Jenkinsfile](Jenkinsfile) which will iterate through all projects in the [fabric8-quickstarts organisation](https://github.com/fabric8-quickstarts) and if the project has a pom.xml and has that maven property, the property will be upgraded to the new version via a Pull Request.

This ensures we automatically keep all the quickstart projects in sync with each other; while using the CI builds to check all changes are valid before merging them. This also works for maven dependencies and maven plugin versions too! Ideally we'd use a BOM to simplify managing versions between projects; but BOM's don't help for maven plugin versions.

### Tips on writing a good quickstart

Try to follow the following guidelines:

* Try minimise the absolute number of versions you use in your pom.xml by using the fabric8 BOMs which provide most dependences for you
* Use a maven property for all versions of maven dependencies or plugins. In IDEA you can select an explicit version in a dependency or plugin and select `Refactor -> Extract -> Property` and let it pick the default name for you.
* Try to minimise fragmentation of maven property names; if there is already a property name in the [common maven properties](https://github.com/fabric8io/fabric8-maven-dependencies/blob/master/pom.xml#L39) then please try to use it


### When you create a new quickstart project

Whenever a new quickstart project is created, we need to setup the CI builds for it. So after you have added the new repository to the [fabric8-quickstarts organisation](https://github.com/fabric8-quickstarts) you need to trigger the [fabric8-ci-seed build](https://fabric8-ci.fusesource.com/job/fabric8-ci-seed/) which will auto-create the test and merge CI builds.

Then the next time we build the [ipaas-quickstarts](https://github.com/fabric8io/ipaas-quickstarts) project your new quickstart will get included in the generated archetypes. To add the archetype to the release you need to

* clone and build [ipaas-quickstarts](https://github.com/fabric8io/ipaas-quickstarts)

```
git clone git@github.com:fabric8io/ipaas-quickstarts.git
cd ipaas-quickstarts
mvn install
```

* add the generated archetype to source control

```
cd archetypes
git add my-sample-archetype
```

* add the new archetype module to the [archetypes pom.xml &lt;modules&gt;](https://github.com/fabric8io/ipaas-quickstarts/blob/master/archetypes/pom.xml#L35)
* commit and win!