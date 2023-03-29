# public_api

this is the final public api document. to update api documentation on https://api.bhuman.ai/
1) update yaml api documentation
2) convert yaml file to openapi.json file with this piece of code
3) replace s3 openapi.json to new one https://s3.console.aws.amazon.com/s3/buckets/api.bhuman.ai?region=us-east-2&tab=objects
4) invalidate cloudfront cache https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-east-2#/distributions/E299G5DD534D6I/invalidations
 
TODO:
make automatic deployments.
