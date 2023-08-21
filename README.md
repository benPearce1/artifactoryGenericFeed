We have a company hosted artifactory generic repo we could use for testing:  https://packages.octopushq.com/ui/repos/tree/General/zip-local


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

The Pro version has a different API surface, so we might need to either use the company artifactory instance or get a trial license:
```
curl -X GET -uadmin:cmVmdGtuOjAxOjE3MjM2MDY0Mzg6bUk4c2tlUzlmMFZkVW1LbWNUUDZPaU80YVN5 http://localhost:8082/artifactory/api/repositories/configurations
{
  "errors" : [ {
    "status" : 400,
    "message" : "This REST API is available only in Artifactory Pro (see: jfrog.com/artifactory/features). If you are already running Artifactory Pro please make sure your server is activated with a valid license key.\n"
  } ]
}%
```

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

To search with the jfrog cli:
`jf rt search "example-repo-local/Test*.zip"`

To download a zip file:
`jf rt dl "example-repo-local/TestWebApp.1.0.2-beta002.zip"`


Artifactory supports folders, which form part of the repository path. I don't think we should support nested folders

Useful API calls:
`curl -X GET -uadmin:cmVmdGtuOjAxOjE3MjM2MTU2MTA6MDBnYnVLaXVkTncwV2ZBU3VvbnVzcVAwc2Ni http://localhost:8082/artifactory/api/storage/example-repo-local/`

```
{
  "repo" : "example-repo-local",
  "path" : "/",
  "created" : "2023-08-15T06:05:41.490Z",
  "lastModified" : "2023-08-15T06:05:41.490Z",
  "lastUpdated" : "2023-08-15T06:05:41.490Z",
  "children" : [ {
    "uri" : "/TestWebApp.1.0.0.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp.1.0.1.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp.1.0.2-beta.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp.1.0.2-beta002.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp.1.0.2-beta003.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp.1.0.2.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp2.1.0.0.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp2.1.0.1.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp2.1.0.2-beta.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp2.1.0.2-beta002.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp2.1.0.2-beta003.zip",
    "folder" : false
  }, {
    "uri" : "/TestWebApp2.1.0.2.zip",
    "folder" : false
  } ],
  "uri" : "http://localhost:8082/artifactory/api/storage/example-repo-local"
}%
```

`curl -X GET -uadmin:cmVmdGtuOjAxOjE3MjM2MTU2MTA6MDBnYnVLaXVkTncwV2ZBU3VvbnVzcVAwc2Ni http://localhost:8082/artifactory/api/storage/example-repo-local/TestWebApp.1.0.0.zi
p`

```
{
  "repo" : "example-repo-local",
  "path" : "/TestWebApp.1.0.0.zip",
  "created" : "2023-08-15T06:09:32.200Z",
  "createdBy" : "admin",
  "lastModified" : "2023-08-15T06:09:32.200Z",
  "modifiedBy" : "admin",
  "lastUpdated" : "2023-08-15T06:09:32.201Z",
  "downloadUri" : "http://localhost:8082/artifactory/example-repo-local/TestWebApp.1.0.0.zip",
  "mimeType" : "application/zip",
  "size" : "679677",
  "checksums" : {
    "sha1" : "d6230604262fa191c6ace5d047562084ae863fbf",
    "md5" : "87cf854c700475483251f622a8b615c9",
    "sha256" : "945319dd0c3585cddcbd455d147d588072fc12513d822228d354af79fb214d8a"
  },
  "originalChecksums" : {
    "sha1" : "d6230604262fa191c6ace5d047562084ae863fbf",
    "md5" : "87cf854c700475483251f622a8b615c9",
    "sha256" : "945319dd0c3585cddcbd455d147d588072fc12513d822228d354af79fb214d8a"
  },
  "uri" : "http://localhost:8082/artifactory/api/storage/example-repo-local/TestWebApp.1.0.0.zip"
}%
```


Sample PRs from OCI repository work:
https://github.com/OctopusDeploy/OctopusDeploy/pull/17361
https://github.com/OctopusDeploy/Calamari/pull/1001
https://github.com/OctopusDeploy/docs/pull/1968
