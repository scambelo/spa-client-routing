# SPA client side routing

A Lambda@Edge Function, executed upon CloudFront's "Origin Request" behavior, to re-write route requests for Angular Applications.

## Why?

S3 origins lack the ability to route Angular requests correctly to /index.html. 

The usual aproach to solve this problem is to configure distribution's Custom Error responses in order to intercept all the 403 and 404 errors form the origin and return `index.html` file instead.

However this aproach has a mayor disadvantage. When the CloudFront distribution is shared with other origins, for instance an API, the Custom Error responses configuration affects all of the origins. That could lead to some missfortunantes behaviors.

This Lambda@Edge Function rewrites Angular URI paths against /index.html.

![Flow diagram](img\diagram-Flow.png "Flow diagram")

The mayor advantage of this solution is that it only affects to desired origins.

![Architecture diagram](img\diagram-Architecture.png "Architecture diagram")

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

1. Deploy the Cloud Formation Template on `us-east-1` region.
2. Configure _**Origin Request**_ Behavior to call ARN of Lambda Version provided in the Cloud Formation template Outputs

>Alternatively it is possible integrate this Cloud Formation template in a larger stack in order to reference the lambda version ARN in the Cloud Front Behavior.

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
## Credit
Thanks to [Paul Taylor](https://github.com/ptylr) for the function's code.

## License
```
MIT License

Copyright (c) 2020 Sergio Cambelo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```