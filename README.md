# ssl-best-practices

Lab to illuminate SSL best practices. The Lab uses OpenSSL to create keys, create a certificate authority (CA), and sign certificates using the CA.

![Final Environment](https://user-images.githubusercontent.com/3911650/40385835-23345e9c-5dc5-11e8-8d85-cfc8727696a3.png)

## Getting Started

Deploy the CloudFormation `infrastructure/cloudformation.json` template. The template creates a user with the following credentials and minimal required permisisons to complete the Lab:

- Username: _student_
- Password: _password_

## Instructions

1. In the Cloud9 environment terminal, generate an RSA private key:

    ```sh
    openssl genpkey -algorithm RSA -aes-128-cbc -pass pass:Cloud_Academy -out key.pem -pkeyopt rsa_keygen_bits:2048
    ```

1. Restrict file access to the private key:

    ```sh
    chmod 400 key.pem
    ```

1. Copy the contents of `src/req.cnf` to a `req.cnf` file in the Cloud9 environment.

1. Create a certificate signing request (CSR) to have a certificate authority (CA) certify your identity:

    ```sh
    openssl req -new -config csr.cnf -key key.pem -out req.csr
    ```

1. Create a CA and root certificate

    ```sh
    openssl req -key ca.key.pem -new -out ca.csr
    chmod 400 ca.key.pem
    ```
    Fill out the fields (use the same `Organization Name` as used in the CSR)
    ```sh
    sudo sh -c 'openssl rand -hex 16 > /etc/pki/CA/serial' # initialize a random certificate serial number
    sudo touch /etc/pki/CA/index.txt # initialize the index file for tracking
    sudo openssl ca -selfsign -days 7300 -md sha256 -in ca.csr -keyfile ca.key.pem -out ca.crt -extensions v3_ca
    chmod 444 ca.crt
    ```

1. Use the CA to sign the certificate in the CSR created earlier:

    ```sh
    sudo openssl ca -days 100 -notext -md sha256 -batch -in req.csr -keyfile ca/ca.key.pem -cert ca/ca.crt -out cert.pem
    sudo chmod 444 cert.pem
    ```

## Cleaning Up

Delete the CloudFormation stack to remove all the resources used in the Lab.