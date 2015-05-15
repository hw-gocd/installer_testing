FROM old-go-centos
MAINTAINER godev@thoughtworks.com


#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/139/dist-all/1/all/pkg/go-server-14.4.0-1343.noarch.rpm
#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/139/dist-all/1/all/pkg/go-agent-14.4.0-1343.noarch.rpm


RUN wget http://download.go.cd/local/[go-version-under-test]/go-server-[go-version-under-test].noarch.rpm


RUN rpm -Uvh go-server-[go-version-under-test].noarch.rpm


EXPOSE 8153
EXPOSE 8154

VOLUME  ["/var/log/", "/etc/go"]

# Cleanup
RUN rm -rf go-server-[go-version-under-test].noarch.rpm

ENTRYPOINT /etc/init.d/go-agent start ; /etc/init.d/go-server start


