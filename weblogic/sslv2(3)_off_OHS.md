# Отключение протоколов SSLv2(3) на OHS

# Oracle HTTP Server

порты 4443, 9999, 6701 

Документация:

![How to Change SSL Protocols (to Disable SSL 2.0/3.0) in Oracle Fusion Middleware Products (Doc ID 1936300.1) OHS](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=513469235011288&id=1936300.1&_afrWindowMode=0&_adf.ctrl-state=e3x5clyql_200#aref_section26)

Configuring SSL in Oracle Fusion Middleware
[[https://docs.oracle.com/cd/E28280_01/core.1111/e10105/sslconfig.htm#ASADM1800]]

New SSL Protocol and Cipher Options for Oracle Fusion Middleware's OPMN/ONS Component (Doc ID 1905314.1)
[[https://support.oracle.com/epmos/faces/DocumentDisplay?parent=DOCUMENT&sourceId=1936300.1&id=1905314.1]]

Patch 18423801: OPMN SECURITY PATCH UPDATE 11.1.1.7.0 (CPUJUL2014)
[[https://support.oracle.com/epmos/faces/ui/patch/PatchDetail.jspx?parent=DOCUMENT&sourceId=1905314.1&patchId=18423801]]

Настройки находятся в трех файлах
<pre>
/db1/app/oracle/Middleware/Oracle_WT1/instances/instance1/config/OHS/ohs1
     ssl.conf
     admin.conf
/u/app/oracle/Middleware/Oracle_WT1/instances/instance1/config/OPMN/opmn/opmn.xml
</pre>

*Проверить какие протоколы работают:*

$ openssl s_client -connect baraka:4443 -ssl2
<pre>
CONNECTED(00000003)
139929607427912:error:1407F0E5:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 14 bytes and written 48 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326193
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:4443 -ssl3
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify return:1
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICejCCAeMCEFdlXCvnQO50bxncfLSl+XEwDQYJKoZIhvcNAQEEBQAwfTELMAkG
A1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1SRURXT09EU0hPUkVTMQ8w
DQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgGA1UEAxMhU2VsZi1TaWdu
ZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMCAXDTE2MDIxMDExMzUwOVoYDzIwNjYw
MTI4MTEzNTA5WjB9MQswCQYDVQQGEwJVUzELMAkGA1UECBMCQ0ExFjAUBgNVBAcT
DVJFRFdPT0RTSE9SRVMxDzANBgNVBAoTBk9SQUNMRTEMMAoGA1UECxMDT0FTMSow
KAYDVQQDEyFTZWxmLVNpZ25lZCBDZXJ0aWZpY2F0ZSBmb3Igb2hzMSAwgZ8wDQYJ
KoZIhvcNAQEBBQADgY0AMIGJAoGBALLZLr0ePfA7LjMjYnTAqRPG6u3P2Z+KX9Ct
ckS8pC2Ma8vGXF3YguesVA/da0rsiDfL5AEWpj4sElTsefLZSv636GEDhP2wbmAA
AGh0SSqvshaC2DV0+6fIFZIDVM9DbqT+yboNtsxCTJSQhcgk6B5ApPsUKwUkYEjM
iXLWM0iVAgMBAAEwDQYJKoZIhvcNAQEEBQADgYEANHyOPHb2Y895s8PY74ZavDvO
/5yCtMBAuaNIhaEzpoRh4tKHsPSPKFYLBIAguKlRDVuup8fD8qVCKOhYJASfSJn7
ogRPVF5W5YoXS1kL8BOhpQWQD/Nx4Zyg4zHN+/N3ibGb5h0jd8UlGYDSGrWJqmtM
Rt6F1ZyUjWNqRzr51kQ=
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
No client certificate CA names sent
---
SSL handshake has read 807 bytes and written 336 bytes
---
New, TLSv1/SSLv3, Cipher is DES-CBC3-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : DES-CBC3-SHA
    Session-ID: 7B155568FD79C6339F571921B94E6808
    Session-ID-ctx:
    Master-Key: F02BCD5C588E7D4FBA3C000DAE70FF876F21DE4A4AE6FBF66EE749AB7E32ABCA2F5887FB488BEA9C86277A1E92CD6151
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326200
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:9999 -ssl2
<pre>
CONNECTED(00000003)
140476139542344:error:1407F0E5:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 14 bytes and written 48 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326212
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:9999 -ssl3
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify return:1
140424221013832:error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate:s3_pkt.c:1257:SSL alert number 42
140424221013832:error:1409E0E5:SSL routines:SSL3_WRITE_BYTES:ssl handshake failure:s3_pkt.c:596:
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICeDCCAeECEAE2FSWh5/PHGvfxJI8VaX0wDQYJKoZIhvcNAQEEBQAwfTELMAkG
A1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1SRURXT09EU0hPUkVTMQ8w
DQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgGA1UEAxMhU2VsZi1TaWdu
ZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMB4XDTE2MDIxMDExMzUxMVoXDTQxMDIw
MzExMzUxMVowfTELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1S
RURXT09EU0hPUkVTMQ8wDQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgG
A1UEAxMhU2VsZi1TaWduZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMIGfMA0GCSqG
SIb3DQEBAQUAA4GNADCBiQKBgQC+aivnTuzCWHZsoNnbq9SeaElUyt6tdtAPVbdL
xu7rIFi70wv95aKFaVEhekYkg90H7/IgwghPwuLIZERDxblZAnSBZjxs582GQob3
lbvZDAxpOMYNZun6S1hLLPHGhemU/VZq1MEQ7HiledFOj93XgZ7xgWpAAbYQJHWY
2j3Y5wIDAQABMA0GCSqGSIb3DQEBBAUAA4GBAJu/b9u78g6Tl97hThlzzeoGUevF
ev7eTQAu3iZQO7QtJN+3xk6ARjmyjoZQakC3IX88grNnKpwH0yw9fsVQMfnWoL9S
BCWa5YGnBMmRBQT5bH6tnWN1lj6krvW0D9vr37IT1riK2sVXUPstgDVPq3XNZcK4
hED82YE5PHvILmI0
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Acceptable client certificate CA names
/C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
/C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
/C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
SSL handshake has read 1297 bytes and written 215 bytes
---
New, TLSv1/SSLv3, Cipher is RC4-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : RC4-SHA
    Session-ID: 29AD78DE4B77645AF98455E826AA3492
    Session-ID-ctx:
    Master-Key: 9FE58BA6407A02C7940C0D024CDABB18CAEF54B4AE1AF0360761D75BA8848885533389C4ADC11ED0AF9F9EB8BBFA72C8
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326215
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:6701 -ssl2
<pre>
CONNECTED(00000003)
139935606724424:error:1407F0E5:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 14 bytes and written 48 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326230
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:6701 -ssl3
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for instance1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for instance1 "
verify return:1
139844087342920:error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate:s3_pkt.c:1257:SSL alert number 42
139844087342920:error:1409E0E5:SSL routines:SSL3_WRITE_BYTES:ssl handshake failure:s3_pkt.c:596:
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIChjCCAe8CEAkbnczD0kYSogu4WsUy7C8wDQYJKoZIhvcNAQEEBQAwgYIxCzAJ
BgNVBAYTAlVTMQswCQYDVQQIEwJDQTEWMBQGA1UEBxMNUkVEV09PRFNIT1JFUzEP
MA0GA1UEChMGT1JBQ0xFMQwwCgYDVQQLEwNPQVMxLzAtBgNVBAMTJlNlbGYtU2ln
bmVkIENlcnRpZmljYXRlIGZvciBpbnN0YW5jZTEgMCAXDTE2MDIxMDExMzUwM1oY
DzIwNjYwMTI4MTEzNTAzWjCBgjELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYw
FAYDVQQHEw1SRURXT09EU0hPUkVTMQ8wDQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsT
A09BUzEvMC0GA1UEAxMmU2VsZi1TaWduZWQgQ2VydGlmaWNhdGUgZm9yIGluc3Rh
bmNlMSAwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMO3kgVhc1NH9ryoqPuN
h5pOZ54MGoDizDglEDHPwtnWsmPBMDG1ch+P1FDNxu1VbwldrYLXC/7uO9qJgtmD
GGyV9CHAmRJUeLSu1wfFPnLe1EFPlPVKsM0TVOCYUsS0pbgdFxFvdugIize5s2zO
KuZwOTFSsCLIMJU+ctC+XCorAgMBAAEwDQYJKoZIhvcNAQEEBQADgYEAWFHzJl/y
InbZ+u3Lg8UwVz3Vd5CYSpK7OCN8ktUfwNJL3ppYUiMCc9bjxvUaW+j8rSY5kmXf
WNQvwmKl/DqgED+K8DwPNN9eOErhoZzL/5Q0k8EZ59gAwf8N4cVdG8RTkMVPbVBq
BlejNufQxKWZ9hBPM5Wr6SItI9SBeZ8eHbg=
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
Acceptable client certificate CA names
/C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
/C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
/C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
SSL handshake has read 1301 bytes and written 219 bytes
---
New, TLSv1/SSLv3, Cipher is DES-CBC3-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : DES-CBC3-SHA
    Session-ID:
    Session-ID-ctx:
    Master-Key: DABF92EFC9C8C4D6D4C3BAF7D46CB5C239C1AAF76AD9DF19619F369A2B17C76573A0559A0DD6C61EB416254E6BE4D0CF
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326233
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:4443 -tls1
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify return:1
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICejCCAeMCEFdlXCvnQO50bxncfLSl+XEwDQYJKoZIhvcNAQEEBQAwfTELMAkG
A1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1SRURXT09EU0hPUkVTMQ8w
DQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgGA1UEAxMhU2VsZi1TaWdu
ZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMCAXDTE2MDIxMDExMzUwOVoYDzIwNjYw
MTI4MTEzNTA5WjB9MQswCQYDVQQGEwJVUzELMAkGA1UECBMCQ0ExFjAUBgNVBAcT
DVJFRFdPT0RTSE9SRVMxDzANBgNVBAoTBk9SQUNMRTEMMAoGA1UECxMDT0FTMSow
KAYDVQQDEyFTZWxmLVNpZ25lZCBDZXJ0aWZpY2F0ZSBmb3Igb2hzMSAwgZ8wDQYJ
KoZIhvcNAQEBBQADgY0AMIGJAoGBALLZLr0ePfA7LjMjYnTAqRPG6u3P2Z+KX9Ct
ckS8pC2Ma8vGXF3YguesVA/da0rsiDfL5AEWpj4sElTsefLZSv636GEDhP2wbmAA
AGh0SSqvshaC2DV0+6fIFZIDVM9DbqT+yboNtsxCTJSQhcgk6B5ApPsUKwUkYEjM
iXLWM0iVAgMBAAEwDQYJKoZIhvcNAQEEBQADgYEANHyOPHb2Y895s8PY74ZavDvO
/5yCtMBAuaNIhaEzpoRh4tKHsPSPKFYLBIAguKlRDVuup8fD8qVCKOhYJASfSJn7
ogRPVF5W5YoXS1kL8BOhpQWQD/Nx4Zyg4zHN+/N3ibGb5h0jd8UlGYDSGrWJqmtM
Rt6F1ZyUjWNqRzr51kQ=
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
No client certificate CA names sent
---
SSL handshake has read 783 bytes and written 345 bytes
---
New, TLSv1/SSLv3, Cipher is DES-CBC3-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : DES-CBC3-SHA
    Session-ID: 2C1E1E4A2BE5901918B7953E2D8FD3F6
    Session-ID-ctx:
    Master-Key: D2CED9393BC010BAFBDF107D1CD96DEC026DFF99989FFC8ACA7F6AD143DB72CB27883432525AE25B141EB2661DB0CED2
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326250
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:9999 -tls1
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify return:1
140219572180808:error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate:s3_pkt.c:1257:SSL alert number 42
140219572180808:error:1409E0E5:SSL routines:SSL3_WRITE_BYTES:ssl handshake failure:s3_pkt.c:596:
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICeDCCAeECEAE2FSWh5/PHGvfxJI8VaX0wDQYJKoZIhvcNAQEEBQAwfTELMAkG
A1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1SRURXT09EU0hPUkVTMQ8w
DQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgGA1UEAxMhU2VsZi1TaWdu
ZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMB4XDTE2MDIxMDExMzUxMVoXDTQxMDIw
MzExMzUxMVowfTELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1S
RURXT09EU0hPUkVTMQ8wDQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgG
A1UEAxMhU2VsZi1TaWduZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMIGfMA0GCSqG
SIb3DQEBAQUAA4GNADCBiQKBgQC+aivnTuzCWHZsoNnbq9SeaElUyt6tdtAPVbdL
xu7rIFi70wv95aKFaVEhekYkg90H7/IgwghPwuLIZERDxblZAnSBZjxs582GQob3
lbvZDAxpOMYNZun6S1hLLPHGhemU/VZq1MEQ7HiledFOj93XgZ7xgWpAAbYQJHWY
2j3Y5wIDAQABMA0GCSqGSIb3DQEBBAUAA4GBAJu/b9u78g6Tl97hThlzzeoGUevF
ev7eTQAu3iZQO7QtJN+3xk6ARjmyjoZQakC3IX88grNnKpwH0yw9fsVQMfnWoL9S
BCWa5YGnBMmRBQT5bH6tnWN1lj6krvW0D9vr37IT1riK2sVXUPstgDVPq3XNZcK4
hED82YE5PHvILmI0
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Acceptable client certificate CA names
/C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
/C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
/C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
SSL handshake has read 1297 bytes and written 198 bytes
---
New, TLSv1/SSLv3, Cipher is RC4-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : RC4-SHA
    Session-ID: 3F202BF274EBEBD85540038BD19D9EB1
    Session-ID-ctx:
    Master-Key: 4E8FCE5C6287F087DEEE91A03ED7EB863D95A97E7326248C8DB917F93FAA911C12B233217D938BCD54D39AF64B349589
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326261
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:6701 -tls1
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for instance1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for instance1 "
verify return:1
139757470390088:error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate:s3_pkt.c:1257:SSL alert number 42
139757470390088:error:1409E0E5:SSL routines:SSL3_WRITE_BYTES:ssl handshake failure:s3_pkt.c:596:
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIChjCCAe8CEAkbnczD0kYSogu4WsUy7C8wDQYJKoZIhvcNAQEEBQAwgYIxCzAJ
BgNVBAYTAlVTMQswCQYDVQQIEwJDQTEWMBQGA1UEBxMNUkVEV09PRFNIT1JFUzEP
MA0GA1UEChMGT1JBQ0xFMQwwCgYDVQQLEwNPQVMxLzAtBgNVBAMTJlNlbGYtU2ln
bmVkIENlcnRpZmljYXRlIGZvciBpbnN0YW5jZTEgMCAXDTE2MDIxMDExMzUwM1oY
DzIwNjYwMTI4MTEzNTAzWjCBgjELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYw
FAYDVQQHEw1SRURXT09EU0hPUkVTMQ8wDQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsT
A09BUzEvMC0GA1UEAxMmU2VsZi1TaWduZWQgQ2VydGlmaWNhdGUgZm9yIGluc3Rh
bmNlMSAwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMO3kgVhc1NH9ryoqPuN
h5pOZ54MGoDizDglEDHPwtnWsmPBMDG1ch+P1FDNxu1VbwldrYLXC/7uO9qJgtmD
GGyV9CHAmRJUeLSu1wfFPnLe1EFPlPVKsM0TVOCYUsS0pbgdFxFvdugIize5s2zO
KuZwOTFSsCLIMJU+ctC+XCorAgMBAAEwDQYJKoZIhvcNAQEEBQADgYEAWFHzJl/y
InbZ+u3Lg8UwVz3Vd5CYSpK7OCN8ktUfwNJL3ppYUiMCc9bjxvUaW+j8rSY5kmXf
WNQvwmKl/DqgED+K8DwPNN9eOErhoZzL/5Q0k8EZ59gAwf8N4cVdG8RTkMVPbVBq
BlejNufQxKWZ9hBPM5Wr6SItI9SBeZ8eHbg=
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
Acceptable client certificate CA names
/C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
/C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
/C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
SSL handshake has read 1301 bytes and written 210 bytes
---
New, TLSv1/SSLv3, Cipher is AES256-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : AES256-SHA
    Session-ID:
    Session-ID-ctx:
    Master-Key: A2E269A9CB4680D91EEF4C11EF191454230F48E5B24BD086663988810E54DA9107194C2C4960179C0D4D27F5A24FD35B
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539326272
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>

Из запрещенных работает SSLv3, SSLv2 не работает. TLSv1 работает.

h2. Отключение SSLv2(3) для OHS

В файлах
/db1/app/oracle/Middleware/Oracle_WT1/instances/instance1/config/OHS/ohs1
     ssl.conf
     admin.conf

заменить
<pre>
SSLProtocol nzos_Version_1_0 nzos_Version_3_0
</pre>
на
<pre>
SSLProtocol All -SSLv2 -SSLv3
</pre>

h2. Установка патча на OPMN и отключение SSLv2(3)

Остановить OHS и установить патч.
<pre>
$ export ORACLE_HOME=/u/app/oracle/Middleware/Oracle_WT1
$ pwd
/u/app/oracle/distr/OPMN/18423801
</pre>

$ /u/app/oracle/Middleware/Oracle_WT1/OPatch/opatch lsinv
<pre>
Oracle Interim Patch Installer version 11.1.0.9.9
Copyright (c) 2012, Oracle Corporation.  All rights reserved.


Oracle Home       : /u/app/oracle/Middleware/Oracle_WT1
Central Inventory : /u/app/oraInventory
   from           : /u/app/oracle/Middleware/Oracle_WT1/oraInst.loc
OPatch version    : 11.1.0.9.9
OUI version       : 11.1.0.9.0
Log file location : /u/app/oracle/Middleware/Oracle_WT1/cfgtoollogs/opatch/opatch2018-10-12_15-28-16PM_1.log


OPatch detects the Middleware Home as "/db1/app/oracle/Middleware"

Lsinventory Output file location : /u/app/oracle/Middleware/Oracle_WT1/cfgtoollogs/opatch/lsinv/lsinventory2018-10-12_15-28-16PM.txt

--------------------------------------------------------------------------------
Installed Top-level Products (1):

Oracle WebTier and Utilities CD                                      11.1.1.7.0
There are 1 products installed in this Oracle Home.


Interim patches (7) :

Patch  7663342      : applied on Wed Feb 10 17:31:06 NOVT 2016
   Created on 15 Jan 2009, 00:17:30 hrs PST8PDT
   Bugs fixed:
     7663342

Patch  6750400      : applied on Wed Feb 10 17:30:56 NOVT 2016
   Created on 3 Nov 2008, 22:33:54 hrs PST8PDT
   Bugs fixed:
     6750400

Patch  6845838      : applied on Wed Feb 10 17:30:39 NOVT 2016
   Created on 3 Nov 2008, 22:00:04 hrs PST8PDT
   Bugs fixed:
     6845838

Patch  7707476      : applied on Wed Feb 10 17:30:15 NOVT 2016
   Created on 10 Feb 2009, 19:13:18 hrs PST8PDT
   Bugs fixed:
     7707476, 7360273, 7284982

Patch  7427144      : applied on Wed Feb 10 17:29:59 NOVT 2016
   Created on 29 Oct 2008, 00:14:14 hrs PST8PDT
   Bugs fixed:
     7427144

Patch  7393921      : applied on Wed Feb 10 17:29:51 NOVT 2016
   Created on 17 Oct 2008, 03:32:19 hrs PST8PDT
   Bugs fixed:
     7393921

Patch  6599470      : applied on Wed Feb 10 17:29:35 NOVT 2016
   Created on 21 Jan 2009, 01:50:17 hrs PST8PDT
   Bugs fixed:
     6599470



--------------------------------------------------------------------------------

OPatch succeeded.
</pre>
$ /u/app/oracle/Middleware/Oracle_WT1/OPatch/opatch apply
<pre>
Oracle Interim Patch Installer version 11.1.0.9.9
Copyright (c) 2012, Oracle Corporation.  All rights reserved.


Oracle Home       : /u/app/oracle/Middleware/Oracle_WT1
Central Inventory : /u/app/oraInventory
   from           : /u/app/oracle/Middleware/Oracle_WT1/oraInst.loc
OPatch version    : 11.1.0.9.9
OUI version       : 11.1.0.9.0
Log file location : /u/app/oracle/Middleware/Oracle_WT1/cfgtoollogs/opatch/18423801_Oct_12_2018_15_28_22/apply2018-10-12_15-28-22PM_1.log


OPatch detects the Middleware Home as "/db1/app/oracle/Middleware"

Applying interim patch '18423801' to OH '/u/app/oracle/Middleware/Oracle_WT1'
Verifying environment and performing prerequisite checks...
All checks passed.

Please shutdown Oracle instances running out of this ORACLE_HOME on the local system.
(Oracle Home = '/u/app/oracle/Middleware/Oracle_WT1')


Is the local system ready for patching? [y|n]
y
User Responded with: Y
Backing up files...

Patching component oracle.opmn, 11.1.1.7.0...

Verifying the update...
Patch 18423801 successfully applied
Log file location: /u/app/oracle/Middleware/Oracle_WT1/cfgtoollogs/opatch/18423801_Oct_12_2018_15_28_22/apply2018-10-12_15-28-22PM_1.log

OPatch succeeded.
</pre>
$ /u/app/oracle/Middleware/Oracle_WT1/OPatch/opatch lsinv
<pre>
...
Interim patches (8) :

Patch  18423801     : applied on Fri Oct 12 15:28:32 NOVT 2018
Unique Patch ID:  17563461
   Created on 5 May 2014, 23:57:45 hrs UTC
   Bugs fixed:
     17988318
...
</pre>

В файле
<pre>
/u/app/oracle/Middleware/Oracle_WT1/instances/instance1/config/OPMN/opmn/opmn.xml
</pre>
заменить:
<pre>
<ssl enabled="true" wallet-file="/u/app/oracle/Middleware/Oracle_WT1/instances/instance1/config/OPMN/opmn/wallet"/>
</pre>
на
<pre>
<ssl enabled="true" wallet-file="/oracle/software/fmw/wallet" ssl-versions="TLSv1.0" ssl-ciphers="SSL_RSA_WITH_AES_128_CBC_SHA,SSL_RSA_WITH_AES_256_CBC_SHA"/>
</pre>

Стартовать OHS

Проверить протоколы на портах 6701, 9999, 4443

$ openssl s_client -connect baraka:6701 -tls1
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for instance1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for instance1 "
verify return:1
140354877142856:error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate:s3_pkt.c:1257:SSL alert number 42
140354877142856:error:1409E0E5:SSL routines:SSL3_WRITE_BYTES:ssl handshake failure:s3_pkt.c:596:
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIChjCCAe8CEAkbnczD0kYSogu4WsUy7C8wDQYJKoZIhvcNAQEEBQAwgYIxCzAJ
BgNVBAYTAlVTMQswCQYDVQQIEwJDQTEWMBQGA1UEBxMNUkVEV09PRFNIT1JFUzEP
MA0GA1UEChMGT1JBQ0xFMQwwCgYDVQQLEwNPQVMxLzAtBgNVBAMTJlNlbGYtU2ln
bmVkIENlcnRpZmljYXRlIGZvciBpbnN0YW5jZTEgMCAXDTE2MDIxMDExMzUwM1oY
DzIwNjYwMTI4MTEzNTAzWjCBgjELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYw
FAYDVQQHEw1SRURXT09EU0hPUkVTMQ8wDQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsT
A09BUzEvMC0GA1UEAxMmU2VsZi1TaWduZWQgQ2VydGlmaWNhdGUgZm9yIGluc3Rh
bmNlMSAwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMO3kgVhc1NH9ryoqPuN
h5pOZ54MGoDizDglEDHPwtnWsmPBMDG1ch+P1FDNxu1VbwldrYLXC/7uO9qJgtmD
GGyV9CHAmRJUeLSu1wfFPnLe1EFPlPVKsM0TVOCYUsS0pbgdFxFvdugIize5s2zO
KuZwOTFSsCLIMJU+ctC+XCorAgMBAAEwDQYJKoZIhvcNAQEEBQADgYEAWFHzJl/y
InbZ+u3Lg8UwVz3Vd5CYSpK7OCN8ktUfwNJL3ppYUiMCc9bjxvUaW+j8rSY5kmXf
WNQvwmKl/DqgED+K8DwPNN9eOErhoZzL/5Q0k8EZ59gAwf8N4cVdG8RTkMVPbVBq
BlejNufQxKWZ9hBPM5Wr6SItI9SBeZ8eHbg=
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
Acceptable client certificate CA names
/C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
/C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
/C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for instance1
---
SSL handshake has read 1301 bytes and written 210 bytes
---
New, TLSv1/SSLv3, Cipher is AES256-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : AES256-SHA
    Session-ID:
    Session-ID-ctx:
    Master-Key: 1934C77A01B4B66F478681821DB53C48C2C4100A3168E919D22C508E8EA0D47945A0D43D07A6D6910EB7ADC1B6AA2387
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333581
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:4443 -tls1
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify return:1
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICejCCAeMCEFdlXCvnQO50bxncfLSl+XEwDQYJKoZIhvcNAQEEBQAwfTELMAkG
A1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1SRURXT09EU0hPUkVTMQ8w
DQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgGA1UEAxMhU2VsZi1TaWdu
ZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMCAXDTE2MDIxMDExMzUwOVoYDzIwNjYw
MTI4MTEzNTA5WjB9MQswCQYDVQQGEwJVUzELMAkGA1UECBMCQ0ExFjAUBgNVBAcT
DVJFRFdPT0RTSE9SRVMxDzANBgNVBAoTBk9SQUNMRTEMMAoGA1UECxMDT0FTMSow
KAYDVQQDEyFTZWxmLVNpZ25lZCBDZXJ0aWZpY2F0ZSBmb3Igb2hzMSAwgZ8wDQYJ
KoZIhvcNAQEBBQADgY0AMIGJAoGBALLZLr0ePfA7LjMjYnTAqRPG6u3P2Z+KX9Ct
ckS8pC2Ma8vGXF3YguesVA/da0rsiDfL5AEWpj4sElTsefLZSv636GEDhP2wbmAA
AGh0SSqvshaC2DV0+6fIFZIDVM9DbqT+yboNtsxCTJSQhcgk6B5ApPsUKwUkYEjM
iXLWM0iVAgMBAAEwDQYJKoZIhvcNAQEEBQADgYEANHyOPHb2Y895s8PY74ZavDvO
/5yCtMBAuaNIhaEzpoRh4tKHsPSPKFYLBIAguKlRDVuup8fD8qVCKOhYJASfSJn7
ogRPVF5W5YoXS1kL8BOhpQWQD/Nx4Zyg4zHN+/N3ibGb5h0jd8UlGYDSGrWJqmtM
Rt6F1ZyUjWNqRzr51kQ=
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
No client certificate CA names sent
---
SSL handshake has read 783 bytes and written 345 bytes
---
New, TLSv1/SSLv3, Cipher is DES-CBC3-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : DES-CBC3-SHA
    Session-ID: 732F3622F33FDF0DC16B0423CB3504E6
    Session-ID-ctx:
    Master-Key: 2545DFCACA36ADE3B57C40670F81F609F11543FBAC3047666BCEAB3B4F6F394333F33EED7E3A1091E41FB978AAC74A05
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333588
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:9999 -tls1
<pre>
CONNECTED(00000003)
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = CA, L = REDWOODSHORES, O = ORACLE, OU = OAS, CN = "Self-Signed Certificate for ohs1 "
verify return:1
139835650090824:error:14094412:SSL routines:SSL3_READ_BYTES:sslv3 alert bad certificate:s3_pkt.c:1257:SSL alert number 42
139835650090824:error:1409E0E5:SSL routines:SSL3_WRITE_BYTES:ssl handshake failure:s3_pkt.c:596:
---
Certificate chain
 0 s:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
   i:/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICeDCCAeECEAE2FSWh5/PHGvfxJI8VaX0wDQYJKoZIhvcNAQEEBQAwfTELMAkG
A1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1SRURXT09EU0hPUkVTMQ8w
DQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgGA1UEAxMhU2VsZi1TaWdu
ZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMB4XDTE2MDIxMDExMzUxMVoXDTQxMDIw
MzExMzUxMVowfTELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1S
RURXT09EU0hPUkVTMQ8wDQYDVQQKEwZPUkFDTEUxDDAKBgNVBAsTA09BUzEqMCgG
A1UEAxMhU2VsZi1TaWduZWQgQ2VydGlmaWNhdGUgZm9yIG9oczEgMIGfMA0GCSqG
SIb3DQEBAQUAA4GNADCBiQKBgQC+aivnTuzCWHZsoNnbq9SeaElUyt6tdtAPVbdL
xu7rIFi70wv95aKFaVEhekYkg90H7/IgwghPwuLIZERDxblZAnSBZjxs582GQob3
lbvZDAxpOMYNZun6S1hLLPHGhemU/VZq1MEQ7HiledFOj93XgZ7xgWpAAbYQJHWY
2j3Y5wIDAQABMA0GCSqGSIb3DQEBBAUAA4GBAJu/b9u78g6Tl97hThlzzeoGUevF
ev7eTQAu3iZQO7QtJN+3xk6ARjmyjoZQakC3IX88grNnKpwH0yw9fsVQMfnWoL9S
BCWa5YGnBMmRBQT5bH6tnWN1lj6krvW0D9vr37IT1riK2sVXUPstgDVPq3XNZcK4
hED82YE5PHvILmI0
-----END CERTIFICATE-----
subject=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
issuer=/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
Acceptable client certificate CA names
/C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
/C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
/C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
/C=US/ST=CA/L=REDWOODSHORES/O=ORACLE/OU=OAS/CN=Self-Signed Certificate for ohs1
---
SSL handshake has read 1297 bytes and written 198 bytes
---
New, TLSv1/SSLv3, Cipher is RC4-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1
    Cipher    : RC4-SHA
    Session-ID: 977AD4BA43CEA34057916F662A3582E4
    Session-ID-ctx:
    Master-Key: BAA46B5AFE37D3B2C042C18467F812E5182637F806FB26534459B0492486540A2986DC84FA30F0F23DFBA5C1F38A4C4C
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333595
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
---
</pre>
$ openssl s_client -connect baraka:9999 -ssl2
<pre>
CONNECTED(00000003)
140277818488648:error:1407F0E5:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 14 bytes and written 48 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333609
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:9999 -ssl3
<pre>
CONNECTED(00000003)
140038257866568:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:337:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 5 bytes and written 7 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333612
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:6701 -ssl3
<pre>
CONNECTED(00000003)
139877174888264:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:337:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 5 bytes and written 7 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333619
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:6701 -ssl2
<pre>
CONNECTED(00000003)
140668735379272:error:1407F0E5:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 14 bytes and written 48 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333621
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:4443 -ssl2
<pre>
CONNECTED(00000003)
139786112558920:error:1407F0E5:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 14 bytes and written 48 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333634
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
</pre>
$ openssl s_client -connect baraka:4443 -ssl3
<pre>
CONNECTED(00000003)
140281526929224:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:337:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 5 bytes and written 7 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1539333638
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---

</pre>

из рабочих протоколов остался только TLSv1