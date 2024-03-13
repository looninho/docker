FROM ubuntu:22.04

LABEL maintainer=<looninho@gmail.com>

ENV DEBIAN_FRONTEND noninteractive
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ARG USER=docker_user

RUN apt-get update && apt-get -y --quiet --no-install-recommends install \
	ca-certificates \
	ccache \
	curl \
	git \
	gnupg \
	lsb-release \
	python3-dev \
	python3-pip \
	apparmor \
	apparmor-profiles \
	apparmor-utils \
	libatomic1 \
	&& apt-get -y autoremove \
	&& apt-get clean autoclean \
	&& rm -rf /var/lib/apt/lists/{apt,dpkg,cache,log} /tmp/* /var/tmp/*

# Some QT-Apps/Gazebo don't not show controls without this
ENV QT_X11_NO_MITSHM 1

# Install Olympe
RUN pip3 install parrot-olympe

################################
#   Sphinx parrot
################################
RUN curl --fail --silent --show-error --location https://debian.parrot.com/gpg | gpg --dearmor | tee /usr/share/keyrings/debian.parrot.com.gpg > /dev/null

RUN echo "deb [signed-by=/usr/share/keyrings/debian.parrot.com.gpg] https://debian.parrot.com/ $(lsb_release -cs) main generic" | tee /etc/apt/sources.list.d/debian.parrot.com.list

# The following lines permit to answer questions that are otherwise raised
# during the package installation. But answering these questions during the docker image
# build seems impossible.
# To see the properties, install the package and then run debconf-get-selections | grep parrot-sphinx
RUN echo "parrot-sphinx sphinx/firmwared_users string ${USER}" | debconf-set-selections
RUN echo "parrot-sphinx sphinx/license_approval boolean true" | debconf-set-selections

RUN apt update -q && DEBIAN_FRONTEND=noninteractive apt install -q -y \
		parrot-sphinx

## install UE4 application
RUN apt install -y parrot-ue4-empty

# Create a user
RUN groupadd -g 1001 -r $USER && \
    useradd -u 1000 -g 1001 --create-home -r $USER && \
    echo "$USER:$USER" | chpasswd && \
    mkdir -p /etc/sudoers.d && \
    echo "${USER} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$USER && \
    chmod 0440 /etc/sudoers.d/$USER && \
    chown 1000:1001 -R /home/$USER && \
    usermod -aG sudo $USER && usermod -a -G dialout $USER

USER $USER
ENV HOME /home/$USER
CMD [ "/sbin/init" ]
