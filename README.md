## **Infrastructure**

### **Server**: 
- **AWS EC2**: This is the virtual server where your applications are running.

### **Load Balancer**: 
- **AWS ALB (Application Load Balancer)**: Sits in front of your EC2 instance. It distributes incoming application traffic across multiple targets, such as EC2 instances. 

### **Certificate Management**: 
- **ACM (AWS Certificate Manager)**: We're using ACM to handle SSL certificates. This will allow our websites to be accessed over HTTPS.

### **Domain**: 
- **power365logistics.com**: This is your primary domain.
- **dev.power365logistics.com**: This is our subdomain for the development environment.

---

## **Containers**

### **nginx-main**:
- **Purpose**: Serves the main website.
- **Ports**: Listens on port 80 inside the container, which is mapped to port 80 of our host.
- **Volume**: Data from `/deploy/main-website` on your server is mounted to `/usr/share/nginx/html` in the container, making your website's files available to the nginx server.
  
### **nginx-dev**:
- **Purpose**: Serves the development version of the website.
- **Ports**: Listens on port 80 inside the container, mapped to port 8080 of our host.
- **Volume**: Data from `/deploy/dev-website` is mounted to `/usr/share/nginx/html` in the container.

---

## **Deployment Process**

### **GitHub Workflows**:

- **For Development (dev branch)**: When you push to the `dev` branch, the following occurs:
  1. Code is checked out from the repo.
  2. SSH setup is initialized.
  3. Code is synchronized to the `/deploy/dev-website` directory on the EC2 instance.
  4. The `nginx-dev` container is rebuilt and restarted.

- **For Main (main branch)**: When you push to the `main` branch, a similar process occurs, but the code is synchronized to the `/deploy/main-website` directory, and the `nginx-main` container is the one rebuilt and restarted.

---

## **Routing & Security**

### **ALB Rules**:
- If a request comes with the host header `dev.power365logistics.com`, it gets routed to the `nginx-dev` target group, which then directs the traffic to the `nginx-dev` container listening on port 8080.
  
- For any request coming over HTTP to either `power365logistics.com` or `dev.power365logistics.com`, there's a rule in the ALB to redirect that request to its HTTPS counterpart. This ensures secure communication.

### **Security Groups**: 
- Ensure that incoming traffic is allowed on the necessary ports (80 for HTTP and 8080 for the dev environment).

---

## **Conclusion**

When a user visits `power365logistics.com`, their request goes to the ALB, which then routes it to the `nginx-main` container. If they visit `dev.power365logistics.com`, the ALB routes the request to the `nginx-dev` container.

Whenever you make changes to your application and push to either the `dev` or `main` branches, the respective containers are updated with the new changes.

For security, any HTTP request is automatically redirected to HTTPS, ensuring encrypted and secure communication.

This setup provides a clear separation between our production (`main`) and development (`dev`) environments while leveraging Docker for consistency and scalability. AWS services like ALB and ACM further enhance the setup by providing efficient load balancing and secure SSL certificate management, respectively.
