FROM eclipse-temurin:11.0.18_10-jre

ARG JMETER_VERSION

ENV JMETER_VERSION ${JMETER_VERSION:-5.4.3}
ENV JMETER_HOME /jmeter/apache-jmeter-$JMETER_VERSION/
ENV PATH $JMETER_HOME/bin:$PATH

# Install Dependencies
RUN apt-get update -y && apt-get -y --no-install-recommends install wget fontconfig

# Install Jmeter
RUN mkdir /jmeter
WORKDIR /jmeter
RUN wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-$JMETER_VERSION.tgz && \
    tar -xzf apache-jmeter-$JMETER_VERSION.tgz && \
    rm apache-jmeter-$JMETER_VERSION.tgz
RUN ln -s /usr/lib/libfontconfig.so.1 /usr/lib/libfontconfig.so && \
    ln -s /lib/libuuid.so.1 /usr/lib/libuuid.so.1 && \
    ln -s /lib/libc.musl-x86_64.so.1 /usr/lib/libc.musl-x86_64.so.1

# Install jmeter plugin manager
WORKDIR $JMETER_HOME
RUN wget -O "lib/cmdrunner-2.2.jar" https://repo1.maven.org/maven2/kg/apc/cmdrunner/2.2/cmdrunner-2.2.jar && \
    wget -O "lib/ext/jmeter-plugins-manager-1.6.jar" https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-manager/1.6/jmeter-plugins-manager-1.6.jar && \
    wget -O "lib/ext/jmeter-junit-xml-converter-0.0.1-SNAPSHOT-jar-with-dependencies.jar" --no-check-certificate https://docs.getxray.app/download/attachments/62262689/jmeter-junit-xml-converter-0.0.1-SNAPSHOT-jar-with-dependencies.jar && \
    wget -O "lib/ext/jmeter-plugins-standard-1.4.0.jar" https://repo1.maven.org/maven2/kg/apc/jmeter-plugins-standard/1.4.0/jmeter-plugins-standard-1.4.0.jar && \
    java -cp lib/ext/jmeter-plugins-manager-1.6.jar org.jmeterplugins.repository.PluginManagerCMDInstaller

# Install required plugins
RUN wget -O "lib/ext/jmeter-plugins-influxdb2-listener-2.6.jar" https://github.com/mderevyankoaqa/jmeter-influxdb2-listener-plugin/releases/download/v2.6/jmeter-plugins-influxdb2-listener-2.6.jar
COPY plugins.txt .
RUN bin/PluginsManagerCMD.sh install $(cat plugins.txt | grep ^[^#])
# Copy Jmeter entrypoint
COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh

EXPOSE 1099 50000 60000

ENTRYPOINT ["/entrypoint.sh"]