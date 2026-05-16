# Assignment1_AWS_CE

# UniEvent – AWS Cloud Deployment

Cloud Computing Assignment 1
Course: CE 308

## Repository Name

`Assignment1_AWS_CE`

## Project Overview

UniEvent is a cloud-hosted university event management platform where students can:

* View university events
* Register for events
* View event posters/images
* Fetch event data automatically from an external Open API

The system is deployed on AWS with a **scalable, secure, and fault-tolerant architecture**.

---

# Architecture Used

AWS services used in this project:

* IAM – Access control
* VPC – Network isolation
* EC2 – Web application hosting
* S3 – Storage for event images/posters
* Elastic Load Balancer – Traffic distribution

### Architecture Flow

1. Users access the web application.
2. Traffic goes through **Elastic Load Balancer**.
3. Load balancer distributes traffic to **multiple EC2 instances**.
4. EC2 instances run the **UniEvent web application**.
5. The application fetches event data from an **Open Events API**.
6. Event posters/images are stored in **S3 bucket**.
7. If one EC2 instance fails, another instance continues serving users.

---

# Step 0 — Login to AWS

# Step 1 — Create IAM User

Purpose: **Security (do not use root account)**

1. Go to **IAM**
2. Click **Users → Create user**
3. Username:

```
cloud-assignment-user
```

4. Enable:

* Console access
* Custom password

5. Attach policy:

```
AdministratorAccess
```

6. Create user and login with that user.

---

# Step 2 — Create VPC

Purpose: **Isolated network for our application**

1. Go to **VPC**
2. Click **Create VPC**
3. Choose:

```
VPC only
```

4. Configure:

```
Name: UniEvent-VPC
IPv4 CIDR: 10.0.0.0/16
```

Create.

---

# Step 3 — Create Subnets

You need **Public and Private Subnets**.

### Public Subnet

1. Go to **Subnets → Create subnet**

```
Name: Public-Subnet
CIDR: 10.0.1.0/24
```

### Private Subnet

Create another:

```
Name: Private-Subnet
CIDR: 10.0.2.0/24
```

---

# Step 4 — Create Internet Gateway

1. Go to **Internet Gateway**
2. Click **Create**

```
Name: UniEvent-IGW
```

3. Attach it to **UniEvent-VPC**

---

# Step 5 — Configure Route Table

1. Go to **Route Tables**
2. Create new:

```
Name: Public-Route-Table
```

3. Add route:

```
0.0.0.0/0 → Internet Gateway
```

4. Associate with **Public Subnet**

Now the public subnet has internet.

---

# Step 6 — Create Security Group

1. Go to **EC2 → Security Groups**
2. Create:

```
Name: UniEvent-SG
```

Add rules:

Inbound Rules

```
SSH   22   My IP
HTTP  80   Anywhere
```

---

# Step 7 — Launch EC2 Instances

You need **two EC2 instances for scalability**.

Go to **EC2 → Launch Instance**

Configuration:

```
Name: UniEvent-Server-1
AMI: Ubuntu
Instance Type: t2.micro
Key Pair: create new
Network: UniEvent-VPC
Subnet: Private Subnet
Security Group: UniEvent-SG
```

Launch.

Now launch **second instance**

```
UniEvent-Server-2
```

---

# Step 8 — Connect to EC2

Click instance → **Connect → SSH**

Example:

```
ssh -i key.pem ubuntu@public-ip
```

---

# Step 9 — Install Web Server

Run inside EC2:

```
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
```

Test:

Open browser:

```
http://EC2-PUBLIC-IP
```

Apache page should appear.

---

# Step 10 — Upload Web Application

Create your webpage.

```
cd /var/www/html
sudo nano index.html
```

Code:

```html
<!DOCTYPE html>
<html>
<head>
<title>UniEvent - University Event Portal</title>

<style>

body{
font-family: Arial;
background-color:#f4f4f4;
text-align:center;
}

h1{
color:#2c3e50;
}

#events{
margin-top:30px;
}

.event-card{
background:white;
padding:20px;
margin:20px;
border-radius:10px;
box-shadow:0 0 10px rgba(0,0,0,0.1);
}

</style>

</head>

<body>

<h1>UniEvent - University Events Portal</h1>

<div id="events"></div>

<script>

const API_KEY = "5ky7GgkhDi76GpKkKbM8gJcvptBhKkch";

const url = "https://app.ticketmaster.com/discovery/v2/events.json?size=5&apikey=" + API_KEY;

fetch(url)
.then(response => response.json())
.then(data => {

let eventsHTML = "";

data._embedded.events.forEach(event => {

eventsHTML += `
<div class="event-card">

<h2>${event.name}</h2>

<p><b>Date:</b> ${event.dates.start.localDate}</p>

<p><b>Venue:</b> ${event._embedded.venues[0].name}</p>

</div>
`;

});

document.getElementById("events").innerHTML = eventsHTML;

})
.catch(error => {
document.getElementById("events").innerHTML = "Error loading events.";
});

</script>

</body>
</html>
```

Save.

Repeat on **second EC2 instance**.

---

# Step 11 — Create S3 Bucket

Purpose: **store event images**

1. Go to **S3**
2. Create bucket

```
Name: unievent-images-bucket
Region: same as EC2
```

3. Upload images.

Example:

```
event1.jpg
event2.jpg
```

---

# Step 12 — Create Load Balancer

Now make system **fault tolerant**.

1. Go to **EC2 → Load Balancers**
2. Create **Application Load Balancer**

Configuration:

```
Name: UniEvent-LB
Scheme: Internet-facing
VPC: UniEvent-VPC
Subnets: Public Subnet
```

Add target group:

Add both instances

```
UniEvent-Server-1
UniEvent-Server-2
```

Create.

---

# Step 13 — Test Load Balancer

Open browser:

```
LoadBalancer-DNS-name
```

Your website should appear.

Stop **Instance 1**.

Refresh website.

It will still work → because **instance 2 handles traffic**.

💡 This demonstrates **fault tolerance**.

---

# Final Architecture

```
Users
   │
   │
Load Balancer
   │
 ┌───────────────┐
 │               │
EC2 Instance 1   EC2 Instance 2
 │               │
 └───────┬───────┘
         │
        S3
 (Event Images)
         │
     Events API
```

---
