FROM centos:centos7
# USER root

ARG MAVEN_VERSION=3.6.0
ARG MAVEN_URL=https://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz

# MongoDB stuff / ansible / OC client
RUN touch 'mongodb-org-4.0.repo' && \
    echo $'[mongodb-org-4.0] \n\
name=MongoDB \n\
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/ \n\
gpgcheck=1 \n\
enabled=1 \n\
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc' >  mongodb-org-4.0.repo  && \
    mv mongodb-org-4.0.repo /etc/yum.repos.d/  && \
    curl -sL https://rpm.nodesource.com/setup_8.x | bash && \
    yum install -y mongodb-org zsh git curl gcc-c++ g++ automake python build-essential wget sudo nodejs java-1.8.0-openjdk-devel psmisc epel-release && \
    yum install -y ansible && \
    mkdir -p /usr/share/maven && \
    curl -fsSL ${MAVEN_URL} | tar -xzC /usr/share/maven --strip-components=1 && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn && \
    yum clean all

# Install jq
# http://stedolan.github.io/jq/
RUN curl -o /usr/local/bin/jq -L https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 && \
  chmod +x /usr/local/bin/jq

# Set the JAVA_HOME variable to make it clear where Java is located
ENV JAVA_HOME /usr/lib/jvm/java

# Add Let's Encrypt CA to OS trsuted store
RUN curl -o /etc/pki/ca-trust/source/anchors/lets-encrypt-x3-cross-signed.crt https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt && \
    update-ca-trust extract

# Install OpenShift Client
RUN curl -m 800 https://mirror.openshift.com/pub/openshift-v3/clients/3.11.98/linux/oc.tar.gz | tar -C /usr/local/bin/ -xzf -

## User account
RUN useradd -ms /bin/bash -d /home/theia theia && \
  chmod g+rw /home && \
  mkdir -p /home/theia/project && \
  mkdir -p /home/theia/ide && \
  chown -R theia:theia /home/theia

ENV HOME /home/theia
USER theia

ENV MAVEN_HOME /home/theia
ENV MAVEN_CONFIG ${MAVEN_HOME}/.m2
ENV MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1"

# NodeJS Stuff
RUN unset NPM_CONFIG_PREFIX && \
  git clone https://github.com/robbyrussell/oh-my-zsh.git $HOME/.oh-my-zsh && \
  cp $HOME/.oh-my-zsh/templates/zshrc.zsh-template $HOME/.zshrc && \
  echo 'unset NPM_CONFIG_PREFIX' >> $HOME/.zshrc && \
  echo 'unset NPM_CONFIG_PREFIX' >> $HOME/.bashrc && \
  curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | sh  && \
  echo 'export NVM_DIR="$HOME/.nvm"' >> $HOME/.zshrc && \
  echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm' >> $HOME/.zshrc && \
  echo 'export NVM_DIR="$HOME/.nvm"' >> $HOME/.bashrc && \
  echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm' >> $HOME/.bashrc && \
  source $HOME/.bashrc && \
  nvm install v10 && \
  nvm install v8 && \
  nvm install v6 && \
  nvm alias default v8  && \
  npm install -g @angular/cli yarn

# Theia IDE install stuff 
ARG GITHUB_TOKEN
ENV GITHUB_TOKEN=${GITHUB_TOKEN}
WORKDIR $HOME/ide

ENV NODE_OPTIONS=--max_old_space_size=8000

RUN source $HOME/.bashrc && \
#   echo '{"private":true,"dependencies":{"@theia/application-package":"next","@theia/typescript":"next","typescript":"latest","@theia/navigator":"next","@theia/terminal":"next","@theia/plugin-ext-vscode":"next","@theia/monaco":"next","@theia/editorconfig":"next","@theia/outline-view":"next","@theia/preferences":"next","@theia/messages":"next","@theia/git":"next","@theia/file-search":"next","@theia/markers":"next","@theia/preview":"next","@theia/callhierarchy":"next","@theia/merge-conflicts":"next","@theia/search-in-workspace":"next","@theia/json":"next","@theia/textmate-grammars":"next","@theia/mini-browser":"next","@theia/plugin-ext":"next", "@theia/extension-manager":"next"},"devDependencies":{"@theia/cli":"next"}}' > package.json && \
  echo '{"private":true,"dependencies":{"@theia/application-package":"0.7.0-next.086ce720","@theia/typescript":"0.7.0-next.086ce720","typescript":"latest","@theia/navigator":"0.7.0-next.086ce720","@theia/terminal":"0.7.0-next.086ce720","@theia/plugin-ext-vscode":"0.7.0-next.086ce720","@theia/monaco":"0.7.0-next.086ce720","@theia/editorconfig":"0.7.0-next.086ce720","@theia/outline-view":"0.7.0-next.086ce720","@theia/preferences":"0.7.0-next.086ce720","@theia/messages":"0.7.0-next.086ce720","@theia/git":"0.7.0-next.086ce720","@theia/file-search":"0.7.0-next.086ce720","@theia/markers":"0.7.0-next.086ce720","@theia/preview":"0.7.0-next.086ce720","@theia/callhierarchy":"0.7.0-next.086ce720","@theia/merge-conflicts":"0.7.0-next.086ce720","@theia/search-in-workspace":"0.7.0-next.086ce720","@theia/json":"0.7.0-next.086ce720","@theia/textmate-grammars":"0.7.0-next.086ce720","@theia/mini-browser":"0.7.0-next.086ce720","@theia/plugin-ext":"0.7.0-next.086ce720", "@theia/extension-manager":"0.7.0-next.086ce720"},"devDependencies":{"@theia/cli":"0.7.0-next.086ce720"}}' > package.json && \
  npm install -g yarn && \
  yarn install --force  
RUN source $HOME/.bashrc && \
  ls -la  && \
  yarn theia build

WORKDIR /home/theia/project
# generic server ports
EXPOSE 4200 4444 8080 9000 3000 8081 8082 8083 8084 3000

ADD https://raw.githubusercontent.com/rht-labs/openshift-theia-ide/master/enablement-workspace-theia/entrypoint /home/theia
USER root
RUN chgrp -R 0 /home/theia && \
  chmod a+x /home/theia/entrypoint && \
  chmod -R g=u /home/theia && \
  chmod g=u /etc/passwd
USER 10001
ENTRYPOINT ["/home/theia/entrypoint"]
CMD source $HOME/.bashrc && cd /home/theia/ide && yarn theia start /home/project --hostname 0.0.0.0 --port 8080
