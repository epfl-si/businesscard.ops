FROM {{ cadi_base_image_uri }}
LABEL maintainer "isas-fsd@groupes.epfl.ch"

# Install perl dependencies needed for businesscard
# RUN cpanm utf8::all HTML::Template JSON::SL XML::LibXML Net::CIDR LWP::Protocol::https

# Enable apache modules. `headers` and `env` modules are enabled by default.
# Make apache listen on port 8080
RUN sed -i \
    -e 's/^#\(LoadModule .*mod_rewrite.so\)/\1/' \
    -e 's/^\s#\(LoadModule .*mod_cgid\?.so\)/\1/' \
    -e 's/^#\(LoadModule .*mod_remoteip.so\)/\1/' \
    -e 's/^Listen 80$/Listen 8080/' \
    /usr/local/apache2/conf/httpd.conf

RUN cpanm --notest \
          Mail::Sendmail \
          MIME::Words

ENV TEQUILA_HOST=test-tequila.epfl.ch
ENV SITE_URL=https://businesscard-preprod.epfl.ch/

RUN echo "PassEnv TEQUILA_HOST" >> /usr/local/apache2/conf/httpd.conf
RUN echo "PassEnv SITE_URL" >> /usr/local/apache2/conf/httpd.conf


# Create the vhosts directory
RUN mkdir -p /var/www/vhosts/


# Copy the SSH private key for the idevfsd-checkouter (c4science),                       in order
# to be able to git clone a private repo in /var/www/vhosts/businesscard.epfl.ch.
# Please read https://cloud.redhat.com/blog/how-to-squash-openshift-docker-build-layers-to-remove-secrets
# on secrets removal.
COPY .ssh /root/.ssh
WORKDIR /var/www/vhosts/
RUN git clone --branch saphirevert/test ssh://git@c4science.ch/diffusion/9003/businesscard.git {{ businesscard_vhost_dir }}
RUN cd {{ businesscard_vhost_dir }} && git rev-parse --abbrev-ref HEAD && cat .git/HEAD


# Include the businesscard apache's configuration, e.g. make everything being
# served from the dinfo standard '/var/www/vhosts/businesscard.epfl.ch'.

RUN echo "Include conf/25-businesscard.epfl.ch.conf" \
        >> /usr/local/apache2/conf/httpd.conf

# RUN echo {{ businesscard_vhost_dir }}
# RUN cp /var/www/vhosts/{{ businesscard_vhost_dir }}/conf/docker/25-businesscard.epfl.ch.conf /usr/local/apache2/conf/docker/25-businesscard.epfl.ch.conf
# RUN touch /usr/local/apache2/conf/25-businesscard.epfl.ch.conf

# COPY /keybase/team/epfl_bsnscrd/dbs.conf /opt/dinfo/etc/dbs.conf

# Ensure that the logs directory exists and is writable by apache
RUN mkdir -p /usr/local/apache2/logs && \
    chmod -R 777 /usr/local/apache2/logs
RUN mkdir -p /var/www/vhosts/{{ businesscard_vhost_dir }}/logs && \
    chmod 777 -R /var/www/vhosts/{{ businesscard_vhost_dir }}/logs
RUN chown www-data:www-data -R /var/www/vhosts/{{ businesscard_vhost_dir }}/htdocs
RUN chown www-data:www-data -R /var/www/vhosts/{{ businesscard_vhost_dir }}/cgi-bin
RUN chown www-data:www-data -R /var/www/vhosts/{{ businesscard_vhost_dir }}/private
RUN chown www-data:www-data -R /usr/local/apache2/logs
# Tequila's sessions directory
RUN mkdir -p /var/www/vhosts/{{ businesscard_vhost_dir }}/Tequila/Sessions && \
    chmod 777 -R /var/www/vhosts/{{ businesscard_vhost_dir }}/Tequila/Sessions

WORKDIR /var/www/vhosts/{{ businesscard_vhost_dir }}

# Temp version marked
RUN touch $(date "+%Y%m%d-%H%M%S")_deployed

{# Image gets squashed; see ../tasks/businesscard-openshift-build-tasks.yml #}
RUN rm -rf /root/.ssh


RUN mkdir -p /var/www/Tequila/Sessions
RUN chmod 777 -R /var/www/Tequila/Sessions


EXPOSE 8080
