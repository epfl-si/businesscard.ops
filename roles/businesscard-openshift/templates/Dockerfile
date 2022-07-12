FROM {{ cadi_base_image_uri }}

USER root
RUN apt update
# RUN apt-get install -y locate
# RUN locate .ssh
RUN ls -al
RUN chown -R dinfo:dinfo /usr/local/apache2
# Create the vhosts directory
RUN mkdir -p /var/www/vhosts/
RUN chown -R dinfo:dinfo /var/www/vhosts
# USER dinfo


# RUN ls -al /home/dinfo/

# Enable apache modules. `headers` and `env` modules are enabled by default.
# Make apache listen on port 8080
RUN sed -i \
    -e 's/^#\(LoadModule .*mod_rewrite.so\)/\1/' \
    -e 's/^\s#\(LoadModule .*mod_cgid\?.so\)/\1/' \
    -e 's/^#\(LoadModule .*mod_remoteip.so\)/\1/' \
    -e 's/^Listen 80$/Listen 8080/' \
    /usr/local/apache2/conf/httpd.conf

# Include the businesscard apache's configuration, e.g. make everything being
# served from the dinfo standard '/var/www/vhosts/businesscard.epfl.ch'.
RUN echo "Include conf/25-businesscard.epfl.ch.conf" \
        >> /usr/local/apache2/conf/httpd.conf


# Copy the SSH private key for the idevfsd-checkouter (c4science), in order
# to be able to git clone a private repo in /var/www/vhosts/businesscard.epfl.ch.
# Please read https://cloud.redhat.com/blog/how-to-squash-openshift-docker-build-layers-to-remove-secrets
# on secrets removal.
COPY .ssh /root/.ssh
WORKDIR /var/www/vhosts/
RUN git clone --branch {{ businesscard_git_branch | default('master') }} ssh://git@c4science.ch/diffusion/9003/businesscard.git {{ businesscard_vhost_dir }}
RUN cd {{ businesscard_vhost_dir }} && git rev-parse --abbrev-ref HEAD && cat .git/HEAD

# Ensure that the logs directory exists and is writable by apache
RUN mkdir -p /usr/local/apache2/logs && \
    chmod 777 -R /usr/local/apache2/logs
RUN mkdir -p /var/www/vhosts/{{ businesscard_vhost_dir }}/logs && \
    chmod 777 -R /var/www/vhosts/{{ businesscard_vhost_dir }}/logs
RUN chown www-data:www-data -R /var/www/vhosts/{{ businesscard_vhost_dir }}/htdocs
RUN chown www-data:www-data -R /var/www/vhosts/{{ businesscard_vhost_dir }}/cgi-bin
RUN chown www-data:www-data -R /var/www/vhosts/{{ businesscard_vhost_dir }}/private
# Tequila's sessions directory
RUN mkdir -p /var/www/vhosts/{{ businesscard_vhost_dir }}/Tequila/Sessions && \
    chmod 777 -R /var/www/vhosts/{{ businesscard_vhost_dir }}/Tequila/Sessions

WORKDIR /var/www/vhosts/{{ businesscard_vhost_dir }}

# Temp version marked
RUN touch $(date "+%Y%m%d-%H%M%S")_deployed

{# Image gets squashed; see ../tasks/businesscard-openshift-build-tasks.yml #}
RUN rm -rf /root/.ssh
