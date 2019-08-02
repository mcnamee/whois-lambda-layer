# Lambda `whois` Layer

Adds support for the `whois` (`jwhois`) tool to your Lambda function.

## Installation

- Going to your AWS console
- Navigate to `Lambda > Layers > Create layer`
- Upload this zip and use the ARN in your Lambda function

---

## TLD Conf

It may be helpful to bundle a `jwhois.conf` file with your Lambda function, to enable you to each adjust which data provider is used for each TLD. You can [download an example here](https://github.com/jonasob/jwhois/blob/master/example/jwhois.conf).

To use the conf file: `whois -c backend/conf/jwhois.conf` (note that the path is from the Lambda function root).

---

## Building a new version

Please see the (AWS guide to create a deployment package](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-linux-binary-package/). The following steps are tailored to `whois`.

1. Launch an Amazon EC2 instance from [the latest Amazon Linux AMI](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
1. Connect to the instance, and then install tools to extract the packages:
    - `sudo yum install -y jwhois rpmdevtools`
1. Download and extract the libraries and the dependencies:
    - `cd /tmp`
    - `yumdownloader jwhois.x86_64`
    - `rpmdev-extract *rpm`
1. Copy the libraries into a directory for the Lambda deployment package:
    - `sudo mkdir -p /var/task`
    - `sudo chown ec2-user:ec2-user /var/task`
    - `cd /var/task`
    - `/bin/cp /tmp/jwhois-4.0-19.7.amzn1.x86_64/usr/* /var/task -R` _(note: replace `jwhois-4.0-19.7.amzn1.x86_64` with the respective directory name)_
1. Copy all dependencies into the Lambda deployment package:
    - `mkdir /var/task/lib`
    - Get a list of all dependencies - `ldd /usr/bin/jwhois`
    - Copy each of the listed dependencies into your deployment package lib folder: eg. `sudo /bin/cp /lib64/libidn.so.11 /var/task/lib/libidn.so.11`
1. Create a deployment package ZIP:
    - `cd /var/task`
    - `zip -r9 /tmp/jwhois-layer.zip *`
1. Download a copy of `jwhois-layer.zip` (eg. via SFTP) and upload as a new layer in your Lambda console
