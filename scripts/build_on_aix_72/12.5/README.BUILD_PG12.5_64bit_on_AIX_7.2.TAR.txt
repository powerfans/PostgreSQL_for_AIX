O## Build on K1 Power9 AIX 7.2(7200-05-02-2114)

# oslevel -s
7200-05-02-2114

# prtconf |head -10
System Model: IBM,9119-MHE

### 2. Build postgres ##########################################################################################

# cat /opt/freeware/etc/yum/yum.conf
[main]
cachedir=/var/cache/yum
keepcache=1
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
plugins=1

[AIX_Toolbox]
name=AIX generic repository
baseurl=http://anonymous:anonymous@XXX.XXX.XXX.XXX/os/aix/aixtoolbox/RPMS/ppc/
enabled=1
gpgcheck=0

[AIX_Toolbox_noarch]
name=AIX noarch repository
baseurl=http://anonymous:anonymous@XXX.XXX.XXX.XXX/os/aix/aixtoolbox/RPMS/noarch
enabled=1
gpgcheck=0


[AIX_Toolbox_72]
name=AIX 7.2 specific repository
baseurl=http://anonymous:anonymous@XXX.XXX.XXX.XXX/os/aix/aixtoolbox/RPMS/ppc-7.2/
enabled=1
gpgcheck=0


[oss4aix_rpm]
name=oss4aix_rpm
baseurl=http://anonymous:anonymous@XXX.XXX.XXX.XXX/os/aix/oss4aix.org/latest/aix72
enabled=1
gpgcheck=0

# yum clean all; yum repolist
# yum install tightvnc tightvnc-server bash unzip vim
# yum install gcc gcc-g++
# type gcc
gcc is /opt/freeware/bin/gcc
# gcc --version
gcc (GCC) 8.3.0

### Install dependencies
# yum install bison flex gawk make gettext gettext-devel ncurses expat sqlite libffi  perl \
               zlib zlib-devel realine readline-devel python3 python3-devel \
               tcl tcl-devel krb5-devel krb5-libs  openldap openldap-devel \
               libxml2 libxml2-devel uuid uuid-devel libxslt libxslt-devel

# bash; export PATH=/opt/freeware/bin:$PATH
# export PATH=/opt/freeware/bin:$PATH
# export LIBPATH=/opt/freeware/lib64:/usr/lib64:/opt/freeware/lib:/usr/lib:/lib
# export PERL=/opt/freeware/bin/perl_64
# export PYTHON=/opt/freeware/bin/python3_64

###Download PostgreSQL source from https://ftp.postgresql.org/pub/source/v12.5/postgresql-12.5.tar.gz
# cd /home; gzip -d postgresql-12.5.tar.gz; 
# tar xf postgresql-12.5.tar; cd /home/postgresql-12.5

### Adjust Mafile to avoid Undefined symbol issue
# cd /home/postgresql-12.5/src
# sed -i 's/\$(stlib) -Wl,-bE:\$(exports_file)/\$^ -Wl,-bexpfull/g' Makefile.shlib
# cd /home/postgresql-12.5/src 
# cp Makefile.shlib Makefile.shlib.pq
# sed -i '350,351s/^/#/g' Makefile.shlib.pq
# cd /home/postgresql-12.5/src/interfaces/libpq
# sed -i 's/Makefile.shlib.*/Makefile.shlib.pq/g' Makefile
### Link as 64-bit, should use 'nm -X64 ...'
# cd /home/postgresql-12.5/                                    
# sed -i 's/\$NM/\$NM \-X64/g' src/backend/port/aix/mkldexport.sh

# cd /home/postgresql-12.5;
OBJECT_MODE=64 \
AR="ar -X64" \
LD="ld -b64" \
CC="gcc -g -maix64 -O2" \
CXX="g++ -g -maix64 -O2" \
CPP="cpp -g -maix64 -O2" \
CFLAGS="-I./src/include -I./src/interfaces/ecpg/include -I/opt/freeware/include -I/usr/include" \
CPPFLAGS="-I./src/include -I./src/interfaces/ecpg/include -I/opt/freeware/include -I/usr/include" \
LDFLAGS="-L/opt/freeware/lib64 -L/usr/lib64 -L/opt/freeware/lib -L/usr/lib -Wl,-blibpath:/opt/freeware/lib64:/opt/freeware/lib:/usr/lib:/lib -lintl -lpthreads" \
LDFLAGS_EX="-Wl,-brtl" \
LDFLAGS_SL="-Wl,-G" \
./configure \
    --prefix=/opt/postgres/12.5 \
    --libdir=/opt/postgres/12.5/lib64 \
    --enable-thread-safety \
    --with-perl \
    --with-uuid=ossp \
    --with-blocksize=8 --with-segsize=1 --with-wal-blocksize=8 \
    --sysconfdir=/etc/sysconfig/postgresql

#gmake -j8 VERBOSE=1 world
#gmake -j8 VERBOSE=1 install-world

# cd /opt/postgres
# tar zcf postgresql12-server-12.5-1PGDG.aix.7.2.tar.gz ./12.5
