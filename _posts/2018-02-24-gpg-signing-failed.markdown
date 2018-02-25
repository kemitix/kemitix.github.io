---
layout: post
title: "gpg: signing failed"
date: 2018-02-24 22:14:00 +0000
---
Trying to setup Jenkins to sign artifacts with the `maven-gpg-plugin` to allow me to deploy to the Sonatype OSS Nexus. However I was getting the error:

```
[INFO] --- maven-gpg-plugin:1.6:sign (sign-artifacts) @ parent ---
gpg: signing failed: No such file or directory
```

[Choon-Chern Lim](https://myshittycode.com/2017/08/07/maven-gpg-plugin-prevent-signing-prompt-or-gpg-signing-failed-no-such-file-or-directory-error/) suggests adding `--pinentry-mode loopback` to fix this.

```xml
<configuration>
    <gpgArguments>
        <arg>--pinentry-mode</arg>
        <arg>loopback</arg>
    </gpgArguments>
</configuration>
```

Which then brought me to the error:

```
[INFO] --- maven-gpg-plugin:1.6:sign (sign-artifacts) @ parent ---
gpg: signing failed: Inappropriate ioctl for device
```

Which led me to actually read the [gpg man page](https://linux.die.net/man/1/gpg2).
Where I found the option `--batch`.

> **--batch**  
> **--no-batch**  
> Use batch mode. Never ask, do not allow interactive commands. --no-batch disables this option. Note that even with a filename given on the command line, gpg might still need to read from STDIN (in particular if gpg figures that the input is a detached signature and no data file has been specified). Thus if you do not want to feed data via STDIN, you should connect STDIN to oq/dev/nullcq.

My next iteration of `maven-gpg-plugin` config is now:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-gpg-plugin</artifactId>
    <version>1.6</version>
    <executions>
        <execution>
            <id>sign-artifacts</id>
            <phase>verify</phase>
            <goals>
                <goal>sign</goal>
            </goals>
            <configuration>
                <gpgArguments>
                    <arg>--batch</arg>
                    <arg>--pinentry-mode</arg>
                    <arg>loopback</arg>
                </gpgArguments>
            </configuration>
        </execution>
    </executions>
</plugin>
```
