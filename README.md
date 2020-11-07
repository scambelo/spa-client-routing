# SPA client side routing

A Lambda@Edge Function, executed upon CloudFront's _Origin Request_ behavior, to re-write route requests for Angular Applications.

## Why?

S3 origins lack the ability to route Angular requests correctly to `/index.html`. 

The usual aproach to solve this problem is to configure distribution's _Custom Error responses_ in order to intercept all the 403 and 404 errors form the origin and return `index.html` file instead.

However this aproach has a mayor disadvantage. When the CloudFront distribution is shared with other origins, for instance an API, the _Custom Error responses_ configuration affects all of the origins. That could lead to some missfortunantes behaviors.

This Lambda@Edge Function rewrites Angular URI paths against `/index.html`.

![Flow diagram](img/diagram-Flow.png "Flow diagram")

The mayor advantage of this solution is that it only affects to desired origins.

![Architecture diagram](img/diagram-Architecture.png "Architecture diagram")

## Lambda Function code

The code of the Lambda Function is vastly based on the work of [Paul Taylor](https://github.com/ptylr/Lambda-at-Edge/tree/master/EdgeAngular) with minor modifications.


```javascript
'use strict';
exports.handler = (event, _context, callback) => {
    const defaultPath = 'index.html';
    const request = event.Records[0].cf.request;
    const isFile = uri => /^\/.+(\.\w+$)/.test(uri);
    if (!isFile(request.uri)) {
        const olduri = request.uri;
        request.uri = '/' + defaultPath;
        console.log('Request for [' + olduri + '], rewritten to [' + request.uri + ']');
    }
    callback(null, request);
};
```

## Deployment

1. Deploy the CloudFormation Template on `us-east-1` region.
2. Configure _Origin Request_ Behavior to call ARN of Lambda Version provided in the CloudFormation template Outputs

>Alternatively it is possible to integrate this CloudFormation template in a larger stack in order to reference the lambda version ARN in the CloudFront Behavior.

```yaml
Resources:
  CloudFrontDist:
    Type: AWS::CloudFront::Distribution
    Properties: 
      DistributionConfig: 
        DefaultCacheBehavior:
          LambdaFunctionAssociations:
            - 
              EventType: origin-request
              LambdaFunctionARN: !Ref LambdaVersion
```
## Credits
Thanks to [Paul Taylor](https://github.com/ptylr) for the function's code.