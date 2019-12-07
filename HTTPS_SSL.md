# HTTPS / SSL

### Tarea 1: Certificado autofirmado
---------------------------------------------------------------------------
#### Esta práctica la vamos a realizar con un compañero. En un primer momento un alumno creará una Autoridad Certficadora y firmará un certificado para la página del otro alumno. Posteriormente se volverá a realizar la práctica con los roles cambiados.

* #### El alumno que hace de Autoridad Certificadora deberá entregar una documentación donde explique los siguientes puntos:

1. Crear su autoridad certificadora (generar el certificado digital de la CA).

###### Vamos a crear un directorio en *root* para crear el escenario:

~~~
mkdir /root/ca
cd /root/ca
mkdir certs crl newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial
touch openssl.cnf
~~~

###### Ahora vamos a rellenar el fichero *openssl.cnf* con:

~~~
# OpenSSL root CA configuration file.
# Copy to `/root/ca/openssl.cnf`.
 
[ ca ]
# `man ca`
default_ca = CA_default
 
[ CA_default ]
# Directory and file locations.
dir               = /root/ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand
 
# The root key and root certificate.
private_key       = $dir/private/ca.key.pem
certificate       = $dir/certs/ca.cert.pem
 
# For certificate revocation lists.
crlnumber         = $dir/crlnumber
crl               = $dir/crl/ca.crl.pem
crl_extensions    = crl_ext
default_crl_days  = 30
 
# SHA-1 is deprecated, so use SHA-2 instead.
default_md        = sha256
 
name_opt          = ca_default
cert_opt          = ca_default
default_days      = 375
preserve          = no
policy            = policy_strict
 
[ policy_strict ]
# The root CA should only sign intermediate certificates that match.
# See the POLICY FORMAT section of `man ca`.
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
 
[ policy_loose ]
# Allow the intermediate CA to sign a more diverse range of certificates.
# See the POLICY FORMAT section of the `ca` man page.
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
 
[ req ]
# Options for the `req` tool (`man req`).
default_bits        = 2048
distinguished_name  = req_distinguished_name
string_mask         = utf8only
 
# SHA-1 is deprecated, so use SHA-2 instead.
default_md          = sha256
 
# Extension to add when the -x509 option is used.
x509_extensions     = v3_ca
 
[ req_distinguished_name ]
# See <https://en.wikipedia.org/wiki/Certificate_signing_request>.
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address
 
# Optionally, specify some defaults.
countryName_default             = ES
stateOrProvinceName_default     = Sevilla
localityName_default            = Dos Hermanas
0.organizationName_default      = IES Gonzalo Nazareno
organizationalUnitName_default  = IESGN
emailAddress_default            = ale95mogra@gmail.com
 
[ v3_ca ]
# Extensions for a typical CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
 
[ v3_intermediate_ca ]
# Extensions for a typical intermediate CA (`man x509v3_config`).
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
 
[ usr_cert ]
# Extensions for client certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "OpenSSL Generated Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection
 
[ server_cert ]
# Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "OpenSSL Generated Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
 
[ crl_ext ]
# Extension for CRLs (`man x509v3_config`).
authorityKeyIdentifier=keyid:always
 
[ ocsp ]
# Extension for OCSP signing certificates (`man ocsp`).
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
~~~
> Ahora vamos a crear la clave privada para la entidad certificadora:
~~~
cd /root/ca
openssl genrsa -aes256 -out private/ca.key.pem 4096
   Generating RSA private key, 4096 bit long modulus (2 primes)
   ..................................................++++
   .++++
   e is 65537 (0x010001)
   Enter pass phrase for private/ca.key.pem:
   Verifying - Enter pass phrase for private/ca.key.pem:

chmod 400 private/ca.key.pem
~~~

###### Ahora creamos el certificado raíz de la entidad certificadora:

~~~
cd /root/ca
openssl req -config openssl.cnf \
      -key private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out certs/ca.cert.pem
~~~

###### Ahora nos pedirá que rellenemos los metadatos de nuevo, pero si le damos a espacio a cada uno de ellos, este nos añadirá los metadatos por defecto que le hemos indicado en el fichero *openssl.cnf*

###### Ahora vamos a cambiarle los permisos a nuestro certificado:

~~~
chmod 444 certs/ca.cert.pem
~~~

###### Ahora vamos a verificar  el certificado con:

~~~
openssl x509 -noout -text -in certs/ca.cert.pem
~~~
###### Salida al comando anterior:
~~~
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3e:dd:f2:6d:99:6c:cc:9d:8d:e6:73:a2:99:c5:ab:64:b8:1d:70:68
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = ES, ST = Sevilla, L = Dos Hermanas, O = IES Gonzalo Nazareno, OU = IESGN, emailAddress = ale95mogra@gmail.com
        Validity
            Not Before: Nov  4 11:20:56 2019 GMT
            Not After : Oct 30 11:20:56 2039 GMT
        Subject: C = ES, ST = Sevilla, L = Dos Hermanas, O = IES Gonzalo Nazareno, OU = IESGN, emailAddress = ale95mogra@gmail.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (4096 bit)
                Modulus:
                    00:b8:f2:15:d7:4d:be:c3:b8:98:a1:ea:60:87:30:
                    42:d8:a6:1c:4e:c8:5a:e3:7c:38:ac:b1:c0:85:7d:
                    8b:a2:b9:b4:e3:dc:27:ee:40:b1:f5:d6:6d:ab:dc:
                    ad:34:6b:08:19:3d:d8:be:80:1a:75:5e:e4:81:cf:
                    87:52:4a:5a:66:2e:16:96:17:8e:cd:73:85:96:39:
                    22:14:e9:b1:47:d0:bb:45:cb:c7:11:c8:f4:b0:83:
                    09:4e:7a:5c:80:c5:be:1d:ba:3d:50:fd:ba:51:9e:
                    60:2a:29:3a:30:99:4c:a7:c8:bd:eb:2c:27:28:96:
                    42:db:f0:68:ff:cf:6e:e0:5f:45:86:00:a2:ae:a8:
                    9a:02:dd:bb:db:ea:67:6f:24:c8:cb:f6:0a:e8:47:
                    16:c4:b7:e4:e4:da:7a:b9:d4:17:5b:bc:67:9f:14:
                    22:4d:24:9a:a8:b0:37:d8:07:be:99:9f:c1:26:e0:
                    ac:c6:9a:7b:0f:5d:fc:7e:82:22:a4:c7:c9:2d:15:
                    41:fe:a3:f1:d0:55:b1:df:0e:a6:13:33:87:bd:81:
                    fa:0a:d6:db:80:49:34:77:2e:08:d5:5b:d2:28:b1:
                    ae:6c:cc:76:e4:ef:8b:43:e2:da:cd:1b:40:69:e5:
                    8a:a6:23:03:8c:c5:22:5f:16:10:29:79:42:cb:d3:
                    47:f9:52:11:95:62:b5:f7:f1:a6:40:97:03:b9:d8:
                    1b:56:1b:ff:08:97:03:9a:70:6e:7c:67:04:54:b7:
                    4b:47:03:d8:f0:22:66:02:e6:13:ab:fb:8b:f1:a5:
                    3e:a4:33:b4:2e:5d:d0:ec:30:94:73:bc:e2:89:db:
                    05:33:62:58:98:c4:67:38:74:ce:b2:a1:a0:ab:a2:
                    31:be:35:79:db:57:9d:c9:4c:28:62:4f:9e:cc:a7:
                    d0:ba:4b:6e:6a:fb:39:c4:ff:57:30:8d:59:5c:59:
                    8f:fb:4d:a6:ec:a7:8e:a3:a2:d9:38:94:f6:b4:8c:
                    c9:d3:d0:2b:87:b8:36:98:df:6c:64:f7:d8:8d:4b:
                    52:b5:6b:fe:ef:ad:00:4a:7b:16:a6:51:63:8b:39:
                    ea:41:89:43:0e:50:6e:e9:ec:74:9b:8a:6f:48:0c:
                    17:70:ef:ab:ed:90:ab:1a:b3:60:7c:6b:6f:d6:05:
                    a4:0e:22:ae:08:e0:d0:a9:79:d8:31:92:18:7e:43:
                    49:31:50:7b:56:16:86:63:08:52:6b:8c:3e:7e:af:
                    5e:66:f7:f2:23:61:a2:b4:00:ad:b4:6a:7f:a4:0f:
                    ac:f0:00:4e:88:c0:b3:7b:4a:74:97:e3:e3:5c:9a:
                    b3:1a:ef:eb:37:62:72:b5:af:68:17:a6:44:4d:1e:
                    d7:02:f1
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                07:76:43:88:1C:E0:55:01:03:C9:19:BF:8C:A9:52:C6:25:55:5A:AF
            X509v3 Authority Key Identifier: 
                keyid:07:76:43:88:1C:E0:55:01:03:C9:19:BF:8C:A9:52:C6:25:55:5A:AF

            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
    Signature Algorithm: sha256WithRSAEncryption
         89:d8:00:aa:7b:a9:91:3f:09:e3:ce:43:fe:d0:16:0b:57:78:
         37:eb:49:a2:3b:f1:5b:c5:b5:4b:e7:69:55:98:84:be:de:cf:
         89:71:5d:15:71:a6:a6:8a:1d:1c:79:7b:35:4e:5a:ee:74:ca:
         5e:ec:d6:bb:ea:d8:51:fe:4e:ed:06:9a:69:0f:6a:b3:bc:63:
         d3:a5:45:86:ad:88:af:5c:e0:29:a8:1a:78:57:18:b1:a9:e1:
         50:be:36:21:35:93:f7:17:73:81:c7:90:f7:12:d0:46:d0:bc:
         6d:09:0c:de:22:95:88:6a:d5:08:0c:f6:a2:ae:88:b4:fb:99:
         9f:77:07:d2:ce:15:46:b5:49:a7:c9:6c:8e:92:e0:75:08:16:
         f6:77:73:ee:f6:79:3b:64:16:c0:e1:14:95:2a:6c:1f:4b:79:
         22:94:20:a2:18:73:4f:b3:b1:ae:ac:b2:59:09:dd:a6:1b:68:
         19:0b:ba:ae:85:57:67:6c:99:03:5f:62:4f:d7:10:3d:80:93:
         4d:13:21:e7:31:a9:02:eb:d4:df:d7:cf:3d:9b:05:b8:f8:3c:
         0e:8c:e8:48:b6:cc:92:8e:a9:83:b2:5e:32:89:01:55:33:60:
         88:7b:39:4f:34:8f:c4:a2:9e:ce:7c:d1:9a:3b:f5:73:8e:7c:
         9f:cc:f7:82:c5:e4:93:45:95:b5:0e:3c:58:ec:51:60:cd:cb:
         97:4c:e7:fd:ae:84:80:cf:5f:e9:4f:5d:da:fe:15:f5:3b:71:
         e9:6d:e5:4f:a3:0d:23:05:ae:ec:ce:49:cc:b9:e8:db:56:01:
         9e:98:a9:b6:8a:fb:82:52:0e:f8:5d:09:7a:f3:97:9e:38:e3:
         b8:6b:6c:f4:98:8b:4a:90:9e:c2:d4:cf:23:95:34:0d:62:3f:
         6f:78:75:91:8c:20:c0:d9:0d:bb:52:06:25:75:0f:44:21:25:
         cd:bd:34:2b:f4:86:69:fd:a4:97:e6:3d:2c:a0:3c:0e:02:10:
         f5:8d:e2:c0:b5:24:12:0c:3a:f7:5c:80:e9:05:c7:b4:02:75:
         47:05:76:43:48:f0:14:67:f7:8b:ff:89:d0:c6:4a:3e:2e:7e:
         e6:56:a0:f7:8d:dc:e7:87:47:2a:8d:db:64:09:5a:6a:94:db:
         38:48:ff:e9:9c:12:22:5d:33:06:0b:cd:a4:0d:32:41:b9:04:
         22:33:fd:06:08:e0:3d:fc:a6:c0:fd:45:93:05:c0:63:cf:3f:
         5e:07:22:45:eb:a2:c4:c2:87:cc:99:22:75:07:ce:fb:ca:15:
         e2:40:69:a8:43:97:23:9f:c7:cf:cc:37:3c:b1:9a:85:40:e3:
         81:3f:51:44:b8:f8:ef:22
~~~

2. Debe recibir el fichero CSR (Solicitud de Firmar un Certificado) de su compañero, debe firmarlo y enviar el certificado generado a su compañero.


###### Tenemos el certificado de nuestra compañera y lo firmamos con:

~~~
openssl x509 -req -in paloma.iesgn.org.csr -CA certs/ca.cert.pem -CAkey private/ca.key.pem -CAcreateserial -out paloma.iesgn.org-firmado.crt
   Signature ok
   subject=C = sp, ST = Seville, L = Dos Hermanas, O = paloma, CN = paloma.iesgn.org
   Getting CA Private Key
   Enter pass phrase for private/ca.key.pem:
~~~

3. ¿Qué otra información debe aportar a tu compañero para que éste configure de forma adecuada su servidor web con el certificado generado?


###### Tenemos que darle el certificado *ca.cert.pem* para que lo importe a su navegador web y que le permita una navegación segura.

* #### El alumno que hace de administrador del servidor web, debe entregar una documentación que describa los siguientes puntos:

1. Crea una clave privada RSA de 4096 bits para identificar el servidor.
   
###### Creamos la clave privada con openssl

~~~
sudo openssl genrsa -out alejandro.iesgn.org.key 4096
   Generating RSA private key, 4096 bit long modulus (2 primes)
   ........................................................................................................................++++
   ...........++++
   e is 65537 (0x010001)
~~~

2. Utiliza la clave anterior para generar un CSR, considerando que deseas acceder al servidor tanto con el FQDN (tunombre.iesgn.org) como con el nombre de host (implica el uso de las extensiones Alt Name).

###### Creamos un fichero de configuración, *alejandro.iesgn.org.conf*, y le añadimos lo siguiente:

~~~
[ req ]
default_bits       = 4096
default_keyfile    = alejandro.iesgn.org.key
distinguished_name = req_distinguished_name
req_extensions     = req_ext

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = SP
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Seville
localityName                = Locality Name (eg, city)
localityName_default        = Dos Hermanas
organizationName            = Organization Name (eg, company)
organizationName_default    = Alejandro
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64
commonName_default          = alejandro.iesgn.org

[ req_ext ]
subjectAltName = @alt_names

[alt_names]
DNS.1   = servidor
~~~

###### Ahora creamos el certificado llamando al fichero de configuración anterior y cuando nos pregunte los metadatos presionamos *ENTER*:

~~~
sudo openssl req -new -nodes -sha256 -config alejandro.iesgn.org.conf -out alejandro.iesgn.org.csr
  Generating a RSA private key
................................................................................................................................................................................................................................................++++
............................................................................................++++
   writing new private key to 'alejandro.iesgn.org.key'
   -----
   You are about to be asked to enter information that will be incorporated
   into your certificate request.
   What you are about to enter is what is called a Distinguished Name or a DN.
   There are quite a few fields but you can leave some blank
   For some fields there will be a default value,
   If you enter '.', the field will be left blank.
   -----
   Country Name (2 letter code) [SP]:
   State or Province Name (full name) [Seville]:
   Locality Name (eg, city) [Dos Hermanas]:
   Organization Name (eg, company) [Alejandro]:
   Common Name (e.g. server FQDN or YOUR name) [alejandro.iesgn.org]:
~~~

3. Envía la solicitud de firma a la entidad certificadora (su compañero).

###### Enviamos el certificado, *alejandro.iesgn.org.crt*, a nuestra compañera que realiza el papel de Autoridad Certificadora

###### Prueba de Metadatos del certificado que le vamos a enviar al nuestra compañera

![Tarea2.1](Certs.png)

4. Recibe como respuesta un certificado X.509 para el servidor firmado y el certificado de la autoridad certificadora.

###### Hemos recibido el certificado firmado por nuestra compañera *alejandro.iesgn.org-firmado.crt*

5. Configura tu servidor web con https en el puerto 443, haciendo que las peticiones http se redireccionen a https (forzar https).

###### Creamos el fichero parejo de */etc/apache2/sites-available/alejandro.conf* copiando el default de apache:

~~~
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/alejandro-ssl.conf
~~~

###### Y dentro tenemos que añadir las siguientes lineas:

~~~
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin alejandro@localhost
                ServerName alejandro.iesgn.org
                DocumentRoot /srv/www/html

                SSLEngine on

                SSLCertificateFile /etc/ssl/certs/alejandro.iesgn.org-firmado.crt
                SSLCertificateKeyFile /etc/ssl/private/alejandro.iesgn.org.key
                SSLCertificateChainFile /etc/ssl/certs/alejandro.iesgn.org.csr

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

        </VirtualHost>
</IfModule>
~~~

###### Activamos el enlace simbólico y activamos el modulo de *ssl*:

~~~
sudo a2ensite alejandro-ssl
sudo a2enmod ssl
~~~

###### Ahora tenemos que mover los fichero a las direcciones indicadas en el *.conf*

~~~
sudo mv alejandro.iesgn.org-firmado.crt /etc/ssl/certs
sudo mv alejandro.iesgn.org.key  /etc/ssl/private
sudo mv alejandro.iesgn.org.csr /etc/ssl/certs
~~~

###### Ahora vamos a realizar la redirección, para esto, tenemos que añadir las siguientes lineas al fichero *alejandro.conf* que es el del puerto 80:

~~~
ServerName alejandro.iesgn.org
Redirect / https://alejandro.iesgn.org
~~~

###### Ahora comprobamos que el *http* es redireccionado a *https* y nos muestra un página de advertencias con el mensaje: *Advertencia: riesgo potencial de seguridad a continuación*

![Tarea2.2](Certs.png)

###### Tenemos que importar el certificado de la Autoridad certificadora en:

###### *Preferencias*>*Privacidad y Seguridad*>*Ver Certificados*>*Autoridades*>*Importar*

![Tarea2.3](Certs.png)
![Tarea2.4](Certs.png)

###### Ahora nos saldrá el candado de conexión segura.

![Tarea2.5](Certs.png)