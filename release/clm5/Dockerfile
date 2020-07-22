# escape=`
FROM centos:7.8.2003

# Image metadata for documentation purposes. Has no effect on docker build behavior.
LABEL `
    org.label-schema.schema-version="1.0" `
    org.label-schema.name="CLM5.0 Development Environment" `
    org.label-schema.version="0.3" `
    org.label-schema.vendor="IBG-3" `
    org.label-schema.license="GPLv2" `
    org.label-schema.build-date="20200622" `
    clm.version.major="5" `
    clm.env.compiler="GCC" `
    clm.env.compiler.version="9.3.1-2" `
    clm.env.mpi="OpenMPI" `
    clm.env.mpi.version="4.0.4" `
    clm.env.lapack.version="3.9.0" `
    clm.env.hdf5.version="1.10.6" `
    clm.env.cmake.version="3.17" `
    clm.env.pnetcdf.version="1.12.1" `
    clm.env.netcdf-c.version="4.6.3" `
    clm.env.netcdf-fortran.version="4.4.5" `
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
    && yum install -y cmake3 devtoolset-9-gcc* hdf5-devel ncl nco ninja-build python3 python3-pip `
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
    && wget -c https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.4.tar.gz `
    && tar zxf openmpi-4.0.4.tar.gz `
    && cd openmpi-4.0.4 `
    && ./configure --prefix=/usr && make -j8 all && make install && ldconfig `
    `
# LAPACK
    && wget -c https://github.com/Reference-LAPACK/lapack/archive/v3.9.0.tar.gz `
    && tar zxf v3.9.0.tar.gz `
    && cd lapack-3.9.0 `
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local -S . -B ./bld `
    && cd bld && make -j && make install `
    `
# HDF5
    && cd /tmp/bld `
    && wget -c https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.6/src/CMake-hdf5-1.10.6.tar.gz `
    && tar zxf CMake-hdf5-1.10.6.tar.gz `
    && cd CMake-hdf5-1.10.6 `
    && sed -i 's/HDF5_BUILD_FORTRAN:BOOL=OFF/HDF5_BUILD_FORTRAN:BOOL=ON/' HDF5options.cmake `
    && ./build-unix.sh `
    && ./HDF5-1.10.6-Linux.sh --prefix=/opt --exclude-subdir --skip-license `
    `
# netCDF C
    && cd /tmp/bld `
    && wget -c https://github.com/Unidata/netcdf-c/archive/v4.6.3.tar.gz `
    && tar zxf v4.6.3.tar.gz `
    && cd netcdf-c-4.6.3 `
    && cmake -DENABLE_PARALLEL4=ON -DCMAKE_PREFIX_PATH=/opt/HDF_Group/HDF5/1.10.6 -DCMAKE_INSTALL_PREFIX=/usr/local -S . -B ./bld `
    && cd bld && make -j && make install `
    `
# netCDF Fortran
    && cd /tmp/bld `
    && wget -c https://github.com/Unidata/netcdf-fortran/archive/v4.4.5.tar.gz `
    && tar zxf v4.4.5.tar.gz `
    && cd netcdf-fortran-4.4.5 `
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local -S . -B ./bld `
    && cd bld && make && make install `
    `
# PnetCDF
    && cd /tmp/bld `
    && wget -c https://github.com/Parallel-NetCDF/PnetCDF/archive/checkpoint.1.12.1.tar.gz `
    && tar zxf checkpoint.1.12.1.tar.gz `
    && cd PnetCDF-checkpoint.1.12.1 `
    && autoreconf -i `
    && ./configure --prefix=/usr/local --enable-netcdf4 --with-netcdf4=/usr/local && make -j && make install `
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
    && echo "" >> /home/clmuser/.bash_profile >> /home/clmuser/.bash_profile `
    && echo "LD_LIBRARY_PATH=/usr/local/lib64" >> /home/clmuser/.bash_profile `
    && echo "source /opt/rh/devtoolset-9/enable" >> /home/clmuser/.bash_profile `
    && echo "source /etc/profile.d/ncarg.sh" >> /home/clmuser/.bash_profile `
    `
# Set CLM5 environment variables
    && echo "" >> /home/clmuser/.bash_profile `
    && echo "# CLM5 environment variables" >> /home/clmuser/.bash_profile `
    && echo "" >> /home/clmuser/.bash_profile `
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
