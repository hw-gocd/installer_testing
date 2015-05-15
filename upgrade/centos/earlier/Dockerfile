FROM centos:centos6
MAINTAINER godev1@thoughtworks.com

RUN apt-get update
RUN apt-get -y -q install openssl wget openjdk-7-jre-headless unzip git



RUN wget http://download.go.cd/gocd-deb/go-server-14.2.0-377.noarch.rpm
RUN wget http://download.go.cd/gocd-deb/go-agent-14.2.0-377.noarch.rpm


RUN rpm -i go-server-14.2.0-377.noarch.rpm
RUN rpm -i go-agent-14.2.0-377.noarch.rpm

EXPOSE 8153
EXPOSE 8154

RUN sed -i -e"s/^DAEMON=.*$/DAEMON=N/" /etc/default/go-server

VOLUME  ["/var/log/", "/etc/go"]

# Cleanup
RUN rm -rf go-server-14.2.0-377.noarch.rpm go-agent-14.2.0-377.noarch.rpm

ENTRYPOINT /etc/init.d/go-agent start ; /etc/init.d/go-server start





