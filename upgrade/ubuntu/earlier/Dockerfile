FROM ubuntu:12.04
MAINTAINER godev1@thoughtworks.com

RUN apt-get update
RUN apt-get -y -q install openssl wget openjdk-7-jre-headless unzip git sudo



RUN wget http://download.go.cd/gocd-deb/go-server-14.2.0-377.deb
RUN wget http://download.go.cd/gocd-deb/go-agent-14.2.0-377.deb


RUN dpkg -i go-server-14.2.0-377.deb
RUN dpkg -i go-agent-14.2.0-377.deb

EXPOSE 8153
EXPOSE 8154

RUN sed -i -e"s/^DAEMON=.*$/DAEMON=N/" /etc/default/go-server

VOLUME  ["/var/log/", "/etc/go"]

RUN /etc/init.d/go-server stop

RUN git clone https://testusergo:%21abcd1234@github.com/gocd/installer-testing.git
RUN cp installer-testing/upgrade/cruise-config-for-go-14.2.xml /tmp/cruise-config.xml

RUN sudo su - go

RUN rm -rf /etc/go/cruise-config.xml
RUN mv /tmp/cruise-config.xml /etc/go
#RUN chown go:go /etc/go/cruise-config.xml

# Cleanup
RUN rm -rf go-server-14.2.0-377.deb go-agent-14.2.0-377.deb

CMD /etc/init.d/go-agent start ; /etc/init.d/go-server start





