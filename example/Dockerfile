FROM centos:7.1.1503
MAINTAINER qmxie@thoughtworks.com

RUN useradd foo
ADD start.sh /
RUN chmod +x /start.sh
ENTRYPOINT ["/start.sh"]
ENV FOO docker
