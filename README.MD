# Example to push to Github package registry

# HOW TO
## Clone this project
## Create  a PGP key if not done yet

### Fast process

The defaults are ok. Choose a strong password and note it, let's name it XXXXXXX

`gpg --full-generate-key` 

The KEYID will be returned, note it down, something like 5FDC21A9E64C767A38965D1EECDF662160A7D786
  
**Export and backup your secret key somewhere safe**

`gpg --export-secret-keys 5FDC21A9E64C767A38965D1EECDF662160A7D786 > secret.key`

**Upload your public key to a public server, all server synchronize between each others**

`gpg --keyserver certserver.pgp.com --send-keys 5FDC21A9E64C767A38965D1EECDF662160A7D786`

you can check that is there with 
`gpg --keyserver certserver.pgp.com --recv-keys 5FDC21A9E64C767A38965D1EECDF662160A7D786`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       
       
## Detailed process
follow https://central.sonatype.org/pages/working-with-pgp-signatures.html       
               
## Configure Gradle       

### Fast process
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
* Create a global gradle.properties. The global properties file should be located in your home directory:
    * On Windows: C:\Users\<you>\.gradle\gradle.properties
    * On Mac/Linux: /Users/<you>/.gradle/gradle.properties

    with following content, here as an example
    ```
    GPG_SIGNING_KEY=5FDC21A9E64C767A38965D1EECDF662160A7D786
    GPG_SIGNING_PASSWORD=XXXXXXX
    ```
GPG_SIGNING_KEY is a bit special and need to be prepared as the useInMemoryPgpKeys method expects an “ascii-armored in-memory PGP secret key”.

`gpg --armor --export-secret-keys 5FDC21A9E64C767A38965D1EECDF662160A7D786 | awk 'NR == 1 { print "GPG_SIGNING_KEY=" } 1' ORS='\\n' >> gradle.properties`  

## Detailed process
follow https://central.sonatype.org/pages/gradle.html
 
# Create Releases or Snapshots

* Change group and version in build.gradle
```
group = 'com.cedricwalter'
version = '1.0-SNAPSHOT'
```

ATTENTION. as expected, SNAPSHOT version can be publisehed more than once, version not (second run return 403 HTTP)

## locally 
require a global gradle.properties (see above) then just run

`./gradle uploadArchives`

## in travis-ci.org
update the secure variable in .travis.yml to match YOUR project credential
```
travis encrypt "GPG_SIGNING_KEY=5FDC21A9E64C767A38965D1EECDF662160A7D786" --add
travis encrypt "GPG_SIGNING_PASSWORD=XXXXXXX" --add
```
Copy the generated GPG_SIGNING_KEY line from your global gradle.properties, value is way to large vor a secure variable, we need to set it as env variable
```
travis env set "GPG_SIGNING_KEY" "\n-----BEGIN PGP PRIVATE KEY BLOCK-----\n\nxxxxxxxxxxxxxxxx...xxxxxxx"
```
that's all!

# Viewing artifacts
Your artifacts are available in nexus at http://oss.sonatype.org/

# Using your artifacts in gradle
```
repositories {
        mavenCentral() 
        jcenter()
    }
```

# Annexes
if gradlew can not be read in travis, run

```
git update-index --chmod=+x gradlew"# cicd-gradle-oss-nexus" 
```