Installation - https://jfrog.com/help/r/jfrog-installation-setup-documentation/install-artifactory-single-node-with-docker

set `JFROG_HOME` environment variable to local path

run following:
```
mkdir -p $JFROG_HOME/artifactory/var/etc/
cd $JFROG_HOME/artifactory/var/etc/
touch ./system.yaml
chown -R 1030:1030 $JFROG_HOME/artifactory/var
```

On linux:
`chmod -R 1030:1030 $JFROG_HOME/artifactory/var`
On mac:
`chmod -R 777 $JFROG_HOME/artifactory/var`

Startup OSS version:
`docker run --name artifactory -v $JFROG_HOME/artifactory/var/:/var/opt/jfrog/artifactory -d -p 8081:8081 -p 8082:8082 releases-docker.jfrog.io/jfrog/artifactory-oss:latest`

might take a minute to startup

Navigate to http://localhost:8082/ui
login with admin/password, reset the password

There should already be a generic example repository created, if not you can add one via: Administration menu / Repositories / Repositories / Add Repositories.

To upload a package create a token via Application / Artifactory / Packages / Set me up / Set Up a Generic client.
Under configure type a password and generate a token

To push a package:
- install the jfrog cli - https://jfrog.com/help/r/jfrog-cli/jfrog-cli-v2-jf-installers
- configure a server in the cli: `jf config add`
  - you only need the base URL, e.g http://localhost:8081
- push a package: `jf rt upload "<file spec to push>" <repo-name>`, e.g `jf rt upload "App*.zip" example-repo-local`
- other upload examples: https://jfrog.com/help/r/jfrog-cli/example-1?tocId=sHmg160r4kTxxIIMYr209w



