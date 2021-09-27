# escape=`
FROM centos:8.4.2105

# Image metadata for documentation purposes. Has no effect on docker build behavior.
LABEL `
    org.label-schema.schema-version="1.0" `
    org.label-schema.name="eCLM" `
    org.label-schema.version="0.1" `
    org.label-schema.vendor="IBG-3" `
    org.label-schema.license="GPLv2" `
    org.label-schema.build-date="2021-09-27" `
    eclm.version.major="alpha" `
    eclm.env.compiler="GCC" `
    eclm.env.compiler.version="8.4.1-1" `
    eclm.env.mpi="OpenMPI" `
    eclm.env.mpi.version="4.0.5" `
    eclm.env.cmake.version="3.18.2" `
    eclm.env.pnetcdf.version="1.12.1" `
    eclm.env.netcdf-c.version="4.7.0" `
    eclm.env.netcdf-fortran.version="4.5.2"

# Install required packages from base repo
RUN dnf update -y `
    && dnf group install -y "Development Tools" `
    && dnf install -y glibc-langpack-en libcurl-devel `
                      man-db man-pages passwd sudo vim wget which tree `
                      python39 python39-pip cmake`
                      epel-release `
    && dnf config-manager --set-enabled powertools `
    `
# Install required packages from EPEL and SCL repos
    && dnf install -y hdf5-devel lapack-devel openssl-devel `
                      openmpi openmpi-devel `
                      netcdf-devel netcdf-fortran netcdf-fortran-devel `
    && dnf clean all `
    `
# Add python symlinks to /usr/local
    && python3 -m pip install --upgrade pip `
    && ln -s /usr/bin/python3 /usr/local/bin/python

# Install PnetCDF
ENV PATH="/usr/lib64/openmpi/bin:${PATH}"
RUN mkdir -vp /tmp/bld `
    && cd /tmp/bld `
    && wget -c https://github.com/Parallel-NetCDF/PnetCDF/archive/checkpoint.1.12.1.tar.gz `
    && tar zxf checkpoint.1.12.1.tar.gz `
    && cd PnetCDF-checkpoint.1.12.1 `
    && autoreconf -i `
    && ./configure --prefix=/usr/local && make -j && make install `
    && cd / && rm -rf /tmp/bld

# Add user account
RUN groupadd -g 1555 clm5docker `
    && useradd clmuser -g clm5docker`
    && echo clm5 | passwd --stdin clmuser `
    && usermod -aG wheel clmuser `
    && rm /run/nologin `
    `
# Set default envs
    && echo "export PATH=/usr/lib64/openmpi/bin:/home/clmuser/.local/bin:$PATH" >> /home/clmuser/.bash_profile `
    && echo "export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:/usr/lib64/pkgconfig" >> /home/clmuser/.bash_profile `
    && echo "export CC=mpicc" >> /home/clmuser/.bash_profile `
    && echo "export FC=mpifort" >> /home/clmuser/.bash_profile `
    && echo "alias ll='ls -phAlv'" >> /home/clmuser/.bash_profile
USER clmuser
WORKDIR /home/clmuser
CMD ["/bin/bash","-l"]
