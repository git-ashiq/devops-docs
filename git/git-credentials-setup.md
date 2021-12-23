#### Setup for HTTPS users using Git credentials
https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-https-unixes.html

```
aws configure
```

```
AWS Access Key ID [None]: Type your IAM user AWS access key ID here, and then press Enter
AWS Secret Access Key [None]: Type your IAM user AWS secret access key here, and then press Enter
Default region name [None]: Type a supported region for CodeCommit here, and then press Enter
Default output format [None]: Type json here, and then press Enter
```

#### Install Git
```
sudo apt install git
```

#### Set up the credential helper
```
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
```


