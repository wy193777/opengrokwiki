# General

- do not add binary files to the repository, all dependencies should be automatically downloadable.

# Building

See [[Developer intro|Developer-intro]]

# Unit testing

**Note**: For full coverage report, a proper junit test environment is required.
That would mean:

  * Your `PATH` must contain directory with Universal ctags binary
    * **Note**: make sure that the directory which contains Universal ctags binary
      is prepended before the directory with plain ctags program.
  * Your `PATH` variable must contain directories which contain binaries of
    appropriate SCM software which means commands `hg`, `sccs`, `cvs`, `git`, `bzr`, `svn`
    (`svnadmin` too). They must be available for the full report.

The tests are normally run in the usual Maven phases.

# Using Findbugs

If you want to run Findbugs (<http://findbugs.sourceforge.net/>) on OpenGrok,
you have to download Findbugs to your machine, and install it where you have
checked out your OpenGrok source code, under the `lib/findbugs` directory,
like this:

```bash
cd ~/.ant/lib
wget http://..../findbugs-x.y.z.tar.gz
gtar -xf findbugs-x.y.z.tar.gz
mv findbugs-x.y.z findbugs
```

You can now run ant with the findbugs target:

```
$ ant findbugs
...
findbugs:
 [findbugs] Executing findbugs from ant task
 [findbugs] Running FindBugs...
 [findbugs] Warnings generated: nnn
 [findbugs] Output saved to findbugs/findbugs.html
```

Now, open `findbugs/findbugs.html` in a web-browser, and start fixing bugs!

If you want to install findbugs some other place than `~/.ant/lib`, you can
untar the `.tar.gz` file to a directory, and use the `findbugs.home` property to
tell ant where to find findbugs, like this (if you have installed fundbugs
under the lib directory):

```bash
ant findbugs -Dfindbugs.home=lib/findbug
```

There is also a `findbugs-xml` ant target that can be used to generate XML files
that can later be parsed, e.g. by Jenkins.

# Using Jacoco

If you want to check test coverage on OpenGrok, download jacoco from
<http://www.eclemma.org/jacoco/>. Place `jacocoagent.jar` and `jacocoant.jar` in the
`opengrok/lib`, `~/.ant/lib` or into classpath (`-lib` option of ant).

Now you can instrument your classes and test them run:

```bash
ant -Djacoco=true -Djacoco.home=/<path_to>/jacoco jacoco-code-coverage
```

Now you should get output data in `jacoco.exec`

Look at `jacoco/index.html` to see how complete your tests are.

# Using Checkstyle

To check that your code follows the standard coding conventions,
you can use checkstyle from <http://checkstyle.sourceforge.net/>

First you must download checkstyle from <http://checkstyle.sourceforge.net/>.
You need Version 6.8 (or newer). Extract the package you have
downloaded, and create a symbolic link to it from `~/.ant/lib/checkstyle`,
e.g. like this:

```bash
cd ~/.ant/lib
unzip ~/Desktop/checkstyle-7.6.zip
ln -s checkstyle-7.6 checkstyle
```

You also have to create symbolic links to the jar files:

```bash
cd checkstyle
ln -s checkstyle-7.6-all.jar checkstyle-all.jar
```

To run checkstyle on the source code, just run ant checkstyle:

```bash
ant checkstyle
```

Output from the command will be stored in the checkstyle directory.

If you want to install checkstyle some other place than `~/.ant/lib`, you can
untar the `.tar.gz` file to a directory, and use the `checkstyle.home` property
to tell ant where to find checkstyle, like this (if you have installed
checkstyle under the lib directory):

```bash
ant checkstyle -Dcheckstyle.home=lib/checkstyle
```

# Using PMD and CPD

To check the quality of the OpenGrok code you can also use PMD
from <https://pmd.github.io/>.

How to install:

```bash
cd ~/.ant/lib
unzip ~/Desktop/pmd-bin-5.5.4.zip
ln -s pmd-5.5.4/ pmd
```

To run PMD on the source code, just run ant pmd:

```bash
ant -Dpmd.home=~/.ant/lib/pmd pmd
```

Output from the command will be stored in the pmd subdirectory:

```
$ ls pmd
pmd_report.html  pmd_report.xml
```

If you want to install PMD some other place than `~/.ant/lib`, you can
unzip the `.zip` file to a directory, and use the `pmd.home` property
to tell ant where to find PMD, like this (if you have installed
PMD under the `./ext_lib directory`):

```bash
ant pmd -Dpmd.home=ext_lib/pmd
```

To run CPD, just use the same as above, but use targets:

```bash
ant -Dpmd.home=ext_lib/pmd cpd cpd-xml
```

Which will result in:

```
$ ls pmd
cpd_report.xml cpd_report.txt
```

# Using JDepend

To see dependencies in the source code, you can use JDepend from
<https://github.com/clarkware/jdepend>.

How to install:

```bash
cd ~/.ant/lib
unzip ~/Desktop/jdepend-2.9.1.zip
ln -s jdepend-2.9.1/ jdepend
cd jdepend/lib
ln -s jdepend-2.9.1.jar jdepend.jar
```

How to analyze:

```bash
ant jdepend
```

Output is stored in the jdepend directory:

```
$ ls jdepend/
report.txt  report.xml
```

# Using SonarQube

Use a sonar runner with included `sonar-project.properties` properties,
e.g. using bash:

```bash
cd <checkout_dir> # it has to contain sonar-project.properties!
export SONAR_RUNNER_OPTS="-Xmx768m -XX:MaxPermSize=256m"
export SERVERIP=10.163.26.78
~//Projects/sonar-runner-2.3/bin/sonar-runner \
    -Dsonar.host.url=http://${SERVERIP}:9000 \
    -Dsonar.jdbc.url=jdbc:h2:tcp://${SERVERIP}:9092/sonar
```

# Using Travis CI

Travis depends on updated and working maven build.
Please see `.travis.yml`, if your branch has this file,
you should be able to connect your Github to Travis CI.
OpenGroks Travis is here: <https://travis-ci.org/OpenGrok/OpenGrok>