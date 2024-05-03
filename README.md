# Enabling Data Encryption for Spring Cloud Config Server in Spring Boot v2.7.*

This repo contains configuration for encrypting properties which is provided by `spring cloud config server`.

> [!NOTE]
> Setup Used:
>   * JDK 8 | 17
>   * Spring Boot v 2.7.*
>   * Spring Cloud v 2021.0.5
>   * keytool utility provided by installed jdk. (keytool is a key and certificate management utility that is part of the Java Development Kit)


## Steps:
**1. Store Creation:**

create store using following command:
```shell
keytool -genkeypair -alias myconfig -keyalg RSA -dname "CN=Config Server,OU=OrganizationalUnitName,O=OrganizationName,L=LocalityOrCityName,S=StateOrProvinceName,C=country-2letter-code" -keypass changeme -keystore keystore.jks -storepass letmein
```

copy created store in a path which can be addressed in your config-server / spring boot services, example:

**windows**:
>C:\path\to\config-server\keystore.jks

**Linux**:
>/path/to/config-server/keystore.jks

**2. Environment Variables Creation:**

  in **windows** add following environment variables:

```properties
  KEYSTORE_PATH=C:\path\to\config-server
  KEYSTORE_PASSWORD=letmein
  KEY_SECRET=changeme
```

  in **linux** add following commands to ~/.bashrc file:

```shell script
  export KEYSTORE_PATH=/path/to/config-server
  export KEYSTORE_PASSWORD=letmein
  export KEY_SECRET=changeme
```

  in **docker** implementation for config-server, add following to corresponding .env file:
 ```properties
   KEYSTORE_PATH=/path/to/config-server
   KEYSTORE_PASSWORD=letmein
   KEY_SECRET=changeme
 ```  

**3. Needed Properties:**

add following properties to bootstrap.yml file for config-server (spring cloud config server):
```yaml
  encrypt:
     keyStore:
       location: file:${KEYSTORE_PATH}/keystore.jks
       password: ${KEYSTORE_PASSWORD}
       alias: myconfig
       secret: {KEY_SECRET}
```   
> [!IMPORTANT]
> As it can be seen in above snippet, environment variables has been used, so sensitive data is not compromised in a shared git repository.

> [!IMPORTANT]
> If you want to enable automatic decryption of cloud config in clients, just add `encrypt.keyStore.*` properties to your services yml file (spring cloud config clients)

**4. Encrypt | Decrypt Text:**

Run `Spring Cloud Config server` locally, then:

* to **encrypt** texts:
```shell script
curl -X POST http://localhost:8888/encrypt -d 'mysecretpassword'
```

* to **decrypt** texts:
```shell script
curl -X POST http://localhost:8888/decrypt -d 'myEncryptedText'
```

encrypted properties in `.properties` | `.yml` files should be updated like following example (i.e.: start with `{cipher}`):

**application.yml**
```yaml
spring:
  datasource:
    username: dbuser
    password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'
``` 

**5. Disable Decryption in Production:** 

Disable decryption of env properties before sending to client in production: To disable it, you can add the following property to the application.yml file in your Spring Boot Cloud Config server application:
```yaml
   spring:
     cloud:
       config:
         server:
           encrypt:
             enabled: false
```
> [!NOTE]
> This will disable the decryption of env properties before sending to client in production, preventing anyone from potentially compromising your sensitive properties.

<hr/>

According to spring cloud config sever documentation[^1], you can safely push plain texts to a shared git repository, and the secret data remains protected. I hope you find it useful for your `Spring Cloud Config Server` data encryption and decryption.

Good luck!

[^1]: https://docs.spring.io/spring-cloud-config/reference/server/encryption-and-decryption.html
