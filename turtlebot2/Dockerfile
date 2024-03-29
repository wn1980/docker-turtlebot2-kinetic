FROM ros:kinetic-perception-xenial

LABEL maintainer="Waipot Ngamsaad <waipotn@hotmail.com>"

SHELL ["/bin/bash", "-c"]

ENV DEBIAN_FRONTEND noninteractive

RUN  apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

RUN sed -i -e 's/http:\/\/archive/mirror:\/\/mirrors/' -e 's/http:\/\/security/mirror:\/\/mirrors/' -e 's/\/ubuntu\//\/mirrors.txt/' /etc/apt/sources.list

# install Ubuntu packages
RUN apt-get update && apt-get upgrade -y 
RUN apt-get install -y \
    apt-transport-https \
    build-essential \
    curl \
    git \
    wget \
    nano \
    bash-completion \
    htop \
    tmux \
    terminator \
    xfonts-base \
    xauth \
    x11-xkb-utils \
    xkb-data \
    dbus-x11 \
    net-tools \
    usbutils \
    sudo \
    #tigervnc-standalone-server \
    #tigervnc-xorg-extension \
    #novnc \
    #fluxbox \
    #jwm \
    #less \
    python3-numpy \
    python-pip \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# install ROS packages
#RUN sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list' && \
#    curl -L http://packages.osrfoundation.org/gazebo.key | apt-key add -

RUN apt-get update --fix-missing && apt-get upgrade -y 
RUN apt-get install -y \
    ros-${ROS_DISTRO}-desktop-full \
    ros-${ROS_DISTRO}-turtlebot* \
    ros-${ROS_DISTRO}-linux-peripheral-interfaces \
    ros-${ROS_DISTRO}-diagnostics \
    ros-${ROS_DISTRO}-navigation \
    ros-${ROS_DISTRO}-hector-slam \
    ros-${ROS_DISTRO}-octomap-server \
    ros-${ROS_DISTRO}-octomap-rviz-plugins \
    ros-${ROS_DISTRO}-vision-opencv \
    ros-${ROS_DISTRO}-depth-image-proc \
    ros-${ROS_DISTRO}-joy \
    ros-${ROS_DISTRO}-serial \
    ros-${ROS_DISTRO}-openni2-launch \
    ros-${ROS_DISTRO}-gmapping \
    #ros-${ROS_DISTRO}-plotjuggler-ros \
    python-rosdep \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# update pip and install some modules
RUN curl https://bootstrap.pypa.io/pip/2.7/get-pip.py | python - && \
    pip install -U --no-cache-dir \
        supervisor \
        supervisor_twiddler

# init rosdep
RUN rm /etc/ros/rosdep/sources.list.d/20-default.list && \
    rosdep init && \
    rm /etc/apt/apt.conf.d/docker-clean

# install nodejs
RUN sh -c 'echo "deb https://deb.nodesource.com/node_8.x `lsb_release -cs` main" > /etc/apt/sources.list.d/nodesource.list' && \
    curl -sSL https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add -

RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y \
    nodejs \
	&& rm -rf /var/lib/apt/lists/*

# install code-server
ENV VERSION=3.12.0
RUN wget https://github.com/cdr/code-server/releases/download/v${VERSION}/code-server_${VERSION}_$(dpkg --print-architecture).deb && \
    dpkg -i code-server_${VERSION}_$(dpkg --print-architecture).deb && \
    rm -f code-server_${VERSION}_$(dpkg --print-architecture).deb

# setup user space
RUN mkdir -p {/workspace,/app}

RUN useradd -m developer && \
    usermod -aG sudo developer && \
    usermod --shell /bin/bash developer && \
    chown -R developer:developer /workspace && \
    chown -R developer:developer /app && \
    ln -sfn /workspace /home/developer/workspace && \
    ln -sfn /app /home/developer/app && \
    echo developer ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/developer && \
    chmod 0440 /etc/sudoers.d/developer

ENV USER=developer

ENV HOME /home/developer

ENV SHELL /bin/bash

USER $USER

WORKDIR /home/developer

# init rosdep
RUN rosdep fix-permissions && rosdep update

# colorize less
#RUN echo "export LESS='-R'" >> ~/.bash_profile && \
#    echo "export LESSOPEN='|pygmentize -g %s'" >> ~/.bash_profile

# enable bash completion
RUN echo "source /usr/share/bash-completion/bash_completion" >> ~/.bashrc && \
    #git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it && \
    #~/.bash_it/install.sh --silent && \
    #rm ~/.bashrc.bak && \
    #bash -i -c "bash-it enable completion git" && \
    echo "source ~/.bashrc" >> ~/.bash_profile 

COPY ./app /app

RUN sudo chown -R developer:developer /app

RUN echo "source /opt/ros/$ROS_DISTRO/setup.bash" >> ~/.bashrc && \
	#echo "source ~/turtlebot_ws/install/setup.bash" >> ~/.bashrc && \
    echo "export TURTLEBOT_3D_SENSOR=asus_xtion_pro" >> ~/.bashrc && \
    echo "export TURTLEBOT_MAP_FILE=~/workspace/maps/map.yaml" >> ~/.bashrc

VOLUME /tmp/.X11-unix

ENV DISPLAY ":1"

EXPOSE 11311 8008

CMD ["sudo", "-E", "/usr/local/bin/supervisord", "-c", "/app/supervisord.conf"]
