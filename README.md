# Travis CI Webinar: Using Experimental CLI Commands with Docker

## Using experimental CLI features

I will show you other methods in doing this, but if you're going to do it through docker's `config.json`, look for the following:

```json
{
  "experimental": "enabled",
  "debug": true
}
```

This for example will make `manifestation` possible, when calling `docker manifest`.

## Things you need to add in your .travis.yml 

```yaml
---
language: shell
sudo: required
dist: xenial
os: linux

services:
  - docker

addons:
  apt:
    packages:
      - docker-ce

env:
  - DEPLOY=false repo=ibmjava:jre docker_archs="amd64 ppc64le s390x"

install:
  - docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

before_script:
  - export ver=$(curl -s "https://pkgs.alpinelinux.org/package/edge/main/x86_64/curl" | grep -A3 Version | grep href | sed 's/<[^>]*>//g' | tr -d " ")
  - export build_date=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
  - export vcs_ref=$(git rev-parse --short HEAD)

  # Montana's crucial workaround
  
script:
  - chmod u+x ./travis.sh
  - chmod u+x /build.sh
  
after_success:
  - docker images
  - docker manifest inspect --verbose lucashalbert/curl # multiarch build
  - docker manifest inspect --insecure lucashalbert/curl # multiarch build 
  - docker manifest inspect --verbose ibmjava:jre # official Docker IBM Java (Multiarch) build
  - docker manifest inspect --insecure ibmjava:jre # official Docker IBM Java (Multiarch) build

branches:
  only:
    - master
  except:
    - /^*-v[0-9]/
    - /^v\d.*$/
    
    # .travis.yml created by Montana Mendy for Travis CI
 ```
 
 In order for Travis to be able to call `manifest` you need to add: 
 
 ```yaml
   - export DOCKER_CLI_EXPERIMENTAL=enabled
 ```
 
 As it suggests, this enables experimental Docker CLI commands.
