FROM old-go-ubuntu
MAINTAINER godev@thoughtworks.com


#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/139/dist-all/1/all/pkg/go-server-[go-version-under-test].deb
#RUN wget --no-check-certificate --user=view --password=password --auth-no-challenge https://build.go.cd/go/files/distributions-all/139/dist-all/1/all/pkg/go-agent-[go-version-under-test].deb


RUN wget http://download.go.cd/local/[go-version-under-test]/go-server-[go-version-under-test].deb


RUN dpkg -i go-server-[go-version-under-test].deb


# The following is needed to use replace old log4j.properties with the new one
# RUN dpkg -i --force-confnew go-agent-[go-version-under-test].deb

EXPOSE 8153
EXPOSE 8154

VOLUME  ["/var/log/", "/etc/go"]

# Cleanup
#RUN rm -rf go-server-[go-version-under-test].deb

CMD /etc/init.d/go-agent start ; /etc/init.d/go-server start


