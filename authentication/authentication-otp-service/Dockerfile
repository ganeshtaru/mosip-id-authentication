FROM openjdk:11

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG spring_config_label

# can be passed during Docker build as build time environment for spring profiles active 
ARG active_profile

# can be passed during Docker build as build time environment for config server URL 
ARG spring_config_url

# can be passed during Docker build as build time environment for hsm client zip file path
#ARG client_zip_path
ARG hsm_client_zip_path

# can be passed during Docker build as build time environment for glowroot 
ARG is_glowroot

# can be passed during Docker build as build time environment for artifactory URL
ARG artifactory_url

# environment variable to pass active profile such as DEV, QA etc at docker runtime
ENV active_profile_env=${active_profile}

# environment variable to pass github branch to pickup configuration from, at docker runtime
ENV spring_config_label_env=${spring_config_label}

# environment variable to pass spring configuration url, at docker runtime
ENV spring_config_url_env=${spring_config_url}

# environment variable to pass glowroot, at docker runtime
ENV is_glowroot_env=${is_glowroot}

# environment variable to pass artifactory url, at docker runtime
ENV artifactory_url_env=${artifactory_url}

# environment variable to pass hsm client zip file path, at docker runtime
#ENV zip_file_path=${client_zip_path}
ENV hsm_zip_file_path=${hsm_client_zip_path}

#ENV work_dir_env=/

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user=mosip

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user_group=mosip

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user_uid=1001

# can be passed during Docker build as build time environment for github branch to pickup configuration from.
ARG container_user_gid=1001

ARG hsm_local_dir=hsm-client

ENV hsm_local_dir_name=${hsm_local_dir}

# install packages and create user
RUN apt-get -y update \
&& apt-get install -y unzip sudo \
&& groupadd -g ${container_user_gid} ${container_user_group} \
&& useradd -u ${container_user_uid} -g ${container_user_group} -s /bin/sh -m ${container_user} \
&& adduser ${container_user} sudo \
&& echo "%sudo ALL=(ALL) NOPASSWD:/home/${container_user}/${hsm_local_dir}/install.sh" >> /etc/sudoers

# set working directory for the user
WORKDIR /home/${container_user}

ENV work_dir=/home/${container_user}

ARG loader_path=${work_dir}/additional_jars

RUN mkdir -p ${loader_path}

ENV loader_path_env=${loader_path}

ENV current_module_env=authentication-otp-service

ADD configure_start.sh configure_start.sh

RUN chmod +x configure_start.sh

ADD target/${current_module_env}-*.jar ${current_module_env}.jar

EXPOSE 8092

EXPOSE 9010

# change permissions of file inside working dir
RUN chown -R ${container_user}:${container_user} /home/${container_user}

# select container user for all tasks
USER ${container_user_uid}:${container_user_gid}

ENTRYPOINT [ "./configure_start.sh" ]

CMD if [ "$is_glowroot_env" = "present" ]; then \
    wget -q --show-progress "${artifactory_url_env}"/artifactory/libs-release-local/io/mosip/testing/glowroot.zip ; \
    unzip glowroot.zip ; \
    rm -rf glowroot.zip ; \
    sed -i "s/<service_name>/${current_module_env}/g" glowroot/glowroot.properties ; \
    java -jar -Djava.security.debug=sunpkcs11 -javaagent:glowroot/glowroot.jar -Dspring.cloud.config.label="${spring_config_label_env}" -Dspring.profiles.active="${active_profile_env}" -Dspring.cloud.config.uri="${spring_config_url_env}" ${current_module_env}.jar ; \
    else \
    java -jar -Djava.security.debug=sunpkcs11 -Dspring.cloud.config.label="${spring_config_label_env}" -Dspring.profiles.active="${active_profile_env}" -Dspring.cloud.config.uri="${spring_config_url_env}" ${current_module_env}.jar ; \
    fi

#Sample docker run command:
# sudo docker run --rm -it -e artifactory_url_env="http://artifactory" -e spring_config_label_env="1.0.9" -e active_profile_env="dev" -e spring_config_url_env="http://config-server/config" -e PKCS11_PROXY_SOCKET=tcp://softhsm-ida:5666 -p 8092:8092 authentication-otp-service:1.0.9