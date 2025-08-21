# Declare the build argument
ARG BASE_IMAGE

# Use the build argument in the FROM instruction
FROM ${BASE_IMAGE}
# FROM ghcr.io/zahidsultan1/wso2mi-staging/wso2mi:4.4.0


COPY extras/CACert_Prod.crt ${WSO2_SERVER_HOME}/repository/resources/security/
COPY extras/client-truststore.jks ${WSO2_SERVER_HOME}/repository/resources/security/
COPY extras/axis2.xml ${WSO2_SERVER_HOME}/conf/axis2/
COPY deployment.toml ${WSO2_SERVER_HOME}/conf/
COPY extras/micro-integrator.sh ${WSO2_SERVER_HOME}/bin/

COPY jar/*.jar ${WSO2_SERVER_HOME}/lib/

RUN mkdir -p ${WSO2_SERVER_HOME}/registry/config/custom/apc-generic-v3
RUN mkdir -p ${WSO2_SERVER_HOME}/registry/config/custom/channelapiv1
RUN mkdir -p ${WSO2_SERVER_HOME}/registry/config/custom/channelapiv2
RUN mkdir -p ${WSO2_SERVER_HOME}/registry/config/custom/channelapiv3
RUN mkdir -p ${WSO2_SERVER_HOME}/registry/config/custom/hbl-easypaisa
RUN mkdir -p ${WSO2_SERVER_HOME}/registry/config/custom/jazzcashwallet
 
COPY registry/apc-generic-v3/ ${WSO2_SERVER_HOME}/registry/config/custom/apc-generic-v3
COPY registry/channelapiv1/ ${WSO2_SERVER_HOME}/registry/config/custom/channelapiv1
COPY registry/channelapiv2/ ${WSO2_SERVER_HOME}/registry/config/custom/channelapiv2 
COPY registry/channelapiv3/ ${WSO2_SERVER_HOME}/registry/config/custom/channelapiv3
COPY registry/hbl-easypaisa/ ${WSO2_SERVER_HOME}/registry/config/custom/hbl-easypaisa
COPY registry/apc-posting/ ${WSO2_SERVER_HOME}/registry/config/custom/apc-posting
COPY registry/jazzcashwallet/ ${WSO2_SERVER_HOME}/registry/config/custom/jazzcashwallet

USER root
RUN rm -f /etc/localtime
RUN ln -sv /usr/share/zoneinfo/Asia/Karachi /etc/localtime
RUN echo "Asia/Karachi" > /etc/timezone
RUN chown 802:802 ${WSO2_SERVER_HOME}/conf/axis2/axis2.xml
RUN chmod 777 ${WSO2_SERVER_HOME}/conf/axis2/axis2.xml
RUN chown 802:802 ${WSO2_SERVER_HOME}/conf/deployment.toml
RUN chmod 644 ${WSO2_SERVER_HOME}/conf/deployment.toml
RUN chown 802:802 ${WSO2_SERVER_HOME}/bin/micro-integrator.sh
RUN chmod 755 ${WSO2_SERVER_HOME}/bin/micro-integrator.sh
#RUN chmod 644 /etc/sysctl.conf
#RUN chmod 644 /etc/security/limits.conf
RUN chown -R 802:802 ${WSO2_SERVER_HOME}/lib/
RUN chmod -R 777 ${WSO2_SERVER_HOME}/lib/
RUN chown -R 802:802 ${WSO2_SERVER_HOME}/dropins/
RUN chmod -R 777 ${WSO2_SERVER_HOME}/dropins/
RUN chown -R 802:802 ${WSO2_SERVER_HOME}/registry/
RUN chmod -R 777 ${WSO2_SERVER_HOME}/registry/
RUN chown 802:802 ${WSO2_SERVER_HOME}/conf/log4j2.properties
RUN chmod 644 ${WSO2_SERVER_HOME}/conf/log4j2.properties

COPY carbonapps/* /tmp/
RUN cp /tmp/*.car ${WSO2_SERVER_HOME}/repository/deployment/server/carbonapps/

COPY mi-dashboard.conf /tmp/mi-dashboard.conf
RUN cat /tmp/mi-dashboard.conf >> ${WSO2_SERVER_HOME}/conf/deployment.toml

