# Using S3 compatible object storage in Golang

You can find more examples and code samples written in Golang that demonstrate how to interact with Amazon Simple Storage Service (Amazon S3) on the page [Golang Code Samples for Amazon S3](https://aws.github.io/aws-sdk-go-v2/docs/code-examples/s3).

[[toc]]

## Installing the required SDKs

* The AWS SDK for Go V2 uses Go Modules, which was a feature introduced in Go 1.11. If you haven't initialized your GitHub/GitLab project as a Go module yet, run the command ==`go mod init github.com/<user>/<repository>`== . This writes a new `go.mod` file in the current root project directory in order to track your code's dependencies. The `go.mod` file must not already exist. If you were to publish this module, there has to be a path from which Go tools can download it.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Application code is located in a subdirectory
If your module code is not placed in the project root directly, but located in its subdirectory, such as the `app`, importing this module from other modules would be done as follows:

```go
import {
  "github.com/<user>/<repository>/app"
}
```

:::
<!-- markdownlint-enable DOCSMD004 -->

* To retrieve the core SDK Go V2 module, the config module which is used for loading the AWS configuration, and the AWS S3 service API client, run the following commands:

```shell
go get github.com/aws/aws-sdk-go-v2
go get github.com/aws/aws-sdk-go-v2/config
go get github.com/aws/aws-sdk-go-v2/service/s3
```

* Both the Go commands, used later for running the application, `go run` locally, and compiling the [final binary executable](/documentation/services/runtimes/golang.html#start-command) `go build`, can download these dependency modules when needed.

## How to get access credentials

Using the following function, ==`getCredentials`== , you can get an object which will be used later for authentication when creating buckets and their content.

Assuming that the code is associated with access to the Zerops Object Storage Service, the [object storage name](/documentation/services/storage/s3.html#object-storage-name) of which was chosen as the ==**`store`**== . The necessary [Storage access details](/documentation/services/storage/s3.html#from-local-environment) values **Access Key Id** and **Secret Access Key** are then taken from the [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables).

<!-- markdownlint-disable DOCSMD004 -->
::: tip
The unique generated id of the created Zerops Object Storage Service instance is used as the value of the **accessKeyId** environment variable. The value of **secretAccessKey** is a 16-character-long random string.
:::
<!-- markdownlint-enable DOCSMD004 -->

```go
// Include the necessary AWS SDK modules.
import (
  "os"
  "github.com/aws/aws-sdk-go-v2/credentials"
)

// Object storage name.
const storeObjectStorageName = "store"

// Function returning a pointer to a variable with an accessKeyId value.
func getAccessKeyIdValue(objectStorageName string) *string {
  // Necessary environment variable name.
  const accessKeyId = "accessKeyId"
  value, found := os.LookupEnv(objectStorageName + "_" + accessKeyId)
  // If the environment variable has been found ...
  if found {
    return &value
  }
  return nil
}

// Function returning a pointer to a variable with a secretAccessKey value.
func getSecretAccessKeyValue(objectStorageName string) *string {
  // Necessary environment variable name.
  const secretAccessKey = "secretAccessKey"
  value, found := os.LookupEnv(objectStorageName + "_" + secretAccessKey)
  // If the environment variable has been found ...
  if found {
    return &value
  }
  return nil
}

// Function returning a pointer to a variable with a user credentials.
func getCredentials(objectStorageName string) *credentials.StaticCredentialsProvider {
  accessKeyIdValue := getAccessKeyIdValue(objectStorageName)
  secretAccessKeyValue := getSecretAccessKeyValue(objectStorageName)
  if accessKeyIdValue != nil && secretAccessKeyValue != nil {
    credentials := credentials.NewStaticCredentialsProvider(*accessKeyIdValue, *secretAccessKeyValue, "")
    // A pointer to a variable with credentials is returned.
    return &credentials
  }
  // If any environment variable not found, return only the nil (it's compatible with a pointer).
  return nil
}
```

## How to get an S3 API client

Once you get the `getCredentials` function from the previous code snippet (supposing all declared variables are also accessible), you can create an S3 SDK client to access the S3 API.

```go
// Include the necessary AWS SDK modules.
import (
  "context"
  "errors"
  "os"
  "github.com/aws/aws-sdk-go-v2/config"
  "github.com/aws/aws-sdk-go-v2/credentials"
  "github.com/aws/aws-sdk-go-v2/service/s3"
)

/**
 * The way that allows you to pass in a <context> to your program. It should only be used at a high level
 * (in the main or top-level request handler). This can be used to derive other contexts. Here, it's only
 * a formal API requirement and not used in any way.
 */
var ctx = context.Background()

// Function declaration: Getting an S3 SDK client
func getS3Client(
  ctx context.Context,
  objectStorageName string,
  credentials *credentials.StaticCredentialsProvider
) (*s3.Client, error) {
  // Necessary environment variable name.
  const apiUrl = "apiUrl"
  // Getting the environment variable value.
  apiUrlValue, apiUrlFound := os.LookupEnv(objectStorageName + "_" + apiUrl)
  if !apiUrlFound {
    return nil, errors.New("non-existed apiUrl environment variable")
  }

  // Obtaining the S3 SDK client configuration based on the passed parameters.
  cnf, err := config.LoadDefaultConfig(
    ctx,
    config.WithCredentialsProvider(credentials),
    // Zerops supports only the S3 default region for API calls.
    // It doesn't mean that the physical HW infrastructure is located there also.
    // All Zerops infrastructure is completely located in Europe/Prague.
    config.WithRegion("us-east-1"),
  )
  if err != nil {
    return nil, err
  }

  // Create a new S3 SDK client instance.
  s3Client := s3.NewFromConfig(
    // Passing the S3 SDK client configuration created before.
    cnf,
    s3.WithEndpointResolver(
      // Applying of the Zerops Object Storage API URL endpoint.
      s3.EndpointResolverFromURL(apiUrlValue),
    ),
    func(opts *s3.Options) {
      // Zerops supports currently only S3 path-style addressing model.
      // The virtual-hosted style model will be supported in near future.
      opts.UsePathStyle = true
    },
  )
  if s3Client != nil {
    return s3Client, nil
  }
  return nil, errors.New("creating an S3 SDK client failed")
}
```

## Creating a new object storage bucket

Once you get the `getCredentials` and `getS3Client` functions from the previous code snippet (supposing all declared variables are also accessible), you can create a named bucket as a container for storing objects. Remember, it's vital that all buckets created in Zerops as a whole have unique names. See the recommendation for [bucket naming convention](/documentation/services/storage/s3.html#used-technology).

```go
// Include the necessary AWS SDK modules.
import {
  "fmt"
  "os"
  "github.com/aws/aws-sdk-go-v2/service/s3"
}

// Required bucket name.
const localBucketName = "records";

// Function declaration: Getting a unique bucket name based on <accessKeyId> value
func getUniqueBucketName(objectStorageName string, localBucketName string) (*string, error) {
  // Necessary environment variable names.
  const accessKeyId = "accessKeyId"
  // All bucket names in the Zerops shared object storage namespace have to be unique!
  // Getting the environment variable value that will be used as the unique prefix.
  value, found := os.LookupEnv(objectStorageName + "_" + accessKeyId)
  if !found {
    return nil, errors.New("non-existed accessKeyId environment variable")
  }
  // Unique bucket name preparation.
  bucketName := value + "." + localBucketName;
  return &bucketName, nil
}

// Function declaration: Getting an S3 SDK client
func createBucket(
  ctx context.Context,
  objectStorageName string,
  s3Client *s3.Client,
  localBucketName string,
) (*s3.CreateBucketOutput, error) {
  // Unique bucket name preparation.
  bucketName, err := getUniqueBucketName(objectStorageName, localBucketName)
  if err != nil {
    return nil, err
  }
  // Invoking the S3 SDK client method to create a new bucket.
  return s3Client.CreateBucket(ctx, &s3.CreateBucketInput{
    Bucket: bucketName,
  })
}

// Function declaration: Getting all buckets for the current user credentials
func listBuckets(ctx context.Context, s3Client *s3.Client) (*s3.ListBucketsOutput, error) {
  // Invoking the S3 SDK client method to get a list of existed buckets.
  return s3Client.ListBuckets(ctx, nil)
}

// Calling the function: getCredentials
storeCredentials := getCredentials(storeObjectStorageName)
if storeCredentials != nil {
  // Calling the function: getS3Client
  s3Client, err := getS3Client(ctx, storeObjectStorageName, storeCredentials)
  if s3Client != nil {
    // Calling the function: createBucket
    createBucketOutput, err := createBucket(ctx, storeObjectStorageName, s3Client, localBucketName)
    // Calling the function: listBuckets
    listBucketsOutput, err := listBuckets(ctx, s3Client)
    if err == nil {
      // Printing the existed bucket names
      for _, bucket := range listBucketsOutput.Buckets {
        fmt.Println(*bucket.Name)
      }
    }
  }
}
```

<!-- markdownlint-disable DOCSMD004 -->
::: tip New bucket URL created
As the S3 path-style addressing model is used, the bucket's effective URL is:

```url
https://s3.app.zerops.io/records
```

:::
<!-- markdownlint-enable DOCSMD004 -->

## Getting an existing object storage bucket ACL setting

Once you have the `getCredentials`, `getUniqueBucketName`, and `getS3Client` functions from the previous code snippet (supposing all declared variables are also accessible), you can check the existence of a bucket and get its ACL setting.

```go
// Include the necessary modules.
import (
  "encoding/json"
)

// Function declaration: Getting an exited bucket ACL
func getBucketAcl(
  ctx context.Context,
  objectStorageName string,
  s3Client *s3.Client,
  localBucketName string,
) (*s3.GetBucketAclOutput, error) {
  bucketName, err := getUniqueBucketName(objectStorageName, localBucketName)
  if bucketName != nil {
    // Check, if the required bucket exists.
    _, err := s3Client.HeadBucket(ctx, &s3.HeadBucketInput{
      Bucket: bucketName,
    })
    if err != nil {
      return nil, err
    }
    // If it exists, get its ACL.
    return s3Client.GetBucketAcl(ctx, &s3.GetBucketAclInput{
      Bucket: bucketName,
    })
  }
  return nil, err
}

// Calling the function: getCredentials
storeCredentials := getCredentials(storeObjectStorageName)
if storeCredentials != nil {
  // Calling the function: getS3Client
  s3Client, err := getS3Client(ctx, storeObjectStorageName, storeCredentials)
  if s3Client != nil {
    // Calling the function: getBucketAcl
    bucketAclOutput, err := getBucketAcl(ctx, storeObjectStorageName, s3Client, localBucketName)
    if err == nil {
      // Formatting and printing the returned value of the bucket ACL.
      grantsJSON, err := json.MarshalIndent(bucketAclOutput.Grants, "", "  ")
      fmt.Printf("%s\n", string(grantsJSON))
    }
  }
}
```

<!-- markdownlint-disable DOCSMD004 -->
::: tip Grants setting for Default buckets
As mentioned in the documentation [Object storage owner identity](/documentation/services/storage/s3.html#object-storage-owner-identity), there is only the one grantee, equal to the owner, with the same name as the chosen [Object storage name](#object-storage-name) (for example, ==`store`== ).

```json
{
  "Grants": [{
    "Grantee": {
      "DisplayName": "store",
      "ID": "zz92gyuvr4yzf0kajfalzg",
      "Type": "CanonicalUser"
    },
    "Permission": "FULL_CONTROL"
  }]
}
```

:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: tip Default bucket's @metadata headers
It's also worth listing the default setting of a bucket's headers related to the HTTPS public read/write access (authenticated access is not currently available through RESTful interface):

```json
{
  "@metadata": {
    "headers": {
      "access-control-allow-origin": "*",
      "access-control-allow-methods": "GET, POST, PUT, DELETE, OPTIONS",
      "access-control-allow-headers": "DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization",
      "access-control-expose-headers": "Content-Length,Content-Range"
    }
  }
}
```

The same default **@metadata headers** setting is also used for all added bucket objects.

:::
<!-- markdownlint-enable DOCSMD004 -->

## Update an existing object storage bucket ACL

Once you have the `getCredentials`, `getUniqueBucketName`, and `getS3Client` functions from the previous code snippet (supposing all declared variables are also accessible), you can check the existence of a bucket and modify its ACL setting to one of [canned grants](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html#canned-acl).

```go
import (
  s3Types "github.com/aws/aws-sdk-go-v2/service/s3/types"
)

// Function declaration: Updating an exited bucket ACL
func putBucketAcl(
  ctx context.Context,
  objectStorageName string,
  s3Client *s3.Client,
  localBucketName string,
  cannedAcl s3Types.BucketCannedACL,
) (*s3.GetBucketAclOutput, error) {
  bucketName, err := getUniqueBucketName(objectStorageName, localBucketName)
  if bucketName != nil {
    // Check, if the required bucket exists.
    _, err := s3Client.HeadBucket(ctx, &s3.HeadBucketInput{
      Bucket: bucketName,
    })
    if err != nil {
      return nil, err
    }
    // If it exists, update its ACL.
    return s3Client.PutBucketAcl(ctx, &s3.PutBucketAclInput{
      Bucket: bucketName,
      ACL: cannedAcl,
    })
  }
  return nil, err
}

// Calling the function: getCredentials
storeCredentials := getCredentials(storeObjectStorageName)
if storeCredentials != nil {
  // Calling the function: getS3Client
  s3Client, err := getS3Client(ctx, storeObjectStorageName, storeCredentials)
  if s3Client != nil {
    // Calling the function: putBucketAcl
    _, err = putBucketAcl(
      ctx,
      storeObjectStorageName,
      s3Client,
      localBucketName,
      s3Types.BucketCannedACLPublicRead,
    )
  }
}
```

Setting the ACL as ==`public-read`== adds another `Grantee` to the bucket's `Grants` list:

```json
{
   "Grantee": {
      "Type": "Group",
      "URI": "http://acs.amazonaws.com/groups/global/AllUsers"
   },
   "Permission": "READ"
}
```

<!-- markdownlint-disable DOCSMD004 -->
::: tip Public read access to a bucket
Setting the ACL to ==`public-read`== means that the URL `https://s3.app.zerops.io/records` will be accessible with read access to anyone.
:::
<!-- markdownlint-enable DOCSMD004 -->

If you have another Zerops Object Storage Service in your project (for example, one with ==`archive`== [Object Storage Name](/documentation/services/storage/s3.html#object-storage-bucket-names)), you can set its bucket access rights to allow access from the ==`store`== service with [read](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html#permissions) access.

<!-- markdownlint-disable DOCSMD004 -->
::: warning Grants work in overwriting mode
Setting ==`AccessControlPolicy`== requires defining the ==`Grants`== property with the complete list of items ( ==`Grantee`== ) and also the ==`Owner`== property for the bucket because it overwrites the previous value. It doesn't work in incremental mode. You will probably create a more sophisticated routine to prepare the **access control policy** you need. Here, it's simplified to the simple calculated value.
:::
<!-- markdownlint-enable DOCSMD004 -->

```go
import (
  s3Types "github.com/aws/aws-sdk-go-v2/service/s3/types"
)

// Function declaration: Setting an bucket access control policy
func putBucketAclPolicy(
  ctx context.Context,
  objectStorageName string,
  s3Client *s3.Client,
  localBucketName string,
  aclPolicy *s3Types.AccessControlPolicy,
) (*s3.PutBucketAclOutput, error) {
  bucketName, err := getUniqueBucketName(objectStorageName, localBucketName)
  if bucketName != nil {
    // Check, if the required bucket exists.
    _, err := s3Client.HeadBucket(ctx, &s3.HeadBucketInput{
      Bucket: bucketName,
    })
    if err != nil {
      return nil, err
    }
    // If it exists, update its ACL.
    return s3Client.PutBucketAcl(ctx, &s3.PutBucketAclInput{
      Bucket: bucketName,
      AccessControlPolicy: aclPolicy,
    })
  }
  return nil, err
}

// Object storage names.
const storeObjectStorageName = "store";
const archiveObjectStorageName = "archive";
// Calling the function: getCredentials
archiveCredentials := getCredentials(archiveObjectStorageName)
if archiveCredentials != nil {
  // Calling the function: getS3Client
  s3Client, err := getS3Client(ctx, archiveObjectStorageName, archiveCredentials)
  if s3Client != nil {
    // Calling the function: getBucketAcl
    bucketAclOutput, err := getBucketAcl(ctx, archiveObjectStorageName, s3Client, localBucketName)
    if err == nil {
      // Getting of the current bucket owner
      currentOwnerDisplayName := *bucketAclOutput.Owner.DisplayName
      currentOwnerId := *bucketAclOutput.Owner.ID
      // Getting existing grants
      currentGrants := bucketAclOutput.Grants

      // A new grantee represents the read access under the identity of the <store> object storage service.
      newGranteeId := getAccessKeyIdValue(storeObjectStorageName)
      // Create a new grantee
      newGrantee := s3.Grantee{
        DisplayName: &storeObjectStorageName,
        ID: &newGranteeId,
        Type: s3Types.TypeCanonicalUser
      }
      // Create a new grant (a new grantee with read access permission).
      newGrant := s3.Grant{Grantee: &newGrantee, Permission: s3Types.PermissionRead}
      // Setting the updated access control policy
      accessControlPolicy := s3Types.AccessControlPolicy{
        Grants: append(currentGrants, newGrant),
        Owner: &s3Types.Owner{
          DisplayName: &currentOwnerDisplayName,
          ID: &currentOwnerId,
        },
      }
      // Calling the function: putBucketAclPolicy
      _, err = putBucketAclPolicy(
        ctx,
        archiveObjectStorageName,
        s3Client,
        localBucketName,
        &accessControlPolicy,
      )
    }
  }
}
```

<!-- markdownlint-disable DOCSMD004 -->
::: warning Cross-account access enabled buckets
If the ACL of buckets is modified to allow access from another account as shown above (read access to buckets of `archive` object storage service using the credentials of `store` object storage service), it's important to understand that these are not implicitly listed among all available buckets of `store` object storage service (for example, using `s3Client.ListBuckets(ctx, nil)`). They are called **external buckets**. You need to access them explicitly based on their known bucket names.
:::
<!-- markdownlint-enable DOCSMD004 -->

## Adding a new bucket object (with body or a file)

Once you have the `getCredentials`, `getUniqueBucketName`, and `getS3Client` functions from the previous code snippet (supposing all declared variables are also accessible), you can put a new object inside a bucket. You have to have WRITE permissions on a bucket to add an object to it.

```go
import (
  "bytes"
  "io"
  "os"
  "path/filepath"
)

func putObject(
  ctx context.Context,
  objectStorageName string,
  s3Client *s3.Client,
  localBucketName string,
  objectKey string,
  objectContent io.Reader,
) (*s3.PutObjectOutput, error) {
  bucketName, err := getUniqueBucketName(objectStorageName, localBucketName)
  if bucketName != nil {
    return s3Client.PutObject(ctx, &s3.PutObjectInput{
      Bucket: bucketName,
      Key: &objectKey,
      Body: objectContent,
    })
  }
  return nil, err
}

// Declaration of an object with a plain text body to be inserted into a bucket.
const objectKey = "K1.txt";
const objectBody = "Description of the K1.";
// Declaration of an object with a file to be inserted into a bucket.
const objectFileKey = "scan_20210815_00001.jpg"
currentDir, _ := os.Getwd()
// The file is located in the subdirectory: ./files
filePath := filepath.Join(currentDir, "files", objectFileKey)
objectFileContent, err := os.Open(filePath)

// Calling the function: getCredentials
storeCredentials := getCredentials(storeObjectStorageName)
if storeCredentials != nil {
  // Calling the function: getS3Client
  s3Client, err := getS3Client(ctx, storeObjectStorageName, storeCredentials)
  if s3Client != nil {
    // Calling the function: putObject
    // Inserting an object with a plain text body into a bucket.
    _, err = putObject(
      ctx,
      storeObjectStorageName,
      s3Client,
      localBucketName,
      // The key, under which the body content will be saved.
      objectKey,
      // Transformation of a plain text body to a binary content.
      bytes.NewReader([]byte(objectBody)),
    }
    // Calling the function: putObject
    // Inserting an object with a file into a bucket.
    _, err = putObject(
      ctx,
      storeObjectStorageName,
      s3Client,
      localBucketName,
      // The key, under which the file will be saved.
      objectFileKey,
      // The binary content of the file.
      objectFileContent,
    }
  }
}
```

## Getting an existing bucket object (with body or a file)

Once you have the `getCredentials`, `getUniqueBucketName`, and `getS3Client` functions from the previous code snippet (supposing all declared variables are also accessible), you can get an object that already exists back from a bucket. You have to have at least READ permissions on a bucket to retrieve an object from it.

```go
import (
  "io"
  "os"
  "path/filepath"
)

func getObject(
  ctx context.Context,
  objectStorageName string,
  s3Client *s3.Client,
  localBucketName string,
  objectKey string,
) (*s3.GetObjectOutput, error) {
  bucketName, err := getUniqueBucketName(objectStorageName, localBucketName)
  if bucketName != nil {
    return s3Client.GetObject(ctx, &s3.GetObjectInput{
      Bucket: bucketName,
      Key: &objectKey,
    })
  }
  return nil, err
}

// Declaration of an object key to be retrieved as a plain text body from a bucket.
const objectKey = "K1.txt";
// Declaration of an object key to be retrieved as a file from a bucket.
const objectFileKey = "scan_20210815_00001.jpg"
currentDir, _ := os.Getwd()
// The file should be written back to the subdirectory: ./files
filePath := filepath.Join(currentDir, "files", objectFileKey)
objectFileContent, err := os.Create(filePath)

// Calling the function: getCredentials
storeCredentials := getCredentials(storeObjectStorageName)
if storeCredentials != nil {
  // Calling the function: getS3Client
  s3Client, err := getS3Client(ctx, storeObjectStorageName, storeCredentials)
  if s3Client != nil {
    // Calling the function: getObject
    // Retrieving an object with a plain text body from a bucket.
    outputBody, err := getObject(ctx, storeObjectStorageName, s3Client, localBucketName, objectKey)
    if err == nil {
      resultBody := outputBody.Body
      defer resultBody.Close()
      bodyContent, err := io.ReadAll(resultBody)
      if err == nil {
        // Getting the previously saved value: "Description of the K1."
        stringBodyContent := string(bodyContent)
      }
    // Calling the function: getObject
    // Retrieving an object with a file from a bucket.
    outputFile, err := getObject(ctx, storeObjectStorageName, s3Client, localBucketName, objectFileKey)
    if err == nil {
      resultFile := getObjectOutputFile.Body
      defer objectFileContent.Close()
      defer resultFile.Close()
      // Saving the retrieved binary object content to a local file.
      _, err = objectFileContentArchive.ReadFrom(resultFile)
    }
  }
}
```
