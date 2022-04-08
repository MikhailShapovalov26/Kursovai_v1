# Курсовая работа по итогам модуля "DevOps и системное администрирование"

## 1. 

Выполнено

## 2.

    sudo ufw status numbered
    Status: active
    
         To                         Action      From
         --                         ------      ----
    [ 1] 22/tcp                     ALLOW IN    Anywhere
    [ 2] 443/tcp                    ALLOW IN    Anywhere
    [ 3] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
    [ 4] 443/tcp (v6)               ALLOW IN    Anywhere (v6)

## 3.

     wget https://hashicorp-releases.website.yandexcloud.net vault/1.9.3vault_1.9.3_linux_amd64.zip
     
     mv vault /usr/bin
 
Работать с vault 

Запуск 

		vault server -dev -dev-root-token-id root 
для обращения к серверу export 

		VAULT_ADDR=http://127.0.0.1:8200 
		
аутенфикации на сервере ваулт export 

		VAULT_TOKEN=root 

создание корневой ЦС 

		vault secrets enable pki 
		
механизм серкетов для выдачи сертификатов 

		vault secrets tune -max-lease-ttl=87600h pki 
создание коренового сертифката 

		vault write -field=certificate pki/root/generate/internal common_name="testhome.com" \ ttl=87600h > CA_cert.crt 

настройка урл адреса CA и CRL 

		vault write pki/config/urls \ issuing_certificates="$VAULT_ADDR/v1/pki/ca" \ crl_distribution_points="$VAULT_ADDR/v1/pki/crl" 
		
создание промежуточной ЦС включаем pki механизм секретов 

		vault secrets enable -path=pki_int pki 
		
настройка pki int для выдачи сертификатов с максимально сроком жизни 

		vault secrets tune -max-lease-ttl=43800h pki_int 
		
генерация промежуточного сертификата 

		vault write -format=json pki_int/intermediate/generate/internal common_name="spbserv.com Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr 
		
подписать промежуточный сертификат закрытым ключом 

		vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem 
		
импорт обратно в VAult 

		vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem 
		
создание роли testhome-dot-com 

		vault write pki_int/roles/spbserv-dot-com allowed_domains="spbserv.com" allow_bare_domains=true allow_subdomains=true max_ttl="720h" 
создаем сертификат


		vault write -format=json pki_int/issue/spbserv-dot-com  common_name="spbserv.com" ttl="720h">spbserv.com.crt
		
Структура 

    cat spbserv.com.crt
    {
      "request_id": "e5b4c11f-86b6-7678-f66b-a2b4e5273ceb",
      "lease_id": "",
      "lease_duration": 0,
      "renewable": false,
      "data": {
        "ca_chain": [
          "-----BEGIN CERTIFICATE-----\nMIIDHjCCAgagAwIBAgIUWZfgJax/9kvd2UFPTCcWkDCH                                     J8gwDQYJKoZIhvcNAQEL\nBQAwADAeFw0yMjA0MDcxODE4MjNaFw0yNzA0MDYxODE4NTNaMC0xKzApBg                                     NVBAMT\nInNwYnNlcnYuY29tIEludGVybWVkaWF0ZSBBdXRob3JpdHkwggEiMA0GCSqGSIb3\nDQEBAQ                                     UAA4IBDwAwggEKAoIBAQDe+b25lZcxCE7krbRHoZnnSIS3twhYZw8lFsCq\nkUANF11PwTeciv7hCNsD                                     5prE5A5kvISnxDkx0a/p8u7xWb95uyKsvXfIrSihcX0T\nn0bPZ92+7tanhKxBZQTdlLdxiqgu6rGIh9                                     KIgcnkJmjDFPLMMx5qJGOs9cIVS+E2\nwQ5JvzwVKW4z1zKxiYpAK1I0q6Wi1A/ZiVKIr/s7acnRvGug                                     isMZrpOEo9JTxybI\nelwJFAiRl/093ouzbikWC9TNFUTOp/1WZ/aArjHLcqDjc4MFLzDxIku9Xd6gSA                                     1n\nY3LP5v+tYlP06R13YipkvWOxQujC8CGEtSeUfpca18nRo3wfAgMBAAGjYzBhMA4G\nA1UdDwEB/w                                     QEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBTbO/RBhA3c\nfh629cA3ayM7cI4fmTAfBgNV                                     HSMEGDAWgBQLo1U66RM+xCTbynqIIuwz5sXxzjAN\nBgkqhkiG9w0BAQsFAAOCAQEAk8WBUqnllD4tzO                                     i0QXLgvnbQufsPUs6F7g51QiQb\nqLucLoIyIKHdHSQI7q3mzmxc5BDCAN+ZzEEyQzId9LNkiN+K1a76                                     ia+gsh3CcQqq\nxSR7pweG4awrBZnVxqboFfJdlkJU689C9rng2AQ98q8QVHQtPWxClSZqD1iV8LYb\n                                     DKv//V3aJD8kJng+8up0UucQRu9oNBV9PX5tWhP3eOJ5emyoWjuRuB7JLYZ8NKNW\nQt7qcytRPn1Z08                                     sLXlEZqNqu6uVRc8HF5YLLmOXAnk0E2kBKcEiu/45EZ9SGIj3w\nVHvpwvmPhtuGXxrDQwtYG6wHCN1e                                     tfL+REpKx7ryuIF/zw==\n-----END CERTIFICATE-----"
        ],
        "certificate": "-----BEGIN CERTIFICATE-----\nMIIDXDCCAkSgAwIBAgIUdjmCYA2ubsE                                     EOdE2Wt/2Uos7RdAwDQYJKoZIhvcNAQEL\nBQAwLTErMCkGA1UEAxMic3Bic2Vydi5jb20gSW50ZXJtZ                                     WRpYXRlIEF1dGhvcml0\neTAeFw0yMjA0MDcxODE4NTRaFw0yMjA1MDcxODE5MjRaMBYxFDASBgNVBAM                                     TC3Nw\nYnNlcnYuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAv4prYog8\nAEhb6S8                                     YXF5FeLN9/NPzImZLKSgYGFNVeovSAkRLtZUddN65jVX3gkH9dMffYOe8\nQYzqJkzGb3gsMeqDViTXj                                     0hAdFbKGMJkEvvAIxb7eiDtJ3K2obH9EDbmR1P4xX6U\n/6N5TbJB51FT+h5NH0fCZXsOthtYd72dXl/                                     p7+MEkt4qf5HGnonsq+GeZtXL3vR5\nkNJ4UMsRAojndJKBAc1za423vGTyUU1F8mQvy53h8/aPRBK/r                                     mVsBN6yGS22v7KZ\n0DLRMDHIefLj66YQ6jZ1bRx+2AKHeybWhG/YpKkWQug0nve+5Mdp9MHMIAOFuDu                                     J\nIfja9yE9yf67dQIDAQABo4GKMIGHMA4GA1UdDwEB/wQEAwIDqDAdBgNVHSUEFjAU\nBggrBgEFBQc                                     DAQYIKwYBBQUHAwIwHQYDVR0OBBYEFAlLqpiWx5Mwuczhle2ps4Hd\nY9uGMB8GA1UdIwQYMBaAFNs79                                     EGEDdx+Hrb1wDdrIztwjh+ZMBYGA1UdEQQPMA2C\nC3NwYnNlcnYuY29tMA0GCSqGSIb3DQEBCwUAA4I                                     BAQBV7sAkRx04v5M6jEjyCX2L\ndlnA4Pg8tXxpFxUfL/8RZK6QVPFEnshW7QiVnw9miZjIiU8m6IJfy                                     +PXyWI466/r\nCXtfhmBo1bz8zLXT/V5yc9bAwma5wojGX0yX8S3HG1Swma64nnzrx9WEII7q+9Ff\n+                                     jDAULAbw0mlXXnbu0Rxr17XkxsayHkjQe8ML49TiY5ir2gtfbJLMZCU7VgAshBd\ndKMgNR+mgnHurfH                                     mxJXEel9S54yRjGzA0WtWvD5L699muwYYo+KZalCSuuUlIk0k\nmv/qsfQvPLXDkdNhj+WlIB+bBW+rl                                     HuLY+4KRrRcepjgC5S8a1n7CEP1bt7AqRNB\n-----END CERTIFICATE-----",
        "expiration": 1651947564,
        "issuing_ca": "-----BEGIN CERTIFICATE-----\nMIIDHjCCAgagAwIBAgIUWZfgJax/9kvd                                     2UFPTCcWkDCHJ8gwDQYJKoZIhvcNAQEL\nBQAwADAeFw0yMjA0MDcxODE4MjNaFw0yNzA0MDYxODE4NT                                     NaMC0xKzApBgNVBAMT\nInNwYnNlcnYuY29tIEludGVybWVkaWF0ZSBBdXRob3JpdHkwggEiMA0GCSqG                                     SIb3\nDQEBAQUAA4IBDwAwggEKAoIBAQDe+b25lZcxCE7krbRHoZnnSIS3twhYZw8lFsCq\nkUANF11P                                     wTeciv7hCNsD5prE5A5kvISnxDkx0a/p8u7xWb95uyKsvXfIrSihcX0T\nn0bPZ92+7tanhKxBZQTdlL                                     dxiqgu6rGIh9KIgcnkJmjDFPLMMx5qJGOs9cIVS+E2\nwQ5JvzwVKW4z1zKxiYpAK1I0q6Wi1A/ZiVKI                                     r/s7acnRvGugisMZrpOEo9JTxybI\nelwJFAiRl/093ouzbikWC9TNFUTOp/1WZ/aArjHLcqDjc4MFLz                                     DxIku9Xd6gSA1n\nY3LP5v+tYlP06R13YipkvWOxQujC8CGEtSeUfpca18nRo3wfAgMBAAGjYzBhMA4G                                     \nA1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBTbO/RBhA3c\nfh629cA3ayM7                                     cI4fmTAfBgNVHSMEGDAWgBQLo1U66RM+xCTbynqIIuwz5sXxzjAN\nBgkqhkiG9w0BAQsFAAOCAQEAk8                                     WBUqnllD4tzOi0QXLgvnbQufsPUs6F7g51QiQb\nqLucLoIyIKHdHSQI7q3mzmxc5BDCAN+ZzEEyQzId                                     9LNkiN+K1a76ia+gsh3CcQqq\nxSR7pweG4awrBZnVxqboFfJdlkJU689C9rng2AQ98q8QVHQtPWxClS                                     ZqD1iV8LYb\nDKv//V3aJD8kJng+8up0UucQRu9oNBV9PX5tWhP3eOJ5emyoWjuRuB7JLYZ8NKNW\nQt                                     7qcytRPn1Z08sLXlEZqNqu6uVRc8HF5YLLmOXAnk0E2kBKcEiu/45EZ9SGIj3w\nVHvpwvmPhtuGXxrD                                     QwtYG6wHCN1etfL+REpKx7ryuIF/zw==\n-----END CERTIFICATE-----",
        "private_key": "-----BEGIN RSA PRIVATE KEY-----\nMIIEpQIBAAKCAQEAv4prYog8AEh                                     b6S8YXF5FeLN9/NPzImZLKSgYGFNVeovSAkRL\ntZUddN65jVX3gkH9dMffYOe8QYzqJkzGb3gsMeqDV                                     iTXj0hAdFbKGMJkEvvAIxb7\neiDtJ3K2obH9EDbmR1P4xX6U/6N5TbJB51FT+h5NH0fCZXsOthtYd72                                     dXl/p7+ME\nkt4qf5HGnonsq+GeZtXL3vR5kNJ4UMsRAojndJKBAc1za423vGTyUU1F8mQvy53h\n8/a                                     PRBK/rmVsBN6yGS22v7KZ0DLRMDHIefLj66YQ6jZ1bRx+2AKHeybWhG/YpKkW\nQug0nve+5Mdp9MHMI                                     AOFuDuJIfja9yE9yf67dQIDAQABAoIBAQCuKybUdP7iXum8\nkwd6znordEOmx+/5Ir1Lklq+m+BJS6T                                     OTN3vIsWjRPfCzh3/QxQ+wZScoHvtuz4b\nT4rNR6AgT5fXmC6u6ks5vsWMWA75uBQRlE9d0Vz0lvn89                                     EUiHlmtGG1I4H6KKlMe\nVG5a8fgDotQMEotxDcHl093RHAliCw1Rg866Oy5DMVSUKzMGO+aeabWPh6e                                     pGe/x\nYX6BQEnEipylh+qlKvCs4FWFR6GzpgjC6VFJ1wMiu56XPfiMEZmHqvpCL7x1Vpab\ngZIG+8o                                     w6L8f3gbo6A0OAyJhJu/mGowTDf34uUYEHBHeqyAgJHvE6WB0+uXTigzN\nHRJTTqINAoGBAPta0I84U                                     A9HwxyF+21YrcMxbNa0v6kacA0l4GYK07526gdJ3HtL\nDXC0vXlprwbpU+61bLoFOuUUEGKE9p8qcTX                                     IoSCBlaji978Wgt+8jZ/I7zs5c2WR\nrCYZiImfFBiJq8+k515zKaw+1nq5EJNw5uAkLqy1dtDZBLe1W                                     ZWdU1MbAoGBAMMU\nnkx7tv74se5G3cr2tbpNKPwRNCtLROf/hMAVJLaq0yHLrrFaaiVpZYndw05lYJE                                     f\nQX34+0gQVlmO6h7u25+LHJmN0l8qVG3YcBFas0BgyNnoBK1TFfi9uDtdpFgNgsBC\nhrGKdMZbqXT                                     SZ4eXYrVVxsKaM+M4PTztkMJJfYSvAoGBAIKI6DToxtMYrqA6iJeB\n7UOogGCEn+B/RV/eosn7p3mLo                                     gBFAIbbNFCq3JeYJfiN5jPa7msw2iQrplKzwKxq\nSz7mQZ9zDLkXnhgT5cWx007++BamwGIoI+Hpcro                                     d7j7MCHkep7WqUyMLwzEauRwJ\nDy2qPL8ewmS1v7h0KuEDQkmvAoGAGVojnGLzjD07unChQFBBm72NV                                     vzRX9L/WHsf\ngGr/A3I90AWNZWUPqF8DICCe0iBgpVt/gmSKG3psDmWBTZXTGu/UPqxsq4tvv3rC\nT                                     a8Q3ov1bkQ132SN1W6qc6SeyAaxeFTs9kO9G04sPo9dfwSQgmpsUzwBBEV5x+JD\nzswIMOcCgYEAiAz                                     IQUtNJU6ocBs+bmgmS1yQIhJz1LPlUiWXic0LF0LGn4wMr7dS\nd/I9F+pQvTXJr1kem3nJ17ymOeZya                                     LmphfEvUgiOYF0lhWOYPAvffC2KxfcWsPXn\n0NnVW0iLeNAYOtE8tf3qJwCpFbxo6NZIwb0iFIsFFsn                                     BtLan45cw9ho=\n-----END RSA PRIVATE KEY-----",
        "private_key_type": "rsa",
        "serial_number": "76:39:82:60:0d:ae:6e:c1:04:39:d1:36:5a:df:f6:52:8b:3b:45:d                                     0"
      },
      "warnings": null
    }
    
        
нарезаем сертификат

    cat spbserv.com.crt | jq -r .data.certificate > spbserv.com.crt.pem
    cat spbserv.com.crt | jq -r .data.ca_chain[] >> spbserv.com.crt.pem
    cat spbserv.com.crt | jq -r .data.private_key > spbserv.com.crt.key

проверяем

    total 28K
    -rw-rw-r-- 1 mikhail mikhail 1,1K апр  7 18:17 CA_cert.crt
    -rw-rw-r-- 1 mikhail mikhail 1,2K апр  7 18:18 intermediate.cert.pem
    -rw-rw-r-- 1 mikhail mikhail  924 апр  7 18:18 pki_intermediate.csr
    -rw-rw-r-- 1 mikhail mikhail 5,6K апр  7 18:19 spbserv.com.crt
    -rw-rw-r-- 1 mikhail mikhail 1,7K апр  7 18:20 spbserv.com.crt.key
    -rw-rw-r-- 1 mikhail mikhail 2,4K апр  7 18:20 spbserv.com.crt.pem
## 5.
Выполнено

потверждение Ris_kursovai_1.jpg в репозитории
## 6.
       systemctl status nginx.service
    ● nginx.service - A high performance web server and a reverse proxy server
         Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
         Active: active (running) since Thu 2022-04-07 18:21:21 UTC; 15h ago
           Docs: man:nginx(8)
        Process: 1763 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
        Process: 1775 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
       Main PID: 1776 (nginx)
          Tasks: 3 (limit: 4488)
         Memory: 6.0M
         CGroup: /system.slice/nginx.service
                 ├─1776 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
                 ├─1777 nginx: worker process
                 └─1778 nginx: worker process
    
    апр 07 18:21:21 spbserv systemd[1]: Starting A high performance web server and a reverse proxy server...
    апр 07 18:21:21 spbserv systemd[1]: Started A high performance web server and a reverse proxy server.
## 7.
    cat /etc/nginx/sites-enabled/spbserv
    server {
            listen 443 ssl;
            server_name spbserv.com;
            keepalive_timeout   70;
    
            ssl_certificate /home/mikhail/vault/spbserv.com.crt.pem;
            ssl_certificate_key /home/mikhail/vault/spbserv.com.crt.key;
            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    
            root /var/www/spbserv/html;
    
            index  index.html index.htm default.html default.htm;
    }
## 8.
Аналогичный рисунок Ris_kursovai_1.jpg в репозитории

## 9. 
До запуска скрипта дата создания 7.04 

     ll
    total 32K
    -rw-rw-r-- 1 mikhail mikhail 1,1K апр  7 18:17 CA_cert.crt
    -rw-rw-r-- 1 mikhail mikhail 1,2K апр  7 18:18 intermediate.cert.pem
    -rw-rw-r-- 1 mikhail mikhail  417 апр  8 10:35 keysec.sh
    -rw-rw-r-- 1 mikhail mikhail  924 апр  7 18:18 pki_intermediate.csr
    -rw-rw-r-- 1 mikhail mikhail 5,6K апр  7 18:19 spbserv.com.crt
    -rw-rw-r-- 1 mikhail mikhail 1,7K апр  7 18:20 spbserv.com.crt.key
    -rw-rw-r-- 1 mikhail mikhail 2,4K апр  7 18:20 spbserv.com.crt.pem

После запуска скрипта дата создания 8.04

     ll
    total 32K
    -rw-rw-r-- 1 mikhail mikhail 1,1K апр  7 18:17 CA_cert.crt
    -rw-rw-r-- 1 mikhail mikhail 1,2K апр  7 18:18 intermediate.cert.pem
    -rw-rw-r-- 1 mikhail mikhail  520 апр  8 10:38 keysec.sh
    -rw-rw-r-- 1 mikhail mikhail  924 апр  7 18:18 pki_intermediate.csr
    -rw-rw-r-- 1 mikhail mikhail 5,6K апр  8 10:38 spbserv.com.crt
    -rw-rw-r-- 1 mikhail mikhail 1,7K апр  8 10:38 spbserv.com.crt.key
    -rw-rw-r-- 1 mikhail mikhail 2,4K апр  8 10:38 spbserv.com.crt.pem
проверяем работу скрипта

    sudo bash keysec.sh            │=pki_int/ type=pki
    Success! Data written to: pki_int/roles/│2022-04-07T18:18:10.613Z [INFO]  core:
    spbserv-dot-com                         │mount tuning of leases successful: path
    Рестарт nginx успешен.  
    
 Сам скрипт
 
     nano keysec.sh                                                                                    
        #!/bin/bash
        export VAULT_ADDR=http://127.0.0.1:8200
        export VAULT_TOKEN=root   
    # генерация нового сертификата
    
    vault write pki_int/roles/spbserv-dot-com allowed_domains="spbserv.com" allow_bare_domains=true allow_subdomains=true max_ttl="720h"
    vault write -format=json pki_int/issue/spbserv-dot-com  common_name="spbserv.com" ttl="720h" > /home/mikhail/vault/spbserv.com.crt
    
    cat /home/mikhail/vault/spbserv.com.crt | jq -r .data.certificate > /home/mikhail/vault/spbserv.com.crt.pem
    cat /home/mikhail/vault/spbserv.com.crt | jq -r .data.ca_chain[] >> /home/mikhail/vault/spbserv.com.crt.pem
    cat /home/mikhail/vault/spbserv.com.crt | jq -r .data.private_key > /home/mikhail/vault/spbserv.com.crt.key
    
    if systemctl restart nginx
    then
    echo -e "\033[1mРестарт nginx успешен.\033[0m"
    else
    echo -e "\033[1m[!!!] Что то не так, надо проверить руками.\033[0m"
    fi
## 10.
Проверяем работает ли обновление сертификата

    vault ll
    total 32K
    -rw-rw-r-- 1 mikhail mikhail 1,1K апр  7 18:17 CA_cert.crt
    -rw-rw-r-- 1 mikhail mikhail 1,2K апр  7 18:18 intermediate.cert.pem
    -rwxrwxr-x 1 mikhail mikhail  923 апр  8 11:35 keysec.sh
    -rw-rw-r-- 1 mikhail mikhail  924 апр  7 18:18 pki_intermediate.csr
    -rw-rw-r-- 1 mikhail mikhail 5,6K апр  8 13:29 spbserv.com.crt
    -rw-rw-r-- 1 mikhail mikhail 1,7K апр  8 13:29 spbserv.com.crt.key
    -rw-rw-r-- 1 mikhail mikhail 2,4K апр  8 13:29 spbserv.com.crt.pem
  
    ➜  vault ll
    total 32K
    -rw-rw-r-- 1 mikhail mikhail 1,1K апр  7 18:17 CA_cert.crt
    -rw-rw-r-- 1 mikhail mikhail 1,2K апр  7 18:18 intermediate.cert.pem
    -rwxrwxr-x 1 mikhail mikhail  923 апр  8 11:35 keysec.sh
    -rw-rw-r-- 1 mikhail mikhail  924 апр  7 18:18 pki_intermediate.csr
    -rw-rw-r-- 1 mikhail mikhail 5,6K апр  8 13:30 spbserv.com.crt
    -rw-rw-r-- 1 mikhail mikhail 1,7K апр  8 13:30 spbserv.com.crt.key
    -rw-rw-r-- 1 mikhail mikhail 2,4K апр  8 13:30 spbserv.com.crt.pem
редактируем автозапуск

    sudo crontab -e
    * * * * * /home/mikhail/vault/keysec.sh
всё работает. 