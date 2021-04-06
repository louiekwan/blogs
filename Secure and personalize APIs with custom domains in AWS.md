
# The benefits of using a custom domain

AWS default domain names are very difficult to remember and not at all user friendly.

There are three main benefits to using a custom domain: branding, trust and manageability.

 - Branding: Using a custom domain for your organizations helps promote your brand. One of the most common best practices is to use a custom domain, by using a custom domain that reflects your brand, you are promoting brand consistency. people won’t need to guess if the site they’re visiting is associated with your company or not when your brand name is right there in the domain.
 - Customer Trust: With a custom domain, your users feel confident that they are providing their credentials to the right party. With all of the fraudulent attempts going around the web nowadays, it’s not surprising that people are wary of clicking on a URL they don’t recognize. Using a custom domain that is clearly connected to your business can encourage customer trust.
 - Manageability: Using a custom domain improves the overall API managment. Containing your authentication services in one place makes your application architecture more maintainable. Applications gain only the access they need and authentication services scale easily. Also, with a custom domain, you can use your own certificate to get an Extended Validation, making phishing harder.


## Offering secure APIs with AWS WAFv2 and certifcate managment

Besides implementing secure HTTPS, we should use private APIs and WAFv2 along with approved encryption algorithms to increase the level of confidence that users are accessing a legitimate service and that their communications remain private and free from interference while offering a level of security and privacy that users expect from your organization.


## Background
API Gateway currently does not support custom domain names for private APIs. 

https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html#apigateway-private-api-design-considerations 

As a work-around, it is possible to make use of ALB to proxy the requests, allowing you to make use of a custom domain name with a private API.

## client -> Corporate Firewall -> Route53 DNS -> VPC -> WAFv2 -> ALB -> interface VPC endpoint -> private API Gateway

You will need to perform the following in addition to the normal private API configuration:

1. Choose a domain name to use for the API which you have control of.
2. Request or import a certificate into ACM for the domain name you chose.

 - [https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-request-public.html)

 - [https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate.html)

3. Create a target group containing the IP addresses of the VPC endpoint's ENIs, protocol set to HTTPS and port set to 443.
4. Create an internal ALB with an HTTPS listener pointing to the created target group, specifying the ACM certificate from step 2. 
 - [https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-application-load-balancer.html](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-application-load-balancer.html) 

5. Create an API Gateway custom domain name for the name you chose and use the ACM certificate you created.
6. Create a base path mapping for the custom domain pointing to the API and stage you require.
The custom domain name and mapping is not to route requests via(private APIs are not supported) but for API Gateway to internally route the request based on the Host header provided by the requester.
 - [https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-custom-domains.html](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-custom-domains.html) 

7. Create a Route53 private hosted zone for the parent domain of the custom domain name, selecting your VPC.
8. Create a DNS CNAME record in the private hosted zone for the chosen name pointing to the DNS A record of the ALB.
 - [https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html) 

9. Allow time for DNS propagation.
10. Call the API using the chosen DNS name and path you defined in the base path mapping.
