# Deployment screenshot index

All 22 supplied screenshots have been reviewed and referenced in [the deployment report](../deployment.md). The numbering retains the capture plan; no screenshot 11 was supplied.

| File | Evidence |
| --- | --- |
| [01-instance-inspection.png](01-instance-inspection.png) | Initial Ubuntu, resources, and runtime inspection |
| [02-atlas-network-access.png](02-atlas-network-access.png) | Initial Atlas IP access-list entry |
| [03-ec2-overview.png](03-ec2-overview.png) | Initial first-instance overview |
| [04-security-groups.png](04-security-groups.png) | Initial restricted SSH and HTTP rules |
| [05-ec2-overview-full.png](05-ec2-overview-full.png) | Full first-instance console overview |
| [06-backend-api-verification.png](06-backend-api-verification.png) | Direct backend API checks on port 3000 |
| [07-backend-systemd.png](07-backend-systemd.png) | Enabled and active systemd backend service |
| [08-nginx-verification.png](08-nginx-verification.png) | Nginx validation and proxied database read |
| [09-frontend-browser.png](09-frontend-browser.png) | Frontend browser with saved travel memories |
| [10-application-ami.png](10-application-ami.png) | Private application AMI in Available state |
| [12-instance-alb-access.png](12-instance-alb-access.png) | Instance HTTP access from the ALB security group |
| [13-target-group-configuration.png](13-target-group-configuration.png) | Target-group configuration before registration |
| [14-second-instance-overview.png](14-second-instance-overview.png) | Second application instance overview |
| [15-second-instance-verification.png](15-second-instance-verification.png) | Second-instance services and database connectivity |
| [16-alb-overview.png](16-alb-overview.png) | Active ALB across two Availability Zones |
| [17-healthy-targets.png](17-healthy-targets.png) | Two Healthy registered targets |
| [18-cloudflare-dns.png](18-cloudflare-dns.png) | Cloudflare application CNAME and direct A record |
| [19-custom-domain-http.png](19-custom-domain-http.png) | Custom-domain HTTP browser page |
| [20-acm-certificate-issued.png](20-acm-certificate-issued.png) | Issued ACM certificate before listener attachment |
| [21-acm-dns-validation.png](21-acm-dns-validation.png) | DNS-only ACM validation CNAME |
| [22-alb-https-listener.png](22-alb-https-listener.png) | ALB HTTP and HTTPS listener configuration |
| [23-custom-domain-https.png](23-custom-domain-https.png) | Final custom-domain browser page supplied as HTTPS evidence |

Earlier screenshots contain historical public IPs. The final AZ placement is app-01 in ap-south-1b and app-02 in ap-south-1a. Screenshot 20 predates certificate attachment; screenshot 22 shows attachment. Screenshot 22 still shows HTTP forwarding rather than redirection. Screenshot 23 hides the URL scheme, so an explicit HTTPS curl transcript would provide stronger TLS evidence.

Not supplied: final ALB security-group rules, Health checks tab, updated Atlas entries for final public IPs, and controlled failure/recovery evidence. These are not represented as completed tests.
