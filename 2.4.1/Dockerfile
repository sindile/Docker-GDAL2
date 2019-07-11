##
# geographica/gdal2
#
# This creates an Ubuntu derived base image that installs GDAL 2.
#
# Ubuntu 18.04 Bionic Beaver
FROM ubuntu:bionic

MAINTAINER Cayetano Benavent <cayetano.benavent@geographica.gs>

ENV ROOTDIR /usr/local/
ENV GDAL_VERSION 2.4.1
ENV OPENJPEG_VERSION 2.3.0
ENV FILEGDB_VERSION 1_5_1

# Load assets
WORKDIR $ROOTDIR/

ADD http://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz $ROOTDIR/src/
ADD https://github.com/uclouvain/openjpeg/archive/v${OPENJPEG_VERSION}.tar.gz $ROOTDIR/src/openjpeg-${OPENJPEG_VERSION}.tar.gz
ADD https://github.com/Esri/file-geodatabase-api/raw/master/FileGDB_API_1.5.1/FileGDB_API_${FILEGDB_VERSION}-64gcc51.tar.gz $ROOTDIR/src/FileGDB_API_${FILEGDB_VERSION}-64gcc51.tar.gz

# Install basic dependencies
RUN apt-get update -y && apt-get install -y \
    software-properties-common \
    build-essential \
    python-dev \
    python3-dev \
    python-numpy \
    python3-numpy \
    libspatialite-dev \
    sqlite3 \
    libpq-dev \
    libcurl4-gnutls-dev \
    libproj-dev \
    libxml2-dev \
    libgeos-dev \
    libnetcdf-dev \
    libpoppler-dev \
    libspatialite-dev \
    libhdf4-alt-dev \
    libhdf5-serial-dev \
    bash-completion \
    cmake
    
# Install File Geodatabase
RUN cd src && tar -zxvf FileGDB_API_${FILEGDB_VERSION}-64gcc51.tar.gz && \
    cp -r FileGDB_API-64gcc51/lib/* /usr/local/lib && \
    cp -r FileGDB_API-64gcc51/include/* /usr/local/include && \
    ldconfig -v

# Compile and install OpenJPEG
RUN cd src && tar -xvf openjpeg-${OPENJPEG_VERSION}.tar.gz && cd openjpeg-${OPENJPEG_VERSION}/ \
    && mkdir build && cd build \
    && cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$ROOTDIR \
    && make && make install && make clean \
    && cd $ROOTDIR && rm -Rf src/openjpeg*

# Compile and install GDAL
RUN cd src && tar -xvf gdal-${GDAL_VERSION}.tar.gz && cd gdal-${GDAL_VERSION} \
    && ./configure --with-python --with-spatialite --with-pg --with-curl --with-openjpeg --with-fgdb=$ROOTDIR/src/FileGDB_API-64gcc51 \
    && make -j $(nproc) && make install && ldconfig \
    && apt-get update -y \
    && cd $ROOTDIR && cd src/gdal-${GDAL_VERSION}/swig/python \
    && python3 setup.py build \
    && python3 setup.py install \
    && apt-get remove -y --purge build-essential \
      python-dev \
      python3-dev \
    && cd $ROOTDIR && rm -Rf src/gdal*

# Output version and capabilities by default.
CMD gdalinfo --version && gdalinfo --formats && ogrinfo --formats
