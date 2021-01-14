FROM justb4/jmeter:5.3

# We will need two useful plugins
# https://jmeter-plugins.org/wiki/TestPlanCheckTool/
# https://jmeter-plugins.org/wiki/ConcurrencyThreadGroup/
ENV PLAN_CHECK_PLUGIN_VERSION=2.4
ENV CUSTOM_THREAD_VERSION=2.8
RUN wget https://jmeter-plugins.org/files/packages/jpgc-plancheck-${PLAN_CHECK_PLUGIN_VERSION}.zip
RUN unzip -o jpgc-plancheck-${PLAN_CHECK_PLUGIN_VERSION}.zip -d ${JMETER_HOME}
RUN wget https://jmeter-plugins.org/files/packages/jpgc-casutg-${CUSTOM_THREAD_VERSION}.zip
RUN unzip -o jpgc-casutg-${CUSTOM_THREAD_VERSION}.zip -d ${JMETER_HOME}

# This added line in user.properties creates a new column in the final jtl report with response header elapsed-time
RUN   echo "sample_variables=elapsed-time" >> /opt/apache-jmeter-5.3/bin/user.properties

EXPOSE 1099

ENTRYPOINT ["/entrypoint.sh"]
