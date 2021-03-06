# Originally from https://github.com/M0E-lnx/ubuntu-32bit with thanks,
# and then https://github.com/matthew-brett/trusty/blob/32/Dockerfile

FROM scratch
# Download from https://partner-images.canonical.com/core/trusty/current/ubuntu-trusty-core-cloudimg-i386-root.tar.gz
ADD ubuntu-trusty-core-cloudimg-i386-root.tar.gz /

# a few minor docker-specific tweaks
# see https://github.com/docker/docker/blob/master/contrib/mkimage/debootstrap
RUN echo '#!/bin/sh' > /usr/sbin/policy-rc.d \
    && echo 'exit 101' >> /usr/sbin/policy-rc.d \
    && chmod +x /usr/sbin/policy-rc.d \
    \
    && dpkg-divert --local --rename --add /sbin/initctl \
    && cp -a /usr/sbin/policy-rc.d /sbin/initctl \
    && sed -i 's/^exit.*/exit 0/' /sbin/initctl \
    \
    && echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup \
    \
    && echo 'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' > /etc/apt/apt.conf.d/docker-clean \
    && echo 'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' >> /etc/apt/apt.conf.d/docker-clean \
    && echo 'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";' >> /etc/apt/apt.conf.d/docker-clean \
    \
    && echo 'Acquire::Languages "none";' > /etc/apt/apt.conf.d/docker-no-languages \
    \
    && echo 'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";' > /etc/apt/apt.conf.d/docker-gzip-indexes \
    && echo 'deb http://archive.ubuntu.com/ubuntu/ trusty universe' >> /etc/apt/sources.list \
    && echo 'deb http://archive.ubuntu.com/ubuntu/ trusty-updates universe' >> /etc/apt/sources.list

#
# Pillow customization
#

RUN apt-get -qq update && DEBIAN_FRONTEND=noninteractive apt-get -y \
    install xvfb \
    git wget python-numpy python-scipy netpbm \
    python-qt4 ghostscript libffi-dev libjpeg-turbo-progs \
    python-setuptools python-virtualenv \
    python-dev python3-dev cmake \
    libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev \
    python-tk python3-tk \
    libharfbuzz-dev libfribidi-dev && apt-get clean

RUN useradd pillow && addgroup pillow sudo && \
    mkdir /home/pillow && chown pillow:pillow /home/pillow

RUN virtualenv -p /usr/bin/python2.7 --system-site-packages /vpy && \
    /vpy/bin/pip install cffi olefile pytest pytest-cov && \
    chown -R pillow:pillow /vpy

ADD depends /depends
RUN cd /depends && ./install_openjpeg.sh && ./install_imagequant.sh && ./install_raqm.sh

RUN echo "#!/bin/bash" >> /test && \
    echo "source /vpy/bin/activate && cd /Pillow " >> test && \
    echo "export DISPLAY=:99.0" >> test && \
    echo "make clean && make install-coverage && /usr/bin/xvfb-run -a pytest -vx --cov PIL --cov-report term Tests" >> test

RUN chmod +x /test

USER pillow
ENTRYPOINT ["linux32"]
CMD ["/test"]

#docker run -v $TRAVIS_BUILD_DIR:/Pillow pythonpillow/ubuntu-trusty-x86
