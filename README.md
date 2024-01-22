# OpenSSLTools
Tools and documentation regarding OpenSSL certificates

## Notes for OpenSSL certificates

### Certificate Authority (CA)
The Certificate Authority is a trusted authority that issues certificates.  
Here are the steps to create a CA:  
1. Generate the private key for the CA. The -aes256 flag is optional, but allows us to protect the CA key from being used by letting us create a passkey.
`openssl [-aes256] genrsa -out ca-key.pem 4096`
2. Generate the CA using the CA key.
`openssl req -new -x509 -sha256 -days 3650 -key ca-key.pem -out ca.pem`
  

### Intermediate Certificate
Intermediate certs are generated and signed by the CA, meaning that they will be trusted on any device that trusts the CA. Here are the steps to create a certificate:  
1. Generate the private key for the intermediate cert.
`openssl [-aes256] genrsa -out cert-key.pem 4096`
2. Generate the certificate signing request. This request is for the CA to sign the intermediate cert, and is used in a later step.
`openssl req -new -sha256 -subj "/CN=example.com" -key cert-key.pem -out cert.csr`
3. Create a config file to set the subject's alternative names. See notes below for more information on this.
`echo "subjectAltName=DNS:example.com,DNS:*.example.com,IP:1.2.3.4" >> extfile.conf"`
4. Generate the intermediate certificate, and sign it with the CA including the alt names. The "-CAcreateserial" parameter is optional, and when generating multiple certificates it adds a serial number that increases.
`openssl x509 -req -sha256 -days 3650 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.conf [-CAcreateserial]`
5. That's it! The intermediate certificate should be placed on the *opposing* server that sends the certificate to the server that has the certificate authority on it.
  
  
#### Notes
1. It is good practice to set the common name "/CN" of the certificate to the domain of the server it will be used on.
2. For the subject alt name, it is important to set the DNS name using just the DNS name and the wildcard DNS name (e.g. `example.com` and `*.example.com`).
3. If generating a **FULL CHAIN certificate**, the certificates are added to the fullchain cert from the **bottom up**. In the example above, to create a fullchain certificate, you would add the certificates in this order:
- `cat cert.pem > fullchain.pem`
- `cat ca.pem >> fullchain.pem` (**note**: we are appending here, hence the >>)
  