---
title: "2 bash scripts for every Javascript/ Typescript package"
date: 2020-07-15 10:28:39
tags:
  - bash
  - script
  - typescript
  - javascript
---

## tag.sh

````bash
#!/usr/bin/env bash

tag=$1


echo "Checking dependecies are installed, installing them if not"
if ! [ -x "$(command -v yarn)" ]; then
  curl -o- -L https://yarnpkg.com/install.sh | bash
fi

if ! [ -x "$(command -v hub)" ]; then
	if ! [ -x "$(command -v brew)" ]; then
    	die "unable to install hub as homebrew is not present"
    fi
    brew install hub
fi

die () {
    echo >&2 "$@"
    exit 1
}

if [ -z "$tag" ]
then
    echo "Showing latest tags for reference:"
    TAGS=$(git fetch --tags && git tag | gsed '/-/!{s/$/_/}' | sort -V | sed 's/_$//'  | tail -n 5)

    echo "$TAGS"

    echo -n "Tag name [in format x.x.x]? "
    read IN_VERSION
    VERSION="v$IN_VERSION"
else
    VERSION="v${tag}"
fi

echo "Releasing $VERSION"

git commit --allow-empty -m "$VERSION"
git tag $VERSION -m "Tagging version $VERSION for deployment"
git push --tags
git push
hub release create -m "$VERSION" $VERSION
echo "Released $VERSION"
```

### Usage

`chmod +x ./tag.sh` to mark the script as executable

Running the script as `./tag.sh` will list the latest `semver` based tags in order, prompt you for a new one, then tag and push the commit and create a release on github.

Passing a version to the script will skip the prompt `./tag.sh 1.0.0`
## version.sh

```bash
#!/usr/bin/env bash

die () {
    echo >&2 "$@"
    exit 1
}

version_type=$1
valid=("major\tminor\tpatch")

[ "$#" -eq 1 ] || die "1 argument required, $# provided"
[[ "\t${valid[@]}\t" =~ "\t${version_type}\t" ]] || die "must be major, minor, or patch"


echo "Checking dependecies are installed, installing them if not"
if ! [ -x "$(command -v yarn)" ]; then
  curl -o- -L https://yarnpkg.com/install.sh | bash
fi

if ! [ -x "$(command -v jq)" ]; then
	if ! [ -x "$(command -v brew)" ]; then
    	die "unable to install jq as homebrew is not present"
    fi
    brew install jq
fi

echo "Version ${version_type}"

yarn version "--${version_type}"
git add package.json
cat package.json | jq .version | xargs ./tag.sh
````

### Usage

`chmod +x ./version.sh` to mark the script as executable

Run with `./version.sh [major, minor, patch]` This will use `yarn` to determine the incremented semver, update package.json and then call `./tag.sh`
