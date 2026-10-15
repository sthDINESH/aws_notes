# CloudFront CDN

## What?

- Amazon CloudFront is AWS's Content Delivery Network (CDN) for delivering websites, files, videos, and APIs with lower latency.
- Think of it as a network of local distribution centers: users receive cached content from a nearby edge location instead of always contacting the origin server.
- The **origin** is the source of the content, such as an S3 bucket, Application Load Balancer, ECS service, or custom web server.
- A **distribution** is the CloudFront configuration that connects users to one or more origins.
- An **edge location** is an AWS location that can cache and serve content closer to users.

## How?

- Create a CloudFront distribution and choose an origin, such as an S3 bucket or application load balancer.
- Configure cache behaviors to decide which paths use which origin and how long objects stay cached.
- CloudFront checks the nearest edge location for a cached object.
- On a cache hit, CloudFront returns the object quickly without contacting the origin.
- On a cache miss, CloudFront retrieves the object from the origin, returns it to the user, and may cache it for later requests.
- Use HTTPS with an AWS Certificate Manager certificate and optionally connect a custom domain through Route 53.
- Use Origin Access Control (OAC) when CloudFront should access a private S3 bucket without making the bucket public.
- Configure invalidations when cached content must be refreshed before its normal expiration time.

## Why?

- CloudFront reduces latency by serving content from locations closer to users.
- It reduces repeated requests to the origin, which can lower origin load and cost.
- It can protect and accelerate applications with HTTPS, AWS WAF, and DDoS protection through AWS Shield Standard.
- It supports global delivery for static assets, APIs, media, and dynamic web applications.
- Caching improves performance, especially for files that do not change frequently.

> **Analogy:** The origin is the main warehouse, CloudFront edge locations are local shops, and the cache is the stock kept in each shop for fast customer service.

> **Typical flow:** User -> nearest CloudFront edge location -> cached content or origin -> response to user


## Configure CouldFront

- **Default root object**: The object (file name) to return when a viewer requests the root URL (/) instead of a specific object. **General** > **Edit** > **Settings**

- **Origins**
- **Behaviours**: Controls how cloudfront serves the data from the bucket, eg. Cache Policy
- **Invalidations**: use to invalidate cache
- **Error pages**: to serve error pages based on status code


## Connect AppRunner backend to CloudFront for Frontend Delivery

### The idea
- Requests matching `/api/*` are sent to the Node.js backend running on App Runner.
- All other paths are served from the S3 origin, which contains the frontend files.
- CloudFront provides one public entry point while routing each path to the correct origin.

### Steps
1. **Add the App Runner origin:**
    - Open the CloudFront distribution and go to **Origins**.
    - Select **Create origin** and enter the App Runner default domain name as the origin domain.
    - Use HTTPS for the connection to the origin so API traffic is encrypted in transit.

2. **Create an API behavior:**
    - Open **Behaviors** and select **Create behavior**.
    - Set the **Path pattern** to `/api/*`.
    - Select the App Runner origin as the **Origin**.
    - Keep the S3 origin as the default behavior for all other paths.

3. **Allow the required HTTP methods:**
    - Allow `GET`, `HEAD`, `OPTIONS`, `PUT`, `POST`, `PATCH`, and `DELETE` when the API uses these methods.
    - Include `POST`, `PUT`, `PATCH`, and `DELETE` because API requests may create, update, or remove data.
    - Allow only the methods the application actually needs to reduce unnecessary attack surface.

4. **Disable API caching:**
    - Set the **Cache policy** to **CachingDisabled** for `/api/*` unless the API explicitly supports safe caching.
    - API responses are often user-specific or change frequently, so caching them could return stale data or expose one user's response to another.

5. **Forward request details to App Runner:**
    - Set the **Origin request policy** to **AllViewerExceptHostHeader**.
    - This forwards the viewer's headers, cookies, query strings, and other request information while excluding the original `Host` header.
    - Excluding the viewer `Host` header allows CloudFront to send the App Runner origin host expected by the origin.

> **Simple example:** A user visits `https://www.example.com/api/users`. The browser sends `Host: www.example.com`, but the App Runner origin may be `abc123.us-east-1.awsapprunner.com`. CloudFront forwards the request to App Runner while using the App Runner hostname as the origin `Host` header. It still forwards useful details such as the user's cookies, authorization header, query strings, and request method. This prevents App Runner from receiving a host name it does not recognize while preserving the information needed by the API.

> **Why not forward the viewer `Host` header?** If CloudFront sent `Host: www.example.com` to an App Runner origin that expects its own App Runner hostname, the origin could reject the request or route it incorrectly. `AllViewerExceptHostHeader` avoids that mismatch.

6. **Configure viewer access:**
    - Set **Viewer protocol policy** to **Redirect HTTP to HTTPS** or **HTTPS only**.
    - Use HTTPS so API credentials, cookies, and request data are encrypted between the client and CloudFront.

7. **Deploy and test:**
    - Save the behavior and deploy the CloudFront distribution.
    - Test a frontend path, such as `/`, and an API path, such as `/api/health`.
    - Confirm that frontend requests come from S3 and API requests reach App Runner without unwanted caching.

> **Why these choices matter:** The path behavior separates static frontend delivery from dynamic API traffic. The API behavior points `/api/*` to App Runner, allows the methods the backend needs, disables caching for changing or private responses, and forwards the request data required by the application.


## Route53 configuration and SSL Certificates

### Why is this needed?

- CloudFront needs a trusted TLS certificate to serve the site over HTTPS with a custom domain.
- Route 53 maps the custom domain to the CloudFront distribution so users do not need to use the generated `cloudfront.net` address.
- The ACM certificate must be requested in **US East (N. Virginia) `us-east-1`** for CloudFront, even when the origin or users are in another Region.

### Steps

1. **Request an ACM certificate:**
    - Switch the AWS Console Region to **US East (N. Virginia) `us-east-1`**.
    - Open **AWS Certificate Manager (ACM)** and select **Request**.
    - Choose **Request a public certificate**.
    - Add the domain names that CloudFront should serve, such as `example.com` and `www.example.com`.
    - Choose **DNS validation**, then select **Request**.

2. **Validate the certificate:**
    - Open the certificate in ACM and view the validation records.
    - Select **Create records in Route 53** when Route 53 hosts the domain, or copy the CNAME name and value into the correct DNS provider.
    - Wait until ACM changes the certificate status to **Issued**.
    - Keep the validation CNAME records in DNS so ACM can renew the certificate automatically.

3. **Attach the certificate to CloudFront:**
    - Open the CloudFront distribution and choose **Edit**.
    - Under **Settings**, add the custom domain names as **Alternate domain names (CNAMEs)**.
    - Select the ACM certificate issued in `us-east-1`.
    - Set **Viewer protocol policy** to **Redirect HTTP to HTTPS** or **HTTPS only**.
    - Save the changes and wait for the distribution deployment to complete.

4. **Create Route 53 routing records:**
    - Open **Route 53** and select the existing public hosted zone for the domain.
    - Create an **Alias A record** for the apex domain:
      - **Record name:** Leave blank for `example.com`.
      - **Record type:** `A`.
      - **Alias:** Enabled.
      - **Route traffic to:** Select the CloudFront distribution.
    - Create another **Alias A record** for `www.example.com` if that hostname is also configured in CloudFront.
    - If IPv6 is enabled, create matching **Alias AAAA records**.

5. **Test the configuration:**
    - Open `https://example.com` and `https://www.example.com` in a browser.
    - Confirm that the certificate is valid, static paths are served from S3, and `/api/*` requests reach App Runner.
    - Check CloudFront deployment status and DNS propagation if the domain does not work immediately.

> **Important:** Do not create a normal A record containing a CloudFront IP address. CloudFront uses changing distributed endpoints, so Route 53 Alias records are the correct way to point a domain to the distribution.
