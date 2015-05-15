FROM centos:centos6
MAINTAINER godev@thoughtworks.com

RUN yum -y update
RUN yum -y -q install wget java-1.7.0-openjdk unzip git which



#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/204/dist-all/1/all/pkg/go-server-[go-version-under-test].noarch.rpm
#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/204/dist-all/1/all/pkg/go-agent-[go-version-under-test].noarch.rpm


RUN wget http://download.go.cd/local/[go-version-under-test]/go-server-[go-version-under-test].noarch.rpm
RUN wget http://download.go.cd/local/[go-version-under-test]/go-agent-[go-version-under-test].noarch.rpm


RUN rpm -i go-server-[go-version-under-test].noarch.rpm
RUN rpm -i go-agent-[go-version-under-test].noarch.rpm

EXPOSE 8153
EXPOSE 8154

RUN sed -i -e"s/^DAEMON=.*$/DAEMON=N/" /etc/default/go-server

VOLUME  ["/var/log/", "/etc/go"]

# Cleanup
RUN rm -rf go-server-[go-version-under-test].noarch.rpm go-agent-[go-version-under-test].noarch.rpm

ENTRYPOINT /etc/init.d/go-agent start ; /etc/init.d/go-server start



