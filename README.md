# Mule configuration properties to read properties
*Source: https://docs.oracle.com/cd/E13222_01/wls/docs81b/config_xml/overview.html*

The `config.xml` file is an XML document that describes the configuration of an entire WebLogic Server domain (is server provider of Java EE apps & an HTTP web server).

# Simple secure Configuration properties:

The purpose of this notes are to learn more about setting up the following configuration properties:
1. Creation of `.properties` or `.yaml` file per environment.
2. Creation of configuration properties global element.
3. Use of `${key}` read property.

## Creation of ``.properties` or `.yaml` file per environment.
- Usually the creation of a porperties file is at `./sample-config-prop/src/main/resources`, by default is known to the Mule context as the classpath for the project. Inside we create the following config file:
    - `dev.yaml` is the configuration file for the development environment. 
    - `test.yaml` is the configuration file for the testing environment.
    - `uat.yaml` is the configuration file for the user acceptance testing environment, which is a type of testing performed by the end user or the client to verify/accept the software system before moving the software application to the production environment.
    - `prod.yaml` is the configuration file for the production environment.
    ```yaml
    http:
        port: "8081"
        path: "read-prop"
    db: 
        host: "localhost"
        port: "3306"
        user: "testUser"
        password: "randomPassword"
    ```
    *The above `.yaml` code is an example of configuration elements that can be set inside the file.*

## Creation of configuration properties global element.

![](https://docs.mulesoft.com/mule-runtime/4.4/_images/mruntime-new-global-element.png)*window for global elements configuration*

- Configuring 1 environment, for example: *Development* set on the `./dev.yaml`:
    - At `./sample-config-prop/src/main/mule/sample-config-prod.xml` (where mule flows can be found), inside the tab of *global elements configuration* 
        - *Create -> Global Configuration Properties -> File: `./dev.yaml`*
        - Then when using (drag & drop) an HTTP Listener Mule event source connector to the flow:
            - when configuring it at *Basic Settings -> Connector Configuration -> Add new/Edit*, the *Port* should be set to: `${http.port}` 
            - *General -> Path:* `${http.path}`
            ```xml
            <http:listener doc:name="Listener" doc:id="040769e5-5b08-46f0-859d-f38f4effcfe8" config-ref="HTTP_Listener_config" path="${http.path}"/>
            ```
            *View from Configuration XML file for the HTTP Listener connector*
            ---
        - The same can be achieved for the Logger connector when you want to output any value from the `dev.yaml` configuration properties file, for example Logger can output the Database port: `3306`
        **Input:**
        ```xml
        <logger level="INFO" doc:name="Logger" doc:id="003d61c8-ff7b-4bb9-8090-84061742f681" message="${db.port}"/>
        ```
        **Output Mule console:**
        ```
        org.mule.runtime.core.internal.processor.LoggerMessageProcessor: 3306
        ```

## Dynamic configuration of key properties

![](https://dz2cdn1.dzone.com/storage/temp/9244849-adding-properties-from-run-configuration.png)*Declaration of Mule properties Arguments at Run configuration.*

- When having multiple environments, a more eficient solution than hard coding the confirgutation properties by `${key}` is setting them at contextual menu over the pproject file: *Run as -> Mule application (configure) -> Arguments ->*

    |Program Arguments|
    |---|
    |`-Denv=dev`| 
    *Apply -> Run*


- Then, at *Global configuration Elements -> Configuration Properties -> General:*

    |Settings|
    |-|
    |``${env}.yaml``|

    - This sets the `<configuration properties>` XML tag to:
    ```xml
    <configuration-properties doc:name="Configuration properties" doc:id="2e6ac94c-d8d2-4f62-a56a-56b28a648595" file="${env}.yaml" />
    ``` 
---

# Secure configuration properties
- This procedure help us to configure properties to deploy on cloudHub, encrypts sensible data like passwords.
- Steps:
    1. Create `.properties` or `.yaml` file per environment.
    2. Add secure configuration module to studio from exchange.
    3. Encrypt passwords:
        1. Anypoint Enterprise Security plugin to studio / Through command:
        ```
        Name as: Anypoint Enterprise Security
        Location as: http://security-update-site-1.4.s3.amazonaws.com
        ```
        2. Choose algorithm and 16 digit number or alphanumeric value.
    4. Create secure properties configuration global elements.
    5. Use `${secure::property}` to read a property.

## Creating `.properties` or `.yaml` file per environment
- Creating under ``./main-project-directory/src/main/resources` the following `.yaml` files for configuration:
    - `dev-secure.yaml` is intended for development environment.
    - `uat-secur.yaml` is intended for User Acceptance Testing environment.
    - `prod-secure.yaml` is intended for production environment.
    *For example: setting the configuration on hierarchy order inside the `dev-secure.yaml`:*
    ```yaml
    http:
        host: "localhost"
        path: "secure-properties"
        port: 8081
    db: 
        host: "localhost"
        port: "3306"
        user: "test"
        password: "newpassword"
    ```
## Adding secure configuration module to Anypoint Studio from Anypoint Platform Exchange

![](https://mulesy.com/wp-content/uploads/2020/03/Capture1.jpg)*View from Anypoint Studio Palette Module configuration when adding modules from Exchange*

- At the right of the Canvas intended for flow creation, on the *Palette* window:
    - *Add module -> Configuration  button -> Mule project -> Modules -> ➕ Add icon -> from Exchange -> Search bar 🔎:*
        - Search for: *Mule Secure Configuration Property Extension* from Mulesoft publisher.
        - *Add -> Finish -> Apply and Close*

## Encrypt passwords
### 1. Anypoint Enterprise Security plugin to studio / Through command
- To promt a properties editor when *right click* on the `dev-secure.yaml` *contextual menu -> Open With -> Mule property Editor* we should first do:

    ![](https://docs.integrationmatters.com/projects/mule4/en/4.1/_images/11-installation_installer-11.png)*View from Anypoint Studio main menu -> Help -> Install new Software:*
    - *Install new Software -> Add* & provide the 2 fields asked: 

    |Name|Location| 
    |-|-|
    |Anypoint Studio Enterprise Security|http://security-update-site-1.4.3.s3.amazonaws.com|

    *Visit https://docs.mulesoft.com/release-notes/connector/anypoint-enterprise-security-release-notes for the corresponding link.*

    *Also for doubts visit: https://docs.mulesoft.com/mule-runtime/3.9/installing-anypoint-enterprise-security when Installing Anypoint Enterprise Security for Anypoint Studio*

    - Select *Premium* check box
        ![](https://miro.medium.com/max/1400/0*zNqwrImzgubIgv03)*Instalation and configuration of Secure Properties Anypoint Studio*
        - *Next -> Accept the terms of the license agreement -> Finish -> Restart Now.*
    - Once the Anypoint Studio IDE restarted, we can open the `./src/main/resources/dev-secure.yaml` file with **Mule properties editor:**
        ![](https://mulesy.com/wp-content/uploads/2020/04/word-image-114.png)*Contextual menu for the recent plugin installed: Mule Properties Editor.*

    - It is highly suggested to use a `.properties` type file to encrypt, and no RAML file.
    ![](https://rumanblogs.com/wp-content/uploads/2016/05/3.jpg)*Window view from Anypoint Studio of Mule Properties Editor*
        - Click on property Name: `password`
        - *Encrypt (You can edit the value to encrypt.) -> Choose Algorithm -> choose Key (alphanumeric 16 digits - any number; eg: password12345678)*
        ### 2. Choose algorithm and 16 digit number or alphanumeric value.
        - Select any algorithm
        *The result of the encryption is a combination of digits like: `![yLtZIHYH5PNd0pu0mN4eDT==]`* 

    - At `./src/main/resources/dev-secure.yaml` file, the Database password property can be set to the encrypted value as follows:
        ```yaml
        http:
            host: "localhost"
            path: "secure-properties"
            port: 8081
        db: 
            host: "localhost"
            port: "3306"
            user: "test"
            password: "![yLtZIHYH5PNd0pu0mN4eDT==]"
        ```
## 4. Create secure properties configuration global elements. 
- At *Anypoint Studio Canvas -> Global elments -> Create -> Connector Configuration -> Secure Properties Config*
    - Provide the file `dev-secure.yaml` 
    - Provide key value: `password12345678`
    - Choose algorithm provided before when encrypting 

## 5. Use `${secure::property}` to read a property.
- Develop a simple REST service to print on the Anypoint console the encrypted properties:
    ```xml
    <http:listener-config name="HTTP_Listener_config1" doc:name="HTTP Listener config" doc:id="9108cc31-8c69-44f2-a52e-9f21fbd83115" >
        <http:listener-connection host="0.0.0.0" port="${secure::http.port}" />
    </http:listener-config>
    <flow name="secure-prop-sampleFlow" doc:id="3991bf15-5ed7-4b28-aac2-e7694bbb29df" >
		<http:listener doc:name="Listener" doc:id="5d67b79c-122a-4141-b1b9-099e6db16ef8" config-ref="HTTP_Listener_config1" path="${secure::http.path}"/>
		<logger level="INFO" doc:name="Logger" doc:id="7551e994-c69b-4a80-a1d9-5f9fd4f280b4" message='the Database port is ${secure::db.port}'/>
	</flow>
    ```
    - Add an *HTTP Listener* to the flow:
        - Set the port to `${secure::http.port}` since we are using secure properties configuration is mandatory to use the prefix `secure::`, ``http.port `` are the resources defined at the `dev-secure.yaml` file, as well as the path `secure-properties`.
        ![](https://dz2cdn1.dzone.com/storage/temp/13775206-13.png)*Window view from the contextual menu when adding a new HTTP configuration*
        
        *`dev-secure.yaml` contents:*
        ```yaml
        http:
            host: "localhost"
            path: "secure-properties"
            port: "8081"
        db: 
            host: "localhost"
            port: "3306"
            user: "test"
            password: "![yLtZIHYH5PNd0pu0mN4eDg==]"
        ```

    - Add a *Logger* to the flow in order to log to the console the `${secure::db.port}`, this can also reach out several contents within the `dev-secure.yaml`
        - After calling the REST web service with a REST client to `http://localhost:8081/secure-properties` GET operation, *the result log to the console is the following:*
        ```
        org.mule.runtime.core.internal.processor.LoggerMessageProcessor: the Database port is. 3306
        ```
## Configuring dynamic Arguments at Anypoint Studio runtime 
- Contextual menu (right click on the main project directory):
    ![](https://blogs.mulesoft.com/wp-content/uploads/2012/06/Screen-Shot-2012-07-20-at-4.23.20-PM.png)
    - *Run as ... -> Run Configurations -> Arguments:*
        |Arguments|
        |-|
        |-Denv=dev|
        |-Denv-sec=dev-secure|
        |-Denc.key=password12345678|

        - *-Denv=dev*: ``env`` serves as a placeholder that references to *dev* which is the `dev.yaml` configuration file.  
        - *-Denv-sec=dev-secure*: ``env-sec`` serves as a placeholder that references to *dev-secure* which is the `dev-secure.yaml` encrypted configuration file.  
        - *-Denc.key=password12345678*: ``enc.key`` serves as a placeholder that references to *encrypted password value* which is within `dev-secure.yaml` configuration file.  
    - Testing again wih a REST client to the URL: , leds us to the same log result on the Anypoint Studio console achieved earlier at **5. Use `${secure::property}` to read a property.**:
    GET operation: http://localhost:8081/secure-properties
    *Result:*
    ```
    org.mule.runtime.core.internal.processor.LoggerMessageProcessor: the Database port is. 3306
    ```