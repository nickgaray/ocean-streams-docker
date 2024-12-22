Place certificates into this directory and they will be loaded into the Java
trust store at container startup.

Each certificate must be in PEM format, and have this form:

  -----BEGIN CERTIFICATE-----
  MIIDczCCAlugAwIBAgIBATANBgkqhkiG9w0BAQsFADBbMQswCQYDVQQGEwJVUzEY
  ...
  ... clipped for brevity
  ...
  fOs/QbP1b0s6Xq5vk3aY0vGZnUXEjnI=
  -----END CERTIFICATE-----

The alias of the certificate will be computed from the basename of the file.
So for example, a file called "MyServerCert.pem" will have an alias of
"MyServerCert".