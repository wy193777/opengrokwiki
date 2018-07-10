{OpenGrok can be installed and used under different use cases, we will just show how to use the wrapper script. Advanced usage depends on your knowledge of running java applications and command line options of {OpenGrok.
Note, that you '''need''' the create the index no matter what is your use case. Without indexes {Opengrok will be simply useless.

# Requirements

You need the following:

- [JDK](http://www.oracle.com/technetwork/java/) 1.8 or higher
- {OpenGrok '''binaries''' from https://github.com/OpenGrok/OpenGrok/releases (.tar.gz file with binaries, not the source code tarball !)
- https://github.com/universal-ctags for analysis (Exuberant ctags can be used too however they are not as supported as Universal ctags)
- A servlet container like [GlassFish](https://glassfish.dev.java.net/) or [Tomcat](http://tomcat.apache.org) 8.0 or later also running with Java at least 1.8
- If history is needed, appropriate binaries (in some cases also cvs/svn repository) must be present on the system (e.g. [Subversion](http://subversion.tigris.org) or [Mercurial](http://www.selenic.com/mercurial/wiki/index.cgi) or SCCS or ... )
- 2GB of memory for indexing process using OpenGrok script (bigger deployments will need more)
- a recent browser for clients - IE, Firefox, recent Chrome or Safari
- Optional tuning (see https://github.com/oracle/opengrok/wiki/Tuning-for-large-code-bases)

# Recommended links to read for advanced users

- the `Opengrok` script
- [Sample setup](https://github.com/OpenGrok/OpenGrok/blob/master/doc/EXAMPLE.txt)
- `CommandLineOptions` class

# Indexing and web application setup

After installing the package or unpacking the binaries in your target directory (e.g. `cd your_target_dir ; gzcat opengrok-VERSION.tar.gz | tar xf -`), you just need to create the index and then you can use the command line interface to grok your sources.

It is also necessary to install SCM environment accessible by OpenGrok for your repositories for the history indexing to work.

# Creating the index

OpenGrok's command line use is a typical case where OpenGrok is deployed as indexing service on a webserver for an active source repository, which is updated automatically using scheduled cron jobs.

'''Project concept''' - one project is one directory underneath <code>SRC_ROOT</code> and usually contains a checkout of a project (or it's branch, version, ...) sources, it can have several attributes (in its XML description), note that interface of projects is being stabilized so it can change. Projects effectively replace need for more web applications with opengrok <code>.war</code> and leave you with one indexer and one web application serving MORE source code repositories - projects. A nice concept is to have directories underneath <code>SRC_ROOT</code> with a naming convention, thereby creating a good overview of projects (e.g. name-version-branch). Then you have a simple update script & simple index refresher script in place, which simplifies management of more repositories.

[[/images/setup-project.png]]

## <u>Step.0</u> - Setting up the Sources. Having the web application container ready.

Source base must be available locally for OpenGrok to work efficiently. No changes are required to your source tree. If the code is under CVS or SVN, OpenGrok requires the '''checked out source''' tree under SRC_ROOT.

'''Note:''' A local CVSROOT (or SVN if opengrok version less than 0.7) repository must be available. File history will not be fetched from a remote server for CVS (& SVN if opengrok version less than 0.7) !. For newer opengrok versions you must use option <code>-r on</code> for remote repositories history to be indexed, also note this can be more resource (time,cpu,network) demanding.
Note also that OpenGrok ignores symbolic links.

Please install web application container of your choice (e.g. [Tomcat](http://tomcat.apache.org/), [Glassfish](https://glassfish.dev.java.net/)) before going to the next step and make sure it's started.

## <u>Step.1</u> - Deploy the web application

We provided you with OpenGrok wrapper script, which should aid in deploying the web application.
Please change to opengrok directory (can vary on your system)

 `cd /usr/opengrok/bin`

and run 

 `./OpenGrok deploy`

This command will do some sanity checks and will deploy the source.war in its directory to one of detected web application containers.
Please follow the error message it provides.
If it fails to discover your container, please refer to optional steps on changing web application properties, which has manual steps on how to do this.
Alternatively use <code>OPENGROK_TOMCAT_BASE</code> environment variable, e.g.

 `OPENGROK_TOMCAT_BASE=/path/to/my/tomcat/install ./OpenGrok deploy`

Note that OpenGrok script expects the directory <code>/var/opengrok</code> to be available to user running opengrok with all permissions. In root user case it will create all the directories needed, otherwise you have to manually create the directory and grant all permissions to the user used.

## <u>Step.2</u> - Populate DATA_ROOT Directory, let the indexer generate the project XML config file, update configuration.xml to your web app

Second step is to just run the indexing (can take a lot of time). After this is done, indexer automatically attempts to upload newly generated configuration to the web application. Most probably you will not be able to use {Opengrok before this is done. <code>OPENGROK_CTAGS</code> environment variable can be used to tell {Opengrok which ctags implementation to use for indexing.

Please change to opengrok directory (can vary on your system)

 `cd /usr/opengrok/bin`

and run, if your SRC_ROOT is prepared under <code>/var/opengrok/src</code>

 `./OpenGrok index`

otherwise (if SRC_ROOT is in different directory) run:

 `./OpenGrok index <absolute_path_to_your_SRC_ROOT>`

specify alternative ctags binary:

 `OPENGROK_CTAGS=<absolute_path_to_ctags_binary> ./OpenGrok index`

NOTE: Please DON'T use symlinks to /var/opengrok/src - rather use above command with parameter.

Above command should try to upload latest index status reflected into configuration.xml to a running source web application.
Once above command finishes without errors(e.g. SEVERE: Failed to send configuration to localhost:2424
), you should be able to enjoy your opengrok and search your sources using latest indexes and setup.

It is assumed that any SCM commands are reachable in one of the components
of the PATH environment variable (e.g. 'git' command for Git repositories).
Likewise, this should be maintained in the environment of the user which runs
the web server instance.

Congratulations, you should now be able to point your browser to http://YOUR_WEBAPP_SERVER:WEBAPPSRV_PORT/source to work with your fresh opengrok installation! :-)

At this time we'd like to point out some customization to OpenGrok script for advanced users.
A common case would be, that you want the data in some other directory than /var/opengrok.
This can be easily achieved by using environment variable OPENGROK_INSTANCE_BASE .
E.g. if my opengrok data directory is <code>/tank/opengrok</code> and my source root is in <code>/tank/source</code> and I'd like to get more verbosity I'd run the indexer as:

 `OPENGROK_VERBOSE=true OPENGROK_INSTANCE_BASE=/tank/opengrok ./OpenGrok index /tank/source`

Since above will also change default location of config file, beforehands(or restart your web container after creating this symlink) I suggest doing below for our case of having opengrok instance in <code>/tank/opengrok</code> :

 `ln -s /tank/opengrok/etc/configuration.xml /var/opengrok/etc/configuration.xml`

A lot more customizations can be found inside the script, you just need to have a look [https://github.com/OpenGrok/OpenGrok/blob/master/OpenGrok at it], eventually create a configuration out of it and use <code>OPENGROK_CONFIGURATION</code> environment variable to point to it. Obviously such setups can be used for nightly cron job updates of index or other automated purposes.

# Optional info

Additional information can be found in the [https://github.com/OpenGrok/OpenGrok/blob/master/doc/EXAMPLE.txt EXAMPLE.txt] in the doc folder

## Custom ctags configuration

To make ctags recognize additional symbols/definitions/etc. it is possible to
specify configuration file with extra configuration options for ctags.

This can be done by setting `OPENGROK_CTAGS_OPTIONS_FILE` environment variable
when running the OpenGrok shell script (or directly with the `-o` option for
`opengrok.jar`). Default location for the configuration file in the OpenGrok
shell script is `etc/ctags.config` under the OpenGrok base directory (by default
the full path to the file will be `/var/opengrok/etc/ctags.config`).

Sample configuration file for Solaris code base:

```
--regex-asm=/^[ \t]*(ENTRY_NP|ENTRY|RTENTRY)+\(([a-zA-Z0-9_]+)\)/\2/f,function/
--regex-asm=/^[ \t]*ENTRY2\(([a-zA-Z0-9_]+),[ ]*([a-zA-Z0-9_]+)\)/\1/f,function/
--regex-asm=/^[ \t]*ENTRY2\(([a-zA-Z0-9_]+),[ ]*([a-zA-Z0-9_]+)\)/\2/f,function/
--regex-asm=/^[ \t]*ENTRY_NP2\(([a-zA-Z0-9_]+),[ ]*([a-zA-Z0-9_]+)\)/\1/f,function/
--regex-asm=/^[ \t]*ENTRY_NP2\(([a-zA-Z0-9_]+),[ ]*([a-zA-Z0-9_]+)\)/\2/f,function/
```

## Introduce own mapping for an extension to analyzer

OpenGrok script doesn't support this out of box, so you'd need to add it there.
Usually to `StdInvocation()` function after line `-jar ${OPENGROK_JAR}`.
It would look like this:

```
-A cs:org.opensolaris.opengrok.analysis.PlainAnalyzer
```

(this will map extension `.cs` to `PlainAnalyzer`)
You should even be able to override OpenGroks analyzers using this option.


## CLI - Command Line Interface Usage

You need to pass location of project file + the query to Search class, e.g. for fulltext search for project with above generated configuration.xml you'd do:

 $ java -cp ./opengrok.jar org.opensolaris.opengrok.search.Search -R /var/opengrok/etc/configuration.xml -f fulltext_search_string

For quick help run:

 $ java -cp ./opengrok.jar org.opensolaris.opengrok.search.Search

Sample search:

[[/images/CLI-search.png]]

## Optional need to change web application properties or name

You might need to modify the web application if you don't store the configuration file in the default location (<code>/var/opengrok/etc/configuration.xml</code>).

To '''configure''' the webapp source.war, look into the parameters defined in <code>WEB-INF/web.xml</code> of <code>source.war</code> (use <code>jar</code> or <code>zip/unzip</code> or your preffered zip tool to get into it - e.g. extract the web.xml file from source.war (<code>$ unzip source.war WEB-INF/web.xml</code>) file, edit web.xml and re-package the jar file (<code>zip -u source.war WEB-INF/web.xml</code>) ) file and change those web.xml parameters appropriately. These sample parameters need modifying.

* CONFIGURATION - the absolute path to XML file containing project configuration (e.g. <code>/var/opengrok/etc/configuration.xml</code>)
* ConfigAddress - port for remote updates to configuration, optional, but '''advised (since there is no authentification)''' to be set to '''localhost''':<some_port> (e.g. localhost:2424), if you choose some_port below 1024 you have to have root privileges.

If you need to change name of the web application from source to something else you need to use special option <code>-w <new_name></code> for indexer to create proper xrefs, besides changing the <code>.war</code> file name. Examples below show just deploying source.war, but you can use it to deploy your new_name.war too.

### Deploy the modified .war file in glassfish/Sun Java App Server: ====

* '''Option 1:''' Use browser and log into <code>glassfish web administration interface</code>

: Common Tasks / Applications / Web Applications , button '''Deploy''' and point it to your source.war webarchive

* '''Option 2:''' Copy the source.war file to `//GLASSFISH///domains///YOURDOMAIN///autodeploy` directory, glassfish will try to deploy it "automagically".
* '''Option 3:''' Use CLI from `//GLASSFISH//` directory:

 `./bin/asadmin deploy /path/to/source.war`

### Deploy the modified .war file in tomcat:

* just copy the source.war file to `//TOMCAT_INSTALL///webapps` directory.

## Optional setup of security manager for Tomcat

On some linux distribution you need to setup permissions for SRC_ROOT and DATA_ROOT. Please check your [http://tomcat.apache.org/tomcat-5.5-doc/security-manager-howto.html Tomcat documentation] on this, or simply disable (your risk!) <u>security manager</u> for Tomcat (e.g. in debian/ubuntu : in file <code>/etc/default/tomcat5.5</code> set <code>TOMCAT5_SECURITY=no</code>).
A sample approach is to <u>edit</u> <code>/etc/tomcat5.5/04webapps.policy</code> (or <code>/var/apache/tomcat/conf/catalina.policy</code>) and set this (it will give opengrok all permissions) for your opengrok webapp instance:

```
grant codeBase "file:${catalina.home}/webapps/source/-" {     
permission java.security.AllPermission;};
grant codeBase "file:${catalina.home}/webapps/source/WEB-INF/lib/-" {     
permission java.security.AllPermission;};
```

Alternatively you can be more restrictive (haven't tested below with a complex setup(e.g. some versioning system which needs local access as cvs), if it will not work, please report through [[Discussions]].

```
grant codeBase "file:${catalina.home}/webapps/source/-" {  
permission java.util.PropertyPermission "subversion.native.library", "read";  
permission java.lang.RuntimePermission "loadLibrary.svnjavahl-1?;  
permission java.lang.RuntimePermission "loadLibrary.libsvnjavahl-1?;  
permission java.lang.RuntimePermission "loadLibrary.svnjavahl";  
permission java.util.PropertyPermission "disableLuceneLocks", "read";  
permission java.util.PropertyPermission "catalina.home", "read";  
permission java.util.PropertyPermission "java.io.tmpdir", "read";  
permission java.util.PropertyPermission "org.apache.lucene.lockdir", "read";  
permission java.util.PropertyPermission "org.apache.lucene.writeLockTimeout", "read";  
permission java.util.PropertyPermission "org.apache.lucene.commitLockTimeout", "read";  
permission java.util.PropertyPermission "org.apache.lucene.mergeFactor", "read";  
permission java.util.PropertyPermission "org.apache.lucene.minMergeDocs", "read";  
permission java.util.PropertyPermission "org.apache.lucene.*", "read";  
permission java.io.FilePermission "/var/lib/tomcat5/temp", "read";  
permission java.io.FilePermission "/var/lib/tomcat5/temp/*", "write";  
permission java.io.FilePermission "/var/lib/tomcat5/temp/*", "delete";};
grant codeBase "file:${catalina.home}/webapps/source/WEB-INF/lib/-" {  
permission java.util.PropertyPermission "subversion.native.library", "read";  
permission java.lang.RuntimePermission "loadLibrary.svnjavahl-1?;  
permission java.util.PropertyPermission "disableLuceneLocks", "read";  
permission java.util.PropertyPermission "catalina.home", "read";  
permission java.util.PropertyPermission "java.io.tmpdir", "read";};
grant codeBase "file:${catalina.home}/webapps/source/WEB-INF/classes/-" {  
permission java.util.PropertyPermission "subversion.native.library", "read";  
permission java.lang.RuntimePermission "loadLibrary.svnjavahl-1?;  
permission java.util.PropertyPermission "disableLuceneLocks", "read";  
permission java.util.PropertyPermission "catalina.home", "read";  
permission java.util.PropertyPermission "java.io.tmpdir", "read";};
```