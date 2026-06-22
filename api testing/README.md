# API Testing

This section contains my walkthroughs and solutions for the API Testing labs from PortSwigger Web Security Academy.

API vulnerabilities are increasingly common in modern applications due to the widespread use of REST APIs, mobile backends, microservices, and third-party integrations. These labs demonstrate how insecure API implementations can expose sensitive functionality, allow privilege escalation, and lead to unauthorized access to data.

The labs in this section focus on identifying hidden API endpoints, abusing undocumented functionality, exploiting mass assignment vulnerabilities, and manipulating server-side API requests.

---

## Labs Completed

### API Reconnaissance & Documentation Abuse

- [x] Exploiting an API endpoint using documentation

### Mass Assignment Vulnerabilities

- [x] Exploiting a mass assignment vulnerability

### Hidden / Deprecated API Endpoints

- [x] Finding and exploiting an unused API endpoint

### Server-Side Parameter Pollution

- [x] Exploiting server-side parameter pollution in a query string
- [x] Exploiting server-side parameter pollution in a REST URL

---

## Skills Practiced

- API Endpoint Discovery
- REST API Testing
- API Documentation Enumeration
- OpenAPI / Swagger Analysis
- Hidden Endpoint Discovery
- HTTP Method Enumeration
- PATCH Request Manipulation
- Parameter Tampering
- Mass Assignment Exploitation
- Server-Side Parameter Pollution
- Query String Manipulation
- REST Path Manipulation
- Password Reset Abuse
- Account Takeover Techniques
- Authorization Testing
- Burp Suite API Testing

---

## Tools Used

- Burp Suite Proxy
- Burp Repeater
- Burp Intruder
- Browser Developer Tools
- PortSwigger Web Security Academy

---

## Key Takeaways

Through these labs I learned:

- how modern APIs expose application functionality
- how API documentation can reveal sensitive endpoints
- how hidden or deprecated endpoints can still remain accessible
- how mass assignment vulnerabilities occur when user-controlled parameters are blindly mapped to backend objects
- how server-side parameter pollution can manipulate internal API requests
- how password reset functionality can be abused through API flaws
- how API authorization weaknesses can lead to privilege escalation and account takeover

These labs significantly improved my understanding of modern API security testing and common API attack surfaces found during penetration tests and bug bounty engagements.
