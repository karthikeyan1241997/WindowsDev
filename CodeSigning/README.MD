# Signing Binaries Using Self-Signed Certificate for Local Development

## Steps involved
 - Create a Self-Signed Certificate (aka Root CA)
 - Install self-Singed Certificate to Windows Root Certificate Store
 - Create a Signing Certificate
 - Convert to Pfx format
 - Sign files using signtool

## Tools Required
- Developer Command Prompt for VS 20**

## Instructions

### 1) Launch Developer Command Prompt
Search Developer command prompt in Windows Search and run the `Developer Command Prompt for VS 20**` with admin previliges.

### 2) Create Self-Signed Certificate
Create the self-signed certicate by running the following command.
```
makecert -r -pe -n "CN=My CA" -ss CA -sr CurrentUser -a sha256 -cy authority -sky signature -sv MyCA.pvk MyCA.cer
```

### 3) Install the self-signed certificate to windows trusted store
Run the following command to install the self-signed certificated generated using the previous command to windows trusted Root CA store
```
certutil -user -addstore Root MyCA.cer
```

Note: You may face file not found error for `MyCA.cer` in 64-bit machines, as genereated `MyCA.cer` is located in `C:\Windows\SysWOW64` directory. In that case run the following command.
```
certutil -user -addstore Root C:\Windows\SysWOW64\MyCA.cer
```

### 4) Create code-signing certificate (Software Publisher Certificate)
Generate code-signing certificate by running the following command which will generate new server certificate signed by our Self-signed certificate
```
makecert -pe -n "CN=My SPC" -a sha256 -cy end -sky signature -ic MyCA.cer -iv MyCA.pvk -sv MySPC.pvk MySPC.cer
```

### 5) Convert to Pfx format
Convert the code-signing certificate to pfx format by executiing the following command
```
pvk2pfx -pvk MySPC.pvk -spc MySPC.cer -pfx MySPC.pfx
``` 

That's it!... You have your own code-signing certificate

### 6) Sign file using SPC
Now, you can sign any file using code-signing certicate using `signtool`
```
signtool sign /v /f MySPC.pfx /t http://timestamp.verisign.com/scripts/timstamp.dll abcfile.exe
```

Note: You can use any timestamp url

## Reference
- https://stackoverflow.com/a/201277