# Desmos fork of New Relic log ingestion S3 lambda function

The New Relic log ingestion S3 function doesn't handle timestamps very well, and upstream PRs to address
this have not been reviewed, so it's necessary to run our own fork of this code.

## Testing locally

See DEVELOPER.md for more detailed instructions, which involve passing data through via an S3 url and a fake message.

```shell
make build
LOG_TYPE=alb LICENSE_KEY=foobar TEST_FILE="./test/mock.json" make run
```

A faster way to test locally is to modify the `handler.py` script to directly run part of the code in the main function and then
use a step debugger. E.g.:

```python
if __name__ == "__main__":
    print(_package_log_payload({ "entry": [
    'https 2024-02-14T23:56:09.786226Z app/knox-production-blue-alb/server-hash 1.1.1.1:28756 1.1.1.1:80 0.000 0.026 0.000 200 200 332 26165 "GET https://www.desmos.com:443/calc_thumbs/production/fuq5f9ufmt.png HTTP/1.1" "Amazon CloudFront" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 arn:aws:elasticloadbalancing:us-east-1:id:targetgroup/knox-production-blue-tg/7826de5ad4fae1b0 "Root=1-65cd5319-6f53b81a505790b803e29fcf" "www.desmos.com" "session-reused" 0 2024-02-14T23:56:09.759000Z "forward" "-" "-" "1.1.1.1:80" "200" "-" "-"""',
'https 2024-02-14T23:56:09.867964Z app/knox-production-blue-alb/server-hash 1.1.1.1:28128 - 0.000 0.022 0.000 200 200 1746 188 "POST https://www.desmos.com:443/data-events HTTP/1.1" "Mozilla/5.0 (X11; CrOS x86_64 14541.0.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/1.1.1.1 Safari/537.36" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 arn:aws:elasticloadbalancing:us-east-1:id:targetgroup/knox-p-AlbLi-CVM7KDU26SWA/6fbe1b2e1516cd26 "Root=1-65cd5319-1a38f07746a9f0f4338078c8" "www.desmos.com" "arn:aws:acm:us-east-1:id:certificate/34af6c4e-ca02-4055-acc5-3c64e1655232" 5 2024-02-14T23:56:09.845000Z "forward" "-" "-" "-" "200" "-" "-"',
'https 2024-02-14T23:56:09.957753Z app/knox-production-blue-alb/server-hash 1.1.1.1:10556 1.1.1.1:80 0.000 0.001 0.000 200 200 1248 16760 "GET https://www.desmos.com:443/assets/img/partner-logos/CA-RGB.png HTTP/1.1" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Safari/605.1.15" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 arn:aws:elasticloadbalancing:us-east-1:id:targetgroup/knox-production-blue-tg/7826de5ad4fae1b0 "Root=1-65cd5319-7fd7dad26c22a9bf4fe646e4" "www.desmos.com" "session-reused" 0 2024-02-14T23:56:09.956000Z "forward" "-" "-" "1.1.1.1:80" "200" "-" "-""',
'https 2024-02-14T23:56:10.053869Z app/knox-production-blue-alb/server-hash 1.1.1.1:15776 - 0.000 0.028 0.000 200 200 2049 207 "POST https://www.desmos.com:443/usage-stats HTTP/1.1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/1.1.1.1 Safari/537.36" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 arn:aws:elasticloadbalancing:us-east-1:id:targetgroup/knox-p-AlbLi-6CYRTEME1L9D/6c484ca0798de0da "Root=1-65cd531a-3176c6855e6dd3d541fde966" "www.desmos.com" "session-reused" 4 2024-02-14T23:56:10.025000Z "forward" "-" "-" "-" "200" "-" "-"',
        'https 2024-02-14T23:56:10.140257Z app/knox-production-blue-alb/server-hash 1.1.1.1:26404 - 0.000 0.029 0.000 200 200 1797 188 "POST https://www.desmos.com:443/data-events HTTP/1.1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/1.1.1.1 Safari/537.36 Edg/1.1.1.1" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 arn:aws:elasticloadbalancing:us-east-1:id:targetgroup/knox-p-AlbLi-CVM7KDU26SWA/6fbe1b2e1516cd26 "Root=1-65cd531a-1e77ea645782e318179dff6c" "www.desmos.com" "session-reused" 5 2024-02-14T23:56:10.111000Z "forward" "-" "-" "-" "200" "-" "-""']
                                 }))
    # lambda_handler('', '')
```

If you're going to do that, you'll also need to manually return a `log_type` or set that environment variable.

## Packaging for deployment

```shell
make build
BUCKET=newrelic-cloudfront-ingestion-lambda-package REGION=us-east-1 make package
```

This will generate a new zip file to our S3 bucket, which you can then load into the lambda functions manually.

## Lambda function setup