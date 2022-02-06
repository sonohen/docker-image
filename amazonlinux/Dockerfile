FROM amazonlinux:2

ARG user=ec2-user
ARG pass=secret
ARG git_user_name=$user
ARG git_user_email=sonohen55@gmail.com

ENV WORK_DIRECTORY=/tmp/initialize
ENV AWS_CLI_URL=https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip
ENV AWS_CLI_FILENAME=awscliv2.zip
ENV USER_HOME_DIR=/home/$user

WORKDIR /

# Install Node.js repo
# See https://github.com/nodesource/
RUN curl -fsSL https://rpm.nodesource.com/setup_16.x | bash -

# Install required packages as root
RUN yum -y update && yum -y install \
	gcc \
	git \
	glibc \
	groff \
	less \
	nodejs \
	openssh-clients \
	sudo \
	tar \
	unzip \
	util-linux-user \
	vim \
	zsh

# Change timezone to Asia/Tokyo
RUN cp -p /usr/share/zoneinfo/Japan /etc/localtime

# Create $WORK_DIRECTORY to download and extract AWS CLI programs
RUN mkdir -p $WORK_DIRECTORY
WORKDIR $WORK_DIRECTORY
RUN curl $AWS_CLI_URL -o $AWS_CLI_FILENAME && unzip $AWS_CLI_FILENAME && ./aws/install

# Add user to avoid running the programs as root user
RUN useradd $user -m -d $USER_HOME_DIR -s /usr/bin/zsh -G wheel -p "$(mkpasswd --method=sha-512 '$pass')"
USER $user

# The following is running as $user (Very important!)

WORKDIR $USER_HOME_DIR

# Copy ssh credentials and aws credential file (API KEY/API SECRET)
RUN mkdir -m 700 -p .ssh/
COPY --chown=$user:$user ssh/ $USER_HOME_DIR/.ssh/
RUN chmod 600 .ssh/*

COPY --chown=$user:$user udemy-sample1.pem $USER_HOME_DIR
RUN chmod 400 udemy-sample1.pem

RUN mkdir -m 700 -p .aws/
COPY --chown=$user:$user aws/ $USER_HOME_DIR/.aws/
RUN chmod 600 .aws/*

# Apply git basic configuration
RUN git config --global user.name $git_user_name
RUN git config --global user.email $git_user_email

# Download dotfiles from my private GitHub repository (secret)
RUN git clone git@github.com:sonohen/dotfiles.git
RUN rm -f $USER_HOME_DIR/.zshrc && ln -s ./dotfiles/.zshrc $USER_HOME_DIR/.zshrc
RUN rm -f $USER_HOME_DIR/.vimrc && ln -s ./dotfiles/.vimrc $USER_HOME_DIR/.vimrc

# Install oh-my-zsh
RUN sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Install Rust
RUN curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh -s -- -y

CMD ["/usr/bin/zsh"]
