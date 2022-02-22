# ansible-terraform-scripts

Using Ansible to create S3 keys for each host.

## Connecting to Ansible

- Use the [CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) to connect to Ansible.

```bash
sudo apt update
sudo apt install python3-pip
pip3 install awscli
pip3 install ansible
```

- Install the Boto SDK

```bash
pip3 install boto boto3
```

## Create a hosts file

```bash
/etc/ansible/hosts
```

to contain

```bash
<host1> ansible_port=5555 ansible_host=<unique ip>
<host2> ansible_port=5555 ansible_host=<unique ip>
```

## Create S3 buckets for each of your hosts

- Install ansible-galaxy collection

```bash
ansible-galaxy collection install amazon.aws
```

OR
If you are in a VM, and want to install ansible-galaxy in your local machine, run the following command

```bash
ansible-galaxy collection install amazon.aws -p
```

- Create a variables file to contain your configuration details

```bash
nano vars.yml
```

to contain

```bash
aws_access_key1 <some-value>
aws_secret_key1 <some-value>
aws_ec2_region <some-value>

aws_access_key2 <some-value>
aws_secret_key2 <some-value>
aws_ec2_region2 <some-value>

```

- Save your configuration details in ansible vault by running:

```bash
ansible-vault encrypt <file-name.yml>
```

You will be prompted to enter a password twice.

- Create a playbook to create an S3 bucket for yourself.

```bash
nano bucket.yml
```

to contain

```bash
---
- name: Create S3 bucket for user 1
  host: 
    <alias>
  connection: local
  auth:
          aws_access_key: "{{ my_access_key1 }}"
          aws_secret_key: "{{ my_secret_key1 }}"
          aws_ec2_region: "{{ my_region1 }}"
  gather_facts: false
  strategy: free
  roles:
    - PUT/upload with custom headers for user 1
  delegate_to: <ip-address>
```

- Create a role.yml file

```bash
- name: PUT/upload with custom headers for user 1
  amazon.aws.aws_s3:
    bucket: <bucket_name> #mybucket
    object: /my/desired/key.txt #key/file path
    src: /usr/local/myfile.txt #file to be uploaded into S3 bucket
    mode: put
    headers: 'x-amz-grant-full-control=emailAddress=owner@example.com'
```

- To view all your keys

```bash
- name: List keys simple
  amazon.aws.aws_s3:
    bucket: mybucket
    mode: list
```

## Run the ansible playbook

```bash
AWS_PROFILE=work ansible-playbook bucket.yml --ask-vault-password
```

- List your S3 buckets

```bash
aws s3 ls
```

## References

- [Managing S3 objects](https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_s3_module.html#examples)
