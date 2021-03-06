######################################################
#
# Agave MyProxy Server
# Tag: agaveapi/myproxy-gateway
#
# This container provides a standard myproxy server running
# on port 7512 for the test users created in the agaveapi/centos-base
# image. It also provides a myproxy server configured with a
# trust relationship runing on port 7513. Both servers are fronted
# by a myproxy gateway instance.
#
# Usage:
# docker run -h docker.example.com -i     \
#            -p 443:443                   \ # Apache SSL proxy to MyProxy Gateway
#            -p 8080:8080                 \ # MyProxy Gateway
#            -p 7512:7512                 \ # MyProxy CA + PAM
#            -p 7513:7513                 \ # MyProxy CA with trust relationship
#            -p 10022:22                  \ # SSHD, SFTP
#            agaveapi/myproxy
#
# This container provides a standard myproxy server running
# on ports 7512 configured to issue certs via pam-based
# usernames and passwords
######################################################

FROM agaveapi/myproxy

MAINTAINER Rion Dooley <dooley@tacc.utexas.edu>

#mpg dependencies
RUN yum -y install java-1.7.0-openjdk-devel git httpd mod_ssl tar

# install Maven
WORKDIR /usr/share
RUN curl -O ftp://mirror.reverse.net/pub/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
RUN tar xzf apache-maven-3.0.5-bin.tar.gz
RUN ln -s /usr/share/apache-maven-3.0.5/bin/mvn /usr/bin/mvn
RUN rm apache-maven-3.0.5-bin.tar.gz

# install MPG
WORKDIR /
RUN mv /myproxy-gateway/hostcerts /hostcerts
RUN git clone https://bitbucket.org/taccaci/myproxy-gateway.git
RUN mv /hostcerts /myproxy-gateway/hostcerts
WORKDIR /myproxy-gateway
RUN git pull; git remote
ADD maven/settings.xml /root/.m2/settings.xml

# generate and sign a service cert
ENV export MPG_SUBJECT=$(grid-cert-info -file myproxy-gateway/hostcerts/mpgcert.pem -subject) && \
    sed -ri 's/MPG_SUBJECT/$MPG_SUBJECT</g' /root/.m2/settings.xml && \
    cat /root/.m2/settings.xml

# build server
WORKDIR /myproxy-gateway
RUN mvn -Dmaven.test.skip=true clean install -P mpg
RUN mkdir mpg-api/target/logs

# Configure apache as a proxy with forced ssl
RUN mkdir -p /var/www/html/myproxy/v1
RUN ln -s /myproxy-gateway/mpg-docs/target/classes /var/www/html/myproxy/v1/docs
RUN chmod -R +rx /var/www/html/myproxy
ADD apache/ssl.conf /etc/httpd/conf.d/ssl.conf
ADD apache/htaccess /var/www/html/.htaccess

ADD supervisord.conf /etc/supervisord.conf
RUN chmod -R 777 /var/log/supervisor
ADD docker_entrypoint.sh /docker_entrypoint.sh

ENV MYPROXY_SERVER_DN "/C=US/O=Texas Advanced Computing Center/OU=Dev Certificate Authorities/OU=dockerCA/CN=host/docker.example.com"

ENTRYPOINT [ "/docker_entrypoint.sh" ]
EXPOSE 10389 22 7512 7513 8080 8090 443 80
CMD ["/usr/bin/supervisord"]
