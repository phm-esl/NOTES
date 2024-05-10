# Building Joe-Editor

insert the following line into joe/tty.c
    #include <util.h>
This prevents:

    error: implicit declaration of functions
    'openoty' ... 'login_tty'



#Building OpenSSL

Source https://www.openssl.org/source/

    mkdir ~/Builds
    cd ~/Builds
    tar -zxf ~/Downloads/openssl-3.0.2.tar
    cd openssl-3.0.2
    ./Configure --prefix=$HOME/Software/openssl-3.0.2
    make
    make test
    make install

    phm@MacBook-Pro openssl-3.0.2 % ls ~/Software/openssl-3.0.2 
    bin	include	lib	share	ssl




#Building WX-Widgets

Source https://wxwidgets.org/downloads/

cd ~/Builds
tar -zxf ~/Downloads/wxWidgets-3.1.6.tar.bz2
cd wxWidgets-3.1.6
./configure \
 --prefix=$HOME/Software/wxWidgets-3.1.6 \
 --enable-webview \
 --enable-compat28
make clean && make -j 10 && make install

The --enable-compat28 needed for OTP 23




#Building Erlang/OTP

Source https://www.erlang.org/downloads

mkdir ~/Builds
cd ~/Builds
tar -zxf ~/Downloads/otp_src_24.3.3
cd otp_src_24.3.3

export PATH="/Users/phm/Software/wxWidgets-3.1.6/bin\
:/Users/phm/Software/openssl-3.0.2/bin\
:$PATH"

export CXX=/usr/bin/g++

./configure \
 --prefix=$HOME/Software/erlang-24.3.3 \
 --with-dynamic-trace=dtrace \
 --with-ssl=$HOME/Software/openssl-3.0.2 \
 --with-wxwidgets=$HOME/Software/wxWidgets-3.1.6 \
 --without-odbc \
 --without-javac \
 --enable-fips

error: taking the address of a temporary object of type "'wxBitmap'"

export CXXFLAGS="-fpermissive"

./configure \
 --prefix=$HOME/Software/erlang-24.3.3 \
 --with-dynamic-trace=dtrace \
 --with-ssl=$HOME/Software/openssl-3.0.2 \
 --without-wxwidgets \
 --without-odbc \
 --without-javac \
 --enable-fips


configure: WARNING: ******************************************************************
configure: WARNING: * Using OpenSSL 3.0 is not yet recommended for production code.  *
configure: WARNING: ******************************************************************


This seems to have disabled the 'crypto' and 'ssl' libraries.  It is claimed
that OpenSSL 3 is supported in Erlang/OTP 24, but the configure warning
contradicts.


make clean && make -j 10 && make install



Trying again with OpenSSL-1.1.1o instead

cd ~/Builds
tar -xf ~/Downloads/openssl-1.1.1o.tar
cd openssl-1.1.1o
./config --prefix=$HOME/Software/openssl-1.1.1o
make
make test
make install



cd ~/Builds/otp_src_24.3.3
./configure \
 --prefix=$HOME/Software/erlang-24.3.3 \
 --with-dynamic-trace=dtrace \
 --with-ssl=$HOME/Software/openssl-1.1.1o \
 --without-wxwidgets \
 --without-odbc \
 --without-javac \
 --enable-fips

make clean && make -j 10 && make install








cd ~/Repos
git clone https://github.com/erlang/otpn GitHub-erlang-otp
cd GitHub-erlang-otp
git checkout -b phm OTP-24.3.3

198> git diff
diff --git a/lib/wx/c_src/gen/wxe_wrapper_5.cpp b/lib/wx/c_src/gen/wxe_wrapper_5.cpp
index 43bda73ded..e66ee3720c 100644
--- a/lib/wx/c_src/gen/wxe_wrapper_5.cpp
+++ b/lib/wx/c_src/gen/wxe_wrapper_5.cpp
@@ -2162,7 +2162,11 @@ void wxMenuItem_GetBitmap(WxeApp *app, wxeMemEnv *memenv, wxeCommand& Ecmd)
   wxMenuItem *This;
   This = (wxMenuItem *) memenv->getPtr(env, argv[0], "This");
   if(!This) throw wxe_badarg("This");
-  const wxBitmap * Result = &This->GetBitmap();
+  // phm fix for:-
+  // "error: taking address of a temporary object of type wxBitmap"
+  // Using the variable Tmp to avoid taking address of temporary
+  const wxBitmap Tmp = This->GetBitmap();
+  const wxBitmap * Result = &Tmp;
   wxeReturn rt = wxeReturn(memenv, Ecmd.caller, true);
   rt.send(  rt.make_ref(app->getRef((void *)Result,memenv), "wxBitmap"));
 
diff --git a/lib/wx/c_src/gen/wxe_wrapper_7.cpp b/lib/wx/c_src/gen/wxe_wrapper_7.cpp
index f4716e1228..f06517541b 100644
--- a/lib/wx/c_src/gen/wxe_wrapper_7.cpp
+++ b/lib/wx/c_src/gen/wxe_wrapper_7.cpp
@@ -2339,7 +2339,13 @@ void wxToolBar_AddTool_4(WxeApp *app, wxeMemEnv *memenv, wxeCommand& Ecmd)
     } else        Badarg("Options");
   };
   if(!This) throw wxe_badarg("This");
-  wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(toolId,label,*bitmap,shortHelp,kind);
+  // phm fix:-
+  // "error: call to member function 'AddTool' is ambiguous"
+  // Using arguments that match "the full AddTool function"
+  // as described in wxWidgets-3.1.6/include/wx-3.1/wx/tbarbase.h
+  // to remove ambiguity.
+  wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(
+    toolId,label,*bitmap,wxBitmapBundle(),kind,shortHelp);
   wxeReturn rt = wxeReturn(memenv, Ecmd.caller, true);
   rt.send(  rt.make_ref(app->getRef((void *)Result,memenv), "wx"));




export PATH="/Users/phm/Software/wxWidgets-3.1.6/bin\
:/Users/phm/Software/openssl-1.1.1o/bin\
:$PATH"
./otp_build setup -a --prefix=$HOME/Software/erlang-24.3.3 \
--with-ssl=$HOME/Software/openssl-1.1.1o

The above otp_build fails.  Apply the two file patches to the tar source,
and try yet again:


cp lib/wx/c_src/gen/wxe_wrapper_{5,7}.cpp \
~/Builds/otp_src_24.3.3/lib/wx/c_src/gen/

cd ~/Builds/otp_src_24.3.3/
make clean
./configure \
 --prefix=$HOME/Software/erlang-24.3.3 \
 --with-dynamic-trace=dtrace \
 --with-ssl=$HOME/Software/openssl-1.1.1o \
 --without-odbc \
 --without-javac \
 --enable-fips

*********************************************************************
**********************  APPLICATIONS DISABLED  **********************
*********************************************************************

jinterface     : Java compiler disabled by user
odbc           : User gave --without-odbc option

*********************************************************************
*********************************************************************
**********************  DOCUMENTATION INFORMATION  ******************
*********************************************************************

documentation  : 
                 fop is missing.
                 Using fakefop to generate placeholder PDF files.

*********************************************************************

make
make install




The compilation errors that the two source file patches manage to work around:


gen/wxe_wrapper_5.cpp:2165:29: error: taking the address of a temporary 
object of type 'wxBitmap' [-Waddress-of-temporary]
  const wxBitmap * Result = &This->GetBitmap();
                            ^~~~~~~~~~~~~~~~~~
1 warning and 1 error generated.
make[3]: *** [aarch64-apple-darwin21.4.0/wxe_wrapper_5.o] Error 1
make[2]: *** [opt] Error 2
make[1]: *** [opt] Error 2
make: *** [libs] Error 2



gen/wxe_wrapper_7.cpp:2342:58: error: call to member function 'AddTool' is 
ambiguous
  wxToolBarToolBase * Result = 
(wxToolBarToolBase*)This->AddTool(toolId,label,*bitmap,shortHelp,kind);
                                                   ~~~~~~^~~~~~~
/Users/phm/Software/wxWidgets-3.1.6/include/wx-3.1/wx/tbarbase.h:311:24: 
note: candidate function
    wxToolBarToolBase *AddTool(int toolid,
                       ^
/Users/phm/Software/wxWidgets-3.1.6/include/wx-3.1/wx/tbarbase.h:527:24: 
note: candidate function
    wxToolBarToolBase *AddTool(int toolid,
                       ^







Build and install Elixir

tar -zxf ~/Downloads/elixir-1.13.4.tar 
cd elixir-1.13.4/
make PREFIX=/Users/phm/Software/erlang-24.3.3 
make PREFIX=/Users/phm/Software/erlang-24.3.3 install


Following instructions from Deploying Elixir by Miguel Cobá,
section Install Phoenix:

mkdir ~/tmp && cd ~/tmp

483> mix local.rebar --force
* creating /Users/phm/.mix/rebar
* creating /Users/phm/.mix/rebar3

484> mix local.hex --force
* creating /Users/phm/.mix/archives/hex-1.0.1

# using installation recipe from phoenixframework.org, not the book:
486> mix archive.install hex phx_new
Resolving Hex dependencies...
Dependency resolution completed:
New:
  phx_new 1.6.9
* Getting phx_new (Hex package)
All dependencies are up to date
Compiling 11 files (.ex)
Generated phx_new app
Generated archive "phx_new-1.6.9.ez" with MIX_ENV=prod
Are you sure you want to install "phx_new-1.6.9.ez"? [Yn] y
* creating /Users/phm/.mix/archives/phx_new-1.6.9

487> mix phx.new demo --live
* creating demo/config/config.exs
* creating demo/config/dev.exs
* creating demo/config/prod.exs
* creating demo/config/runtime.exs
* creating demo/config/test.exs
* creating demo/lib/demo/application.ex
* creating demo/lib/demo.ex
* creating demo/lib/demo_web/views/error_helpers.ex
* creating demo/lib/demo_web/views/error_view.ex
* creating demo/lib/demo_web/endpoint.ex
* creating demo/lib/demo_web/router.ex
* creating demo/lib/demo_web/telemetry.ex
* creating demo/lib/demo_web.ex
* creating demo/mix.exs
* creating demo/README.md
* creating demo/.formatter.exs
* creating demo/.gitignore
* creating demo/test/support/conn_case.ex
* creating demo/test/test_helper.exs
* creating demo/test/demo_web/views/error_view_test.exs
* creating demo/lib/demo/repo.ex
* creating demo/priv/repo/migrations/.formatter.exs
* creating demo/priv/repo/seeds.exs
* creating demo/test/support/data_case.ex
* creating demo/lib/demo_web/controllers/page_controller.ex
* creating demo/lib/demo_web/views/page_view.ex
* creating demo/test/demo_web/controllers/page_controller_test.exs
* creating demo/test/demo_web/views/page_view_test.exs
* creating demo/assets/vendor/topbar.js
* creating demo/lib/demo_web/templates/layout/root.html.heex
* creating demo/lib/demo_web/templates/layout/app.html.heex
* creating demo/lib/demo_web/templates/layout/live.html.heex
* creating demo/lib/demo_web/views/layout_view.ex
* creating demo/lib/demo_web/templates/page/index.html.heex
* creating demo/test/demo_web/views/layout_view_test.exs
* creating demo/lib/demo/mailer.ex
* creating demo/lib/demo_web/gettext.ex
* creating demo/priv/gettext/en/LC_MESSAGES/errors.po
* creating demo/priv/gettext/errors.pot
* creating demo/assets/css/phoenix.css
* creating demo/assets/css/app.css
* creating demo/assets/js/app.js
* creating demo/priv/static/robots.txt
* creating demo/priv/static/images/phoenix.png
* creating demo/priv/static/favicon.ico

Fetch and install dependencies? [Yn] 
* running mix deps.get
* running mix deps.compile

We are almost there! The following steps are missing:

    $ cd demo

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server


488> cd demo






Build and install GNU Make 4.3


tar -xf ~/Downloads/make-4.3.tar
./configure --prefix=$HOME/Software/make-4.3
make
make install








Build and install GNU Grep 3.7
MacOS only provides BSD grep 2.6.0-FreeBSD

cd ~/Builds
tar -xf ~/Downloads/grep-3.7.tar 
cd grep-3.7/
./configure --help
./configure --prefix=$HOME/Software/grep-3.7
make 
make install




Build and install ncurses
The configure option --enable-widec enables UTF8

https://invisible-island.net/ncurses/ncurses.faq.html

cd ~/Builds
tar -xf ~/Downloads/ncurses-6.3.tar 
cd ./ncurses-6.3/
./configure --prefix=$HOME/Software/ncurses-6.3 --enable-widec
make clean && make && make install


Build and install nano editor

cd ~/Builds
tar -xf ~/Downloads/ncurses-6.3.tar 
cd ncurses-6.3/
export LDFLAGS="-L$HOME/Software/ncurses-6.3/lib"
./configure --prefix=$HOME/Software/nano-6.3
make && make install




Note to self: need to populate the LDFLAGS environment
parameter from the $HOME/Software/*/lib/ directories.







Build and install PostgreSQL

Download source code location:
    https://ftp.postgresql.org/pub/source/v14.3/postgresql-14.3.tar.bz2

Documentation postgresql-14-A4.pdf location:

    https://www.postgresql.org/docs/14/index.html

Build process:

    cd ~/Builds
    tar -jxf ~/Downloads/postgresql-14.3.tar.bz2 
    cd postgresql-14.3/
    ./configure --prefix=$HOME/Software/postgresql-14.3
    make world
    make install-world
    mkdir -p ~/Media/Data/Postgres

Open new terminal to refresh Bash $PATH to include the new s/w:

    type initdb
    initdb is /Users/phm/Software/postgresql-14.3/bin/initdb

    initdb -D $HOME/Media/Data/Postgres

Logging output to console:

    The files belonging to this database system will be owned by user "phm".
    This user must also own the server process.

    The database cluster will be initialized with locale "en_GB.UTF-8".
    The default database encoding has accordingly been set to "UTF8".
    The default text search configuration will be set to "english".

    Data page checksums are disabled.

    fixing permissions on existing directory /Users/phm/Media/Data/Postgres ... ok
    creating subdirectories ... ok
    selecting dynamic shared memory implementation ... posix
    selecting default max_connections ... 100
    selecting default shared_buffers ... 128MB
    selecting default time zone ... Europe/London
    creating configuration files ... ok
    running bootstrap script ... ok
    performing post-bootstrap initialization ... ok
    syncing data to disk ... ok

    initdb: warning: enabling "trust" authentication for local connections
    You can change this by editing pg_hba.conf or using the option -A, or
    --auth-local and --auth-host, the next time you run initdb.

    Success. You can now start the database server using:

        pg_ctl -D /Users/phm/Media/Data/Postgres -l logfile start

Start DB server as instructed:

    $> pg_ctl -D /Users/phm/Media/Data/Postgres -l logfile start
    waiting for server to start.... done
    server started

    $> createdb test
    $> psql test
    psql (14.3)
    Type "help" for help.

    test=# help
    You are using psql, the command-line interface to PostgreSQL.
    Type:  \copyright for distribution terms
           \h for help with SQL commands
           \? for help with psql commands
           \g or terminate with semicolon to execute query
           \q to quit
    test=# 










#Copenhagen Project

The Copenhagen Project source code has dependency requirements. One is the
PostGIS extension, which in turn depends on:

 * https://libgeos.org/ depends on Cmake -- DONE
 * https://proj.org/ depends on LibTIFF and Sqlite3 -- DONE
 * http://gdal.org/ -- DONE
 * http://www.xmlsoft.org/ -- DONE
 * https://github.com/json-c/json-c -- DONE





#Build and install CMake

Required for building libgeos.org, that in turn is required for
PostGIS.net...

    tar -xf ~/Downloads/cmake-3.23.2.tar 
    cd cmake-3.23.2/
    ./configure --prefix=$HOME/Software/cmake-3.23.2
    make 
    make install

Start a new shell to refresh the $PATH:

    type cmake
    cmake is /Users/phm/Software/cmake-3.23.2/bin/cmake





#Build and install libgeos

    cd ~/Builds
    tar -jxf ~/Downloads/geos-3.10.3.tar.bz2 
    cd geos-3.10.3
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Release ..
    export CMAKE_INSTALL_PREFIX="$HOME/Software/geos-3.10.3"
    make
    make install

Stupid developers assume incorrectly what I want when I specify the
installation prefix. No, without the /usr/local/ path, you morons:

    mv ~/Software/geos-3.10.3/usr/local/* ~/Software/geos-3.10.3/

Even more stupid:

    geos-config --includes
    /usr/local/include

Oh, for the love of goodness!

Set the CPATH and the LIBRARY_PATH environment variables so that the libgeos
files can be found by the PostGIS build.




#Build and install proj.org

The Cmake build system introduced with version proj-9.0.1 is broken, for me. 
The previous proj-8.2.1 uses GNU autotools to build, which succeeds on Apple
M1 Silicon.

There's a dependency on SQLite, so in addition to PostGres, we also need
SQLite...

    cd ~/Build
    tar -xf ~/Downloads/sqlite-autoconf-3390000.tar 
    cd sqlite-autoconf-3390000/
    ./configure --prefix=$HOME/Software/sqlite-3.39.0
    make
    make install

And LibTIFF

    cd ~/Builds
    tar -xf ~/Downloads/tiff-4.4.0.tar 
    cd tiff-4.4.0/
    ./configure --prefix=$HOME/Software/tiff-4.4.0
    make
    make install

That was refreshingly easy! Back to proj-8.2.1...

    cd ~/Builds
    tar -xf ~/Downloads/proj-8.2.1.tar
    cd proj-8.2.1/

    export SQLITE3_CFLAGS="$HOME/Software/sqlite-3.39.0/include"
    export SQLITE3_LIBS="$HOME/Software/sqlite-3.39.0/lib"
    export TIFF_CFLAGS="$HOME/Software/tiff-4.4.0/include"
    export TIFF_LIBS="$HOME/Software/tiff-4.4.0/lib"
    export LIBS="-ltiff -lsqlite3"

    ./configure --prefix=$HOME/Software/proj-8.2.1

    make
    make install

Success! (-: Notice that the linker needed a hint to pick up the Tiff and
Sqlite3 libraries.








#Build and install gdal.org

Avoiding GDAL version 3.5 because of an experimental CMake-based build.

    cd ~/Builds
    tar -xf ~/Downloads/gdal-3.4.3rc1.tar
    cd gdal-3.4.3/
    ./configure --prefix=$HOME/Software/gdal-3.4.3
    make
    make install




#Build and install LibXML2

No comment: like falling off a log...

    http://xmlsoft.org/download/libxml2-sources-2.9.10.tar.gz
    tar -xf ~/Downloads/libxml2-sources-2.9.10.tar 
    cd libxml2-2.9.10/
    ./configure --prefix=$HOME/Software/libxml2-2.9.10
    make
    make install



#Build and install JSON-C

Collect a tagged version as a source code archive.

https://github.com/json-c/json-c/releases/tag/json-c-0.16-20220414

Using the `cmake-configure` script. Not installing Doxygen.

    cd ~/Builds
    tar -xf ~/Downloads/json-c-json-c-0.16-20220414.tar 
    cd json-c-json-c-0.16-20220414/
    mkdir build
    cd build
    ../cmake-configure --prefix=$HOME/Software/json-c-0.16
    make
    make install

And, surprise, this time the Cmake build *does* correctly install the result
according to the `--prefix` option spec.




#Build and install GNU Autoconf

The `protobuf-c` tagged source archive needs `autoreconf`: 

    cd ~/Builds
    tar -xf ~/Downloads/autoconf-2.71.tar 
    cd autoconf-2.71/
    ./configure --prefix=$HOME/Software/autoconf-2.71
    make
    make install




#Build and install GNU Automake

The `autoreconf` tool needs `aclocal`

    cd ~/Builds
    tar -xf ~/Downloads/automake-1.16.5.tar 
    cd automake-1.16.5/
    ./configure --prefix=$HOME/Software/automake-1.15.5
    make
    make install


#Build and install GNU Libtool

The `aclocal` tool needs `libtool`... and on and on and on... The
installation of `aclocal` and `libtool` provided with Apple Xcode is
incomplete, missing `libtoolize`

    tar -xf ~/Downloads/libtool-2.4.6.tar 
    ./configure --prefix=$HOME/Software/libtool-2.4.6

This will fix the following message from `protobuf-c-1.4.0/autogen.ac`

Makefile.am:4: error: Libtool library used but 'LIBTOOL' is undefined
Makefile.am:4:   The usual way to define 'LIBTOOL' is to add 'LT_INIT'
Makefile.am:4:   to 'configure.ac' and run 'aclocal' and 'autoconf' again.
Makefile.am:4:   If 'LT_INIT' is in 'configure.ac', make sure
Makefile.am:4:   its definition is in aclocal's search path.

The `ACLOCAL_PATH` environment variable might be relevant. Keeping note of
it here:

    export ACLOCAL_PATH="$HOME/Software/automake-1.15.5/share/aclocal/"




#Build and install pkg-config

    cd ~/Builds/
    tar -xf ~/Downloads/pkg-config-pkg-config-0.29.2.tar.gz 
    cd pkg-config-pkg-config-0.29.2/

libtoolize: putting auxiliary files in '.'.
libtoolize: linking file './ltmain.sh'
libtoolize: You should add the contents of the following files to 'aclocal.m4':
libtoolize:   '/Users/phm/Software/libtool-2.4.6/share/aclocal/libtool.m4'
libtoolize:   '/Users/phm/Software/libtool-2.4.6/share/aclocal/ltoptions.m4'
libtoolize:   '/Users/phm/Software/libtool-2.4.6/share/aclocal/ltsugar.m4'
libtoolize:   '/Users/phm/Software/libtool-2.4.6/share/aclocal/ltversion.m4'
libtoolize:   '/Users/phm/Software/libtool-2.4.6/share/aclocal/lt~obsolete.m4'
libtoolize: Consider adding 'AC_CONFIG_MACRO_DIRS([m4])' to configure.ac,
libtoolize: and rerunning libtoolize and aclocal.
libtoolize: Consider adding '-I m4' to ACLOCAL_AMFLAGS in Makefile.am.


    aclocal
    autoconf
    libtoolize

Still getting errors:

    ./autogen.sh

glib.mk:28: error: Libtool library used but 'LIBTOOL' is undefined
glib.mk:28:   The usual way to define 'LIBTOOL' is to add 'LT_INIT'
glib.mk:28:   to 'configure.ac' and run 'aclocal' and 'autoconf' again.
glib.mk:28:   If 'LT_INIT' is in 'configure.ac', make sure
glib.mk:28:   its definition is in aclocal's search path.

Carrying on regardless:

    ./configure --prefix=$HOME/Software/pkg-config-0.29.2

NOPE!  AAARRRRRG!!!!

Giving up on all that nonsense.

I will build PostGIS without protobuf, after all...





#Build and install protobuf-c

Needs GNU Automake and Autoconf


https://github.com/protobuf-c/protobuf-c/tags

    cd ~/Builds
    tar -xf ~/Downloads/protobuf-c-1.4.0.tar
    cd protobuf-c-1.4.0/

    ./autogen.sh
    ./configure --prefix=$HOME/Software/protobuf-c-1.4.0
    make
    make install






#Build and install PostGIS

NOTE: Version postgis-3.2.1 has build problems on Mac M1 ARM64

 * https://groups.google.com/g/postgis-users/c/EzbXR8GFOKw
 * https://trac.osgeo.org/postgis/ticket/5091

Using postgis-3.1.5 instead.

The Postgres configuration tool knows where the stuff is. These guys are not
morons (-:

    pg_config --pkglibdir
    /Users/phm/Software/postgresql-14.3/lib

    pg_config --configure
    '--prefix=/Users/phm/Software/postgresql-14.3'

Trying to solve linker errors:

    export LDFLAGS="-stdlib=libc++"

    tar -xf ~/Downloads/postgis-3.1.5.tar 
    cd postgis-3.1.5/
    ./configure --without-protobuf
    make

PostGIS was built successfully. Ready to install.
HURRAH!!!

    make install

The `--without-protobuf` is specified because trying to build `protobuf-c`
was unsuccessful (frankly, it's broken!) and the option suppresses the
following configure error:

    configure: error: unable to find protobuf-c/protobuf-c.h using CPPFLAGS. 
    You can disable MVT and Geobuf support using --without-protobuf




The PostGIS extension is now installed in
`~/Software/postgresql-14.3/share/extension/`

This should solve the following error:

    2022-06-29 12:11:07.041 BST [41750] ERROR: \
      could not open extension control file \
      "/Users/phm/Software/postgresql-14.3/share/extension/postgis.control": \
      No such file or directory

    ls /Users/phm/Software/postgresql-14.3/share/extension/postgis.control
    /Users/phm/Software/postgresql-14.3/share/extension/postgis.control







#Kashet build requirements





    git clone --mirror ssh://github.com/Kashet/kashet.git kashet/.git
    cd kashet
    git config --local --bool core.bare false
    git reset --hard

First Kashet failure: the `Makefile` strictly specifies `python3.9`, but MacOS only
provides `python3` with version 3.8.9. Installing Python-3.9 from source...


##Python3.9

    cd ~/Builds
    tar -xf ~/Downloads/Python-3.9.13.tar 
    cd Python-3.9.13/
    ./configure \
      --prefix=$HOME/Software/${PWD##*/} \
      --enable-optimizations \
      --with-openssl=$HOME/Software/openssl-1.1.1o
    make
    make install

Second Kashet failure:

    $> make
    python3.9 deps/hive/scripts/config.py
    Traceback (most recent call last):
      File "/Users/phm/Repos/Kashet/kashet/deps/hive/scripts/config.py", line 3, in <module>
        import yaml
    ModuleNotFoundError: No module named 'yaml'
    make: *** [Makefile:42: config-nodes] Error 1
    error = 2 

Installing PyYAML, and its dependency LibYAML...


##PyYAML module

Requires LibYAML as dependency.

Download the tagged source archive https://github.com/yaml/libyaml/tags

    cd ~/Builds
    tar -xf ~/Downloads/libyaml-0.2.5.tar 
    ./bootstrap

Gives error:

    configure.ac:56: error: possibly undefined macro: AC_PROG_LIBTOOL
      If this token and others are legitimate, please use m4_pattern_allow.
      See the Autoconf documentation.

Trying suggestion: https://stackoverflow.com/a/53831263

    find ~/Software/ -name 'libtool.m4'
    /Users/phm/Software//libtool-2.4.6/share/aclocal/libtool.m4

Insert the path found into the `./bootstrap` script command:

    cat bootstrap
    #!/bin/sh
    exec autoreconf -fvi \
    -Wall,no-obsolete \
    -I$HOME/Software/libtool-2.4.6/share/aclocal/

Success!

    ./bootstrap
    ./configure --prefix=$HOME/Software/libyaml-0.2.5
    make
    make install

New terminal to refresh `$PATH` etc. to add LibYAML.

    cd ~/Builds
    tar -xf ~/Downloads/PyYAML-6.0.tar 
    cd PyYAML-6.0/
    python3.9 ./setup.py install

Third Kashet failure:

    $> make
    python3.9 deps/hive/scripts/config.py
    Traceback (most recent call last):
      File "/Users/phm/Repos/Kashet/kashet/deps/hive/scripts/config.py",
        line 241, in <module>
        raise TypeError(f'Invalid port configuration on {node}')
    TypeError: Invalid port configuration on api_node
    make: *** [Makefile:42: config-nodes] Error 1
    error = 2 

This is odd, as there appears to be the necessary data in the file:

    $> cat api_node/node.config.yml 
    ports:
      - '8080:8080'
    type: 'frontend'
    userConfig:
      port: 8080
    version:
      0.1.0

The `make` instruction works on this branch.

    git checkout hernan.new_hive_version_compatibility
    git pull origin hernan.new_hive_version_compatibility

    export MAMBU_USER="..."
    export MAMBU_PASS="..."
    export TN_USER="...@erlang-solutions.com"
    export TN_PASS='...'

    make
    make run

    ./scripts/smoke_tests.py 

    tail -F ../logs/adminapi/hive_node.log 
    tail -F ../logs/corebanking/hive_node.log 
    tail -F ../logs/adminapi/hive_node.log 

The failure stems from this server not responding:

    2022-07-07T12:04:18:::z phm@SugoiRingo:~/Repos/Kashet/kashet/scripts
    tail -F ../logs/adminapi/hive_node.log 

    2022-07-07T11:03:54.801470+00:00\
     /hive_node/apps/hive_requests/src/hive_requests.erl:294 error:\
     Gun error timeout
    2022-07-07T11:03:54.801739+00:00\
     /hive_node/apps/hive_users/src/hive_user.erl:48 warning:\
      unable to create user, reason: service_unavailable
    ^C

    $> ping erlang.sandbox.mambu.com
    PING sand-fend-ireland2.pvahghpazr.eu-west-1.elasticbeanstalk.com (18.200.146.22): 56 data bytes
    Request timeout for icmp_seq 0
    Request timeout for icmp_seq 1
    Request timeout for icmp_seq 2
    Request timeout for icmp_seq 3
    Request timeout for icmp_seq 4
    Request timeout for icmp_seq 5
    Request timeout for icmp_seq 6
    ^C
    --- sand-fend-ireland2.pvahghpazr.eu-west-1.elasticbeanstalk.com ping statistics ---
    8 packets transmitted, 0 packets received, 100.0% packet loss

Perhaps the sandbox account has expired?

Running a specific node in a local container:

    make node=kyc run_node

    make node=logic run_node

    (logic@172.19.0.2)6> hive_config:remote_node_types().
    [corebanking,kyc,notifications]

    (logic@172.19.0.2)14> ls ("/var/log/hive").
    hive_node.log     

    (logic@172.19.0.2)15> io:format("~s~n", \
      [element(2, file:read_file("/var/log/hive/hive_node.log"))] ).

    (logic@172.19.0.2)20> hive:sync_call(kyc,{lists,seq,[1,5]}).                    [1,2,3,4,5]          
    [1,2,3,4,5]          











#Python version 3.10

Date: 2022-07-11

    cd ~/Builds
    tar -xf ~/Downloads/Python-3.10.5.tar 
    cd Python-3.10.5/
    ./configure \
      --prefix=$HOME/Software/${PWD##*/} \
      --enable-optimizations \
      --with-openssl=$HOME/Software/openssl-1.1.1o
    make
    make install

New terminal to refresh `$PATH` etc. to add LibYAML.

    cd ~/Builds
    tar -xf ~/Downloads/PyYAML-6.0.tar 
    cd PyYAML-6.0/
    python3.10 ./setup.py install


#OTP/Erlang version 25

Date: 2022-07-12

Apply a patch to one C file:

    $> diff -p lib/wx/c_src/gen/wxe_wrapper_7.cpp{~,} 
    *** lib/wx/c_src/gen/wxe_wrapper_7.cpp~	2022-06-21 07:26:14.000000000 +0100
    --- lib/wx/c_src/gen/wxe_wrapper_7.cpp	2022-07-12 10:59:22.000000000 +0100
    *************** void wxToolBar_AddTool_4(WxeApp *app, wx
    *** 2339,2345 ****
          } else        Badarg("Options");
        };
        if(!This) throw wxe_badarg("This");
    !   wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(toolId,label,*bitmap,shortHelp,kind);
        wxeReturn rt = wxeReturn(memenv, Ecmd.caller, true);
        rt.send(  rt.make_ref(app->getRef((void *)Result,memenv), "wx"));
      
    --- 2339,2346 ----
          } else        Badarg("Options");
        };
        if(!This) throw wxe_badarg("This");
    !   wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(
    !     toolId,label,*bitmap,wxBitmapBundle(),kind,shortHelp);
        wxeReturn rt = wxeReturn(memenv, Ecmd.caller, true);
        rt.send(  rt.make_ref(app->getRef((void *)Result,memenv), "wx"));


    ./configure \
      --prefix=$HOME/Software/erlang-${PWD##*_} \
      --with-dynamic-trace=dtrace \
      --with-ssl=$HOME/Software/openssl-1.1.1o \
      --without-wxwidgets \
      --without-odbc \
      --without-javac \
      --enable-fips

    make && make install

Build and install Elixir

    tar -zxf ~/Downloads/elixir-1.13.4.tar 
    cd elixir-1.13.4/
    make clean
    make PREFIX=/Users/phm/Software/erlang-25.0.2
    make PREFIX=/Users/phm/Software/erlang-24.0.2 install











#GNU findutils

Depends on `xz` to decompress the source archive:

    cd ~/Builds
    tar -xf ~/Downloads/xz-5.2.5
    cd xz-5.2.5
    ./configure --prefix=$HOME/Software/${PWD##*/}
    make && make install

New shell to refresh `$PATH`

    cd ~/Builds
    xz -d ~/Downloads/findutils-4.9.0.tar.xz
    tar -xf ~/Downloads/findutils-4.9.0.tar 
    cd findutils-4.9.0
    ./configure --prefix=$HOME/Software/${PWD##*/}
    make && make install


#GNU sed

    cd ~/Builds/
    xz -d ~/Downloads/sed-4.8.tar.xz 
    tar -xf ~/Downloads/sed-4.8.tar 
    cd sed-4.8/
    ./configure --prefix=$HOME/Software/${PWD##*/}
    make && make install










#Git-SCM

Date 2022-08-01

https://git-scm.com/book/en/v2/Getting-Started-Installing-Git


Missing GNU libtool for gettext. Set `NO_GETTEXT` as work-around.

Missing asciidoc, xmlto & docbook2x. Skip building `doc` and `info` targets.

    cd ~/Builds/
    tar -xf ~/Downloads/git-2.37.1.tar 
    cd git-2.37.1/
    NO_GETTEXT=y make prefix="$HOME/Software/${PWD##*/}" all
    NO_GETTEXT=y make prefix="$HOME/Software/${PWD##*/}" install

The above last two lines as described in the `INSTALL` file:

    make prefix=/usr all doc info ;# as yourself
    make prefix=/usr install install-doc install-html install-info ;# as root



##Asciidoc

Installing from PyPI

Starting from 10.0 release, AsciiDoc.py can be installed from PyPI repository
by doing the following:

    $ python3 -m pip install asciidoc




##Gettext

There a couple of dependencies on gettext and libintl in Xmlto and Getopt

    cd ~/Builds/
    tar -xf ~/Downloads/gettext-0.21.tar 
    cd gettext-0.21/
    ./configure --prefix="$HOME/Software/${PWD##*/}"
    make
    make install


##Getopt

Needed to build xmlto. Missing `getopt`
"You need getopt from <http://software.frodo.looijaard.name/getopt/>".
And in turn Getopt needs gettext, 

The Makefile needs alteration, to avoid this linker error:

    gcc  -o getopt getopt.o
    Undefined symbols for architecture arm64:
      "_libintl_bindtextdomain", referenced from:
          _main in getopt.o
      "_libintl_gettext", referenced from:
          _main in getopt.o
          _parse_error in getopt.o
          _print_help in getopt.o
      "_libintl_setlocale", referenced from:
          _main in getopt.o
      "_libintl_textdomain", referenced from:
          _main in getopt.o
    ld: symbol(s) not found for architecture arm64
    clang: error: linker command failed with exit code 1 (use -v to see invocation)

The Mikefile line to change, adding `-lintl` for gettext libraries:

    LDFLAGS=-lintl

Build sequence:

    cd ~/Builds/
    tar -xf ~/Downloads/getopt-1.1.6.tar 
    cd getopt-1.1.6/
    make
    make install prefix="$HOME/Software/${PWD##*/}"



##Xmlto

Source archive https://releases.pagure.org/xmlto/

    cd ~/Builds/
    tar -xf ~/Downloads/xmlto-0.0.28.tar 
    cd xmlto-0.0.28/
    ./configure --prefix="$HOME/Software/${PWD##*/}"
    make
    make install

##Docbook2x

    cd ~/Builds/
    tar -xf ~/Downloads/docbook2X-0.8.8.tar 
    cd docbook2X-0.8.8/
    ./configure --prefix="$HOME/Software/${PWD##*/}"
    make
    make install


##XSL style sheet catalog

https://www.linuxfromscratch.org/blfs/view/8.1/pst/docbook-xsl.html

    cd ~/Builds/
    tar -xf ~/Downloads/docbook-xsl-1.79.1.tar.bz2 
    cd docbook-xsl-1.79.1/

    install -v -m755 -d "$HOME/Software/${PWD##*/}"

cp -v -R VERSION assembly common eclipse epub epub3 extensions fo        \
         highlighting html htmlhelp images javahelp lib manpages params  \
         profiling roundtrip slides template tests tools webhelp website \
         xhtml xhtml-1_1 xhtml5                                          \
         "$HOME/Software/${PWD##*/}"

ln -s VERSION "$HOME/Software/${PWD##*/}/VERSION.xsl"

install -v -m644    README \
                    "$HOME/Software/${PWD##*/}"/README.txt
install -v -m644    RELEASE-NOTES* NEWS* \
                    "$HOME/Software/${PWD##*/}"

Rebuild the catalog:

Catalog="$HOME/Software/${PWD##*/}/catalog"
Share="$HOME/Software/${PWD##*/}"

mkdir "${Share}"

xmkcatalog --noout --create "${Catalog}"

xmlcatalog --noout --add "rewriteSystem" \
           "http://docbook.sourceforge.net/release/xsl/1.79.1" \
           "${Share}" "${Catalog}"

xmlcatalog --noout --add "rewriteURI" \
           "http://docbook.sourceforge.net/release/xsl/1.79.1" \
           "${Share}" "${Catalog}"

xmlcatalog --noout --add "rewriteSystem" \
           "http://docbook.sourceforge.net/release/xsl/current" \
           "${Share}" "${Catalog}"

xmlcatalog --noout --add "rewriteURI" \
           "http://docbook.sourceforge.net/release/xsl/current" \
           "${Share}" "${Catalog}"

export XML_CATALOG_FILES="${HOME}/Software/docbook-xsl-1.79.1/catalog"

cd "${Share}/html" && ln -s ../VERSION.xsl


##Git-SCM
Make Git-SCM, without the `info`:

cd ~/Builds/git-2.37.1

make XMLTO='xmlto --skip-validation' \
     prefix="$HOME/Software/${PWD##*/}" \
     all doc

Lots of warnings:

    ASCIIDOC git-commit.xml
    XMLTO git-commit.1
I/O error : Attempt to load network entity http://www.oasis-open.org/docbook/xml/4.5/docbookx.dtd
/Users/phm/Builds/git-2.37.1/Documentation/git-commit.xml:2: warning: failed to load external entity "http://www.oasis-open.org/docbook/xml/4.5/docbookx.dtd"
D DocBook XML V4.5//EN" "http://www.oasis-open.org/docbook/xml/4.5/docbookx.dtd"

make prefix="$HOME/Software/${PWD##*/}" install

I give up! I can't buid Git with full documentation and internationalisation

    cd ~/Builds/
    tar -xf ~/Downloads/git-2.37.1.tar 
    cd git-2.37.1/
    make prefix="$HOME/Software/${PWD##*/}" all
    make prefix="$HOME/Software/${PWD##*/}" install








# NMep

https://nmap.org/download.html#source

    cd ~/Builds/
    tar -xf ~/Downloads/nmap-7.92.tar 
    cd nmap-7.92/
    ./configure --prefix=$HOME/Software/${PWD##*/}
    make && make install







# Coreutils

https://www.gnu.org/software/coreutils/coreutils.html

    cd ~/Builds/
    tar -zxf ~/Downloads/coreutils-9.1.tar.gz
    cd coreutils-9.1
    ./configure --prefix=$HOME/Software/${PWD##*/}
    make && make install





# MKV-tools, and dependencies

Sources for Ogg and Vorbis https://xiph.org/downloads/

    cd ~/Builds
    tar -zxf  ~/Downloads/libogg-1.3.5.tar.gz 
    cd libogg-1.3.5/
    ./configure --prefix="$HOME/Software/${PWD##*/}"
    make
    make install

    cd ~/Builds
    tar -Jxf ~/Downloads/libvorbis-1.3.7.tar.xz 
    cd libvorbis-1.3.7/
    ./configure --prefix="$HOME/Software/${PWD##*/}"
    make
    make install

The build of mkvtool uses qmake from the Qt project.
https://github.com/qt/qtbase/releases/tag/v5.15.8-lts-lgpl

Giving up -- building entire Qt just for this tool is not worth the effort!








# OpenMP libraries


https://github.com/llvm/llvm-project/archive/refs/tags/llvmorg-16.0.2.tar.gz

tar -zxf ~/Downloads/llvm-project-llvmorg-16.0.2.tar.gz

cd llvm-project-llvmorg-16.0.2/openmp/
mkdir build
cd build
cmake -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ ..

cmake -DCMAKE_INSTALL_PREFIX=${HOME}/Software/openmp-16.0.2 ..

make
make install












# OpenSSL 3.0.11

https://www.openssl.org/source/openssl-3.0.11.tar.gz

mkdir ~/Builds
cd ~/Builds
tar -zxf ~/Downloads/openssl-3.0.11.tar.gz
cd ./openssl-3.0.11/
./Configure --prefix="${HOME}/Software/openssl-3.0.11"
make
make test
make install


# WX-Widgets

https://wxwidgets.org/downloads/
wxWidgets-3.2.2.1.tar.bz2

rm -r ~/Software/wxWidgets-3.2.2.1/
chmod ugo-rwx ~/Software/wxWidgets-3.1.6



cd ~/Builds
tar -jxf ~/Downloads/wxWidgets-3.2.2.1.tar.bz2 
cd ./wxWidgets-3.2.2.1/
./configure \
 --prefix="${HOME}/Software/${PWD##*/}" \
 --enable-webview \
 --enable-compat28
make && make install





# OTP

https://github.com/erlang/otp/releases/download/OTP-26.1/
otp_src_26.1.tar.gz
otp_doc_html_26.1.tar.gz

rm -r ~/Builds/otp_src_26.1/
echo -e "${PATH//:/\\\n}"
echo -e "${LDFLAGS//-L/\\\n-L}"

cd ~/Builds
tar -zxf ~/Downloads/otp_src_26.1.tar.gz
cd ./otp_src_26.1/

export PATH=${HOME}/Software/wxWidgets-3.2.2.1/bin:\
${HOME}/Software/openssl-3.0.11/bin:\
${PATH}

export CXX=/usr/bin/g++
export CXXFLAGS="-fpermissive"

./configure \
--prefix="${HOME}/Software/erlang-26.1" \
--with-dynamic-trace=dtrace \
--with-ssl="${HOME}/Software/openssl-3.0.11" \
--with-wxwidgets="${HOME}/Software/wxWidgets-3.2.2.1" \
--without-odbc \
--without-javac


gen/wxe_wrapper_7.cpp:2342:58:
 error: call to member function 'AddTool' is ambiguous


./lib/wx/c_src/gen/wxe_wrapper_7.cpp
cp ./lib/wx/c_src/gen/wxe_wrapper_7.cpp{,-ORIGINAL}

1045> diff ./lib/wx/c_src/gen/wxe_wrapper_7.cpp{-ORIGINAL,}
2342c2342,2348
<   wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(toolId,label,*bitmap,shortHelp,kind);
---
>   // phm fix:-
>   // "error: call to member function 'AddTool' is ambiguous"
>   // Using arguments that match "the full AddTool function"
>   // as described in ./wxWidgets-3.2.2.1/include/wx/tbarbase.h
>   // to remove ambiguity
>   wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(
>     toolId,label,*bitmap,wxBitmapBundle(),kind,shortHelp);




make SHELL='sh -x'

+ echo ' LD	../priv/aarch64-apple-darwin22.6.0/wxe_driver.so'
 LD	../priv/aarch64-apple-darwin22.6.0/wxe_driver.so
+ /usr/bin/g++ \
 -mmacosx-version-min=10.10 \
 -bundle \
 -flat_namespace \
 -undefined warning \
 -fPIC \
 ${LDFLAGS} \
 -framework IOKit \
 -framework Carbon \
 -framework Cocoa \
 -framework QuartzCore \
 -framework AudioToolbox \
 -framework System \
 -framework OpenGL \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_stc-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_xrc-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_gl-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_aui-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_webview-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_html-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_osx_cocoau_core-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_baseu_xml-3.2.a \
 /Users/phm/Software/wxWidgets-3.2.2.1/lib/libwx_baseu-3.2.a \
 -framework OpenGL \
 -framework AGL \
 -framework WebKit \
 -lwxjpeg-3.2 \
 -lwxpng-3.2 \
 -lwxregexu-3.2 \
 -lwxscintilla-3.2 \
 -lexpat \
 -ltiff \
 -lz \
 -framework Security \
 -lpthread \
 -liconv \
 -o ../priv/aarch64-apple-darwin22.6.0/wxe_driver.so
ld: warning: -undefined warning is deprecated
ld: warning: -undefined warning is deprecated
ld: Undefined symbols:
  _enif_alloc, referenced from:
      WxeApp::OnInit() in wxe_impl.o
      _newMemEnv in wxe_impl.o
      _newMemEnv in wxe_impl.o
  _enif_alloc_env, referenced from:
      wxControlWithItems_Append_2(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_2.o
      wxControlWithItems_appendStrings_2(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_2.o
      wxControlWithItems_setClientData(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_2.o
      wxControlWithItems_Insert_3(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_2.o
      wxControlWithItems_insertStrings_3(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_2.o
      wxEvtHandler_Connect(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_3.o
      wxTreeCtrl_AddRoot(WxeApp*, wxeMemEnv*, wxeCommand&) in wxe_wrapper_7.o
      ...

clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[3]: *** [Makefile:181: ../priv/aarch64-apple-darwin22.6.0/wxe_driver.so] Error 1
make[3]: Leaving directory '/Users/phm/Builds/otp_src_26.1/lib/wx/c_src'


NOTE: this could be caused by the --disable-dynamic configuration option in
wxWidgets.



I absolutely lack context and haven't tried this myself, but I saw
this which rings a bell so I'll paste it here with the hope it helps:

https://github.com/bjorng/otp/commit/cb165869063478c2061b7ec23e0ebb3f1d53df23




# Erlang/OTP from latest commit

2023-09-25T11:30

Using the latest commit on `master` of Erlang/OTP repository where the patch
Radek suggested has been merge in.

    cd ~/Repos/
    git clone https://github.com/erlang/otp.git Github-erlang-otp
    cd ./Github-erlang-otp/
    git checkout master
    joe lib/wx/c_src/gen/wxe_wrapper_7.cpp

    $> git diff
    diff --git a/lib/wx/c_src/gen/wxe_wrapper_7.cpp b/lib/wx/c_src/gen/wxe_wrapper_7.cpp
    index f4716e1228..4e375f8646 100644
    --- a/lib/wx/c_src/gen/wxe_wrapper_7.cpp
    +++ b/lib/wx/c_src/gen/wxe_wrapper_7.cpp
    @@ -2339,7 +2339,8 @@ void wxToolBar_AddTool_4(WxeApp *app, wxeMemEnv *memenv, wxeCommand& Ecmd)
         } else        Badarg("Options");
       };
       if(!This) throw wxe_badarg("This");
    -  wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(toolId,label,*bitmap,shortHelp,kind);
    +  wxToolBarToolBase * Result = (wxToolBarToolBase*)This->AddTool(
    +    toolId,label,*bitmap,wxBitmapBundle(),kind,shortHelp);
       wxeReturn rt = wxeReturn(memenv, Ecmd.caller, true);
       rt.send(  rt.make_ref(app->getRef((void *)Result,memenv), "wx"));

    ./configure \
      --prefix="${HOME}/Software/erlang-26.1" \
      --with-dynamic-trace=dtrace \
      --with-ssl="${HOME}/Software/openssl-3.0.11" \
      --with-wxwidgets="${HOME}/Software/wxWidgets-3.2.2.1" \
      --without-odbc \
      --without-javac 
    make
    make install

    $> erl
    Erlang/OTP 27 [DEVELOPMENT] [erts-14.1] [source-7ea625a785] [64-bit] [smp:10:10] [ds:10:10:10] [async-threads:1] [jit] [dtrace]

    Eshell V14.1 (press Ctrl+G to abort, type help(). for help)
    1> wx:demo().
    ok



# Elixir

The `${PREFIX}` value is specified for installation alongside the Erlang/OTP

    git clone https://github.com/elixir-lang/elixir.git Github-elixir-lang
    cd ./Github-elixir-lang/
    git checkout v1.15.6
    make PREFIX=/Users/phm/Software/erlang-26.1 clean
    make PREFIX=/Users/phm/Software/erlang-26.1 compile
    make PREFIX=/Users/phm/Software/erlang-26.1 install

Open a new terminal to refresh the environment:

    $> iex
    Erlang/OTP 27 [DEVELOPMENT] [erts-14.1] [source-7ea625a785] [64-bit] [smp:10:10] [ds:10:10:10] [async-threads:1] [jit] [dtrace]

    Interactive Elixir (1.15.6) - press Ctrl+C to exit (type h() ENTER for help)
    iex(1)> 

Success!





# Upgrade rebar3

https://github.com/erlang/rebar3/releases
https://codeload.github.com/erlang/rebar3/tar.gz/refs/tags/3.22.1

    cd ~/Builds/
    tar -zxf ~/Downloads/rebar3-3.22.1.tar.gz 
    cd ./rebar3-3.22.1/
    ./bootstrap

    ./rebar3 --version
    rebar 3.22.1 on Erlang/OTP 27 Erts 14.1

    mkdir -p ~/Software/rebar3-3.22.1/bin
    cp ./rebar3 ~/Software/rebar3-3.22.1/bin/

Launch a new shell to update the `$PATH` environment.

Notice that there is Bash shell-completion script provided too:

    ./apps/rebar/priv/shell-completion/bash/rebar3





# Ghostscript

Ghostscript Print Description Language

[Releases](https://ghostscript.com/releases/gpdldnld.html)

    tar -zxf ~/Downloads/ghostpdl-10.02.0.tar.gz 
    cd ghostpdl-10.02.0/
    ./configure  --prefix="${HOME}/Software/${PWD##*/}"  
    make -j10 && make install

In a new terminal, to refresh the value of `$PATH`

    ps2pdf  ~/Downloads/jmc.ps ~/Documents/jmc.pdf



# ImageMagick

Not successful. There's a known "won't fix" defect when running `display` on
Mac OSX.

    git clone https://github.com/ImageMagick/ImageMagick.git ImageMagick
    cd ImageMagick
    ./configure --prefix="${HOME}/Software/ImageMagick-7.1.1"  
    make && make install

    git clone https://github.com/ImageMagick/ImageMagick.git ImageMagick
    cd ImageMagick
    ./configure \
    --enable-delegate-build \
    --with-x=yes \
    --prefix="${HOME}/Software/ImageMagick-7.1.1"  
    make && make install

    export DISPLAY=:0


    display: delegate library support not built-in '' (X11) @ error/display.c/DisplayI




https://www.xquartz.org/

https://github.com/orgs/Homebrew/discussions/4993

https://github.com/ImageMagick/ImageMagick/issues/5818

Example options for building Imagemajick:

    ./configure \
    CPPFLAGS='-I/opt/local/include' \
    LDFLAGS='-L/opt/local/lib' \
     --enable-delegate-build \
     --enable-shared \
     --disable-static \
     --with-modules \
     --with-quantum-depth=16 \
     --enable-hdri --with-gslib \
     --disable-silent-rules \
     --disable-dependency-tracking \
     --with-rsvg \
     --with-gs-font-dir=/opt/local/share/ghostscript/fonts/ \
     --with-x \
     --prefix=${HOME}/bin/ImageMagick7.0.8-65

