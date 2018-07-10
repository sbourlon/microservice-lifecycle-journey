## Microservice Lifecycle Journey

* [Goal of the project](#goal)
* [Design](#design)
* [Develop](#develop)
* [Test](#test)

## Goal of the project <a name="goal"></a>
The goal of this experimental project is to standardize as much as possible the lifecycle of a microservice
to unify the CI/CD pipeline of all microservices: design, document, develop, deploy, test, secure, monitor.

The core component is OpenAPI (https://github.com/OAI/OpenAPI-Specification/tree/master/versions) which
standardize the contract between all actors: end users, product owners, developers, and testers.

Then, tools specialized on each actor standardize the workflow:
- design: https://openapi.design 
- document: https://github.com/Rebilly/ReDoc
- develop: https://github.com/openapitools/openapi-generator
- deploy: FaaS or https://www.docker.com/ on top of a PaaS like Kubernetes, OpenShift, Cloud Foundry, Amazon ECS
- test: https://github.com/google/oatts
- secure: https://istio.io/
- monitor and self-healing (alert handlers): https://prometheus.io/ and http://opentracing.io/

Requirements:
- API Lifecycle: OpenAPITools/openapi-generator (https://github.com/openapitools/openapi-generator)
- Docker (https://www.docker.com/community-edition#/download)
- Basic API test generator: google/oatts (https://github.com/google/oatts)
- NodeJS used by oatts
- Mocha, JavaScript test framework used by oatts: `npm install --global mocha chakram`

### Design <a name="design"></a>

Using OpenAPI v3.0.0 specification: https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md

[hello.yaml](hello.yaml)

Validate that `hello.yaml` is compliant to the OpenAPI specification.

```
java -jar ~/Downloads/openapi-generator-cli.jar validate --input-spec ./hello.yaml
```

### Develop <a name="develop"></a>
#### Build

Automatically build a Go server implementating the foundation of the Hello World microservice from `hello.yaml`

```
java -jar ~/Downloads/openapi-generator-cli.jar generate --input-spec ./hello.yaml --generator-name go-server --output ./out
```

#### Start

```
cd out
docker build . -t helloworld
docker run -p 5000:8080 helloworl:latest
```

#### Manual test
```
curl localhost:5000
```
should return:
```
Hello World!% 
```

### Test <a name="test"></a>

#### Generate the test suite

Automatically generate API tests for the API specification from `hello.yaml`.
This step is independant from the build step, so tests can be generated before the code implementation.

Google OpenAPI Test Templates is still new, so more tests could be added to their [Github project](https://github.com/google/oatts)
or added manually.

```
oatts generate --spec ./hello.yaml --writeTo ./out/test
```

#### Start the test suite
```
cd out/test
mocha --recursive  .
```
should output:
```
tests for /
    tests for get
      ✓ should respond 200 for "Hello World"
      ✓ should respond default for "unexpected error"


  2 passing (39ms)
```
