# Dockerfile for building Ansible image from source for Debian 8 (jessie), with as few additional software as possible.
#
# @see http://docs.ansible.com/ansible/intro_installation.html#running-from-source
#
# Version  1.0
#


# pull base image
FROM debian:jessie

MAINTAINER William Yeh <william.pjyeh@gmail.com>


RUN echo "===> Adding Ansible's prerequisites..."         && \
    apt-get update -y  &&  apt-get install --fix-missing  && \
    DEBIAN_FRONTEND=noninteractive  \
        apt-get install --no-install-recommends -y -q  \
                build-essential ca-certificates        \
                python python-pip python-dev           \
                libffi-dev libssl-dev                  \
                libxml2-dev libxslt1-dev zlib1g-dev    \
                git sudo curl rsync                 && \
    apt-get -y --purge remove python-cffi           && \
    \
    \
    echo "===> Fix strange bug under Jessie: cannot import name IncompleteRead"  && \
    easy_install -U pip    && \
    \
    pip install --upgrade cffi pywinrm              && \
    pip install --upgrade pyyaml jinja2 pycrypto    && \
    \
    \
    echo "===> Downloading Ansible's source tree..."            && \
    git clone git://github.com/ansible/ansible.git --recursive  && \
    \
    \
    echo "===> Compiling Ansible..."      && \
    cd ansible                            && \
    bash -c 'source ./hacking/env-setup'  && \
    \
    \
    echo "===> Moving useful Ansible stuff to /opt/ansible ..."  && \
    mkdir -p /opt/ansible                && \
    mv /ansible/bin   /opt/ansible/bin   && \
    mv /ansible/lib   /opt/ansible/lib   && \
    mv /ansible/docs  /opt/ansible/docs  && \
    rm -rf /ansible                      && \
    \
    \
    echo "===> Clean up..."                                                  && \
    apt-get remove -y --auto-remove \
            build-essential python-pip python-dev git libffi-dev libssl-dev  && \
    apt-get clean                                                            && \
    rm -rf /var/lib/apt/lists/*                                              && \
    \
    \
    echo "===> Adding hosts for convenience..."  && \
    mkdir -p /etc/ansible                        && \
    echo 'localhost' > /etc/ansible/hosts


ENV PATH        /opt/ansible/bin:$PATH
ENV PYTHONPATH  /opt/ansible/lib:$PYTHONPATH
ENV MANPATH     /opt/ansible/docs/man:$MANPATH


COPY ansible-playbook-wrapper /usr/local/bin/

ONBUILD  RUN  DEBIAN_FRONTEND=noninteractive  apt-get update   && \
              echo "===> Updating TLS certificates..."         && \
              apt-get install -y openssl ca-certificates

ONBUILD  WORKDIR  /tmp
ONBUILD  COPY  .  /tmp
ONBUILD  RUN  \
              echo "===> Diagnosis: host information..."  && \
              ansible -c local -m setup all



# default command: display Ansible version
CMD [ "ansible-playbook", "--version" ]
