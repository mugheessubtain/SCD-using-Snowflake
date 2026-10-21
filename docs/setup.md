# Environment setup

## 1. EC2 instance

- Launch an Amazon Linux instance (t2.xlarge with 32 GB storage works well for NiFi + JupyterLab).
- In the security group, allow the ports you need (JupyterLab `4888`, NiFi `2080`), or use SSH port forwarding instead:

```bash
ssh -i "<key>.pem" ec2-user@<EC2_PUBLIC_DNS> -L 4888:localhost:4888 -L 2080:localhost:2080
```

- Copy the compose file to the instance:

```bash
scp -r -i "<key>.pem" docker_exp ec2-user@<EC2_PUBLIC_DNS>:/home/ec2-user/docker_exp
```

## 2. Docker and Docker Compose

```bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo yum install -y libxcrypt-compat

sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## 3. Start the containers

```bash
cd docker_exp
sudo docker-compose up -d
docker ps
```

| Service | URL |
|---|---|
| JupyterLab | http://localhost:4888/lab |
| Apache NiFi | http://localhost:2080/nifi/ |

All containers share the `/opt/workspace` volume, so files generated in JupyterLab are visible to NiFi.

## 4. Generate test data

Open `faker.ipynb` in JupyterLab and run it. It writes `FakeDataset/customer_<timestamp>.csv` with 10,000 customer records. Then copy the file into NiFi's input folder:

```bash
docker exec -it nifi bash
mkdir -p scd
cp /opt/workspace/nifi/FakeDataset/customer_<timestamp>.csv scd/
```

## 5. Push files to S3 with NiFi

In the NiFi UI, build a flow that picks up files from the `scd/` folder and uploads them to your S3 bucket, using an IAM access key with write access to that bucket.
