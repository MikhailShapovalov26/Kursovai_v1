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
 
далее я начал работать с vault 

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

		vault write pki_int/issue/spbserv-dot-com  common_name="spbserv.com" ttl="720h">spbserv.com.crt
		
Далее я сталкнулся с проблемой

    cat spbserv.com.crt
    Key                 Value
    ---                 -----
    ca_chain            [-----BEGIN CERTIFICATE-----
    MIIDHjCCAgagAwIBAgIUHUvX+rslb5aa9Z2/BIJqBvsvHNowDQYJKoZIhvcNAQEL
    BQAwADAeFw0yMjA0MDUxMDI2MjRaFw0yNzA0MDQxMDI2NTRaMC0xKzApBgNVBAMT
    InNwYnNlcnYuY29tIEludGVybWVkaWF0ZSBBdXRob3JpdHkwggEiMA0GCSqGSIb3
    DQEBAQUAA4IBDwAwggEKAoIBAQCzrxgwXwbidearN84fncM/s6Un6MY9ATO2b43s
    ArxviEVH2xv2S/GsLc1iEsGw+qVetAQpIr/q/zE7yyksaGMJT1uAdMsUySLRQLFQ
    FqtICFFG9O9zoefizQPtur+G0gP9NnZINoFakL/mEUAXQCglK1GeRlr1YaF+Sedb
    gZ1Bl3fPlRgwe1EsTL+oiFBUQWap+m7s+I9Ek0qkvchtXMWGBSqmoGSyJQI6V5Gz
    FHrVm23EkzwNn8WmvvZs65IuyZw8z3HnD9iL+a0Z6JIb307jDQrWgPtnuKA/696j
    wRoIjjEaJzNFJUhauabORZimnFnFbs8RVnkRfnqXvjlVwo+ZAgMBAAGjYzBhMA4G
    A1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBQu+ryKVNVC
    Y8qr+MYNiR8asH0rwDAfBgNVHSMEGDAWgBS7KVyZ9IojZuebtUFGTOAJNR092jAN
    BgkqhkiG9w0BAQsFAAOCAQEAScf4G+qr1yYZLGlSPAvz9Zie6BYv3LBPnQMIgR+e
    xSfpLHuCK4Kw3k0+zYsJdGxRpo86SlM38bc/Ej5F8g7wpNVz98P0YP9uTzUOrHnW
    Jxx5UFq2BsnwLvlkdmTTgM1OI74bppsLzJGuelD6Gzjc9ptwD7gQvkfZ5pedyyIk
    WcQ+jq9ceLufU/eyfhnwDA2mN6PcMdxkri+tXc9jH70eIee5fPSftUTJz2/aDU/q
    4U1TPoRPsF+QVSSoivKnHfJklaaqcjoDtZx2IqOmdR6iW7btpNzSe2+taLwHE4H1
    QZ6dGwa2HTvJUOZz1hWLjGmjXa6rm+TqZy9RMORdgMB9tA==
    -----END CERTIFICATE-----]
    certificate         -----BEGIN CERTIFICATE-----
    MIIDXDCCAkSgAwIBAgIUYILmUz6H7Nm7UMxBI17rou1Vd1IwDQYJKoZIhvcNAQEL
    BQAwLTErMCkGA1UEAxMic3Bic2Vydi5jb20gSW50ZXJtZWRpYXRlIEF1dGhvcml0
    eTAeFw0yMjA0MDUxMTExMDhaFw0yMjA1MDUxMTExMzdaMBYxFDASBgNVBAMTC3Nw
    YnNlcnYuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAo92epK5v
    HPzZfwo4VWLcIJJklz5D5+iICqDOFjUTJd2a10kSTSuMaYC0B67DRyQmZIcAp3Fo
    TLps0aT4zy22R06dJ8sQn5GFdlI0LwspeYa2NZCbaAL5tS5fPICPjp73jsrkCfQi
    b7juwh67o8gZPjrglFWMCmyAYprjs5iFKDVj1uLTxowXGadSsYK2Qm3/oauRrFb0
    2f3ftt8bQjcS4ouliwmBEUN4BdjvxWh0H7GqCyV5hDxosfd4G5ne0nkW+ZwlyDZB
    MkZ2FJa5b9RjuL4mMUYGD6SV6zVkveLN0yJla65m3NFBRa4fnfJraRp83vOogako
    P6WWIo+FwjSd4QIDAQABo4GKMIGHMA4GA1UdDwEB/wQEAwIDqDAdBgNVHSUEFjAU
    BggrBgEFBQcDAQYIKwYBBQUHAwIwHQYDVR0OBBYEFCnQBqxegLQbPCmPQhmCE2vw
    t4mOMB8GA1UdIwQYMBaAFC76vIpU1UJjyqv4xg2JHxqwfSvAMBYGA1UdEQQPMA2C
    C3NwYnNlcnYuY29tMA0GCSqGSIb3DQEBCwUAA4IBAQASYZxDyuyZtQE4g6STEORb
    HleoByZK5e9xMAJ+QY3m0uxUNUX+fKcoe77G7DP3T1DFUDA0vHLGismu38B9JQgc
    n7A2nZOGgnJDZieuo1TOWClk85TJXVFVQBSbbDlh6gI50P1iUA9dMWblJtJs0dpj
    ynQNpoZsMQLrcPIOd1uBkaPa8WO5wYVmIX4932WVqE/CMMQTc3gpjHOjMDqenwZ0
    Z8riVhq9tTLTBaW8aIrowzNtFERWTJfc6QpFBmW7JnlwCfUAN2Mb9BuiQZzHmJDP
    ty/PeYn2naGoaBUniIEeiwhuELqlHZk57u9Ab5SC5SiqKHiezKCBwzJKecX4TYVX
    -----END CERTIFICATE-----
    expiration          1651749097
    issuing_ca          -----BEGIN CERTIFICATE-----
    MIIDHjCCAgagAwIBAgIUHUvX+rslb5aa9Z2/BIJqBvsvHNowDQYJKoZIhvcNAQEL
    BQAwADAeFw0yMjA0MDUxMDI2MjRaFw0yNzA0MDQxMDI2NTRaMC0xKzApBgNVBAMT
    InNwYnNlcnYuY29tIEludGVybWVkaWF0ZSBBdXRob3JpdHkwggEiMA0GCSqGSIb3
    DQEBAQUAA4IBDwAwggEKAoIBAQCzrxgwXwbidearN84fncM/s6Un6MY9ATO2b43s
    ArxviEVH2xv2S/GsLc1iEsGw+qVetAQpIr/q/zE7yyksaGMJT1uAdMsUySLRQLFQ
    FqtICFFG9O9zoefizQPtur+G0gP9NnZINoFakL/mEUAXQCglK1GeRlr1YaF+Sedb
    gZ1Bl3fPlRgwe1EsTL+oiFBUQWap+m7s+I9Ek0qkvchtXMWGBSqmoGSyJQI6V5Gz
    FHrVm23EkzwNn8WmvvZs65IuyZw8z3HnD9iL+a0Z6JIb307jDQrWgPtnuKA/696j
    wRoIjjEaJzNFJUhauabORZimnFnFbs8RVnkRfnqXvjlVwo+ZAgMBAAGjYzBhMA4G
    A1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBQu+ryKVNVC
    Y8qr+MYNiR8asH0rwDAfBgNVHSMEGDAWgBS7KVyZ9IojZuebtUFGTOAJNR092jAN
    BgkqhkiG9w0BAQsFAAOCAQEAScf4G+qr1yYZLGlSPAvz9Zie6BYv3LBPnQMIgR+e
    xSfpLHuCK4Kw3k0+zYsJdGxRpo86SlM38bc/Ej5F8g7wpNVz98P0YP9uTzUOrHnW
    Jxx5UFq2BsnwLvlkdmTTgM1OI74bppsLzJGuelD6Gzjc9ptwD7gQvkfZ5pedyyIk
    WcQ+jq9ceLufU/eyfhnwDA2mN6PcMdxkri+tXc9jH70eIee5fPSftUTJz2/aDU/q
    4U1TPoRPsF+QVSSoivKnHfJklaaqcjoDtZx2IqOmdR6iW7btpNzSe2+taLwHE4H1
    QZ6dGwa2HTvJUOZz1hWLjGmjXa6rm+TqZy9RMORdgMB9tA==
    -----END CERTIFICATE-----
    private_key         -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEAo92epK5vHPzZfwo4VWLcIJJklz5D5+iICqDOFjUTJd2a10kS
    TSuMaYC0B67DRyQmZIcAp3FoTLps0aT4zy22R06dJ8sQn5GFdlI0LwspeYa2NZCb
    aAL5tS5fPICPjp73jsrkCfQib7juwh67o8gZPjrglFWMCmyAYprjs5iFKDVj1uLT
    xowXGadSsYK2Qm3/oauRrFb02f3ftt8bQjcS4ouliwmBEUN4BdjvxWh0H7GqCyV5
    hDxosfd4G5ne0nkW+ZwlyDZBMkZ2FJa5b9RjuL4mMUYGD6SV6zVkveLN0yJla65m
    3NFBRa4fnfJraRp83vOogakoP6WWIo+FwjSd4QIDAQABAoIBAH0xVoEO28lTzH9o
    uX1S2EbyUXPTmGHXoAgurwT8a7KkSiZsp1TaDp6UO/caqAr0LXjkQ7WpyTvFulm5
    JnZywC5ee2bpl7uxnDu3tjKy3m8AYrktz+15SHoKAazhs8wM26n2jJ6mLKEasx8Q
    B9+rgs2ugeISMbnNB5FOMOUHg8QhPMZOS1Fxa3ZQjh3xtnAPlg0trJIx7mUyx9E6
    MjsVlYOHf2KL9crAIa8BNCmu+1aGHXNM9Y2+9PMECEdDCVReH3n0wkx0HNstYWiU
    Ut1UEe7b4lgmz0Cm5SY1QY5nrp/Vb1jS7l59ZmMYwhmZcFO3m6k3mSTtcWgXJP0i
    ZlPS1SECgYEAwe/smlk6W1xDGF3QGJ3OUmYXCoCNcUR8fpIiafnQZ863mpgK5ENe
    g22436zY66hSC/qFHoYHqphEuiBLXEcbCpzto6cuhaDSBRBFXDt6YIrexgfvGgxK
    urh9IgTa30UoKfoIiT+fr4rXxx3qR6mZsxgISVZyLgMD9RUeCptZY6UCgYEA2E4f
    jvdaHSFt5i1fauzBblx/gVpxw67b7DZ35ZnhV74fJU+PkfDjFPkhu+IFcLGxgT3i
    F40OZIikUTmTChlfztLOvQXT6ZZfSBLMe/HWx567/7PR8KCsIhOxZoER2HL7OnDE
    X35QMNGITrEeEFqwOnMacFCJFoigNmMQ6ZaJDI0CgYEAtnwvE1FwgvT+wVfM7szW
    jlw3xA8QiIsb5fFV5ohFXNh7lUEJxp3JujutYPMArkYU5eaWChGt9w0OZmDq6GqT
    /FmLlplCQkUAOfmEenQRA/TICGkAyG7WhnoAbNlKphop383A6HxworovrdHtV/8z
    e/zaFz/7cmYt/Bghy3NAGm0CgYEAhII3awm0tqvH+35IOeSYCte3dLLHhq0UJPyp
    Loq6NVpPEjhPJ4R+WFbWh5bK5mK07wvN+cd7zbK3ltrCbSlmO/mAlOOBElQAQtLh
    WfypKtjfKqIqNlL3oFiYEMd4+zRVG1QBuM5UqdNywWJXnIUx+FyTEcMEeD1yiF7f
    +Xkys/ECgYAYbN8vXja8POfuBPJRDDprynsbd/hg/G+cLPHM8GyiFwkrjUMJ9JbB
    GP2VC4d7Os6ATtQYoxfJynZbLssxGtUTvVMCMLVhU3afuMdIuXmbF6zuq/aHZM8M
    XgN0XqazKApDooJTe7Qyb0SSXtQnkzBDgX2JfPlDnsFPh/QaTs6mHA==
    -----END RSA PRIVATE KEY-----
    private_key_type    rsa
    serial_number       60:82:e6:53:3e:87:ec:d9:bb:50:cc:41:23:5e:eb:a2:ed:55:77:52
    
и пытаюсь нарезать

		 cat spbserv.com.crt | jq -r .data.certificate > spbserv.com.crt.pem
		parse error: Invalid numeric literal at line 1, column 4
		
я сравнил свой сертификат и сертификат который получается при стандартном создание сертификата
и разница в первых строках

     vault cat example.com.crt
    {
      "request_id": "af9a6c00-04c6-913b-7282-b9fc07645e5e",
      "lease_id": "",
      "lease_duration": 0,
      "renewable": false,
      "data": {
        "ca_chain": [
          "-----BEGIN CERTIFICATE-----\nMIIDpjCCAo6gAwIBAgIUaRE/M0bDZd85fHgVXZDzvanX4YQwDQYJKoZIhvcNAQEL\nBQAwFjEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMjIwNDA1MTAxODM4WhcNMjcw\nNDA0MTAxOTA4W
Прощу помочь разобраться. Решить самостоятельно не получается. Хост имя поменял принципиально, чтоб понять как работает vault в других условиях. 

    /etc/hosts
    127.0.0.1 localhost
    127.0.0.1 spbserv.com

