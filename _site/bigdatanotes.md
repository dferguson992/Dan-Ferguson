1) Generate data, upload to S3, use it for EMRS; generate data, upload to Redshift.
  Use tpch-kit to generate data.  This is the industry standard when benchmarking databases against fake data sets.
  SIDE NOTE: S3 multipart upload automatically happens for files larger than 100MB when utilizing aws s3 cp command and other high-level abstractions.
