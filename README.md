# ibm-cloud-env

[![Build Status](https://travis-ci.org/ibm-cloud-tools/ibm-cloud-env.svg?branch=master)](https://travis-ci.org/ibm-cloud-tools/ibm-cloud-env)

The `ibm-cloud-env` module allows to abstract environment variables from various Cloud compute providers, such as, but not limited to, CloudFoundry and Kubernetes, so the application could be environment-agnostic.

The module allows to define an array of search patterns that will be executed one by one until required value is found.

### Installation

```bash
npm install ibm-cloud-env
```
 
### Usage

Create a JSON file containing your mappings and initialize the module

```javascript
const IBMCloudEnv = require('ibm-cloud-env');
IBMCloudEnv.init("/path/to/the/mappings/file/relative/to/prject/root");
```

In case mappings file path is not specified in the `IBMCloudEnv.init()` the module will try to load mappings from a default path of `/server/config/mappings.json`.
 
#### Supported search patterns types
ibm-cloud-config supports searching for values using three search pattern types - cloudfoundry, env, file. 
- Using `cloudfoundry` allows to search for values in VCAP_SERVICES and VCAP_APPLICATIONS environment variables
- Using `env` allows to search for values in environment variables
- Using `file` allows to search for values in text/json files

#### Example search patterns
- cloudfoundry:service-instance-name - searches through parsed VCAP_SERVICES environment variable and returns the `credentials` object of the matching service instance name
- cloudfoundry:$.JSONPath - searches through parsed VCAP_SERVICES and VCAP_APPLICATION environment variables and returns the value that corresponds to JSONPath
- env:env-var-name - returns environment variable named "env-var-name"
- env:env-var-name:$.JSONPath - attempts to parse the environment variable "env-var-name" and return a value that corresponds to JSONPath
- file:/server/config.text - returns content of /server/config.text file
- file:/server/config.json:$.JSONPath - reads the content of /server/config.json file, tries to parse it, returns the value that corresponds to JSONPath

#### mappings.json file example
```javascript
{
    "service1-credentials": {
        "searchPatterns": [
            "cloudfoundry:my-service1-instance-name", 
            "env:my-service1-credentials", 
            "file:/localdev/my-service1-credentials.json" 
        ]
    },
    "service2-username": {
        "searchPatterns":[
            "cloudfoundry:$.service2[@.name=='my-service2-instance-name'].credentials.username",
            "env:my-service2-credentials:$.username",
            "file:/localdev/my-service1-credentials.json:$.username" 
        ]
    }
}
```

### Using the values in application

In your application retrieve the values using below commands

```javascript
var service1credentials = IBMCloudEnv.getDictionary("service1-credentials"); // this will be a dictionary
var service2username = IBMCloudEnv.getString("service2-username"); // this will be a string
```

Following the above approach your application can be implemented in an runtime-environment agnostic way, abstracting differences in environment variable management introduced by different cloud compute providers.
