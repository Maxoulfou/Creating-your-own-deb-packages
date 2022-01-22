# Creating-your-own-deb-packages

# Prerequisites

```shell
sudo apt-get install -y gcc dpkg-dev gpg
```


## Step 0: Creating a Simple Hello World Program

```shell
mkdir -p ~/example/hello-world-program

echo '#include <stdio.h>
int main() {
    printf("hello packaged world\n");
    return 0;
}' > ~/example/hello-world-program/hello.c
```

```shell
cd ~/example/hello-world-program
gcc -o hello-world hello.c
```


## Step 1: Creating a deb Package

> deb package name format convention : `<package-name>_<version>-<release-number>_<architecture>`

- `package-name` is the name of our package, hello-world in our case,
- `version` is the version of the software, 0.0.1 in our case,
- `release-number` is used to track different releases of the same software version; it’s usually set to 1, but hypothetically if there was an error in the packaging (e.g. a file was missed, or the description had an error in it, or a post-install script was wrong), this number would be increased to track the change, and
- `architecture` is the target architecture of the platform, amd64 in this example; however if your package is architecture-independent (e.g. a python script), then you can set this to all.

> Theoretically, the directory doesn’t have to follow this naming convention; however some of the tools we will use later in the tutorial require this directory naming convention.


```shell
mkdir -p ~/example/hello-world_0.0.1-1_amd64
```

This directory will be the root of the package. Since we want our hello-world binary to be installed system wide, we’ll have to store it under usr/bin/hello-world with the following commands:

```shell
cd ~/example/hello-world_0.0.1-1_amd64
mkdir -p usr/bin
cp ~/example/hello-world-program/hello-world usr/bin/.
```

Each package requires a control file which needs to be located under the DEBIAN directory. You can copy and paste the following to create one:

```shell
mkdir -p ~/example/hello-world_0.0.1-1_amd64/DEBIAN

echo "Package: hello-world
Version: 0.0.1
Maintainer: example <example@example.com>
Depends: libc6
Architecture: amd64
Homepage: http://example.com
Description: A program that prints hello" \
> ~/example/hello-world_0.0.1-1_amd64/DEBIAN/control
```

Note that we’re assuming an amd64 Architecture for this tutorial, if your binary is for a different architecture, adjust accordingly. If you’re distributing a platform-independent package, you can set the architecture to `all`.

By this point you should have created two files:

```shell
~/example/hello-world_0.0.1-1_amd64/usr/bin/hello-world
~/example/hello-world_0.0.1-1_amd64/DEBIAN/control
```

To build the .deb package, run:

```shell
dpkg --build ~/example/hello-world_0.0.1-1_amd64
```

This will output a deb package under ~/example/hello-world_0.0.1.deb.

You can inspect the info of the deb by running:

```shell
dpkg-deb --info ~/example/hello-world_0.0.1.deb
```

which will show:

```shell
new Debian package, version 2.0.
size 2832 bytes: control archive=336 bytes.
    182 bytes,     7 lines      control
Package: hello-world
Version: 0.0.1
Maintainer: example <example@example.com>
Depends: libc6
Architecture: amd64
Homepage: http://example.com
Description: A program that prints hello
```

You can also view the contents by running:

```shell
dpkg-deb --contents ~/example/hello-world_0.0.1.deb
```

which will show:

```shell
drwxrwxr-x alex/alex         0 2021-05-17 16:21 ./
drwxrwxr-x alex/alex         0 2021-05-17 16:18 ./usr/
drwxrwxr-x alex/alex         0 2021-05-17 16:18 ./usr/bin/
-rwxrwxr-x alex/alex     16696 2021-05-17 16:18 ./usr/bin/hello-world
```

This package can then be installed using `dpkg -i`

```shell
dpkg -i ~/example/hello-world_0.0.1-1_amd64.deb
```

Done !
