## Launch New EC2 Instance.

- Sign in to the [AWS Console](https://aws.amazon.com/console/)
- Go to Services -> EC2.
- Click **Launch Instance**.

**1. Choose AMI:**
- Go to [Wordpress AMIs](https://bitnami.com/stack/wordpress/cloud/aws/amis) and find the latest version for "Asia Pacific (Sydney)".
- Click it to be redirected to the AWS EC2 launch page, or search for the AMI when launching.

**2. Choose Instance Type:**
- Select **t3a.micro** (or **t3.micro**) for the instance type.
- t3a.micro uses AMD CPUs and costs a little less.
- If a service limit is hit with t3a.micro, just use t3.micro.

**3. Configure Instance:**
Leave all the settings on their defaults, except for:
- Enable termination protection = **Yes**.
- Credit specification = **No**.

**4. Add Storage:**
Leave all the settings on their defaults, except for:
- Delete on Termination = **No**.

**5. Add Tags:**
Add the following tags (case-sensitive):
- "Name": **"instanceName"**.
- "project": **"companyName"**.

**6. Configure Security Groups:**
- Pick **"Select an existing security group"**.
- Select **WordpressPublic** and **DeveloperSSH**.

**Launch Instance.**
- Choose **"Create a new key pair"**.
- Call the key pair the same name as the Name tag (step 5), **"instanceName"**.
- Click **Launch** and select _Create a new key pair_, and give the key pair a name like "company-name".
- Download the key, and save it somewhere on your machine. It is suggested to save all keys to **~/.ssh/**.

**Name the Instance volume**
- Go to AWS -> EC2 -> Volumes.
- Find the new Volume that was created for this instance. It may have a blank name.
- Set the name to the name of the Instance so we can keep track of it.
- Assign the instance with an elastic IP:

## Elastic IP.
When assigning an elastic IP to an instance:
- Please go to the elastic IP in **EC2 -> Elastic IPs** and name the new IP the name of the instance.

## Set up SSH
- Go into `~/.ssh` folder where your SSH key is located.
- Run `chmod 400 MY_KEY.pem`.
- SSH into the instance with `ssh -i MY_KEY.pem bitnami@IP_ADDRESS`.
- Edit `~/.ssh/authorized_keys` and add the following lines to enable our SSH:
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC0IGjTYAd8NE0QmI+Gc3qU1Lq/aVqjRHRVpbI42WtF2REcWNH2sqeYzEHVs3NNwsRndc1u5aINEehu/rjRMZmbRQbzO6eAKioaQ2Dl2qoZ4emUshKp6I7COFqRgcFD3Jgu4Qey3Q469N5WY37fehHt1XBA+PmY7qX64tWTu9fr2/DwnlK/9AlpEUfUGULLrwS3giH5PFB3KdTEblrmQ55yvAWRIFRrGat54BtqGwdS63OtWBGwFETuXzuv2aB5psmenX/79mi7Gwxk9e7BANDUvRW80T+MzvU5SZMhyB/IEQtTljxbYy7BUlyTKeRYkYi01OPaoxYJQUJdNSJ/t/yz heirloomhoney

# JosephDunne
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQD36J4SqpVox9bv+aHDLA/MUaPJss8pYUjaVcy2SbN8Wr/605Cu0biye7zqA0XrEh9xQEwEAKQIrHUnJwPGwnvwF733MgtI+kqBsXjsVWYNqnvoM5DwtB28tGJ5RZehdIDjhRTOwcpk+UGsya4awOsj8i6g1vwNEbgmzdo5/10Ay4azHnAPWxlq7yCI72vbmES0mXJlN55S2Fc9EYRlKfpTnOYpIxTAw9avE1+m7WER3Dz6IWhVZRqCZWQ+zaVuhyG2LAr7JzBAK1MRFK9YbuJWCZBdz1SkylNuAr85EJ++BqU7wGRlhRRk3FdY8x8VIKuAywdlt11jy69XrRYHq/tcD8dFAwPgClzOnwY/NUxLz7MA4BQlqDWRTiaKxQ1nH4vIqIJutY4Tg6O88cvIaioGySdQkJ+QWvJFdCUdQqtny444La4zp6qR6ULTy5VZs0L5lv7IQnEnjZa9yX4nE5VI6urTK06xb3cJ0clYNiNPnfdERz62IJSim0xUBYhEeJ24eRjsf4Jha0xUyUTOxyGGYJPqe+VlxC1NSz7RaNDPubftYTNy5VF/A7/VnZMbfMTiEQuC4kQ4LiWXEC8uDU43bchbU8BoAVEXiJZg4W47TPUZg1Mk/1j58XktdHfhYupiAWW338OAnBr+mfmHVWigk2oH6PvAEjK7VwQny8hs2w== josephdunne1993@gmail.com

# rajul 2

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpmdXjvyinSjX58X+J2DEpulAXodaICF4jSs7xHF6Dr3zFVPPQdjJzDn/kfJn2PAvaWA4QRGPU08S/YAETTOCiGcM5FXqBXR/qbMn1ohfZb7r+NoOvMb4YBTNgjOZxI7dOtNM39igQ72rpn9ZjP0gqHPOEsV/HBnp+5+dsHL6uuYiOXJ1HqwJzZMcjZqqUDb+hglT1MoIOGXx7S8cs/RQRNRYHMG3e9yifhFQOShBBte1XidUthg9Jht8yOhUHz/wQETwHg8PbJgAASCuDSHx7HRlegMwwnnVi6RvV/IaJW64u1FEFlCiZhv93/QHcVR9kCJ29TFXpNL3eevR/LQoSCTJU5kWtlGN7pJrvXZTM/4QefgQ+ZS8boyCivetJuA/KPHfpHdog520twDY9+y//Eoa/qtkPi9zscwJm5WFAVWXtYWq/b6PWTwmMj5x/tGeMFqfcDR98Rj0cY4PIAAbCvgNXfFfQJ8CORZVpBa+RiFk63of+TDzCG7vuQP2VsOU= OKMG@180-150-90-35.b4965a.per.nbn.aussiebb.net

# MathewBowyer
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKU5Qwc2dNJEcUh80BCGAA/qD72fc0Ak/LWJ9WriBW74 mathewbowyer@hotmail.com




```
- Exit the instance.
- Add an entry in your `~/.ssh/config` for the instance that uses your `id_rsa` key.
- Test SSH. If it works, it should be safe to backup and remove the initial key created for the instance, `MY_KEY.pem`.

## Setting the Timezone
- Check that the correct timezone is being used by running `date`.
- If it is not correct, run `sudo timedatectl set-timezone Australia/Perth`.
- Confirm new timezone by running `date`.
- Now reboot the instance with `sudo reboot` so that services (like cron) can use the new timezone.

## Remove the Bitnami banner and /bitnami pages.
- SSH in with `ssh INSTANCE_NAME`.
- Edit file with `nano /opt/bitnami/apache2/conf/httpd.conf`
- Near the bottom of the file, **comment out** the line like so: `# Include "/opt/bitnami/apps/bitnami/banner/conf/banner.conf"`.
- Restart apache with `sudo service bitnami restart apache`.
- Confirm that the site is live and the banner is removed.

## Backup the temporary key.
- Create password-protected version of `MY_KEY.pem` with `openssl rsa -aes256 -in MY_KEY.pem -out MY_KEY-priv.pem`. **Ask Staff for what the password should be**.
- Backup the key to S3 with `aws s3 cp ./MY_KEY-priv.pem s3://okmg-developers/keys/MY_KEY-priv.pem`.

## Adding SSL.
- Follow this guide: [Bitnami Guide](https://docs.bitnami.com/aws/how-to/understand-bncert/)
- Using the Bncert tool should be enough to set up SSL.

## Free up space.

```
sudo apt clean
sudo apt autoclean
sudo apt autoremove
sudo apt --purge autoremove
```
