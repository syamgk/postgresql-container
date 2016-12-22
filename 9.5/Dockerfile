FROM registry.centos.org/centos/centos

# PostgreSQL image for OpenShift.
# Volumes:
#  * /var/lib/psql/data   - Database cluster for PostgreSQL
# Environment:
#  * $POSTGRESQL_USER     - Database user name
#  * $POSTGRESQL_PASSWORD - User's password
#  * $POSTGRESQL_DATABASE - Name of the database to create
#  * $POSTGRESQL_ADMIN_PASSWORD (Optional) - Password for the 'postgres'
#                           PostgreSQL administrative account

MAINTAINER sgk <sgk@redhat.com>

ENV HOME=/var/lib/pgsql \
    PGUSER=postgres

LABEL io.k8s.description="PostgreSQL is an advanced Object-Relational database management system" \
      io.k8s.display-name="PostgreSQL 9.5" \
      io.openshift.expose-services="5432:postgresql" \
      io.openshift.tags="database,postgresql,postgresql95,rh-postgresql95"

EXPOSE 5432

ADD root /


ENV PG_REPO "https://download.postgresql.org/pub/repos/yum"
ENV PG_MAJOR "9.5"
ENV PG_MAJOR_NODOT "95"
ENV RHEL_VERSION "7.3"
ENV REPO_RHEL_ARCH "x86_64"

ENV PG_REPOURL="${PG_REPO}/${PG_MAJOR}/redhat/rhel-${RHEL_VERSION}-${REPO_RHEL_ARCH}"

RUN rpm --import ${PG_REPO}/RPM-GPG-KEY-PGDG && rpm --import ${PG_REPO}/RPM-GPG-KEY-PGDG-${PG_MAJOR_NODOT}

RUN echo -e "[postgres_repo]\nname=Postgres Repo\nbaseurl=${PG_REPOURL}\nenabled=1\ngpgcheck=1" > /etc/yum.repos.d/postgres.repo

RUN yum install -y centos-release-scl-rh &&\
        INSTALL_PKGS="rsync tar gettext bind-utils nss_wrapper"  \
        && yum install -y pgdg-centos${PG_MAJOR_NODOT} \
        && yum install -y postgresql${PG_MAJOR_NODOT}-server \
        && yum install -y \
                postgresql${PG_MAJOR_NODOT} \
                postgresql${PG_MAJOR_NODOT}-contrib \
        && yum clean all &&\
        localedef -f UTF-8 -i en_US en_US.UTF-8 && \
        test "$(id postgres)" = "uid=26(postgres) gid=26(postgres) groups=26(postgres)" && \
        mkdir -p /var/lib/pgsql/data && \
        /usr/libexec/fix-permissions /var/lib/pgsql && \
        /usr/libexec/fix-permissions /var/run/postgresql &&\
        INSTALL_PKGS="rsync tar gettext bind-utils nss_wrapper" &&\
        yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
        rpm -V $INSTALL_PKGS && \
        yum clean all
# Get prefix path and path to scripts rather than hard-code them in scripts
ENV CONTAINER_SCRIPTS_PATH=/usr/share/container-scripts/postgresql

# When bash is started non-interactively, to run a shell script, for example it
# looks for this variable and source the content of this file. This will enable
# the SCL for all scripts without need to do 'scl enable'.
ENV BASH_ENV=${CONTAINER_SCRIPTS_PATH}/scl_enable \
    ENV=${CONTAINER_SCRIPTS_PATH}/scl_enable \
    PROMPT_COMMAND=". ${CONTAINER_SCRIPTS_PATH}/scl_enable" \
    PATH=/usr/pgsql-${PG_MAJOR}/bin:$PATH
VOLUME ["/var/lib/pgsql/data"]

USER 26
ENTRYPOINT ["container-entrypoint"]
CMD ["run-postgresql"]

