# escape=`
FROM centos:7.8.2003

# Image metadata for documentation purposes. Has no effect on docker build behavior.
LABEL `
    org.label-schema.schema-version="1.0" `
    org.label-schema.name="CLM5.0 Development Environment" `
    org.label-schema.version="0.2" `
    org.label-schema.vendor="HPSC TerrSys" `
    org.label-schema.license="GPLv2" `
    org.label-schema.build-date="20200529" `
    clm.version.major="5" `
    clm.env.compiler="GCC" `
    clm.env.compiler.version="9.x" `
    clm.env.mpi="OpenMPI" `
    clm.env.mpi.version="3.1.x" `
    clm.env.cmake.version="3.17.x" `
    clm.env.pnetcdf.version="1.12.1" `
    clm.env.netcdf-c.version="4.8.0-dev" `
    clm.env.netcdf-fortran.version="4.6.0-dev" `
    clm.env.esmf.version="8.0.1"

# Import cime config files to image
WORKDIR /tmp/.cime
COPY cime_config/* /tmp/.cime/

# Enable man pages on CentOS docker container. See https://unix.stackexchange.com/a/493499
RUN sed -i 's/tsflags=nodocs/#tsflags=nodocs/' /etc/yum.conf `
    `
# Install required packages from base repo
    && yum update -y `
    && yum install -y file git install libcurl-devel libtool m4 make man-db man-pages sudo vim wget which tree `
                      centos-release-scl epel-release `
                      perl-App-cpanminus perl-autodie perl-Env `
    `
# Install required packages from EPEL and SCL repos
    && yum install -y cmake3 devtoolset-9-gcc* hdf5-devel lapack-devel openmpi3-devel ncl nco python3 python3-pip `
    && yum clean all `
    `
# Install LibXML Perl library which is needed by CLM5
    && cpanm XML::LibXML `
    `
# Set cmake3 as the default cmake command
    && alternatives --install /usr/local/bin/cmake cmake /usr/bin/cmake3 20 `
                    --slave /usr/local/bin/ctest ctest /usr/bin/ctest3 `
                    --slave /usr/local/bin/cpack cpack /usr/bin/cpack3 `
                    --slave /usr/local/bin/ccmake ccmake /usr/bin/ccmake3 `
                    --family cmake

# Required envs for building libraries
ENV PATH=/usr/lib64/openmpi3/bin:${PATH}
ENV LD_LIBRARY_PATH=/usr/lib64/openmpi3/lib:${LD_LIBRARY_PATH}
ENV CPATH=/usr/include/openmpi3-x86_64:/usr/include/openmpi3-x86_64/mpp:/usr/include/openmpi3-x86_64/openshmem/oshmem:/usr/include/lapacke:${CPATH}
ENV CPLUS_INCLUDE_PATH=/usr/include/openmpi3-x86_64/openmpi/ompi/mpi/cxx:${CPLUS_INCLUDE_PATH}
ENV ESMF_DIR=/tmp/bld/esmf-ESMF_8_0_1
ENV ESMF_INSTALL_PREFIX=/usr/local
ENV ESMF_INSTALL_MODDIR=include
ENV ESMF_INSTALL_LIBDIR=lib
ENV ESMF_INSTALL_BINDIR=bin
ENV ESMF_NETCDF=split
ENV ESMF_NETCDF_INCLUDE=/usr/local/include
ENV ESMF_NETCDF_LIBS="-lnetcdf -lnetcdff"
ENV ESMF_NETCDF_LIBPATH=/usr/local/lib64
ENV ESMF_NETCDF_FLIBPATH=/usr/local/lib64
ENV ESMF_NETCDF_CPATH=/usr/local
ENV ESMF_NETCDF_FPATH=/usr/local

# Prepare build environment
RUN source /opt/rh/devtoolset-9/enable `
    && mkdir -vp /tmp/bld `
    && cd /tmp/bld `
    `
# Build and install PnetCDF
    && wget -c https://github.com/Parallel-NetCDF/PnetCDF/archive/checkpoint.1.12.1.tar.gz `
    && tar zxf checkpoint.1.12.1.tar.gz `
    && cd PnetCDF-checkpoint.1.12.1 `
    && autoreconf -i `
    && ./configure --prefix=/usr/local && make && make tests && make install `
    `
# Build and install netCDF C libraries
    && mkdir -vp /tmp/bld/netcdf-c `
    && cd /tmp/bld/netcdf-c `
    && git clone https://github.com/Unidata/netcdf-c.git . `
    && git checkout e1ef7a958c29 `
    && cmake -DENABLE_PARALLEL4=ON -S . -B ./bld `
    && cd bld && make -j && make test && make install `
    `
# Build and install netCDF Fortran libraries
    && mkdir -vp /tmp/bld/netcdf-fortran `
    && cd /tmp/bld/netcdf-fortran `
    && git clone https://github.com/Unidata/netcdf-fortran.git . `
    && git checkout d796c83449 `
    && cmake -LA -DCMAKE_C_COMPILER=/opt/rh/devtoolset-9/root/usr/bin/gcc -DCMAKE_Fortran_COMPILER=/opt/rh/devtoolset-9/root/usr/bin/gfortran -S . -B ./bld `
    && cd bld && make -j && make test && make install `
    `
# Build and install ESMF libraries
    && cd /tmp/bld `
    && wget -c https://github.com/esmf-org/esmf/archive/ESMF_8_0_1.tar.gz `
    && tar zxf ESMF_8_0_1.tar.gz `
    && cd esmf-ESMF_8_0_1 `
    && make -j && make install `
    `
# Cleanup
    && cd /tmp && rm -rf /tmp/bld

# Add user account
WORKDIR /home
RUN groupadd -g 1555 clm5docker `
    && useradd clmuser -g clm5docker`
    && echo clm5 | passwd --stdin clmuser `
    && echo "clmuser  ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers `
    `
# Set compiler and library paths for CLM5
    && echo "" >> /home/clmuser/.bash_profile `
    && echo "# Set compiler and library paths for CLM5" >> /home/clmuser/.bash_profile `
    && echo "source /opt/rh/devtoolset-9/enable" >> /home/clmuser/.bash_profile `
    && echo "source /etc/profile.d/ncarg.sh" >> /home/clmuser/.bash_profile `
    && echo "export PATH=/usr/lib64/openmpi3/bin:\$PATH" >> /home/clmuser/.bash_profile `
    && echo "export LD_LIBRARY_PATH=/usr/lib64/openmpi3/lib:/usr/local/lib64:\$LD_LIBRARY_PATH" >> /home/clmuser/.bash_profile `
    && echo "export CPATH=/usr/include/openmpi3-x86_64:/usr/include/openmpi3-x86_64/mpp:/usr/include/openmpi3-x86_64/openshmem/oshmem:/usr/include/lapacke:\$CPATH" >> /home/clmuser/.bash_profile `
    && echo "export CPLUS_INCLUDE_PATH=/usr/include/openmpi3-x86_64/openmpi/ompi/mpi/cxx:\$CPLUS_INCLUDE_PATH" >> /home/clmuser/.bash_profile `
    `
# Set CLM5 environment variables
    && echo "" >> /home/clmuser/.bash_profile `
    && echo "# CLM5 environment variables" >> /home/clmuser/.bash_profile `
    && echo "export PROJECT=/p/project" >> /home/clmuser/.bash_profile `
    && echo "export SCRATCH=/p/scratch" >> /home/clmuser/.bash_profile `
    && echo "export BUDGET_ACCOUNTS=slts" >> /home/clmuser/.bash_profile `
    && echo "export CESMDATAROOT=/p/scratch/CESMDATAROOT" >> /home/clmuser/.bash_profile `
    && echo "export CSMDATA=\$CESMDATAROOT/inputdata" >> /home/clmuser/.bash_profile `
    `
# Copy cime config files to home directory
    && mv /tmp/.cime /home/clmuser/.cime

USER clmuser
WORKDIR /home/clmuser
CMD ["/bin/bash","-l"]
