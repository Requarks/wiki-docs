---
title: Windows
description: Getting started with a Wiki.js installation on Windows
published: true
date: 2021-04-03T00:12:08.885Z
tags: setup
editor: markdown
dateCreated: 2019-05-04T04:36:05.505Z
---

Before going any further, make sure your system meets all the [requirements](/install/requirements).

# Install

1. Open a **Powershell** prompt in administrator mode.
2. If you are using **Windows 7 / Windows Server 2008 R2 or older**, you must run the following command. *(otherwise skip this step)*
  ```powershell
  [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
  ```
3. Download the latest version of Wiki.js:
  ```powershell
  Invoke-WebRequest -Uri "https://github.com/Requarks/wiki/releases/download/2.5.201/wiki-js-windows.tar.gz" -OutFile "wiki-js.tar.gz"
  ```

4. Extract the package to the final destination of your choice:
  ```powershell
  New-Item -Path "C:\" -Name "wiki" -ItemType "directory"
  tar xzf wiki-js.tar.gz -C "C:\wiki"
  cd C:\wiki
  ```
  > The **tar** utility is only available on Windows 10. On earlier versions, you'll need a 3rd-party utility like [7-zip](https://www.7-zip.org/) to extract the file.
  {.is-warning}
5. Rename the sample config file to `config.yml`:
  ```powershell
  Rename-Item -Path config.sample.yml -NewName config.yml
  ```
6. Edit the config file using your favorite text editor (e.g. Notepad) and fill in your database and port settings ([Configuration Reference](/install/config)):
  ```powershell
  notepad .\config.yml
  ```
7. ***For SQLite installations only:*** *(skip this step otherwise)* Fetch native bindings for SQLite3:
  ```bash
  npm rebuild sqlite3
  ```
8. Run Wiki.js
  ```powershell
  node server
  ```
9. Wait until you are invited to open to the setup page in your browser.
10. Complete the setup wizard to finish the installation.

# Run as service

There are several solutions to run Wiki.js as a background service, e.g.:

- [AlwaysUp](https://www.coretechnologies.com/products/AlwaysUp/)
- [PM2](http://pm2.keymetrics.io/) / [pm2-windows-service](https://www.npmjs.com/package/pm2-windows-service)
- [forever](https://www.npmjs.com/package/forever)
- [node-windows](https://github.com/coreybutler/node-windows)

![](https://a.icons8.com/djxbtnYm/GBjHDS/svg.svg){.align-abstopright}

# Run under IIS

## Install iisnode

If we want to run wikijs under iis then we first need to install [iisnode](https://github.com/Azure/iisnode/releases).

## Setup a IIS site

Next we need to setup a new site under IIS.
Configuring new sites under IIS is outside the scope of this document

make a note of the directory that the site points to
In this case we're going to assume this is **C:\\wikijs\\** for the sake of simplicity

## Create an init file

Next we need to create a javascript startup file
place this under **C:\\wikijs\\** or your site directory

**wikiSiteInit.js**
```js
require('./server/index.js');
```

## Web.Config file

Next we're going to create a Web.Config file
place this under **C:\\wikijs\\** or your site directory

**Web.Config**
```xml
<configuration>
  <system.webServer>

  <!-- Uncomment the following line if Node.js is not available via the environment path -->
  <!-- <iisnode nodeProcessCommandLine="{Path to node.exe}" /> -->

  <!-- In some cases additional flags are needed to ignore deprecation warnings, this is due to iisnode using Buffer() -->
  <!-- <iisnode nodeProcessCommandLine="node.exe --no-deprecation --no-warnings"/> -->

    <handlers>
      <add name="iisnode" path="wikiSiteInit.js" verb="*" modules="iisnode" />
    </handlers>

    <rewrite>
      <rules>
        <rule name="default">
          <match url="/*" />
            <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
          <action type="Rewrite" url="wikiSiteInit.js" />
        </rule>
      </rules>
    </rewrite>

  </system.webServer>
</configuration>
```

  > You'll notice above that we may need to pass in the flags
  **--no-deprecation --no-warnings**
  to get things to work. This is due to a bug associated with iisnode's use of the depreciated Buffer() [tjanczuk/iisnode#668](https://github.com/tjanczuk/iisnode/issues/668)
{.is-warning}
