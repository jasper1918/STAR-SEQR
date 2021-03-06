FROM ubuntu:16.04

MAINTAINER Jeff Jasper jasper1918@gmail.com

RUN apt-get -y update --fix-missing && \
    apt-get install -y curl wget gzip bzip2 unzip git \
    g++ make libboost-dev libboost-thread-dev libboost-system-dev zlib1g-dev ncurses-dev \
    libglib2.0-0 libxext6 libsm6 libxrender1 libxml2-dev libxslt-dev ca-certificates git gcc libdb5.3 libdb5.3-dev libcurl4-openssl-dev && \
    apt-get clean && dpkg-reconfigure locales && locale-gen en_US.UTF-8 && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install miniconda to /opt/conda
RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda2-4.1.11-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh

# Use Tini
RUN apt-get install -y curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb

# Set envinroment
ENV PATH /opt/conda/bin:$PATH
ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]

# Install dependencies for STAR-SEQR
WORKDIR /opt

#Install new STAR
RUN git clone https://github.com/alexdobin/STAR.git /opt/STAR
ENV PATH /opt/STAR/bin/Linux_x86_64_static/:$PATH

RUN wget http://ccb.jhu.edu/software/stringtie/dl/gffread-0.9.8c.Linux_x86_64.tar.gz && \
    tar -zxvf gffread-0.9.8c.Linux_x86_64.tar.gz
ENV PATH /opt/gffread-0.9.8c.Linux_x86_64/:$PATH

RUN conda config --add channels conda-forge && \
    conda config --add channels defaults && \
    conda config --add channels r && \
    conda config --add channels bioconda && \
    conda install -y pip nose coverage cython pysam pandas samtools biobambam velvet ucsc-gtftogenepred salmon && \
    conda clean -ilty

# Install STAR-SEQR
RUN wget https://github.com/ExpressionAnalysis/STAR-SEQR/archive/v0.5.0.tar.gz  && \
    tar -zxvf v0.5.0.tar.gz && \
    cd STAR-SEQR-0.5.0 && \
    python setup.py build && \
    python setup.py install && \
    python setup.py clean && \
    nosetests --with-coverage --cover-package=starseqr_utils

# Install items for basespace
RUN pip install awscli

COPY run_basespace.py /opt/

ENV PATH /opt/run_basespace.py:$PATH
