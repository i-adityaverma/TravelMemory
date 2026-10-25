# Travel Memory — AWS Deployment Report

**Assignment 2 · Aditya Verma · 25 October 2026**

- **Application:** [https://travel.iadityalabs.xyz](https://travel.iadityalabs.xyz)
- **Repository:** [i-adityaverma/TravelMemory](https://github.com/i-adityaverma/TravelMemory)
- **Region:** Mumbai, `ap-south-1`

## 1. Implementation overview

The Travel Memory MERN application was deployed on two Ubuntu EC2 instances in different Availability Zones. Each instance runs Nginx, the React production build, and the Node.js/Express backend. An AWS Application Load Balancer distributes requests, while both backends use a shared MongoDB Atlas database. Cloudflare provides DNS, and AWS Certificate Manager provides the ALB's HTTPS certificate.

The original frontend and backend source code was retained. Deployment used environment variables, Nginx configuration, a systemd service, and AWS networking.

## 2. Architecture

![Deployment architecture](architecture.svg)

[Editable draw.io diagram](architecture.drawio)

**Request flow:** Browser → ALB HTTPS:443 → Nginx HTTP:80 → Express:3000 → MongoDB Atlas. Nginx serves React files directly and proxies API requests. Cloudflare is **DNS only** and resolves the hostname without proxying application traffic.

| Component | Configuration |
| --- | --- |
| App-01 | `travelmemory-app-01`, t2.micro, ap-south-1b |
| App-02 | `travelmemory-app-02`, t2.micro, ap-south-1a |
| VPC | `vpc-079b7559ae745d27c` |
| AMI | `travelmemory-app-v1`, Private |
| ALB | `travelmemory-alb`, Internet-facing, IPv4 |
| Target group | `travelmemory-app-tg`, Instance targets, HTTP:80 |
| ALB DNS | `travelmemory-alb-1786523090.ap-south-1.elb.amazonaws.com` |

## 3. EC2 preparation

App-01 used Ubuntu 24.04.5 LTS, x86_64, approximately 1 GB RAM, and a 6.8 GB root filesystem. Installed packages were `nginx`, `git`, `curl`, and `ca-certificates`. SSH and initial direct HTTP access were restricted to the operator's public IPv4 /32.

![Instance inspection](screenshots/01-instance-inspection.png)

![App-01 overview](screenshots/03-ec2-overview.png)

![Initial security-group rules](screenshots/04-security-groups.png)

## 4. Source and runtime setup

The application was cloned from [UnpredictablePrashant/TravelMemory](https://github.com/UnpredictablePrashant/TravelMemory). Node.js was installed with nvm; the deployed version was **22.23.4**, with **npm 10.9.9**.

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
nvm install 22
nvm alias default 22
cd ~/TravelMemory/backend
npm ci
```

## 5. MongoDB Atlas and backend configuration

Each EC2 instance's public outbound IP was allowed in Atlas Network Access as a /32 entry. The backend environment file contained:

```dotenv
MONGO_URI="<ATLAS_CONNECTION_STRING>"
PORT=3000
```

The file was restricted with `chmod 600 .env`. The backend was started with `node index.js` and verified using `/hello` and `/trip` on port 3000. Both returned HTTP 200; `/trip` returned persisted database records.

![Initial Atlas access-list entry](screenshots/02-atlas-network-access.png)

![Backend API verification](screenshots/06-backend-api-verification.png)

## 6. Persistent backend service

The [systemd service](../deploy/travelmemory-backend.service) was installed at `/etc/systemd/system/travelmemory-backend.service`. It runs as `ubuntu`, starts on boot, and restarts after process failure.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now travelmemory-backend
sudo systemctl status travelmemory-backend --no-pager
```

**Outcome:** The service was enabled and active, and database requests succeeded.

![Systemd service verification](screenshots/07-backend-systemd.png)

## 7. Frontend configuration and build

The existing `frontend/src/url.js` reads the backend URL from an environment variable. A relative `/api` URL was supplied so the same build works through the instance, ALB, and custom domain.

```bash
cd ~/TravelMemory/frontend
printf 'REACT_APP_BACKEND_URL=/api\n' > .env.production
npm ci
GENERATE_SOURCEMAP=false npm run build
```

**Outcome:** The production build completed with warnings.

## 8. Nginx configuration

The build was copied to `/var/www/travelmemory/`. The [Nginx site configuration](../deploy/travelmemory.nginx.conf) serves React files and forwards `/api/` to `http://127.0.0.1:3000/`. The trailing slash removes the API prefix; React routes fall back to `index.html`.

```bash
sudo mkdir -p /var/www/travelmemory
sudo cp -r ~/TravelMemory/frontend/build/. /var/www/travelmemory/
sudo unlink /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/travelmemory /etc/nginx/sites-enabled/travelmemory
sudo nginx -t
sudo systemctl reload nginx
curl -i --max-time 15 http://127.0.0.1/api/trip
```

**Outcome:** Nginx validation passed and the proxied API returned saved trips. The symlink commands apply to initial setup only.

![Nginx validation and API response](screenshots/08-nginx-verification.png)

![Frontend displaying database records](screenshots/09-frontend-browser.png)

## 9. Application write test

A travel memory was submitted through the UI and appeared in subsequent database reads. An initial attempt failed because Short Description was empty; supplying that required field resolved the save failure without an application source change.

## 10. AMI and second application instance

A private AMI was created from app-01 using **Actions → Image and templates → Create image**. App-02 was launched from this image in **ap-south-1a**, while app-01 runs in **ap-south-1b**. App-02's public outbound IP was added to Atlas.

**Outcome:** Both services started on app-02, and its proxied API returned the same shared database records.

![Available private AMI](screenshots/10-application-ami.png)

![App-02 overview](screenshots/14-second-instance-overview.png)

![App-02 services and API verification](screenshots/15-second-instance-verification.png)

## 11. Security groups and load balancing

The instance group allows HTTP:80 from `travelmemory-alb-sg`, plus direct HTTP and SSH from the operator's /32. Express port 3000 has no public ingress rule. The ALB accepts public HTTP:80 and HTTPS:443.

Both instances were registered in `travelmemory-app-tg` on port 80. The ALB uses public subnets in both AZs. The configured health-check procedure uses HTTP `/api/hello`, success code 200.

**Outcome:** The ALB became Active, both targets became Healthy, and the application was accessible through its DNS name.

![Instance access from the ALB security group](screenshots/12-instance-alb-access.png)

![Target-group configuration](screenshots/13-target-group-configuration.png)

![Active ALB across two AZs](screenshots/16-alb-overview.png)

![Two Healthy application targets](screenshots/17-healthy-targets.png)

## 12. Cloudflare DNS

The following records were added with Auto TTL and **DNS only** proxy status:

| Type | Name | Target |
| --- | --- | --- |
| CNAME | `travel` | `travelmemory-alb-1786523090.ap-south-1.elb.amazonaws.com` |
| A | `direct` | App-01 public IPv4; `3.110.117.89` in the DNS capture |

**Outcome:** The application loaded at `travel.iadityalabs.xyz`. The direct record provides the assignment's EC2 mapping and remains subject to the instance's IP restriction. Public IPs can change after stop/start; historical screenshot addresses must be checked before reuse.

![Cloudflare DNS records](screenshots/18-cloudflare-dns.png)

![Custom-domain HTTP access](screenshots/19-custom-domain-http.png)

## 13. HTTPS configuration

An ACM public certificate for `travel.iadityalabs.xyz` was requested in Mumbai using RSA 2048, DNS validation, and export disabled. Its validation CNAME was added to Cloudflare as DNS only. Once Issued, the certificate was attached to an ALB HTTPS:443 listener forwarding to the target group.

**Outcome:** ACM validation succeeded and the HTTPS listener was configured. TLS terminates at the ALB; traffic from the ALB to Nginx remains HTTP.

![Issued ACM certificate](screenshots/20-acm-certificate-issued.png)

The certificate capture predates attachment; the listener capture below shows it attached.

![ACM validation CNAME](screenshots/21-acm-dns-validation.png)

![ALB HTTPS listener](screenshots/22-alb-https-listener.png)

![Final custom-domain browser page](screenshots/23-custom-domain-https.png)

## 14. Outcomes and failover test

| Validation | Outcome |
| --- | --- |
| Backend and database | HTTP 200 from port 3000 and persisted JSON records |
| Process management | Backend service enabled and active |
| Frontend and Nginx | Production build completed; Nginx validation and API proxy passed |
| Create and read a trip | Saved successfully with required fields and displayed in the application |
| Second instance | Services active and shared database accessible |
| Load balancing | Two Healthy targets across ap-south-1a and ap-south-1b |
| Custom domain and TLS | Cloudflare DNS working; ACM certificate issued and attached to HTTPS listener; final browser screenshot supplied |
| One-instance shutdown | Intermittent 504s initially; after waiting, the application served successfully without configuration changes. Eventual availability on the survivor was confirmed; the delay was not measured and restoration of both targets was not recorded. |

The shutdown result is consistent with the ALB's health-check detection period, but **zero-downtime failover was not demonstrated**. The health endpoint checks Express availability, not database connectivity.

The HTTP:80 listener still forwards rather than redirects to HTTPS. The final browser screenshot hides the URL scheme; an explicit HTTPS curl transcript would strengthen TLS evidence. Dependency audits reported 19 backend and 103 frontend vulnerabilities; remediation was outside this deployment's scope.

The source, report, screenshots, diagram, and deployment configuration are included in the GitHub repository. See the [screenshot index](screenshots/README.md) for all 22 captures and the [submission checklist](submission-checklist.md) for final checks and Vlearn submission.
