## Troubleshooting TLS issues

### curl

[curl](https://curl.se/)  is a tool for transferring data from or to a server using URLs.

We can download a page across the network, for example with:

```
curl https://example.org/
```

... or stop at just the headers

```
curl -I https://example.org/
```

Adding verbosity lets us see some info about the TLS certificate.

```
curl -vI https://example.org/
```

If we are having certificate issues we can skip the certificate validation.

```
curl -vIk https://example.org/
```

### Listing ciphers with nmap

[Nmap](https://nmap.org/) is an open source tool for network exploration and security auditing.

Nmap includes NSE, the Nmap Scripting Engine, which lets one write [Lua] scripts to automate all sorts of
networking task.  Nmap comes bundled with some useful scripts including one for enumerating tls cypers
for a remote host.

```
$ nmap -sV --script ssl-enum-ciphers -p 443 192.168.5.5
```

### Validating certificate chain with OpenSSL

We can use `openssl` from the command line to connect to a remote host and do some certificate inspection.
The following example is a quick one liner to look at the certificate chain.  We are looking at the stderr
output deliberately.

```
% (echo -n | openssl s_client -connect example.com:443) > /dev/null
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root G2
verify return:1
depth=1 C = US, O = DigiCert Inc, CN = DigiCert Global G2 TLS RSA SHA256 2020 CA1
verify return:1
depth=0 C = US, ST = California, L = Los Angeles, O = Internet\C2\A0Corporation\C2\A0for\C2\A0Assigned\C2\A0Names\C2\A0and\C2\A0Numbers, CN = www.example.org
verify return:1
DONE
```
