# Multi-configuration Isolation

Seata supports Multi-configuration Isolation since 0.6.1,You can configure it in the following steps.

## use case 

Suppose we now have a test environment in which we want to read only the configuration items corresponding to the test environment.

### 1.Environment Configuration 

Seata provides two ways to set up different environments:

- **-Denv=test**,where test is the name of the environment.
```shell

e.g.(Linux)

sh seata-server.sh -Denv=test
```
- Use **SEATA_CONFIG_ENV** as the key of environment variable,and it's value will be the name of the environment.[**recommended**]
```shell

e.g.(Linux)

#vi /etc/profile 

export SEATA_CONFIG_ENV=test

:wq

#source /etc/profile
```

### 2.Name the new configuration file

- Rename file.conf to file-env.conf,where env is the name of the environment. e.g. **file-test.conf**
- Rename registry.conf to registry-env.conf,where env is the name of the environment. e.g. **registry-test.conf**

After all the steps have been set up, you can start using Seata configuration isolation.