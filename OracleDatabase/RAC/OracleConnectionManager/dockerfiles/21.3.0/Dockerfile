# LICENSE UPL 1.0
#
# Copyright (c) 2022 Oracle and/or its affiliates.
#
# ORACLE DOCKERFILES PROJECT
# --------------------------
# This is the Dockerfile for Oracle Connection Manager
# 
# REQUIRED FILES TO BUILD THIS IMAGE
# ----------------------------------
# (1) LINUX.X64_213000_client.zip 
#     Oracle Database 21c Client (21.3) for Linux x86-64
#     Download Oracle Connection Manager as part of the Oracle Database Client
#     from https://www.oracle.com/technetwork/database/enterprise-edition/downloads/oracle21c-linux-210000-5022980.html 
#
# HOW TO BUILD THIS IMAGE
# -----------------------
# Put all downloaded files in the same directory as this Dockerfile
# Run: 
#      $ docker build -t oracle/cman:21.3.0 . 
#
# Pull base image
# ---------------
FROM oraclelinux:7-slim

# Environment variables required for this build (do NOT change)
# -------------------------------------------------------------
# Linux Env Variable
ENV SETUP_LINUX_FILE="setupLinuxEnv.sh" \
    INSTALL_DIR=/opt/scripts \
# Grid Env variables
    INVENTORY=/u01/app/oracle/oraInventory \
    DB_BASE=/u01/app/oracle \
    DB_HOME=/u01/app/oracle/product/cman \
    INSTALL_FILE_1="LINUX.X64_213000_client.zip" \
    DB_INSTALL_RSP="client21c_install.rsp" \
    DB_SETUP_FILE="setupDB.sh" \
    RUN_FILE="runOracle.sh" \
    CONFIG_CMAN_FILE="configCMAN.sh" \
    CHECK_SPACE_FILE="checkSpace.sh" \
    CMANORA="cman.ora" \
    FUNCTIONS="functions.sh" \
    INSTALL_DB_BINARIES_FILE="installDBBinaries.sh" \
    container="true" 
# Use second ENV so that variable get substituted
# hadolint ignore=DL3044
ENV  INSTALL_SCRIPTS=$INSTALL_DIR/install \
     PATH=/bin:/usr/bin:/sbin:/usr/sbin:$PATH \
     SCRIPT_DIR=$INSTALL_DIR/startup \
     DB_PATH=$DB_HOME/bin:$DB_HOME/OPatch/:/usr/sbin:$PATH \
     DB_LD_LIBRARY_PATH=$DB_HOME/lib:/usr/lib:/lib

# Copy binaries
# -------------
# Grid Binaries
COPY $INSTALL_FILE_1 $SETUP_LINUX_FILE $CHECK_SPACE_FILE $CLIENT_RSP $INSTALL_DB_BINARIES_FILE $INSTALL_DIR/install/

COPY $RUN_FILE $CMANORA $CONFIG_CMAN_FILE $FUNCTIONS $INSTALL_DIR/startup/

RUN chmod 755 $INSTALL_DIR/install/*.sh && \
    sync && \
    $INSTALL_DIR/install/$CHECK_SPACE_FILE && \
    $INSTALL_DIR/install/$SETUP_LINUX_FILE && \
    $INSTALL_DIR/install/$DB_SETUP_FILE  && \
    sed -e '/hard *memlock/s/^/#/g' -i /etc/security/limits.d/oracle-database-preinstall-21c.conf && \
    sed -e '/ *nofile /s/^/#/g' -i /etc/security/limits.d/oracle-database-preinstall-21c.conf && \
    su oracle -c "$INSTALL_DIR/install/$INSTALL_DB_BINARIES_FILE EE" && \
    $INVENTORY/orainstRoot.sh && \
    $DB_HOME/root.sh && \
    rm -f /etc/sysctl.d/99-oracle-database-preinstall-21c-sysctl.conf && \
    rm -f /etc/sysctl.d/99-sysctl.conf && \ 
    rm -f /etc/rc.d/init.d/oracle-database-preinstall-21c-firstboot && \
    chown -R oracle:oinstall $SCRIPT_DIR && \
    chmod 755 $SCRIPT_DIR/*.sh && \
    chmod 666 $SCRIPT_DIR/*.ora && \
    sync 

USER  oracle
WORKDIR /home/oracle
EXPOSE 1521 5500

# Define default command to start Oracle Database.
# hadolint ignore=DL3025
CMD exec $SCRIPT_DIR/$RUN_FILE
