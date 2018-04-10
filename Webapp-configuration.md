# Webapp configuration

<!-- toc -->
- [webapp parameters](#webapp-parameters)
  * [Changing webapp parameters](#changing-webapp-params)
- [Path Descriptions](#path-descriptions)
<!-- tocstop -->

## webapp parameters

To configure the webapp `source.war`, look into the parameters defined in
`web.xml` of `source.war` file and change them (see note1) appropriately.

* **HEADER** – the fragment of HTML that will be used to display title or
              logo of your project
* **SRC_ROOT** – absolute path name of the root directory of your source tree
* **DATA_ROOT** – absolute path of the directory where OpenGrok data
                 files are stored

  * File `header_include` can be created under `DATA_ROOT`.
    The contents of this file will be appended to the header of each
    web page after the OpenGrok logo element.
  * File `footer_include` can be created under `DATA_ROOT`.
    The contents of this file will be appended to the footer of each
    web page after the information about last index update.
  * The file `body_include` can be created under `DATA_ROOT`.
    The contents of this file will be inserted above the footer of the web
    application's "Home" page.
  * The file `error_forbidden_include` can be created under `DATA_ROOT`.
    The contents of this file will be displayed as the error page when
    the user is forbidden to see a particular project with `HTTP 403` code.

### Changing webapp parameters

`web.xml` is the deployment descriptor for the web application. It is in a Jar
file named `source.war`, you can change it as follows:

* **Option 1**:
  Unzip the file to `TOMCAT/webapps/source/` directory and
     change the `source/WEB-INF/web.xml` and other static html files like
     `index.html` to customize to your project.

* **Option 2**:
  Extract the `web.xml` file from `source.war` file

  ```bash
  unzip source.war WEB-INF/web.xml
  ```

  edit `web.xml` and re-package the jar file.

  ```bash
  zip -u source.war WEB-INF/web.xml
  ```

  Then copy the war files to `TOMCAT/webapps` directory.

* **Option 3**:
  Edit the Context container element for the webapp

  Copy `source.war` to `TOMCAT/webapps`

  When invoking OpenGrok to build the index, use `-w <webapp>` to set the
     context. If you change this(or set using `OPENGROK_WEBAPP_CONTEXT`) later,
     FULL clean reindex is needed.

  After the index is built, there's a couple different ways to set the
  Context for the servlet container:

  * Add the Context inside a Host element in `TOMCAT/conf/server.xml`

      ```xml
      <Context path="/<webapp>" docBase="source.war">
          <Parameter name="DATA_ROOT" value="/path/to/data/root" override="false" />
          <Parameter name="SRC_ROOT" value="/path/to/src/root" override="false" />
          <Parameter name="HEADER" value='...' override="false" />
      </Context>
      ```
   * Create a Context file for the webapp

     This file will be named `<webapp>.xml`.

     For Tomcat, the file will be located at:
     `TOMCAT/conf/<engine_name>/<hostname>`, where `<engine_name>`
     is the Engine that is processing requests and `<hostname>` is a Host
     associated with that Engine.  By default, this path is
     `TOMCAT/conf/Catalina/localhost` or `TOMCAT/conf/Standalone/localhost`.

     This file will contain something like the Context described above.

## Path Descriptions

OpenGrok can use path descriptions in various places (e.g. while showing
directory listings or search results). Example descriptions are in `paths.tsv`
file (delivered as `/usr/opengrok/doc/paths.tsv` by OpenGrok package on Solaris).

The `paths.tsv` file is read by OpenGrok indexing script from the configuration
directory (the same where `configuration.xml` is located) which will create file
`dtags.eftar` in the index subdirectory under `DATA_ROOT` directory which will
then be used by the webapp to display the descriptions.

The file contains descriptions for directories one per line. Path to the
directory and its description are separated by tab. The path to the directory
is absolute path under the `SRC_ROOT` directory.

For example, if the `SRC_ROOT` directory contains the following directories:

* foo
* bar
* bar/blah
* random
* random/code

then the `paths.tsv` file contents can look like this:

```
/foo  source code for foo
/bar  source code for bar
/bar/blah source code for blah
```

Note that only some paths can have a description.
