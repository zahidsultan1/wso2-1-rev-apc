FROM internal-registry.tkg.mobilink.net.pk/wso2-prod/wso2/wso2am:4.1

COPY secret-conf.properties ${WSO2_SERVER_HOME}/repository/conf/security/secret-conf.properties

USER root

RUN chown -R 802:802 /home/wso2carbon/wso2am-4.1.0/repository/resources/security/
RUN chmod -R 777 /home/wso2carbon/wso2am-4.1.0/repository/resources/security/

RUN rm -f /etc/localtime
RUN ln -sv /usr/share/zoneinfo/Asia/Karachi /etc/localtime
RUN echo "Asia/Karachi" > /etc/timezone

COPY *.jar /home/wso2carbon/wso2am-4.1.0/repository/components/lib/
RUN chown 802:802 /home/wso2carbon/wso2am-4.1.0/repository/components/lib/*.jar

COPY deployment-final.toml /home/wso2carbon/wso2am-4.1.0/repository/conf/deployment.toml
RUN chown 802:802 /home/wso2carbon/wso2am-4.1.0/repository/conf/deployment.toml

RUN chown wso2carbon:wso2 ${WSO2_SERVER_HOME}/repository/conf/security/secret-conf.properties

RUN /home/wso2carbon/wso2am-4.1.0/bin/profileSetup.sh -Dprofile=gateway-worker --skipConfigOptimization

