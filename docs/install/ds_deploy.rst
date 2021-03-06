.. _ds-infr-label:

###################
Data Security (DS)
###################

In-depth descriptions on how to set up a service tenant and an infrastructure tenant are available. These include step-by-step instructions to deploy the enforcement infrastructure on existing Java application servers. In addition, a streamlined, deployment-script-based setup as well as an automated, easy-to-use, self-contained, two-step, docker-based setup is provided for jump-starting a SUNFISH deployment.

The referred scripts and configuration files are located at ``https://github.com/sunfish-prj/Data-Security/tree/master/ds/doc/install``. The sub-folder ``service`` contains necessary files for the service tenant deployment, the ``infrastructure`` folder for the infrastructure tenant deployment respectively.
The ``docker`` folder again contains the same structure, but for the dockerized setup.

Setting-Up a Service Tenant
===========================
It is assumed that a service is already running in the service tenant.

Step-By-Step Setup
------------------
Although not recommended, the SUNFISH data security enforcement infrastructure can be deployed following the succeeding steps. However, depending on the deployment use case, additional steps or adaptions to either configuration or system components may still be necessary. For demonstration purposes a two tenant setup is assumed.
The sample configuration ships with precompiled Tomcat applications, which can be found in the respective ``webapps`` directory of either tenant. Additionally, a sample configuration for the service tenant can be found in the respective ``conf`` directory.
To deploy the service tenant follow these steps:

  * Copy the content of the provided ``./tomcat/webapps`` directory to ``CATALINA_HOME/webapps`` directory
  * Copy the content of the provided ``./tomcat/conf`` directory to ``CATALINA_HOME/conf`` directory
  * Copy the content of the provided ``./proxy/`` directory to any desired directory (referred to as ``PROXY_HOME``)


In a divergent deployment scenario, the respective configurations of the SUNFISH components and the SUNFISH proxy need to be adapted individually. To start the SUNFISH data security enforcement infrastructure simply start your local Tomcat instance and execute the ``start.sh`` script, located in your ``PROXY_HOME`` directory.

Using the Deployment Script
---------------------------
The attached deployment script is an easy way to automatically setup a service tenant. For this, the following two steps are necessary:

 * Adapt the configuration if necessary (``config.sh``)
 * Execute the deployment script (``./deploy.sh``)

The deployment script will automatically create all necessary resources and copy them to their designated destination. No further steps are necessary. To start the SUNFISH data security enforcement infrastructure simply start your local Tomcat instance and execute the ``start.sh`` script, located in your ``PROXY_HOME`` directory.


Configuration Directives
^^^^^^^^^^^^^^^^^^^^^^^^
The infrastructure tenant features several configuration options before installation. The following parameters are available:

 * ``TOMCAT_PORT``: Defines the port of the local Tomcat instance
 * ``CATALINA_HOME``: Defines the home directory of the local Tomcat instance (e.g. (``/usr/local/tomcat/``)
 * ``PEP_URL_PDP``: Defines the URL of the designated *PDP* for the *PEP*
 * ``PEP_URLS_PIPS``: Defines the possible *PIPs* available to the *PEP*. Multiple URLs can be specified, separated by a comma
 * ``PEP_ZONE``: Defines the tenant name the *PEP* is located in
 * ``PEP_URL_DM``: Specifies the URL to the data masking service
 * ``PEP_URL_ANON``: Sepcifies the URL to the anonymisation service
 * ``PIP_DATABASE``: Defines possible database values for the *PIP*. Each setting consists of a key and a value. In general three entries are necessary in order to setup a new service inside the service tenant:

   * **Host for ID**:  Assign a hostname to a specific service. The key must be in the format ``host.<service_id>``. The value represents a single URL to the designated service.
   * **Tenant for ID**: Assign a service to a specific tenant. The key must be in the format ``zone.<service_id>``. The value defines the tenant the service is located at.
   * **PEP for Tenant**: Assign a *PEP* to a specific tenant. The key must be in the format ``pep.<tenant name>``. The value represents a single URL to the designated *PEP*.
   
   In addition, a special PIP with SLI interface can be used. This pip is automatically deployed on the infrastructure tenant as /pip-sli. and retrieves host, tenant, and PEP for a service fromthe SLI.
   It requires ``SLI`` to be set to the endpoint of the SLI to use.
   To use it, set ``USE_PIP_SLI`` in the service's config.sh and add the endpoint of this PIP (running on the infrastructure tenant) to ``PEP_URLS_PIPS``.

 * ``PROXY_HOME``: Defines the home directory of the SUNFISH proxy (e.g. (``/usr/local/proxy/``)
 * ``PROXY_IP``: Defines the IP address the SUNFISH Proxy will run on
 * ``PROXY_PORT``: Defines the port the SUNISH Proxy will listen to
 * ``PROXY_PEP[<service_id>]``: Defines the URL of the *PEP* guarding the service *<service_id>* for the SUNFISH Proxy. Multiple services can be defined; should match the service IDs in the PIP database. The proxy interprets the first part of any path as service_id and strips it from the request forwarded to the PEP declared for <service_id>.



Dockerised Setup
----------------
The docker-based deployment also features a configuration file containing essentially the same (at this point mostly self-explanatory) directives and a deployment script. This script has to be invoked after editing the configuration file just as it is the case for the regular deployment-script-based setup.

To actually deploy the docker container, once the configuration file has been adapted, the following steps need to be performed:

 * Download and etract the 'Docker Latest' release from the `Releases` tab in the GitHub repository.
 * The preconfigured docker containers *demotenant.tar* and  *infra.tar* need to be loaded: ``docker load -i demotenant.tar`` ``docker load -i infra.tar``
 * The deployment script of the infrastructure has to be executed (``./deploy.sh``) inside the infrastructure directory
 * The IP address of the docker container will be printed out. Copy it and press Enter.
 * This address needs to be configured as infrastructure tenant IP address for the demotenant cotnainer in the file tenant/config.sh
 * Execute the demo tenant by invoking deploy.sh

This should start a docker container, inside which the proxy is running on ``PROXY_PORT`` and the PEG and the PIP are running as web applications on a Tomcat server in the same container on ``TOMCAT_PORT``. Both ports are mapped to their respective counterparts on the host machine.

The demo tenant includes a demo application, but no DS policies. A postman collection is also part of the 'Docker Latest' release. It contains a sample policy allowing access to /demo-app/demo/ds/index.* and denying everything else. It should work out-of-the-box for the default setup.
The default configuration of the proxy assigns the demo application the identifier 'demo'. Consequently, the demo application (and the sample DS policy) can be tested, by invoking ``http://localhost:10000/demo/demo-app/demo/ds/index.html``. The DS component can also by bypassed for debugging purposes by connecting to ``http://localhost:8081/demo-app/demo/ds/index.html``.


Setting-Up a Service
--------------------
To add a new service to the SUNFISH data security enforcement infrastructure, the following steps are necessary:


* Add a `host` for the `service id` to the configuration file ``config.sh`` or, if the SUNFISH tenant has already been setup, to the configuration file located in ``CATALINA_HOME/conf/sunfish/pip/database/pip_database.config`` 
* Add a `tenant` for the `service id` to the configuration file ``config.sh`` or, if the SUNFISH tenant has already been setup, to the configuration file located in ``CATALINA_HOME/conf/sunfish/pip/database/pip_database.config``. It is important to note that this step needs to be performed for all operational tenants, as long as the PIP database containing the service configuration is not replicated between all tenants.
* Add a `pep` for the `tenant` of the `service` to the configuration file ``config.sh`` or, if the SUNFISH tenant has already been setup, to the configuration file located in ``CATALINA_HOME/conf/sunfish/pip/database/pip_database.config``. It is important to note that this step needs to be performed for all operational tenants, as long as the PIP database containing the service configuration is not replicated between all tenants.
* Restart your local Service Tenant Tomcat in order to apply the changes

Adding Policies
--------------------
By default, any deployed service requires dedicated policies in order for the SUNFISH data security enforcement infrastructure to work. Policies can be added via the *PAP* and the defined **API** (see also Chapter `SUNFISH Policy Administration Point (PAP) API`). A sample policy, allowing access to a defined service is shown below:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
    <Policy xmlns="urn:oasis:names:tc:xacml:3.0:core:schema:wd-17" xmlns:ns2="urn:sunfish" PolicyId="urn:sunfish:policy:demo-proxy-https" Version="1.0" RuleCombiningAlgId="urn:oasis:names:tc:xacml:1.0:rule-combining-algorithm:deny-overrides">
        <Description>Demo Permit-All Policy </Description>
        <Target>
            <AnyOf>
                <AllOf>
                    <Match MatchId="urn:oasis:names:tc:xacml:1.0:function:string-equal">
                        <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">129.27.142.49</AttributeValue>
                        <AttributeDesignator Category="urn:sunfish:attribute-category:service" AttributeId="urn:sunfish:attribute:id" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="true"/>
                    </Match>
                    <Match MatchId="urn:oasis:names:tc:xacml:3.0:function:string-starts-with">
                        <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">/demo-app/demo/</AttributeValue>
                        <AttributeDesignator Category="urn:sunfish:attribute-category:response" AttributeId="urn:sunfish:attribute:request:path" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false"/>
                    </Match>
                </AllOf>
                <AllOf>
                    <Match MatchId="urn:oasis:names:tc:xacml:1.0:function:string-equal">
                        <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">129.27.142.49</AttributeValue>
                        <AttributeDesignator Category="urn:sunfish:attribute-category:service" AttributeId="urn:sunfish:attribute:id" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="true"/>
                    </Match>
                    <Match MatchId="urn:oasis:names:tc:xacml:3.0:function:string-starts-with">
                        <AttributeValue DataType="http://www.w3.org/2001/XMLSchema#string">/demo-app/demo/</AttributeValue>
                        <AttributeDesignator Category="urn:sunfish:attribute-category:request" AttributeId="urn:sunfish:attribute:request:path" DataType="http://www.w3.org/2001/XMLSchema#string" MustBePresent="false"/>
                    </Match>
                </AllOf>
            </AnyOf>
        </Target>
        <Rule RuleId="urn:sunfish:rule:permit" Effect="Permit">
            <Target/>
        </Rule>
    </Policy>



Setting-Up an Infrastructure Tenant
===================================

Step-By-Step Setup
------------------
Although not recommended, the SUNFISH data security enforcement infrastructure can be deployed following the succeeding steps. However, depending on the deployment use case, additional steps or adaptions to either configuration or system components may still be necessary. For demonstration purposes a two tenant setup is assumed.
The sample configuration ships with precompiled Tomcat applications, which can be found in the respective ``webapps`` directory of either tenant. Additionally, a sample configuration for the infrastructure tenant can be found in the respective ``conf`` directory.
To deploy the service tenant follow these steps:

  * Copy the content of the provided ``webapps`` directory to ``CATALINA_HOME/webapps`` directory
  * Copy the content of the provided ``conf`` directory to ``CATALINA_HOME/conf`` directory

In a divergent deployment scenario, the respective configurations of the SUNFISH components need to be adapted individually. To start the SUNFISH data security enforcement infrastructure simply start your local Tomcat instance.


Using the Deployment Script
---------------------------
The attached deployment script is an easy way to automatically setup an infrastructure tenant. For this, the following two steps are necessary:

 * Adapt the configuration if necessary (``config.sh``)
 * Execute the deployment script (``./deploy.sh``)

The deployment script will automatically create all necessary resources and copy them to their designated destination. No further steps are necessary. To start the SUNFISH data security enforcement infrastructure simply start your local Tomcat instance.


Configuration Directives
^^^^^^^^^^^^^^^^^^^^^^^^
The infrastructure tenant features several configuration options before installation. The following parameters are available:

 * ``TOMCAT_PORT``: Defines the port of the local Tomcat instance
 * ``CATALINA_HOME``: Defines the home directory of the local Tomcat instance (e.g. (``/usr/local/tomcat/``)
 * ``PAP_URL_RI``: Defines the URL of the designated **Registry Interface** for the *PAP*
 * ``PDP_URLS_PRPS``: Defines the possible *PRPs* available to the *PDP*. Multiple URLs can be specified, separated by a comma
 * ``PDP_URLS_PIPS``: Defines the possible *PIPs* available to the *PDP*. Multiple URLs can be specified, separated by a comma
 * ``PRP_URL_RI``: Defines the URL of the designated **Registry Interface** for the *PRP*
 * ``PIP_DATABASE``: Defines possible database values for the *PIP*. Each setting consists of a key and a value. In general, no additional values are necessary for the *PIP* in the infrastructure tenant.


Dockerised Setup
----------------
The docker-based deployment also features a configuration file containing essentially the same (at this point mostly self-explanatory) directives and a deployment script. This script has to be invoked after editing the configuration file just as it is the case for the regular deployment-script-based setup.

To actually deploy the docker container, once the configuration file has been adapted, the following steps need to be performed:

 * Download the infrastructure docker container (``infrastructure.tar``) from the `Releases` tab in the GitHub repository and copy it to ``install/docker/infrastructure/``
 * The preconfigured docker container *infrastructure.tar* needs to be loaded: ``docker load -i infrastructure.tar``
 * The deployment script has to be executed (``./deploy.sh``)

This should start a docker container, inside which the PDP, the PRP and the PIP are running as web applications on a Tomcat server on ``TOMCAT_PORT`` which is mapped to the same port on the host machine.


