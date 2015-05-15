FROM go-server-upgrade-to-latest-ubuntu
MAINTAINER godev@thoughtworks.com


RUN wget http://download.go.cd/local/[go-version-under-test]/go-agent-[go-version-under-test].deb


RUN dpkg -i --force-confnew go-agent-[go-version-under-test].deb


EXPOSE 8153
EXPOSE 8154

VOLUME  ["/var/log/", "/etc/go"]

# Cleanup
#RUN rm -rf go-agent-[go-version-under-test].deb

ENTRYPOINT /etc/init.d/go-agent start ; /etc/init.d/go-server start


