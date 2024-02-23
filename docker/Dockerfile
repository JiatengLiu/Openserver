####################################
###### CONTENT for Dockerfile ######
####################################

# Extends from father image [CHANGE WITH YOUR NEED]
FROM nvidia/cuda:11.6.2-devel-ubuntu20.04

# Set locale
ENV DEBIAN_FRONTEND noninteractive

# Change anaconda source
# ADD means copy file from host machine to containers
ADD sources.list /etc/apt/
ENV PATH /opt/conda/bin:$PATH

# Install basic dependencies
#RUN rm /etc/apt/sources.list.d/cuda.list && \
#	rm /etc/apt/sources.list.d/nvidia-ml.list
	
RUN apt-get update && apt-get install -y --no-install-recommends \
	bzip2 \
	g++ \
	git \
	vim \
	python-dev \
	python3-pip \
	build-essential \
	wget && \
	rm -rf /var/lib/apt/lists/*

# Install Anaconda for python 
ADD Anaconda3-2023.09-0-Linux-x86_64.sh /home/anaconda.sh
RUN /bin/bash /home/anaconda.sh -b -p /opt/conda && \
	ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
	rm /home/anaconda.sh

# Change sources for conda, add tsinghua sources and remove defaults
RUN conda config --add channels http://mirrors.aliyun.com/anaconda/pkgs/msys2 && \
	conda config --add channels http://mirrors.aliyun.com/anaconda/pkgs/r && \
	conda config --add channels http://mirrors.aliyun.com/anaconda/pkgs/main
   - 
  - 
  - http://mirrors.aliyun.com/anaconda/pkgs/main
  - defaults


# Change sources for pip3
RUN mkdir ~/.pip && \
	echo "[global]\nindex-url = http://mirrors.aliyun.com/pypi/simple/\n[install]\ntrusted-host = mirrors.aliyun.com" > ~/.pip/pip.conf

# Initialize workspace
RUN mkdir /workspace
WORKDIR /workspace

CMD ["/bin/bash"]

