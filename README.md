# actions-s3-cache

This action enables caching dependencies to s3 compatible storage, e.g. minio, AWS S3

It also has github [actions/cache@v2](https://github.com/actions/cache) fallback if s3 save & restore fails

## Usage

```yaml
name: dev ci

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build_test:
    runs-on: [ubuntu-latest]

    steps:
      - uses: step-security/s3-actions-cache@v1
        with:
          endpoint: ${{ secrets.ENDPOINT }} # optional, default s3.amazonaws.com
          insecure: false # optional, use http instead of https. default false
          accessKey: ${{ secrets.ACCESS_KEY }}  # required
          secretKey: ${{ secrets.SECRET_KEY }} # required
          sessionToken: ${{ secrets.SESSION_TOKEN }} # optional
          bucket: ${{ secrets.BUCKET }}   # required
          use-fallback: true # optional, use github actions cache fallback, default true

          # actions/cache compatible properties: https://github.com/actions/cache
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          path: |
            node_modules
            .cache
          restore-keys: |
            ${{ runner.os }}-yarn-
```

You can also set env instead of using `with`:

```yaml
      - uses: step-security/s3-actions-cache@v1
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.ACCESS_KEY }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.SECRET_KEY }}
          # AWS_SESSION_TOKEN: "xxx"
          AWS_REGION: "us-east-1"
        with:
          endpoint: ${{ secrets.ENDPOINT }}
          bucket: ${{ secrets.BUCKET }}
          use-fallback: false
          key: test-${{ runner.os }}-${{ github.run_id }}
          path: |
            test-cache
            ~/test-cache
```

To write to the cache only:

```yaml
      - uses: step-security/s3-actions-cache/save@v1
        with:
          accessKey: ${{ secrets.ACCESS_KEY }} # required
          secretKey: ${{ secrets.SECRET_KEY }} # required
          bucket: ${{ secrets.BUCKET }} # required
          # actions/cache compatible properties: https://github.com/actions/cache
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          path: |
            node_modules
```

To restore from the cache only:

```yaml
      - uses: step-security/s3-actions-cache/restore@v1
        with:
          accessKey: ${{ secrets.ACCESS_KEY }} # required
          secretKey: ${{ secrets.SECRET_KEY }} # required
          bucket: ${{ secrets.BUCKET }} # required
          # actions/cache compatible properties: https://github.com/actions/cache
          key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
          path: |
            node_modules
```

## Restore keys

`restore-keys` works similar to how github's `@actions/cache@v2` works: It search each item in `restore-keys`
as prefix in object names and use the latest one

## Amazon S3 permissions

When using this with Amazon S3, the following permissions are necessary:

 - `s3:PutObject`
 - `s3:GetObject`
 - `s3:ListBucket`
 - `s3:GetBucketLocation`
 - `s3:ListBucketMultipartUploads`
 - `s3:ListMultipartUploadParts`
