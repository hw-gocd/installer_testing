FROM ubuntu:12.04
MAINTAINER godev1@thoughtworks.com

RUN apt-get update
RUN apt-get -y -q install openssl wget openjdk-7-jre-headless unzip git


#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/204/dist-all/1/all/pkg/go-server-[go-version-under-test].deb
#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/204/dist-all/1/all/pkg/go-agent-[go-version-under-test].deb

RUN wget http://download.go.cd/local/[go-version-under-test]/go-server-[go-version-under-test].deb
RUN wget http://download.go.cd/local/[go-version-under-test]/go-agent-[go-version-under-test].deb


RUN dpkg -i go-server-[go-version-under-test].deb
RUN dpkg -i go-agent-[go-version-under-test].deb

EXPOSE 8153
EXPOSE 8154

RUN sed -i -e"s/^DAEMON=.*$/DAEMON=N/" /etc/default/go-server

VOLUME  ["/var/log/", "/etc/go"]

# Cleanup
RUN rm -rf go-server-[go-version-under-test].deb go-agent-[go-version-under-test].deb

ENTRYPOINT /etc/init.d/go-agent start ; /etc/init.d/go-server start


